# Travel Itinerary Status Checker (Svelte + Vite + Firebase)

A simple web application for users to check the status of their AI-generated travel itineraries. Enter the `jobId` provided by the backend API, and the app will connect to Firestore in real time to display the status and itinerary details.

---

## Live Demo

**Cloudflare Pages URL:** _[https://e94689af.status-checker1.pages.dev/](https://e94689af.status-checker1.pages.dev/)_

---

## Features

- **Real-time updates** from Firestore via `onSnapshot`.
- **Simple interface** with a single jobId input.
- Displays itinerary details (day, theme, activities) once ready.
- Deployable to **Cloudflare Pages**.

---

## Tech Stack

- **Svelte + Vite** — Frontend framework
- **Firebase Firestore** — Real-time database
- **Cloudflare Pages** — Deployment

---

## Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/MahyarManzelati/working_ui.git
cd working_ui
```

### 2. Environment Variables

Copy `.env.example` to `.env` and fill in your Firebase project configuration from the Firebase Console → Project Settings:

```bash
VITE_FIREBASE_API_KEY=your-key
VITE_FIREBASE_AUTH_DOMAIN=your-project.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your-project-id
VITE_FIREBASE_APP_ID=your-app-id
VITE_FIREBASE_STORAGE_BUCKET=your-project.appspot.com
VITE_FIREBASE_MESSAGING_SENDER_ID=your-sender-id
```

> Vite exposes only variables prefixed with `VITE_`.

### 3. Install Dependencies

```bash
npm install
```

### 4. Run Locally

```bash
npm run dev
```

Open [http://localhost:5173](http://localhost:5173) in your browser.

---

## Deployment (Cloudflare Pages)

1. Push your repo to GitHub.
2. In Cloudflare Pages, create a project from your repo.
3. Build settings:

   - Framework preset: **Vite**
   - Build command: `npm run build`
   - Output directory: `dist`

4. Set the same environment variables in Pages (with `VITE_` prefix).
5. Deploy — Cloudflare will give you a public URL.

---

## Usage

1. Obtain a `jobId` from the backend API (`working` repository).
2. Enter it into the UI input field.
3. View real-time status and itinerary updates.

---

## Firestore Security Rules Example

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /itineraries/{jobId} {
      allow read;
      allow write: if false;
    }
  }
}
```

---
