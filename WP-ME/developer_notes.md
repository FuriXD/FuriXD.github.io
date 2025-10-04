# Developer Notes: Third-Party Integrations & Cost Management

## Overview

This document covers integration requirements, SDKs, rate limits, cost estimates, and implementation considerations for all third-party services used in the travel planning platform.

---

## Service Integrations

### 1. **@mapbox** - Map Rendering & Geocoding

**Purpose**: Interactive maps, GeoJSON visualization, route plotting

**SDK/Library**: 
- `mapbox-gl-js` v2.15+ (client-side)
- `@mapbox/mapbox-sdk` (Node.js server-side)

**Installation**:
```bash
npm install mapbox-gl @mapbox/mapbox-sdk
```

**Rate Limits**:
- Free tier: 50,000 map loads/month
- Geocoding: 100,000 requests/month
- Static images: 50,000 requests/month

**Cost Estimates**:
| Usage | Cost |
|-------|------|
| 100k map loads | $0.50 per 1k = $50/month |
| 100k geocoding requests | $0.50 per 1k = $50/month |
| 10k static images | Free (under 50k) |

**Implementation Notes**:
```javascript
import mapboxgl from 'mapbox-gl';

mapboxgl.accessToken = process.env.MAPBOX_ACCESS_TOKEN;

const map = new mapboxgl.Map({
  container: 'map-container',
  style: 'mapbox://styles/mapbox/streets-v12',
  center: [139.7004, 35.6595],
  zoom: 12
});

// Add GeoJSON source
map.on('load', () => {
  map.addSource('itinerary', {
    type: 'geojson',
    data: '/api/v1/map/geojson?itinerary_id=itn_123'
  });
});
```

**Rate Limit Handling**:
```javascript
// Client-side caching
const cachedMaps = new Map();

function loadMap(itineraryId) {
  if (cachedMaps.has(itineraryId)) {
    return cachedMaps.get(itineraryId);
  }
  // Load from API
}
```

**Alternative**: Use `Leaflet.js` with free OpenStreetMap tiles (no API key required)

---

### 2. **@google** - OAuth Authentication

**Purpose**: Social login with Google accounts

**SDK/Library**:
- `@react-oauth/google` (React)
- `google-auth-library` (Node.js)

**Installation**:
```bash
npm install @react-oauth/google google-auth-library
```

**Rate Limits**:
- 10,000 requests/day (OAuth API)
- Unlimited authentications (no quota)

**Cost**: Free (no charges for OAuth)

**Implementation**:
```javascript
// Client-side (React)
import { GoogleOAuthProvider, GoogleLogin } from '@react-oauth/google';

function LoginPage() {
  return (
    <GoogleOAuthProvider clientId={process.env.GOOGLE_CLIENT_ID}>
      <GoogleLogin
        onSuccess={credentialResponse => {
          fetch('/api/v1/auth/google', {
            method: 'POST',
            body: JSON.stringify({ 
              id_token: credentialResponse.credential 
            })
          });
        }}
      />
    </GoogleOAuthProvider>
  );
}

// Server-side (Node.js)
const { OAuth2Client } = require('google-auth-library');
const client = new OAuth2Client(process.env.GOOGLE_CLIENT_ID);

async function verifyGoogleToken(idToken) {
  const ticket = await client.verifyIdToken({
    idToken,
    audience: process.env.GOOGLE_CLIENT_ID
  });
  return ticket.getPayload();
}
```

**Security Considerations**:
- Always verify tokens server-side
- Check token expiration
- Validate audience matches your client ID

---

### 3. **@twilio** - SMS OTP for Phone Authentication

**Purpose**: Send verification codes via SMS

**SDK/Library**: `twilio` v4.0+

**Installation**:
```bash
npm install twilio
```

**Rate Limits**:
- 1,000 messages/second (per account)
- Recommend: 3 SMS per phone per hour (application-level)

**Cost Estimates**:
| Region | Cost per SMS |
|--------|--------------|
| USA | $0.0079 |
| Japan | $0.085 |
| India | $0.0087 |
| Europe | $0.10 |

