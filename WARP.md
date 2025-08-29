# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

Orbital CTF is a retro space-themed Capture The Flag platform built with Next.js 15, Prisma ORM, and Tailwind CSS. It features team-based competition, real-time scoring, challenge management, and multi-flag support.

## Commands

### Development
```bash
npm run dev              # Start development server with Turbopack (http://localhost:3000)
npm run build            # Build for production (runs prisma generate + next build)
npm run start            # Start production server
npm run lint             # Run ESLint
```

### Database Operations
```bash
npx prisma migrate dev    # Create and apply new migrations
npx prisma migrate reset  # Reset database and re-apply all migrations
npx prisma generate       # Generate Prisma Client
npm run prisma:seed       # Seed database with initial data
npx prisma studio         # Open Prisma Studio GUI
```

### Docker Operations
```bash
docker-compose up -d      # Run the platform in Docker containers
docker-compose down       # Stop and remove containers
docker-compose build      # Rebuild Docker images
```

## Architecture Overview

### Frontend
- **Framework**: Next.js 15 with App Router (`/src/app`)
- **Components**: React Server Components by default, client components marked with `'use client'`
- **Styling**: Tailwind CSS with mobile-first responsive design
- **3D Graphics**: Three.js with React Three Fiber for space-themed visuals
- **State Management**: URL search params for shareable state, minimal client-side state

### Backend
- **API Routes**: Next.js route handlers in `/src/app/api/*`
- **Database**: SQLite with Prisma ORM
- **Authentication**: NextAuth.js with credentials provider
- **Password Hashing**: bcrypt.js
- **Session Management**: JWT-based sessions

### Database Schema (`/prisma/schema.prisma`)
Key models and relationships:
- `User` → `Team` (many-to-one)
- `Challenge` → `ChallengeFlag` (one-to-many for multi-flag support)
- `Challenge` → `UnlockCondition` (one-to-many for gated challenges)
- `Team` → `Submission` → `Challenge` (submission tracking)
- `Team` → `TeamPointHistory` (scoring timeline)
- `Challenge` → `Hint` → `TeamHint` (hint system with costs)

## API Endpoints Structure

All API routes follow RESTful patterns in `/src/app/api/`:

### Public Endpoints
- `/api/auth/*` - Authentication (signin, signup, signout)
- `/api/challenges` - Get available challenges
- `/api/challenges/[id]` - Get specific challenge, submit flags
- `/api/leaderboard` - Get team rankings
- `/api/teams/*` - Team management (join, create, leave)
- `/api/announcements` - Platform announcements

### Protected Endpoints  
- `/api/submissions` - Submit challenge flags
- `/api/hints/[id]` - Purchase and view hints
- `/api/profile` - User profile operations

### Admin Endpoints (`/api/admin/*`)
- `/api/admin/challenges` - Challenge CRUD operations
- `/api/admin/users` - User management
- `/api/admin/teams` - Team administration
- `/api/admin/metrics` - Platform statistics

## Challenge Ingestion System

The platform supports automatic challenge import from filesystem:

### Directory Structure
```
challenges/
├── [category]/
│   └── [challenge-name]/
│       ├── challenge.json    # Required: Challenge metadata
│       ├── solve/
│       │   └── solve.md      # Optional: Solution writeup
│       └── files/            # Optional: Downloadable files
│           └── [files...]
```

### Challenge JSON Schema
```json
{
  "title": "Challenge Title",
  "description": "Challenge description in Markdown",
  "category": "web|crypto|pwn|reverse|forensics|misc",
  "difficulty": "easy|medium|hard",
  "points": 100,
  "flag": "flag{single_flag}",  // For single-flag challenges
  "multipleFlags": true,         // For multi-flag challenges
  "flags": [                     // If multipleFlags is true
    { "flag": "flag{part1}", "points": 50 },
    { "flag": "flag{part2}", "points": 50 }
  ],
  "hints": [
    { "content": "Hint text", "cost": 10 }
  ],
  "unlockConditions": [
    { "type": "CHALLENGE_SOLVED", "requiredChallengeId": "..." },
    { "type": "TIME_REMAINDER", "timeThresholdSeconds": 3600 }
  ],
  "link": "https://challenge-instance.example.com"  // Optional external link
}
```

