# Video Testimony Platform

> A full-stack SaaS platform for collecting, curating, and showcasing video testimonies.

**Role:** Lead Full-Stack Developer  
**Timeline:** 2024 - Present  
**Status:** Live in Production
shareheart.io

---

## Overview

Built a complete video testimony platform that enables organizations to collect authentic video stories from their community through a streamlined 3-step capture flow, then display them on beautiful, embeddable video walls.

The platform serves organizations ranging from small businesses to enterprises, handling everything from video recording and processing to subscription billing and custom domain management.

## Tech Stack

| Category | Technologies |
|----------|-------------|
| **Frontend** | React 18, TypeScript, Vite |
| **UI/Styling** | Tailwind CSS, shadcn/ui, Radix UI, Framer Motion |
| **Backend** | Supabase (PostgreSQL, Auth, Row-Level Security) |
| **Serverless** | Supabase Edge Functions (Deno) |
| **Video** | Mux (upload, transcoding, adaptive streaming, captions) |
| **Payments** | Stripe (subscriptions, webhooks, customer portal) |
| **Hosting** | Cloudflare Pages (with custom domains) |
| **State** | TanStack Query (React Query) |
| **Forms** | React Hook Form, Zod validation |

## Key Features I Built

### 1. Video Capture Flow
Built a mobile-first, 3-step video recording experience that works entirely in the browser:
- WebRTC-based video recording with live preview
- Client-side video validation and re-record capability
- Direct upload to Mux with progress tracking
- Automatic fallback to Supabase Storage if Mux is unavailable
- Responsive design optimized for mobile submission

### 2. Video Processing Pipeline
Integrated Mux for professional-grade video handling:
- Direct uploads via signed URLs (no server bandwidth needed)
- Automatic transcoding to adaptive bitrate streaming
- AI-powered caption generation
- Thumbnail extraction
- Webhook handlers for processing status updates

### 3. Video Wall Display
Created an embeddable, shareable video wall component:
- Featured video carousel with hero display
- Responsive grid layout for community testimonies
- Filter tabs (All / Featured / Recent)
- Lazy loading with intersection observers
- Custom branding support (colors, logos)
- Share dialog with copy-to-clipboard and social links

### 4. Subscription Billing System
Implemented complete Stripe integration:
- 4-tier pricing model (Free, Growth, Brand, Custom)
- Monthly and annual billing cycles
- Stripe Checkout for secure payments
- Customer portal for self-service management
- Webhook handlers for subscription lifecycle events
- Usage tracking and plan limit enforcement
- Grace periods for downgrade overflow handling

### 5. Custom Domain System
Built white-label domain support from scratch:
- DNS verification via TXT records
- Cloudflare API integration for automatic SSL
- Domain status tracking and validation
- Support for subdomains and apex domains
- Detailed setup guides for major DNS providers

### 6. Admin Dashboard
Comprehensive management interface:
- Review queue for pending submissions
- Approve/reject workflow with email notifications
- Feature toggle to highlight standout testimonies
- Bulk actions for efficient management
- Video analytics (views, watch time)
- Team management with role-based access
- Organization settings and branding

### 7. Multi-Tenant Architecture
Designed for B2B SaaS from day one:
- Organization-scoped data with Row-Level Security
- Team invitations via email with secure tokens
- Role-based permissions (Owner, Admin, Member)
- Per-organization plan limits and usage tracking
- Isolated data between tenants

### 8. Email Automation
Built transactional email system using React Email:
- Welcome emails for new signups
- Submission confirmation for testimony givers
- Approval notifications with video wall links
- Team invitation emails
- Subscription change notifications
- Downgrade reminder campaigns

## Technical Challenges Solved

### Challenge: Video Upload Reliability
**Problem:** Large video files (100MB+) frequently failed on mobile networks.

**Solution:** Implemented Mux direct uploads with resumable upload support, client-side file validation before upload, and automatic retry logic with exponential backoff. Added Supabase Storage as a fallback for edge cases.

### Challenge: Real-Time Plan Limit Enforcement
**Problem:** Needed to enforce video/story limits without blocking the UI or creating race conditions.

**Solution:** Built a hooks-based system (`usePlanLimits`) that checks limits client-side for UX, with server-side enforcement via RLS policies and Edge Functions. Added overflow handling with grace periods for plan downgrades.

### Challenge: Custom Domains at Scale
**Problem:** Supporting customer-owned domains while maintaining SSL and routing.

**Solution:** Integrated Cloudflare API for automatic DNS verification and SSL provisioning. Built a verification flow using TXT records, with background jobs to check domain status and clean up orphaned entries.

## Architecture Highlights

```
┌─────────────────────────────────────────────────────────────────┐
│                        Cloudflare Pages                         │
│                    (React SPA + Edge Routing)                   │
└─────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                         Supabase                                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │  PostgreSQL │  │    Auth     │  │    Edge Functions       │  │
│  │    + RLS    │  │   (JWT)     │  │    (40+ functions)      │  │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                                  │
                    ┌─────────────┼─────────────┐
                    ▼             ▼             ▼
              ┌─────────┐  ┌───────────┐  ┌───────────┐
              │   Mux   │  │  Stripe   │  │ Cloudflare│
              │ (Video) │  │ (Billing) │  │   (DNS)   │
              └─────────┘  └───────────┘  └───────────┘
```

## Edge Functions Built

| Category | Functions | Purpose |
|----------|-----------|---------|
| **Video** | 6 | Upload URLs, webhooks, batch signing, resolution logging |
| **Billing** | 5 | Checkout, portal, webhooks, billing info, plan enforcement |
| **Email** | 8 | Welcome, submission, approval, invitation, subscription notifications |
| **Domains** | 3 | Add, verify, cleanup custom domains |
| **Maintenance** | 5 | Cleanup jobs, grace period processing, overflow handling |
| **Other** | 5 | AI question generation, campaign tokens, analytics |

## Results

- **Production-ready SaaS** serving real customers
- **Custom domain support** for white-label deployments
- **Mobile-first** video capture with 90%+ completion rate

## Contact

Interested in discussing this project or my approach to building it?

- **GitHub:** JaStacks
- **LinkedIn:** https://www.linkedin.com/in/jareice-graham-93226b224
- **Email:** jtgraham@uci.edu

