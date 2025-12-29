# Part 14: Build & Deployment

**Author: Saiprasad Hegde**

## Table of Contents
1. [Introduction to Deployment](#introduction-to-deployment)
2. [Environment Variables](#environment-variables)
3. [Build Optimization](#build-optimization)
4. [Production Builds](#production-builds)
5. [Deploying Next.js to Vercel](#deploying-nextjs-to-vercel)
6. [Deploying React to Netlify](#deploying-react-to-netlify)
7. [Docker Containerization](#docker-containerization)
8. [CI/CD with GitHub Actions](#cicd-with-github-actions)
9. [Monitoring and Analytics](#monitoring-and-analytics)
10. [Production Checklist](#production-checklist)
11. [SEO Optimization](#seo-optimization)
12. [Security Best Practices](#security-best-practices)
13. [Complete Deployment Example](#complete-deployment-example)
14. [Best Practices](#best-practices)
15. [Common Pitfalls](#common-pitfalls)

---

## Introduction to Deployment

Deployment is the process of making your application available to users on the internet. It's like opening the doors to your College Management System and letting students, instructors, and admins use it.

**Real-World Analogy**: Deployment is like opening a new campus:
- **Build**: Constructing the buildings (compiling your code)
- **Testing**: Inspecting for safety (running tests)
- **Deployment**: Opening to students (making it live)
- **Monitoring**: Campus security cameras (analytics and error tracking)

---

## Environment Variables

Environment variables store configuration that changes between environments (development, staging, production).

### Next.js Environment Variables

Create environment files:

**.env.local** (Development - not committed):
```bash
# Database
DATABASE_URL=postgresql://localhost:5432/college_dev

# API Keys
NEXT_PUBLIC_API_URL=http://localhost:3000/api
STRIPE_SECRET_KEY=sk_test_xxxxxxxxxxxxx
NEXT_PUBLIC_STRIPE_PUBLIC_KEY=pk_test_xxxxxxxxxxxxx

# Authentication
NEXTAUTH_SECRET=your-dev-secret-key
NEXTAUTH_URL=http://localhost:3000

# Third-party services
SENDGRID_API_KEY=your-sendgrid-key
CLOUDINARY_URL=cloudinary://your-cloudinary-url
```

**.env.production** (Production):
```bash
# Database
DATABASE_URL=postgresql://prod-server:5432/college_prod

# API Keys
NEXT_PUBLIC_API_URL=https://college.com/api
STRIPE_SECRET_KEY=sk_live_xxxxxxxxxxxxx
NEXT_PUBLIC_STRIPE_PUBLIC_KEY=pk_live_xxxxxxxxxxxxx

# Authentication
NEXTAUTH_SECRET=your-production-secret-key
NEXTAUTH_URL=https://college.com

# Third-party services
SENDGRID_API_KEY=your-production-sendgrid-key
CLOUDINARY_URL=cloudinary://your-production-cloudinary-url
```

**.env.example** (Template - committed to git):
```bash
# Database
DATABASE_URL=

# API Keys
NEXT_PUBLIC_API_URL=
STRIPE_SECRET_KEY=
NEXT_PUBLIC_STRIPE_PUBLIC_KEY=

# Authentication
NEXTAUTH_SECRET=
NEXTAUTH_URL=

# Third-party services
SENDGRID_API_KEY=
CLOUDINARY_URL=
```

### Using Environment Variables

```typescript
// next.config.js
module.exports = {
  env: {
    CUSTOM_KEY: process.env.CUSTOM_KEY,
  },
  // OR use built-in support (recommended)
};

// In your code - Server side only
const dbUrl = process.env.DATABASE_URL;
const stripeSecret = process.env.STRIPE_SECRET_KEY;

// In your code - Client side (must be prefixed with NEXT_PUBLIC_)
const apiUrl = process.env.NEXT_PUBLIC_API_URL;
const stripePublic = process.env.NEXT_PUBLIC_STRIPE_PUBLIC_KEY;

// Type safety with TypeScript
// env.d.ts
declare namespace NodeJS {
  interface ProcessEnv {
    DATABASE_URL: string;
    STRIPE_SECRET_KEY: string;
    NEXT_PUBLIC_API_URL: string;
    NEXT_PUBLIC_STRIPE_PUBLIC_KEY: string;
    NEXTAUTH_SECRET: string;
    NEXTAUTH_URL: string;
  }
}
```

### .gitignore

```bash
# Environment variables
.env
.env.local
.env.*.local

# Keep .env.example
!.env.example

# Dependencies
node_modules/

# Build outputs
.next/
out/
dist/
build/

# Testing
coverage/

# Misc
.DS_Store
*.log
```

---

## Build Optimization

Optimize your build for production.

### Next.js Configuration

**next.config.js**:
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Enable React strict mode
  reactStrictMode: true,

  // Compress responses
  compress: true,

  // Remove console logs in production
  compiler: {
    removeConsole: process.env.NODE_ENV === 'production',
  },

  // Image optimization
  images: {
    domains: ['images.college.edu', 'cdn.college.edu'],
    formats: ['image/avif', 'image/webp'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
    minimumCacheTTL: 60,
  },

  // Headers for security and caching
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          {
            key: 'X-DNS-Prefetch-Control',
            value: 'on'
          },
          {
            key: 'Strict-Transport-Security',
            value: 'max-age=63072000; includeSubDomains; preload'
          },
          {
            key: 'X-Frame-Options',
            value: 'SAMEORIGIN'
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff'
          },
          {
            key: 'X-XSS-Protection',
            value: '1; mode=block'
          },
          {
            key: 'Referrer-Policy',
            value: 'origin-when-cross-origin'
          },
        ],
      },
    ];
  },

  // Redirects
  async redirects() {
    return [
      {
        source: '/old-login',
        destination: '/login',
        permanent: true,
      },
    ];
  },

  // Rewrites (for API proxy)
  async rewrites() {
    return [
      {
        source: '/api/:path*',
        destination: 'https://api.college.edu/:path*',
      },
    ];
  },

  // Webpack customization
  webpack: (config, { isServer }) => {
    // Add custom webpack config here
    return config;
  },
};

module.exports = nextConfig;
```

### Bundle Analysis

```javascript
// next.config.js with bundle analyzer
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer({
  // Your Next.js config
});
```

Run: `ANALYZE=true npm run build`

---

## Production Builds

### Next.js Build

```bash
# Build for production
npm run build

# Output:
# .next/           - Build output
# .next/static/    - Static assets (CSS, JS)
# .next/server/    - Server-side code

# Start production server
npm run start
```

### Build Output Analysis

```bash
Route (app)                              Size     First Load JS
┌ ○ /                                    1.2 kB         85.3 kB
├ ○ /about                               890 B          84.1 kB
├ ƒ /api/students                        0 B            0 B
├ ○ /dashboard                           2.4 kB         86.5 kB
└ ○ /login                               1.8 kB         85.9 kB

○  (Static)  automatically rendered as static HTML (uses no initial props)
ƒ  (Dynamic) server-rendered on demand
```

**Symbols**:
- `○` Static: Pre-rendered at build time
- `ƒ` Dynamic: Rendered on each request
- `●` SSG: Static Site Generation
- `λ` SSR: Server-Side Rendering

### Vite Build (React)

```bash
# Build for production
npm run build

# Output:
# dist/            - Build output
# dist/assets/     - Hashed static assets

# Preview build locally
npm run preview
```

**vite.config.ts**:
```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    outDir: 'dist',
    sourcemap: false,
    minify: 'terser',
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          router: ['react-router-dom'],
        },
      },
    },
  },
});
```

---

## Deploying Next.js to Vercel

Vercel is the company behind Next.js and offers the best deployment experience.

### Setup and Configuration

1. **Install Vercel CLI**:
```bash
npm install -g vercel
```

2. **Login to Vercel**:
```bash
vercel login
```

3. **Deploy**:
```bash
vercel
```

### Vercel Configuration

**vercel.json**:
```json
{
  "buildCommand": "npm run build",
  "outputDirectory": ".next",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "framework": "nextjs",
  "regions": ["iad1"],
  "env": {
    "DATABASE_URL": "@database-url",
    "NEXTAUTH_SECRET": "@nextauth-secret"
  },
  "build": {
    "env": {
      "NEXT_PUBLIC_API_URL": "https://api.college.edu"
    }
  }
}
```

### Environment Variables in Vercel

**Via Dashboard**:
1. Go to project settings
2. Navigate to "Environment Variables"
3. Add variables for Production, Preview, and Development
4. Click "Save"

**Via CLI**:
```bash
# Add environment variable
vercel env add DATABASE_URL production

# Pull environment variables
vercel env pull .env.local
```

### Custom Domains

1. **Add Domain in Vercel Dashboard**:
   - Go to "Domains" tab
   - Click "Add"
   - Enter domain: `college.edu`

2. **Configure DNS**:
   - Add A record: `76.76.21.21`
   - OR CNAME record: `cname.vercel-dns.com`

3. **SSL Certificate**:
   - Automatically provisioned by Vercel
   - Free SSL from Let's Encrypt

### Preview Deployments

Every push to a branch creates a unique preview URL:

```bash
# Push to feature branch
git checkout -b feature/new-dashboard
git push origin feature/new-dashboard

# Vercel automatically deploys to:
# https://college-git-feature-new-dashboard-username.vercel.app
```

---

## Deploying React to Netlify

Netlify is great for static React applications.

### Setup

1. **Install Netlify CLI**:
```bash
npm install -g netlify-cli
```

2. **Login**:
```bash
netlify login
```

3. **Initialize**:
```bash
netlify init
```

### Netlify Configuration

**netlify.toml**:
```toml
[build]
  command = "npm run build"
  publish = "dist"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-XSS-Protection = "1; mode=block"
    X-Content-Type-Options = "nosniff"
    Referrer-Policy = "strict-origin-when-cross-origin"

[[headers]]
  for = "/static/*"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"

[dev]
  command = "npm run dev"
  port = 3000

[functions]
  directory = "netlify/functions"
```

### Environment Variables

```bash
# Add via CLI
netlify env:set DATABASE_URL "your-database-url"

# OR via dashboard:
# Site settings > Build & deploy > Environment
```

### Deploy

```bash
# Deploy to preview
netlify deploy

# Deploy to production
netlify deploy --prod
```

### Serverless Functions (Optional)

**netlify/functions/hello.ts**:
```typescript
import { Handler } from '@netlify/functions';

export const handler: Handler = async (event, context) => {
  return {
    statusCode: 200,
    body: JSON.stringify({ message: 'Hello from Netlify Function!' }),
  };
};
```

Access at: `https://yoursite.netlify.app/.netlify/functions/hello`

---

## Docker Containerization

Containerize your application for consistent deployment across environments.

### Dockerfile for Next.js

**Dockerfile**:
```dockerfile
# Multi-stage build for smaller image

# Stage 1: Dependencies
FROM node:18-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --only=production

# Stage 2: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 3: Runner
FROM node:18-alpine AS runner
WORKDIR /app

ENV NODE_ENV production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000

CMD ["node", "server.js"]
```

**next.config.js** (for standalone mode):
```javascript
module.exports = {
  output: 'standalone',
};
```

**.dockerignore**:
```
node_modules
.next
.git
.env.local
npm-debug.log
```

### Docker Compose

**docker-compose.yml**:
```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - NEXTAUTH_SECRET=${NEXTAUTH_SECRET}
    depends_on:
      - db

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=college
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  postgres_data:
```

### Build and Run

```bash
# Build image
docker build -t college-app .

# Run container
docker run -p 3000:3000 college-app

# With docker-compose
docker-compose up -d
```

---

## CI/CD with GitHub Actions

Automate testing and deployment on every commit.

### GitHub Actions Workflow

**.github/workflows/ci.yml**:
```yaml
name: CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run type check
        run: npm run type-check

      - name: Run tests
        run: npm run test:ci

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/coverage-final.json

  build:
    runs-on: ubuntu-latest
    needs: test

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build application
        run: npm run build
        env:
          NEXT_PUBLIC_API_URL: ${{ secrets.NEXT_PUBLIC_API_URL }}

      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: build
          path: .next

  deploy:
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v3

      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
```

### E2E Testing in CI

**.github/workflows/e2e.yml**:
```yaml
name: E2E Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  e2e:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright
        run: npx playwright install --with-deps

      - name: Build app
        run: npm run build

      - name: Run Playwright tests
        run: npm run test:e2e

      - name: Upload test results
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
```

---

## Monitoring and Analytics

### Error Tracking with Sentry

```bash
npm install @sentry/nextjs
```

**sentry.client.config.ts**:
```typescript
import * as Sentry from '@sentry/nextjs';

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 1.0,
  beforeSend(event, hint) {
    // Filter out specific errors
    if (event.exception) {
      const error = hint.originalException;
      if (error && error.message?.includes('ResizeObserver')) {
        return null; // Ignore this error
      }
    }
    return event;
  },
});
```

**Usage**:
```typescript
import * as Sentry from '@sentry/nextjs';

try {
  await riskyOperation();
} catch (error) {
  Sentry.captureException(error, {
    tags: {
      section: 'enrollment',
    },
    extra: {
      studentId: student.id,
      courseId: course.id,
    },
  });
}
```

### Google Analytics

```typescript
// lib/gtag.ts
export const GA_TRACKING_ID = process.env.NEXT_PUBLIC_GA_ID;

export const pageview = (url: string) => {
  if (typeof window !== 'undefined' && window.gtag) {
    window.gtag('config', GA_TRACKING_ID!, {
      page_path: url,
    });
  }
};

export const event = ({
  action,
  category,
  label,
  value,
}: {
  action: string;
  category: string;
  label: string;
  value?: number;
}) => {
  if (typeof window !== 'undefined' && window.gtag) {
    window.gtag('event', action, {
      event_category: category,
      event_label: label,
      value: value,
    });
  }
};
```

**app/layout.tsx**:
```typescript
import Script from 'next/script';
import { GA_TRACKING_ID } from '@/lib/gtag';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <head>
        <Script
          src={`https://www.googletagmanager.com/gtag/js?id=${GA_TRACKING_ID}`}
          strategy="afterInteractive"
        />
        <Script id="google-analytics" strategy="afterInteractive">
          {`
            window.dataLayer = window.dataLayer || [];
            function gtag(){dataLayer.push(arguments);}
            gtag('js', new Date());
            gtag('config', '${GA_TRACKING_ID}');
          `}
        </Script>
      </head>
      <body>{children}</body>
    </html>
  );
}
```

### Vercel Analytics

```bash
npm install @vercel/analytics
```

```typescript
// app/layout.tsx
import { Analytics } from '@vercel/analytics/react';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  );
}
```

---

## Production Checklist

### Pre-Deployment Checklist

- [ ] **Tests**: All tests passing
- [ ] **Build**: Production build works locally
- [ ] **Environment Variables**: All production env vars set
- [ ] **Database**: Migrations applied
- [ ] **Error Tracking**: Sentry or similar configured
- [ ] **Analytics**: GA or similar configured
- [ ] **SEO**: Meta tags, sitemap, robots.txt
- [ ] **Performance**: Lighthouse score > 90
- [ ] **Security**: All security headers set
- [ ] **HTTPS**: SSL certificate configured
- [ ] **Backup**: Database backup strategy
- [ ] **Monitoring**: Health checks configured
- [ ] **Documentation**: README updated

### Post-Deployment Checklist

- [ ] **Smoke Tests**: Critical flows work
- [ ] **Error Monitoring**: Check Sentry for errors
- [ ] **Performance**: Real User Monitoring
- [ ] **Analytics**: Events tracking correctly
- [ ] **SSL**: Certificate valid and auto-renews
- [ ] **Domain**: DNS configured correctly
- [ ] **Team Notification**: Team knows about deployment
- [ ] **Rollback Plan**: Know how to rollback if needed

---

## SEO Optimization

### Next.js Metadata

```typescript
// app/layout.tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: {
    default: 'College Management System',
    template: '%s | College Management System',
  },
  description: 'Comprehensive college management system for students, instructors, and administrators',
  keywords: ['college', 'education', 'management', 'students', 'courses'],
  authors: [{ name: 'Saiprasad Hegde' }],
  openGraph: {
    type: 'website',
    locale: 'en_US',
    url: 'https://college.edu',
    siteName: 'College Management System',
    images: [
      {
        url: 'https://college.edu/og-image.jpg',
        width: 1200,
        height: 630,
        alt: 'College Management System',
      },
    ],
  },
  twitter: {
    card: 'summary_large_image',
    site: '@college',
    creator: '@college',
  },
  robots: {
    index: true,
    follow: true,
    googleBot: {
      index: true,
      follow: true,
      'max-video-preview': -1,
      'max-image-preview': 'large',
      'max-snippet': -1,
    },
  },
};
```

### Dynamic Metadata

```typescript
// app/students/[id]/page.tsx
import type { Metadata } from 'next';

