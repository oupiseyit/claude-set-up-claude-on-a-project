# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Small Express REST API (users + health check) backed by an in-memory store. It's a course starter project; the app code is a sandbox for Claude Code setup.

## Commands

- `npm run dev`: start the API with `node --watch` on `PORT` (default 3000)
- `npm test`: run all tests (built-in `node --test` runner, no Jest/Mocha)
- `node --test --test-name-pattern="404" tests/users.test.js`: run a single test by name
- `npm run lint`: ESLint (`eslint:recommended`)

CI (`.github/workflows/ci.yml`) runs `npm run lint` and then `npm test` on Node 22, so both must pass.

## Architecture

- `server.js` builds the Express app, mounts one router per resource (`/users`, `/health`) and exports `app`. It only calls `listen()` when run directly (`require.main === module`), which lets tests import it without opening a port.
- `routes/<resource>.js` holds one `express.Router()` per resource. Routes validate input and return status codes (400 for a missing field, 404 when a record isn't found, 201 on create) with `{ error: "..." }` bodies.
- `db/store.js` is the only data-access layer: module-level arrays with an incrementing `nextId`. Data resets on restart and is shared across tests in the same process.

## Conventions

- Use CommonJS (`require`/`module.exports`), not ES modules. ESLint is configured with `sourceType: "script"`.
- Routes read and write data through `db/store.js` functions only; they never touch the arrays directly. To add a resource, create a new route file and a store function, then mount the route in `server.js`.
- Write tests with `node:test` + `node:assert` + `supertest` against the exported `app`. Don't add a test framework.
- Convert `req.params.id` to a number with `Number(...)` before store lookups, because the store compares ids with `===`.
- Read config from `process.env` (see `.env.example`). Never read or commit `.env`.
