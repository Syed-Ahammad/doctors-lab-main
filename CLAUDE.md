# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Doctors Portal is a full-stack dental appointment booking system. Patients book appointments by specialty and date; admins manage users, doctors, and appointment options. Payments are processed via Stripe.

## Repository Structure

```
doctors-lab-main/
├── Client/   # React 18 frontend (Create React App)
└── Server/   # Node.js/Express backend
```

## Development Commands

### Frontend (`Client/`)
```bash
npm start          # Dev server on http://localhost:3000
npm run build      # Production build
npm test           # Run tests (React Testing Library / Jest)
npm test -- --testPathPattern=<filename>  # Run a single test file
```

### Backend (`Server/`)
```bash
npm run start-dev  # Dev server with nodemon on port 5000
npm start          # Production start (node index.js)
```

## Environment Variables

**`Client/.env.local`** — Firebase config keys, Stripe publishable key (`REACT_APP_STRIPE_PK`), ImgBB API key, and API base URL pointing to the backend.

**`Server/.env`** — `DB_USER`, `DB_PASS` (MongoDB Atlas), `ACCESS_TOKEN` (JWT secret), `STRIPE_SECRET_KEY`.

## Architecture

### Auth Flow (dual-token system)
1. Firebase handles identity (email/password). `AuthProvider` (`src/contexts/AuthProvider.js`) wraps the app and exposes `user`, `loading`, and auth methods via `AuthContext`.
2. After Firebase login/signup, `useToken` hook calls `GET /jwt?email=` on the backend. The backend verifies the email exists in MongoDB, then issues a 1-hour JWT stored in `localStorage` as `accessToken`.
3. All protected API calls send `Authorization: Bearer <token>` in headers. The server's `verifyJWT` middleware decodes it; `verifyAdmin` then checks MongoDB for `role === 'admin'`.

### Role System
- First user registered is automatically made admin (checked server-side in `POST /users`).
- `useAdmin` hook queries `GET /users/admin/:email` and returns `[isAdmin, isAdminLoading]`.
- `AdminRoute` wraps admin-only dashboard pages; `PrivateRoute` wraps all authenticated pages.

### Data Fetching Pattern
React Query (`@tanstack/react-query`) is used throughout the frontend. Queries are keyed by route + filter (e.g., `['bookings', user?.email]`). Appointment slot availability is refetched on date change by invalidating the `appointmentOptions` query key.

### Appointment Slot Availability
Two API versions exist:
- `GET /appointmentOptions?date=` — JS-side filtering (v1, kept for reference)
- `GET /v2/appointmentOptions?date=` — MongoDB aggregation pipeline with `$lookup` + `$setDifference` (v2, preferred, used by the frontend)

### Payment Flow
1. User clicks "Pay" on `MyAppointment` → navigates to `/dashboard/payment/:id`.
2. Route loader fetches the booking from `GET /bookings/:id`.
3. `Payment` page calls `POST /create-payment-intent` with the price → receives `clientSecret`.
4. Stripe `CardElement` confirms the payment; on success, `POST /payments` saves the transaction and patches the booking (`paid: true`, `transactionId`).

### Image Upload
Doctor profile photos are uploaded to ImgBB (external service) from `AddDoctor`. The returned image URL is stored in MongoDB's `doctors` collection.

### Key Patterns
- Dashboard layout uses DaisyUI drawer for mobile sidebar. Sidebar links are conditionally rendered based on `isAdmin`.
- `ConfirmationModal` (shared component) is reused for delete confirmations across admin pages.
- `react-hot-toast` is used for all user-facing success/error feedback.
- Email notifications (nodemailer + Mailgun) are implemented but fully commented out in `Server/index.js`.
