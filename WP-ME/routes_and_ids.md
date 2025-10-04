# Routes, CSS IDs & Service References

## Frontend Routes

### Public Routes
- `/` - Landing page with hero, features, sample itinerary
- `/login` - Authentication modal/page
- `/signup` - User registration
- `/about` - About the platform
- `/features` - Feature showcase
- `/pricing` - Pricing plans
- `/destinations` - Browse destinations
- `/blog` - Travel tips and guides
- `/contact` - Contact form
- `/privacy` - Privacy policy
- `/terms` - Terms of service
- `/security` - Security practices

### Protected Routes (Require Authentication)
- `/dashboard` - User dashboard with trip list
- `/itinerary/new` - Create new itinerary wizard
- `/itinerary/:id` - View specific itinerary
- `/itinerary/:id/edit` - Edit itinerary
- `/itinerary/:id/download` - Download PDF
- `/itinerary/:id/share` - Share itinerary (generate public link)
- `/profile` - User profile settings
- `/profile/preferences` - Travel preferences
- `/bookings` - Booking history
- `/favorites` - Saved/favorited itineraries

### Partner/Sponsored Routes
- `/sponsored/partners` - List of partner companies
- `/sponsored/:partner_id` - Partner detail page
- `/offers` - Special offers from partners

### Admin Routes (Role-Based)
- `/admin` - Admin dashboard
- `/admin/users` - User management
- `/admin/itineraries` - Itinerary moderation
- `/admin/partners` - Partner management
- `/admin/analytics` - Platform analytics

---

## API Routes

### Authentication Endpoints
- `POST /api/v1/auth/google` - Google OAuth authentication
- `POST /api/v1/auth/phone/send` - Send OTP to phone
- `POST /api/v1/auth/phone/verify` - Verify phone OTP
- `POST /api/v1/auth/login` - Email/password login
- `POST /api/v1/auth/register` - New user registration
- `POST /api/v1/auth/refresh` - Refresh access token
- `POST /api/v1/auth/logout` - Logout and invalidate tokens
- `POST /api/v1/auth/reset-password` - Request password reset
- `POST /api/v1/auth/verify-email` - Verify email address

### Itinerary Endpoints
- `GET /api/v1/itineraries` - List user's itineraries (paginated)
- `POST /api/v1/itineraries` - Create new itinerary
- `GET /api/v1/itineraries/:id` - Get itinerary details
- `PUT /api/v1/itineraries/:id` - Update itinerary
- `DELETE /api/v1/itineraries/:id` - Delete itinerary
- `POST /api/v1/itineraries/:id/duplicate` - Duplicate itinerary
- `GET /api/v1/itineraries/:id/download` - Download PDF
- `POST /api/v1/itineraries/:id/share` - Generate shareable link
- `GET /api/v1/itineraries/shared/:token` - Access shared itinerary (public)

### Map Endpoints
- `GET /api/v1/map/geojson?itinerary_id=:id` - Get GeoJSON for itinerary
- `GET /api/v1/map/kml?itinerary_id=:id` - Get KML for itinerary
- `POST /api/v1/map/geocode` - Geocode address to coordinates
- `POST /api/v1/map/route` - Calculate route between points

### User Endpoints
- `GET /api/v1/users/me` - Get current user profile
- `PUT /api/v1/users/me` - Update user profile
- `DELETE /api/v1/users/me` - Delete account (GDPR compliance)
- `GET /api/v1/users/me/preferences` - Get user preferences
- `PUT /api/v1/users/me/preferences` - Update preferences
- `POST /api/v1/users/me/avatar` - Upload avatar image

### Partner/Sponsored Endpoints
- `GET /api/v1/sponsored/partners` - List sponsored partners
- `GET /api/v1/sponsored/partners/:id` - Get partner details
- `GET /api/v1/sponsored/offers` - Get current offers
- `POST /api/v1/analytics/impression` - Track sponsored impression
- `POST /api/v1/analytics/click` - Track sponsored click
- `POST /api/v1/analytics/conversion` - Track booking conversion

### Search & Discovery
- `GET /api/v1/destinations/search?q=:query` - Search destinations
- `GET /api/v1/destinations/:id` - Get destination details
- `GET /api/v1/hotels/search` - Search hotels
- `GET /api/v1/restaurants/search` - Search restaurants
- `GET /api/v1/activities/search` - Search activities

