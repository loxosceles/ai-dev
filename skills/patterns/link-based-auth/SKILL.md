---
name: link-based-auth
description: Personalized links without explicit login for authentication. Apply when implementing authentication flows that don't require traditional login.
type: pattern
---

# Authentication Patterns

**This is a reference pattern.** Learn from the approach, adapt to your context — don't copy verbatim.

## Link-Based Authentication

**Pattern**: Provide personalized experiences without requiring explicit login

**Use Case**: Portfolio sites, demo applications, or any scenario where you want to provide personalized content without friction of traditional authentication.

### Core Concept

Use personalized links containing tokens that authenticate visitors automatically. Each link is associated with a virtual user identity, allowing for secure, personalized content delivery without login screens.

### Components

1. **Link Generator**
   - Creates personalized URLs for specific visitors
   - Associates each link with visitor metadata (company, role, etc.)
   - Stores link-visitor associations in database

2. **Virtual User Identities**
   - Each personalized link corresponds to a virtual user in identity provider (e.g., AWS Cognito)
   - These users exist solely to provide authentication context
   - No password or explicit login required

3. **Edge Authentication Function**
   - Intercepts requests at CDN edge (e.g., Lambda@Edge, CloudFront Functions)
   - Validates tokens from URL parameters
   - Sets authentication cookies for browser session

4. **API Client**
   - Uses centralized auth context for all API requests
   - Provides personalized data based on visitor's identity
   - Environment-aware authentication (API key vs tokens)

## Authentication Flow

1. **Link Creation**
   - Admin creates a personalized link for a visitor
   - System creates a virtual user identity and generates access tokens
   - Link with embedded token is shared with visitor

2. **Visitor Access**
   - Visitor clicks the personalized link
   - CDN receives request with token parameter
   - Edge function validates token and sets authentication cookies

3. **Frontend Authentication**
   - Frontend loads with authentication cookies already set
   - API client extracts tokens from cookies
   - API requests include token in Authorization header

4. **API Authorization**
   - AppSync API validates the token against Cognito
   - Resolvers use the visitor's identity to return personalized content
   - Access control ensures visitors only see content intended for them

## Security Considerations

- **Tokenized Links**: Each URL contains a secure token tied to a specific virtual Cognito user
- **Content Isolation**: Visitors can only access personalized content intended for them
- **Tamper-Proof Design**: URL manipulation is ineffective as authentication is tied to backend Cognito identities
- **Cookie Security**: Authentication cookies use SameSite=Strict to prevent CSRF attacks
- **Content Security Policy**: Headers mitigate XSS risks
- **Short Token Expiration**: Tokens have limited validity periods
- **Client-Side Authentication**: Tokens are processed in the browser for static site compatibility
- **Security Scope**: Designed for tamper-proofing personalized content, not for highly sensitive data

## Direct URL Access

When a visitor accesses the base URL without a personalized link:

- No authentication tokens are available
- The Apollo client cannot make authenticated requests
- Only public content is visible
- Personalized features are not accessible

This is by design - the system expects visitors to use personalized links for the full experience.

## Technical Implementation

The authentication flow is implemented across several components. The system is designed to work in both deployed environments (with real authentication) and local development (with mock data):

### Architecture Overview

```
┌─────────────────────────────────────┐
│           Auth Context              │
│  - Centralized token management     │
│  - Environment detection            │
│  - Auth header generation           │
└─────────────────┬───────────────────┘
                  │
    ┌─────────────┼─────────────┐
    │             │             │
┌───▼───┐    ┌───▼───┐    ┌───▼───┐
│Local  │    │Advocate│    │AI     │
│Inter- │    │Greeting│    │Advo-  │
│ceptor │    │Hooks   │    │cate   │
│(Mock) │    │        │    │Hooks  │
└───────┘    └───────┘    └───────┘
```

### 1. **Centralized Auth Context**:

```typescript
// Single source of truth for authentication state
export function useAuth() {
  return {
    isAuthenticated,
    environment,
    tokens,
    visitorParam,
    getAuthHeaders: (routeType) => ({
      /* headers */
    }),
    getQueryContext: (routeType) => ({ headers: getAuthHeaders(routeType) })
  };
}
```

### 2. **Environment-Aware Authentication**:

```typescript
// Different auth strategies per environment
const getAuthHeaders = (routeType: RouteType) => {
  if (environment === 'local') {
    return { 'x-api-key': process.env.NEXT_PUBLIC_APPSYNC_API_KEY };
  }

  if (routeType === 'public') {
    return { 'x-api-key': process.env.NEXT_PUBLIC_APPSYNC_API_KEY };
  }

  if (tokens.accessToken) {
    return { Authorization: `Bearer ${tokens.accessToken}` };
  }
};
```

### 3. **Local Development Interceptor**:

```typescript
// Generic interceptor for local development
export function useLocalRequestInterceptor() {
  const { environment } = useAuth();
  const searchParams = useSearchParams();
  const visitorParam = searchParams?.get('visitor');

  // Should intercept if in local environment with visitor parameter
  const shouldIntercept = environment === 'local' && !!visitorParam;

  return {
    shouldIntercept,
    getAdvocateGreetingMock: () => ({
      linkId: 'local-interceptor',
      companyName: 'Local Demo Company',
      recruiterName: `Local Visitor ${visitorParam?.substring(0, 6)}`,
      context: 'local development',
      greeting: 'Welcome to local development!',
      message: 'This is mock data from the local request interceptor.',
      skills: ['React', 'TypeScript', 'AWS', 'Node.js', 'GraphQL']
    }),
    getAIAdvocateMock: (question) => ({
      answer: 'This is a simulated AI response for local development.',
      context: 'Local interceptor response'
    })
  };
}
```

### 4. **Hook-Based Data Fetching**:

```typescript
// All data fetching uses centralized auth
export function useAdvocateGreeting() {
  const { isAuthenticated, getQueryContext } = useAuth();

  const { data, loading, error } = useQuery(GET_ADVOCATE_GREETING, {
    skip: !isAuthenticated,
    fetchPolicy: 'cache-and-network',
    context: getQueryContext('protected')
  });

  return {
    greetingData: data?.getAdvocateGreeting,
    isLoading: loading,
    error: error?.message || null,
    isAuthenticated
  };
}
```

### 5. **Environment Management**:

- **Local**: `NEXT_PUBLIC_ENVIRONMENT=local` (set in dev script)
- **Deployed Dev**: `NEXT_PUBLIC_ENVIRONMENT=dev` (set in deploy script)
- **Production**: `NEXT_PUBLIC_ENVIRONMENT=prod` (set in deploy script)

### 6. **Component Integration**:

```typescript
// Example of how components integrate with the auth system
function AdvocateGreetingComponent() {
  // Check for local interception first
  const interceptor = useLocalRequestInterceptor();
  const { isAuthenticated } = useAuth();

  // Use the standard query if authenticated
  const { greetingData: realGreetingData, isLoading } = useAdvocateGreeting();

  // Use interceptor data if available, otherwise use real data
  const greetingData = interceptor.shouldIntercept
    ? interceptor.getAdvocateGreetingMock()
    : realGreetingData;

  // Component rendering logic...
}
```

This authentication architecture provides a seamless, secure experience for visitors while maintaining strict access control and personalization capabilities. The local development mode allows for rapid UI development without needing real authentication tokens.

## Related Documentation

- [Frontend Architecture](frontend.md) - How the frontend integrates with the authentication system
- [Infrastructure Architecture](infrastructure/overview.md) - How the infrastructure supports the authentication system
- [Local Development Guide](../guides/local-development.md) - How to use the local development mode
