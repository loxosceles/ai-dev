---
name: component-organization
description: Frontend directory structure and file organization patterns. Apply when creating new components or restructuring frontend code.
type: pattern
---

# Frontend Architecture Patterns

**This is a reference pattern.** Learn from the approach, adapt to your context — don't copy verbatim.

Patterns for organizing frontend applications in serverless architectures.

## Technology Stack Considerations

Common choices for serverless frontends:

- **Framework**: Next.js, Nuxt, SvelteKit, or static site generators
- **Language**: TypeScript for type safety
- **Styling**: Tailwind CSS, CSS Modules, or styled-components
- **Data Fetching**: GraphQL clients (Apollo, urql), REST clients (fetch, axios)
- **Authentication**: Auth context with identity provider integration

## Directory Structure Pattern

**Principle**: Organize by feature or domain, not by file type

```
frontend/
├── app/                 # Next.js app directory
│   ├── globals.css      # Global styles
│   ├── layout.tsx       # Root layout
│   └── page.tsx         # Home page
├── components/          # Reusable UI components
│   ├── ui/              # Base UI components
│   ├── advocate-greeting-modal.tsx
│   ├── ai-question.tsx
│   ├── footer.tsx
│   ├── header.tsx
│   ├── hero-section.tsx
│   └── ...
├── lib/                 # Utility functions and hooks
│   ├── advocate-greeting/
│   ├── ai-advocate/
│   ├── apollo/
│   ├── auth/
│   └── local/
├── public/              # Static assets
├── queries/             # GraphQL queries
│   ├── advocate-greeting.ts
│   └── developers.ts
└── shared/              # Shared types and constants
    ├── constants.ts
    └── types.ts
```

## Key Components

### Authentication

The authentication system is implemented in the `lib/auth` directory:

- **auth-context.tsx**: Provides authentication context to the application
- **cookie-auth.ts**: Handles cookie-based authentication
- **auth-utils.ts**: Utility functions for authentication

### AI Advocate

The AI advocate functionality is implemented in the `lib/ai-advocate` directory:

- **use-ai-advocate.ts**: Hook for interacting with the AI advocate
- **use-ai-advocate-dev.ts**: Development version of the hook

### Advocate Greeting

The advocate greeting functionality is implemented in the `lib/advocate-greeting` directory:

- **advocate-greeting-service.ts**: Service for fetching advocate greetings
- **use-advocate-greeting.ts**: Hook for using advocate greetings
- **use-advocate-greeting-dev.ts**: Development version of the hook

### Local Development

The local development functionality is implemented in the `lib/local` directory:

- **use-local-request-interceptor.ts**: Hook for intercepting requests in local development

## Data Flow

1. **Authentication**: The auth context provides authentication state to the application
2. **Data Fetching**: Components use hooks to fetch data from the GraphQL API
3. **Rendering**: Components render based on the fetched data
4. **User Interaction**: User interactions trigger data fetching and state updates

## Authentication Flow

The frontend authentication flow is implemented as follows:

1. **Token Extraction**: The auth context extracts tokens from cookies
2. **Authentication State**: The auth context provides authentication state to components
3. **Request Headers**: The auth context provides headers for authenticated requests
4. **Local Development**: In local development, the auth context uses a visitor parameter

For more details on the authentication flow, see the [Authentication Flow](../auth-flow.md) document.

## Local Development

The frontend supports local development with mock data:

1. **Environment Detection**: The auth context detects the local environment
2. **Request Interception**: The local request interceptor intercepts requests
3. **Mock Data**: The interceptor provides mock data for advocate greetings and AI advocate

## Deployment

The frontend is built as a static site and deployed to S3 with CloudFront:

1. **Build**: The Next.js application is built with `next build`
2. **Export**: The built application is exported as static files
3. **Deployment**: The static files are uploaded to S3
4. **Distribution**: CloudFront distributes the static files

For more details on the deployment process, see the [Deployment Guide](../deployment.md) document.
