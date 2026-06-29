# Core Value Recognition — deploy on GitHub Pages

This folder is a complete, self-contained site. **`index.html` is the only file you need to upload.**

> ✅ This build already has the Eastgate Microsoft sign-in IDs baked in
> (clientId `459afc01-…` / tenantId `1fd983f3-…`). You do **not** need to
> edit the file. The only remaining setup is registering the site URL in
> Azure — see section 2.

---

## 1. Put it on GitHub

1. Create a new repository (e.g. `core-value-recognition`).
2. Upload **`index.html`** to the repo root (drag-and-drop in the GitHub web UI is fine).
3. Repo → **Settings → Pages** → *Build and deployment* → Source = **Deploy from a branch**, Branch = **main**, folder = **/ (root)** → **Save**.
4. After a minute your site is live at:
   `https://<your-username>.github.io/<repo-name>/`
   (current deployment: `https://egs-people-operations.github.io/core-value-recognition/`)

> **Re-uploading a new build?** Delete the old `index.html` and upload the new
> one so it fully overwrites — keep the filename `index.html` at the repo root.
> Then hard-reload the site (Ctrl+Shift+R) since GitHub Pages caches aggressively.
> Quick check you have the right file: open `index.html` on GitHub, Ctrl+F for
> `459afc01` — if it's not found, it's an old build.

---

## 2. Register the site URL in Azure (required for Microsoft sign-in)

The IDs are already in the file; Azure just needs to trust your site's URL.

1. [Azure Portal](https://portal.azure.com) → **Microsoft Entra ID** → **App registrations** → open the app (Application ID `459afc01-7725-4d17-a077-078cc321a911`).
2. Left menu → **Authentication**.
3. Under **Single-page application (SPA)** → **Add URI** → paste the **exact** Pages URL, including the trailing slash:
   `https://egs-people-operations.github.io/core-value-recognition/`
   *(optionally add `…/core-value-recognition/index.html` too).*
4. **Save**.

Reload the live site and click **Sign in with Microsoft** — you'll get a real popup, and the top-right badge will no longer say "demo data".

> ⚠️ **Important:** sign-in only works on the **real GitHub Pages URL**. Testing
> from a preview / different domain gives `AADSTS50011: redirect URI does not
> match`, because the code sends its own current URL as the redirect.

---

## 3. Shared feed for everyone, with privacy (Supabase)

Without a backend, recognitions are stored only in each person's own browser. To make **all** colleagues share one feed — while keeping nominators anonymous to everyone except admins — connect a free [Supabase](https://supabase.com) project.

**How privacy works:** the server only ever sends *anonymized* rows to regular users (no nominator name/email — it isn't in the data at all, so it can't be seen in browser dev tools). Admins enter a passphrase that's checked **server-side** to reveal who nominated whom. Everyone still sees the recipient (for "Most recognized") and the comment.

### Steps
1. [supabase.com](https://supabase.com) → sign in → **New project** (region: Singapore). Wait ~2 min.
2. **SQL Editor → New query** → open `supabase-setup.sql` (in this folder), **replace `CHANGE_ME_ADMIN_PASSPHRASE`** with your own secret, paste the whole script, **Run**.
3. **Project Settings → API** → copy the **Project URL** and the **anon public** key.
4. Open `index.html`, find the config block near the top, and fill in:

```js
supabase: {
  url:     'https://xxxx.supabase.co',   // Project URL
  anonKey: 'eyJ...'                       // anon public key
},
adminEmails: [
  'hr.lead@eastgate-software.com'         // who may unlock the admin view
],
```

Commit and reload. The feed is now shared in real time (refreshes every ~12s).

### Using the admin view
- Anyone listed in `adminEmails` sees an **"Admin view"** button in the header after signing in.
- Clicking it asks for the **admin passphrase** (the one you set in the SQL). Enter it → nominator names appear. Click **"Exit admin"** to go back to the anonymous view.
- The passphrase lives only in the database function, never in `index.html`.

> The anon key is safe to expose (that's its purpose). Row security only allows
> inserting recognitions and reading the anonymized feed; full identities require
> the passphrase checked inside the database.

### Per-person point tiers (optional)
By default everyone gets the tier in the demo control (EGSer = 1, Team leader = 3, VNM member = 5). To assign tiers by person, edit the `roleByEmail` block near the top of `index.html`:

```js
roleByEmail: {
  'an.nguyen@eastgate-software.com': 'Team leader',
  'lan.tran@eastgate-software.com': 'VNM team member'
}
```

---

**Need to change copy, colors, or layout?** Those edits should be made in the source design (`Core Value Recognition.dc.html`) and re-exported — editing the bundled `index.html` directly is not recommended beyond the config block above.
