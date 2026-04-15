---
project_name: 'DEVZONE'
user_name: 'Matthieu'
date: '2026-03-13'
sections_completed: ['technology_stack', 'language_rules', 'framework_rules', 'testing_rules', 'code_quality', 'workflow_rules', 'critical_rules']
status: 'complete'
rule_count: 42
optimized_for_llm: true
---

# Project Context for AI Agents

_This file contains critical rules and patterns that AI agents must follow when implementing code in this project. Focus on unobvious details that agents might otherwise miss._

---

## Technology Stack & Versions

### Backend
- **Runtime:** Node.js >=22.8.0
- **Framework:** NestJS 11.1.0 + Apollo GraphQL 13.1.0
- **Database:** PostgreSQL with Prisma ORM 6.15.0
- **TypeScript:** 5.8.3 — strict mode, CommonJS module, target es2017, `noImplicitAny: false`
- **Caching:** Redis 5.0.1
- **Storage:** AWS S3 (SDK 3.806.0) for media files
- **AI:** @ai-sdk/openai 1.3.22, @mastra/core 0.10.0
- **Email:** Resend 6.4.1
- **Push notifications:** expo-server-sdk 3.15.0
- **Deploy:** Docker (Node 22.9.0-alpine), CircleCI, AWS ECR

### Mobile
- **Runtime:** Node.js ^18.8.0 (different from backend!)
- **Framework:** React Native 0.81.5 + Expo ~54.0.31 + React 19.1.0
- **Navigation:** Expo Router ~6.0.21 (file-based routing)
- **API Client:** Apollo Client 3.13.8 + GraphQL Codegen
- **TypeScript:** ~5.9.2 — strict mode, ESNext module
- **Styling:** Emotion (@emotion/native 11.11.0) — NOT StyleSheet
- **Forms:** react-hook-form 7.56.3 + Zod 3.22.4 + @hookform/resolvers
- **i18n:** react-intl 7.1.11 + FormatJS CLI
- **Design System:** Evanescent (internal, under `/evanescent/src/`)
- **Analytics:** Pendo (rn-pendo-sdk 3.8.0)
- **Auth:** Google Sign-In, Apple Sign-In
- **Build:** EAS (Expo Application Services)

## Critical Implementation Rules

### Language-Specific Rules (TypeScript)

- **Two TypeScript configs:** Backend = CommonJS/es2017 with decorators; Mobile = ESNext with path aliases. Never mix module systems.
- **Backend allows `noImplicitAny: false`** — mobile does not. Backend is more lenient with untyped code.
- **Path aliases (mobile only):** Always use `@app/`, `@domains/`, `@shared/`, `@theme/`, `@nara/evanescent/` — never relative paths crossing domain boundaries.
- **Evanescent is isolated:** The design system MUST NOT import from `@app/`, `@domains/`, or `@shared/`. Dependency flows one way: app → evanescent.
- **Backend error handling:** Use `ApolloError` for GraphQL errors, `UserInputError` for input validation. Never throw raw `Error`.
- **No `console.log`:** ESLint blocks it in both projects. Use `console.warn`/`console.error` only, or NestJS `Logger` in backend.
- **Backend decorators required:** NestJS relies on `emitDecoratorMetadata` + `experimentalDecorators` — never disable these.
- **Async/await only:** Use async/await, not raw Promise chains.

### Framework-Specific Rules

#### Backend (NestJS)
- **Module pattern:** Every feature gets its own module (`*.module.ts`) that declares providers, imports, and exports. Never put services in a shared "utils" module.
- **GraphQL file convention:** Each feature has `*.resolver.ts` (queries/mutations), `*.entity.ts` (GraphQL types), `*.input.ts` (input types). Follow this naming strictly.
- **Repository pattern:** Database access goes through `*Repository` classes, not directly in services. Services handle business logic only.
- **Event-driven:** Use `EventEmitter2` for cross-module side effects (e.g., thumbnail generation after media upload). Define event constants in `events/events.constants.ts`.
- **DI injection:** Use constructor injection. For custom providers, use `@Inject(TOKEN)` pattern (e.g., `@Inject(STORAGE_PROVIDER)`).
- **Prisma access:** Always through injected `PrismaService` — never import Prisma client directly.

#### Mobile (React Native / Expo)
- **Routing:** Expo Router file-based routing. Screens in `src/app/`, grouped by `(auth)` and `(app)` layouts.
- **Domain architecture:** Feature logic in `src/domains/{feature}/`. Shared code in `src/shared/`.
- **Styling:** Use Emotion `styled()` exclusively. NEVER use `StyleSheet.create()`.
- **Forms:** Always react-hook-form + Zod schema. Never manage form state manually.
- **i18n:** All user-facing text through react-intl. Never hardcode strings. Use FormatJS CLI for extraction.
- **Design system:** Use Evanescent components (`@nara/evanescent/`) for UI primitives before creating custom ones.
- **Apollo Client:** GraphQL queries/mutations via Apollo hooks. Run `api:generate-types` after schema changes.

