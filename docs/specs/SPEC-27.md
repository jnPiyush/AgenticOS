# Technical Specification: PawConnect Platform

**Epic**: #27  
**Status**: Draft  
**Author**: AgentX Solution Architect  
**Date**: 2026-02-18  
**Related Documents**: [PRD-27.md](../prd/PRD-27.md) | [ADR-27.md](../adr/ADR-27.md) | [UX-27.md](../ux/UX-27.md)

---

## Table of Contents

1. [System Architecture](#1-system-architecture)
2. [Database Schema](#2-database-schema)
3. [API Specification](#3-api-specification)
4. [AI Matching Service](#4-ai-matching-service)
5. [Authentication & Authorization](#5-authentication--authorization)
6. [File Upload & Storage](#6-file-upload--storage)
7. [Real-time Messaging](#7-real-time-messaging)
8. [Payment Integration](#8-payment-integration)
9. [Email Service](#9-email-service)
10. [Deployment Guide](#10-deployment-guide)
11. [Monitoring & Observability](#11-monitoring--observability)
12. [Testing Strategy](#12-testing-strategy)

---

## 1. System Architecture

### 1.1 Technology Stack

**Frontend:**
- **Framework**: Next.js 14 (React 18)
- **Language**: TypeScript 5.x
- **Styling**: Tailwind CSS 3.x
- **State Management**: React Context API + React Query
- **Forms**: React Hook Form + Zod validation
- **UI Components**: Headless UI + Radix UI
- **Maps**: Leaflet.js (OpenStreetMap) + Google Maps API

**Backend:**
- **Runtime**: Node.js 20 LTS
- **Framework**: Express.js 4.x
- **Language**: TypeScript 5.x
- **ORM**: Prisma 5.x
- **Validation**: Zod
- **Authentication**: Passport.js + Azure AD B2C SDK
- **WebSocket**: Azure SignalR SDK

**Database:**
- **Primary**: PostgreSQL 16 (Azure Database for PostgreSQL)
- **Extensions**: PostGIS (geospatial), pg_trgm (full-text search)

**Infrastructure:**
- **Cloud Provider**: Microsoft Azure
- **Hosting**: Azure App Service (Linux, Node.js 20)
- **Storage**: Azure Blob Storage
- **CDN**: Azure CDN
- **AI/ML**: Azure OpenAI Service (GPT-4o-mini)
- **Auth**: Azure AD B2C
- **Messaging**: Azure SignalR Service
- **Monitoring**: Azure Monitor + Application Insights

**External Services:**
- **Payments**: Stripe API
- **Email**: Twilio SendGrid
- **Maps**: Google Maps Geocoding API
- **Content Moderation**: Azure Content Safety API

### 1.2 System Components

```
┌─────────────────────────────────────────────────────────────┐
│                     Client (Next.js PWA)                    │
│  - SSR for SEO                                              │
│  - Client-side routing                                      │
│  - Progressive Web App (offline support)                    │
└───────────────┬─────────────────────────────────────────────┘
                │
                │ REST API + WebSocket
                │
┌───────────────▼─────────────────────────────────────────────┐
│                  API Gateway / Load Balancer                │
│                    (Azure Front Door)                       │
└───────────────┬─────────────────────────────────────────────┘
                │
        ┌───────┴────────┐
        │                │
        ▼                ▼
┌──────────────┐  ┌──────────────┐
│  Next.js SSR │  │ API Server   │
│  Server      │  │ (Express.js) │
│              │  │              │
│ - Page routes│  │ - REST API   │
│ - API routes │  │ - Business   │
│              │  │   logic      │
└──────────────┘  └───┬──────────┘
                      │
        ┌─────────────┼──────────────┬────────────┐
        │             │              │            │
        ▼             ▼              ▼            ▼
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│PostgreSQL│  │Azure Blob│  │Azure AI  │  │ Azure    │
│          │  │ Storage  │  │ OpenAI   │  │ SignalR  │
└──────────┘  └──────────┘  └──────────┘  └──────────┘
```

---

## 2. Database Schema

### 2.1 Entity-Relationship Diagram

```
┌─────────────┐         ┌─────────────┐         ┌─────────────┐
│   users     │◄───────┤    dogs     │────────►│ dog_photos  │
│─────────────│ 1    N │─────────────│ 1    N  │─────────────│
│ id (PK)     │         │ id (PK)     │         │ id (PK)     │
│ email       │         │ user_id(FK) │         │ dog_id (FK) │
│ name        │         │ name        │         │ url         │
│ phone       │         │ breed       │         │ order       │
│ location    │         │ age         │         │ created_at  │
│ created_at  │         │ size        │         └─────────────┘
└─────────────┘         │ energy      │
       │                │ temperament │
       │ 1              │ location    │
       │                │ vaccinated  │
       │                └─────────────┘
       │                       │
       │                       │ N
       │                       ▼
       │                ┌─────────────┐
       │                │  playdates  │
       │                │─────────────│
       │                │ id (PK)     │
       │       ┌───────►│ requester   │
       │       │        │   _dog_id   │
       │       │        │ recipient   │
       │       │        │   _dog_id   │
       │       │        │ status      │
       │       │        │ date_time   │
       │       │        │ location    │
       │       │        │ feedback    │
       │       │        └─────────────┘
       │ 1     │
       ├───────┤
       │       │ N
       ▼       ▼
┌─────────────┐         ┌─────────────┐
│   events    │◄───────┤   rsvps     │
│─────────────│ 1    N │─────────────│
│ id (PK)     │         │ id (PK)     │
│ organizer   │         │ event_id(FK)│
│   _id (FK)  │         │ user_id (FK)│
│ title       │         │ dog_ids[]   │
│ category    │         │ status      │
│ date_time   │         │ created_at  │
│ location    │         └─────────────┘
│ max_attend  │
│ created_at  │
└─────────────┘

┌─────────────┐         ┌─────────────┐         ┌─────────────┐
│  service_   │◄───────┤  bookings   │────────►│   reviews   │
│  providers  │ 1    N │─────────────│ 1    1  │─────────────│
│─────────────│         │ id (PK)     │         │ id (PK)     │
│ id (PK)     │         │ provider_id │         │ booking_id  │
│ user_id(FK) │         │   (FK)      │         │   (FK)      │
│ business_nm │         │ customer_id │         │ user_id(FK) │
│ category    │         │   (FK)      │         │ rating      │
│ bio         │         │ service     │         │ comment     │
│ certific[]  │         │ date_time   │         │ photos[]    │
│ location    │         │ status      │         │ created_at  │
│ pricing     │         │ total       │         └─────────────┘
│ rating_avg  │         │ stripe_id   │
│ created_at  │         │ created_at  │
└─────────────┘         └─────────────┘

┌─────────────┐         ┌─────────────┐
│    posts    │◄───────┤  comments   │
│─────────────│ 1    N │─────────────│
│ id (PK)     │         │ id (PK)     │
│ user_id(FK) │         │ post_id(FK) │
│ dog_id (FK) │         │ user_id(FK) │
│ caption     │         │ content     │
│ photos[]    │         │ created_at  │
│ likes_count │         └─────────────┘
│ created_at  │
└─────────────┘

┌─────────────┐
│  messages   │
│─────────────│
│ id (PK)     │
│ sender_id   │
│   (FK)      │
│ recipient_id│
│   (FK)      │
│ content     │
│ read        │
│ created_at  │
└─────────────┘

┌─────────────┐
│  follows    │
│─────────────│
│ id (PK)     │
│ follower_id │
│   (FK)      │
│ following_id│
│   (FK)      │
│ created_at  │
└─────────────┘
```

### 2.2 Schema Definitions (Prisma)

```prisma
// schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id              String            @id @default(uuid())
  email           String            @unique
  name            String
  phone           String?
  avatarUrl       String?
  location        String?           // Address string
  locationPoint   Unsupported("geography(Point,4326)")? // PostGIS point
  role            UserRole          @default(OWNER)
  createdAt       DateTime          @default(now())
  updatedAt       DateTime          @updatedAt
  
  // Relations
  dogs            Dog[]
  organizedEvents Event[]           @relation("EventOrganizer")
  rsvps           RSVP[]
  serviceProvider ServiceProvider?
  bookingsAsCustomer Booking[]      @relation("CustomerBookings")
  reviews         Review[]
  posts           Post[]
  sentMessages    Message[]         @relation("MessageSender")
  receivedMessages Message[]        @relation("MessageRecipient")
  followers       Follow[]          @relation("Following")
  following       Follow[]          @relation("Follower")
  
  @@index([email])
  @@index([locationPoint], type: Gist) // PostGIS spatial index
  @@map("users")
}

enum UserRole {
  OWNER
  SERVICE_PROVIDER
  ADMIN
}

model Dog {
  id              String            @id @default(uuid())
  userId          String
  name            String
  breed           String
  age             Int               // in years
  size            DogSize
  gender          DogGender
  energyLevel     EnergyLevel
  temperament     String[]          // ["friendly", "playful", "shy"]
  description     String?
  vaccinated      Boolean           @default(false)
  vaccinationDate DateTime?
  location        String?           // Inherits from user if null
  locationPoint   Unsupported("geography(Point,4326)")? // PostGIS
  visibility      Visibility        @default(PUBLIC)
  createdAt       DateTime          @default(now())
  updatedAt       DateTime          @updatedAt
  
  // Relations
  user            User              @relation(fields: [userId], references: [id], onDelete: Cascade)
  photos          DogPhoto[]
  requestedPlaydates Playdate[]     @relation("PlaydateRequester")
  receivedPlaydates  Playdate[]     @relation("PlaydateRecipient")
  posts           Post[]
  
  @@index([userId])
  @@index([locationPoint], type: Gist) // PostGIS spatial index
  @@index([breed, size, age]) // Matching optimization
  @@map("dogs")
}

enum DogSize {
  SMALL   // <20 lbs
  MEDIUM  // 20-50 lbs
  LARGE   // 50-100 lbs
  XLARGE  // >100 lbs
}

enum DogGender {
  MALE
  FEMALE
}

enum EnergyLevel {
  LOW
  MEDIUM
  HIGH
}

enum Visibility {
  PUBLIC
  FRIENDS
  PRIVATE
}

model DogPhoto {
  id          String   @id @default(uuid())
  dogId       String
  url         String
  thumbnailUrl String?
  order       Int      @default(0)
  createdAt   DateTime @default(now())
  
  dog         Dog      @relation(fields: [dogId], references: [id], onDelete: Cascade)
  
  @@index([dogId])
  @@map("dog_photos")
}

model Playdate {
  id              String          @id @default(uuid())
  requesterDogId  String
  recipientDogId  String
  status          PlaydateStatus  @default(PENDING)
  proposedDate    DateTime
  proposedTime    String          // "2:00 PM - 4:00 PM"
  location        String
  locationPoint   Unsupported("geography(Point,4326)")? // PostGIS
  message         String?
  safetyChecklist Boolean         @default(false)
  requesterFeedback String?
  recipientFeedback String?
  requesterRating Int?            // 1-5 or thumbs up/down (1=down, 5=up)
  recipientRating Int?
  createdAt       DateTime        @default(now())
  updatedAt       DateTime        @updatedAt
  
  requesterDog    Dog             @relation("PlaydateRequester", fields: [requesterDogId], references: [id])
  recipientDog    Dog             @relation("PlaydateRecipient", fields: [recipientDogId], references: [id])
  
  @@index([requesterDogId, recipientDogId])
  @@index([status, proposedDate])
  @@map("playdates")
}

enum PlaydateStatus {
  PENDING
  ACCEPTED
  DECLINED
  COUNTER_PROPOSED
  COMPLETED
  CANCELLED
}

model Event {
  id          String          @id @default(uuid())
  organizerId String
  title       String
  description String
  category    EventCategory
  date        DateTime
  time        String          // "10:00 AM - 12:00 PM"
  location    String
  locationPoint Unsupported("geography(Point,4326)")? // PostGIS
  imageUrl    String?
  maxAttendees Int?
  createdAt   DateTime        @default(now())
  updatedAt   DateTime        @updatedAt
  
  organizer   User            @relation("EventOrganizer", fields: [organizerId], references: [id])
  rsvps       RSVP[]
  
  @@index([organizerId])
  @@index([date, category])
  @@index([locationPoint], type: Gist) // PostGIS spatial index
  @@map("events")
}

enum EventCategory {
  TRAINING_CLASS
  MEETUP
  ADOPTION_EVENT
  CHARITY_WALK
  DOG_SHOW
  OTHER
}

model RSVP {
  id          String      @id @default(uuid())
  eventId     String
  userId      String
  dogIds      String[]    // Array of dog IDs attending
  status      RSVPStatus  @default(GOING)
  createdAt   DateTime    @default(now())
  
  event       Event       @relation(fields: [eventId], references: [id], onDelete: Cascade)
  user        User        @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@unique([eventId, userId]) // One RSVP per user per event
  @@index([eventId])
  @@index([userId])
  @@map("rsvps")
}

enum RSVPStatus {
  GOING
  MAYBE
  NOT_GOING
}

model ServiceProvider {
  id              String              @id @default(uuid())
  userId          String              @unique
  businessName    String
  category        ServiceCategory
  bio             String
  certifications  String[]
  pricing         Json?               // {service: price}
  location        String
  locationPoint   Unsupported("geography(Point,4326)")? // PostGIS
  photoUrls       String[]
  ratingAverage   Float               @default(0)
  ratingCount     Int                 @default(0)
  createdAt       DateTime            @default(now())
  updatedAt       DateTime            @updatedAt
  
  user            User                @relation(fields: [userId], references: [id], onDelete: Cascade)
  bookings        Booking[]
  
  @@index([category, locationPoint], type: Gist) // Category + location search
  @@index([ratingAverage]) // Rating filter
  @@map("service_providers")
}

enum ServiceCategory {
  GROOMING
  TRAINING
  BOARDING
  VET
  WALKING
}

model Booking {
  id              String          @id @default(uuid())
  providerId      String
  customerId      String
  service         String
  date            DateTime
  time            String          // "2:00 PM"
  status          BookingStatus   @default(PENDING)
  notes           String?
  totalAmount     Int             // in cents
  stripePaymentId String?
  createdAt       DateTime        @default(now())
  updatedAt       DateTime        @updatedAt
  
  provider        ServiceProvider @relation(fields: [providerId], references: [id])
  customer        User            @relation("CustomerBookings", fields: [customerId], references: [id])
  review          Review?
  
  @@index([providerId, status])
  @@index([customerId])
  @@index([date])
  @@map("bookings")
}

enum BookingStatus {
  PENDING
  CONFIRMED
  COMPLETED
  CANCELLED
}

model Review {
  id          String   @id @default(uuid())
  bookingId   String   @unique
  userId      String
  rating      Int      // 1-5 stars
  comment     String
  photoUrls   String[]
  verified    Boolean  @default(true) // Verified booking
  createdAt   DateTime @default(now())
  
  booking     Booking  @relation(fields: [bookingId], references: [id], onDelete: Cascade)
  user        User     @relation(fields: [userId], references: [id])
  
  @@index([bookingId])
  @@map("reviews")
}

model Post {
  id          String    @id @default(uuid())
  userId      String
  dogId       String?
  caption     String
  photoUrls   String[]
  likesCount  Int       @default(0)
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
  
  user        User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  dog         Dog?      @relation(fields: [dogId], references: [id], onDelete: SetNull)
  comments    Comment[]
  
  @@index([userId, createdAt])
  @@index([dogId])
  @@map("posts")
}

model Comment {
  id          String   @id @default(uuid())
  postId      String
  userId      String
  content     String
  createdAt   DateTime @default(now())
  
  post        Post     @relation(fields: [postId], references: [id], onDelete: Cascade)
  
  @@index([postId])
  @@map("comments")
}

model Message {
  id            String   @id @default(uuid())
  senderId      String
  recipientId   String
  content       String
  read          Boolean  @default(false)
  createdAt     DateTime @default(now())
  
  sender        User     @relation("MessageSender", fields: [senderId], references: [id])
  recipient     User     @relation("MessageRecipient", fields: [recipientId], references: [id])
  
  @@index([senderId, recipientId])
  @@index([recipientId, read]) // Unread messages
  @@map("messages")
}

model Follow {
  id          String   @id @default(uuid())
  followerId  String
  followingId String
  createdAt   DateTime @default(now())
  
  follower    User     @relation("Follower", fields: [followerId], references: [id], onDelete: Cascade)
  following   User     @relation("Following", fields: [followingId], references: [id], onDelete: Cascade)
  
  @@unique([followerId, followingId])
  @@index([followerId])
  @@index([followingId])
  @@map("follows")
}
```

### 2.3 PostGIS Setup

```sql
-- Enable PostGIS extension
CREATE EXTENSION IF NOT EXISTS postgis;
CREATE EXTENSION IF NOT EXISTS pg_trgm; -- For full-text search

-- Add spatial indexes (Prisma doesn't handle these automatically)
CREATE INDEX idx_users_location ON users USING GIST (location_point);
CREATE INDEX idx_dogs_location ON dogs USING GIST (location_point);
CREATE INDEX idx_events_location ON events USING GIST (location_point);
CREATE INDEX idx_service_providers_location ON service_providers USING GIST (location_point);

-- Helper function: Convert address to coordinates (to be called from API)
-- API will use Google Geocoding API, then insert Point
-- Example: ST_SetSRID(ST_MakePoint(longitude, latitude), 4326)
```

---

## 3. API Specification

### 3.1 REST API Endpoints

**Base URL**: `https://api.pawconnect.com/v1`

**Authentication**: Bearer token (JWT from Azure AD B2C) in `Authorization` header

#### 3.1.1 Authentication

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| POST | `/auth/register` | Register new user with email/password | No |
| POST | `/auth/login` | Login with email/password | No |
| GET | `/auth/oauth/google` | Redirect to Google OAuth | No |
| GET | `/auth/oauth/google/callback` | Handle Google OAuth callback | No |
| POST | `/auth/refresh` | Refresh JWT token | Yes |
| POST | `/auth/logout` | Logout (invalidate token) | Yes |
| POST | `/auth/forgot-password` | Request password reset email | No |
| POST | `/auth/reset-password` | Reset password with token | No |

#### 3.1.2 Users

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| GET | `/users/me` | Get current user profile | Yes |
| PATCH | `/users/me` | Update current user profile | Yes |
| DELETE | `/users/me` | Delete account (GDPR right to erasure) | Yes |
| GET | `/users/:id` | Get user profile by ID | Yes |
| GET | `/users/:id/dogs` | Get user's dogs | Yes |

#### 3.1.3 Dogs

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| GET | `/dogs` | List dogs (with filters: location, breed, size) | Yes |
| POST | `/dogs` | Create dog profile | Yes |
| GET | `/dogs/:id` | Get dog profile by ID | Yes |
| PATCH | `/dogs/:id` | Update dog profile | Yes (owner only) |
| DELETE | `/dogs/:id` | Delete dog profile | Yes (owner only) |
| POST | `/dogs/:id/photos` | Upload dog photo | Yes (owner only) |
| DELETE | `/dogs/:id/photos/:photoId` | Delete dog photo | Yes (owner only) |

#### 3.1.4 Playdates

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| GET | `/playdates/matches` | Get AI-powered match suggestions for user's dog | Yes |
| POST | `/playdates` | Request a playdate | Yes |
| GET | `/playdates` | List playdates (sent/received) | Yes |
| GET | `/playdates/:id` | Get playdate details | Yes |
| PATCH | `/playdates/:id` | Accept/decline/counter-propose playdate | Yes |
| POST | `/playdates/:id/feedback` | Submit playdate feedback/rating | Yes |
| DELETE | `/playdates/:id` | Cancel playdate | Yes |

#### 3.1.5 Events

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| GET | `/events` | List events (with filters: date, category, location) | Yes |
| POST | `/events` | Create event | Yes |
| GET | `/events/:id` | Get event details | Yes |
| PATCH | `/events/:id` | Update event | Yes (organizer only) |
| DELETE | `/events/:id` | Delete event | Yes (organizer only) |
| POST | `/events/:id/rsvp` | RSVP to event | Yes |
| DELETE | `/events/:id/rsvp` | Cancel RSVP | Yes |
| GET | `/events/:id/attendees` | Get event attendees | Yes |

#### 3.1.6 Service Providers

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| GET | `/service-providers` | Search providers (filters: category, location, rating) | Yes |
| POST | `/service-providers` | Create provider profile | Yes |
| GET | `/service-providers/:id` | Get provider profile | Yes |
| PATCH | `/service-providers/:id` | Update provider profile | Yes (owner only) |
| DELETE | `/service-providers/:id` | Delete provider profile | Yes (owner only) |
| GET | `/service-providers/:id/reviews` | Get provider reviews | Yes |

#### 3.1.7 Bookings

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| POST | `/bookings` | Create booking request | Yes |
| GET | `/bookings` | List bookings (customer or provider) | Yes |
| GET | `/bookings/:id` | Get booking details | Yes |
| PATCH | `/bookings/:id` | Accept/decline/reschedule booking | Yes |
| DELETE | `/bookings/:id` | Cancel booking | Yes |
| POST | `/bookings/:id/review` | Submit review after booking | Yes |

#### 3.1.8 Posts (Social Feed)

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| GET | `/posts` | Get social feed (following users) | Yes |
| POST | `/posts` | Create post | Yes |
| GET | `/posts/:id` | Get post details | Yes |
| DELETE | `/posts/:id` | Delete post | Yes (owner only) |
| POST | `/posts/:id/like` | Like post | Yes |
| DELETE | `/posts/:id/like` | Unlike post | Yes |
| POST | `/posts/:id/comments` | Add comment | Yes |
| GET | `/posts/:id/comments` | Get comments | Yes |
| DELETE | `/posts/:id/comments/:commentId` | Delete comment | Yes (owner only) |

#### 3.1.9 Follows

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| POST | `/users/:id/follow` | Follow user | Yes |
| DELETE | `/users/:id/follow` | Unfollow user | Yes |
| GET | `/users/:id/followers` | Get followers list | Yes |
| GET | `/users/:id/following` | Get following list | Yes |

#### 3.1.10 Messages

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| GET | `/messages` | List conversations | Yes |
| POST | `/messages` | Send message | Yes |
| GET | `/messages/:userId` | Get conversation with user | Yes |
| PATCH | `/messages/:id/read` | Mark message as read | Yes |

### 3.2 API Request/Response Examples

#### 3.2.1 Create Dog Profile

**Request:**
```http
POST /v1/dogs
Authorization: Bearer <jwt_token>
Content-Type: application/json

{
  "name": "Buddy",
  "breed": "Golden Retriever",
  "age": 2,
  "size": "LARGE",
  "gender": "MALE",
  "energyLevel": "HIGH",
  "temperament": ["friendly", "playful", "gentle"],
  "description": "Loves fetch and swimming!",
  "vaccinated": true,
  "vaccinationDate": "2025-06-15",
  "visibility": "PUBLIC"
}
```

**Response:**
```json
{
  "id": "dog-uuid-123",
  "userId": "user-uuid-456",
  "name": "Buddy",
  "breed": "Golden Retriever",
  "age": 2,
  "size": "LARGE",
  "gender": "MALE",
  "energyLevel": "HIGH",
  "temperament": ["friendly", "playful", "gentle"],
  "description": "Loves fetch and swimming!",
  "vaccinated": true,
  "vaccinationDate": "2025-06-15T00:00:00.000Z",
  "visibility": "PUBLIC",
  "photos": [],
  "createdAt": "2026-02-18T12:00:00.000Z",
  "updatedAt": "2026-02-18T12:00:00.000Z"
}
```

#### 3.2.2 Get Playdate Matches

**Request:**
```http
GET /v1/playdates/matches?dogId=dog-uuid-123&radius=10
Authorization: Bearer <jwt_token>
```

**Response:**
```json
{
  "matches": [
    {
      "dog": {
        "id": "dog-uuid-789",
        "name": "Max",
        "breed": "Labrador Retriever",
        "age": 3,
        "size": "LARGE",
        "gender": "MALE",
        "energyLevel": "HIGH",
        "photos": [
          {"url": "https://cdn.pawconnect.com/dogs/max-1.jpg"}
        ],
        "owner": {
          "id": "user-uuid-999",
          "name": "Sarah Johnson",
          "avatarUrl": "https://cdn.pawconnect.com/avatars/sarah.jpg"
        },
        "distance": 2.3
      },
      "matchScore": 95,
      "matchReason": "Similar size, age, and energy level. Both love fetch and are friendly with other dogs."
    },
    {
      "dog": {
        "id": "dog-uuid-101",
        "name": "Luna",
        "breed": "Australian Shepherd",
        "age": 2,
        "size": "MEDIUM",
        "gender": "FEMALE",
        "energyLevel": "HIGH",
        "photos": [
          {"url": "https://cdn.pawconnect.com/dogs/luna-1.jpg"}
        ],
        "owner": {
          "id": "user-uuid-202",
          "name": "Mike Chen",
          "avatarUrl": "https://cdn.pawconnect.com/avatars/mike.jpg"
        },
        "distance": 4.7
      },
      "matchScore": 88,
      "matchReason": "Compatible energy levels and age. Luna is slightly smaller but very playful."
    }
  ],
  "metadata": {
    "totalMatches": 15,
    "returned": 2,
    "page": 1,
    "perPage": 10
  }
}
```

#### 3.2.3 Request Playdate

**Request:**
```http
POST /v1/playdates
Authorization: Bearer <jwt_token>
Content-Type: application/json

{
  "requesterDogId": "dog-uuid-123",
  "recipientDogId": "dog-uuid-789",
  "proposedDate": "2026-03-01",
  "proposedTime": "2:00 PM - 4:00 PM",
  "location": "Central Park Dog Run, New York, NY",
  "message": "Hi! Buddy would love to play with Max. How about Saturday afternoon?"
}
```

**Response:**
```json
{
  "id": "playdate-uuid-456",
  "requesterDog": {
    "id": "dog-uuid-123",
    "name": "Buddy"
  },
  "recipientDog": {
    "id": "dog-uuid-789",
    "name": "Max"
  },
  "status": "PENDING",
  "proposedDate": "2026-03-01T00:00:00.000Z",
  "proposedTime": "2:00 PM - 4:00 PM",
  "location": "Central Park Dog Run, New York, NY",
  "message": "Hi! Buddy would love to play with Max. How about Saturday afternoon?",
  "createdAt": "2026-02-18T12:00:00.000Z"
}
```

---

## 4. AI Matching Service

### 4.1 Architecture

```
Client → API Server → AI Matching Service → Azure OpenAI
                   ↓                       ↓
              PostgreSQL               Prompt Cache
             (candidate dogs)        (Redis - future)
```

### 4.2 Matching Algorithm Flow

1. **Input**: Source dog ID
2. **Query PostgreSQL**: Get source dog profile (breed, age, size, energy, temperament, location)
3. **Query PostgreSQL**: Get candidate dogs within radius (PostGIS):
   ```sql
   SELECT * FROM dogs
   WHERE ST_DWithin(
     location_point,
     (SELECT location_point FROM dogs WHERE id = $1),
     16093 -- 10 miles in meters
   )
   AND id != $1
   AND vaccinated = true
   LIMIT 50;
   ```
4. **Rule-based Pre-filtering**: Remove obviously incompatible dogs (e.g., XLarge vs Small)
5. **Call Azure OpenAI**: Send structured prompt with source dog + candidates
6. **Parse Response**: Extract match scores and reasons
7. **Sort & Return**: Top 10 matches with scores

### 4.3 Azure OpenAI Prompt

```typescript
const prompt = `You are a dog playdate compatibility expert. Analyze the following dog profiles and calculate compatibility scores (0-100) for each candidate dog with the source dog.

Consider these factors:
1. Size compatibility (dogs should be within 1 size category)
2. Age compatibility (similar life stage: puppy, adult, senior)
3. Energy level match (high-energy dogs match better with high-energy dogs)
4. Temperament compatibility (e.g., shy dogs may not match with overly playful dogs)
5. Gender (some dogs prefer same/opposite gender)

Source Dog:
- Name: ${sourceDog.name}
- Breed: ${sourceDog.breed}
- Age: ${sourceDog.age} years
- Size: ${sourceDog.size}
- Energy Level: ${sourceDog.energyLevel}
- Temperament: ${sourceDog.temperament.join(", ")}
- Description: ${sourceDog.description}

Candidate Dogs:
${candidates.map((dog, i) => `
${i + 1}. ${dog.name} (ID: ${dog.id})
   - Breed: ${dog.breed}
   - Age: ${dog.age} years
   - Size: ${dog.size}
   - Energy Level: ${dog.energyLevel}
   - Temperament: ${dog.temperament.join(", ")}
   - Description: ${dog.description}
`).join("\n")}

Return a JSON array with match scores and brief reasons (max 100 characters):

{
  "matches": [
    {
      "dogId": "uuid",
      "matchScore": 95,
      "reason": "Similar size, age, and energy. Both love fetch."
    }
  ]
}

Only include dogs with match score >= 60. Sort by match score descending.`;

const response = await openai.chat.completions.create({
  model: "gpt-4o-mini",
  messages: [
    { role: "system", content: "You are a dog behavior expert specializing in playdate compatibility." },
    { role: "user", content: prompt }
  ],
  response_format: { type: "json_object" },
  temperature: 0.3,
  max_tokens: 2000
});

const matches = JSON.parse(response.choices[0].message.content);
```

### 4.4 Fallback (Rule-based Matching)

If Azure OpenAI fails or times out:

```typescript
function ruleBasedMatching(sourceDog: Dog, candidates: Dog[]): Match[] {
  return candidates
    .map(candidate => {
      let score = 100;
      
      // Size compatibility (-30 if >1 size category difference)
      const sizeOrder = ["SMALL", "MEDIUM", "LARGE", "XLARGE"];
      const sizeDiff = Math.abs(
        sizeOrder.indexOf(sourceDog.size) - sizeOrder.indexOf(candidate.size)
      );
      if (sizeDiff > 1) score -= 30;
      
      // Age compatibility (-20 if age difference >3 years)
      const ageDiff = Math.abs(sourceDog.age - candidate.age);
      if (ageDiff > 3) score -= 20;
      
      // Energy level match (-20 if not matching)
      if (sourceDog.energyLevel !== candidate.energyLevel) score -= 20;
      
      // Temperament overlap (+10 for each shared trait)
      const sharedTraits = sourceDog.temperament.filter(t =>
        candidate.temperament.includes(t)
      );
      score += sharedTraits.length * 10;
      
      return {
        dogId: candidate.id,
        matchScore: Math.max(0, Math.min(100, score)),
        reason: "Rule-based match"
      };
    })
    .filter(match => match.matchScore >= 60)
    .sort((a, b) => b.matchScore - a.matchScore)
    .slice(0, 10);
}
```

### 4.5 Caching Strategy (Post-MVP)

- Cache common match results in Redis (TTL: 24 hours)
- Cache key: `matches:${sourceDogId}:${lastUpdated}`
- Invalidate on source dog profile update

---

## 5. Authentication & Authorization

### 5.1 Azure AD B2C Configuration

**Tenant**: `pawconnect.b2clogin.com`

**User Flows:**
1. **Sign-up/Sign-in (susi)**: Email/password + Google/Facebook/Apple OAuth
2. **Password Reset**: Email-based reset flow
3. **Profile Edit**: Update user attributes

**Custom Policies** (Post-MVP):
- Social account linking (merge Google + Facebook accounts)
- Passwordless authentication (magic links)

### 5.2 JWT Token Structure

```json
{
  "sub": "user-uuid-123",
  "email": "user@example.com",
  "name": "John Doe",
  "role": "OWNER",
  "iat": 1708256400,
  "exp": 1708342800,
  "iss": "https://pawconnect.b2clogin.com",
  "aud": "pawconnect-api"
}
```

**Token Expiration:**
- Access Token: 1 hour
- Refresh Token: 7 days

### 5.3 Authorization Middleware

```typescript
// middleware/auth.ts
import { Request, Response, NextFunction } from "express";
import jwt from "jsonwebtoken";

interface AuthRequest extends Request {
  user?: {
    id: string;
    email: string;
    role: string;
  };
}

export function authenticate(req: AuthRequest, res: Response, next: NextFunction) {
  const token = req.headers.authorization?.replace("Bearer ", "");
  
  if (!token) {
    return res.status(401).json({ error: "Unauthorized" });
  }
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET!) as any;
    req.user = {
      id: decoded.sub,
      email: decoded.email,
      role: decoded.role
    };
    next();
  } catch (error) {
    return res.status(401).json({ error: "Invalid token" });
  }
}

export function authorize(...roles: string[]) {
  return (req: AuthRequest, res: Response, next: NextFunction) => {
    if (!req.user || !roles.includes(req.user.role)) {
      return res.status(403).json({ error: "Forbidden" });
    }
    next();
  };
}

// Usage:
// app.patch("/dogs/:id", authenticate, authorize("OWNER", "ADMIN"), updateDog);
```

---

## 6. File Upload & Storage

### 6.1 Architecture

```
Client → API Server → Azure Blob Storage
                   ↓
             Content Safety API
              (moderation)
```

### 6.2 Photo Upload Flow

1. **Client**: User selects photo (React)
2. **Client**: Upload file to API endpoint `/dogs/:id/photos`
3. **API**: Validate file (size <5MB, type: image/jpeg, image/png, image/webp)
4. **API**: Call Azure Content Safety API (check for inappropriate content)
5. **API**: Upload to Azure Blob Storage (container: `dog-photos`)
6. **API**: Generate thumbnail (Sharp library, 400x400)
7. **API**: Save URL to PostgreSQL (`dog_photos` table)
8. **API**: Return photo URL with CDN prefix

### 6.3 Implementation

```typescript
// services/storage.ts
import { BlobServiceClient } from "@azure/storage-blob";
import sharp from "sharp";

const blobServiceClient = BlobServiceClient.fromConnectionString(
  process.env.AZURE_STORAGE_CONNECTION_STRING!
);

export async function uploadDogPhoto(file: Buffer, dogId: string): Promise<string> {
  const containerClient = blobServiceClient.getContainerClient("dog-photos");
  const filename = `${dogId}/${Date.now()}.jpg`;
  const blockBlobClient = containerClient.getBlockBlobClient(filename);
  
  // Resize and compress image
  const processedImage = await sharp(file)
    .resize(1200, 1200, { fit: "inside", withoutEnlargement: true })
    .jpeg({ quality: 85 })
    .toBuffer();
  
  // Upload to Azure Blob Storage
  await blockBlobClient.uploadData(processedImage, {
    blobHTTPHeaders: { blobContentType: "image/jpeg" }
  });
  
  // Generate thumbnail
  const thumbnail = await sharp(file)
    .resize(400, 400, { fit: "cover" })
    .jpeg({ quality: 80 })
    .toBuffer();
  
  const thumbnailFilename = `${dogId}/thumbnails/${Date.now()}.jpg`;
  const thumbnailClient = containerClient.getBlockBlobClient(thumbnailFilename);
  await thumbnailClient.uploadData(thumbnail, {
    blobHTTPHeaders: { blobContentType: "image/jpeg" }
  });
  
  // Return CDN URL
  const cdnUrl = `https://cdn.pawconnect.com/${filename}`;
  const thumbnailUrl = `https://cdn.pawconnect.com/${thumbnailFilename}`;
  
  return { url: cdnUrl, thumbnailUrl };
}
```

### 6.4 Content Moderation

```typescript
// services/moderation.ts
import { ContentSafetyClient } from "@azure-rest/ai-content-safety";

const client = new ContentSafetyClient(
  process.env.AZURE_CONTENT_SAFETY_ENDPOINT!,
  { key: process.env.AZURE_CONTENT_SAFETY_KEY! }
);

export async function moderateImage(imageBuffer: Buffer): Promise<boolean> {
  const response = await client.path("/image:analyze").post({
    body: {
      image: { content: imageBuffer.toString("base64") }
    }
  });
  
  if (response.status !== "200") {
    throw new Error("Content moderation failed");
  }
  
  const result = response.body;
  
  // Check for inappropriate content (violence, sexual, hate)
  const isInappropriate =
    result.violenceAnalysis?.severity > 2 ||
    result.sexualAnalysis?.severity > 2 ||
    result.hateAnalysis?.severity > 2;
  
  return !isInappropriate; // Return true if safe
}
```

---

## 7. Real-time Messaging

### 7.1 Architecture

```
Client (React) ↔ Azure SignalR ↔ API Server (Hub)
```

### 7.2 Azure SignalR Setup

```typescript
// server.ts
import * as signalR from "@microsoft/signalr";

const connection = new signalR.HubConnectionBuilder()
  .withUrl(process.env.AZURE_SIGNALR_CONNECTION_STRING!)
  .build();

connection.on("ReceiveMessage", (senderId, message) => {
  // Handle incoming message
  console.log(`${senderId}: ${message}`);
});

await connection.start();
```

### 7.3 Message Hub (API Server)

```typescript
// hubs/messageHub.ts
export class MessageHub {
  async sendMessage(senderId: string, recipientId: string, content: string) {
    // Save to database
    const message = await prisma.message.create({
      data: { senderId, recipientId, content, read: false }
    });
    
    // Send via SignalR
    await this.clients.user(recipientId).sendAsync("ReceiveMessage", {
      id: message.id,
      senderId,
      content,
      createdAt: message.createdAt
    });
  }
}
```

### 7.4 Client Integration (React)

```typescript
// hooks/useChat.ts
import { HubConnectionBuilder } from "@microsoft/signalr";
import { useEffect, useState } from "react";

export function useChat(userId: string) {
  const [messages, setMessages] = useState([]);
  const [connection, setConnection] = useState(null);
  
  useEffect(() => {
    const conn = new HubConnectionBuilder()
      .withUrl("/api/chat")
      .withAutomaticReconnect()
      .build();
    
    conn.on("ReceiveMessage", (message) => {
      setMessages((prev) => [...prev, message]);
    });
    
    conn.start().then(() => setConnection(conn));
    
    return () => conn.stop();
  }, []);
  
  const sendMessage = (recipientId: string, content: string) => {
    connection?.invoke("SendMessage", recipientId, content);
  };
  
  return { messages, sendMessage };
}
```

---

## 8. Payment Integration

### 8.1 Stripe Setup

**Account Type**: Stripe Connect (Platform)

**Flow**: Service provider bookings → Platform collects fee → Payout to provider

### 8.2 Booking Payment Flow

1. **Customer**: Submit booking request
2. **Provider**: Accept booking
3. **API**: Create Stripe Payment Intent
4. **Client**: Collect payment (Stripe Elements)
5. **API**: Confirm payment → Update booking status to CONFIRMED
6. **API**: Send confirmation email to both parties
7. **Post-Service**: Release funds to provider (Stripe Connect Transfer)

### 8.3 Implementation

```typescript
// services/stripe.ts
import Stripe from "stripe";

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: "2023-10-16"
});

export async function createPaymentIntent(
  bookingId: string,
  amount: number,
  providerId: string
): Promise<string> {
  const paymentIntent = await stripe.paymentIntents.create({
    amount: amount * 100, // Convert to cents
    currency: "usd",
    metadata: { bookingId, providerId },
    application_fee_amount: Math.round(amount * 100 * 0.1), // 10% platform fee
    transfer_data: {
      destination: providerId // Provider's Stripe Connect account ID
    }
  });
  
  return paymentIntent.client_secret!;
}

export async function handleWebhook(event: Stripe.Event) {
  switch (event.type) {
    case "payment_intent.succeeded":
      const paymentIntent = event.data.object as Stripe.PaymentIntent;
      const bookingId = paymentIntent.metadata.bookingId;
      
      // Update booking status
      await prisma.booking.update({
        where: { id: bookingId },
        data: {
          status: "CONFIRMED",
          stripePaymentId: paymentIntent.id
        }
      });
      
      // Send confirmation email
      await sendBookingConfirmationEmail(bookingId);
      break;
    
    case "payment_intent.payment_failed":
      // Handle payment failure
      break;
  }
}
```

---

## 9. Email Service

### 9.1 SendGrid Templates

| Template | Purpose | Trigger |
|----------|---------|---------|
| `welcome` | Welcome new user | User registration |
| `email-verification` | Verify email address | User registration |
| `playdate-request` | Notify recipient of playdate request | Playdate requested |
| `playdate-accepted` | Notify requester of acceptance | Playdate accepted |
| `event-reminder` | Remind user of upcoming event | 24h before event |
| `booking-confirmation` | Confirm service booking | Booking confirmed |
| `booking-reminder` | Remind user of upcoming booking | 24h before booking |
| `password-reset` | Password reset link | Password reset request |

### 9.2 Implementation

```typescript
// services/email.ts
import sgMail from "@sendgrid/mail";

sgMail.setApiKey(process.env.SENDGRID_API_KEY!);

export async function sendPlaydateRequest(playdate: Playdate) {
  const recipientUser = await prisma.user.findUnique({
    where: { id: playdate.recipientDog.userId }
  });
  
  await sgMail.send({
    to: recipientUser.email,
    from: "notifications@pawconnect.com",
    templateId: "d-playdate-request",
    dynamicTemplateData: {
      recipientName: recipientUser.name,
      requesterDogName: playdate.requesterDog.name,
      proposedDate: playdate.proposedDate,
      location: playdate.location,
      message: playdate.message,
      acceptUrl: `https://pawconnect.com/playdates/${playdate.id}/accept`
    }
  });
}
```

---

## 10. Deployment Guide

### 10.1 Azure Resources (Production)

**Resource Group**: `pawconnect-prod`

| Resource | SKU | Purpose |
|----------|-----|---------|
| Azure App Service Plan | S1 (Standard) | Host API + Next.js |
| Azure App Service (API) | Linux, Node 20 | Express.js API |
| Azure App Service (Frontend) | Linux, Node 20 | Next.js SSR |
| Azure Database for PostgreSQL | B2s (2 vCore, 4GB) | Primary database |
| Azure Blob Storage | General Purpose v2 | Photo storage |
| Azure CDN | Standard Microsoft | Static assets + images |
| Azure SignalR Service | Free (20 connections) | Real-time messaging |
| Azure OpenAI | GPT-4o-mini | AI matching |
| Azure AD B2C | Free (50K MAU) | Authentication |
| Azure Monitor | Pay-as-you-go | Logging + monitoring |

### 10.2 Environment Variables

```bash
# Database
DATABASE_URL=postgresql://user:pass@pawconnect-db.postgres.database.azure.com:5432/pawconnect

# Azure
AZURE_STORAGE_CONNECTION_STRING=DefaultEndpointsProtocol=https;...
AZURE_CDN_ENDPOINT=https://cdn.pawconnect.com
AZURE_OPENAI_API_KEY=sk-...
AZURE_OPENAI_ENDPOINT=https://pawconnect-openai.openai.azure.com
AZURE_SIGNALR_CONNECTION_STRING=Endpoint=https://...
AZURE_AD_B2C_CLIENT_ID=...
AZURE_AD_B2C_CLIENT_SECRET=...
AZURE_CONTENT_SAFETY_ENDPOINT=https://...
AZURE_CONTENT_SAFETY_KEY=...

# External Services
STRIPE_SECRET_KEY=sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
SENDGRID_API_KEY=SG....
GOOGLE_MAPS_API_KEY=AIza...

# App
JWT_SECRET=random-secret-key
NODE_ENV=production
API_BASE_URL=https://api.pawconnect.com
FRONTEND_URL=https://pawconnect.com
```

### 10.3 Deployment Steps

1. **Provision Azure Resources** (manual or Terraform)
2. **Configure Azure AD B2C** (create tenant, user flows)
3. **Set up PostgreSQL**:
   ```bash
   psql -h pawconnect-db.postgres.database.azure.com -U admin -d pawconnect
   CREATE EXTENSION postgis;
   CREATE EXTENSION pg_trgm;
   ```
4. **Run Prisma Migrations**:
   ```bash
   npx prisma migrate deploy
   ```
5. **Configure GitHub Secrets** (for Actions):
   - `AZURE_WEBAPP_PUBLISH_PROFILE_API`
   - `AZURE_WEBAPP_PUBLISH_PROFILE_FRONTEND`
   - All environment variables
6. **Deploy via GitHub Actions** (push to `main` branch)

### 10.4 GitHub Actions Workflow

```yaml
# .github/workflows/deploy.yml
name: Deploy to Azure

on:
  push:
    branches: [main]

jobs:
  deploy-api:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "20"
      
      - name: Install dependencies
        run: npm ci
        working-directory: ./api
      
      - name: Run tests
        run: npm test
        working-directory: ./api
      
      - name: Build
        run: npm run build
        working-directory: ./api
      
      - name: Deploy to Azure App Service
        uses: azure/webapps-deploy@v2
        with:
          app-name: pawconnect-api
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE_API }}
          package: ./api
  
  deploy-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "20"
      
      - name: Install dependencies
        run: npm ci
        working-directory: ./frontend
      
      - name: Build Next.js
        run: npm run build
        working-directory: ./frontend
        env:
          NEXT_PUBLIC_API_URL: ${{ secrets.API_BASE_URL }}
      
      - name: Deploy to Azure App Service
        uses: azure/webapps-deploy@v2
        with:
          app-name: pawconnect-frontend
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE_FRONTEND }}
          package: ./frontend
```

---

## 11. Monitoring & Observability

### 11.1 Azure Monitor Configuration

**Metrics to Track:**
- **Application Performance**:
  - Request rate, response time (p50, p95, p99)
  - Error rate (4xx, 5xx)
  - Database query latency
  - AI matching latency
- **Business Metrics**:
  - User signups, active users (DAU, MAU)
  - Playdates requested/accepted
  - Bookings created/confirmed
  - Events created/RSVPd
- **Infrastructure**:
  - CPU, memory utilization
  - Database connections, queries/sec
  - Blob storage requests
  - CDN bandwidth

**Alerts:**
- Error rate >5% for 5 minutes
- API response time (p95) >2 seconds for 5 minutes
- Database connection pool >80% for 5 minutes
- AI matching failure rate >10%

### 11.2 Application Insights Integration

```typescript
// server.ts
import * as appInsights from "applicationinsights";

appInsights.setup(process.env.APPLICATIONINSIGHTS_CONNECTION_STRING)
  .setAutoDependencyCorrelation(true)
  .setAutoCollectRequests(true)
  .setAutoCollectPerformance(true)
  .setAutoCollectExceptions(true)
  .setAutoCollectDependencies(true)
  .start();

const client = appInsights.defaultClient;

// Track custom events
export function trackPlaydateRequested(playdateId: string) {
  client.trackEvent({
    name: "PlaydateRequested",
    properties: { playdateId }
  });
}

export function trackAIMatchingLatency(durationMs: number, success: boolean) {
  client.trackMetric({
    name: "AIMatchingLatency",
    value: durationMs
  });
  client.trackEvent({
    name: "AIMatchingRequest",
    properties: { success }
  });
}
```

### 11.3 Logging Strategy

**Log Levels:**
- **ERROR**: Application errors, exceptions, failed API calls
- **WARN**: Degraded performance, fallback triggered
- **INFO**: Business events (user signup, playdate created)
- **DEBUG**: Detailed request/response (disabled in production)

**Structured Logging:**
```typescript
import winston from "winston";

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || "info",
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: "error.log", level: "error" }),
    new winston.transports.File({ filename: "combined.log" })
  ]
});

