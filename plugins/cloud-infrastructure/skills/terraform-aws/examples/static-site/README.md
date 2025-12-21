# Static Site with CloudFront and Custom Domain

Complete example deploying a static website with:
- **S3** for static file storage
- **CloudFront** CDN with HTTPS
- **ACM** SSL/TLS certificate
- **Route53** DNS configuration
- **Lambda@Edge** for security headers (optional)

## Architecture

```
User → Route53 (DNS) → CloudFront (CDN) → S3 (Static Files)
                            ↓
                      ACM Certificate
                            ↓
                      Lambda@Edge (optional)
```

## Prerequisites

- Terraform >= 1.5.0
- AWS CLI configured
- Route53 hosted zone already created
- Domain registered

## Quick Start

1. **Configure variables**:
   ```bash
   cp terraform.tfvars.example terraform.tfvars
   # Edit terraform.tfvars with your domain and project name
   ```

2. **Initialize Terraform**:
   ```bash
   terraform init
   ```

3. **Plan deployment**:
   ```bash
   terraform plan -out=tfplan
   ```

4. **Apply configuration**:
   ```bash
   terraform apply tfplan
   ```

5. **Upload your site**:
   ```bash
   aws s3 sync ./build s3://$(terraform output -raw bucket_name)
   ```

6. **Invalidate CloudFront cache**:
   ```bash
   aws cloudfront create-invalidation \
     --distribution-id $(terraform output -raw distribution_id) \
     --paths "/*"
   ```

## Files

- `main.tf` - Main infrastructure resources
- `variables.tf` - Input variable declarations
- `locals.tf` - Environment-specific configuration
- `outputs.tf` - Output values
- `versions.tf` - Terraform and provider versions
- `backend.tf` - Remote state configuration
- `providers.tf` - AWS provider configuration

## Costs

Estimated monthly costs:
- **S3**: ~$0.023 per GB stored + $0.09 per GB transferred
- **CloudFront**: $0.085 per GB (first 10 TB) + $0.0075 per 10,000 requests
- **Route53**: $0.50 per hosted zone + $0.40 per million queries
- **ACM**: Free
- **Lambda@Edge**: $0.60 per 1M requests (if used)

Typical small site: **$1-5/month**

## Deployment Process

### First-Time Deployment

1. Terraform creates infrastructure (~10-15 minutes for CloudFront)
2. Upload site files to S3
3. CloudFront serves content globally

### Updates

1. Build your static site locally
2. Sync to S3: `aws s3 sync ./build s3://BUCKET_NAME`
3. Invalidate CloudFront cache (only changed paths if needed)

## Customization

### Enable Lambda@Edge

Uncomment the Lambda@Edge sections in `main.tf` to add:
- Security headers (CSP, X-Frame-Options, etc.)
- Request/response manipulation
- A/B testing
- URL rewrites

### Multi-Environment

The `locals.tf` file is configured for dev/staging/prod environments:

```hcl
locals {
  environments = {
    dev = { domain_name = "dev.example.com", ... }
    prod = { domain_name = "www.example.com", ... }
  }
}
```

Deploy each environment separately:
```bash
terraform workspace new prod
terraform apply -var="environment=prod"
```

### SPA Support

The CloudFront distribution includes custom error responses for Single Page Applications (React, Vue, etc.):

```hcl
custom_error_response {
  error_code         = 404
  response_code      = 200
  response_page_path = "/index.html"
}
```

## Security Features

- ✅ S3 bucket not publicly accessible (CloudFront OAI only)
- ✅ HTTPS enforced (HTTP redirects to HTTPS)
- ✅ TLS 1.2+ only
- ✅ S3 server-side encryption
- ✅ CloudFront compression enabled
- ✅ DNS validated SSL certificate

## Troubleshooting

### Certificate validation stuck

**Problem**: ACM certificate shows "Pending validation"

**Solution**:
- Check Route53 records were created
- Wait up to 30 minutes for DNS propagation
- Verify hosted zone is correct

### CloudFront returns 403

**Problem**: CloudFront returns access denied

**Solution**:
- Verify S3 bucket policy allows CloudFront OAI
- Check that objects exist in S3
- Ensure default root object is set (index.html)

### Site shows old content

**Problem**: Updates not visible on site

**Solution**:
- Invalidate CloudFront cache: `aws cloudfront create-invalidation ...`
- Or wait for TTL to expire (default: 1 hour)
- For instant updates, set lower TTL in CloudFront cache behavior

## Outputs

After deployment, Terraform outputs:

- `website_url` - Your site URL (https://yourdomain.com)
- `bucket_name` - S3 bucket for file uploads
- `distribution_id` - CloudFront distribution ID for cache invalidation
- `nameservers` - Route53 nameservers (if zone was created)

## Cleanup

To destroy all resources:

```bash
# Empty S3 bucket first
aws s3 rm s3://$(terraform output -raw bucket_name) --recursive

# Destroy infrastructure
terraform destroy
```

**Note**: CloudFront distributions take 15-30 minutes to fully delete.

## Next Steps

- Set up CI/CD to auto-deploy on git push
- Add CloudWatch alarms for error rates
- Implement Lambda@Edge for advanced routing
- Add WAF rules for DDoS protection
- Enable CloudFront access logs for analytics
