---
title: Integrating Cloudflare R2 with Laravel and Custom Domain
slug: cloudflare-r2-laravel-custom-domain
description: How to set up Cloudflare R2 as Laravel's filesystem using the S3 driver, with a custom domain for public file access.
tags:
  - technical
  - learning
added: 2026-07-21T00:00:00.000Z
---

Cloudflare R2 is an object storage service compatible with the Amazon S3 API. Because of this, Laravel can access R2 using the built-in `s3` driver through Flysystem.

In this example:

* Bucket: `my-app-assets`
* Custom domain: `assets.example.com`
* Domain and bucket names have been anonymized.

## 1. Install the S3 Adapter

Run:

```bash
composer require league/flysystem-aws-s3-v3
```

## 2. Create an R2 API Token

Go to the Cloudflare Dashboard:

```text
R2 Object Storage
→ Manage R2 API Tokens
→ Create API Token
```

Grant `Object Read & Write` permissions, preferably scoped to the specific bucket being used.

Save the following information:

```text
Access Key ID
Secret Access Key
Account ID
Bucket Name
```

Never store these credentials directly in your repository.

## 3. Configure the Laravel Environment

Add the following to your `.env` file:

```env
FILESYSTEM_DISK=r2

R2_ACCESS_KEY_ID=your-access-key
R2_SECRET_ACCESS_KEY=your-secret-key
R2_BUCKET=my-app-assets
R2_ENDPOINT=https://ACCOUNT_ID.r2.cloudflarestorage.com
R2_URL=https://assets.example.com
R2_REGION=auto
```

R2 uses the `auto` region. The value `us-east-1` can also be used by applications that don't support it.

Important notes:

* `R2_ENDPOINT` is used by Laravel to communicate with the R2 API.
* `R2_URL` is used to generate public URLs through the custom domain.

Do not replace the API endpoint with the custom domain.

## 4. Add the R2 Disk

Open `config/filesystems.php` and add:

```php
'disks' => [

    'r2' => [
        'driver' => 's3',
        'key' => env('R2_ACCESS_KEY_ID'),
        'secret' => env('R2_SECRET_ACCESS_KEY'),
        'region' => env('R2_REGION', 'auto'),
        'bucket' => env('R2_BUCKET'),
        'endpoint' => env('R2_ENDPOINT'),
        'url' => env('R2_URL'),
        'use_path_style_endpoint' => false,
        'throw' => true,
    ],

],
```

Then clear the configuration cache:

```bash
php artisan config:clear
```

## 5. Upload a File

Example upload through a controller:

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;

public function upload(Request $request)
{
    $request->validate([
        'file' => ['required', 'file', 'max:10240'],
    ]);

    $path = Storage::disk('r2')->putFile(
        'uploads',
        $request->file('file')
    );

    return response()->json([
        'path' => $path,
        'url' => Storage::disk('r2')->url($path),
    ]);
}
```

Resulting URL:

```text
https://assets.example.com/uploads/example.jpg
```

## 6. Connect a Custom Domain

In the Cloudflare Dashboard:

```text
R2
→ Select Bucket
→ Settings
→ Custom Domains
→ Connect Domain
```

Example domain:

```text
assets.example.com
```

The domain must be on a Cloudflare zone linked to the R2 account. A custom domain is recommended for production because it supports Cloudflare caching, WAF, access control, redirects, and analytics.

## 7. Testing

Upload a simple file through Laravel Tinker:

```bash
php artisan tinker
```

```php
Storage::disk('r2')->put(
    'test.txt',
    'Hello from Laravel and Cloudflare R2'
);
```

Check the URL:

```php
Storage::disk('r2')->url('test.txt');
```

Expected result:

```text
https://assets.example.com/test.txt
```

## Common Issues

### Upload succeeds but URL doesn't use the custom domain

Make sure the following configuration is set:

```env
R2_URL=https://assets.example.com
```

Then run:

```bash
php artisan config:clear
```

### Custom domain is accessible but upload fails

Make sure `R2_ENDPOINT` uses the API endpoint:

```env
R2_ENDPOINT=https://ACCOUNT_ID.r2.cloudflarestorage.com
```

Not:

```env
R2_ENDPOINT=https://assets.example.com
```

### File returns HTTP 404 or 403

Check that:

* The custom domain is active.
* The file actually exists in the bucket.
* The path and filename are correct.
* The bucket is intended for public access.

### Presigned URLs don't work through the custom domain

R2 presigned URLs only work using the S3 API domain:

```text
ACCOUNT_ID.r2.cloudflarestorage.com
```

Presigned URLs cannot use a custom domain directly.

## Conclusion

Integrating Laravel with Cloudflare R2 requires two different URLs:

```text
API endpoint → upload, delete, and storage operations
Custom domain → public file access
```

Keeping these separate is important. Using the custom domain as the API endpoint is one of the most common reasons why R2 connections from Laravel fail.
