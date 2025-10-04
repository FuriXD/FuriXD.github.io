# Cloud Storage Model: Itinerary Data Management

## Storage Architecture

### AWS S3 Implementation (Recommended)

#### Bucket Structure

```
s3://travelplanner-prod/
├── itineraries/
│   ├── {user_id}/
│   │   ├── {itinerary_id}/
│   │   │   ├── itinerary.json          # Core itinerary data
│   │   │   ├── itinerary.pdf           # Generated PDF
│   │   │   ├── map.geojson            # Map data
│   │   │   ├── thumbnail.jpg          # Preview image
│   │   │   └── metadata.json          # Additional metadata
├── uploads/
│   ├── {user_id}/
│   │   ├── avatar.jpg
│   │   └── documents/
├── backups/
│   ├── {YYYY-MM-DD}/
│   │   ├── database-backup.sql.gz
│   │   └── manifest.json
└── temp/
    └── {session_id}/
        └── preview.pdf                 # Temporary previews (auto-delete)
```

#### Bucket Configuration

```json
{
  "BucketName": "travelplanner-prod",
  "Region": "us-east-1",
  "Versioning": {
    "Status": "Enabled",
    "MfaDelete": "Enabled"
  },
  "Encryption": {
    "Rules": [
      {
        "ApplyServerSideEncryptionByDefault": {
          "SSEAlgorithm": "AES256"
        }
      }
    ]
  },
  "PublicAccessBlock": {
    "BlockPublicAcls": true,
    "IgnorePublicAcls": true,
    "BlockPublicPolicy": true,
    "RestrictPublicBuckets": true
  },
  "LifecycleConfiguration": {
    "Rules": [
      {
        "Id": "DeleteTempFiles",
        "Prefix": "temp/",
        "Status": "Enabled",
        "Expiration": {
          "Days": 1
        }
      },
      {
        "Id": "ArchiveOldItineraries",
        "Prefix": "itineraries/",
        "Status": "Enabled",
        "Transitions": [
          {
            "Days": 90,
            "StorageClass": "INTELLIGENT_TIERING"
          },
          {
            "Days": 365,
            "StorageClass": "GLACIER"
          }
        ]
      },
      {
        "Id": "DeleteOldBackups",
        "Prefix": "backups/",
        "Status": "Enabled",
        "Expiration": {
          "Days": 90
        }
      }
    ]
  },
  "CorsConfiguration": {
    "CorsRules": [
      {
        "AllowedHeaders": ["*"],
        "AllowedMethods": ["GET", "PUT", "POST"],
        "AllowedOrigins": ["https://travelplanner.com"],
        "ExposeHeaders": ["ETag"],
        "MaxAgeSeconds": 3600
      }
    ]
  }
}
```

