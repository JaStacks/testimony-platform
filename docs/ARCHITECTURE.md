# Technical Architecture Deep Dive

This document provides additional technical detail for those interested in the system design decisions behind the Video Testimony Platform.

## System Design Principles

### 1. Serverless-First
Every backend operation runs as a Supabase Edge Function (Deno runtime). This provides:
- Zero server management
- Automatic scaling
- Pay-per-invocation pricing
- Geographic distribution

### 2. Security by Default
- **Row-Level Security (RLS)** on every table
- **JWT verification** in every Edge Function
- **Webhook signature validation** for Stripe and Mux
- **Scoped API keys** with minimal permissions

### 3. Progressive Enhancement
- Core functionality works without JavaScript (forms submit normally)
- Video recording gracefully degrades to file upload
- Offline-friendly with optimistic UI updates

## Data Model Overview

```
organizations
├── users (team members)
├── campaigns (collections)
│   ├── testimonies (video submissions)
│   └── default_videos (intro videos)
├── custom_domains
└── subscriptions (Stripe data)
```

### Key Relationships

- **Organizations** are the top-level tenant boundary
- **Users** belong to organizations with roles (owner/admin/member)
- **Campaigns** (collections) belong to organizations
- **Testimonies** belong to campaigns and contain video metadata
- **Subscriptions** link organizations to Stripe for billing

## Video Processing Flow

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│    Client    │     │ Edge Function│     │         Mux  │
│   (Browser)  │     │              │     │              │
└──────┬───────┘     └──────┬───────┘     └──────┬───────┘
       │                    │                    │
       │ 1. Request upload                       │
       │───────────────────>│                    │
       │                    │ 2. Create upload   │
       │                    │───────────────────>
       │                    │                    │
       │                    │ 3. Signed URL    
       │                    │<───────────────────
       │ 4. Upload URL 
       │<───────────────────│
       │                    │                    │
       │ 5. Direct upload (no server bandwidth)  │
       │────────────────────────────────────────>│
       │                    │                    │
       │                    │ 6. Webhook: ready  │
       │                    │<───────────────────│
       │                    │                    │
       │                    │ 7. Update DB       │
       │                    │ (playback ID,      │
       │                    │  captions, etc.)   │
       │                    │                    │
```

### Why Mux Direct Uploads?

1. **No server bandwidth** — Videos go directly from browser to Mux
2. **Resumable** — Large files can resume after network interruption
3. **Signed URLs** — Time-limited, secure upload endpoints
4. **Automatic processing** — Transcoding starts immediately

## Subscription Architecture

### Plan Enforcement Strategy

```typescript
// Client-side: Fast feedback, not authoritative
const { canUpload, remaining } = usePlanLimits('videosPerMonth');

// Server-side: Authoritative enforcement
// Edge Function checks before creating upload URL
const usage = await getMonthlyUsage(orgId);
const limits = await getPlanLimits(orgId);
if (usage >= limits.videosPerMonth) {
  throw new Error('Plan limit reached');
}

// Database-level: Final safety net
// RLS policies prevent inserts beyond limits
```

### Downgrade Handling

When a customer downgrades (e.g., Brand → Growth):
1. **Grace period** (7 days) to review overflow content
2. **Email reminders** at 3 days and 1 day remaining
3. **Automatic archival** of overflow testimonies after grace period
4. **No data deletion** — archived content can be restored on upgrade

## Authentication & Authorization

### Auth Flow
- **Supabase Auth** with email/password and magic links
- **JWT tokens** validated on every Edge Function call
- **Organization context** determined by JWT claims + request params

### Permission Model

```
Owner
├── All admin permissions
├── Delete organization
├── Transfer ownership
└── Manage billing

Admin  
├── Manage campaigns
├── Approve/reject testimonies
├── Invite team members
└── View analytics

Member
├── View campaigns
├── View testimonies
└── View analytics (limited)
```

### Row-Level Security Example

```sql
-- Users can only see testimonies from their organization's campaigns
CREATE POLICY "Users can view org testimonies" ON testimonies
  FOR SELECT
  USING (
    campaign_id IN (
      SELECT id FROM campaigns 
      WHERE organization_id IN (
        SELECT organization_id FROM organization_users 
        WHERE user_id = auth.uid()
      )
    )
  );
```

## Performance Optimizations

### Video Wall Loading
1. **Lazy loading** — Videos load as they enter viewport
2. **Thumbnail placeholders** — Mux thumbnails show before video loads
3. **Batch signing** — Single Edge Function call signs all video URLs
4. **Staggered animations** — Perceived performance via progressive reveal

### Database Queries
1. **Composite indexes** on frequently filtered columns
2. **Materialized counts** for dashboard stats
3. **Pagination** with cursor-based infinite scroll
4. **Connection pooling** via Supabase's built-in Supavisor

### Bundle Size
1. **Code splitting** per route
2. **Tree shaking** for UI components
3. **Dynamic imports** for heavy dependencies (video player, charts)
4. **Preloading** of likely next routes

## Error Handling Philosophy

### Client-Side
- **Optimistic updates** with rollback on failure
- **Toast notifications** for user-facing errors
- **Error boundaries** to prevent full-page crashes
- **Retry logic** for transient network failures

### Server-Side
- **Structured logging** with request IDs
- **Graceful degradation** (e.g., Mux → Supabase Storage fallback)
- **Webhook retry handling** with idempotency keys
- **Dead letter queues** for failed async operations

## Monitoring & Observability

- **Supabase Dashboard** for database metrics and logs
- **Mux Dashboard** for video analytics
- **Stripe Dashboard** for billing events
- **Cloudflare Analytics** for traffic and performance

---

*This architecture has been refined over months of production use, handling real customer traffic and edge cases.*

