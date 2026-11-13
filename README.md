# ARMSTRONG 2026 — Classroom Guinea Pig Name Election

A responsive, dark-and-gold classroom voting website with campaign advertisements, a shared ballot, live-ish results (15-second polling), and a teacher/admin dashboard.

## What works
- Every student votes from their own device using the same website URL.
- Each randomly generated one-time code can cast exactly one ballot, enforced atomically by PostgreSQL.
- Students do not need to sign up, enter a name, or share an email address.
- Votes are shared in Supabase and results refresh automatically.
- The admin can add/remove candidate names, publish/remove ads, generate ballot codes, and open/close voting.
- The site includes example candidates **Armstrong, Fluffy, and Peanut**. Replace the latter two with the real nominees before distributing codes. All names are displayed equally on the ballot.

## Setup (approximately 15 minutes)
1. Create a free project at https://supabase.com/dashboard .
2. Open **SQL Editor** and run the entire contents of `supabase/setup.sql`.
3. In **Authentication → Users**, create a teacher/admin user with an email and password. Prefer a teacher-owned email. Copy that user's UUID.
4. In SQL Editor, run:
   ```sql
   insert into public.admins(user_id) values ('PASTE-ADMIN-USER-UUID-HERE');
   ```
5. In **Project Settings → API**, copy your Project URL and publishable/anon key. **Never use the service-role key in the website.**
6. Copy `.env.example` to `.env` and fill in the two `VITE_` variables.
7. Run `npm install` and `npm run dev` for local testing.
8. To share the website with the class, deploy the project to Vercel (https://vercel.com/new), import the project, and set the same `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` environment variables in Vercel. Deploy, then share the resulting URL.
9. In the website's **Admin** tab, sign in, update the nominee list, and generate one code per student. Download/print the codes and distribute them privately, one per student. Keep the admin password private.
10. When the election is over, click **CLOSE VOTING** in Admin.

## Important election notes
- One vote per **code**, not cryptographically one vote per person. The teacher should distribute one code per student and avoid sharing extra codes.
- Do not post voting codes publicly. Do not store student names alongside voting codes.
- Results are public and update every 15 seconds. If you prefer to hide them until polls close, modify the UI accordingly.
- If a candidate has received votes, removing them is blocked by the database to preserve election integrity. You can remove an unused candidate.
- Avoid putting copyrighted soundtrack recordings or game footage on the public website without permission. The site intentionally uses original text and emoji illustrations.
- Ads are labeled as advertisements and do not affect the vote counting or candidate order.

## Project structure
- `src/main.jsx` — website, voting, results, admin features.
- `src/style.css` — styling.
- `supabase/setup.sql` — database schema, policies, one-time-code voting functions, example candidates/ads.

## Privacy/security
- Voter codes are stored as SHA-256 hashes in the database; plaintext codes are shown only once when generated.
- The `cast_class_vote` database function uses a single atomic update to prevent code reuse.
- Row-level security prevents public reading of individual ballots and code hashes.
- Public results only expose aggregate totals.
- The admin dashboard requires Supabase Auth and an explicit allowlist entry.


## Direct video advertisements (new)

In **Admin → Publish a name advertisement**, choose a candidate, enter a headline and ad copy, and optionally choose a **.mp4** or **.webm** file (maximum **50 MB**). The admin uploads the file directly to the Supabase Storage `campaign-videos` bucket. Videos appear with click-to-play controls in the public Campaigns section; they do not autoplay. All candidates can have video advertisements.

**Existing Supabase installations:** Run only the section headed `VIDEO AD UPLOADS` at the bottom of `supabase/setup.sql` in the SQL Editor. New installations should run the entire file once. The bucket is public so anyone with the URL can watch uploaded videos; avoid videos containing classmates' faces or personal information without permission. Upload and deletion require an authorized admin login. Removing an ad also removes its video when possible.


## Two-vote election upgrade

After running `supabase/setup.sql`, run `supabase/two-vote-upgrade.sql` in Supabase SQL Editor.
The updated ballot requires exactly two different candidates and one code per student.
Existing votes are preserved, and used codes cannot be reused. Results count name votes (two per completed new ballot).
For a clean election, upgrade before anyone votes. The old `cast_class_vote` function is no longer callable by site visitors.
