# FriendZone + Supabase Truth or Dare

## Files
- `friendzone.html`: single-page website with your Love Letter, Birthday Gift, and new Truth or Dare feature.
- `supabase_truth_or_dare.sql`: schema, RPC functions, and access controls.

## 1. Run the SQL
1. Open your Supabase project.
2. Go to SQL Editor -> New query.
3. Paste all of `supabase_truth_or_dare.sql` and run it.
4. If your project does not permit creating `pgcrypto` in the `extensions` schema, check Database -> Extensions and enable `pgcrypto`, then rerun the SQL.

## 2. Configure the HTML
In `friendzone.html`, replace:
- `YOUR_SUPABASE_URL` with Project URL from Project Settings -> API.
- `YOUR_SUPABASE_PUBLISHABLE_OR_ANON_KEY` with the publishable key (or legacy anon key) from Project Settings -> API.

Never put a `service_role` key or secret key in browser HTML/JavaScript.

## 3. Deploy
Deploy `friendzone.html` to your existing static hosting (for example, the same hosting where FriendZone currently lives). Use HTTPS. Links use the current page URL, so the page should be hosted at the final public URL before creating games.

## 4. Flow
- Create a game: creates a game row and generates a high-entropy owner token in the browser. The database stores only its SHA-256 hash.
- Player link: loads the public game and allows a player to submit one response per form submission. This basic version does not enforce one response per person; use sign-in or a one-time invite token if that is needed.
- Owner link: contains the raw owner token in the URL fragment. It is checked by the `td_get_owner_responses` RPC; direct table access is revoked from browser roles.
- Games expire 90 days after creation. Update `expires_at` default in SQL if you want a different lifetime.

## Security notes
- Anyone who obtains the full Owner Link can read the responses for that game. Treat it like a password.
- The publishable/anon key is expected to be visible in frontend code; the RPC functions and revoked table grants constrain what it can do.
- The SQL functions are `SECURITY DEFINER`; review the SQL and test in a separate project before deploying to production.
- Public games and submissions are anonymous in this version. Add CAPTCHA/rate limiting or authenticated users if abuse becomes a concern.
- Avoid collecting sensitive personal information. Truth answers may be visible to the game owner and should be treated accordingly.
- Owner token is sent to Supabase only when calling the owner RPC. Do not add analytics that capture URL fragments.
- Existing Love Letter and Birthday Gift data remain encoded in the URL and are not private or encrypted.

## Troubleshooting
- “Game not found or expired”: confirm SQL ran successfully and the page is using the correct project.
- “Invalid owner access”: use the exact Owner Link produced when creating the game.
- If RPC functions do not appear immediately, refresh the page and check Supabase Database -> Functions.