### Admin Endpoints
- `GET /api/v1/admin/stats` - Platform statistics
- `GET /api/v1/admin/users` - List all users (paginated)
- `PUT /api/v1/admin/users/:id` - Update user (ban, role change)
- `GET /api/v1/admin/itineraries` - List all itineraries
- `DELETE /api/v1/admin/itineraries/:id` - Remove itinerary

---

## CSS IDs & Classes

### Layout Elements
- `#header` - Main navigation header
- `#hero` - Hero section on landing page
- `#footer` - Page footer
- `#main-content` - Main content wrapper
- `#sidebar` - Sidebar navigation (dashboard)
- `.container` - Centered content container (max-width: 1200px)
- `.section` - Generic page section

### Navigation
- `.nav-menu` - Navigation menu list
- `.nav-item` - Individual navigation item
- `.logo` - Company logo
- `.mobile-menu-toggle` - Hamburger menu button
- `.user-dropdown` - User profile dropdown menu

### Buttons & CTAs
- `.btn-primary` - Primary action button (gradient purple)
- `.btn-secondary` - Secondary button (white)
- `.btn-tertiary` - Tertiary/text button
- `.btn-danger` - Delete/destructive action
- `.btn-success` - Success/confirm action
- `.btn-ghost` - Transparent outline button

### Itinerary Components
- `#itinerary-card` - Main itinerary card container
- `.itinerary-header` - Itinerary title section
- `.itinerary-meta` - Metadata (dates, budget, etc.)
- `.itinerary-body` - Main content area
- `.day-item` - Individual day container
- `.day-title` - Day heading
- `.day-activities` - Activities list for a day
- `.activity-item` - Single activity card
- `.activity-time` - Time badge (morning/afternoon/evening)
- `.activity-name` - Activity title
- `.activity-location` - Location/address

### Map Components
- `#map-container` - Map canvas container
- `.map-marker` - Custom map marker
- `.map-popup` - Info popup on map
- `.map-controls` - Zoom/navigation controls

### Sponsored Content
- `.sponsored-banner` - Sponsored content banner
- `.sponsored-badge` - "SPONSORED" label
- `.sponsored-content` - Sponsored item container
- `.sponsored-disclosure` - Legal disclosure text
- `.tier-badge` - Budget tier indicator (budget/medium/premium)

### Forms
- `.form-group` - Form field wrapper
- `.form-label` - Form label
- `.form-input` - Text input field
- `.form-select` - Dropdown select
- `.form-checkbox` - Checkbox input
- `.form-error` - Error message
- `.form-hint` - Helper text

### Cards & Lists
- `.card` - Generic card component
- `.card-header` - Card header
- `.card-body` - Card content
- `.card-footer` - Card footer
- `.list-group` - Vertical list container
- `.list-item` - List item

### Budget Components
- `.budget-summary` - Budget overview section
- `.budget-tier` - Budget tier grid
- `.tier-card` - Individual tier card
- `.tier-name` - Tier label (Budget/Medium/Premium)
- `.tier-price` - Price display
- `.cost-breakdown` - Itemized cost list
- `.cost-total` - Total cost display

### Modal/Dialog
- `.modal` - Modal overlay
- `.modal-content` - Modal content container
- `.modal-header` - Modal title section
- `.modal-body` - Modal main content
- `.modal-footer` - Modal actions

### Recommendations
- `.recommendation-grid` - Grid layout for hotels/restaurants
- `.recommendation-card` - Individual recommendation card
- `.feature-grid` - Feature showcase grid
- `.feature-card` - Feature card
- `.feature-icon` - Feature icon container

### Status & Badges
- `.badge` - Generic badge
- `.badge-success` - Green success badge
- `.badge-warning` - Yellow warning badge
- `.badge-danger` - Red danger badge
- `.badge-info` - Blue info badge
- `.status-active` - Active status indicator
- `.status-pending` - Pending status indicator

### Utility Classes
- `.hidden` - Display: none
- `.sr-only` - Screen reader only (accessibility)
- `.text-center` - Centered text
- `.text-right` - Right-aligned text
- `.text-muted` - Gray/muted text
- `.mt-1` to `.mt-5` - Margin top utilities
- `.mb-1` to `.mb-5` - Margin bottom utilities
- `.p-1` to `.p-5` - Padding utilities

