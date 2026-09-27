# AMC Mock Exam Grader

Two pages, backed by **Netlify Blobs** (Netlify's own built-in storage) — no third-party account, no API keys, nothing to sign up for beyond the Netlify account you're already using to deploy:

- `index.html` — student page (name, problem set display, A–E / numeric answers, submit, view results)
- `admin.html` — admin page (edit header text and problem set, set the answer key, view/delete submissions)
- `netlify/functions/config.js` and `netlify/functions/submissions.js` — small serverless functions that read/write the shared data
- No login on either page (as requested) — treat this as a prototype, not something to put real student data behind.

## Important: this needs a "real" Netlify deploy, not drag-and-drop

The old version could be dragged onto netlify.com/drop because it was pure static files. This version has **one dependency** (`@netlify/blobs`) that needs to be installed for the functions to work, and the drag-and-drop drop zone doesn't run an install step. Use one of these two options instead:

### Option A — Netlify CLI (fastest, from your own computer)

1. Install Node.js if you don't have it (https://nodejs.org).
2. Open a terminal in this folder and run:
   ```
   npm install
   npm install -g netlify-cli
   netlify deploy --prod
   ```
3. Follow the prompts (log in, create/select a site, publish directory is `.`). It'll give you a live URL when done.

### Option B — Connect a GitHub repo (best if you'll keep editing this)

1. Create a new GitHub repo and push this whole folder to it.
2. In Netlify: **Add new site → Import an existing project**, pick the repo.
3. Build settings: leave build command blank, publish directory `.` — Netlify reads `netlify.toml` (already included) for the rest.
4. Deploy. Every time you push a change, Netlify rebuilds automatically.

Either way, Netlify Blobs works automatically once the site is deployed on Netlify's platform — there's no dashboard step to turn it on, no keys to copy anywhere.

## How it works

- **Config** is stored as one JSON blob (`main`) in a Blobs store called `amc-config`. It holds the header text, kicker label, the posted problem set (text or PDF), number of problems, each problem's type (A–E or numeric 000–999 AIME-style), and the answer key.
- **Submissions** are stored one-per-blob in a store called `amc-submissions`, each holding a student's name, answers, and a score computed against the answer key at submission time. "View results" on the student page re-fetches and re-grades against the *current* key, so if you post the key after the fact, students can revisit and see an updated score.
- Both pages talk to the functions through `/api/config` and `/api/submissions`, which `netlify.toml` redirects to the actual function paths.
- Answers left blank in the admin answer key stay **ungraded** — students see "no answer key posted yet" for those problems until you fill them in.
- PDFs are uploaded as base64 directly into the config blob; keep them under ~8MB, or paste an external link instead (e.g. a direct link to a PDF hosted elsewhere).

## Things you may want to add later

- Password-protect `admin.html` (Netlify Site settings → Visitor access has a simple built-in password option, or use Netlify Identity for something sturdier).
- Per-problem point values / different scoring for AMC 12 vs AMC 10.
- CSV export of submissions.
- Rate-limiting or basic auth on the `/api/*` functions themselves.

Let me know if you'd like help with any of these.
