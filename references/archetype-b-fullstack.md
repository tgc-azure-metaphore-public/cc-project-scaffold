# Archetype B: Full-Stack Product

Multi-user web app with auth, persistent database, and a real frontend. The default stack for anything public-facing.

## Stack (locked defaults)

- **Framework**: Next.js 15+ (App Router)
- **Language**: TypeScript
- **ORM**: Prisma
- **Database**: SQLite for local dev, Turso (libSQL) or Vercel Postgres for prod
- **Auth**: NextAuth v5 (Auth.js)
- **Styling**: Tailwind CSS
- **Deploy**: Vercel

Override any of these only if the user explicitly requests it.

## Files to generate

```
<project-name>/
├── CLAUDE.md
├── README.md
├── .gitignore
├── .env.template
├── package.json
├── tsconfig.json
├── next.config.ts
├── tailwind.config.ts
├── postcss.config.mjs
├── prisma/
│   └── schema.prisma
├── src/
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   └── globals.css
│   ├── components/
│   │   └── .gitkeep
│   └── lib/
│       ├── db.ts              ← Prisma client singleton
│       └── auth.ts            ← NextAuth config
├── prompts/
│   └── PROMPT-01-INIT-PROJECT.md
└── (optional) kill-criteria.md
```

## package.json

```json
{
  "name": "<project-name>",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "db:push": "prisma db push",
    "db:studio": "prisma studio"
  },
  "dependencies": {
    "next": "^15.0.0",
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "@prisma/client": "^6.0.0",
    "next-auth": "^5.0.0-beta.25",
    "@auth/prisma-adapter": "^2.7.0"
  },
  "devDependencies": {
    "typescript": "^5.6.0",
    "@types/node": "^22.0.0",
    "@types/react": "^19.0.0",
    "@types/react-dom": "^19.0.0",
    "prisma": "^6.0.0",
    "tailwindcss": "^3.4.0",
    "postcss": "^8.4.0",
    "autoprefixer": "^10.4.0",
    "eslint": "^9.0.0",
    "eslint-config-next": "^15.0.0"
  }
}
```

## prisma/schema.prisma skeleton

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}

// NextAuth models
model User {
  id            String    @id @default(cuid())
  name          String?
  email         String?   @unique
  emailVerified DateTime?
  image         String?
  accounts      Account[]
  sessions      Session[]
  createdAt     DateTime  @default(now())
}

model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String?
  access_token      String?
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String?
  session_state     String?
  user              User    @relation(fields: [userId], references: [id], onDelete: Cascade)
  @@unique([provider, providerAccountId])
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)
}

// Add project-specific models below
```

## .env.template

```
# Database
DATABASE_URL="file:./dev.db"

# Next Auth
AUTH_SECRET="<generate via: openssl rand -base64 32>"
AUTH_URL="http://localhost:3000"

# OAuth providers (optional — choose what you need)
# AUTH_GOOGLE_ID=
# AUTH_GOOGLE_SECRET=
# AUTH_GITHUB_ID=
# AUTH_GITHUB_SECRET=
```

## src/lib/db.ts

```typescript
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as { prisma: PrismaClient };

export const prisma = globalForPrisma.prisma ?? new PrismaClient();

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;
```

## src/app/layout.tsx

```typescript
import type { Metadata } from "next";
import "./globals.css";

export const metadata: Metadata = {
  title: "<Project Name>",
  description: "<One-line description>",
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

## src/app/page.tsx

```typescript
export default function Home() {
  return (
    <main className="min-h-screen p-8">
      <h1 className="text-4xl font-bold">Welcome to <Project Name></h1>
      <p className="mt-2 text-gray-600"><One-line description></p>
    </main>
  );
}
```

## CLAUDE.md section additions for Archetype B

In the project CLAUDE.md template, include:

- Tech stack: "Next.js 15 (App Router) + TypeScript + Prisma + SQLite/Turso + NextAuth + Tailwind"
- Folder structure (show the tree above)
- Deploy: Vercel
- Required env vars list
- "First commands": `npm install`, `npm run db:push`, `npm run dev`

## Deploy hint

```
1. `npm i -g vercel` then `vercel login`
2. From project root: `vercel --prod`
3. Set env vars in Vercel dashboard
4. Add Turso database: `npm i -g @turso/cli` then `turso db create <name>`
```

## Common add-ons (ask the user if they want these)

- **Stripe** for payments (archetypes where users pay)
- **Resend** or similar for transactional email
- **Vercel Blob** or S3 for file uploads
- **UploadThing** for image uploads specifically
- **tRPC** if the user wants typed RPC instead of REST

Do not add these by default. Generate a lean scaffold; let PROMPT-02+ add complexity.