### Emergency Contacts
- `.emergency-section` - Emergency info container
- `.emergency-banner` - Red alert banner
- `.contact-grid` - Emergency contacts grid
- `.contact-item` - Individual contact card

### Loading States
- `.loading` - Loading spinner
- `.skeleton` - Skeleton loading placeholder
- `.shimmer` - Shimmer loading effect

---

## Service Mentions (@ notation)

### Authentication & Identity
- `@google` - Google OAuth (authentication)
- `@firebase_auth` - Firebase Authentication (managed auth service)
- `@auth0` - Auth0 (enterprise auth alternative)
- `@twilio` - Twilio SMS (phone verification)

### Maps & Location
- `@mapbox` - Mapbox GL JS (interactive maps)
- `@openstreetmap` - OpenStreetMap (free map tiles)
- `@google_maps` - Google Maps API (alternative)

### Payments
- `@stripe` - Stripe (payment processing)
- `@paypal` - PayPal (payment alternative)
- `@square` - Square (payment alternative)

### Cloud Storage
- `@aws_s3` - Amazon S3 (file storage)
- `@gcp_storage` - Google Cloud Storage
- `@azure_blob` - Azure Blob Storage

### Communication
- `@twilio` - Twilio (SMS, voice)
- `@sendgrid` - SendGrid (email delivery)
- `@mailgun` - Mailgun (email alternative)
- `@postmark` - Postmark (transactional email)

### Analytics & Monitoring
- `@google_analytics` - Google Analytics (web analytics)
- `@mixpanel` - Mixpanel (product analytics)
- `@amplitude` - Amplitude (analytics alternative)
- `@datadog` - DataDog (infrastructure monitoring)
- `@sentry` - Sentry (error tracking)
- `@newrelic` - New Relic (APM)

### Data & APIs
- `@openexchangerates` - Currency exchange rates
- `@xe` - XE Currency API (alternative)
- `@booking` - Booking.com API (hotel bookings)
- `@amadeus` - Amadeus Travel API (flights, hotels)
- `@skyscanner` - Skyscanner API (flights)

### PDF Generation
- `@puppeteer` - Puppeteer (headless Chrome for PDF)
- `@wkhtmltopdf` - wkhtmltopdf (HTML to PDF)
- `@jspdf` - jsPDF (client-side PDF)

### Security
- `@cloudflare` - Cloudflare (CDN, DDoS protection)
- `@recaptcha` - Google reCAPTCHA (bot protection)
- `@snyk` - Snyk (security scanning)
- `@vault` - HashiCorp Vault (secrets management)

### Infrastructure
- `@aws` - Amazon Web Services
- `@gcp` - Google Cloud Platform
- `@azure` - Microsoft Azure
- `@vercel` - Vercel (frontend hosting)
- `@netlify` - Netlify (frontend alternative)
- `@railway` - Railway (full-stack hosting)

### Database
- `@postgresql` - PostgreSQL (primary database)
- `@mongodb` - MongoDB (document database alternative)
- `@redis` - Redis (caching, rate limiting)
- `@prisma` - Prisma ORM
- `@typeorm` - TypeORM (ORM alternative)

### Messaging & Queues
- `@rabbitmq` - RabbitMQ (message queue)
- `@kafka` - Apache Kafka (event streaming)
- `@bull` - Bull/BullMQ (job queue)

### Search
- `@algolia` - Algolia (search as a service)
- `@elasticsearch` - Elasticsearch (search engine)
- `@meilisearch` - Meilisearch (search alternative)

### Content Delivery
- `@cloudinary` - Cloudinary (image optimization)
- `@imgix` - Imgix (image processing)

---

## Hashtags for Marketing & Meta Tags

### Content Tags
- `#itinerary` - Itinerary content
- `#budget` - Budget planning content
- `#bookings` - Booking-related content
- `#local-experiences` - Local/authentic experiences
- `#family-travel` - Family travel content
- `#business-travel` - Business travel content
- `#solo-travel` - Solo traveler content
- `#couple-travel` - Romantic/couple travel
- `#group-travel` - Group trip planning

### Destination Tags
- `#japan` - Japan-related content
- `#tokyo` - Tokyo city content
- `#kyoto` - Kyoto city content
- `#usa` - USA travel
- `#india` - India travel
- `#singapore` - Singapore travel
- `#vietnam` - Vietnam travel
- `#indonesia` - Indonesia travel

