# Project Scope — Doctors Portal

## Project Summary

Doctors Portal is a full-stack dental clinic management system that allows patients to book appointments online, make payments, and track their appointment history. Clinic admins can manage doctors, users, and appointment options through a dedicated dashboard.

---

## Current Scope (Implemented)

### Authentication & Authorization
- Firebase email/password registration and login
- JWT-based API security (1-hour expiry, stored in localStorage)
- Role-based access: `patient` and `admin`
- First registered user is automatically assigned admin role
- Protected routes: `PrivateRoute` (authenticated) and `AdminRoute` (admin-only)

### Appointment Booking
- 6 dental specialties: Teeth Orthodontics, Cosmetic Dentistry, Teeth Cleaning, Cavity Protection, Pediatric Dental, Oral Surgery
- Date-based slot browsing with real-time availability (MongoDB aggregation pipeline)
- One booking per patient per treatment per day (duplicate prevention)
- Modal-based booking form with auto-populated patient info

### Payment
- Stripe card payment integration (PaymentIntent API)
- Transaction ID stored against each booking
- Booking marked `paid: true` after successful payment
- Payment history accessible via user dashboard

### User Dashboard
- View all personal appointments with payment status
- Pay for unpaid appointments (links to Stripe checkout page)

### Admin Dashboard
- View and manage all registered users
- Promote any user to admin role
- Add doctors with name, email, specialty, and photo (ImgBB upload)
- View and delete doctors (with confirmation modal)

### Infrastructure
- Backend deployed on Vercel (`doctors-lab-server-bice.vercel.app`)
- MongoDB Atlas (cloud database)
- Frontend bootstrapped with Create React App + Tailwind CSS + DaisyUI

---

## Known Gaps in Current Implementation

| Area | Issue |
|------|-------|
| Email notifications | Fully coded with Mailgun/Nodemailer but commented out |
| Google OAuth | Button exists in Login UI but not wired up |
| Duplicate user prevention | `POST /users` has a TODO — same email can be inserted twice |
| JWT expiry | Token expires in 1 hour with no refresh mechanism |
| Input sanitization | No server-side validation on request bodies |
| Image storage | Relies on third-party ImgBB (no self-hosted fallback) |

---

## Future Advanced Features

### 1. Doctor Availability & Scheduling
- Let each doctor define their own working hours and days off
- Patients select a specific doctor when booking, not just a specialty
- Calendar view for doctors to see their daily/weekly schedule
- Block-out dates for vacations or public holidays

### 2. Real-Time Notifications
- Enable the existing email notification code (Mailgun/Nodemailer) for booking confirmations and reminders
- SMS notifications via Twilio (24-hour appointment reminders)
- In-app notification bell with unread badge using WebSockets or Firebase Realtime Database

### 3. Video Consultation (Telemedicine)
- Add a "Virtual" appointment type alongside in-clinic
- Integrate WebRTC (e.g., Daily.co or Twilio Video) for in-browser video calls
- Generate a secure join link and send it to patient and doctor via email before the slot

### 4. Patient Medical Records
- Store and display patient dental history per visit (notes, diagnosis, prescriptions)
- File/image upload for X-rays and reports (store on AWS S3 or Cloudinary)
- Doctors can view a patient's full history before a consultation

### 5. Prescription & Invoice Management
- Doctors generate digital prescriptions after each visit
- Auto-generate downloadable PDF invoices per payment (using `pdfkit` or `react-pdf`)
- Email invoice automatically after successful Stripe payment

### 6. Advanced Admin Analytics Dashboard
- Charts for appointments per day/week/month (Recharts or Chart.js)
- Revenue tracking with Stripe webhook integration
- Most booked specialties and busiest time slots
- Patient retention rates and new vs. returning patient metrics

### 7. Review & Rating System
- Patients leave a star rating and written review after a completed appointment
- Reviews displayed on the Home page (replacing static testimonials)
- Admin can moderate and respond to reviews

### 8. Multi-Clinic / Branch Support
- Support multiple clinic locations under one portal
- Patients select a branch when booking
- Admins scoped to a specific branch; super-admin sees all branches

### 9. Google OAuth & Social Login
- Wire up the existing Google login button using `signInWithPopup` (Firebase already installed)
- Add Facebook login as a secondary option
- Handle account merging if the same email is registered via email/password and OAuth

### 10. Token Refresh & Session Management
- Implement refresh tokens with a longer expiry (7 days) stored in an `httpOnly` cookie
- Short-lived access tokens (15 minutes) auto-refreshed in the background
- "Remember me" toggle on the login page

### 11. Progressive Web App (PWA)
- Add a service worker for offline support and caching of static assets
- "Add to Home Screen" prompt for mobile users
- Push notifications for appointment reminders via Web Push API

### 12. Search, Filter & Pagination
- Search patients by name or email in the admin AllUsers page
- Filter appointments by date range, specialty, or payment status
- Server-side pagination on all admin list pages (currently loads all records)

### 13. Waitlist System
- When all slots for a date/specialty are booked, patients can join a waitlist
- Automatic notification and slot assignment if a cancellation occurs

### 14. Appointment Cancellation & Rescheduling
- Patients can cancel or reschedule up to 24 hours before the appointment
- Cancelled slots are returned to available inventory immediately
- Partial or full refund via Stripe Refund API on cancellation

---

## Tech Stack Extensions (Recommended for Future Features)

| Feature | Suggested Technology |
|---------|---------------------|
| Real-time updates | Socket.io or Firebase Realtime Database |
| File/image storage | AWS S3 or Cloudinary |
| PDF generation | `react-pdf` / `pdfkit` |
| SMS notifications | Twilio |
| Video calls | Daily.co or Twilio Video |
| Charts/analytics | Recharts or Chart.js |
| Push notifications | Web Push API + service worker |
| Server-side validation | `express-validator` or `joi` |
| API rate limiting | `express-rate-limit` |
| Testing | Jest + Supertest (backend), React Testing Library (frontend) |
