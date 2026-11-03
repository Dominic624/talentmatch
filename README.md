# TalentMatch

An AI-matched freelance marketplace. Clients get a short, ranked list of relevant freelancers and proposals instead of a flood of irrelevant ones. Freelancers get projects that fit their skills. Messaging, milestone tracking, escrow-style payments and two-way reviews are built in.

![Client dashboard](docs/screenshots/2-client-dashboard.png)

## Features

| Area | What it does |
| --- | --- |
| **Smart matching** | Every freelancer–project pair gets a 0–100 match score with a plain-language "why this match" explanation. |
| **Relevant proposals only** | Proposals are scored when they arrive. Low-relevance ones (below 60%) are hidden by default, with a toggle to show them. |
| **Client flow** | Post a project, see AI-ranked freelancers, invite them, shortlist or decline proposals, hire, define milestones. |
| **Freelancer flow** | Recommended projects ranked by match, search and filters, send proposals, submit milestone work, track earnings. |
| **Milestones and escrow** | Pending → Funded → Submitted → Paid. Funding holds money in escrow, approval releases it. Every step is written to a transaction ledger. |
| **Messaging** | One thread per project and freelancer, available before and after hiring. |
| **Reviews** | Two-way star ratings and comments after a contract completes. Profiles show the average, rating breakdown and completion record. |
| **Notifications** | In-app bell for new proposals, shortlists, funding, submissions, payments and messages. |
| **Dashboards** | Client (projects, spend, escrow, pending actions) and freelancer (earnings chart, proposals, recommendations). A small admin overview too. |
| **Invoices** | Plain-text invoice per contract. |

## Quick start

Requires **Node.js 22.13 or newer** (it uses the built-in `node:sqlite` module, so there is no database to install).

```bash
npm install
cp .env.example .env      # optional; sensible defaults are built in
npm run seed              # demo data: 12 freelancers, 4 clients, 10 projects, proposals, a live contract, reviews
npm start                 # http://localhost:3000
```

Demo accounts (password `Demo@123`):

| Role | Email |
| --- | --- |
| Client | `client@demo.com` |
| Freelancer | `freelancer@demo.com` |
| Admin | `admin@demo.com` |

The login page also has one-click demo buttons.

Run the tests with `npm test`.

### Suggested demo path

1. Log in as the client. Open **Post a project** and create one. You land on its AI-ranked matches.
2. Open *E-commerce website redesign* → **Proposals** to see relevant proposals ranked, with low-relevance ones hidden.
3. Open its contract. Approve the submitted milestone to release payment, fund the next one, and chat.
4. Log out and log in as the freelancer to see recommended projects, send a proposal and submit work.
5. When every milestone is paid, both sides can leave a review.

## How matching works

`src/matching.js` is a transparent weighted model, so every score can be explained:

```
score = 50% skill overlap
      + 15% budget fit      (project budget vs. the freelancer's rate over ~60 hours)
      + 15% rating          (average review rating, neutral for newcomers)
      + 10% success rate    (completed contracts / total contracts)
      + 10% availability
```

Each result also returns the matched and missing skills and a one-line explanation, such as *"Matches 3 of 4 required skills · fits your budget · rated 4.8★ · available now"*. The same scorer ranks freelancers for clients, projects for freelancers, and incoming proposals.

## Tech stack

- **Backend:** Node.js, Express, SQLite (`node:sqlite`), signed HttpOnly session cookies, scrypt password hashing
- **Frontend:** vanilla JavaScript single-page app with a hash router, no build step
- **Tests:** `node:test` covering matching, auth, the full hire → escrow → review flow, and permissions

## Project structure

```
src/
  server.js     REST API, auth middleware, escrow rules
  matching.js   match scoring
  db.js         SQLite schema and transactions
  auth.js       password hashing and signed tokens
  seed.js       demo data
public/         index.html, styles.css, app.js (the SPA)
test/           API and matching tests
docs/           screenshots and design mockups
```

## API overview

All routes are under `/api` and return JSON. Auth uses an HttpOnly cookie set by login or register.

| Method | Route | Who |
| --- | --- | --- |
| POST | `/auth/register`, `/auth/login`, `/auth/logout` | anyone |
| GET / PATCH | `/me` | signed in |
| GET | `/users/:id` | signed in (profile, reviews, rating breakdown) |
| GET / POST | `/projects` | signed in (freelancers see match-ranked results) |
| GET | `/projects/:id`, `/projects/:id/matches` | signed in / owning client |
| POST | `/projects/:id/invite`, `/projects/:id/proposals` | client / freelancer |
| GET | `/projects/:id/proposals` | owning client |
| PATCH | `/proposals/:id` | client (shortlist or decline) |
| POST | `/proposals/:id/hire` | client (creates the contract and milestones) |
| GET | `/contracts`, `/contracts/:id`, `/contracts/:id/invoice` | contract parties |
| POST | `/milestones/:id/fund`, `/submit`, `/approve` | client / freelancer / client |
| POST | `/contracts/:id/reviews` | contract parties, after completion |
| GET / POST | `/projects/:id/messages` | client and applicants |
| GET / POST | `/notifications`, `/notifications/read` | signed in |
| GET | `/dashboard` | role-specific |

## Known limitations

These are deliberate scope choices for a project demo, so say so plainly in a review:

- **Payments are simulated.** Escrow is a ledger in the database. No real money moves and no payment gateway (for example Razorpay) is connected. The fund and release steps are the natural place to plug one in.
- **Matching is rule-based**, not a learned model or embeddings. It is explainable, but it only compares skill names exactly (case-insensitive), so "ReactJS" and "React" are different skills.
- **Chat refreshes every 4 seconds** instead of using WebSockets, and has no file attachments or typing indicator.
- **The "40% faster hiring" figure is the project's goal, not a measurement.** The dashboard shows a measured number instead: the share of received proposals that were filtered out as low-relevance.
- No email notifications, password reset, dispute handling or automated deployment config yet.

## Screenshots and mockups

Real captures from the running app are in [`docs/screenshots`](docs/screenshots). The earlier design-concept images are in [`docs/mockups`](docs/mockups) and are illustrative only, not captures of this app.

## License

MIT