// Usage
logger.info("Playdate requested", {
  userId: "user-uuid-123",
  playdateId: "playdate-uuid-456",
  recipientDogId: "dog-uuid-789"
});
```

---

## 12. Testing Strategy

### 12.1 Test Pyramid

```
                 ┌──────────┐
                 │   E2E    │  10% (Playwright)
                 │  Tests   │
                 └──────────┘
             ┌────────────────┐
             │  Integration   │  30% (Jest + Supertest)
             │     Tests      │
             └────────────────┘
         ┌──────────────────────┐
         │     Unit Tests       │  60% (Jest)
         └──────────────────────┘
```

### 12.2 Unit Tests

**Coverage Target**: 80% (functions, branches)

**Examples:**
```typescript
// tests/services/matching.test.ts
describe("AI Matching Service", () => {
  it("should return top 5 matches sorted by score", async () => {
    const sourceDog = mockDog({ id: "1", breed: "Golden Retriever" });
    const candidates = [
      mockDog({ id: "2", breed: "Labrador" }),
      mockDog({ id: "3", breed: "Poodle" })
    ];
    
    const matches = await matchingService.getMatches(sourceDog, candidates);
    
    expect(matches).toHaveLength(2);
    expect(matches[0].matchScore).toBeGreaterThanOrEqual(matches[1].matchScore);
  });
  
  it("should fallback to rule-based matching if AI fails", async () => {
    jest.spyOn(openai, "chat.completions.create").mockRejectedValue(new Error("API error"));
    
    const matches = await matchingService.getMatches(sourceDog, candidates);
    
    expect(matches).toBeDefined();
    expect(matches.length).toBeGreaterThan(0);
  });
});
```

### 12.3 Integration Tests

**Scope**: API endpoints, database interactions

```typescript
// tests/api/playdates.test.ts
describe("POST /v1/playdates", () => {
  it("should create a playdate request", async () => {
    const response = await request(app)
      .post("/v1/playdates")
      .set("Authorization", `Bearer ${validToken}`)
      .send({
        requesterDogId: "dog-1",
        recipientDogId: "dog-2",
        proposedDate: "2026-03-01",
        proposedTime: "2:00 PM - 4:00 PM",
        location: "Central Park"
      });
    
    expect(response.status).toBe(201);
    expect(response.body).toMatchObject({
      status: "PENDING",
      requesterDog: { id: "dog-1" }
    });
    
    // Verify database record
    const playdate = await prisma.playdate.findUnique({
      where: { id: response.body.id }
    });
    expect(playdate).toBeDefined();
  });
});
```

### 12.4 E2E Tests (Playwright)

**Critical User Flows:**
1. User registration → dog profile creation → playdate request
2. Service provider search → booking request → payment
3. Event creation → RSVP → attendance

```typescript
// tests/e2e/playdate-flow.spec.ts
import { test, expect } from "@playwright/test";