### Feature Tags
- `#travel-tech` - Technology/innovation
- `#travel-planning` - Planning tools
- `#budget-travel` - Budget-conscious travel
- `#luxury-travel` - Premium experiences
- `#sustainable-travel` - Eco-friendly travel
- `#cultural-experiences` - Cultural immersion
- `#food-travel` - Culinary experiences

### Meta Tags for SEO

```html
<meta name="keywords" content="itinerary, travel planning, budget calculator, trip planner, #itinerary, #travel-tech">
<meta property="og:title" content="TravelPlanner - Create Your Perfect Itinerary">
<meta property="og:description" content="Professional travel planning with day-by-day itineraries, budget breakdowns, and interactive maps. #travel-planning #itinerary">
<meta property="og:image" content="https://cdn.travelplanner.com/og-image.jpg">
<meta property="og:url" content="https://travelplanner.com">
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="TravelPlanner - Your Perfect Trip Awaits">
<meta name="twitter:description" content="Create personalized itineraries with budget planning and local recommendations. #itinerary #travel-tech">
```

---

## CSS Variables Reference

```css
:root {
  /* Colors */
  --primary: #667eea;
  --primary-dark: #5568d3;
  --secondary: #764ba2;
  --accent: #f093fb;
  --success: #4caf50;
  --danger: #ef5350;
  --warning: #ff9800;
  
  /* Grays */
  --gray-50: #fafafa;
  --gray-100: #f5f5f5;
  --gray-200: #eeeeee;
  --gray-300: #e0e0e0;
  --gray-700: #616161;
  --gray-900: #212121;
  
  /* Shadows */
  --shadow-sm: 0 1px 3px rgba(0,0,0,0.12);
  --shadow-md: 0 4px 6px rgba(0,0,0,0.1);
  --shadow-lg: 0 10px 25px rgba(0,0,0,0.15);
  
  /* Spacing */
  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --spacing-lg: 2rem;
  --spacing-xl: 4rem;
  
  /* Border Radius */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-xl: 16px;
  
  /* Transitions */
  --transition-fast: 150ms ease;
  --transition-base: 300ms ease;
  --transition-slow: 500ms ease;
}
```

---

## Example Usage in Components

### React Component with CSS Classes

```jsx
function ItineraryCard({ itinerary }) {
  return (
    <div id="itinerary-card" className="card">
      <div className="card-header itinerary-header">
        <h2>{itinerary.title}</h2>
        <div className="itinerary-meta">
          <span className="meta-item">
            📍 {itinerary.destinations.join(', ')}
          </span>
          <span className="meta-item badge badge-success">
            ${itinerary.budget}
          </span>
        </div>
      </div>
      
      <div className="card-body itinerary-body">
        {itinerary.days.map((day, i) => (
          <div key={i} className="day-item">
            <h3 className="day-title">Day {day.number}</h3>
            <div className="day-activities">
              {day.activities.map((activity, j) => (
                <div key={j} className="activity-item">
                  <span className="activity-time">{activity.time}</span>
                  <h4 className="activity-name">{activity.name}</h4>
                  <p className="activity-location">{activity.location}</p>
                </div>
              ))}
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}
```

### API Call Example

```javascript
// Fetch itinerary
const response = await fetch('/api/v1/itineraries/itn_abc123', {
  headers: {
    'Authorization': `Bearer ${token}`
  }
});

// Track sponsored impression
await fetch('/api/v1/analytics/impression', {
  method: 'POST',
  body: JSON.stringify({
    item_id: 'hotel_001',
    partner_id: 'booking_com',
    itinerary_id: 'itn_abc123'
  })
});
```

---

## Naming Conventions

### General Rules
- Use kebab-case for CSS classes: `.itinerary-card`
- Use camelCase for JavaScript IDs: `itineraryCard`
- Use snake_case for API endpoints: `/api/v1/auth/phone/verify`
- Prefix IDs with entity type: `usr_123`, `itn_abc`, `sess_xyz`

### Component Naming
- Components: `ItineraryCard`, `DaySchedule`, `BudgetSummary`
- Utility functions: `formatCurrency()`, `calculateDistance()`
- API services: `AuthService`, `ItineraryService`

---

**Last Updated**: October 4, 2025  
**Document Version**: 1.0
