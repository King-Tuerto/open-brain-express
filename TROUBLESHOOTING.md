# Something wrong?

*[Versión en español: SOLUCION-DE-PROBLEMAS.md](SOLUCION-DE-PROBLEMAS.md)*

There is no support channel for this project — you are meant to work through
problems on your own, ideally with Claude Code or claude.ai open in the same
conversation that built this for you. Paste any of the checks below (or a
screenshot) and ask it to run the fix. Everything on this page already exists
somewhere in the repo; this just collects it by what you're actually SEEING,
not by which file or session it came from.

Find your symptom below.

---

## "It worked before — now everything errors"

**Likely cause: your Supabase project paused itself.** Free-tier Supabase
projects pause after about a week with no real API activity. A paused project
looks exactly like a broken one from the outside — the app can't reach the
database at all.

**Confirm it:** run this with your own project URL and anon key (both are in
`config.js`) —

```
curl -s "https://YOUR-PROJECT.supabase.co/rest/v1/thoughts?select=id&limit=1" \
  -H "apikey: YOUR_ANON_KEY"
```

This is the same request the keep-alive workflow pings on a schedule
(`.github/workflows/keep-alive.yml`) and the same shape used to detect a
live project in [UPGRADE.md](UPGRADE.md)'s Step 0. A live project answers in
under a second — with real rows, an empty list, or even a permissions error,
it doesn't matter which, an answer is an answer. A paused one hangs or fails
to connect at all.

(While you're in there: running that exact request with NO key — drop the
`apikey` header entirely — should fail too. If it comes back with real rows
instead, your database's security is open to anyone with your public key.
That's a separate, more serious problem — see UPGRADE.md's Step 1, check 4,
for what that means and how to close it.)

**Fix it:**
1. Go to [supabase.com/dashboard](https://supabase.com/dashboard) and log in
2. Open your organization, find the project — it will be labelled "Paused"
3. Click **Resume project**, confirm

It comes back within a few minutes. Nothing is lost — same database, same
data, same config, same URL. This works for up to a year after a project
paused; Supabase's own limit, not this project's. (If you're reading this
more than a year after you last touched the project, the automatic button
will be gone and you'd need Supabase's manual backup-recovery process
instead — worth knowing, essentially never going to apply to you.)

**Reduce how often this happens:** this project already ships two
independent pings meant to keep the project active — a pg_cron job inside
your own database (`keep-brain-awake`, set up in Session 2 / Sesión 2 Step 6,
or UPGRADE.md Step 6e if you upgraded) and a GitHub Actions workflow that
pings from outside it. Neither is proven to actually stop a pause — only that
a real request goes out twice a week. If it still happens, this page, not
those pings, is the actual fix.

---

## Saving does nothing, or shows an error you don't understand

Try the save again and read whatever red text appears — the app does show an
error message rather than failing completely silently. Then check, roughly in
order of how often each one is the actual cause:

1. **`config.js` still has its placeholder values.** Open it and check for
   `PASTE_YOUR_PROJECT_URL_HERE` or `PASTE_YOUR_PUBLIC_KEY_HERE` — if either
   is still there, the app was never actually connected to a database. This
   is the exact same check the keep-alive workflow runs on itself before it
   pings anything.
2. **You're not signed in, or your session expired.** Look for the status
   dot near the top of the app — it should say "connected," not "database
   error" or a setup screen. Log out and back in.
3. **The project is paused.** See the symptom above — a paused project makes
   saving fail the same way it makes everything else fail.

---

## Search comes back empty, or finds the wrong things

This almost always means the thought was saved but never got a
meaning-fingerprint (its embedding) — keyword search still works without
one, but "search by meaning" does not.

**Confirm it**, in the Supabase SQL Editor:

```sql
select count(*) from thoughts where embedding is null;
```

A number greater than zero means some thoughts are missing their fingerprint.
For a **brand new** capture, give it 15–30 seconds first — enrichment runs
right after saving, not instantly (if it never arrives at all, that's the
next symptom below, not this one). For **older** thoughts that have sat
un-fingerprinted for a while, run the same backfill this project already
ships for exactly this — [UPGRADE.md](UPGRADE.md) Step 7, dry run first:

```
curl -s -X POST "https://YOUR-PROJECT.supabase.co/functions/v1/backfill-brain" \
  -H "Authorization: Bearer YOUR_SERVICE_ROLE_KEY" \
  -H "Content-Type: application/json" \
  -d '{"dry_run": true}'
```

It reports exactly how many thoughts need one and what it will cost (a small
amount of real money against your OpenRouter key) before it spends anything.
Drop `dry_run` to actually run it.

---

## Captures never get tags, a category, or a summary

This is enrichment not firing — a different problem from search above, and
it has its own two-part check, both already used during the build itself
(Session 2 / Sesión 2, Step 6, and UPGRADE.md Step 6d):

**1. Does the trigger exist?**

```sql
select tgname from pg_trigger
 where tgrelid = 'thoughts'::regclass and not tgisinternal;
```

You should see `on_thought_created`. If it's missing, nothing calls
`enrich-thought` when you save — re-run the trigger block from Session 2
Step 6 / Sesión 2 Paso 6 (`webhook.sql`).

**2. If the trigger is there, check the function's own logs:**
Supabase dashboard → Edge Functions → `enrich-thought` → Logs.

- **Logs are empty** → the function never ran. That points back to check 1,
  or to `pg_net` not being enabled.
- **Logs show errors** → almost always `OPENROUTER_API_KEY` missing or
  mistyped, or no credit left on the OpenRouter account.

---

## The Telegram bot has gone quiet

Check what Telegram itself thinks is happening — it keeps this for you,
no Supabase login needed:

```
https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getWebhookInfo
```

Read `last_error_message` in the response — it usually says exactly what's
wrong (this is the same check Session 2b / Sesión 2b Step 5 has you run if
nothing comes back the first time). The most common finding: a `401`, which
means the `telegram-bot` function got deployed without the
`--no-verify-jwt` flag. Redeploy it with the flag:

```
npx supabase functions deploy telegram-bot --no-verify-jwt
```

If the webhook looks fine but a specific person's messages are ignored
while yours work — that's `TELEGRAM_CHAT_ID` doing exactly what it's meant
to (Session 2b / Sesión 2b Step 4 locks the bot to one phone on purpose).

If it's been silent for everyone including you, also check the project
isn't paused — see the first symptom on this page.

---

## None of these match what you're seeing

Paste a screenshot or the actual error text into Claude Code or claude.ai —
whichever one built this with you — and ask it to diagnose it live. It has
access to your logs and your database in a way this page can't.