test("complete playdate flow", async ({ page }) => {
  // Login
  await page.goto("/login");
  await page.fill("#email", "user@example.com");
  await page.fill("#password", "password123");
  await page.click("button[type=submit]");
  
  // Navigate to playdate matching
  await page.click("text=Find Playdates");
  await expect(page).toHaveURL("/playdates");
  
  // View match suggestions
  await expect(page.locator(".match-card")).toHaveCount(5);
  
  // Request playdate
  await page.click(".match-card:first-child button:has-text('Request Playdate')");
  await page.fill("#date", "2026-03-01");
  await page.fill("#time", "2:00 PM - 4:00 PM");
  await page.fill("#location", "Central Park");
  await page.click("button:has-text('Send Request')");
  
  // Verify success
  await expect(page.locator("text=Playdate request sent")).toBeVisible();
});
```

### 12.5 Performance Tests

**Tools**: k6, Artillery

**Scenarios:**
- **Load Test**: 500 concurrent users, 10-minute duration
- **Spike Test**: 0 → 1000 users in 1 minute
- **Soak Test**: 200 users, 1-hour duration

```javascript
// tests/performance/load.js (k6)
import http from "k6/http";
import { check } from "k6";

export let options = {
  vus: 500, // Virtual users
  duration: "10m"
};

