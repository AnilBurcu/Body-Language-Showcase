# Body-Language-Showcase

A cross-platform mobile application for learning and practicing body language analysis, built with React Native and Expo.

![React Native](https://img.shields.io/badge/React%20Native-0.81.5-61DAFB?style=flat-square&logo=react)
![Expo](https://img.shields.io/badge/Expo-54-000020?style=flat-square&logo=expo)
![TypeScript](https://img.shields.io/badge/TypeScript-5.9-3178C6?style=flat-square&logo=typescript)
![Supabase](https://img.shields.io/badge/Supabase-2.98-3FCF8E?style=flat-square&logo=supabase)
![RevenueCat](https://img.shields.io/badge/RevenueCat-9.12-F25A5A?style=flat-square)
![Platform](https://img.shields.io/badge/Platform-iOS%20%7C%20Android-lightgrey?style=flat-square)

## Overview

ReadBody is a body language education app available on both the App Store and Google Play. Users learn to read nonverbal cues through structured lessons, interactive quizzes, and AI-powered image analysis. The app supports Turkish and English, with a premium subscription tier for advanced content.

Built as a solo project from the ground up, covering everything from database schema design and Edge Functions to native authentication flows and in-app purchases. The codebase follows a feature-based modular architecture with 11 domain modules, backed by Supabase with 72+ Row Level Security policies.

Core features include multi-provider authentication (Apple, Google, Email), a full subscription system through RevenueCat with StoreKit 2, push notifications with actionable categories, and a privacy-first analytics pipeline with GDPR consent gating.

## Screenshots

<!-- Add 3-4 app screenshots here -->

## Tech Stack

| Category           | Technology                              | Purpose                                                                       |
| ------------------ | --------------------------------------- | ----------------------------------------------------------------------------- |
| **Mobile**         | React Native 0.81.5                     | Cross-platform UI                                                             |
| **Mobile**         | Expo ~54.0.33 (EAS)                     | Build, deploy, OTA updates                                                    |
| **Mobile**         | Expo Router ~6.0.23                     | File-based routing with typed routes and async route loading                  |
| **Mobile**         | Hermes                                  | JavaScript engine                                                             |
| **State**          | Zustand ^5.0.9                          | Client state management (13 stores)                                           |
| **State**          | TanStack React Query ^5.90.12           | Server state, caching, smart retry (38+ hooks)                                |
| **State**          | react-native-mmkv ^4.1.0                | Persistent storage (3 instances: encrypted, app, cache)                       |
| **Backend**        | Supabase ^2.98.0                        | Database, auth, Edge Functions, RLS                                           |
| **Backend**        | Supabase Edge Functions                 | Push notifications, AI analysis (EN/TR), account deletion, RevenueCat webhook |
| **Auth**           | expo-apple-authentication               | Native Apple Sign-In (iOS)                                                    |
| **Auth**           | expo-auth-session + expo-web-browser    | Google OAuth with PKCE flow                                                   |
| **Auth**           | Supabase Auth                           | Email/password, session management                                            |
| **Monetization**   | react-native-purchases ^9.12.0          | RevenueCat SDK with StoreKit 2                                                |
| **Infrastructure** | @sentry/react-native ~7.2.0             | Error tracking with PII scrubbing                                             |
| **Infrastructure** | posthog-react-native ^4.37.2            | Product analytics (EU host, GDPR consent-gated)                               |
| **Infrastructure** | @react-native-community/netinfo         | Network-aware query management                                                |
| **Infrastructure** | async-mutex ^0.5.0                      | Token refresh and notification registration locking                           |
| **UI**             | react-native-reanimated ~4.1.1          | Animations (23 files)                                                         |
| **UI**             | react-native-gesture-handler ~2.28.0    | Gesture interactions                                                          |
| **UI**             | @shopify/flash-list 2.0.2               | High-performance lists (9 files)                                              |
| **UI**             | expo-image ~3.0.11                      | Optimized image rendering                                                     |
| **UI**             | expo-haptics                            | Tactile feedback across auth and interaction flows                            |
| **UI**             | expo-linear-gradient                    | Gradient backgrounds                                                          |
| **UI**             | expo-blur                               | Blur effects                                                                  |
| **UI**             | react-native-keyboard-controller 1.18.5 | Keyboard-aware form handling                                                  |
| **UI**             | react-hook-form ^7.71.2 + zod ^3.25.76  | Form state and schema validation                                              |
| **UI**             | i18next ^25.7.3 + react-i18next         | Localization (TR/EN)                                                          |
| **Notifications**  | expo-notifications ~0.32.16             | Push and local notifications                                                  |
| **DX**             | Husky + lint-staged                     | Pre-commit linting                                                            |
| **DX**             | ESLint ^9.0.0                           | Code quality                                                                  |

## Architecture

### Folder Structure

```
src/
  app/                  # Expo Router file-based routes
    (auth)/             #   Auth group: login, register, welcome, verify-email, forgot/reset-password
    (main)/             #   Main group: tabs (home, learn, analyze, quiz, profile), modals, details
  features/             # Domain modules (see below)
  components/           # Shared UI components (Button, Skeleton, ProgressBar, ErrorBoundary, Toast...)
  hooks/                # Shared hooks (useNotifications, useHaptics, useBadges, useNetworkStatus...)
  lib/                  # Core setup (supabase, sentry, posthog, queryClient, storage)
  services/             # Service layer (analytics, revenueCat, notifications, updateService)
  stores/               # Global Zustand stores
  providers/            # React context providers (QueryClient, Network)
  constants/            # Config, styles, query keys
  theme/                # Theme configuration
  types/                # TypeScript types and Supabase generated types
  utils/                # Utility functions (logger, version, i18n-helper, resetAllStores)
  i18n/                 # Localization files (en.json, tr.json)
  test-utils/           # Test setup and mocks
supabase/
  functions/            # Edge Functions (5)
  migrations/           # Database schema with RLS policies
scripts/                # Content import pipeline
```

### Feature Modules

The codebase is organized into 11 feature modules, each containing its own components, hooks, API layer, stores, and types:

| Module          | Domain                                               |
| --------------- | ---------------------------------------------------- |
| `auth`          | Authentication flows (Apple, Google, email/password) |
| `home`          | Home screen, daily challenges, continue learning     |
| `learn`         | Content browsing, category lists                     |
| `lesson`        | Lesson detail, completion tracking, saved lessons    |
| `category`      | Category screens and navigation                      |
| `quiz`          | Quiz sessions, scoring, difficulty levels            |
| `analyze`       | AI-powered image analysis with history               |
| `profile`       | User profile, premium modal, settings, badges, XP    |
| `notifications` | Notification inbox, preferences, mark-read           |
| `onboarding`    | Onboarding flow                                      |
| `skills`        | Skill tracking and progress                          |

### Data Flow

The app uses a three-layer state architecture:

- **Zustand (13 stores)** handles client-side state: auth session, UI state (toast, notification banners), network status, consent flags, and feature-specific state (quiz progress, analysis state, lesson navigation). Stores persist to MMKV via a custom Zustand storage adapter.

- **TanStack React Query (38+ hooks)** manages all server state: data fetching, caching (5min stale, 10min GC), mutations, and error handling. The QueryClient integrates with Sentry for automatic error capture and implements smart retry logic that skips 401/403/404 responses.

- **MMKV (3 instances)** provides the persistence layer. `secureStorage` is encrypted with a key derived from the hardware keychain via expo-secure-store and holds auth tokens. `appStorage` stores user preferences, onboarding state, rate limit data, and notification config. `cacheStorage` handles query cache persistence.

A `NetworkProvider` wraps the app and connects NetInfo to React Query's online manager, pausing queries when offline and refetching when connectivity returns.

## Key Features

### Authentication

Three authentication providers are implemented:

- **Apple Sign-In** -- Native iOS authentication via `expo-apple-authentication`. Cancellation is detected by error code (`ERR_REQUEST_CANCELED`), not string matching. (`src/features/auth/hooks/useAppleAuth.ts`)

- **Google OAuth** -- PKCE flow using `expo-auth-session` to generate a code challenge URL, `expo-web-browser` to open the consent screen, and Supabase to exchange the authorization code for a session. The code verifier is stored in MMKV automatically. (`src/features/auth/hooks/useGoogleAuth.ts`)

- **Email/Password** -- Full flow including registration, login, email verification, forgot password, and password reset. All forms use `react-hook-form` with `zod` schema validation and `react-native-keyboard-controller` for keyboard handling. (`src/app/(auth)/`)

- **Login Rate Limiting** -- Attempt count and lockout timestamp are persisted in MMKV to survive app restarts. (`src/lib/storage.ts:84-104`)

### Monetization

RevenueCat SDK (StoreKit 2 on iOS) handles the full subscription lifecycle:

- SDK configuration with optional user ID to prevent anonymous windows on iOS (`src/services/revenueCat.ts:33-70`)
- Purchase flow with entitlement verification (`src/services/revenueCat.ts:107-127`)
- Restore purchases (`src/services/revenueCat.ts:130-141`)
- Customer info listener for real-time entitlement updates (`src/services/revenueCat.ts:163-168`)
- Server-side webhook via Edge Function (`supabase/functions/revenueCat-webhook/index.ts`)
- Premium modal screen (`src/app/(main)/premium-modal.tsx`)
- Custom hook for React Query integration (`src/features/profile/hooks/useRevenueCat.ts`)

### Push Notifications

A 7-step registration flow handles the full push notification lifecycle (`src/services/notifications/notificationService.ts:314-369`):

1. Physical device validation
2. Android channel setup (messages, updates, promotions) -- single-time via MMKV flag
3. Permission check (no auto-request)
4. EAS Project ID validation
5. Expo push token acquisition
6. Server registration with mutex locking and retry with exponential backoff
7. iOS actionable category registration (message, update, promotion, reminder)

Additional notification features:

- **Smart foreground handler** -- suppresses banner and sound when the user is already viewing the target screen (`src/services/notifications/notificationService.ts:163-178`)
- **Token refresh** with cooldown throttling (in-memory + MMKV dual check) and mutex serialization (`src/services/notifications/notificationService.ts:376-413`)
- **Background handler** for processing notifications when the app is not in the foreground (`src/services/notifications/backgroundHandler.ts`)
- **Daily reminder scheduler** with randomized motivational messages (`src/services/notifications/notificationService.ts:578-601`)
- **Token deactivation** on logout with device ID fallback (`src/services/notifications/notificationService.ts:417-446`)
- **Push notification Edge Function** for server-side delivery (`supabase/functions/Push-notifications/index.ts`)

### Analytics and Monitoring

- **Sentry** -- Production-only error tracking with a `beforeSend` hook that scrubs authorization headers, API keys, tokens, secrets, and passwords from both request headers and breadcrumbs. URL query parameters containing tokens are also filtered. (`src/lib/sentry.ts:11-76`)

- **Sentry ErrorBoundary** -- Wraps the app with a user-facing fallback UI that offers retry and navigation to home. (`src/components/feedback/ErrorBoundary.tsx`)

- **PostHog** -- Initialized lazily and only after GDPR analytics consent. All autocapture is disabled; events are sent explicitly through a wrapper service. The analytics service queues up to 50 events when PostHog is not yet initialized and flushes them automatically once it becomes ready. EU-hosted. (`src/lib/posthog.ts`, `src/services/analytics.ts`)

- **React Query error capture** -- Both query and mutation errors are automatically forwarded to Sentry with sanitized query keys (tokens, passwords, and API keys are redacted). (`src/lib/queryClient.ts:33-60`)

### Offline and Storage

Three MMKV storage instances serve different security and performance needs (`src/lib/storage.ts`):

| Instance        | Encrypted                   | Purpose                                                   |
| --------------- | --------------------------- | --------------------------------------------------------- |
| `secureStorage` | Yes (hardware keychain key) | Supabase auth tokens                                      |
| `appStorage`    | No                          | Preferences, onboarding, rate limits, notification config |
| `cacheStorage`  | No                          | Query cache persistence                                   |

The encrypted instance derives its key from `expo-secure-store` (hardware keychain) and is initialized asynchronously before any auth operations. A custom `StateStorage` adapter connects it to Supabase's auth persistence layer.

Network-aware query management pauses React Query operations when offline and resumes on reconnection (`src/providers/NetworkProvider.tsx`, `src/hooks/useOnlineManager.ts`).

### Localization

Turkish and English are supported via `i18next` and `react-i18next`. Translation files are located at `src/i18n/en.json` and `src/i18n/tr.json`.

### Force / Soft Update System

The app checks an `app_version` table in Supabase on launch to determine if a force update (blocking) or soft update (dismissible) is required, comparing against the current app version. Localized update messages are supported in both TR and EN. (`src/services/updateService.ts`)

### Content Import Pipeline

A CLI-based content import tool (`scripts/importContent.ts`) supports validate, dry-run, clean, and force modes for managing lesson and quiz content. (`package.json:16-19`)

## Technical Challenges

### Mutex-Based Token Refresh Serialization

Supabase auth token refresh is serialized using `async-mutex` to prevent race conditions when multiple concurrent requests trigger a refresh simultaneously. Lock acquisition is bounded by a timeout to prevent indefinite blocking, while network calls are bounded by a 20-second fetch timeout. (`src/lib/supabase.ts:49-71`)

### Encrypted MMKV with SecureStore Key Derivation

Auth tokens are stored in an encrypted MMKV instance. The encryption key is generated using `crypto.getRandomValues`, stored in the hardware keychain via `expo-secure-store`, and retrieved on subsequent launches. This ensures tokens are encrypted at rest with a key that never leaves the secure enclave. (`src/lib/storage.ts:28-42`)

### PKCE and crypto.subtle Polyfill for Hermes

Hermes does not ship WebCrypto. The Supabase client requires `crypto.subtle.digest` for PKCE SHA-256 code challenges and `crypto.getRandomValues` for code verifier generation. Both are polyfilled using `expo-crypto`'s native implementations at module level before the Supabase client is created. (`src/lib/supabase.ts:9-43`)

### Smart Foreground Notification Suppression

The notification handler tracks the currently active screen and compares it against the notification's target screen. If the user is already viewing the relevant content, the banner, sound, and badge update are suppressed to avoid redundant alerts. (`src/services/notifications/notificationService.ts:163-178`)

### Login Rate Limiting with MMKV Persistence

Login attempts and lockout timestamps are persisted in MMKV, surviving app restarts and preventing brute-force attempts even if the app is force-closed during a lockout period. (`src/lib/storage.ts:84-104`)

### Sentry PII Scrubbing Pipeline

A `beforeSend` hook processes every error event before transmission, filtering sensitive headers (`authorization`, `apikey`, `x-supabase-auth`, `x-rc-api-key`), scrubbing breadcrumb data containing tokens, secrets, or passwords, and sanitizing URL query parameters that may contain access tokens. (`src/lib/sentry.ts:11-65`)

## Privacy and Security

- **Row Level Security** -- 72+ RLS policies enforced at the database level across all tables (`supabase/migrations/20260312000000_full_schema.sql`)
- **GDPR Consent** -- PostHog analytics is initialized only after explicit user consent; shutdown flushes events and opts out (`src/lib/posthog.ts:26-70`)
- **PII Scrubbing** -- Sentry events are sanitized before transmission, removing auth headers, tokens, and sensitive query parameters (`src/lib/sentry.ts`)
- **Encrypted Storage** -- Auth tokens are stored in MMKV encrypted with a hardware keychain-derived key (`src/lib/storage.ts`)
- **Account Deletion** -- Server-side Edge Function for complete account and data removal (`supabase/functions/delete-account/index.ts`)
- **Apple Privacy Manifest** -- Declared API usage reasons for UserDefaults, SystemBootTime, DiskSpace, and FileTimestamp (`app.config.ts:81-102`)

---

_This is a showcase repository. Source code is private._
_For inquiries, reach out via [LinkedIn](https://www.linkedin.com/in/anil-burcu/)._
