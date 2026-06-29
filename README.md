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

### Per-person point tiers (optional)
By default everyone gets the tier in the demo control (EGSer = 1, Team leader = 3, VNM member = 5). To assign tiers by person, edit the `roleByEmail` block near the top of `index.html`:

```js
roleByEmail: {
  'an.nguyen@eastgate-software.com': 'Team leader',
  'lan.tran@eastgate-software.com': 'VNM team member'
}
```

---

## 3. Optional — shared live feed for everyone (Supabase)

Without a backend, each visitor sees the demo feed locally. To make **all** colleagues see the same recognitions in real time, add a free [Supabase](https://supabase.com) project:

1. Create a project → **SQL Editor** → run:

```sql
create table recognitions (
  id          bigint generated always as identity primary key,
  from_name   text,
  from_email  text,
  to_name     text,
  to_email    text,
  value_key   text,
  reason      text,
  created_at  timestamptz default now()
);
alter table recognitions enable row level security;
create policy "read"   on recognitions for select using (true);
create policy "insert" on recognitions for insert with check (true);
```

2. Project **Settings → API** → copy the **Project URL** and **anon public key**.
3. Paste them into the `supabase: { url: '', anonKey: '' }` line of the config block in `index.html`.

The feed, counters, and leaderboard then run off live data instead of the demo set.

---

**Need to change copy, colors, or layout?** Those edits should be made in the source design (`Core Value Recognition.dc.html`) and re-exported — editing the bundled `index.html` directly is not recommended beyond the config block above.
