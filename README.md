# DevBlog

A full stack blogging platform: React front end, Node.js/Express REST API, PostgreSQL, JWT auth.

## Features
- Register / log in with JWT auth (passwords hashed with bcrypt)
- Create, edit, delete posts; save as draft or publish
- Tags (many-to-many), tag filtering, and paginated feed
- Comments on published posts
- Responsive layout (Flexbox), light and dark mode

## Security
- All SQL is parameterized (SQL injection)
- React escapes rendered text and no raw HTML is injected (XSS)
- Input validation with zod, Helmet headers, CORS allow-list, rate limiting on auth routes
- Ownership checks on every edit and delete

## Run it
```bash
# 1. Database + API (needs Docker)
docker compose up --build

# 2. Front end (new terminal)
cd client
npm install
npm run dev        # http://localhost:5173
```

Without Docker: create a Postgres database, run `server/schema.sql` on it, copy
`server/.env.example` to `server/.env`, then `cd server && npm install && node --env-file=.env src/index.js`.

## API
| Method | Route | Auth |
|---|---|---|
| POST | /api/auth/register, /api/auth/login | no |
| GET | /api/posts?page=1&tag=node | no |
| GET | /api/posts/mine | yes |
| GET | /api/posts/:id | optional (drafts visible to author) |
| POST / PUT / DELETE | /api/posts, /api/posts/:id | yes (owner) |
| POST | /api/posts/:id/comments | yes |

## Next steps
Add tests (Vitest + Supertest), a GitHub Actions workflow, and deploy.
