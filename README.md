# RideCheck (Supabase edition)

Vehicle maintenance tracker with real user accounts. Data lives in
Supabase (Postgres) and syncs across devices. The app itself is one static
`index.html` with no build step.

## Setup

1. **Create a Supabase project** at supabase.com (free tier is fine).

2. **Create the tables**: Dashboard -> SQL Editor -> paste the contents of
   `schema.sql` -> Run. This creates the tables, the Row Level Security
   policies, and the activation system described below.

3. **Load the suggested-maintenance dataset**: SQL Editor -> paste
   `maintenance_templates.sql` -> Run (after `schema.sql`). This creates the
   `maintenance_templates` table and fills it with the starter dataset.
   Edit or add rows later in Table Editor; re-running the file will not
   overwrite your edits.

4. **Add your keys to `index.html`**: Project Settings -> API. Copy the
   Project URL and the publishable (or legacy anon) key into
   `SUPABASE_URL` and `SUPABASE_KEY` near the top of the script.
   These are safe to be public *because* of the RLS policies in step 2.
   Never put the `service_role` / secret key in this file.

5. **Auth settings** (Authentication -> Providers -> Email):
   - By default Supabase requires email confirmation. For personal use you
     can turn "Confirm email" off so sign-up logs you straight in.
   - If you keep it on, set Authentication -> URL Configuration -> Site URL
     to wherever you host the app, or the confirmation link will point to
     the wrong place.

6. **Host `index.html` anywhere static** (Supabase itself doesn't serve web
   pages): Cloudflare Pages, Netlify (drag-and-drop), GitHub Pages, or
   Vercel all work for free. Then add that URL as the Site URL (step 4).

## Account activation (owner approval)

Anyone can click "Create account", but new accounts are **inactive**: the
database refuses to read or write any garage data for them. They see a
"pending activation" screen instead.

- **First, approve yourself.** Sign up in the app, then run snippet A at the
  bottom of `schema.sql` in the SQL Editor (with your email).
- **To let someone in**, create a single-use code with snippet B, send it to
  them, and they enter it on the pending screen. Or approve them directly
  with snippet C, no code needed.
- **To remove someone**, use snippet D.
- Set `OWNER_CONTACT` in `index.html` (e.g. your email or WhatsApp) so
  people know how to reach you.
- Codes are single-use, can't be listed from the browser, and the check
  runs inside Postgres, so editing the page's JavaScript can't bypass it.

## Suggested maintenance

Each vehicle has a "Suggested maintenance" button (it also opens right after
you add a motorcycle or scooter). It lists templates matching the vehicle
type, lets the user pick which to track, and offers a conservative (shorter)
or relaxed (longer) interval. Kilometre values are converted for vehicles
tracked in miles, and months become days (months x 30).

Template columns: `applies_to` (`{motorcycle,scooter}`), `interval_km_min`,
`interval_km_max`, `interval_months`, `default_selected`, `notes`, `source`.
Add make/model-specific rows later by adding columns for them.

## Notes

- Each user's whole garage is stored as one JSON row (`garages.data`). Simple
  and fine for personal use; if you ever want per-component queries or
  reports, split it into `vehicles` and `components` tables.
- Free-tier Supabase projects can be paused after a period of inactivity.
  Check their current policy; unpausing is one click in the dashboard.
#