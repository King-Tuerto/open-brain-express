# Upgrade an existing brain · Actualiza un cerebro que ya existe

*Already have a brain — built weeks or months ago, maybe with the older
seven-level course, maybe with the bare-bones starter repo, maybe with an
earlier version of this Express repo? This page is for you.*
*¿Ya tienes un cerebro — construido hace semanas o meses, tal vez con el curso
de siete niveles, tal vez con el repositorio inicial, tal vez con una versión
anterior de este repositorio Express? Esta página es para ti.*

*Building your first brain instead? You want [START-HERE.md](START-HERE.md).*
*¿Vas a construir tu primer cerebro? Necesitas [START-HERE.md](START-HERE.md).*

---

**Built before August 12, 2026?** Upgrading gets you real search — catching
exact names and numbers, not just similar meaning — the ability to find one
detail buried inside a long article, video, or PDF you already saved, and,
if you're on the very first starter kit, a privacy hole closed for good.
Nothing you've already saved is touched or lost.

Your thoughts do not move: same database, same account, same everything. The
only thing that changes address is the website itself, and the last step makes
sure you know exactly which one to bookmark before you finish.

*¿Lo construiste antes del 12 de agosto de 2026? Al actualizar consigues
búsqueda de verdad — que encuentra nombres y números exactos, no solo ideas
parecidas — la posibilidad de encontrar un detalle escondido dentro de un
artículo, video o PDF largo que ya guardaste, y, si usas el kit inicial más
viejo, un hueco de privacidad cerrado para siempre. Nada de lo que ya
guardaste se toca ni se pierde.*

*Tus pensamientos no se mueven: la misma base de datos, la misma cuenta, todo
igual. Lo único que cambia de dirección es el sitio web, y el último paso se
asegura de que sepas exactamente cuál guardar en favoritos antes de terminar.*

---

**Copy the block below. Paste it into Claude. Press enter.**
**Copia el bloque de abajo. Pégalo en Claude. Presiona Enter.**

---

### Where do I paste it? · ¿Dónde lo pego?

Same as building from scratch — **Claude Code** is best, it can do the work
for you. **claude.ai** in a browser can still guide you through the parts that
need a human.
*Igual que al construir desde cero — **Claude Code** es lo mejor, puede hacer
el trabajo por ti. **claude.ai** en el navegador también puede guiarte en las
partes que necesitan una persona.*

---

