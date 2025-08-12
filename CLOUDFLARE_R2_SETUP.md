# Cloudflare R2 Setup Guide for PayloadCMS

This guide will help you configure Cloudflare R2 storage for your PayloadCMS media uploads.

## 1. Create Cloudflare R2 Bucket

1. Log in to your [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. Navigate to **R2 Object Storage** in the sidebar
3. Click **Create bucket**
4. Choose a bucket name (e.g., `my-payload-media`)
5. Select a location (auto is recommended)
6. Click **Create bucket**

## 2. Generate R2 API Tokens

1. In the R2 dashboard, click **Manage R2 API tokens**
2. Click **Create API token**
3. Choose **Custom token** for granular permissions
4. Configure the token:
   - **Token name**: `PayloadCMS Media Upload`
   - **Permissions**:
     - `Object:Read` ✅
     - `Object:Write` ✅
     - `Object:Delete` ✅ (optional, for file deletion)
   - **Bucket resources**: Select your specific bucket or choose "All buckets"
   - **TTL**: Set appropriate expiration (or leave blank for no expiration)
5. Click **Create API token**
6. **Save the Access Key ID and Secret Access Key** - you'll need these for your environment variables

## 3. Configure Environment Variables

Update your `.env` file with your Cloudflare R2 credentials:

```env
# Cloudflare R2 Storage Configuration
S3_REGION=auto
S3_ENDPOINT=https://YOUR-ACCOUNT-ID.r2.cloudflarestorage.com
S3_BUCKET=your-bucket-name
S3_ACCESS_KEY_ID=your-r2-access-key-id
S3_SECRET_ACCESS_KEY=your-r2-secret-access-key
S3_FORCE_PATH_STYLE=true
```

**To find your Account ID:**

1. Go to your Cloudflare dashboard
2. Look at the right sidebar - your Account ID is displayed there
3. Replace `YOUR-ACCOUNT-ID` in the endpoint URL above

## 4. Optional: Configure Custom Domain

For better performance and SEO, you can configure a custom domain for your R2 bucket:

1. In your R2 bucket settings, click **Settings**
2. Scroll to **Custom Domains**
3. Click **Connect Domain**
4. Enter your custom domain (e.g., `media.yourdomain.com`)
5. Follow the DNS configuration instructions
6. Once configured, update your `S3_ENDPOINT` in `.env`:
   ```env
   S3_ENDPOINT=https://media.yourdomain.com
   ```

## 5. Testing the Configuration

1. Start your PayloadCMS development server:

   ```bash
   npm run dev
   ```

2. Navigate to the admin panel (http://localhost:3000/admin)
3. Go to the Media collection
4. Try uploading an image or file
5. Verify the file is uploaded to your R2 bucket

## 6. Production Considerations

### CORS Configuration

If you plan to enable client-side uploads, configure CORS in your R2 bucket:

1. In your R2 bucket settings, go to **Settings**
2. Scroll to **CORS policy**
3. Add a CORS policy like:
   ```json
   [
     {
       "AllowedOrigins": ["https://yourdomain.com"],
       "AllowedMethods": ["GET", "PUT", "POST", "DELETE"],
       "AllowedHeaders": ["*"],
       "ExposeHeaders": [],
       "MaxAgeSeconds": 3000
     }
   ]
   ```

### Security Best Practices

1. **Limit API Token Permissions**: Only grant the minimum required permissions
2. **Use Bucket-Specific Tokens**: Create separate tokens for different environments
3. **Rotate Tokens Regularly**: Set reasonable TTL values and rotate tokens periodically
4. **Environment Separation**: Use different buckets for development, staging, and production

## 7. Cost Optimization

- **Lifecycle Rules**: Configure automatic deletion of old files if needed
- **Storage Classes**: R2 has one storage class with no egress fees
- **Monitor Usage**: Keep track of storage and request metrics in the Cloudflare dashboard

## 8. Troubleshooting

### Common Issues:

1. **403 Forbidden Errors**: Check your API token permissions and bucket access
2. **Endpoint Not Found**: Verify your Account ID in the endpoint URL
3. **CORS Errors**: Configure CORS policy for client-side uploads
4. **File Not Loading**: Check the `S3_FORCE_PATH_STYLE` setting

### Debug Mode:

Add this to your PayloadCMS config for debugging:

```typescript
s3Storage({
  collections: {
    media: true,
  },
  bucket: process.env.S3_BUCKET || '',
  config: {
    endpoint: process.env.S3_ENDPOINT,
    region: process.env.S3_REGION || 'auto',
    credentials: {
      accessKeyId: process.env.S3_ACCESS_KEY_ID || '',
      secretAccessKey: process.env.S3_SECRET_ACCESS_KEY || '',
    },
    forcePathStyle: true,
    logger: console, // Enable debugging
  },
}),
```

## Support

- [Cloudflare R2 Documentation](https://developers.cloudflare.com/r2/)
- [PayloadCMS Storage Adapters Documentation](https://payloadcms.com/docs/upload/storage-adapters)
- [AWS S3 SDK v3 Documentation](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/client/s3/) (R2 is S3-compatible)
