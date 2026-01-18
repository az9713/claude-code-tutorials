---
name: security
description: Security best practices - OWASP, authentication, and secure coding
---

# Security Best Practices

## OWASP Top 10 Prevention

### 1. Injection (SQL, NoSQL, Command)

```typescript
// ❌ SQL Injection vulnerable
const query = `SELECT * FROM users WHERE id = '${userId}'`;

// ✅ Parameterized queries
const query = 'SELECT * FROM users WHERE id = $1';
await db.query(query, [userId]);

// ✅ ORM with proper escaping
await User.findOne({ where: { id: userId } });

// ❌ Command injection vulnerable
exec(`convert ${filename} output.png`);

// ✅ Use arrays, not string concatenation
execFile('convert', [filename, 'output.png']);
```

### 2. Broken Authentication

```typescript
// Password hashing
import bcrypt from 'bcrypt';

const SALT_ROUNDS = 12;

async function hashPassword(password: string): Promise<string> {
  return bcrypt.hash(password, SALT_ROUNDS);
}

async function verifyPassword(password: string, hash: string): Promise<boolean> {
  return bcrypt.compare(password, hash);
}

// Rate limiting login attempts
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts
  message: 'Too many login attempts, please try again later'
});

app.post('/login', loginLimiter, loginHandler);
```

### 3. Cross-Site Scripting (XSS)

```typescript
// ❌ Direct HTML insertion
element.innerHTML = userInput;
res.send(`<div>${userComment}</div>`);

// ✅ Text content (auto-escaped)
element.textContent = userInput;

// ✅ Sanitize HTML if needed
import DOMPurify from 'dompurify';
element.innerHTML = DOMPurify.sanitize(userInput);

// ✅ React auto-escapes by default
<div>{userComment}</div>

// ⚠️ Dangerous - avoid unless necessary
<div dangerouslySetInnerHTML={{ __html: sanitizedHtml }} />

// Content Security Policy header
Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'
```

### 4. Insecure Direct Object References

```typescript
// ❌ No authorization check
app.get('/documents/:id', async (req, res) => {
  const doc = await Document.findById(req.params.id);
  res.json(doc);
});

// ✅ Verify ownership/access
app.get('/documents/:id', async (req, res) => {
  const doc = await Document.findById(req.params.id);

  if (!doc) {
    return res.status(404).json({ error: 'Not found' });
  }

  if (doc.ownerId !== req.user.id && !req.user.isAdmin) {
    return res.status(403).json({ error: 'Forbidden' });
  }

  res.json(doc);
});
```

### 5. Security Misconfiguration

```typescript
// Secure headers with Helmet
import helmet from 'helmet';
app.use(helmet());

// Disable X-Powered-By
app.disable('x-powered-by');

// Environment-specific settings
if (process.env.NODE_ENV === 'production') {
  app.set('trust proxy', 1);
  sessionConfig.cookie.secure = true;
}

// Don't expose stack traces in production
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({
    error: process.env.NODE_ENV === 'production'
      ? 'Internal server error'
      : err.message
  });
});
```

## Authentication Patterns

### JWT Best Practices

```typescript
import jwt from 'jsonwebtoken';

// Short-lived access tokens
const accessToken = jwt.sign(
  { userId: user.id, role: user.role },
  process.env.JWT_SECRET,
  { expiresIn: '15m' }
);

// Long-lived refresh tokens (store in HTTP-only cookie)
const refreshToken = jwt.sign(
  { userId: user.id, tokenVersion: user.tokenVersion },
  process.env.JWT_REFRESH_SECRET,
  { expiresIn: '7d' }
);

// Set refresh token as HTTP-only cookie
res.cookie('refreshToken', refreshToken, {
  httpOnly: true,
  secure: process.env.NODE_ENV === 'production',
  sameSite: 'strict',
  maxAge: 7 * 24 * 60 * 60 * 1000 // 7 days
});
```

### Session Security

```typescript
import session from 'express-session';
import RedisStore from 'connect-redis';

app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: process.env.NODE_ENV === 'production',
    httpOnly: true,
    sameSite: 'strict',
    maxAge: 24 * 60 * 60 * 1000 // 24 hours
  }
}));
```

## Input Validation

```typescript
import { z } from 'zod';

// Define schema
const createUserSchema = z.object({
  email: z.string().email().max(255),
  password: z.string().min(8).max(100),
  name: z.string().min(1).max(100).regex(/^[a-zA-Z\s]+$/),
  age: z.number().int().min(18).max(120).optional()
});

// Validate input
function validateInput<T>(schema: z.Schema<T>, data: unknown): T {
  const result = schema.safeParse(data);
  if (!result.success) {
    throw new ValidationError(result.error.errors);
  }
  return result.data;
}

// Use in route
app.post('/users', (req, res) => {
  const userData = validateInput(createUserSchema, req.body);
  // Safe to use userData
});
```

## Secrets Management

```typescript
// ❌ Never hardcode secrets
const API_KEY = 'sk_live_abc123';

// ✅ Use environment variables
const API_KEY = process.env.API_KEY;

// ✅ Validate required env vars at startup
const requiredEnvVars = ['DATABASE_URL', 'JWT_SECRET', 'API_KEY'];
for (const envVar of requiredEnvVars) {
  if (!process.env[envVar]) {
    throw new Error(`Missing required environment variable: ${envVar}`);
  }
}

// ✅ Use secret management services in production
// AWS Secrets Manager, HashiCorp Vault, etc.
```

## Security Checklist

- [ ] All user input validated and sanitized
- [ ] Parameterized queries for all database operations
- [ ] Passwords hashed with bcrypt (cost ≥ 12)
- [ ] HTTPS enforced everywhere
- [ ] Security headers set (Helmet)
- [ ] CORS configured restrictively
- [ ] Rate limiting on authentication endpoints
- [ ] JWT tokens short-lived with refresh mechanism
- [ ] Sensitive data not logged
- [ ] Dependencies regularly updated
- [ ] No secrets in code or version control