### Ingestion Process
- Set `INGEST_CHALLENGES_AT_STARTUP=true` in `.env`
- Challenges are imported on server start via `/src/instrumentation.ts`
- Files are copied to `/public/uploads/[challenge-name]/`
- Use `ChallengeIngestionService` in `/src/lib/challenge-ingestion.ts`

## Development Guidelines

### Next.js Conventions
- Use App Router structure with `page.tsx` files in route directories
- Keep most components as React Server Components (RSC)
- Only use `'use client'` when needed for interactivity
- Prefer server components for data fetching
- Use React Server Actions for form handling
- Prefer named exports: `export function Button()` over `export default`

### File & Directory Naming
- **Directories**: kebab-case (e.g., `auth-form`, `challenge-card`)
- **Component Files**: PascalCase (e.g., `AuthForm.tsx`, `ChallengeCard.tsx`)
- **API Routes**: lowercase (e.g., `route.ts`)
- **Utilities**: camelCase (e.g., `formatDate.ts`)

### Code Organization
```
src/
├── app/              # Next.js App Router pages and API routes
├── components/       # Reusable React components
├── lib/             # Server-side utilities and services
├── utils/           # Client-side utilities
├── types/           # TypeScript type definitions
└── middleware.ts    # Next.js middleware for auth protection
```

### Tailwind CSS Patterns
- Use responsive prefixes: `w-full md:w-1/2 lg:w-1/3`
- Apply state variants: `hover:bg-blue-600 focus:ring-2`
- Use spacing utilities: `space-y-4` for consistent gaps
- Arbitrary values when needed: `top-[117px]`

## Configuration

### Environment Variables
```env
# Authentication
NEXTAUTH_SECRET="your-secret-here"        # Generate with: openssl rand -base64 32
NEXTAUTH_URL="http://localhost:3000"      # Your app URL

# Database
DATABASE_URL="file:./dev.db"              # SQLite database file

# Challenge Import
INGEST_CHALLENGES_AT_STARTUP=true         # Auto-import challenges on start
CHALLENGES_DIR="./challenges"             # Directory containing challenges
```

### Database Setup
1. **Development**: SQLite file at `prisma/dev.db`
2. **Production**: Can use PostgreSQL/MySQL by changing provider in `schema.prisma`
3. **Migrations**: Tracked in `/prisma/migrations/`

## Quick Start

1. **Clone and install dependencies**
   ```bash
   git clone <repository>
   cd bipci-command-hq
   npm install
   ```

2. **Configure environment**
   ```bash
   cp .env.example .env
   # Edit .env with your values
   ```

3. **Initialize database**
   ```bash
   npx prisma migrate reset  # Creates DB and runs migrations
   npm run prisma:seed       # Add initial data
   ```

4. **Add challenges** (optional)
   - Place challenge directories in `./challenges/`
   - Set `INGEST_CHALLENGES_AT_STARTUP=true` in `.env`

5. **Start development server**
   ```bash
   npm run dev
   ```
   Open http://localhost:3000

## Common Development Tasks

### Add a new database model
1. Edit `/prisma/schema.prisma`
2. Run `npx prisma migrate dev --name describe_change`
3. Update TypeScript types will auto-generate

### Create a new API endpoint
1. Create route file: `/src/app/api/[resource]/route.ts`
2. Export async functions: `GET`, `POST`, `PUT`, `DELETE`
3. Use `prisma` from `/src/lib/prisma.ts` for DB operations
4. Check authentication with `getServerSession(authOptions)`

### Add a new page
1. Create directory: `/src/app/[page-name]/`
2. Add `page.tsx` with default export
3. Use `loading.tsx` for loading states
4. Use `error.tsx` for error boundaries

### Work with challenges
- Import: Place in `./challenges/[category]/[name]/` with `challenge.json`
- Export: Use admin API `/api/admin/challenges/export`
- Test: Submit flags via `/api/challenges/[id]/submit`

## Authentication Flow

1. **Registration**: `/api/auth/signup` → Creates user with bcrypt password
2. **Login**: NextAuth.js credentials provider → JWT session
3. **Protected Routes**: Middleware checks session in `middleware.ts`
4. **Admin Access**: Check `isAdmin` flag on user session

## Team System

- Users can create or join teams using invite codes
- Team leaders can manage members and team settings  
- Scoring aggregates individual member submissions
- Point history tracked in `TeamPointHistory` for timeline visualization