type Props = {
  params: { id: string };
};

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const student = await fetchStudent(params.id);

  return {
    title: student.name,
    description: `Student profile for ${student.name}`,
    openGraph: {
      title: student.name,
      description: `Student profile for ${student.name}`,
      images: [student.avatar],
    },
  };
}

export default function StudentPage({ params }: Props) {
  // Component code
}
```

### Sitemap

```typescript
// app/sitemap.ts
import { MetadataRoute } from 'next';

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const students = await fetchAllStudents();

  const studentUrls = students.map((student) => ({
    url: `https://college.edu/students/${student.id}`,
    lastModified: student.updatedAt,
    changeFrequency: 'weekly' as const,
    priority: 0.8,
  }));

  return [
    {
      url: 'https://college.edu',
      lastModified: new Date(),
      changeFrequency: 'daily',
      priority: 1,
    },
    {
      url: 'https://college.edu/about',
      lastModified: new Date(),
      changeFrequency: 'monthly',
      priority: 0.8,
    },
    ...studentUrls,
  ];
}
```

### Robots.txt

```typescript
// app/robots.ts
import { MetadataRoute } from 'next';

export default function robots(): MetadataRoute.Robots {
  return {
    rules: {
      userAgent: '*',
      allow: '/',
      disallow: ['/admin/', '/api/'],
    },
    sitemap: 'https://college.edu/sitemap.xml',
  };
}
```

---

## Security Best Practices

### Security Headers

```javascript
// next.config.js
module.exports = {
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          {
            key: 'X-DNS-Prefetch-Control',
            value: 'on'
          },
          {
            key: 'Strict-Transport-Security',
            value: 'max-age=63072000; includeSubDomains; preload'
          },
          {
            key: 'X-Frame-Options',
            value: 'SAMEORIGIN'
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff'
          },
          {
            key: 'X-XSS-Protection',
            value: '1; mode=block'
          },
          {
            key: 'Referrer-Policy',
            value: 'origin-when-cross-origin'
          },
          {
            key: 'Permissions-Policy',
            value: 'camera=(), microphone=(), geolocation=()'
          },
        ],
      },
    ];
  },
};
```

### Content Security Policy

```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const nonce = Buffer.from(crypto.randomUUID()).toString('base64');

  const cspHeader = `
    default-src 'self';
    script-src 'self' 'nonce-${nonce}' 'strict-dynamic';
    style-src 'self' 'nonce-${nonce}';
    img-src 'self' blob: data: https:;
    font-src 'self';
    object-src 'none';
    base-uri 'self';
    form-action 'self';
    frame-ancestors 'none';
    upgrade-insecure-requests;
  `.replace(/\s{2,}/g, ' ').trim();

  const requestHeaders = new Headers(request.headers);
  requestHeaders.set('x-nonce', nonce);
  requestHeaders.set('Content-Security-Policy', cspHeader);

  const response = NextResponse.next({
    request: {
      headers: requestHeaders,
    },
  });

  response.headers.set('Content-Security-Policy', cspHeader);

  return response;
}
```

### Environment Variable Validation

```typescript
// lib/env.ts
import { z } from 'zod';

