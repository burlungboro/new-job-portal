# 🚀 Job Portal — Implementation Plan

## Overview

A full-stack **Job Portal** web application with two distinct user roles:

- **Recruiters** — Post, manage, and track job listings
- **Job Seekers** — Browse, filter, and apply for jobs across multiple categories

**Current Stack (already scaffolded):**

| Layer | Technology |
|-------|----------|
| Frontend | React 19 + Vite 8 + TailwindCSS 4 |
| Backend | Node.js + Express 5 |
| Database | MySQL 2 (connection pool, promise-based) |
| Auth | JWT (secret already in `.env.example`) |
| Dev Server | Vite (client) + Nodemon (server) |

---

## 🗂️ Job Categories

The portal will support the following job categories (filterable):

| # | Category |
|---|----------|
| 1 | Technology & IT |
| 2 | Marketing & Sales |
| 3 | Design & Creative |
| 4 | Finance & Accounting |
| 5 | Healthcare & Medicine |
| 6 | Education & Training |
| 7 | Human Resources |
| 8 | Legal & Compliance |
| 9 | Engineering |
| 10 | Customer Support |
| 11 | Operations & Logistics |
| 12 | Remote / Work From Home |

---

## 👥 User Roles & Features

### 🔵 Job Seeker (Candidate)

| Feature | Description |
|---------|-------------|
| Register / Login | Email + password authentication with JWT |
| Profile Setup | Name, photo, resume upload, skills, experience |
| Browse Jobs | Paginated list of all active job listings |
| **Filter & Search** | Filter by category, location, job type, salary range, experience level, date posted |
| Save / Bookmark Jobs | Save jobs to a personal wishlist |
| Apply for Jobs | One-click apply with uploaded resume + cover letter |
| Application Tracker | View status of all submitted applications |
| Job Alerts | Email notification when new jobs match saved preferences |

### 🟠 Recruiter (Employer)

| Feature | Description |
|---------|-------------|
| Register / Login | Separate recruiter account type |
| Company Profile | Logo, company name, website, about section |
| Post a Job | Form with title, description, category, type, location, salary, deadline |
| Manage Listings | Edit, pause, close, or delete job postings |
| View Applicants | See all applicants per job with resume & cover letter |
| Application Actions | Mark applicants as Shortlisted / Rejected / Under Review |
| Dashboard | Stats — total jobs posted, total applicants, open positions |

---

## 🔎 Filters & Search System

The job listing page will have a powerful filtering panel:

```
Search Bar:     [🔍 Job title or keyword...]
────────────────────────────────────────────────
Category:       [ All | Technology | Design | Finance | ... ]
Job Type:       [ Full-time | Part-time | Contract | Internship | Freelance ]
Location:       [ City / State / Remote ]
Experience:     [ Fresher | 1-3 yrs | 3-5 yrs | 5+ yrs ]
Salary Range:   [ ₹0 ─────●───── ₹20 LPA ]  (range slider)
Date Posted:    [ Any | Today | Last 7 days | Last 30 days ]
────────────────────────────────────────────────
[ Apply Filters ]  [ Clear All ]
```

> **Implementation**: Filters are passed as query parameters to `GET /api/jobs`. The server applies `WHERE` clauses dynamically based on provided filters.

---

## 🗄️ Database Schema (MySQL)

