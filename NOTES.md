# Notes

## CLAUDE.md: what's in, what's out

**In:** a one-line description, the four commands I actually use (`dev`, `test`, a single-test invocation, `lint`), a short architecture section (`server.js` exports `app` and only listens when run directly; one router per resource; `db/store.js` as the only data layer), and conventions Claude couldn't safely guess: CommonJS only, routes go through the store, `node:test` + `supertest` with no new framework, and `Number(req.params.id)` because the store compares ids with `===`. I also noted that CI runs lint before tests, so Claude knows both have to pass.

**Out:** a file-by-file listing and descriptions of each route, because Claude can read those from the code and they go stale. Setup steps (`npm install`) and course instructions stay in the README. No secrets or `.env` values, only a pointer to `.env.example`. No one-off tasks. Every line is there to stop Claude making a specific mistake or asking a question.

## Permission rules

- **allow:** `npm test`, `npm run lint`, `node --test`. These are read-only and run all the time, so a prompt for each one would just be noise.
- **ask:** `git push`. Pushing is outward-facing and hard to undo, so I want to confirm each one.
- **deny:** reading `.env` / `.env.*`, and `git push --force` / `git push -f`.

**Without the deny rules:** Claude could read `.env` while debugging config. That would put real secrets into the conversation and possibly into a commit, a log, or a pasted snippet. A force-push could overwrite shared history on the remote and destroy other people's work, which can't easily be recovered. The `ask` rule alone isn't enough there, because one hasty "yes" is all it takes. A deny rule can't be clicked through.

Trade-off: `.env.*` also blocks `.env.example`. I kept it because it's safer to block every `.env` variant than to miss one like `.env.local`. If Claude needs the config keys, I can share `.env.example` with it myself.

## Verification

In a fresh session, `/memory` lists `CLAUDE.md` as loaded and `/permissions` shows the allow, ask and deny rules above. When I asked "How do I run the tests here?", Claude answered from `CLAUDE.md` (`npm test`, plus the single-test command) without any extra explanation from me.