**Monthly estimate**: 10,000 OTPs × $0.01 avg = **$100/month**

**Implementation**:
```javascript
const twilio = require('twilio');
const client = twilio(
  process.env.TWILIO_ACCOUNT_SID,
  process.env.TWILIO_AUTH_TOKEN
);

async function sendOTP(phoneNumber, otp) {
  try {
    const message = await client.messages.create({
      body: `Your TravelPlanner verification code is: ${otp}. Valid for 5 minutes.`,
      from: process.env.TWILIO_PHONE_NUMBER,
      to: phoneNumber
    });
    return message.sid;
  } catch (error) {
    console.error('Twilio error:', error);
    throw new Error('Failed to send OTP');
  }
}
```

**Rate Limit Implementation**:
```javascript
const Redis = require('ioredis');
const redis = new Redis(process.env.REDIS_URL);

async function checkOTPRateLimit(phoneNumber) {
  const key = `otp:ratelimit:${phoneNumber}`;
  const count = await redis.incr(key);
  
  if (count === 1) {
    await redis.expire(key, 3600); // 1 hour
  }
  
  if (count > 3) {
    throw new Error('Rate limit exceeded. Try again in 1 hour.');
  }
}
```

**Cost Optimization**:
- Use regional phone numbers (cheaper)
- Implement CAPTCHA before OTP send
- Cache OTPs for resend requests (don't send duplicate)

---

### 4. **@stripe** - Payment Processing (Optional)

**Purpose**: Accept payments for premium features, bookings

**SDK/Library**: `stripe` v11.0+, `@stripe/react-stripe-js`

**Installation**:
```bash
npm install stripe @stripe/react-stripe-js @stripe/stripe-js
```

**Rate Limits**:
- 100 requests/second (API)
- 1,000 requests/second (webhooks)

**Cost**:
- 2.9% + $0.30 per successful charge (USA)
- 3.25% + $0.30 (international cards)
- No monthly fees

**Implementation**:
```javascript
// Server-side
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);

async function createPaymentIntent(amount, currency) {
  const paymentIntent = await stripe.paymentIntents.create({
    amount: amount * 100, // cents
    currency,
    metadata: { itinerary_id: 'itn_123' }
  });
  return paymentIntent.client_secret;
}

// Webhook handler
app.post('/webhook/stripe', async (req, res) => {
  const sig = req.headers['stripe-signature'];
  const event = stripe.webhooks.constructEvent(
    req.body,
    sig,
    process.env.STRIPE_WEBHOOK_SECRET
  );
  
  if (event.type === 'payment_intent.succeeded') {
    // Handle successful payment
  }
  
  res.json({ received: true });
});
```

**Security**:
- Never expose secret key client-side
- Use Stripe.js for PCI compliance
- Verify webhook signatures

---

### 5. **@aws_s3** - Cloud Storage

**Purpose**: Store itinerary PDFs, user uploads, backups

**SDK/Library**: `@aws-sdk/client-s3` v3.0+

**Installation**:
```bash
npm install @aws-sdk/client-s3 @aws-sdk/s3-request-presigner
```

**Rate Limits**:
- 3,500 PUT/LIST requests per second per prefix
- 5,500 GET/HEAD requests per second per prefix

**Cost Estimates**:
| Usage | Cost |
|-------|------|
| 10 GB storage | $0.023/GB = $0.23/month |
| 100k PUT requests | $0.005/1k = $0.50/month |
| 1M GET requests | $0.0004/1k = $0.40/month |
| 100 GB data transfer out | $0.09/GB = $9/month |

**Monthly estimate (moderate usage)**: **~$15-30/month**

**Implementation**:
```javascript
const { S3Client, PutObjectCommand, GetObjectCommand } = require('@aws-sdk/client-s3');
const { getSignedUrl } = require('@aws-sdk/s3-request-presigner');

const s3 = new S3Client({ region: 'us-east-1' });

// Upload itinerary PDF
async function uploadItinerary(userId, itineraryId, pdfBuffer) {
  const key = `itineraries/${userId}/${itineraryId}.pdf`;
  
  await s3.send(new PutObjectCommand({
    Bucket: process.env.S3_BUCKET_NAME,
    Key: key,
    Body: pdfBuffer,
    ContentType: 'application/pdf',
    ServerSideEncryption: 'AES256',
    Metadata: {
      userId,
      itineraryId,
      createdAt: new Date().toISOString()
    }
  }));
  
  return key;
}

// Generate signed URL (1 hour expiration)
async function getDownloadUrl(key) {
  const command = new GetObjectCommand({
    Bucket: process.env.S3_BUCKET_NAME,
    Key: key
  });
  
  return await getSignedUrl(s3, command, { expiresIn: 3600 });
}
```

**Cost Optimization**:
- Use S3 Lifecycle policies to move old files to Glacier ($0.004/GB)
- Enable intelligent tiering
- Use CloudFront CDN to reduce data transfer costs
- Compress PDFs before upload

**Storage Structure**:
```
s3://travel-itineraries/
├── itineraries/
│   ├── usr_123/
│   │   ├── itn_abc.pdf
│   │   └── itn_def.pdf
├── uploads/
│   └── usr_123/
│       └── avatar.jpg
└── backups/
    └── 2025-10-04/
        └── database.sql.gz
```

---

### 6. **@firebase_auth** - Alternative to Custom Auth

**Purpose**: Managed authentication with OAuth, phone, email

**SDK/Library**: `firebase` v10.0+

**Installation**:
```bash
npm install firebase
```

**Rate Limits**:
- Unlimited authentications (Spark/Blaze plans)
- 10k SMS verifications/month (free)
- Additional SMS: $0.01-0.06 per verification

**Cost**:
- Free tier: 10k phone auths/month
- Blaze plan: Pay-as-you-go after free tier

**Implementation**:
```javascript
import { initializeApp } from 'firebase/app';
import { getAuth, signInWithPopup, GoogleAuthProvider } from 'firebase/auth';

const app = initializeApp({
  apiKey: process.env.FIREBASE_API_KEY,
  authDomain: 'travelplanner.firebaseapp.com',
  projectId: 'travelplanner'
});

const auth = getAuth(app);
const provider = new GoogleAuthProvider();

// Google Sign-In
async function signInWithGoogle() {
  const result = await signInWithPopup(auth, provider);
  const idToken = await result.user.getIdToken();
  // Send to backend for verification
}
```

**Advantages over Custom Auth**:
- No server-side token management
- Built-in rate limiting
- Automatic security updates
- Lower development cost

---

### 7. **@openexchangerates** or **@xe** - Currency Conversion

**Purpose**: Real-time currency exchange rates

**SDK/Library**: REST API (no official SDK)

**Installation**:
```bash
npm install axios
```

**Rate Limits**:
- Free plan: 1,000 requests/month
- Paid: 100,000 requests/month ($12/month)

**Cost**: $12/month (paid plan)

**Implementation**:
```javascript
const axios = require('axios');

async function getCurrencyRates(baseCurrency = 'USD') {
  const response = await axios.get(
    `https://openexchangerates.org/api/latest.json?app_id=${process.env.OXR_API_KEY}&base=${baseCurrency}`
  );
  return response.data.rates;
}

