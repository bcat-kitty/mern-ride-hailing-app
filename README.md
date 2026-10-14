# MERN Ride-Hailing App

A full-stack, Uber-style ride-hailing application built on the **MERN stack** (MongoDB, Express, React, Node.js) with real-time booking and tracking over Socket.IO. It implements both sides of the marketplace — riders and drivers ("captains") — including live GPS, a cash payment confirmation flow, and email-OTP password resets.

## What it is

A working two-sided ride-hailing MVP: a rider books a trip, a nearby captain receives the request in real time, accepts it, verifies an OTP to start the ride, and both sides are kept in sync through completion and payment. It's a realistic base for a local ride-hailing product or a portfolio-grade example of real-time full-stack development.

## Features

### Rider
- JWT-authenticated register / login
- Live map with current GPS location (Google Maps)
- Pickup & destination autocomplete
- Vehicle selection with fare estimates, then ride confirmation
- Real-time ride tracking and status updates
- In-app cash payment confirmation
- Forgot-password reset via email OTP
- Profile management

### Captain (driver)
- Separate auth flow; set **active** on login
- Real-time GPS tracking dashboard
- Incoming ride-request popups (pickup, destination, distance, fare)
- Accept / ignore requests
- OTP-verified ride start
- Payment alert (toast + browser notification) and ride completion
- Dashboard stats: today's rides, today's earnings, lifetime totals

## Architecture

### High level

```
┌────────────┐   REST (Axios) + WebSocket (Socket.IO)   ┌───────────────┐
│  React/Vite │ ◄──────────────────────────────────────► │ Express/Node   │
│  frontend   │                                           │ backend        │
│  (rider +   │                                           │  ├─ REST API    │
│   captain   │                                           │  ├─ Socket.IO   │
│   apps)     │                                           │  └─ services    │
└────────────┘                                           └───────┬────────┘
       │                                                         │
       ▼                                                         ▼
 Google Maps API                                          MongoDB Atlas
 (autocomplete, distance,                                 (users, captains,
  fare, live map)                                          rides)
```

Two transport layers run in parallel: **REST** for request/response actions (auth, profile, fare calc) and **Socket.IO** for everything real-time (ride requests, OTP start, payment, ride end, and captain location broadcast every ~10 seconds).

### Backend (`Backend/`)

```
controllers/   # route handlers (user, captain, ride, map)
models/        # Mongoose schemas (user, captain, ride)
routes/        # Express routers
services/      # business logic (ride lifecycle, maps, email)
middlewares/   # JWT auth guards (role-based: user vs captain)
socket.js      # Socket.IO setup & real-time event handlers
server.js      # app entry point
```

- **Auth:** JWT tokens, bcrypt password hashing, role-based protected routes.
- **Fares:** computed dynamically as `base + (km × rate) + (min × rate)` per vehicle type, using the Google Maps distance matrix.
- **Email:** Nodemailer over SMTP delivers OTPs for ride start and password resets.

### Frontend (`frontend/`)

```
src/
├── pages/        # route-level screens (Home, Riding, CaptainRiding, …)
├── components/   # reusable UI (panels, map, ride popups)
├── context/      # React contexts: User, Captain, Socket
└── App.jsx       # route definitions
```

GSAP drives the sliding-panel animations; React contexts hold the authenticated user/captain and the shared socket connection.

### Ride & payment flow

```
Rider books ride
  → Captain receives real-time request popup
  → Captain accepts → OTP sent to rider
  → Captain enters OTP → ride starts
  → Rider pays (cash) → socket emits payment-made
  → Captain gets alert → confirms & finishes ride
  → Both redirect home; captain stats update instantly
```

## What stands out

- **Genuinely real-time, both sides** — ride requests, OTP start, payment, and completion are all socket events, with captain GPS broadcast on an interval, so rider and captain views stay in lockstep without polling.
- **Complete marketplace loop** — not just booking, but accept/reject, OTP-verified start, payment confirmation, and live-updating driver earnings.
- **Production-minded auth** — separate rider/captain identities, role-based route guards, hashed passwords, and a real email-OTP reset flow.
- **Real Google Maps integration** — autocomplete, distance/time, and fare estimation, not mocked placeholders.

## Tech stack

**Frontend:** React 18, Vite, Tailwind CSS, GSAP, Socket.IO Client, Axios, React Router v6.
**Backend:** Node.js, Express, MongoDB Atlas + Mongoose, Socket.IO, JWT, bcrypt, Nodemailer, Google Maps API.

## How to run

### Prerequisites

- Node.js 18+ and npm
- A MongoDB Atlas connection string (or local MongoDB)
- A Google Maps API key
- SMTP credentials for OTP email (Gmail app password recommended)

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
npm run dev     # auto-reload (or: npm start)
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

Open http://localhost:5173. Register as a rider in one browser and as a captain in another (or an incognito window) to exercise the full real-time flow.

## Ideal for

Building a ride-hailing MVP for a local market, learning full-stack development with real-time features, or as a MERN + Socket.IO portfolio project to customize.
