# UI Security & Anti-Patterns Guide

**IMPORTANT: Security must be a core consideration in all UI development. This guide outlines critical security patterns to prevent vulnerabilities in React/Next.js components.**

## Table of Contents
1. [XSS Prevention - dangerouslySetInnerHTML](#1-xss-prevention---dangerouslysetinnerhtml)
2. [URL Handling & Navigation](#2-url-handling--navigation)
3. [External Link Security](#3-external-link-security)
4. [Image Handling](#4-image-handling)
5. [Client-Side Storage Security](#5-client-side-storage-security)
6. [Event Handlers](#6-event-handlers)
7. [Dynamic Class Names](#7-dynamic-class-names)
8. [RegEx & Search Security (ReDoS)](#8-regex--search-security-redos)
9. [React's Automatic XSS Protection](#9-reacts-automatic-xss-protection)
10. [URL State Synchronization Security](#10-url-state-synchronization-security)
11. [Authentication & Session Management](#11-authentication--session-management)
12. [SSR/SSG Security & State Hydration](#12-ssrssg-security--state-hydration)
13. [CSRF Protection](#13-csrf-protection)
14. [Audit Logging & Monitoring](#14-audit-logging--monitoring)
15. [Third-Party Library Security](#15-third-party-library-security)
- [UI Security Checklist](#ui-security-checklist)

---

## 1. XSS Prevention - dangerouslySetInnerHTML

**The Golden Rule: Never trust user-generated or external content when rendering HTML.**

### ❌ FORBIDDEN:
```tsx
// DANGEROUS - Critical XSS vulnerability
<div dangerouslySetInnerHTML={{ __html: content }} />
```

### ✅ REQUIRED - Sanitize with DOMPurify:
```tsx
import DOMPurify from 'dompurify';

const sanitized = DOMPurify.sanitize(content, {
  ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'h2', 'h3', 'ul', 'ol', 'li', 'a'],
  ALLOWED_ATTR: ['href', 'target', 'rel', 'class'],
});
<div dangerouslySetInnerHTML={{ __html: sanitized }} />
```

### When DOMPurify is REQUIRED:
- Content from CMS or external APIs
- User-generated content
- Markdown converted to HTML
- Rich text editor output

### When safe WITHOUT DOMPurify:
- Static, hardcoded HTML in codebase
- Trusted mockData (development only)

**Installation:** `npm install dompurify @types/dompurify`

**❌ NEVER use `eval()` or `new Function()` with user input**

---

## 2. URL Handling & Navigation

### ❌ FORBIDDEN:
```tsx
// DANGEROUS - XSS and open redirect vulnerabilities
<Link href={`/search?q=${query}`}>Search</Link>
router.push(userInput);
```

### ✅ REQUIRED - Encode parameters:
```tsx
// Option 1: Encode manually
<Link href={`/search?q=${encodeURIComponent(query)}`}>Search</Link>

// Option 2: Use Next.js router params (preferred)
router.push({ pathname: '/search', query: { q: query } });

// Option 3: Use URL API for complex URLs
const url = new URL('/search', window.location.origin);
url.searchParams.set('q', query);
<Link href={url.pathname + url.search}>Search</Link>
```

### ✅ REQUIRED - Validate redirects:
```tsx
const allowedPaths = ['/products', '/cart', '/checkout'];
const isAllowed = allowedPaths.some(path => userInput.startsWith(path));
if (!isAllowed) return null; // Block unsafe redirects
```

**❌ NEVER use `window.location.href = userInput` without validation**

---

## 3. External Link Security

### ❌ FORBIDDEN:
```tsx
<a href={externalUrl} target="_blank">External</a>
```

### ✅ REQUIRED - Always include rel="noopener noreferrer":
```tsx
<a href={externalUrl} target="_blank" rel="noopener noreferrer">External</a>
<Link href="https://example.com" target="_blank" rel="noopener noreferrer">Link</Link>
```

**Why:** Without `noopener`, external sites can access `window.opener` and redirect your page to phishing sites (tabnabbing attack).

---

## 4. Image Handling

**❌ NEVER use `<img src={userInput} />` - Potential XSS via onerror attribute**

### ✅ ALWAYS use Next.js Image component:
```tsx
import Image from 'next/image';
<Image src={url} alt="..." width={500} height={300} />
```

Configure remote domains in `next.config.mjs`:
```javascript
images: {
  remotePatterns: [{ protocol: 'https', hostname: 'trusted-domain.com' }]
}
```

---

## 5. Client-Side Storage Security

### NEVER store in localStorage/sessionStorage:
- Authentication tokens (use httpOnly cookies)
- API secrets or keys
- Passwords (even hashed)
- Credit card information, SSN, or PII
- Session IDs
- Cart contents with customer/order data
- User profile information
- Internal user/customer IDs

**Why:** localStorage is accessible via JavaScript and vulnerable to XSS attacks. If an attacker injects a script, they can read everything in localStorage.

### SAFE to store:
- UI preferences (theme, language, dark mode)
- Non-sensitive settings (items per page, sort order, grid/list view)
- Site template preferences
- Non-sensitive filter states

### REQUIRED - Sanitize before storing/retrieving:
```tsx
// ❌ Bad - storing unsanitized data
localStorage.setItem('userNote', userInput);

// ✅ Good - validate and sanitize
const sanitized = DOMPurify.sanitize(userInput);
localStorage.setItem('userNote', sanitized);
```

---

## 6. Event Handlers

**❌ NEVER execute user input:** `onClick={() => eval(userCode)}`

### ✅ Use predefined actions:
```tsx
const SafeButton = ({ action }: { action: 'save' | 'delete' }) => {
  const handleClick = () => action === 'save' ? save() : deleteItem();
  return <button onClick={handleClick}>Submit</button>;
};
```

---

## 7. Dynamic Class Names

**❌ FORBIDDEN:** `<div className={`bg-${userColor}`}>` - CSS injection vulnerability

### ✅ REQUIRED - Use validated options:
```tsx
type Color = 'primary' | 'secondary' | 'destructive';
const SafeDiv = ({ color }: { color: Color }) => (
  <div className={`bg-${color}`}>Content</div>
);
```

---

## 8. RegEx & Search Security (ReDoS)

**The Problem:** Certain regex patterns cause exponential time complexity, freezing the browser.

### ❌ FORBIDDEN patterns:
```tsx
/^(a+)+$/.test(userInput) // Catastrophic backtracking
/^([a-z]+)*$/.test(userInput) // Nested quantifiers
new RegExp(userInput, 'i') // User-controlled pattern
```

### ✅ Use safe alternatives:
```tsx
// For search - use string methods (NOT regex)
products.filter(p => p.name.toLowerCase().includes(query.toLowerCase()))

// For validation - use Zod
z.string().email().safeParse(email).success
```

### When to avoid regex:
- User-controlled patterns (NEVER)
- Large dataset filtering
- Email/URL validation (use Zod or URL API)

### When regex is safe:
- Simple patterns with limited quantifiers
- Fixed-format inputs (ZIP codes)

---

## 9. React's Automatic XSS Protection

### ✅ React automatically escapes JSX:
```tsx
<h1>Hello, {username}!</h1> // Safe even if username = "<script>..."
```

**❌ NEVER bypass escaping:** `ref.current.innerHTML = userContent`

---

## 10. URL State Synchronization Security

### NEVER sync to URL (visible in browser history, logs, analytics):
- User PII (names, emails, phone numbers, addresses)
- Authentication tokens or session IDs
- Internal user/customer IDs
- Cart contents with customer data
- Credit card or payment information
- API keys or secrets

### SAFE to sync to URL:
- Public filter states (category, price range, sort order)
- Search queries (non-sensitive)
- Pagination state (page number, items per page)
- View preferences (grid/list view)

### When using state management with URL sync (e.g., Jotai's `atomWithHash`, Zustand persist):

```tsx
// ❌ Bad - syncing sensitive data to URL
const userAtom = atomWithHash('user', { id: 123, email: 'user@example.com' });

// ✅ Good - only non-sensitive filter state
const filterAtom = atomWithHash('filters', { category: 'electronics', sort: 'price-asc' });

// ✅ Good - validate and sanitize before syncing
const searchAtom = atomWithHash('q', '', {
  serialize: (value) => encodeURIComponent(value),
  deserialize: (value) => DOMPurify.sanitize(decodeURIComponent(value))
});
```

### REQUIRED - Review all URL-synced state:
- Audit all atoms/stores using URL persistence
- Ensure only safe, non-sensitive data is exposed
- Validate and sanitize all data before syncing

---

## 11. Authentication & Session Management

### NEVER store authentication data in client state:
- Authentication tokens
- Session IDs
- User credentials
- Refresh tokens

### ✅ REQUIRED - Use secure, HTTP-only cookies:
```typescript
// Server-side cookie configuration (Next.js API route example)
response.cookies.set('auth_token', token, {
  httpOnly: true,      // Inaccessible to JavaScript
  secure: true,        // HTTPS only
  sameSite: 'strict',  // CSRF protection
  maxAge: 3600,        // 1 hour
  path: '/'
});
```

### ✅ REQUIRED - Use Context API or AuthProvider for auth state:
```tsx
// ✅ Good - Auth context (does not store tokens, only auth status)
const AuthContext = createContext<{ isAuthenticated: boolean; user: User | null }>(null);

export const AuthProvider = ({ children }: { children: React.ReactNode }) => {
  const [isAuthenticated, setIsAuthenticated] = useState(false);
  const [user, setUser] = useState<User | null>(null);

  // Check auth status from server, don't store tokens
  useEffect(() => {
    fetch('/api/auth/me', { credentials: 'include' }) // Sends httpOnly cookie
      .then(res => res.json())
      .then(data => {
        setIsAuthenticated(true);
        setUser(data.user);
      })
      .catch(() => setIsAuthenticated(false));
  }, []);

  return (
    <AuthContext.Provider value={{ isAuthenticated, user }}>
      {children}
    </AuthContext.Provider>
  );
};

// ❌ Bad - storing tokens in state or localStorage
const [authToken, setAuthToken] = useState(localStorage.getItem('token'));
```

### Key principles:
- Store tokens only in secure, HTTP-only cookies (server-side)
- Client state should only track authentication status (boolean) and non-sensitive user info (name, email for display)
- All authenticated API calls should use `credentials: 'include'` to send cookies
- Isolate auth state from UI state atoms/stores

---

## 12. SSR/SSG Security & State Hydration

**The Risk:** When using SSR/SSG (Next.js), state can leak between users if not properly isolated.

### ❌ NEVER share sensitive state globally in SSR:
```tsx
// ❌ Bad - global state with user data in SSR
const userStore = create((set) => ({
  userId: null,
  email: null,
  cart: []
}));
```

### ✅ REQUIRED - Isolate per-user state:
```tsx
// ✅ Good - Server components fetch data per-request
export default async function ProfilePage() {
  const session = await getServerSession();
  const userData = await fetchUserData(session.userId);
  return <ProfileClient data={userData} />;
}

// ✅ Good - Client components use local state, not global
'use client';
export const ProfileClient = ({ data }: { data: UserData }) => {
  const [localState, setLocalState] = useState(data);
  return <div>{localState.name}</div>;
};
```

### REQUIRED - SSR/SSG security checklist:
- [ ] Never use global atoms/stores for sensitive user-specific data in SSR
- [ ] Fetch user-specific data per-request in Server Components
- [ ] Hydrate only non-sensitive state from server to client
- [ ] Clear sensitive state on logout
- [ ] Verify state isolation between users in SSR environments

---

## 13. CSRF Protection

### ✅ REQUIRED for all state-changing operations:
- Use `sameSite: 'strict'` or `'lax'` on cookies
- Implement CSRF tokens for critical actions (delete, update, purchase)
- Verify origin headers on server-side

```typescript
// Next.js middleware example
export function middleware(request: NextRequest) {
  const origin = request.headers.get('origin');
  const allowedOrigins = ['https://yourdomain.com'];

  if (request.method !== 'GET' && !allowedOrigins.includes(origin)) {
    return new Response('Forbidden', { status: 403 });
  }

  return NextResponse.next();
}
```

### Content Security Policy (CSP):

Add to `next.config.mjs`:
```javascript
const securityHeaders = [
  {
    key: 'Content-Security-Policy',
    value: "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline';"
  },
  {
    key: 'X-Frame-Options',
    value: 'DENY'
  },
  {
    key: 'X-Content-Type-Options',
    value: 'nosniff'
  },
  {
    key: 'Referrer-Policy',
    value: 'strict-origin-when-cross-origin'
  }
];

export default {
  async headers() {
    return [{ source: '/:path*', headers: securityHeaders }];
  }
};
```

---

## 14. Audit Logging & Monitoring

**IMPORTANT: Monitor and log security-critical events:**

### Log on client-side (send to analytics/monitoring):
- Failed authentication attempts (excessive failures)
- Authorization failures (accessing restricted resources)
- Suspicious state manipulation patterns
- XSS/injection attempt indicators

### Log on server-side:
- All authentication events (login, logout, token refresh)
- Failed authorization attempts
- Sensitive data access (cart checkout, profile updates)
- Rate limiting violations

### Example: Monitor suspicious activity
```tsx
const useSecurityMonitoring = () => {
  useEffect(() => {
    const handleError = (event: ErrorEvent) => {
      // Log potential XSS attempts
      if (event.message.includes('<script>') || event.message.includes('onerror')) {
        fetch('/api/security/log', {
          method: 'POST',
          body: JSON.stringify({
            type: 'xss_attempt',
            message: event.message,
            timestamp: Date.now()
          })
        });
      }
    };

    window.addEventListener('error', handleError);
    return () => window.removeEventListener('error', handleError);
  }, []);
};
```

---

## 15. Third-Party Library Security

### Before adding dependencies:
- [ ] Run `npm audit` - check for vulnerabilities
- [ ] Verify last update < 1 year (avoid abandoned packages)
- [ ] Check GitHub for security issues
- [ ] Confirm TypeScript support
- [ ] Review library's data handling practices
- [ ] Check if library requires unsafe patterns (`dangerouslySetInnerHTML`, `eval`)

**Avoid:** Unmaintained packages (>2 years), known CVEs, libraries requiring `dangerouslySetInnerHTML`

### Regularly audit dependencies:
```bash
npm audit
npm audit fix
npm outdated
```

---

## UI Security Checklist

Before completing any UI component:

### Input Handling:
- [ ] User input validated with Zod
- [ ] URL parameters encoded with `encodeURIComponent()`
- [ ] No `eval()`, `Function()`, or `innerHTML`
- [ ] Data sanitized before storing in localStorage
- [ ] Search queries sanitized before filtering/display

### HTML Rendering:
- [ ] `dangerouslySetInnerHTML` only with DOMPurify
- [ ] CMS/API content sanitized
- [ ] React's escaping not bypassed
- [ ] No user-controlled regex patterns

### Links & Navigation:
- [ ] External links have `rel="noopener noreferrer"`
- [ ] Redirects validated against whitelist
- [ ] Next.js Link used for internal navigation
- [ ] URL state sync contains no PII or sensitive data

### Client-Side Storage:
- [ ] Next.js Image component used
- [ ] No sensitive data in localStorage/sessionStorage
- [ ] No auth tokens, session IDs, or PII stored client-side
- [ ] No cart contents with customer data in localStorage
- [ ] Only non-sensitive UI preferences stored

### Authentication & State Management:
- [ ] Auth tokens stored in httpOnly cookies only
- [ ] Auth state isolated in Context API or AuthProvider
- [ ] No global state with user-specific sensitive data in SSR
- [ ] State properly isolated between users in SSR/SSG
- [ ] Clear sensitive state on logout

### CSRF & Headers:
- [ ] Cookies use `sameSite: 'strict'` or `'lax'`
- [ ] CSP headers configured in `next.config.mjs`
- [ ] Origin validation on state-changing operations

### Dependencies & Monitoring:
- [ ] `npm audit` shows no high/critical vulnerabilities
- [ ] Third-party libraries reviewed for security issues
- [ ] Security-critical events logged appropriately

---

## Back to Main Documentation

See [CLAUDE.md](../../CLAUDE.md) for the complete component creation workflow and project overview.