// Cache rates (update hourly)
const NodeCache = require('node-cache');
const ratesCache = new NodeCache({ stdTTL: 3600 });

async function convertCurrency(amount, from, to) {
  let rates = ratesCache.get('rates');
  
  if (!rates) {
    rates = await getCurrencyRates();
    ratesCache.set('rates', rates);
  }
  
  const usdAmount = amount / rates[from];
  return usdAmount * rates[to];
}
```

**Cost Optimization**:
- Cache rates for 1-6 hours
- Use free tier for development
- Alternative: European Central Bank API (free, limited currencies)

---

### 8. **@booking** / **@hotels** - Hotel Booking APIs

**Purpose**: Fetch hotel availability, pricing, booking

**SDK/Library**: Booking.com API or Hotels.com API

**Rate Limits**:
- Varies by partner agreement
- Typically: 10-50 requests/second

**Cost**:
- Commission-based: 8-15% per booking
- No API usage fees

**Implementation** (conceptual):
```javascript
async function searchHotels(location, checkIn, checkOut) {
  const response = await fetch(
    `https://api.booking.com/hotels/search?` +
    `location=${location}&check_in=${checkIn}&check_out=${checkOut}`,
    {
      headers: {
        'Authorization': `Bearer ${process.env.BOOKING_API_KEY}`
      }
    }
  );
  return response.json();
}
```

**Alternatives**:
- Amadeus API (flight + hotel)
- Expedia Partner Solutions
- Direct hotel chain APIs (Marriott, Hilton)

---

### 9. **PDF Generation** - @puppeteer or wkhtmltopdf

**Purpose**: Generate printable itinerary PDFs

**SDK/Library**: `puppeteer` v21.0+

**Installation**:
```bash
npm install puppeteer
```

**Cost**:
- Self-hosted: Server compute costs only
- Cloud function: ~$0.0001 per generation (AWS Lambda)

**Implementation**:
```javascript
const puppeteer = require('puppeteer');

