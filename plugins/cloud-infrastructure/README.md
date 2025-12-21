# Cloud Infrastructure Plugin

Terraform patterns for serverless and static site infrastructure on AWS.

## Focus Areas

This plugin provides opinionated Terraform patterns for:
- **Route53**: DNS management and domain configuration
- **S3**: Static asset hosting with versioning and encryption
- **CloudFront**: CDN distribution with custom domains
- **ACM**: SSL/TLS certificate management
- **Lambda**: Serverless functions and edge computing

Perfect for JAMstack applications, static sites, and serverless web applications.

## Components

### Skills

- **terraform-aws**: Focused best practices for Route53, S3, CloudFront, ACM, and Lambda with Terraform

### Agents

- **terraform-validator**: Validates Terraform configurations for syntax, best practices, and security issues

## Common Use Cases

1. **Static Website with Custom Domain**
   - S3 bucket for hosting
   - CloudFront distribution
   - Route53 DNS configuration
   - ACM certificate for HTTPS

2. **Serverless Application**
   - Lambda functions
   - CloudFront for API Gateway caching
   - S3 for static assets
   - Route53 for custom domains

3. **Edge Functions**
   - Lambda@Edge for request/response manipulation
   - CloudFront distributions
   - ACM certificates

## Usage

The skill automatically activates when you:
- Create terraform configuration for supported services
- Ask about Route53, S3, CloudFront, ACM, or Lambda
- Request infrastructure validation

## Requirements

- Terraform >= 1.5.0
- AWS CLI configured
- AWS credentials with appropriate permissions

## Version

0.2.0 - Simplified to focus on serverless and static site infrastructure
