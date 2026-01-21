---
title: Freebo - Full-Stack Freelancing Marketplace
description: A modern freelancing platform built with React, TypeScript, and Node.js featuring real-time messaging, order management, and a polished UI.
date: 2024-12-20 12:00:00
label: Full-Stack / TypeScript
image: '/TimBunkerPortfolio/images/freebo-cover.png'
---

# Freebo: Freelancing Marketplace

## Overview

A full-stack web application for connecting freelancers with clients. Built from the ground up with modern technologies and a focus on clean architecture and user experience.

---

## Features

### For Freelancers
- **Service Listings**: Create detailed service offerings with images, pricing, and delivery terms
- **Order Management**: Track orders through states (pending, in progress, delivered, completed)
- **Messaging**: Real-time communication with potential and current clients
- **Profile Management**: Showcase skills, portfolio, and reviews

### For Clients
- **Discovery**: Search and filter services by category, price, and ratings
- **Feed**: Browse listings, text posts, and events from the community
- **Order Tracking**: Monitor order progress and communicate with freelancers
- **Reviews**: Rate and review completed services

---

## Technical Architecture

### Frontend

**Stack**: React 18 + TypeScript + Vite + Material-UI

```
client/src/
├── components/
│   ├── atoms/         # Basic UI elements
│   ├── molecules/     # Composite components
│   ├── organisms/     # Complex features
│   └── modals/        # Modal dialogs
├── pages/
│   ├── AuthPage       # Login/registration
│   ├── SearchPage     # Service discovery
│   ├── ProfilePage    # User profiles
│   ├── MessagesPage   # Conversations
│   ├── OrdersPage     # Order management
│   └── CreatePage     # Content creation
├── hooks/             # Custom React hooks
├── store/             # Redux state management
├── api/               # API client functions
└── types/             # TypeScript definitions
```

**Key Frontend Features**:
- **Atomic Design**: Components organized by complexity
- **Type Safety**: Full TypeScript coverage
- **State Management**: Redux Toolkit for global state
- **Custom Hooks**: `useListings`, `usePosts`, etc. for data fetching
- **Responsive Design**: Mobile-first Material-UI theming

### Backend

**Stack**: Node.js + Express + Prisma + PostgreSQL

```
server/src/
├── routes/
│   ├── auth.ts         # JWT authentication
│   ├── users.ts        # User CRUD
│   ├── posts.ts        # Feed content
│   ├── listings.ts     # Service listings
│   ├── orders.ts       # Order lifecycle
│   └── messages.ts     # Conversations
├── middleware/
│   └── auth.ts         # JWT verification
├── services/           # Business logic
└── types/              # Shared types
```

**Key Backend Features**:
- **RESTful API**: Clean endpoint design
- **JWT Authentication**: Secure token-based auth
- **Prisma ORM**: Type-safe database queries
- **File Uploads**: Image handling for listings and profiles

---

## Data Model

```
User
├── id, email, username, avatarUrl
├── bio, skills[], isVerified
└── listings[], orders[], messages[]

Post (polymorphic)
├── TEXT: title, content, imageUrls[]
├── EVENT: date, time, location, rsvpCount
└── LISTING: price, category, deliveryDays

Order
├── id, status, price, requirements
├── buyer (User), seller (User)
└── listing (Post), messages[]

Conversation
├── participants (User[])
└── messages[]
```

---

## UI Components

### Feed System
The app supports multiple post types with polymorphic rendering:

- **TextPostCard**: Community discussions and updates
- **EventPostCard**: Local meetups and events with RSVP
- **ServiceCard**: Freelance service offerings with pricing
- **FeedPost**: Generic content with images

### Modal System
Global modals managed through Redux:
- `ListingDetailModal`: Full service details
- `CreateListingModal`: Service creation flow
- `CommentModal`: Discussions on any post
- `SendMessageModal`: Direct messaging
- `EditProfileModal`: Profile updates

---

## Key Implementation Details

### Polymorphic Post Rendering

```typescript
const PostCard: React.FC<{ post: PostPreview }> = ({ post }) => {
  switch (post.type) {
    case 'TEXT':
      return <TextPostCard {...textProps} />;
    case 'EVENT':
      return <EventPostCard {...eventProps} />;
    case 'LISTING':
      return <ServiceCard {...listingProps} />;
    default:
      return <FeedPost {...defaultProps} />;
  }
};
```

### Feed Filtering

```typescript
const FeedFilterPills: React.FC = () => {
  const dispatch = useAppDispatch();
  const activeFilters = useAppSelector(state => state.ui.feedFilterTypes);
  
  return (
    <Stack direction="row" spacing={1}>
      {['TEXT', 'EVENT', 'LISTING'].map(type => (
        <Chip
          key={type}
          selected={activeFilters.includes(type)}
          onClick={() => dispatch(toggleFeedFilter(type))}
        />
      ))}
    </Stack>
  );
};
```

---

## What I Learned

- **Full-Stack TypeScript**: Sharing types between frontend and backend
- **Prisma ORM**: Schema design, migrations, and efficient queries
- **Material-UI Theming**: Custom design systems with MUI
- **State Architecture**: When to use local vs. global state
- **API Design**: RESTful patterns for complex resources
- **Authentication Flows**: JWT tokens, refresh strategies, protected routes

---

## Status

Currently in active development. Core features implemented:
- ✅ User authentication
- ✅ Service listings CRUD
- ✅ Feed with multiple post types
- ✅ Messaging system
- ✅ Order management
- ✅ User profiles

Coming soon:
- 🔄 Real-time notifications
- 🔄 Payment integration
- 🔄 Review system
- 🔄 Search optimization