async function generateItineraryPDF(htmlContent) {
  const browser = await puppeteer.launch({
    headless: 'new',
    args: ['--no-sandbox']
  });
  
  const page = await browser.newPage();
  await page.setContent(htmlContent, { waitUntil: 'networkidle0' });
  
  const pdf = await page.pdf({
    format: 'A4',
    printBackground: true,
    margin: { top: '20mm', right: '20mm', bottom: '20mm', left: '20mm' }
  });
  
  await browser.close();
  return pdf;
}
```

**Performance Optimization**:
- Use headless Chrome in production
- Implement queue system (Bull, BullMQ) for PDF generation
- Cache generated PDFs in S3
- Consider serverless (AWS Lambda) for auto-scaling

**Alternative**: `jsPDF` (client-side, limited formatting)

---

## Cost Summary (Monthly Estimates)

| Service | Usage | Estimated Cost |
|---------|-------|----------------|
| @mapbox | 100k map loads | $50 |
| @twilio | 10k SMS | $100 |
| @aws_s3 | 10GB + 1M requests | $20 |
| @openexchangerates | Currency API | $12 |
| @stripe | 100 transactions × $50 avg | $145 (2.9%) |
| @firebase_auth | 15k phone auths | $50 |
| Compute (AWS EC2 t3.medium) | 730 hours | $30 |
| Database (RDS PostgreSQL) | db.t3.micro | $15 |
| CloudFront CDN | 100GB transfer | $8 |
| **TOTAL** | | **~$430/month** |

**At scale** (100k users, 10k itineraries/day):
- @mapbox: $2,000/month
- @twilio: $500/month
- @aws_s3: $200/month
- Compute: $500/month (auto-scaling)
- **TOTAL**: **~$3,500/month**

---

## Rate Limiting Strategy

### Application-Level Rate Limiting

```javascript
const { Ratelimit } = require('@upstash/ratelimit');
const { Redis } = require('@upstash/redis');

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_REST_URL,
  token: process.env.UPSTASH_REDIS_REST_TOKEN
});

const ratelimit = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(10, '1 h'), // 10 requests per hour
  analytics: true
});

app.post('/api/v1/itineraries', async (req, res) => {
  const identifier = req.user.id;
  const { success, limit, reset, remaining } = await ratelimit.limit(identifier);
  
  if (!success) {
    return res.status(429).json({
      error: 'Rate limit exceeded',
      retry_after: Math.ceil((reset - Date.now()) / 1000)
    });
  }
  
  // Process request
});
```

---

## Monitoring & Alerting

### Cost Alerts (AWS CloudWatch)

```javascript
// Set up billing alarm
const cloudwatch = new AWS.CloudWatch();

