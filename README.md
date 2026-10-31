# Prashant Latta Books — Digital Archive

A premium, responsive static frontend for a public online-reading library, with a Supabase-powered private admin dashboard.

## What is included

- Responsive editorial design for mobile and desktop
- Public book catalogue, search, topic and language filters
- Online reading modal
- Admin sign-in, add/edit/delete, publish/unpublish (requires Supabase setup)
- SQL schema and Row Level Security (RLS) policies
- No public sign-up flow
- Static hosting compatible with Cloudflare Pages, GitHub Pages, and Netlify

## Important: demo mode vs production

Before Supabase is configured, the site displays three clearly labelled sample/demo book previews. The Admin button explains that setup is required. **No real administrator login or database exists until you configure Supabase.** Do not advertise the admin system as secure/active before completing the setup and tests below.

This project does not contain a backend secret. `config.js` must contain only the Supabase project URL and the public anon/publishable key. Never add a `service_role` key to frontend files.

## 1. Create the Supabase backend

1. Create a project at https://supabase.com/.
2. In Supabase, open **SQL Editor**.
3. Run all of `supabase-schema.sql`.
4. Open **Project Settings → API** (or API Keys) and copy the Project URL and public anon/publishable key.
5. Paste those two values into `config.js`:

```js
window.LIBRARY_CONFIG = {
  supabaseUrl: "https://YOUR_PROJECT_REF.supabase.co",
  supabaseAnonKey: "YOUR_PUBLIC_ANON_OR_PUBLISHABLE_KEY"
};
```

Only the public anon/publishable key belongs in a browser app. Never paste service_role/secret keys into `config.js`.

## 2. Create the first administrator

1. In Supabase **Authentication → Users**, create the owner user with your own email and a strong password. If email confirmation is required, complete it.
2. Copy that user's UUID from the user details.
3. In SQL Editor, run this query after replacing the placeholder:

```sql
insert into public.admin_users (user_id)
values ('YOUR_AUTH_USER_UUID');
```

Do not put your password in SQL or in the project. Public sign-up is not implemented in this app. Only users explicitly listed in `admin_users` can perform admin operations.

## 3. Add the project to hosting

### Cloudflare Pages (static hosting)

1. Create a private or public GitHub repository and upload the project files.
2. In Cloudflare Dashboard, open **Workers & Pages → Create → Pages → Connect to Git**.
3. Select the repository.
4. For a plain HTML project, leave the build command blank and set the output directory to `/` (the repository root, according to the dashboard's current UI).
5. Deploy. The provided `index.html` is the site entry point.
6. Future commits to the connected branch trigger a new deployment.

Official docs: https://developers.cloudflare.com/pages/

### Netlify (static hosting)

1. Keep a backup of your current site first.
2. Upload the full project folder (the folder containing `index.html`) using Netlify's current manual deploy workflow, or connect its Git repository.
3. Ensure the published directory contains `index.html`, `styles.css`, `app.js`, and `config.js`.
4. Deploy the updated project to the same site only if you intend to replace that site's contents.

Official docs: https://docs.netlify.com/

### GitHub Pages

Can host the static public frontend, but a connected repository is required for convenient updates. Configure Supabase exactly as above. GitHub Pages does not provide a backend itself.

## 4. Test before sharing publicly

- With Supabase configured, open the public site in an incognito window: published books should be visible; drafts should not.
- Attempt Admin sign-in with an incorrect password: it must fail.
- Sign in with the explicitly registered admin account: the dashboard should load.
- Create a test book as Draft: it must not appear in the public library.
- Publish it: it should appear publicly.
- Edit, unpublish, then delete the test book.
- Try opening the Admin page without signing in and verify no admin actions are possible.
- In Supabase, review RLS policies and verify that an unauthenticated insert/update/delete request is rejected.
- Check browser console and mobile layout.
- Keep a backup of the source project and book manuscripts.

## Important limitations

- Published text must be delivered to the reader's browser, so determined readers can copy it or take screenshots. No ordinary website can guarantee zero copying, screenshots, or extraction.
- This starter stores reading content as text/HTML in the database, not original PDF/DOCX uploads. It does not implement private file storage or PDF conversion.
- `safeContent()` applies a small browser-side allow-list for basic HTML; it is not a substitute for a proven server-side sanitizer if untrusted authors will ever submit content. Only the owner should add content.
- Supabase, Cloudflare Pages, GitHub, and Netlify free plans have quotas and terms that can change. “Free” does not mean unlimited or guaranteed forever.
- Configure a custom domain only if you own one; it is optional.
