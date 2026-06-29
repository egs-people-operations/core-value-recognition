# Core Value Recognition — deploy on GitHub Pages

This folder is a complete, self-contained site. **`index.html` is the only file you need to upload.**

---

## 1. Put it on GitHub

1. Create a new repository (e.g. `core-value-recognition`).
2. Upload **`index.html`** to the repo root (drag-and-drop in the GitHub web UI is fine).
3. Repo → **Settings → Pages** → *Build and deployment* → Source = **Deploy from a branch**, Branch = **main**, folder = **/ (root)** → **Save**.
4. After a minute your site is live at:
   `https://<your-username>.github.io/<repo-name>/`

That's it — out of the box it runs in **demo mode** (mock sign-in + sample feed) so you can show it immediately.

---

## 2. Turn on real Microsoft sign-in (Entra ID / Azure AD)

### a. Register the app in Azure
1. [Azure Portal](https://portal.azure.com) → **Microsoft Entra ID** → **App registrations** → **New registration**.
2. Name: `Core Value Recognition`.
3. Supported account types: **Accounts in this organizational directory only** (single tenant — Eastgate only).
4. **Redirect URI** → platform **Single-page application (SPA)** → paste your exact Pages URL:
   `https://<your-username>.github.io/<repo-name>/`
   *(add `https://<your-username>.github.io/<repo-name>/index.html` as a second SPA redirect URI too, to be safe).*
5. **Register**.
6. On the overview page copy the **Application (client) ID** and **Directory (tenant) ID**.

### b. Paste the IDs into the site
Open `index.html` in any text editor (or GitHub's web editor → pencil icon). Near the **top** of the file find this block and fill in the two values:

```js
window.EGS_CONFIG = {
  msal: {
    clientId: '',   // ← paste Application (client) ID
    tenantId: ''    // ← paste Directory (tenant) ID
  },
  ...
};
```

Save / commit. Reload the site — the **Sign in with Microsoft** button now does a real popup login, and each person's monthly points are tied to their account.

> The site auto-uses its own URL as the MSAL redirect, so no extra config is needed beyond registering that URL in step *a.4*.

### Per-person point tiers (optional)
By default everyone gets the tier set in the demo control (EGSer = 1, Team leader = 3, VNM member = 5). To assign tiers by person, list them in the same config block:

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
