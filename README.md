# Omni — informational website

This is a static website for **Omni**, a private, single-user personal AI assistant. It exists so that Omni's Google OAuth client has a public homepage and privacy policy, which Google requires before an app can be moved from **Testing** to **In production**.

The site is informational only. It has no sign-in, forms, analytics, cookies, or third-party scripts.

## Contents

| File | Purpose |
| --- | --- |
| `index.html` | Homepage: what Omni is, its integrations, how it uses Google Calendar and Gmail, and contact details |
| `privacy.html` | Privacy Policy, written for Google's OAuth and API Services User Data Policy requirements |
| `terms.html` | Terms of Service for a private, single-user assistant |
| `styles.css` | Shared styles for both pages (plain CSS, no build step) |
| `Logo.png` | Original Omni logo (1254×1254) |
| `logo-256.png` | 256px copy of the logo, used for the header, homepage, and favicon |
| `.nojekyll` | Tells GitHub Pages to serve the files as they are, without a Jekyll build |
| `.gitignore` | Keeps credentials, tokens, and environment files out of the repository |

All links between pages are relative, so the site works at any GitHub Pages URL.

> **Never commit** OAuth client secrets, tokens, API keys, `.env` files, email contents, or Notion data to this repository. If you use GitHub's free plan, the repository must be public for Pages to work.

## Before publishing: resolve every review marker

Facts that couldn't be confirmed from this repository are marked with a highlighted `[REVIEW: …]` tag in the HTML. Replace or delete every one before you deploy:

```sh
grep -n "REVIEW" index.html privacy.html terms.html
```

Then remove the `<mark class="review">` wrappers. The homepage, privacy policy, and terms of service must agree on features, scopes, data flows, and contact details.

## Deploy with GitHub Pages

This repository is named `kiankian.github.io`, so GitHub serves it as a user site at the root of that domain.

1. Commit the files and push to `main` on `github.com/kiankian/kiankian.github.io`.
2. On GitHub, open the repository's **Settings → Pages**.
3. Under **Build and deployment**, set **Source** to **Deploy from a branch**, then choose branch **`main`** and folder **`/ (root)`**. Click **Save**.
4. After a minute or two, check that the site is live, and turn on **Enforce HTTPS** if it isn't already on.

### Expected URLs

No custom domain is configured (there is no `CNAME` file), so the URLs will be:

- Homepage: `https://kiankian.github.io/`
- Privacy Policy: `https://kiankian.github.io/privacy.html`
- Terms of Service: `https://kiankian.github.io/terms.html`

Open all three in a private browser window to confirm they load without signing in.

## Configure Google Cloud → Google Auth Platform