### `users`
```sql
CREATE TABLE users (
  id          INT AUTO_INCREMENT PRIMARY KEY,
  name        VARCHAR(100) NOT NULL,
  email       VARCHAR(150) UNIQUE NOT NULL,
  password    VARCHAR(255) NOT NULL,
  role        ENUM('seeker', 'recruiter') NOT NULL,
  avatar_url  VARCHAR(500),
  created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### `recruiter_profiles`
```sql
CREATE TABLE recruiter_profiles (
  id           INT AUTO_INCREMENT PRIMARY KEY,
  user_id      INT NOT NULL REFERENCES users(id),
  company_name VARCHAR(200) NOT NULL,
  company_logo VARCHAR(500),
  website      VARCHAR(300),
  about        TEXT,
  location     VARCHAR(200)
);
```

### `seeker_profiles`
```sql
CREATE TABLE seeker_profiles (
  id          INT AUTO_INCREMENT PRIMARY KEY,
  user_id     INT NOT NULL REFERENCES users(id),
  resume_url  VARCHAR(500),
  skills      TEXT,
  experience  VARCHAR(50),
  headline    VARCHAR(300)
);
```

### `categories`
```sql
CREATE TABLE categories (
  id    INT AUTO_INCREMENT PRIMARY KEY,
  name  VARCHAR(100) UNIQUE NOT NULL,
  icon  VARCHAR(100)
);
```

### `jobs`
```sql
CREATE TABLE jobs (
  id              INT AUTO_INCREMENT PRIMARY KEY,
  recruiter_id    INT NOT NULL REFERENCES users(id),
  category_id     INT NOT NULL REFERENCES categories(id),
  title           VARCHAR(200) NOT NULL,
  description     TEXT NOT NULL,
  requirements    TEXT,
  job_type        ENUM('full-time', 'part-time', 'contract', 'internship', 'freelance') NOT NULL,
  location        VARCHAR(200),
  is_remote       BOOLEAN DEFAULT FALSE,
  salary_min      DECIMAL(10,2),
  salary_max      DECIMAL(10,2),
  experience_req  VARCHAR(50),
  deadline        DATE,
  status          ENUM('active', 'paused', 'closed') DEFAULT 'active',
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### `applications`
```sql
CREATE TABLE applications (
  id            INT AUTO_INCREMENT PRIMARY KEY,
  job_id        INT NOT NULL REFERENCES jobs(id),
  seeker_id     INT NOT NULL REFERENCES users(id),
  cover_letter  TEXT,
  resume_url    VARCHAR(500),
  status        ENUM('pending', 'under_review', 'shortlisted', 'rejected') DEFAULT 'pending',
  applied_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE KEY unique_application (job_id, seeker_id)
);
```

### `saved_jobs`
```sql
CREATE TABLE saved_jobs (
  id         INT AUTO_INCREMENT PRIMARY KEY,
  seeker_id  INT NOT NULL REFERENCES users(id),
  job_id     INT NOT NULL REFERENCES jobs(id),
  saved_at   TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE KEY unique_save (seeker_id, job_id)
);
```

---

## 🌐 API Endpoints (Express REST)

### Auth (`/api/auth`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/register` | Register new user (seeker or recruiter) |
| POST | `/api/auth/login` | Login, returns JWT token |
| GET | `/api/auth/me` | Get current logged-in user profile |

### Jobs (`/api/jobs`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/jobs` | List all jobs (with query filters) |
| GET | `/api/jobs/:id` | Get single job details |
| POST | `/api/jobs` | Create job *(Recruiter only)* |
| PUT | `/api/jobs/:id` | Update job *(Recruiter only)* |
| DELETE | `/api/jobs/:id` | Delete job *(Recruiter only)* |
| PATCH | `/api/jobs/:id/status` | Change job status (active/paused/closed) |

### Categories (`/api/categories`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/categories` | List all job categories |

### Applications (`/api/applications`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/applications` | Apply to a job *(Seeker only)* |
| GET | `/api/applications/my` | Get seeker's own applications |
| GET | `/api/applications/job/:jobId` | Get all applicants for a job *(Recruiter only)* |
| PATCH | `/api/applications/:id/status` | Update application status *(Recruiter only)* |

### Saved Jobs (`/api/saved-jobs`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/saved-jobs` | Save / bookmark a job |
| GET | `/api/saved-jobs` | Get seeker's saved jobs |
| DELETE | `/api/saved-jobs/:jobId` | Remove saved job |

### Profile (`/api/profile`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/profile` | Get own profile |
| PUT | `/api/profile` | Update profile (seeker or recruiter) |

---

## 🖥️ Frontend Pages & Routes

### Public Pages
| Route | Page | Description |
|-------|------|-------------|
| `/` | Home | Hero section, featured jobs, category grid, stats |
| `/jobs` | Job Listings | Filterable, searchable paginated job list |
| `/jobs/:id` | Job Detail | Full job description + Apply button |
| `/login` | Login | Email/password form |
| `/register` | Register | Role selection then seeker or recruiter form |
| `/companies` | Companies | Browse companies/recruiters |

### Seeker (Protected)
| Route | Page |
|-------|------|
| `/dashboard` | Seeker dashboard — applied jobs, saved jobs, profile stats |
| `/profile` | Edit seeker profile, upload resume |
| `/applications` | Track application statuses |
| `/saved-jobs` | Bookmarked jobs |

### Recruiter (Protected)
| Route | Page |
|-------|------|
| `/recruiter/dashboard` | Stats — posted jobs, applicants, open positions |
| `/recruiter/jobs` | Manage all posted jobs |
| `/recruiter/jobs/new` | Post a new job |
| `/recruiter/jobs/:id/edit` | Edit a job posting |
| `/recruiter/jobs/:id/applicants` | View applicants for a job |
| `/recruiter/profile` | Edit company profile |

---

## 📁 Folder Structure

### Client (`/client/src`)
```
src/
├── assets/              # Images, icons, SVGs
├── components/
│   ├── common/          # Navbar, Footer, Button, Input, Badge, Modal
│   ├── jobs/            # JobCard, JobList, JobFilters, JobDetail
│   ├── auth/            # LoginForm, RegisterForm
│   ├── dashboard/       # StatsCard, ApplicationRow
│   └── recruiter/       # JobForm, ApplicantCard
├── pages/
│   ├── Home.jsx
│   ├── Jobs.jsx
│   ├── JobDetail.jsx
│   ├── Login.jsx
│   ├── Register.jsx
│   ├── seeker/
│   │   ├── Dashboard.jsx
│   │   ├── Profile.jsx
│   │   ├── Applications.jsx
│   │   └── SavedJobs.jsx
│   └── recruiter/
│       ├── Dashboard.jsx
│       ├── ManageJobs.jsx
│       ├── PostJob.jsx
│       └── Applicants.jsx
├── hooks/               # useAuth, useJobs, useFilters, usePagination
├── context/             # AuthContext.jsx
├── services/            # api.js (axios instance + API call functions)
├── utils/               # formatDate, formatSalary, helpers
├── App.jsx
├── main.jsx
└── index.css
```

### Server (`/server`)
```
server/
├── config/
│   └── db.js            # MySQL connection pool (already created)
├── controllers/
│   ├── authController.js
│   ├── jobController.js
│   ├── applicationController.js
│   ├── categoryController.js
│   ├── savedJobController.js
│   └── profileController.js
├── middleware/
│   ├── authMiddleware.js  # JWT verification
│   └── roleMiddleware.js  # Recruiter / Seeker role guards
├── routes/
│   ├── authRoutes.js
│   ├── jobRoutes.js
│   ├── applicationRoutes.js
│   ├── categoryRoutes.js
│   ├── savedJobRoutes.js
│   └── profileRoutes.js
├── database/
│   └── schema.sql         # All CREATE TABLE statements
├── .env.example
├── .gitignore
├── package.json
└── server.js
```

---

## 🎨 Design System (TailwindCSS 4)

### Color Palette
| Token | Purpose | Value |
|-------|---------|-------|
| Primary | CTAs, links, active states | Indigo `#4F46E5` |
| Primary Dark | Hover states | `#3730A3` |
| Accent | Badges, highlights | Violet `#7C3AED` |
| Success | Active jobs, shortlisted | Emerald `#10B981` |
| Warning | Under Review status | Amber `#F59E0B` |
| Danger | Rejected, close job | Rose `#F43F5E` |
| Neutral BG | Page background | Slate `#F8FAFC` |
| Card BG | Cards, panels | White + shadow |

### Typography
- **Font**: `Inter` (Google Fonts)
- **Headings**: `font-bold`, sizes `text-4xl` down to `text-xl`
- **Body**: `text-gray-600`, `text-sm` / `text-base`

### Key UI Components
- **JobCard** — Glassmorphism card with company logo, title, tags (remote/type), salary, deadline, quick-apply button
- **FilterPanel** — Sticky sidebar with dropdowns, checkboxes, range slider
- **CategoryGrid** — Icon-based colored cards on homepage
- **StatusBadge** — Color-coded pill badges per application status
- **HeroSection** — Full-width gradient banner with search bar

---

## 🔐 Authentication & Authorization Flow

```
Register → Role selected (seeker | recruiter)
         → bcrypt hash password
         → Insert into users table
         → Return JWT token

Login    → Verify email/password
         → Return JWT with payload: { id, role, name }

Protected Routes → authMiddleware.js verifies token
               → roleMiddleware.js checks role before sensitive actions
```

**JWT Payload:**
```json
{
  "id": 1,
  "role": "recruiter",
  "name": "Alice Corp",
  "iat": 1700000000,
  "exp": 1700086400
}
```

---

## 📦 Additional Packages to Install

### Client
```bash
npm install react-router-dom axios react-hook-form
npm install react-hot-toast lucide-react
npm install @radix-ui/react-slider
```

### Server
```bash
npm install bcryptjs jsonwebtoken multer
npm install express-validator
```

---

## 🛣️ Development Phases

### Phase 1 — Foundation & Auth (Week 1)
- [ ] MySQL schema setup (`database/schema.sql`)
- [ ] JWT Auth routes (register, login, /me)
- [ ] AuthContext + protected routes on client
- [ ] Navbar with role-aware links
- [ ] Login & Register pages

### Phase 2 — Job Listings & Filters (Week 2)
- [ ] Seed categories table
- [ ] `GET /api/jobs` with filter query params
- [ ] Job listing page with filter sidebar
- [ ] Job detail page
- [ ] Homepage with hero + category grid

### Phase 3 — Recruiter Features (Week 3)
- [ ] Recruiter dashboard
- [ ] Post / Edit / Delete job routes & UI
- [ ] Manage jobs page (status toggle)

### Phase 4 — Seeker Features (Week 4)
- [ ] Apply to jobs (POST /api/applications)
- [ ] Seeker dashboard & application tracker
- [ ] Save / bookmark jobs
- [ ] Seeker profile with resume upload (Multer)

### Phase 5 — Recruiter Applicant Management (Week 5)
- [ ] Applicants list per job
- [ ] Status update (shortlist / reject / review)

### Phase 6 — Polish & Optimization (Week 6)
- [ ] Responsive design (mobile-first)
- [ ] Loading skeletons, empty states, error states
- [ ] Pagination
- [ ] SEO meta tags per page
- [ ] Code cleanup & documentation

---

## ✅ Verification Checklist

| Check | Method |
|-------|--------|
| API endpoints return correct data | Thunder Client / Postman |
| JWT auth guards work | Try accessing protected route without token |
| Filters return correct filtered jobs | Manual query testing |
| Recruiter cannot apply to jobs | Role middleware test |
| Seeker cannot post jobs | Role middleware test |
| Duplicate application blocked | DB UNIQUE constraint test |
| Responsive UI | Browser dev tools + mobile device |

---

## 🔮 Future Enhancements (Post-MVP)

- **Email notifications** — Job alerts, application status updates (Nodemailer)
- **AI Job Match** — Suggest jobs based on seeker skills
- **Resume Parser** — Auto-fill profile from uploaded resume
- **Interview Scheduler** — Calendar integration for scheduling interviews
- **Admin Panel** — Manage users, categories, and reported listings
- **Analytics** — Recruiter insights: views per job, conversion rates
- **Chat System** — In-app messaging between seeker and recruiter
