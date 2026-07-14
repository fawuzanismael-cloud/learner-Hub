# LearnHub — Full-Stack Learning Management System

A real backend replaces the browser-only prototype: Node.js/Express API + MongoDB database,
bcrypt-hashed passwords, JWT sessions in secure httpOnly cookies, and exams graded on the
server (so correct answers never reach the browser).

## Tech stack
- **Backend:** Node.js, Express, MongoDB (Mongoose)
- **Auth:** bcryptjs (password hashing), jsonwebtoken (sessions via httpOnly cookie)
- **Security:** helmet (HTTP headers), express-rate-limit (login brute-force protection)
- **Frontend:** plain HTML/CSS/JS served as static files from the same server (no build step)

## Project structure
```
learnhub/
├── server.js              # app entry point
├── config/db.js           # MongoDB connection
├── models/                # User, Course, Progress, Activity schemas
├── routes/                # auth, courses, progress, users, dashboard
├── middleware/auth.js     # JWT verification + role checks
├── utils/                 # token helpers, activity logging, seed data
└── public/                # frontend (index.html, app.js, style.css)
```

## 1. Run it locally

**Prerequisites:** Node.js 18+, and either a local MongoDB or a free MongoDB Atlas cluster.

```bash
cd learnhub
npm install
cp .env.example .env
```

Edit `.env`:
- `MONGO_URI` — your MongoDB connection string
- `JWT_SECRET` — a long random string (generate one: `node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"`)
- `ADMIN_EMAIL` / `ADMIN_PASSWORD` / `ADMIN_NAME` — your admin login (created automatically the first time the server starts)

```bash
npm start
```

Visit `http://localhost:5000`. Sign in with the admin credentials from your `.env` to reach the
admin dashboard, or register a new account to try the learner side.

## 2. Get a free MongoDB database (MongoDB Atlas)

1. Go to mongodb.com/cloud/atlas and create a free account.
2. Create a free **M0 cluster**.
3. Under **Database Access**, add a database user with a password.
4. Under **Network Access**, add `0.0.0.0/0` (allow access from anywhere) so your host can connect.
5. Click **Connect → Drivers**, copy the connection string, and put it in `MONGO_URI` — it looks like:
   `mongodb+srv://<user>:<password>@<cluster>.mongodb.net/learnhub`

## 3. Deploy for free (Render.com)

1. Push this project to a GitHub repository.
2. Go to render.com → **New → Web Service** → connect your repo.
3. Settings:
   - **Build command:** `npm install`
   - **Start command:** `npm start`
4. Under **Environment**, add every variable from `.env.example` with your real values
   (`MONGO_URI`, `JWT_SECRET`, `ADMIN_EMAIL`, `ADMIN_PASSWORD`, `ADMIN_NAME`, and `NODE_ENV=production`).
5. Deploy. Render gives you a live URL like `https://learnhub-xxxx.onrender.com`.
6. Open it, log in with your admin account, and you're live.

(Railway.app, Fly.io, and Vercel-with-serverless-adapter work too — same idea: set the env vars,
point the start command at `npm start`.)

## Security notes — what changed from the prototype
- Passwords are hashed with **bcrypt** (12 rounds), never stored or sent in plain text.
- Sessions use a **JWT in an httpOnly, SameSite cookie** — not readable by JavaScript, so it
  can't be stolen via XSS the way a localStorage token could.
- **Exam grading happens on the server.** The correct answer index is stripped from quiz
  questions before they're ever sent to a learner's browser, and scoring is computed in
  `routes/progress.js`, not in client-side JS.
- **Rate limiting** on `/api/auth/login` (10 attempts / 15 min per IP) slows down password
  guessing.
- **helmet** sets standard security HTTP headers.
- Only pre-seeded accounts get the `admin` role — there's no self-registration path to admin.

### Still worth doing before handling real user data at scale
- Put the app behind HTTPS (Render/Railway do this automatically on their default domains).
- Add email verification and a password-reset flow.
- Add server-side input validation library (e.g. `zod` or `joi`) for stricter request checking.
- Take regular MongoDB Atlas backups (Atlas free tier has limited backup options — paid tiers add more).
- Consider adding 2FA for the admin account.

## API summary
| Method | Route | Access | Purpose |
|---|---|---|---|
| POST | /api/auth/register | public | create a learner account |
| POST | /api/auth/login | public (rate-limited) | sign in |
| POST | /api/auth/logout | logged in | sign out |
| GET | /api/auth/me | logged in | current user |
| GET | /api/courses | public | list courses (answers hidden unless admin) |
| POST | /api/courses | admin | create course |
| DELETE | /api/courses/:id | admin | delete course |
| POST | /api/courses/:id/materials | admin | upload video/note |
| DELETE | /api/courses/:id/materials/:materialId | admin | remove material |
| POST | /api/courses/:id/quiz | admin | add exam question |
| DELETE | /api/courses/:id/quiz/:questionId | admin | remove question |
| POST | /api/progress/:courseId/materials/:materialId/view | learner | mark material viewed |
| POST | /api/progress/:courseId/exam/submit | learner | submit exam (graded server-side) |
| GET | /api/progress/me | learner | own progress + results |
| GET | /api/users | admin | list learners |
| DELETE | /api/users/:id | admin | remove learner |
| GET | /api/dashboard/stats | admin | overview stats |
| GET | /api/dashboard/activity | admin | recent activity feed |
| GET | /api/dashboard/results | admin | all learners' exam results |
