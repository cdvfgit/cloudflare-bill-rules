# cloudflare-bill-rules (free sample)

Two rules that would have stopped some of the most expensive Cloudflare bills of 2026.
Copy them into your `AGENTS.md` (or `CLAUDE.md`) so your AI coding agent stops writing
these patterns.

## CF001 — A Durable Object alarm must never re-arm itself unconditionally

A Durable Object alarm loop reportedly produced **$34,000 in 8 days with zero users**.
Another, written by an AI agent, cost **$10,811**.

```ts
// ❌ runs forever, even when there is nothing to do
async alarm() {
  await this.doWork();
  await this.ctx.storage.setAlarm(Date.now() + 60_000);
}

// ✅ re-arm only while there is work
async alarm() {
  const more = await this.doWork();
  if (more) await this.ctx.storage.setAlarm(Date.now() + 60_000);
}
```

## CF006 — Every D1 `UPDATE` and `DELETE` has a `WHERE`

A missing `WHERE` in request code reportedly cost **$5,000+ in about 10 seconds**.

```ts
// ❌ updates every row, on every request
await env.DB.prepare("UPDATE users SET last_seen = ?").bind(now).run();

// ✅
await env.DB.prepare("UPDATE users SET last_seen = ? WHERE id = ?").bind(now, userId).run();
```

## Want all 7 rules + the fuses?

**[Cloudflare Bill Shield](https://csdev.gumroad.com/l/bill-shield)** ($5) includes:

- the full `AGENTS.md` with 7 rules (queue self-loops, unbatched DO writes, `KV.list()` on
  hot paths, non-idempotent crons, retry ceilings…)
- `guard.ts`: drop-in `guardedAlarm()`, self-call loop guard and queue retry cap — tested,
  zero dependencies
- a config checklist and an emergency plan for when usage spikes

MIT licensed sample. PRs with new incidents welcome.