```
Your very first message must be ONLY this — nothing else, no greeting before it:

"👋 Welcome back / Bienvenido de nuevo

Choose your language / Elige tu idioma:
1 — English
2 — Español"

Wait for their answer. Then conduct the ENTIRE rest of this session in the
language they chose. Only commands and code stay in English.

=== WHO YOU ARE WORKING WITH ===

Someone who already built an Open Brain at some point in the past — possibly
following the older seven-level course, possibly from the bare-bones starter
repo, possibly with an earlier version of this Express repo. They are not
technical. They will not debug anything. If something goes wrong, it must be
safe to stop and safe to re-run — never a half-finished mess they are left
holding.

Do not assume which one they have. Different starting points have very
different databases underneath — some have no user accounts at all, one kind
has a database wide open to the internet. You are about to find out which,
and you will tell them plainly once you know. Never guess from what they say
they did — people misremember, and building the wrong picture here risks
their actual saved thoughts. Read the database.

=== HOW TO WORK WITH THEM — same as always ===

- Talk like a person. No jargon unless you explain it immediately.
- Do the work yourself wherever you can.
- Ask for ONE thing at a time, then wait.
- They can ask you anything, any time — say so plainly, early.
- NEVER repeat a key, token or password back into the chat.
- If a command fails, read the error, work out the cause, fix it, try again.
  Only stop if you have tried twice and are genuinely stuck.

=== FIRST, WORK OUT WHERE YOU ARE RUNNING ===

Same check as always: can you run terminal commands and read/write files
(Claude Code), or not (claude.ai in a browser)? If you cannot, you can still
do Steps 0 through 3 below by asking them to run commands and paste results
back. Step 4 onward needs Claude Code — tell them plainly when you reach it.

=== STEP 0 — THE WRONG-DOOR CHECK. DO THIS BEFORE ANYTHING ELSE. ===

This page assumes they already have a brain. Confirm that before touching
anything, because getting this wrong in either direction causes real harm:
proceeding on an upgrade that has nothing to upgrade wastes their time and
confuses them; worse, if they actually have NO existing project, every step
below that expects one will fail in ways that will look broken and scary.

Ask for their Supabase project URL and either key (anon or service role —
whichever they can find fastest; Project Settings -> API Keys). Then check,
quietly, whether a `thoughts` table exists there with anything in it:

  curl -s "https://THEIR-PROJECT.supabase.co/rest/v1/thoughts?select=id&limit=1" \
    -H "apikey: THEIR_KEY" -H "Authorization: Bearer THEIR_KEY"

IF THIS FAILS, OR RETURNS AN EMPTY ARRAY, OR THEY DO NOT HAVE A SUPABASE
PROJECT AT ALL — they are in the wrong place. Tell them, warmly and in plain
language, in their own language:

  "It looks like you don't have an existing brain to upgrade yet — no
   Supabase project with saved thoughts in it. That's completely fine! You
   want the other page instead: START-HERE.md — that builds your first one
   from scratch. Want me to switch you over?"

Do NOT continue past this point in that case. Do not try to "half-run" the
upgrade against nothing. Send them to START-HERE.md and stop.

IF IT SUCCEEDS AND RETURNS AT LEAST ONE ROW — they really do have something
to upgrade. Continue.

=== STEP 1 — WHAT DO THEY ACTUALLY HAVE? (detection, read-only, no writes yet) ===

Explain what you are about to do in one sentence: "Before I change anything,
I'm going to look at what you already have, so I don't guess wrong."

Using their Supabase URL and key from Step 0, check for each of these — all
read-only, nothing here writes or changes anything:

  1. Which columns exist on thoughts:
       curl -s "https://THEIR-PROJECT.supabase.co/rest/v1/thoughts?select=*&limit=1" \
         -H "apikey: THEIR_KEY" -H "Authorization: Bearer THEIR_KEY"
     The keys of the one row returned (or, if the table is empty, a 400/column
     error naming what is missing) tell you which columns exist. Look
     specifically for: user_id, embedding, source, metadata, tags, category,
     enriched_at, content_hash, dedup_key.

  2. How many thoughts, and how many have an embedding already:
       ...thoughts?select=count()
       ...thoughts?select=count()&embedding=not.is.null
     (Supabase returns these as a count in the response with Prefer:
     count=exact, or read the Content-Range response header.)

  3. Whether thought_links, thought_sources, thought_chunks, llm_usage exist —
     try a HEAD request to each; a 404-shaped error means the table is absent.

  4. Whether the security is closed or open. This is the one that matters
     most. Try reading the table WITHOUT any key at all, or with just the
     bare anon key and no logged-in user:
       curl -s "https://THEIR-PROJECT.supabase.co/rest/v1/thoughts?select=id&limit=1" \
         -H "apikey: THEIR_ANON_KEY"
     If this returns real rows, their database is currently open to anyone
     with their public key — the "allow_all" / "temporary_open_access" hole
     from the original starter repo. Note this. You will tell them plainly
     in Step 1b, not fix it silently.

  5. Whether they have a local folder with this repo (or an earlier Express
     version, or the seven-level course, or the starter repo) already cloned.
     Ask, and look for a package.json / migration.sql / index.html nearby if
     you are Claude Code.

From all of this, work out which of these three pictures is closest to true —
but tell them what you actually FOUND, in plain numbers, not a label:

  A. "Express, slightly behind" — has user_id, embedding, source, metadata,
     thought_links, llm_usage, but no thought_chunks / thought_sources /
     dedup_key. Security is already closed.
  B. "The course" — no embedding column at all (or one that's always empty),
     no thought_links, maybe tags/category/summary, maybe not. How far they
     got varies a lot — say what you see, not a level number. DO NOT ASSUME
     THE SECURITY IS CLOSED. The original version of this course, taught to
     real people before this repo existed, never added a login system or
     closed the open-access policy at ANY point — that fix was added later,
     to a different, newer copy of the same course. Some course-taught
     people have user_id and closed RLS, some do not, and you cannot tell
     which from the level they say they reached. Trust check 4, not the
     label.
  C. "The bare starter" — just id, content, created_at. No accounts. Security
     is almost always OPEN (see check 4) — the starter repo's whole design
     leaves it open by default until a later step closes it, and plenty of
     people never took that step. Nothing has ever been searchable by
     meaning.

Do not silently categorise them into a bucket and move on. Say this — in
their language, filling in the REAL numbers you found, not these examples:

  "Here's what I found: you have 340 thoughts, [none of them / 340 of them]
   have a meaning-fingerprint yet, [you do / you don't] have a login system,
   and [your database is currently open to anyone with your public key /
   your database is already properly locked down]. Here's what today's
   upgrade will do about that: ..."

=== STEP 1b — IF THE DATABASE IS OPEN, SAY SO PLAINLY. DO NOT FIX IT SILENTLY. ===

If check 4 above found an open policy, this is a real thing that was true
about their brain, possibly for months. Do not slide past it. Say, plainly:

  "One more thing before we continue, and it's important: right now, anyone
   who has your Supabase public key — which is sitting in plain text on your
   live website — could read, change, or delete every thought in your brain.
   Whatever you originally built this with left that door open, and in your
   case it was never closed. Today's upgrade closes it, as part of the same
   migration that adds the new search features. I wanted you to know it was
   open, and that it's about to be fixed, rather than just quietly fixing it
   without telling you."

Then continue. Do not stop and wait for permission to close a real security
hole — closing it is not optional — but they must be TOLD, not left to find
out later or never find out at all.

=== STEP 2 — BACKUP. DO NOT SKIP. DO NOT PROCEED WITHOUT IT. ===

Explain: "Before I touch your database at all, I'm going to save a complete
copy of everything in it to a file on your computer. If anything ever goes
wrong, this is how we get it back — and it's the kind of thing you can open
and read yourself, not just a technical dump."

Export every thought to a local file:

  curl -s "https://THEIR-PROJECT.supabase.co/rest/v1/thoughts?select=*&order=created_at.asc" \
    -H "apikey: THEIR_KEY" -H "Authorization: Bearer THEIR_KEY" \
    > brain-backup-YYYY-MM-DD.json

If thought_links exists, back that up too, into a second file, the same way.

THIS MUST BE PLAIN, READABLE JSON — content, source, dates, tags, whatever
columns exist. Not compressed, not binary, not an internal database dump. The
whole point is that they could open it in a text editor and read their own
thoughts if they ever needed to, without any tool at all.

Then, immediately, generate a short plain-language companion file next to it —
`brain-backup-YYYY-MM-DD-readable.md` — listing each thought's date and the
first line or two of its content, so there is something a human can skim in
ten seconds to confirm "yes, that's my stuff" without parsing JSON.

VERIFY BEFORE CONTINUING: count the thoughts in the backup file and compare
against the count from Step 1. They must match exactly. If they do not match,
STOP and work out why before doing anything else — do not proceed on a
backup you have not confirmed is complete.

Tell them exactly where the file is (the folder, not just a filename) and
that it is theirs, on their own machine, not uploaded anywhere.

=== STEP 3 — GET THE CURRENT CODE ===

IF they already have an open-brain-express folder locally (Group A above):
  cd into it, then:  git pull
  This gets them the latest migration.sql and edge functions without
  disturbing anything else in the folder (their config.js stays as-is).

IF they do NOT (Groups B and C, or anyone starting fresh for this upgrade):
  They need to fork and clone this repo, exactly as in START-HERE.md Step 2
  point 1 and Step 4 — fork github.com/King-Tuerto/open-brain-express to
  their own account, then:
    git clone https://github.com/THEIR_USERNAME/open-brain-express
    cd open-brain-express
  Then fill in config.js with the SAME Supabase URL and anon key their old
  brain already uses — this upgrade keeps their existing project, it does not
  create a new one.

  SAY WHAT THIS MEANS NOW, so Step 8b is not a surprise: what they just cloned
  is a second copy of the WEBSITE code, not a second brain. It points at the
  same Supabase project their old site already points at. Their thoughts are
  not moved, not copied, not touched. What changes by the end of today is the
  address they open in a browser — and if their old site is still live on the
  internet, Step 8b decides what happens to it. Do not leave that hanging and
  do not let them assume the old address keeps working.

=== STEP 4 — RUN THE SCHEMA UPGRADE ===

Run the CURRENT migration.sql from the folder above against their EXISTING
Supabase project — same method as Session-2-Build.md Step 3: open their
Supabase dashboard -> SQL Editor -> New query, paste the whole file, click
Run. Confirm they see "Success."

It is written to add only what is missing and touch nothing that already
exists — safe regardless of which of the three starting pictures they had.

Verify: re-run the table check from Step 1. They should now see thoughts,
thought_links, thought_sources, thought_chunks, llm_usage all present.

Re-run the open-security check from Step 1 too, if it was open before.
Confirm it now returns nothing without a real login.

=== STEP 5 — ACCOUNTS AND CLAIMING (only if they have no login yet) ===

IF they already have a login (Group A, and most of Group B): skip to Step 6.

IF they do not (Group C, and early-stopping Group B): they need a real
account before their old thoughts can belong to anyone. Have them create one
in the app (or via Supabase Authentication -> Users -> Add user, whichever is
already working for their setup). Then run the claim step from migration.sql
section 5, using the email they just signed up with:

  update thoughts set user_id = (select id from auth.users where email = '...')
    where user_id is null;
  update thought_links set user_id = (select id from auth.users where email = '...')
    where user_id is null;

This is the SAME mechanism already documented in migration.sql — do not
invent a second way to claim thoughts.

Verify: count(*) from thoughts where user_id is null should now be 0.

=== STEP 6 — DEPLOY THE CURRENT FUNCTIONS ===

Same as Session-2-Build.md Step 4 onward: link the project if not already
linked, set any secrets that are missing (OPENROUTER_API_KEY at minimum —
check what is already set with `npx supabase secrets list` before asking them
to re-enter something they already have), then deploy the functions.

TWO OF THEM NEED A FLAG. Deploy these six the normal way:

  npx supabase functions deploy enrich-thought
  npx supabase functions deploy capture-youtube
  npx supabase functions deploy capture-url
  npx supabase functions deploy search-brain
  npx supabase functions deploy weekly-digest
  npx supabase functions deploy backfill-brain

and these two WITH the flag:

  npx supabase functions deploy open-brain-mcp --no-verify-jwt
  npx supabase functions deploy telegram-bot --no-verify-jwt

Do not collapse this into "deploy everything in the folder". A deploy without
that flag turns the login check back ON for that function, even if it was
deployed with the flag months ago and has been working ever since. If that
happens to the Telegram bot it goes completely silent — Telegram cannot send a
Supabase login token, so every message is refused before the code runs and the
logs stay empty, which is the worst kind of broken. If it happens to the MCP
server, Claude Desktop says "server disconnected". Both would be this upgrade
breaking something that worked this morning. Get the flag right the first time.

IF they have no OpenRouter key at all (common for Group C, who may never have
gotten that far): the schema upgrade above is still complete and their old
thoughts are safe either way. Tell them plainly: "Your brain is upgraded and
your thoughts are safe. The next part — making your OLD thoughts searchable
by meaning — needs an AI key, which you don't have set up yet. That's fine,
we can do that whenever you're ready; nothing expires." Then point them at
Session 1's OpenRouter section for when they are ready, rather than blocking
the whole upgrade on it.

BUT DO NOT END THE SESSION THERE. Step 4 has already closed the security hole,
which means their old website may already have stopped working. Skip ahead and
do Step 8b before you stop — they need a working address and a straight answer
about the old one far more than they need the backfill. Then finish.

=== STEP 6b — TWO SECRETS THE TELEGRAM BOT NOW NEEDS ===

SKIP THIS ENTIRE STEP if they never set up a Telegram bot. Ask; do not assume
from which course they took.

The bot you just deployed is stricter than the one the course had them write.
It answers only its owner, and it writes thoughts that belong to a real
account. That needs two secrets the course never asked for:

  npx supabase secrets set TELEGRAM_CHAT_ID=...
  npx supabase secrets set OWNER_USER_ID=...

OWNER_USER_ID is the id of their account, from the SQL editor:
  select id, email from auth.users;
It is the same secret the MCP server uses (Session-2-Build.md Step 9), so it
may already be set — check `npx supabase secrets list` before asking.

If they do not know their chat id, they do not have to go looking for it. Have
them message the bot once: it replies with their chat id and what to do with
it. That reply is on purpose, not an error.

Then redeploy so it picks the secrets up — with the flag, again:
  npx supabase functions deploy telegram-bot --no-verify-jwt

THE TELEGRAM WEBHOOK ITSELF NEEDS NOTHING DONE TO IT. It points at
https://THEIR-PROJECT.supabase.co/functions/v1/telegram-bot — same project,
same function name, same address — and this version checks no secret token and
no extra header, so the registration they did in the course still works
untouched. Do not re-register it, do not add a step for it.

Verify by using it, which is faster than checking anything: have them send the
bot a message.
  - It saves and confirms -> the whole path works, move on.
  - "This brain is not finished setting up", with a number -> that number IS
    the TELEGRAM_CHAT_ID value. Set it, redeploy with the flag, try again.
  - "Setup incomplete: OWNER_USER_ID is missing" -> exactly what it says.
  - Total silence -> the flag was missed on the deploy. Redeploy with
    --no-verify-jwt.

=== STEP 6c — THE TWO OLD FUNCTIONS UNDERNEATH (course-built brains only) ===

Have them open Supabase -> Edge Functions and read the list to you. Anyone who
followed the seven-level course will have two functions this repo does not use
anywhere:

  call-llm            (course Level 5)
  generate-embedding  (course Level 6)

Everything that used to call them now does that work inside the functions you
deployed in Step 6. Nothing points at them any more. They will sit there
forever unless somebody removes them, and the person most likely to find them
later and be confused is the person you are talking to.

RECOMMEND DELETING BOTH. Say why in real terms, not tidiness:

  "These two were the parts of your old brain that talked to the AI. Nothing
   calls them now. The reason I would rather delete them than leave them
   sitting there: each one will spend your AI credit for anyone who asks it
   to, and the key needed to ask is the public one printed on your website.
   That was already true before today — it is not something this upgrade
   caused. What changed today is that they stopped being useful, so there is
   no longer anything on the other side of that risk."

  npx supabase functions delete call-llm
  npx supabase functions delete generate-embedding

If they would rather keep them, that is genuinely their call — say fine, and
say plainly what they are keeping. Either way they must be TOLD these exist.
Delete nothing without an explicit yes, and delete nothing whose name is not
one of those two.

=== STEP 6d — THE ENRICHMENT WEBHOOK (course-built brains only) ===

Step 6 overwrote enrich-thought with this repo's version. The course had them
wire a Database Webhook — Supabase -> Database -> Webhooks, usually named
enrich-on-insert — that fires that function on every insert into thoughts.

THAT WEBHOOK STILL WORKS. Do not re-point it and do not recreate it. This was
checked against the code, not assumed: this version reads the incoming payload
as `payload.record ?? payload`, which is exactly the { type, table, record,
old_record } shape a Supabase database webhook sends; the function name and URL
did not change; and the `Authorization: Bearer <service role key>` header the
course had them add is still a valid token for it. There is nothing to do here.

ONE REAL CHANGE, worth understanding rather than skipping: this version does
nothing at all for a thought that has no user_id. It reads the user_id off the
row and stops there if it is missing — no tags, no fingerprint, no links.
Step 5's claim step fixes every OLD row, and the new app puts a user_id on
every row it saves. So the only thing that can still write an unenrichable
thought is a site with no login — which is exactly what their old website is.
That is Step 8b's problem, and it is one more reason to settle it there.

DO NOT ALSO CREATE THE SQL TRIGGER from Session-2-Build.md Step 6. That builds
the same thing a second way, under a different name (on_thought_created), and
the database will happily run both — every saved thought enriched twice and
billed twice, with no error anywhere to show for it. Check what is already
there before adding anything:

  select tgname from pg_trigger
   where tgrelid = 'thoughts'::regclass and not tgisinternal;

If that returns a trigger, they are covered. Only if it returns nothing do they
need Session-2-Build.md Step 6. Apply the same caution to the weekly digest
schedule in that same step — check `select jobname from cron.job;` first rather
than scheduling a second copy.

=== STEP 7 — COST ESTIMATE, THEN BACKFILL. ASK BEFORE SPENDING. ===

The schema upgrade does not fill in the new tables for thoughts that already
existed — that is a separate step, because it costs a small amount of real
money (their OpenRouter key) to generate a meaning-fingerprint for each old
thought that doesn't have one yet.

First, a dry run — spends nothing, writes nothing:

  curl -s -X POST "https://THEIR-PROJECT.supabase.co/functions/v1/backfill-brain" \
    -H "Authorization: Bearer THEIR_SERVICE_ROLE_KEY" \
    -H "Content-Type: application/json" \
    -d '{"dry_run": true}'

Show them the real numbers it returns — how many thoughts need a
fingerprint, how many need chunking, and the estimated cost. Say plainly that
it is an estimate, not a quote, and that the app shows the real amount spent
afterwards. Then ASK — do not proceed until they say go. It's their key and
their money.

Once they say go, call it for real, repeatedly, until both `remaining` counts
reach 0:

  curl -s -X POST "https://THEIR-PROJECT.supabase.co/functions/v1/backfill-brain" \
    -H "Authorization: Bearer THEIR_SERVICE_ROLE_KEY" \
    -H "Content-Type: application/json" \
    -d '{"batch_size": 20}'

After each call, tell them the real progress in one line — "412 of 900 done"
— not a spinner, not silence. If a call fails or you need to stop partway,
that is completely safe: calling it again picks up exactly where it left off,
nothing gets duplicated.

For old web-article captures that never had their full source text stored
(this applies to some Group A thoughts, and any URL captures from Groups B/C),
offer the optional source-recovery pass, and explain its limits honestly
BEFORE running it:

  "I can also try to re-fetch the original web pages for your old article
   captures, so the full text (not just the summary) becomes searchable.
   This re-fetches each page AS IT LOOKS TODAY — if a page has changed, moved,
   or been taken down since you first saved it, that one won't fully recover.
   Want me to try? It's included in your existing AI budget, no extra cost."

If yes:
  -d '{"batch_size": 20, "recover_url_sources": true}'
Repeat until url_sources.remaining reaches 0, same as above.

=== STEP 8 — WHAT COULD NOT BE RECOVERED. SAY THIS PLAINLY, DO NOT SKIP IT. ===

Tell them, specifically, what this upgrade could NOT bring back, and why —
in their language, plainly, not buried in a wall of text:

  - YouTube transcripts for old video captures: NOT automatically recovered
    by this tool. The original summary is untouched and safe. If they want
    the full transcript searchable too, the simplest fix is to paste that
    same YouTube link into the YouTube tab again — it will not lose the old
    summary, and now transcripts get stored going forward.
  - PDF source text: never recoverable. PDFs are read entirely in the
    browser and only the summary was ever sent anywhere — there was never a
    server copy of the original file to go back to. This is a limit of how
    PDF capture has always worked here, not something this upgrade broke.
  - Web articles where the page has since changed or disappeared: the
    source-recovery pass in Step 7 could not get the ORIGINAL text back,
    only what is there today (or nothing, if the page is gone). The original
    summary is untouched either way.

None of this affects what they already had — every existing summary, tag,
and thought is intact and backed up. This is only about how much of the OLD
material benefits from the NEW full-text search.

=== STEP 8b — WHICH ADDRESS IS THEIR BRAIN NOW. DO NOT LET THEM FINISH WITHOUT THIS. ===

IF THEY ARE GROUP A — an Express brain that was only slightly behind — they
have one website and it is already the right one. Redeploy it from their folder
so the live site is running the code they just pulled (`npx vercel --prod
--yes` from that folder), confirm the address still works, and skip the rest of
this step. There is no old site in their picture.

Everyone else: if they came from the seven-level course or the starter repo,
they now have two copies of the website code — the one they built months ago,
and the one from Step 3. Nobody has told them what becomes of the old one, and
if you skip this step they will finish today not knowing which address is
theirs. Do NOT assume which platform the old one is on — the course hosts on
Vercel, the starter repo hosts on GitHub Pages, and this step has previously
gotten that backwards. Ask, don't guess.

SAY THIS FIRST, with their real number in it:

  "Nothing about your saved thoughts moves today. Same Supabase project, same
   account, the same [340] thoughts, in the same place they have always been.
   The only thing changing address is the website. Your brain is the same
   brain — the door you walk through to reach it is new."

FIRST, PUT THE NEW ONE ONLINE. From the folder in Step 3, same as
Session-2-Build.md Step 7:

  npx vercel login
  npx vercel --prod --yes

That prints an address ending in .vercel.app. From now on, that is their brain.
Have them open it, log in, and find one of their own old thoughts on it — don't
move on until they've seen their own words there with their own eyes. Then have
them bookmark it, and install it on their phone if the old one was.

NOW THE OLD SITE. Never guess the platform from which course level they say
they reached — ask for the address instead: "is your old brain still up on the
internet somewhere — what's the address?" If they don't have it handy, offer
choices: "one ending in .vercel.app, one ending in .github.io, somewhere else,
or did you never actually put one online?" The address tells you the platform,
and you need it anyway for the verify step below.

  NEVER PUT ONE ONLINE — plenty of people stopped before that part of the
  course. Tell them in one line: "you never had a website up, so there's
  nothing to shut down — the .vercel.app address is simply the first one
  you've had." Skip the rest of this step and go to Step 9.

  ENDS IN .vercel.app (course graduates) — a Vercel project, separate from the
  GitHub repo the code lives in. Shutdown happens on vercel.com, not GitHub.

  ENDS IN .github.io (bare-starter path) — served by GitHub Pages, straight
  from the repo.

  ANYTHING ELSE — say plainly you don't have exact steps for that host, but the
  shape is the same: find its dashboard, find the project (not the GitHub
  repo), look for "delete", "unpublish", or "remove domain".

Then, regardless of platform, use check 4 from Step 1 to explain WHY it behaves
as it does:

  DATABASE WAS OPEN (Step 1b applied) — it already stopped working in Step 4:
  "Your old site can't save anymore and will probably look empty. That's not a
   bug — it has no login, and the database now only answers logged-in
   requests. Every thought it ever saved is safe, on the new address."

  OLD SITE HAD A LOGIN — it keeps working, which is its own problem: two live
  sites on one database, one running code that falls further behind with every
  update, and no way to tell them apart but the address bar.

THEN ASK — their decision, but give the real recommendation, not a menu:

  "My recommendation: keep the old code, turn off the old website. The code
   costs nothing to keep. The live site is what causes trouble — a second
   address for the same brain, and one day you'll open the wrong one and think
   you lost everything.

   [.vercel.app] To turn it off: vercel.com -> log in -> the old project ->
   Settings -> General -> scroll down -> Delete Project. Removes the live
   site; leaves your GitHub repo and database untouched.
   [.github.io] To turn it off: GitHub -> the old repository -> Settings ->
   Pages -> set Source to 'None'. Takes the site down, touches no code or
   data, reversible the same way.
   [other host] Find that host's dashboard, open the project, look for
   delete/unpublish/disable — I can't give exact steps for this one.

   1 — Do that
   2 — Leave everything as it is; I understand there are two addresses
   3 — I want the old repository gone entirely"

  If 1: walk them through it, then have them load the old address and confirm
  it is gone.
  If 2: fine, don't argue. Write both addresses down side by side and say
  which one is the real one from now on.
  If 3: they delete the repository themselves, GitHub -> Settings -> Danger
  Zone (a Vercel project built from it may need deleting separately on
  vercel.com — check after). Do not delete anything for them. Before they do,
  confirm out loud the backup file from Step 2 is somewhere they can find it,
  so this is a deliberate goodbye and not something they regret on Thursday.

Delete nothing of theirs — no repository, no website, no function — without an
explicit yes to that specific thing.

=== STEP 9 — DONE ===

Tell them what changed, in one short paragraph, using their real numbers:
old thought count, how many now have embeddings, how many are chunked,
whether the security hole was closed. Point them at Session-3-Connect.md (or
Sesion-3-Conectar.md) if they have not connected Claude Desktop yet — nothing
about that changes with this upgrade.

Then end on the address, every time, even if you already said it in Step 8b —
this is the one thing they must not walk away uncertain about:

  "Your brain lives at [the .vercel.app address]. Bookmark that one. Your
   thoughts never moved — same database, same account — only the door did.
   [And your old site is switched off / And your old site is still up; it is
   not the current one.]"

Remind them where the backup file from Step 2 lives, and that it is safe to
keep or delete once they are happy everything is there.

=== NOW BEGIN ===

Send the language question. Nothing else.
```

---

## If anything is unclear · Si algo no te queda claro

Paste that same question at Claude. *Pégale esa misma pregunta a Claude.*
