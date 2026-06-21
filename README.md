# One-Story AI Tools

A collection of AI-powered tools for real estate professionals, built with Next.js and the Anthropic API.

## Tools

### AI Listing Description Generator

Generates polished, MLS-ready property listing descriptions from structured property details. Supports six tones (Professional, Luxury, Warm and Inviting, Modern, Lifestyle-Focused, Editorial), three length targets (~600 / ~1,200 / ~1,800 characters), and one-click refinements (Make Shorter, More Luxurious, Make Warmer).

**Key design decisions:**

- The Anthropic API key is read server-side only — it never reaches the browser.
- All user input is validated and sanitized on the server before the prompt is built.
- The prompt includes Fair Housing Act compliance and no-fabrication guardrails.
- A sliding-window rate limiter (10 req / min per IP) protects the API route.

---

## Tech stack

| Layer | Choice |
|---|---|
| Framework | Next.js 16 (App Router) |
| Language | TypeScript |
| Styling | Tailwind CSS v4 |
| AI | Anthropic API (`claude-sonnet-4-6`) via `@anthropic-ai/sdk` |

---

## Local development

### Prerequisites

- Node.js 18 or later
- An [Anthropic API key](https://console.anthropic.com/)

### Setup

1. **Clone the repo**

   ```bash
   git clone <repo-url>
   cd one-story-ai-tools
   ```

2. **Install dependencies**

   ```bash
   npm install
   ```

3. **Create your environment file**

   ```bash
   cp .env.example .env.local
   ```

   Open `.env.local` and add your Anthropic API key:

   ```
   ANTHROPIC_API_KEY=your-key-here
   ```

4. **Start the dev server**

   ```bash
   npm run dev
   ```

   Open [http://localhost:3000](http://localhost:3000).

### Other useful commands

```bash
npm run build   # Production build (runs TypeScript checks)
npm run lint    # ESLint
npm start       # Serve the production build locally
```

---

## Project structure

```
app/
  api/generate-listing/route.ts   # Server-side API route (Anthropic calls happen here)
  globals.css                     # CSS variables and base styles
  layout.tsx
  page.tsx                        # Main page — wires form ↔ results
components/
  ListingForm.tsx                  # Property input form
  ResultsArea.tsx                  # Description output, Copy, Regenerate, Refine
lib/
  config.ts        # Server-only: AI model name, max tokens, rate limit config
  fieldLimits.ts   # Character caps shared by client (maxLength) and server (sanitize)
  rateLimiter.ts   # In-memory sliding-window rate limiter (swap for Redis in prod)
  types.ts         # Shared TypeScript types
```

---

## Deployment

### Vercel (recommended)

1. Push the repo to GitHub.
2. Import the project at [vercel.com/new](https://vercel.com/new).
3. In **Settings → Environment Variables**, add `ANTHROPIC_API_KEY` with your key.
4. Deploy. Vercel automatically runs `npm run build` on each push to `main`.

### Other platforms (Fly.io, Railway, Render, etc.)

The app is a standard Node.js Next.js application. Set the `ANTHROPIC_API_KEY` environment variable in your platform's secrets manager and run:

```bash
npm run build
npm start        # listens on PORT (default 3000)
```

### Rate limiter note

The built-in rate limiter stores state in process memory. It works correctly for single-instance deployments. For multi-instance or serverless deployments with many concurrent instances, swap the in-memory store in `lib/rateLimiter.ts` for a shared Redis store (e.g. Upstash Redis).

---

## Environment variables

| Variable | Required | Description |
|---|---|---|
| `ANTHROPIC_API_KEY` | Yes | Your Anthropic API key. Never expose this client-side. |

See `.env.example` for a template.

---

## Security notes

- `ANTHROPIC_API_KEY` is read in `app/api/generate-listing/route.ts` only. It is never imported into any client component and never bundled into the browser build.
- All submitted fields are allowlist-validated and character-capped server-side before the prompt is constructed.
- The server logs only the Anthropic error status code and class name on failure — never the prompt or user-submitted data.
- `.env.local` and `.claude/settings.local.json` are gitignored.