Make these changes in the [Google Cloud console](https://console.cloud.google.com/auth/overview), with the project that owns Omni's Desktop OAuth client selected.

### Branding

Go to **Google Auth Platform → Branding**:

- **App name:** use exactly the name shown on the website (currently "Omni"). Google expects the consent-screen name to match the homepage. *Confirm this matches your existing consent screen.*
- **User support email:** your email address.
- **App logo:** leave this empty. Uploading a logo (including `Logo.png`) triggers Google's brand verification, and the consent screen may not show it until that's done. A personal-use app doesn't need one.
- **Application home page:** `https://kiankian.github.io/`
- **Application privacy policy link:** `https://kiankian.github.io/privacy.html`
- **Application terms of service link:** `https://kiankian.github.io/terms.html`
- **Authorized domains:** add `kiankian.github.io`. `github.io` is on the Public Suffix List, so `kiankian.github.io` counts as its own registrable domain. Confirm the console accepts it.
- **Developer contact information:** your email address.

Save the changes.

### Data Access

Go to **Google Auth Platform → Data Access** and check that the listed scopes match exactly what Omni requests and what `privacy.html` describes. For example:

- `https://www.googleapis.com/auth/gmail.readonly` — **restricted** scope
- `https://www.googleapis.com/auth/calendar.readonly` or `calendar.events` — **sensitive** scope

Remove any scopes Omni doesn't use.

### Switch to In production

Go to **Google Auth Platform → Audience**:

1. Check that **User type** is **External**. Personal `@gmail.com` accounts can't use Internal.
2. Under **Publishing status**, click **Publish app** and confirm.

After publishing:

- Re-authorize Omni (run its Google OAuth flow again) for each account. Refresh tokens issued while the app was in Testing still have the 7-day limit, but new ones issued in production don't.
- Because Omni requests sensitive or restricted scopes and isn't verified, the consent screen will show **"Google hasn't verified this app"**. As the owner, you can click **Advanced → Go to Omni (unsafe)** to continue.

## Verifying domain ownership in Google Search Console

Google requires domain verification for **brand verification** and **app verification**. Whether it's needed just to save the authorized domain or publish an unverified personal-use app can change; follow whatever the console asks for.

If you need it:

1. Sign in to [Google Search Console](https://search.google.com/search-console) with a Google account that is an **Owner or Editor** of the Cloud project.
2. Add a **URL prefix** property for `https://kiankian.github.io/`. A **Domain** property won't work, because it needs DNS records and you don't control DNS for `github.io`.
3. Verify it using one of these methods:
   - **HTML file:** download the `google<token>.html` file Search Console provides, add it to the root of this repository, commit, push, wait for Pages to redeploy, then click **Verify**. This file only proves site ownership; it isn't a secret.
   - **HTML tag:** add the `<meta name="google-site-verification" …>` tag to the `<head>` of `index.html`, push, then click **Verify**.
4. Keep the verification file or tag in place. Removing it can cause verification to lapse.

**Caveat:** don't assume a `github.io` subdomain will be accepted for brand or app verification. Google's homepage requirements say the homepage must be on a domain you own, and a shared hosting subdomain may be treated as a third-party platform. If you ever submit Omni for full verification and Google rejects the domain, the dependable fix is a custom domain you own (configured in GitHub Pages settings, verified in Search Console as a Domain property, then used for the homepage, privacy policy, and authorized domain).

## OAuth verification considerations for a personal-use app

- **Testing vs. production tokens.** In Testing, with External user type and scopes beyond basic profile, Google issues refresh tokens that expire after 7 days. In production, that 7-day limit no longer applies, even if the app is unverified.
- **Unverified apps and personal use.** Google's help center describes a personal-use case: an app that isn't shared with anyone else can be used by clicking past the unverified-app screen, without completing verification. Unverified apps are also limited to 100 new users over the app's lifetime, which doesn't matter for a single user.
- **Sensitive vs. restricted scopes.** Calendar scopes are *sensitive*, and Gmail read scopes such as `gmail.readonly` are *restricted*. Full verification for restricted scopes requires a third-party security assessment (CASA) and annual recertification. That's usually unnecessary for a single-owner personal tool. Only submit for verification if you plan to let other people use Omni.
- **Other ways refresh tokens can still stop working**, even in production:
  - You revoke access.
  - The token goes unused for 6 months.
  - You change your Google password (this revokes tokens that have Gmail scopes).
  - The per-account limit on live refresh tokens for one client is exceeded.
  - A Google Workspace admin restricts the app.
- **Keep the policy accurate.** If Omni adds scopes, services, or model providers, update `privacy.html`, `terms.html` (and their effective dates), and the homepage before requesting the new permissions.

### Official references

- [Google API Services User Data Policy](https://developers.google.com/terms/api-services-user-data-policy)
- [OAuth app verification help center](https://support.google.com/cloud/answer/13463073)
- [Unverified apps](https://support.google.com/cloud/answer/7454865)
- [Using OAuth 2.0 — refresh token expiration](https://developers.google.com/identity/protocols/oauth2#expiration)
- [Restricted scope verification](https://developers.google.com/identity/protocols/oauth2/production-readiness/restricted-scope-verification)
- [GitHub Pages: configuring a publishing source](https://docs.github.com/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)

Google changes these requirements and console labels from time to time, so check the pages above before relying on this guide.
