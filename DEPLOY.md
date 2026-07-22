# Deploy Guide — Build For Yourself Now

Three steps: GitHub repo → Cloudflare Pages project → domain.

---

## Step 1 — Create the GitHub repo and push

Open PowerShell in VS Code (`Ctrl + ~`) and run these commands **from inside the project folder**.

```powershell
# Navigate to the project folder
cd "Z:\Dropbox\Artwork\_DansbeeDesigns\_Products\_Websites\BuildForYourselfNow"

# Initialize git
git init
git add .
git commit -m "Initial commit — Build For Yourself site"

# Create the repo on GitHub (requires GitHub CLI — see note below)
gh repo create buildforyourselfnow --public --source=. --remote=origin --push
```

> **If you don't have GitHub CLI (`gh`) installed:**
> 1. Go to https://github.com/new
> 2. Name the repo `buildforyourselfnow`, set it Public, don't add a README
> 3. Then run in PowerShell:
> ```powershell
> git remote add origin https://github.com/YOUR-USERNAME/buildforyourselfnow.git
> git branch -M main
> git push -u origin main
> ```

---

## Step 2 — Connect the repo to Cloudflare Pages

1. Go to https://dash.cloudflare.com and log in
2. In the left sidebar: **Workers & Pages** → **Create application** → **Pages** → **Connect to Git**
3. Authorize GitHub if prompted, then select the `buildforyourselfnow` repo
4. Configure the build:

| Setting | Value |
|---|---|
| Project name | `buildforyourselfnow` |
| Production branch | `main` |
| Build command | *(leave blank)* |
| Build output directory | *(leave blank)* |
| Root directory | *(leave blank)* |

5. Click **Save and Deploy**

Cloudflare will deploy the site. It takes about 30 seconds. You'll get a URL like `buildforyourselfnow.pages.dev` — that's your site live.

---

## Step 3 — Point your domain to Cloudflare Pages

Since your domain (`buildforyourselfnow.com`) is already registered with Cloudflare:

1. In Cloudflare Pages: open your new project → **Custom domains** tab → **Set up a custom domain**
2. Enter `buildforyourselfnow.com` → **Continue**
3. Cloudflare will detect the domain is already in your account and add the DNS record automatically
4. Also add `www.buildforyourselfnow.com` → Cloudflare will set up a redirect

DNS propagates in minutes since the domain is already at Cloudflare.

---

## Ongoing workflow (how to update the site)

Every time you edit `index.html` or `style.css` in VS Code, push with:

```powershell
cd "Z:\Dropbox\Artwork\_DansbeeDesigns\_Products\_Websites\BuildForYourselfNow"
git add .
git commit -m "Describe what you changed"
git push
```

Cloudflare Pages detects the push and auto-deploys within ~30 seconds. No manual steps.

---

## File structure

```
BuildForYourselfNow/
├── index.html        ← Main homepage
├── build-log.html    ← Private creator log (protected by Cloudflare Access)
├── style.css         ← All styles
├── .gitignore        ← Git ignore rules
└── DEPLOY.md         ← This file (not served by Cloudflare)
```

---

## Step 4 — Protect /build-log with Cloudflare Access

Cloudflare Access locks a URL path to approved email addresses only. It's free and built into your account.

### 4a — Enable Zero Trust

1. In the Cloudflare dashboard, click **Zero Trust** in the left sidebar (or go to https://one.dash.cloudflare.com)
2. If it's your first time, you'll be asked to pick a team name — use something like `buildforyourself` (this is internal only)
3. Select the **Free plan** when prompted

### 4b — Create the Access Application

1. In Zero Trust: **Access** → **Applications** → **Add an application**
2. Choose **Self-hosted**
3. Fill in:

| Field | Value |
|---|---|
| Application name | `Build Log` |
| Session duration | `24 hours` (or longer — your choice) |
| Application domain | `buildforyourselfnow.com` |
| Path | `build-log` |

4. Click **Next**

### 4c — Create the Access Policy

1. Policy name: `Owner only`
2. Action: **Allow**
3. Under **Include**, set the rule to:
   - Selector: **Emails**
   - Value: `cale@dansbee.com`
4. Click **Next** → **Add application**

### 4d — Test it

Visit `https://buildforyourselfnow.com/build-log` in an incognito window. You should see a Cloudflare Access login screen asking for your email. Enter `cale@dansbee.com` — Cloudflare sends a one-time code to that address. After you verify, you're in and stay authenticated for the session duration you set.

Anyone else who visits that URL gets the access denied screen.

> **Note:** The `noindex, nofollow` meta tag in `build-log.html` also tells search engines not to index the page, as a second layer of obscurity.
