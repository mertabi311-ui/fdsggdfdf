# replit.md

## Overview

"Get Robux" is a mobile application built with Expo (React Native) that presents a multi-step flow where users enter a Roblox username, select a Robux package, go through simulated "searching" and "preparing" animations, and are then directed to a CPA (Cost Per Action) offer wall for "verification." The app has an Express backend server that proxies the CPA offer feed and serves a landing page. The project uses a PostgreSQL database with Drizzle ORM for user storage, though the current storage implementation is in-memory.

## User Preferences

Preferred communication style: Simple, everyday language.

## System Architecture

### Frontend (Expo / React Native)
- **Framework**: Expo SDK 54 with React Native 0.81, using the new architecture
- **Routing**: expo-router with file-based routing (`app/` directory). Currently has a single main screen (`app/index.tsx`) with a step-based flow managed via React state
- **State Management**: React Query (`@tanstack/react-query`) for server state, local React state for UI flow
- **UI Components**: Custom components using React Native primitives, expo-image, expo-haptics, react-native-webview (for displaying the CPA offer wall), and various Expo modules
- **Fonts**: Inter font family loaded via `@expo-google-fonts/inter`
- **Error Handling**: Custom ErrorBoundary class component with ErrorFallback UI
- **Platform Support**: Primarily mobile (iOS/Android), with basic web support. iOS tablet support is disabled.

### Backend (Express Server)
- **Framework**: Express 5 running on Node.js
- **Entry Point**: `server/index.ts`
- **Routes**: Defined in `server/routes.ts` — currently has one API endpoint (`GET /api/offers`) that proxies the CPA offer feed from a CloudFront URL
- **CORS**: Configured dynamically based on Replit environment variables and localhost origins
- **Static Serving**: In production, serves a pre-built static landing page from `server/templates/landing-page.html` and Expo web bundles from `dist/`
- **Build Pipeline**: 
  - Dev: `expo:dev` runs Metro bundler, `server:dev` runs the Express server with tsx
  - Prod: `expo:static:build` builds the Expo web app, `server:build` bundles the server with esbuild, `server:prod` runs the production server

### Database
- **ORM**: Drizzle ORM with PostgreSQL dialect
- **Schema**: Defined in `shared/schema.ts` — currently has a `users` table with `id` (UUID), `username`, and `password` fields
- **Schema Validation**: Uses `drizzle-zod` to generate Zod schemas from Drizzle table definitions
- **Migrations**: Output to `./migrations` directory via `drizzle-kit`
- **Current Storage**: The `server/storage.ts` file implements an in-memory storage (`MemStorage`) rather than using the Drizzle/Postgres connection. The database configuration exists but isn't actively wired into the routes yet.
- **Connection**: Requires `DATABASE_URL` environment variable for PostgreSQL

### Project Structure
```
app/              — Expo Router screens (file-based routing)
components/       — Reusable React Native components
constants/        — App-wide constants (colors)
lib/              — Client utilities (API client, query client)
server/           — Express backend server
shared/           — Shared code between client and server (DB schema)
scripts/          — Build scripts
assets/           — Images, fonts, and other static assets
attached_assets/  — Reference files (Chrome extension code, not part of the app)
migrations/       — Drizzle database migrations
```

### Key Design Decisions
1. **Monorepo structure**: Frontend (Expo) and backend (Express) live in the same repository, sharing code via the `shared/` directory
2. **In-memory storage as default**: The app starts with MemStorage for rapid development, with Drizzle+Postgres ready to be plugged in
3. **CPA offer proxy**: The backend proxies external offer feed requests to avoid CORS issues on the client
4. **Replit-optimized**: Build scripts and environment variable handling are tailored for Replit's deployment infrastructure

## External Dependencies

### Third-Party Services
- **CPA Offer Wall**: CloudFront-hosted offer feed at `d2xohqmdyl2cj3.cloudfront.net` — proxied through the Express server's `/api/offers` endpoint
- **PostgreSQL**: Required via `DATABASE_URL` environment variable (Replit provisioned or external)

### Key NPM Packages
- **expo** (~54.0.27): Core framework for cross-platform mobile development
- **express** (^5.0.1): Backend HTTP server
- **drizzle-orm** (^0.39.3) + **drizzle-kit**: Database ORM and migration tooling
- **@tanstack/react-query** (^5.83.0): Server state management
- **react-native-webview** (^13.16.0): Used to display the CPA offer wall in-app
- **pg** (^8.16.3): PostgreSQL client for Node.js
- **zod** + **drizzle-zod**: Runtime type validation for schemas
- **esbuild**: Server bundling for production builds
- **tsx**: TypeScript execution for development server