await cloudwatch.putMetricAlarm({
  AlarmName: 'HighMonthlyBill',
  ComparisonOperator: 'GreaterThanThreshold',
  EvaluationPeriods: 1,
  MetricName: 'EstimatedCharges',
  Namespace: 'AWS/Billing',
  Period: 86400, // 1 day
  Threshold: 500, // $500
  Statistic: 'Maximum',
  AlarmActions: [process.env.SNS_TOPIC_ARN]
}).promise();
```

### API Usage Monitoring

```javascript
// Track third-party API calls
const apiMetrics = {
  mapbox: 0,
  twilio: 0,
  stripe: 0
};

function trackAPICall(service) {
  apiMetrics[service]++;
  
  // Log to monitoring service (DataDog, New Relic)
  console.log(`[METRICS] ${service}: ${apiMetrics[service]} calls today`);
}
```

---

## Error Handling

### Graceful Degradation

```javascript
async function getMapData(itineraryId) {
  try {
    return await fetchFromMapbox(itineraryId);
  } catch (error) {
    if (error.status === 429) {
      // Rate limit exceeded - use cached data
      return getCachedMapData(itineraryId);
    }
    
    if (error.status >= 500) {
      // Mapbox service error - use static map
      return generateStaticMap(itineraryId);
    }
    
    throw error;
  }
}
```

### Retry Logic

```javascript
async function withRetry(fn, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 2 ** i * 1000));
    }
  }
}

// Usage
const rates = await withRetry(() => getCurrencyRates());
```

---

## Development vs Production

### Environment Configuration

```javascript
// .env.development
MAPBOX_ACCESS_TOKEN=pk.test...
STRIPE_SECRET_KEY=sk_test...
AWS_S3_BUCKET=travelplanner-dev

// .env.production
MAPBOX_ACCESS_TOKEN=pk.prod...
STRIPE_SECRET_KEY=sk_live...
AWS_S3_BUCKET=travelplanner-prod
```

### Testing with Mocks

```javascript
// __mocks__/twilio.js
module.exports = {
  messages: {
    create: jest.fn().mockResolvedValue({ sid: 'SM123' })
  }
};

// In tests
jest.mock('twilio');
const twilio = require('twilio');

test('sends OTP', async () => {
  await sendOTP('+14155552671', '123456');
  expect(twilio.messages.create).toHaveBeenCalled();
});
```

---

## Performance Optimization

### Caching Strategy

```javascript
const NodeCache = require('node-cache');

const cache = {
  maps: new NodeCache({ stdTTL: 3600 }), // 1 hour
  hotels: new NodeCache({ stdTTL: 1800 }), // 30 minutes
  currency: new NodeCache({ stdTTL: 3600 }) // 1 hour
};

async function getCachedOrFetch(cacheKey, fetchFn, cacheStore) {
  const cached = cacheStore.get(cacheKey);
  if (cached) return cached;
  
  const data = await fetchFn();
  cacheStore.set(cacheKey, data);
  return data;
}
```

### Batch Processing

```javascript
// Batch geocoding requests
async function geocodeBatch(addresses) {
  const batches = chunk(addresses, 50); // Mapbox allows 50 per request
  const results = [];
  
  for (const batch of batches) {
    const response = await mapboxClient.geocoding
      .forwardGeocode({ query: batch })
      .send();
    results.push(...response.body.features);
  }
  
  return results;
}
```

---

## Recommended Development Tools

- **API Testing**: Postman, Insomnia
- **Monitoring**: DataDog, New Relic, Sentry
- **Log Management**: Loggly, Papertrail, CloudWatch Logs
- **CI/CD**: GitHub Actions, CircleCI, Jenkins
- **Infrastructure**: Terraform, AWS CDK
- **Secret Management**: AWS Secrets Manager, HashiCorp Vault

---

**Last Updated**: October 4, 2025  
**Maintainer**: Engineering Team