#### IAM Policy (Least Privilege)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowAppServerAccess",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT_ID:role/TravelPlannerAppRole"
      },
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::travelplanner-prod/itineraries/*",
      "Condition": {
        "StringEquals": {
          "s3:x-amz-server-side-encryption": "AES256"
        }
      }
    },
    {
      "Sid": "AllowBackupAccess",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT_ID:role/BackupRole"
      },
      "Action": [
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::travelplanner-prod/backups/*"
    },
    {
      "Sid": "DenyInsecureTransport",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::travelplanner-prod",
        "arn:aws:s3:::travelplanner-prod/*"
      ],
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    }
  ]
}
```

### Implementation Code

```javascript
const { S3Client, PutObjectCommand, GetObjectCommand, DeleteObjectCommand } = require('@aws-sdk/client-s3');
const { getSignedUrl } = require('@aws-sdk/s3-request-presigner');

const s3Client = new S3Client({
  region: process.env.AWS_REGION,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY
  }
});

const BUCKET_NAME = process.env.S3_BUCKET_NAME;

// Save itinerary to S3
async function saveItinerary(userId, itineraryId, data) {
  const key = `itineraries/${userId}/${itineraryId}/itinerary.json`;
  
  await s3Client.send(new PutObjectCommand({
    Bucket: BUCKET_NAME,
    Key: key,
    Body: JSON.stringify(data),
    ContentType: 'application/json',
    ServerSideEncryption: 'AES256',
    Metadata: {
      userId,
      itineraryId,
      createdAt: new Date().toISOString()
    },
    Tagging: `Environment=production&Type=itinerary&UserId=${userId}`
  }));
  
  return key;
}

// Generate signed URL for PDF download (expires in 1 hour)
async function getDownloadUrl(userId, itineraryId) {
  const key = `itineraries/${userId}/${itineraryId}/itinerary.pdf`;
  
  const command = new GetObjectCommand({
    Bucket: BUCKET_NAME,
    Key: key
  });
  
  return await getSignedUrl(s3Client, command, { expiresIn: 3600 });
}

// Delete itinerary (GDPR right to be forgotten)
async function deleteItinerary(userId, itineraryId) {
  const prefix = `itineraries/${userId}/${itineraryId}/`;
  
  // List all objects with prefix
  const listCommand = new ListObjectsV2Command({
    Bucket: BUCKET_NAME,
    Prefix: prefix
  });
  
  const { Contents } = await s3Client.send(listCommand);
  
  // Delete all objects
  for (const object of Contents || []) {
    await s3Client.send(new DeleteObjectCommand({
      Bucket: BUCKET_NAME,
      Key: object.Key
    }));
  }
}
```

---

## GCP Cloud Storage Implementation (Alternative)

### Bucket Configuration

```javascript
const { Storage } = require('@google-cloud/storage');

const storage = new Storage({
  projectId: process.env.GCP_PROJECT_ID,
  keyFilename: process.env.GCP_KEY_FILE
});

const bucket = storage.bucket('travelplanner-prod');

// Create bucket with encryption
async function createBucket() {
  await storage.createBucket('travelplanner-prod', {
    location: 'US',
    storageClass: 'STANDARD',
    uniformBucketLevelAccess: { enabled: true },
    encryption: {
      defaultKmsKeyName: 'projects/PROJECT_ID/locations/global/keyRings/KEYRING/cryptoKeys/KEY'
    },
    lifecycle: {
      rule: [
        {
          action: { type: 'Delete' },
          condition: { age: 90, matchesPrefix: ['temp/'] }
        },
        {
          action: { type: 'SetStorageClass', storageClass: 'NEARLINE' },
          condition: { age: 90, matchesPrefix: ['itineraries/'] }
        }
      ]
    }
  });
}

// Upload file
async function uploadFile(userId, itineraryId, data) {
  const filename = `itineraries/${userId}/${itineraryId}/itinerary.json`;
  const file = bucket.file(filename);
  
  await file.save(JSON.stringify(data), {
    contentType: 'application/json',
    metadata: {
      userId,
      itineraryId,
      createdAt: new Date().toISOString()
    }
  });
  
  return filename;
}

// Generate signed URL
async function getSignedUrl(userId, itineraryId) {
  const filename = `itineraries/${userId}/${itineraryId}/itinerary.pdf`;
  const file = bucket.file(filename);
  
  const [url] = await file.getSignedUrl({
    version: 'v4',
    action: 'read',
    expires: Date.now() + 3600 * 1000 // 1 hour
  });
  
  return url;
}
```

---

## Azure Blob Storage Implementation

```javascript
const { BlobServiceClient } = require('@azure/storage-blob');

const blobServiceClient = BlobServiceClient.fromConnectionString(
  process.env.AZURE_STORAGE_CONNECTION_STRING
);

const containerName = 'travelplanner-prod';

// Upload blob
async function uploadBlob(userId, itineraryId, data) {
  const containerClient = blobServiceClient.getContainerClient(containerName);
  const blobName = `itineraries/${userId}/${itineraryId}/itinerary.json`;
  const blockBlobClient = containerClient.getBlockBlobClient(blobName);
  
  await blockBlobClient.upload(JSON.stringify(data), Buffer.byteLength(JSON.stringify(data)), {
    blobHTTPHeaders: { blobContentType: 'application/json' },
    metadata: {
      userId,
      itineraryId,
      createdAt: new Date().toISOString()
    }
  });
  
  return blobName;
}

// Generate SAS URL
async function getSASUrl(userId, itineraryId) {
  const containerClient = blobServiceClient.getContainerClient(containerName);
  const blobName = `itineraries/${userId}/${itineraryId}/itinerary.pdf`;
  const blobClient = containerClient.getBlobClient(blobName);
  
  const expiresOn = new Date();
  expiresOn.setHours(expiresOn.getHours() + 1);
  
  const sasUrl = await blobClient.generateSasUrl({
    permissions: 'r',
    expiresOn
  });
  
  return sasUrl;
}
```

---

## Retention Policy

### Data Types & Retention Periods

| Data Type | Retention Period | Storage Class | Notes |
|-----------|------------------|---------------|-------|
| Active Itineraries | Indefinite | Standard | User can delete anytime |
| Inactive Itineraries (>90 days) | Indefinite | Intelligent Tiering | Auto-transition to cheaper storage |
| Old Itineraries (>1 year) | Indefinite | Glacier | Deep archive |
| Temporary Files | 24 hours | Standard | Auto-delete |
| PDF Previews | 1 hour | Standard | Session-based |
| Database Backups | 90 days | Standard | Daily backups |
| User Uploads | Until account deletion | Standard | Profile pictures, etc. |
| Deleted Data (soft delete) | 30 days | Standard | Allow recovery |

### Legal/GDPR Considerations

```javascript
// Soft delete (30-day grace period)
async function softDeleteItinerary(userId, itineraryId) {
  const key = `itineraries/${userId}/${itineraryId}/itinerary.json`;
  
  // Add deletion metadata
  await s3Client.send(new CopyObjectCommand({
    Bucket: BUCKET_NAME,
    CopySource: `${BUCKET_NAME}/${key}`,
    Key: key,
    Metadata: {
      deletedAt: new Date().toISOString(),
      deletedBy: userId
    },
    MetadataDirective: 'REPLACE'
  }));
  
  // Schedule permanent deletion after 30 days
  await scheduleCleanup(key, 30);
}

// Hard delete (GDPR right to be forgotten)
async function hardDeleteUserData(userId) {
  // Delete all user itineraries
  const prefix = `itineraries/${userId}/`;
  await deleteAllWithPrefix(prefix);
  
  // Delete uploads
  const uploadsPrefix = `uploads/${userId}/`;
  await deleteAllWithPrefix(uploadsPrefix);
  
  // Log deletion for audit
  await auditLog.log({
    action: 'HARD_DELETE',
    userId,
    timestamp: new Date().toISOString()
  });
}
```

---

## Access Control

### Role-Based Access

```javascript
// Check if user can access itinerary
async function canAccessItinerary(userId, itineraryId) {
  // Get itinerary metadata from database
  const itinerary = await db.itineraries.findOne({ id: itineraryId });
  
  if (!itinerary) return false;
  
  // Check ownership
  if (itinerary.userId === userId) return true;
  
  // Check if shared with user
  if (itinerary.sharedWith?.includes(userId)) return true;
  
  // Check if public
  if (itinerary.isPublic) return true;
  
  return false;
}

// Generate shareable link (public access with token)
async function createShareableLink(userId, itineraryId) {
  const token = crypto.randomBytes(32).toString('hex');
  
  // Store token in database
  await db.shareTokens.insert({
    token,
    itineraryId,
    createdBy: userId,
    createdAt: new Date(),
    expiresAt: new Date(Date.now() + 30 * 24 * 60 * 60 * 1000) // 30 days
  });
  
  return `https://travelplanner.com/itinerary/shared/${token}`;
}
```

---

## Cleanup Rules

### Automated Cleanup Jobs

```javascript
// Run daily cleanup job (AWS Lambda / Cloud Function)
async function dailyCleanup() {
  const now = new Date();
  
  // 1. Delete temp files older than 24 hours
  await deleteOlderThan('temp/', 1);
  
  // 2. Delete soft-deleted items after 30 days
  await permanentlyDeleteSoftDeleted(30);
  
  // 3. Archive old itineraries to Glacier (>1 year)
  await archiveOldItineraries(365);
  
  // 4. Delete expired share tokens
  await db.shareTokens.deleteMany({
    expiresAt: { $lt: now }
  });
  
  // 5. Delete orphaned files (no database record)
  await cleanupOrphanedFiles();
}

async function deleteOlderThan(prefix, days) {
  const cutoffDate = new Date();
  cutoffDate.setDate(cutoffDate.getDate() - days);
  
  const listCommand = new ListObjectsV2Command({
    Bucket: BUCKET_NAME,
    Prefix: prefix
  });
  
  const { Contents } = await s3Client.send(listCommand);
  
  for (const object of Contents || []) {
    if (object.LastModified < cutoffDate) {
      await s3Client.send(new DeleteObjectCommand({
        Bucket: BUCKET_NAME,
        Key: object.Key
      }));
    }
  }
}
```

### Scheduled Jobs (Cron)

```javascript
// Schedule with node-cron or AWS EventBridge
const cron = require('node-cron');

// Daily at 2 AM UTC
cron.schedule('0 2 * * *', async () => {
  console.log('Running daily cleanup...');
  await dailyCleanup();
});

// Weekly backup (Sunday 3 AM)
cron.schedule('0 3 * * 0', async () => {
  console.log('Running weekly backup...');
  await backupDatabase();
});
```

---

## Backup Strategy

### Database Backups

```javascript
const { exec } = require('child_process');
const { promisify } = require('util');
const execAsync = promisify(exec);

async function backupDatabase() {
  const timestamp = new Date().toISOString().split('T')[0]; // YYYY-MM-DD
  const filename = `database-backup-${timestamp}.sql.gz`;
  
  // Dump database
  await execAsync(`pg_dump ${process.env.DATABASE_URL} | gzip > /tmp/${filename}`);
  
  // Upload to S3
  const fileStream = fs.createReadStream(`/tmp/${filename}`);
  
  await s3Client.send(new PutObjectCommand({
    Bucket: BUCKET_NAME,
    Key: `backups/${timestamp}/${filename}`,
    Body: fileStream,
    ServerSideEncryption: 'AES256',
    StorageClass: 'STANDARD_IA'
  }));
  
  // Clean up temp file
  fs.unlinkSync(`/tmp/${filename}`);
  
  console.log(`Backup completed: ${filename}`);
}
```

### Disaster Recovery

```javascript
async function restoreFromBackup(backupDate) {
  const filename = `database-backup-${backupDate}.sql.gz`;
  const key = `backups/${backupDate}/${filename}`;
  
  // Download from S3
  const command = new GetObjectCommand({
    Bucket: BUCKET_NAME,
    Key: key
  });
  
  const { Body } = await s3Client.send(command);
  const writeStream = fs.createWriteStream(`/tmp/${filename}`);
  Body.pipe(writeStream);
  
  await new Promise((resolve, reject) => {
    writeStream.on('finish', resolve);
    writeStream.on('error', reject);
  });
  
  // Restore database
  await execAsync(`gunzip < /tmp/${filename} | psql ${process.env.DATABASE_URL}`);
  
  console.log(`Database restored from ${backupDate}`);
}
```

---

## Cost Optimization

### Storage Costs (AWS S3)

```javascript
// Estimate monthly storage costs
function estimateStorageCosts(users, avgItinerariesPerUser, avgSizePerItinerary) {
  const totalItineraries = users * avgItinerariesPerUser;
  const totalStorageGB = (totalItineraries * avgSizePerItinerary) / (1024 ** 3);
  
  const costs = {
    standard: totalStorageGB * 0.023, // $0.023 per GB/month
    intelligentTiering: totalStorageGB * 0.0125, // After 90 days
    glacier: totalStorageGB * 0.004, // After 1 year
    requests: {
      put: (totalItineraries * 0.005) / 1000, // $0.005 per 1k PUT
      get: (totalItineraries * 10 * 0.0004) / 1000 // $0.0004 per 1k GET
    }
  };
  
  return costs;
}

// Example: 10,000 users, 5 itineraries each, 2MB per itinerary
const costs = estimateStorageCosts(10000, 5, 2 * 1024 * 1024);
console.log('Estimated monthly costs:', costs);
// Output: ~$2-5/month for storage, $1-2 for requests
```

### CloudFront CDN Integration

```javascript
// Reduce data transfer costs with CloudFront
const cloudfrontUrl = `https://d123456789.cloudfront.net/itineraries/${userId}/${itineraryId}/itinerary.pdf`;

// Savings: $0.085/GB (S3 transfer) vs $0.085-0.025/GB (CloudFront, tiered)
```

---

## Monitoring & Alerts

```javascript
// AWS CloudWatch metrics
const cloudwatch = new AWS.CloudWatch();

async function trackStorageMetrics() {
  await cloudwatch.putMetricData({
    Namespace: 'TravelPlanner/Storage',
    MetricData: [
      {
        MetricName: 'TotalStorageGB',
        Value: await getTotalStorageGB(),
        Unit: 'Gigabytes',
        Timestamp: new Date()
      },
      {
        MetricName: 'DailyUploads',
        Value: await getDailyUploadsCount(),
        Unit: 'Count',
        Timestamp: new Date()
      }
    ]
  }).promise();
}

// Alert on high costs
await cloudwatch.putMetricAlarm({
  AlarmName: 'HighS3Costs',
  MetricName: 'EstimatedCharges',
  Namespace: 'AWS/Billing',
  Statistic: 'Maximum',
  Period: 86400,
  EvaluationPeriods: 1,
  Threshold: 100, // $100
  ComparisonOperator: 'GreaterThanThreshold',
  AlarmActions: [process.env.SNS_TOPIC_ARN]
}).promise();
```

---

**Summary Checklist**:
- [x] S3/GCS/Azure bucket configured with encryption
- [x] Lifecycle policies for cost optimization
- [x] IAM/Access policies with least privilege
- [x] Signed URLs for secure sharing (1-hour expiration)
- [x] Daily automated cleanup jobs
- [x] 90-day backup retention
- [x] Soft delete (30-day grace period)
- [x] GDPR-compliant hard delete
- [x] Monitoring and cost alerts
- [x] Disaster recovery procedures

**Last Updated**: October 4, 2025
