---
title: "Aze: a Full-Stack Starter I Kept Rebuilding, So I Saved It"
description: I published aze-mini, an MIT-licensed full-stack starter with Next.js, NestJS, Postgres, Redis, Docker and Helm. Here is what it is, why it is opinionated on purpose, and what you get when you clone it.
date: 2026-08-28
tags: [starter, fullstack, nestjs, nextjs, devops]
---

Every side project I start begins the same way. Not with the idea, with the
plumbing. A monorepo layout, an API with auth, a database with migrations, a
cache, Docker files, CI, and at some point the question "how does this get
deployed?" By the time the plumbing works, the weekend is over and the idea
hasn't been touched.

So eventually I put all of that plumbing into one repository and published it.
It is called [Aze](https://github.com/aruzone/aze-mini) (the repo is
`aze-mini`), it is MIT licensed, and it is a full-stack starter. Clone it,
delete the demo code, and start building the thing you actually meant to
build.

<!--more-->

## What is in it

The stack, in one breath: an **Nx monorepo** holding a **Next.js** client and
a **NestJS** API, with **Prisma** on **Postgres**, a **Redis** cache, Docker
images, a **Helm** chart, and an **Argo CD** application. TypeScript
everywhere.

That sentence describes a lot of starters. What makes this one different is
not the list of technologies. It is that the boring decisions have already
been made, and written down where you can argue with them.

## Opinionated on purpose

Most starters hand you a pile of choices and get out of the way. That sounds
friendly, but it means you spend your first evening re-deciding things that
have standard answers. Aze makes the decisions and documents each one in an
[ADR](https://github.com/aruzone/aze-mini/tree/main/docs/adr). A few examples:

- **One database.** Postgres only. There is no file-based fallback that works
  in dev and behaves differently in production.
- **Auth fails closed.** Every API route requires a token unless it explicitly
  opts out with a decorator. A new route is protected by default, not exposed
  by default.
- **The cache fails open.** If Redis is down, the API gets slower. It does
  not break. Those are two different failure modes and they should not be
  confused.
- **One error shape.** Every refusal comes back as
  `{ statusCode, timestamp, path, message }`, so a client has exactly one
  thing to read.
- **The token never reaches browser JavaScript.** The client calls the API
  from its own server, and the JWT lives in an httpOnly cookie.

You own your clone. If you disagree with any of these, that is fine. The
point of writing them down is that a change you make later is a decision you
made on purpose, not an accident.

## Platform and Demo

The repository is deliberately split into two tiers:

- **Platform** is what you keep: auth, the request perimeter, the cache, the
  error handling, the session, the CI.
- **Demo** is a small product catalogue with a seeded user, there to show
  each pattern once. Read it, then delete it.

Deleting the demo is a supported operation, not an afterthought.
[docs/demo.md](https://github.com/aruzone/aze-mini/blob/main/docs/demo.md) is
a removal guide: the exact paths to delete, the order to work in, and how to
check that what is left still works. Lint rules are set up so platform code
cannot quietly depend on demo code, which means the demo is genuinely
removable rather than tangled in.

## Clone and own, no strings

One thing worth saying plainly, because most starters hide it: **there is no
update path**. When you clone Aze, you own the result. Fixes made to the
starter later (including security fixes) will not reach your project, and
there is no supported way to pull them in.

This is written down in an
[ADR](https://github.com/aruzone/aze-mini/blob/main/docs/adr/0004-clone-and-own-no-update-path.md)
because it shapes everything else. Since the security posture you clone is
the posture you keep, the parts that matter, like auth, tokens and headers,
are held to a production bar, and the parts that are *not* held to that bar
say so out loud.

## Built to be read by coding agents

A lot of the code I write now gets written with AI agents in the loop, and
agents work from what a repository tells them. So Aze carries:

- **AGENTS.md** is the working brief: every command, the module layout, what
  each file is for. `CLAUDE.md` just points at it, so no tool reads a
  different version of the truth.
- **CONTEXT.md** is the project's vocabulary, including words to avoid, so an
  agent's output uses the same terms the code does.
- **Commands that need no local knowledge**: `npm run test`, `npm run lint`,
  `npm run build` all work for an agent that has never seen the repo.
- **A documentation checker**: `npm run check:docs` fails if a document
  names a file that does not exist or describes a route the API does not
  serve. Docs that cannot quietly go stale.

## Getting started

If you have Docker, this is the whole setup:

```bash
git clone https://github.com/aruzone/aze-mini.git
cd aze-mini

docker compose up -d --build --wait

# optional: seed the demo catalogue and a user to sign in as
docker compose run --rm migrate npx prisma db seed
```

Client at `localhost:3000`, API at `localhost:3030/api`, and an interactive
OpenAPI page at `localhost:3030/api/docs`.

For day-to-day work there is a local-toolchain path too (Node 24, with Docker
for Postgres and Redis only). Both are written up as runbooks in
[docs/agents/](https://github.com/aruzone/aze-mini/tree/main/docs/agents).

One note: the compose file carries committed public credentials for local
use. They are not secrets and are not fit to be. Replace them before running
that file anywhere but your own machine.

## What it does not do

The starter runs, migrates and deploys. Carrying real users' data asks for a
few more decisions, and rather than leaving you to discover them in
production, they are listed in
[docs/deployment.md](https://github.com/aruzone/aze-mini/blob/main/docs/deployment.md):
TLS and an Ingress, database backups, a general rate limit beyond login,
token revocation, shared throttle counts across replicas, and real secret
management. A summary table says what is already handled and what is yours.

## That is it

No dashboard, no CLI, no paid tier. A repository with the plumbing done, the
reasoning written down, and the demo code waiting to be deleted. If that
sounds like a useful starting point, take a look:

**[github.com/aruzone/aze-mini](https://github.com/aruzone/aze-mini)**. It is
MIT licensed; use it for whatever you like.
