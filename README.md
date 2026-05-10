<div align="center">

# Bodica

**A production-grade body language learning app, shipping on iOS and Android.**

Built end-to-end as a solo project — from database schema and Edge Functions to native auth flows, in-app purchases, and a privacy-first analytics pipeline.

[![React Native](https://img.shields.io/badge/React%20Native-0.81-61DAFB?style=flat-square&logo=react&logoColor=white)](https://reactnative.dev/)
[![Expo](https://img.shields.io/badge/Expo-54-000020?style=flat-square&logo=expo&logoColor=white)](https://expo.dev/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.9-3178C6?style=flat-square&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Supabase](https://img.shields.io/badge/Supabase-Postgres%20%2B%20Edge-3FCF8E?style=flat-square&logo=supabase&logoColor=white)](https://supabase.com/)
[![RevenueCat](https://img.shields.io/badge/RevenueCat-StoreKit%202-F25A5A?style=flat-square)](https://www.revenuecat.com/)
[![Platform](https://img.shields.io/badge/Platform-iOS%20%7C%20Android-lightgrey?style=flat-square)](#)
[![License](https://img.shields.io/badge/Source-Private-1f2937?style=flat-square)](#)

[App Store](https://apps.apple.com/app/id6756843038) · [Google Play](#) · [Contact](#contact)

</div>

---

## Overview

Bodica teaches users to read nonverbal communication through structured lessons, interactive quizzes, and AI-powered photo analysis. The app ships in Turkish and English, with a premium subscription tier for advanced content and unlimited AI analyses.

The codebase is organized as a feature-first React Native application running on Hermes with the New Architecture enabled. State is split across three layers — Zustand for client state, TanStack Query for server state, and encrypted MMKV for persistence — with a Supabase backend handling Postgres, Auth, and serverless Edge Functions. Every cross-cutting concern (auth, storage, telemetry, networking, error handling) is treated as a first-class problem with explicit solutions.

This README is a tour of the engineering decisions behind the app.

## At a Glance

|                  |                                                                                            |
| ---------------- | ------------------------------------------------------------------------------------------ |
| **Platforms**    | iOS 15.1+, Android, single TypeScript codebase                                             |
| **Stores**       | Live on the App Store (Bodica) and Google Play                                             |
| **Codebase**     | ~57K lines, 11 feature modules, 56 test files                                              |
| **Backend**      | Supabase Postgres with 70+ Row Level Security policies, 6 Edge Functions                   |
| **Auth**         | Native Apple, native Google, and email OTP — all token-based, no browser redirect          |
| **State**        | 10 global Zustand stores, 38+ React Query hooks, 3 MMKV instances (1 hardware-encrypted)   |
| **Monetization** | RevenueCat with StoreKit 2 and webhook-driven entitlements                                 |
| **Privacy**      | GDPR consent gating, Sentry PII scrubbing, encrypted token storage, Apple Privacy Manifest |

## Screenshots

> _Screenshots coming soon._

<!--
<p align="center">
  <img src="docs/screenshots/home.png" width="200" />
  <img src="docs/screenshots/learn.png" width="200" />
  <img src="docs/screenshots/analyze.png" width="200" />
  <img src="docs/screenshots/quiz.png" width="200" />
</p>
-->

## Tech Stack

**Mobile runtime**
React Native 0.81 · Expo SDK 54 (EAS) · Expo Router 6 (typed routes, async loading) · Hermes · New Architecture · React 19 · React Compiler

**State & data**
Zustand 5 · TanStack React Query 5 · react-native-mmkv 4 · @supabase/supabase-js 2 · async-mutex

**Backend**
Supabase Postgres · Supabase Auth · Supabase Edge Functions (Deno) · Row Level Security

**Authentication**
expo-apple-authentication (native Apple) · @react-native-google-signin/google-signin (native Google) · Supabase email OTP

**Monetization**
react-native-purchases (RevenueCat) · StoreKit 2 · server-side webhook validation

**UI & UX**
react-native-reanimated 4 · react-native-gesture-handler · @shopify/flash-list · expo-image · expo-haptics · expo-linear-gradient · expo-blur · react-native-keyboard-controller · phosphor-react-native

**Forms & validation**
react-hook-form · zod

**Localization**
i18next · react-i18next · expo-localization (TR / EN)

**Notifications**
expo-notifications (push, local, actionable categories, background handler)

**Observability**
@sentry/react-native (errors, PII scrubbing, ErrorBoundary) · posthog-react-native (EU-hosted, consent-gated)

**Quality & DX**
Jest · @testing-library/react-native · ESLint 9 · Husky · lint-staged · TypeScript strict · expo-doctor

## Architecture

### System Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                         Mobile Client                            │
│                    React Native + Expo + Hermes                  │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────────┐  │
│  │   Zustand    │  │ React Query  │  │       MMKV x3          │  │
│  │ client state │  │ server state │  │ (encrypted/app/cache)  │  │
│  └──────────────┘  └──────────────┘  └────────────────────────┘  │
│         │                 │                       │              │
│  ┌──────┴─────────────────┴───────────────────────┴──────────┐   │
│  │              Feature modules (11)                         │   │
│  │  auth · home · learn · lesson · category · quiz ·         │   │
│  │  analyze · profile · notifications · onboarding · skills  │   │
│  └───────────────────────────┬───────────────────────────────┘   │
└──────────────────────────────┼───────────────────────────────────┘
                               │ HTTPS, mutex-serialized refresh
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│                          Supabase                                │
│   Postgres + Row Level Security · Auth · Edge Functions (Deno)  │
│                                                                  │
│   AI analysis · Push delivery · Push receipts · Account delete   │
│   RevenueCat webhook · Subscription verification                 │
└──────────────────────────────────────────────────────────────────┘
                               │
                ┌──────────────┼──────────────┐
                ▼              ▼              ▼
            Sentry         PostHog        RevenueCat
          (errors)       (analytics)    (entitlements)
```

### State Management — Three Layers

| Layer            | Tool                             | Responsibility                                                                                                                                                                                           |
| ---------------- | -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Client state** | Zustand (10 stores)              | Auth session, UI state (toasts, banners), network status, GDPR consent, prompt queue, feature-specific state (quiz progress, analysis flow). Persists to MMKV via a custom storage adapter.              |
| **Server state** | TanStack React Query (38+ hooks) | Fetching, caching (5 min stale, 10 min GC), mutations, smart retry that skips 401/403/404, automatic Sentry capture with sanitized query keys.                                                           |
| **Persistence**  | react-native-mmkv (3 instances)  | `secureStorage` (encrypted with a hardware-keychain-derived key, holds auth tokens), `appStorage` (preferences, onboarding, rate limits, notification config), `cacheStorage` (React Query persistence). |

A `NetworkProvider` wires NetInfo into React Query's online manager — queries pause offline and refetch on reconnect.

### Folder Structure

```
src/
├── app/                  Expo Router file-based routes
│   ├── (auth)/           Welcome, login, OTP verification
│   └── (main)/           Tabs (home, learn, analyze, quiz, profile),
│                         lesson, category, premium, notifications, settings
│
├── features/             Domain modules — components, hooks, api, stores, types
│   ├── analyze/          AI photo analysis with history
│   ├── auth/             Apple, Google, email OTP flows
│   ├── category/         Category browsing
│   ├── home/             Home screen, daily challenges, continue learning
│   ├── learn/            Content library, search
│   ├── lesson/           Lesson player, completion, saved lessons
│   ├── notifications/    Inbox, preferences
│   ├── onboarding/       First-run flow
│   ├── profile/          Profile, premium modal, badges, XP, settings
│   ├── quiz/             Quiz sessions, scoring, difficulty
│   └── skills/           Skill tracking and progress
│
├── components/           Shared UI — ui/, feedback/, layout/, navigation/, skeletons/, icons/
├── hooks/                Cross-cutting hooks (notifications, haptics, network, lifecycle, theme)
├── lib/                  Core setup — supabase, sentry, posthog, queryClient, storage
├── services/             Service layer — analytics, revenueCat, notifications, updateService
├── stores/               Global Zustand stores
├── providers/            React context providers (QueryClient, Network)
├── constants/            App config, query keys, design constants
├── theme/                Theme tokens
├── i18n/                 Translation files (en.json, tr.json)
├── types/                TypeScript types and Supabase generated schema
├── utils/                Logger, version helpers, i18n helpers, store reset
└── test-utils/           Jest setup and shared mocks
```

### Feature Module Anatomy

Each feature module follows the same shape — predictable, navigable, isolated:

```
features/<module>/
├── api/          Supabase queries and mutations
├── components/   Feature-specific React components
├── hooks/        React Query hooks and feature logic
├── stores/       Zustand stores (when local state is needed)
├── types/        TypeScript types
└── utils/        Pure helpers
```

## Engineering Highlights

### Hardware-Backed Encrypted Token Storage

Auth tokens live in an encrypted MMKV instance whose key is generated with `crypto.getRandomValues`, persisted in the iOS/Android hardware keychain via `expo-secure-store`, and loaded asynchronously before any auth call. The key never leaves the secure enclave; tokens are encrypted at rest. A custom `StateStorage` adapter wires this into Supabase's auth persistence layer.

### Mutex-Serialized Token Refresh

Concurrent requests can race the same Supabase refresh, producing duplicate refreshes and torn sessions. Refresh is serialized with `async-mutex` behind a custom `processLock` that:

- Races acquisition against a configurable timeout (no indefinite blocking)
- Drains queued acquires after a timeout to prevent mutex leaks
- Mirrors GoTrue's `ProcessLockAcquireTimeoutError` shape (`isAcquireTimeout` flag) so its internal filtering still works
- Bounds the underlying network call with a 20-second `fetchWithTimeout`

### WebCrypto Polyfill for Hermes

Hermes ships without WebCrypto, but Supabase auth needs `crypto.subtle.digest` (Apple nonce SHA-256, internal request IDs) and `crypto.getRandomValues`. Both are polyfilled at module load using `expo-crypto`. The `digest` polyfill serializes calls with a mutex — concurrent native digests in Hermes can SIGSEGV in `JavaScriptTypedArray::~JavaScriptTypedArray` due to a TypedArray allocation race.

### Token-Based Authentication, Three Providers

All three flows skip the browser redirect entirely:

- **Apple** — Native sheet via `expo-apple-authentication`, identity token exchanged through `signInWithIdToken`. Cancellation detected by error code (`ERR_REQUEST_CANCELED`), not string matching.
- **Google** — Native Google Sign-In SDK returns the ID token directly to `signInWithIdToken`. No PKCE, no deep link round-trip. Supabase's "Skip nonce check" is required because the free SDK doesn't expose a nonce parameter.
- **Email OTP** — Passwordless `signInWithOtp` + `verifyOtp`. No magic links to parse, no password to manage. Cooldown and expiry timers persist across navigation.

Login attempts and lockout timestamps are persisted to MMKV — brute-force resistance survives a force-close.

### Push Notifications, End to End

A 7-step registration flow handles device validation, Android channel setup (gated by an MMKV flag so it runs once), permission inspection (no auto-prompt), EAS project ID validation, Expo token acquisition, server registration with mutex locking and exponential backoff, and iOS actionable category registration.

The runtime layer adds:

- **Smart foreground suppression** — banner, sound, and badge are suppressed when the active screen already shows the notification's target.
- **Token refresh throttling** — dual cooldown check (in-memory and MMKV) plus mutex serialization prevents stampedes.
- **Background handler** for off-foreground processing.
- **Daily reminder scheduler** with randomized motivational copy.
- **Token deactivation on logout** with a device ID fallback.
- **Server delivery** through a Supabase Edge Function with receipt verification.

### Sentry PII Scrubbing Pipeline

A `beforeSend` hook intercepts every event before transmission and:

- Filters sensitive headers (`authorization`, `apikey`, `x-supabase-auth`, `x-rc-api-key`)
- Scrubs breadcrumb data containing tokens, secrets, or passwords
- Sanitizes URL query parameters that could carry access tokens

Production-only initialization, with an `ErrorBoundary` providing a user-facing fallback that offers retry and home navigation. React Query errors are forwarded automatically, with query keys sanitized before capture.

### Consent-Gated Analytics

PostHog is initialized lazily and only after the user grants explicit GDPR consent. Autocapture is fully disabled — events are sent through a wrapper service that queues up to 50 events while PostHog is uninitialized and flushes them once it's ready. EU-hosted by configuration. Withdrawal flushes pending events and opts the user out cleanly.

### Force / Soft Update Gate

On launch, the client queries an `app_version` table and decides between blocking force updates and dismissible soft updates by comparing against the running app version. Update copy is localized in TR and EN.

### Subscription Lifecycle

RevenueCat with StoreKit 2 drives the full lifecycle: SDK configuration with optional user ID (avoids anonymous-user windows on iOS), purchase flow with entitlement verification, restore, real-time entitlement updates via the customer info listener, and a server-side Edge Function that consumes RevenueCat webhooks for the source of truth.

### Backend — Edge Functions

Six Deno-based Edge Functions handle server-only concerns:

| Function                | Purpose                             |
| ----------------------- | ----------------------------------- |
| `analyze-body-language` | AI-powered photo analysis (TR / EN) |
| `push-notifications`    | Server-side push delivery           |
| `check-push-receipts`   | Push receipt verification           |
| `revenueCat-webhook`    | Subscription event ingestion        |
| `verify-subscription`   | Server-side entitlement check       |
| `delete-account`        | GDPR account and data deletion      |

## Privacy & Security

- **Row Level Security** — 70+ RLS policies enforce data ownership at the database level, not the client.
- **Encrypted token storage** — hardware-keychain-derived MMKV key, asynchronously initialized before auth.
- **GDPR consent gating** — analytics disabled until explicit opt-in; opt-out flushes and shuts down cleanly.
- **PII scrubbing** — Sentry events sanitized before transmission (headers, breadcrumbs, query params).
- **Account deletion** — server-side Edge Function performs full account and data removal.
- **Apple Privacy Manifest** — declared API usage reasons for UserDefaults, SystemBootTime, DiskSpace, and FileTimestamp. Tracking explicitly disabled.
- **Email-locked entitlements** — only the email address that originally purchased a subscription can use or restore it; no Apple-ID-level cross-transfer.

## Quality & Tooling

- **Tests** — 56 test files across hooks, services, lib utilities, and feature flows, plus a separate Supabase integration suite that runs serially against a real schema.
- **Static analysis** — TypeScript strict mode, ESLint 9 with Expo config, expo-doctor in the pre-production gate.
- **Pre-commit** — Husky + lint-staged run ESLint on changed `.ts/.tsx` only.
- **Schema typing** — Supabase types are generated and checked into the repo; database changes are caught at compile time.
- **Pre-production check** — single `npm run pre-production` runs `depcheck`, `npm audit`, `npm dedupe`, `expo install --fix`, and `expo-doctor` before a release build.
- **OTA updates** — `expo-updates` with `appVersion` runtime policy; native binaries change only when needed.

## Engineering Decisions Worth Calling Out

- **Feature-first, not layer-first.** Modules own their components, hooks, API, stores, and types. Cross-cutting concerns live in `lib/`, `services/`, and `hooks/`.
- **Three storage instances, not one.** Encryption has a real cost — only auth tokens pay it. Preferences and the React Query cache use unencrypted MMKV.
- **Token-based auth everywhere.** No browser redirects, no deep-link parsing, no PKCE state machine for the user to escape from. Each provider hands an ID token straight to Supabase.
- **Server state and client state never mix.** React Query owns server data; Zustand owns UI state. They communicate through hooks, not shared stores.
- **Privacy is a build-time decision.** Sentry scrubbing is in `beforeSend`, PostHog init is gated on a Zustand consent flag, and the Apple Privacy Manifest is generated from `app.config.ts`.

## Contact

This is a showcase repository — source code is private.

For collaboration, employment, or technical questions, reach out at [anl@icodex.dev](mailto:anl@icodex.dev).
