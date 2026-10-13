# MERN Ride-Hailing App

A full-stack Uber-style ride-hailing application built on the MERN stack (MongoDB, Express, React, Node.js). It supports real-time ride booking, live GPS tracking, Socket.IO notifications, a cash payment confirmation flow, and email-OTP password resets.

## Features

### Rider
- JWT-authenticated register / login
- Live map with current GPS location (Google Maps)
- Pickup & destination autocomplete
- Ride booking with vehicle selection and fare estimates
- Real-time ride tracking and status updates
- In-app cash payment confirmation
- Forgot-password reset via email OTP
- Profile management

### Captain (Driver)
- Separate auth flow; set active on login
- Real-time GPS tracking dashboard
- Incoming ride-request popups with distance and fare
- Accept / ignore requests
- OTP-verified ride start
- Payment alerts and ride completion
- Dashboard stats: today's rides, today's earnings, totals

### Core System
- **Socket.IO** real-time events (ride requests, OTP start, payment, ride end)
- GPS location broadcast every 10 seconds
- **Google Maps API** for distance/time, autocomplete, and fare estimation
- Dynamic fares: `base + (km × rate) + (min × rate)` per vehicle type
- **Nodemailer** SMTP for OTP emails
- Role-based route guards for riders and captains

## Tech Stack

**Frontend:** React 18, Vite, Tailwind CSS, GSAP, Socket.IO Client, Axios, React Router v6

**Backend:** Node.js, Express, MongoDB Atlas + Mongoose, Socket.IO, JWT, bcrypt, Nodemailer, Google Maps API

## Getting Started

### Backend

```bash
cd Backend
npm install
```

Create `Backend/.env`:

```env
PORT=3000
MONGODB_URI=<your_mongodb_connection_string>
JWT_SECRET=<your_jwt_secret>
GOOGLE_MAPS_API=<your_google_maps_api_key>

SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your@gmail.com
SMTP_PASS=your_app_password
SMTP_FROM="Ride Hailing Support" <your@gmail.com>
```

```bash
npm run dev
```

### Frontend

```bash
cd frontend
npm install
```

Create `frontend/.env`:

```env
VITE_BASE_URL=http://localhost:3000
```

```bash
npm run dev
```

Then open http://localhost:5173.

## Ride & Payment Flow

```
User books ride
  → Captain receives real-time request popup
  → Captain accepts → OTP sent to rider
  → Captain enters OTP → ride starts
  → Rider pays (cash) → socket emits payment event
  → Captain confirms & finishes ride
  → Both redirect home; captain stats update
```

## Project Structure

```
.
├── Backend/
│   ├── controllers/   # route handlers (user, captain, ride, map)
│   ├── models/        # Mongoose schemas
│   ├── routes/        # Express routers
│   ├── services/      # business logic (ride, maps, email)
│   ├── middlewares/   # auth middleware
│   ├── socket.js      # Socket.IO setup
│   └── server.js      # entry point
└── frontend/
    └── src/
        ├── pages/       # route-level components
        ├── components/  # reusable UI
        └── context/     # User, Captain, Socket contexts
```

## Ideal For

Building a ride-hailing MVP, learning full-stack development with real-time features, or as a MERN + Socket.IO portfolio base to customize.
