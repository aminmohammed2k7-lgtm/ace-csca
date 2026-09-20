# ACE CSCA — v13

A single-file exam prep app plus a published course file. No build step, no
server, no framework.

```
public/
  index.html     the whole app
  content.json   the published course — this is the file you edit
  _headers       caching rules for Cloudflare
wrangler.jsonc   only used if you deploy as a Worker, not as Pages
```

---

## What changed from v12

v12 kept everything you added in your own browser's localStorage. Deployed
as-is, a student would have opened an empty app, and nothing you added on
your laptop would ever have reached them.

**1. content.json is now the course.** The app fetches it at launch. It is
held in memory only and is never written to a student's storage, so deleting
a test from the file deletes it for everyone on their next visit.

**2. Stable ids.** v12 stamped every test and deck with `Date.now()`, so the
same test re-published tomorrow was a different test as far as the app was
concerned — and every student's best score and mistake-bank entry pointed at
an id that no longer existed. Ids now come from subject + title
(`t-math-sets-and-inequalities`). Republish as often as you like; progress
stays attached.

**3. Flashcard scheduling keyed by card content, not position.** Deleting the
second card of a deck used to shift every card's review history up by one on
every device that already had it.

**4. The Content screen is hidden from students** (see studio mode below).

**5. Resources open their link.** The "Open →" button used to pop a toast and
do nothing. Resources now take a URL — your YouTube walkthroughs, a Drive PDF,
a Skool post.

---

## Deploying (about 10 minutes, free)

1. **GitHub** — make a free account, create a repository, and upload this
   whole folder with *Add file → Upload files*.
2. **Cloudflare** — make a free account → *Workers & Pages* → *Create* →
   connect the repository.
   - Build command: **leave empty**
   - Output / asset directory: **`public`**
3. Cloudflare gives you an address like `ace-csca.pages.dev`. Custom domains
   and SSL are free if you buy a domain later.
4. To update anything, edit or replace the file in GitHub. Cloudflare
   redeploys by itself in about a minute.

A note on the other free hosts: **Vercel's** free plan is limited to
non-commercial use and counts taking payment from visitors as commercial.
**GitHub Pages** forbids using it to run an online business. Use GitHub to
store the files, and Cloudflare to serve them.

Opening `index.html` by double-clicking it still works, but browsers block
`fetch` on `file://` addresses, so content.json will not load that way. To
test locally, run `python3 -m http.server` inside `public/` and open
`http://localhost:8000`.

---

## Studio mode — how you add content

The Content screen is hidden by default. To turn it on for **your** browser:

    https://your-site.pages.dev/#studio

That is remembered on that device. `#studio-off` turns it back off. The
version label bottom-left reads "v13 · studio" while it is on.

This is a convenience switch, not a login. content.json is a public file —
anyone who types its address can read it. It is fine for free content; see
"What this cannot do" below.

### The publishing loop

1. Open your site with `#studio`.
2. Build tests, questions, decks, cards, drills and resources in the Content
   screen.
3. **Export JSON**. You get `ace-csca-content.json`.
4. Open it, add a `"version"` line at the top and bump the number each time:
   ```json
   { "version": 2, "tests": [ ... ], "decks": [ ... ] }
   ```
5. Rename it `content.json` and replace the one in `public/` on GitHub.
6. About a minute later every student has it.

Bumping `version` matters on **your** device only: it is how the studio copy
knows to pull down content you published from somewhere else, instead of
sitting on the local drafts it already has. Students always get the latest
file regardless.

### Editing by hand

content.json is plain JSON, so you can also type into it directly. The shape:

```json
{
  "version": 3,
  "tests": [{
    "id": "t-math-sets-and-inequalities",
    "subj": "Mathematics",
    "title": "Sets and inequalities",
    "diff": "Beginner",
    "min": 35,
    "mock": false,
    "questions": [{
      "text": "If 3x - 7 = 11, what is x?",
      "opts": ["6", "4", "18", "3"],
      "ans": 0,
      "expl": "Add 7 to both sides, then divide by 3."
    }]
  }],
  "decks": [{
    "id": "d-math-core-formulas",
    "subj": "Mathematics",
    "title": "Core formulas",
    "cards": [{ "f": "Quadratic formula", "b": "x = (-b +/- sqrt(b^2-4ac)) / 2a" }]
  }],
  "drills":    [{ "level": 1, "subj": "Mathematics", "text": "14 x 6", "answer": 84 }],
  "resources": [{ "subj": "Mathematics", "type": "Formula Sheet",
                  "title": "Algebra sheet", "desc": "One page of essentials",
                  "pages": "1 page", "url": "https://..." }]
}
```

Rules worth remembering:

- `subj` must be exactly `Mathematics`, `Physics` or `Chemistry`.
- `diff` must be `Beginner`, `Intermediate` or `Advanced`.
- `ans` counts from **0**, so `0` means the first option.
- `type` on a resource must be one of `Formula Sheet`, `Video Lecture`,
  `Study Guide`, `Past Papers`.
- **Never change an `id`** once students have used it — their scores and card
  schedules are filed under it. Change the title freely; keep the id.
- Anything malformed is skipped rather than breaking the page. Open the
  browser console (F12) after a publish to see what was skipped.

The app still shuffles options and question order on every attempt, so
writing every question with the answer first is fine.

---

## What this cannot do yet

Everything in content.json is public, and progress lives in each student's
own browser. That means:

- No logins, so no paid tier — anyone with the address gets everything.
- Progress does not follow a student from phone to laptop, and clearing
  browser data wipes it.

Both need a real database behind a login. Supabase is the usual free start
(50,000 monthly users, 500 MB database), though free projects pause after a
week of inactivity and the paid plan is $25/month. That is a phase-2 job —
launch free content on this first, and add accounts once people are paying.