const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  NEXTAUTH_SECRET: z.string().min(32),
  NEXTAUTH_URL: z.string().url(),
  STRIPE_SECRET_KEY: z.string().startsWith('sk_'),
  NEXT_PUBLIC_API_URL: z.string().url(),
});

export const env = envSchema.parse(process.env);
```

---

## Complete Deployment Example

### Full Deployment Workflow

**Step 1: Prepare Code**
```bash
# Ensure all tests pass
npm run test

# Ensure build works
npm run build

# Check for type errors
npm run type-check

# Lint code
npm run lint
```

**Step 2: Configure Environment**
```bash
# Create production environment file
cp .env.example .env.production

# Fill in production values
vim .env.production
```

**Step 3: Deploy to Vercel**
```bash
# Install Vercel CLI
npm i -g vercel

# Login
vercel login

# Deploy
vercel --prod
```

**Step 4: Configure Domain**
```bash
# Add domain in Vercel dashboard
# Update DNS records at registrar

# Verify domain
vercel domains verify college.edu
```

**Step 5: Setup Monitoring**
```bash
# Add Sentry
npm install @sentry/nextjs

# Configure Sentry
npx @sentry/wizard -i nextjs

# Add environment variables to Vercel
vercel env add SENTRY_DSN production
```

**Step 6: Verify Deployment**
```bash
# Check site is live
curl -I https://college.edu