export default function () {
  const res = http.get("https://api.pawconnect.com/v1/dogs", {
    headers: { Authorization: `Bearer ${__ENV.JWT_TOKEN}` }
  });
  
  check(res, {
    "status is 200": (r) => r.status === 200,
    "response time < 500ms": (r) => r.timings.duration < 500
  });
}
```

---

## 13. Security Checklist

- [x] **Authentication**: Azure AD B2C with MFA support
- [x] **Authorization**: RBAC (owner, provider, admin)
- [x] **Data Encryption**: TLS 1.3 in transit, AES-256 at rest
- [x] **Password Hashing**: bcrypt (salt rounds: 12)
- [x] **SQL Injection**: Prisma ORM (parameterized queries)
- [x] **XSS Protection**: React auto-escaping, CSP headers
- [x] **CSRF Protection**: SameSite cookies, CSRF tokens
- [x] **Rate Limiting**: Express rate-limit (100 req/min per IP)
- [x] **Content Moderation**: Azure Content Safety API
- [x] **GDPR Compliance**: Data export, deletion, consent
- [x] **PCI-DSS**: Stripe handles payment data (SAQ-A)
- [x] **Secrets Management**: Azure Key Vault (post-MVP)
- [x] **Dependency Scanning**: Dependabot, npm audit
- [x] **Vulnerability Scanning**: Azure Security Center

---

## Appendix A: Technology Decision Matrix

| Category | Chosen | Alternatives Considered | Reason |
|----------|--------|-------------------------|--------|
| Frontend Framework | Next.js | CRA, Vue/Nuxt | SSR for SEO, performance, team expertise |
| Backend Framework | Express.js | FastAPI, .NET Core | JavaScript consistency, team familiarity |
| Database | PostgreSQL | MongoDB, MySQL | Relational integrity, PostGIS, ACID |
| AI/ML | Azure OpenAI | Custom ML, AWS Bedrock | Fast MVP, no training data, Azure alignment |
| Auth | Azure AD B2C | Auth0, Firebase Auth | GDPR compliance, Azure strategy |
| Payments | Stripe | PayPal, Square | Developer experience, marketplace features |
| Storage | Azure Blob | AWS S3, Cloudinary | Azure strategy, cost-effective |
| CDN | Azure CDN | Cloudflare, AWS CloudFront | Azure integration |
| Messaging | Azure SignalR | Socket.io, Pusher | Managed service, auto-scaling |
| Email | SendGrid | AWS SES, Postmark | Deliverability, template management |

---

**Last Updated**: 2026-02-18  
**Version**: 1.0  
**Status**: Draft - Pending Approval

**Next Steps**:
1. Engineering Lead review (database schema, API design)
2. AI/ML Specialist review (matching algorithm, Azure OpenAI setup)
3. Security Lead review (authentication, data protection)
4. DevOps review (deployment pipeline, monitoring)

---

**Author**: AgentX Solution Architect  
**Contributors**: Product Manager, UX Designer  
**Reviewers**: TBD
