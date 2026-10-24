---
name: web-stack
description: Use when creating a webpage, website, web app, or HTTP API, or when choosing its UI, API, or Cloudflare infrastructure. Default stack is TypeScript, React, TanStack, Vite, Zod, Drizzle, Effect, and Alchemy. Leave an existing project stack and games alone.
---

# Web stack

Use this for a new webpage, website, web app, or HTTP API. An existing project keeps its stack unless the user asks to migrate. A game uses the `game-stack` skill. A native macOS app uses the `macos-stack` skill.

## Stack

- TypeScript
- React for UI. Interactive UI is React, not vanilla JavaScript
- TanStack Router, TanStack Query, and TanStack Form when the UI needs them
- Vite
- Zod at the boundary
- Drizzle for relational data
- Effect for services, errors, concurrency, and dependencies
- Alchemy (`alchemy` + `effect`) for the Cloudflare program and its resources

UI stays React. Effect is the service layer and the Alchemy program. A React component becomes Effect only when it is that service boundary.

Declare only the Cloudflare resources the app uses: Workers for the API, D1 for relational data, R2 for files, Durable Objects for per-entity coordination, Queues for background work. Alchemy is the only source for those resources. A second Wrangler or `cf` config does not join it.

## Before writing Alchemy or Effect

Follow current docs. Do not invent resource names or Effect APIs.

- Open https://alchemy.run/llms.txt and read only the pages this app needs. Resource reference is in https://alchemy.run/llms-full.txt, and only for the resource being used.
- Effect APIs come from the installed `effect` types and https://effect.website.

Install with `pnpm add alchemy@latest effect`. The entry is `alchemy.run.ts`. The local loop is `pnpm alchemy dev`. Credentials stay in Alchemy profiles (`alchemy profile edit`, or the first deploy). Do not ask for `CLOUDFLARE_ACCOUNT_ID` or `CLOUDFLARE_API_TOKEN`.

A production deploy goes through GitHub Actions. A deploy the user did not ask for waits.

## Leave out

XState and RxJS stay out unless the user asks for them.