# Check SSL
curl -vI https://college.edu

# Run Lighthouse audit
npx lighthouse https://college.edu --view
```

---

## Best Practices

1. **Use Environment Variables**: Never hardcode secrets
2. **Automate with CI/CD**: Test and deploy automatically
3. **Monitor Everything**: Errors, performance, usage
4. **Security First**: Always use HTTPS, set security headers
5. **Optimize Images**: Use Next.js Image component
6. **CDN Everything**: Serve static assets from CDN
7. **Database Backups**: Automate daily backups
8. **Health Checks**: Monitor application health
9. **Gradual Rollouts**: Use feature flags for new features
10. **Documentation**: Document deployment process

---

## Common Pitfalls

1. **Exposing Secrets**: Committing .env files
2. **No SSL**: Deploying without HTTPS
3. **Missing Error Tracking**: No visibility into production errors
4. **No Backups**: Losing data without backups
5. **Hardcoded URLs**: Not using environment variables
6. **No Monitoring**: Flying blind in production
7. **Large Bundles**: Not optimizing bundle size
8. **Slow Builds**: Not caching dependencies
9. **No Rollback Plan**: Can't quickly revert bad deploys
10. **Ignoring Security**: Missing security headers

---

## Conclusion

Congratulations! You've completed the **Frontend Development Guide**!

You've learned:
- React fundamentals and advanced patterns
- Next.js and routing
- Forms, validation, and styling
- State management
- Performance optimization
- Testing strategies
- Deployment and production best practices

### Next Steps

1. **Build Projects**: Apply what you've learned
2. **Contribute to Open Source**: Practice on real-world code
3. **Stay Updated**: Follow React and Next.js updates
4. **Join Communities**: Discord, Reddit, Twitter
5. **Keep Learning**: Frontend evolves constantly

### Resources

- [React Documentation](https://react.dev)
- [Next.js Documentation](https://nextjs.org/docs)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [MDN Web Docs](https://developer.mozilla.org)
- [Web.dev](https://web.dev)

---

**Navigation:**
- **Previous**: [Part 13: Testing](13_Testing.md)
- **[Back to Index](README.md)**

---

**Author: Saiprasad Hegde**

**Thank you for completing this guide! Happy coding!**