### Testing Rules

#### Backend
- **Test file naming:** `*.spec.ts` — always colocated with source in `src/`.
- **E2E tests:** Separate config at `test/jest-e2e.json`. Keep unit and e2e tests distinct.
- **Transformer:** ts-jest for all `.ts`/`.js` files.
- **Environment:** Node (not jsdom).

#### Mobile
- **Test file naming:** `*.test.tsx` or `*.spec.tsx` in `__tests__/` directories next to the component.
- **Preset:** jest-expo — never use plain jest config for mobile.
- **Path aliases in tests:** Module name mappers are configured — use `@domains/`, `@shared/`, etc. in test imports, same as source.
- **GraphQL in tests:** `.gql` files transformed via `jest-transform-graphql` — import queries directly.
- **Component testing:** Use `@testing-library/react-native` — never use `enzyme` or direct renderer.
- **Setup:** Global setup at `config/test/jest.setup.ts` — add shared mocks there, not in individual tests.

### Code Quality & Style Rules

- **Prettier (both):** Single quotes, trailing commas (`all`), auto end-of-line. Mobile adds `printWidth: 80`, `bracketSameLine: false`.
- **Type naming:** PascalCase for all TypeScript types and interfaces (ESLint-enforced).
- **Backend entity naming:** Always suffix with `Entity` (e.g., `NarativeEntity`, `UserEntity`).
- **File naming:** kebab-case for all files in both projects (e.g., `friend-request.module.ts`, `narative-day.service.ts`).
- **Backend file convention:** Each feature module contains: `*.module.ts`, `*.service.ts`, `*.repository.ts`, `*.resolver.ts`, `*.entity.ts`, `*.input.ts`.
- **Git hooks (mobile):** Husky runs ESLint on pre-commit and Prettier on pre-push. Code must pass both before landing.
- **Import organization:** ESLint enforces import ordering. Mobile restricts cross-boundary imports (evanescent cannot import app code).

### Development Workflow Rules

- **Branch → environment:** `develop` → dev, `integration` → integration, `master` → production. Never push directly to `master`.
- **Prisma migrations:** Run `npm run prisma:migrate` locally after schema changes. Migrations auto-run on deploy via `prisma migrate deploy`.
- **GraphQL codegen:** After any backend schema change, run `npm run api:generate-types` in mobile to regenerate TypeScript types.
- **i18n workflow:** After adding new user-facing strings, run `npm run translations:extract` then `npm run translations:compile` in mobile.
- **Docker build:** Multi-stage — dev dependencies are NOT in production image. Ensure new dependencies are in the correct `dependencies` vs `devDependencies`.
- **Environment variables:** Mobile uses `EXPO_PUBLIC_` prefix for all client-side env vars. Backend uses standard `process.env`.
- **AWS region:** `eu-west-3` (Paris). All infrastructure in this region.

### Critical Don't-Miss Rules

- **"Narative" spelling:** The project uses `narative` (single 'r') consistently — NOT `narrative`. Follow this everywhere: models, files, variables, routes.
- **Root .gitignore:** Ignores `mobile/` and `backend/` directories. These are tracked separately — be aware of this when working at the monorepo root.
- **Node version mismatch:** Backend requires Node >=22.8.0, mobile requires ^18.8.0. Never assume the same Node version across projects.
- **S3 media access:** Always use presigned URLs (`@aws-sdk/s3-request-presigner`). Never expose raw S3 bucket paths or credentials to clients.
- **Phone number security:** Phone numbers are stored as hashes (`HashedPhoneNumber` model). Never store or log plain phone numbers.
- **Multer memory storage:** File uploads use in-memory storage — large files are buffered in RAM. Be mindful of upload size limits.
- **ffmpeg dependency:** Backend Docker image installs ffmpeg for video processing. Any video-related features require ffmpeg to be available.
- **Portrait only:** Mobile app is locked to portrait orientation. Never add landscape layout logic.
- **Pendo analytics:** Track analytics events via Pendo SDK. Follow existing event patterns — don't create ad-hoc tracking.

---

## Usage Guidelines

**For AI Agents:**
- Read this file before implementing any code
- Follow ALL rules exactly as documented
- When in doubt, prefer the more restrictive option
- Update this file if new patterns emerge

**For Humans:**
- Keep this file lean and focused on agent needs
- Update when technology stack changes
- Review quarterly for outdated rules
- Remove rules that become obvious over time

Last Updated: 2026-03-13
