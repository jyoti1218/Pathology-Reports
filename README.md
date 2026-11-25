# PathReport (private online copy)

PathReport hosted on GitHub Pages. You sign in with your UT work Microsoft account, and your dashboard cases, templates, quick phrases and site formats sync through a private folder in **your own OneDrive** (`Apps/PathReport`).

* This repository contains only the app code. It holds no data.
* Do not commit a PathReport file that you saved "with your settings" (one with embedded data). Use `index.html` from this folder.
* No patient names, MRNs or DOBs. The dashboard holds accession numbers and pending tests only.

## 1. Put the app on GitHub Pages

1. Sign in at github.com. Create a new repository, e.g. `pathreport`.
2. Choose **Add file → Upload files** and upload `index.html`, `.nojekyll` and `README.md`. Commit.
3. Go to **Settings → Pages**. Set Source to *Deploy from a branch*, branch `main`, folder `/ (root)`, then Save.
4. After about a minute the address appears, e.g. `https://YOURNAME.github.io/pathreport/`. Copy it exactly, including the final `/`.

Note: GitHub Pages sites are publicly reachable, even from a private repository, unless you have GitHub Enterprise. That is acceptable here because the page contains only code and templates without patient data. Access to your data is locked behind the Microsoft sign-in and your OneDrive.

## 2. Register the app in Microsoft Entra (Azure)

1. Go to https://entra.microsoft.com (or portal.azure.com → *Microsoft Entra ID*) and sign in with your UT account.
2. Go to **App registrations → New registration**.
   * Name: `PathReport`
   * Supported account types: *Accounts in this organizational directory only (single tenant)*
   * Redirect URI: platform **Single-page application (SPA)**, URL = your Pages address from step 1.4, e.g. `https://YOURNAME.github.io/pathreport/`
   * Click **Register**.
3. From the Overview page, copy the **Application (client) ID** and the **Directory (tenant) ID**.
4. Go to **API permissions → Add a permission → Microsoft Graph → Delegated** and add:
   * `Files.ReadWrite.AppFolder`
   * `User.Read` (usually already present)
5. No client secret is needed.

If UT does not let you register apps, or asks for **admin approval** when you sign in, send UT IT the request: "SPA app registration, delegated Graph permissions User.Read and Files.ReadWrite.AppFolder only (app's own OneDrive folder, no access to other files or users)."

## 3. Connect the app

1. Open your Pages address.
2. Click **Save to OneDrive** (top), then section **3. Cloud sync**.
3. Paste the Client ID and Tenant ID and click **Save sync settings**.
4. Open **Dashboard** and click **Sign in to sync**. Choose your UT account.

The first computer that signs in uploads its templates and cases. Each additional computer that signs in for the first time receives the cloud templates (they are not overwritten), and its dashboard cases are merged in.

Optional: to avoid typing the IDs on each computer, edit `index.html` on GitHub. Search for `const CLOUD_DEFAULT` and fill in `clientId: '...'` and `tenantId: '...'`. These IDs are not secrets.

## How the merge works

* **Dashboard cases** merge case by case, and the newest edit of each case wins. Deleted cases stay deleted on every computer.
* **Templates, quick phrases and sites:** the most recently changed copy wins. When another computer has newer templates, the page reloads to apply them.
* Sync runs when the app opens, a few seconds after each change, when you return to the tab, and every 2 minutes.
* Each browser also keeps a local copy, so the app works offline. It syncs again on your next sign-in.

## 4. Automatic updates: new versions publish themselves to GitHub

Do this once on the computer where you receive new PathReport versions.

1. On GitHub, open your picture menu and go to **Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token**.
   * Name: `PathReport publish`. Expiration: up to 1 year. Set a reminder to renew it.
   * Repository access: **Only select repositories → pathreport**.
   * Permissions: **Repository permissions → Contents → Read and write**. Leave everything else at No access.
   * Click **Generate** and copy the token (`github_pat_…`).
2. Open a PathReport file on that computer. Go to **Save to OneDrive → 4. Publish new versions to GitHub**. Fill in your GitHub user name, `pathreport`, `main`, and paste the token. Click **Save**.

From then on, whenever you open a newer PathReport file (for example Version 227), it does three things:

* compares its version with the one on GitHub;
* if it is newer, uploads itself as `index.html`, removing any data saved inside the file and building in your Azure Client and Tenant IDs;
* updates GitHub Pages about a minute later.

Opening an older file never replaces a newer one. You can also click **Publish this version now**, or choose **Publish a version file…** to pick a file.

The token is stored only in that browser on that computer. It never goes into a saved PathReport file, OneDrive or GitHub. If it is ever exposed, delete it on GitHub; it can only change the pathreport repository.

## Backups in OneDrive

Each signed-in computer writes a snapshot, at most once a day and again whenever the app version changes, to `OneDrive/Apps/PathReport/backups/PathReport-backup-<date>-<time>-v<version>.json`. Each snapshot holds all cases and settings. To bring cases back from one, open **Dashboard → Import** and choose the snapshot file.

The live files `PathReport-dashboard.json` and `PathReport-settings.json` in the same folder are updated with every sync. OneDrive also keeps its own version history for them.
