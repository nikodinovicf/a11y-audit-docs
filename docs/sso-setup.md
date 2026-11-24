# SSO setup guide

This guide explains how to audit an app behind single sign-on (SSO) with the most common identity providers. Each provider section covers:

- what to request from the client
- which hostnames to add to `a11y.config.json`
- what to click during login
- the problems you are likely to hit

The extension never automates the identity provider. **You** log in once in a real Chrome window with **A11y: Login (SSO)**. The extension saves the browser session (cookies, `localStorage`, and the app's `sessionStorage`) and reuses it for every **A11y: Run Audit** until the session expires. MFA, push approvals and CAPTCHAs therefore work as usual.

## Contents

- [Before you start: what to request from the client](#before-you-start-what-to-request-from-the-client)
- [Auditing with your personal account](#auditing-with-your-personal-account)
- [How login works](#how-login-works)
- [Microsoft Entra ID (Azure AD, Microsoft 365)](#microsoft-entra-id-azure-ad-microsoft-365)
- [Okta](#okta)
- [Google Workspace / Sign in with Google](#google-workspace--sign-in-with-google)
- [Auth0](#auth0)
- [OneLogin](#onelogin)
- [Ping Identity (PingOne, PingFederate)](#ping-identity-pingone-pingfederate)
- [AD FS and other self-hosted SAML providers](#ad-fs-and-other-self-hosted-saml-providers)
- [Any other provider: find the hostname yourself](#any-other-provider-find-the-hostname-yourself)
- [Troubleshooting](#troubleshooting)

---

## Before you start: what to request from the client

Send this list to the client before the audit starts. Most SSO problems come from a missing item on it.

1. **A dedicated test account**, not a real employee's account. Give it the lowest role that still reaches every page in scope. If some pages exist only for admins, ask for a second, admin test account.
2. **A staging or UAT environment**, if one exists. The audit loads every listed page, and a page that changes data on a plain page load would also do so in production.
3. **An MFA method you can complete.** Either enroll your own phone (authenticator app or SMS) for the test account, or have someone at the client approve pushes during the login. You log in only once per session, not once per page.
4. **An exception from device-based access policies.** Many enterprises allow only managed or compliant devices. Your Chrome is not managed by them, so ask them to exclude the test account from these policies or to allow it on staging (see each provider below).
5. **A session lifetime of at least 8 hours** for the test account, so the session does not expire mid-audit.
6. **The list of pages in scope**: URLs or paths, including any page that needs a specific role or data.
7. **The SSO login hostname**, if they use a custom domain such as `login.client.com`. You can also find it yourself (see [below](#any-other-provider-find-the-hostname-yourself)).

Copy-paste version for an email:

> For the accessibility audit we need:
> 1. A dedicated test account (lowest role that can reach all pages in scope; a second admin account if admin pages are in scope).
> 2. Access to staging/UAT if available.
> 3. An MFA method the auditor can complete (enrolling the auditor's authenticator app is easiest).
> 4. The test account excluded from device-compliance / managed-device access policies.
> 5. A session lifetime of at least 8 hours for that account.
> 6. The list of URLs or pages in scope.
> 7. The hostname of your SSO login page if it is a custom domain.

If the client gives no test account and you log in with your own, see the next section.

---

## Auditing with your personal account

A test account is a recommendation, not a requirement. Any account works if you can complete its login and MFA yourself. With your own account you approve MFA on your own phone, and nobody at the client has to approve anything. Keep these points in mind:

1. **The audit sees what your account sees.** Pages your role cannot open are not audited. Do not claim them in the VPAT. Write the role you tested with in the VPAT's notes, for example "Evaluated as a standard user; admin pages not in scope". To cover admin pages, someone with that role must run the audit.
2. **Interaction steps act as you, on real data.** Every step in `states` is your own action in the app. On production, use only steps that open or show something: dialogs, menus, form validation errors. Never write steps that save, submit, send, delete or pay.
3. **The saved session is your personal access.** Anyone holding it is logged in as you. It is encrypted in the OS keychain, but run **A11y: Clear Session** right after each audit. Do not keep it for days.
4. **`.a11y/` contains client data as your account sees it:** URLs, HTML snippets, and possibly names or other personal data shown on the pages. The folder is git-ignored. Do not send it to anyone or upload it anywhere. The exported report also contains page URLs and HTML snippets, so read it before you share it.
5. **Session lifetime and device rules still apply.** If your account must sign in again every hour, the audit stops each time and you complete MFA again. Audit fewer pages per run, or ask whether the sign-in frequency can be longer. If your account may sign in only from a company-managed device, run the extension on that device.

Of the client checklist above, items 2, 6 and 7 still apply: staging if it exists, the pages in scope, and the SSO hostname.

## How login works

1. Set `baseUrl` and `pages` in `a11y.config.json`.
2. Run **A11y: Login (SSO)**. Chrome opens on `baseUrl`, and the app redirects you to the identity provider.
3. Log in as usual. If the provider asks **"Stay signed in?"** or shows a **"Keep me signed in"** checkbox, choose **Yes** or check it. This gives a longer-lived session.
4. Wait until the app itself has fully loaded, for example the dashboard. Then click **Done** in the VS Code notification.
5. Run **A11y: Run Audit**.

When a page redirects to one of the identity-provider hostnames during the audit, the extension treats the session as expired. It stops, saves partial results, and offers **Log In**. That is why the hostname list matters.

Run Audit does not ask for a login first: a public site needs none, and a site behind SSO stops at the first redirect to the identity provider. An app that redirects to its **own** login page (for example `/login` on the app's hostname) is not detected this way, and the login page would be audited instead. For such apps, always run **A11y: Login (SSO)** before the audit.

**Built-in identity-provider hostnames.** A hostname also matches all of its subdomains, so `okta.com` matches `acme.okta.com`.

| Provider | Built-in hostnames |
|---|---|
| Microsoft | `login.microsoftonline.com`, `login.microsoftonline.us`, `login.microsoft.com`, `login.windows.net`, `login.live.com`, `b2clogin.com`, `ciamlogin.com` |
| Okta | `okta.com`, `oktapreview.com`, `okta-emea.com` |
| Google | `accounts.google.com` |
| Auth0 | `auth0.com` |
| OneLogin | `onelogin.com` |
| Ping | `pingidentity.com`, `pingone.com` |

Any **custom domain** (for example `login.client.com` or `sso.client.com`) must be added with `idpHosts`.

---

## Microsoft Entra ID (Azure AD, Microsoft 365)

The most common provider in US enterprises.

**Config**

```json
{
  "baseUrl": "https://staging.client.com",
  "pages": ["/dashboard", "/settings"]
}
```

No `idpHosts` entry is needed for standard Entra ID, Azure AD B2C (`*.b2clogin.com`), Entra External ID (`*.ciamlogin.com`), or US Government cloud (`login.microsoftonline.us`). If the tenant is federated to AD FS or another provider, add that host too; see [AD FS](#ad-fs-and-other-self-hosted-saml-providers).

**During login**

- Enter the test account email, then the password.
- Complete MFA. Microsoft Authenticator may show a two-digit number that must be typed into the phone (number matching).
- On **"Stay signed in?"** choose **Yes**.
- If a **"Pick an account"** screen appears, choose the test account.

**What to request from the client's admin**

- **Conditional Access:** exclude the test account from policies that require a compliant, Hybrid-joined or app-protected device. Without this, login ends on *"You can't get there from here"* (error `AADSTS53000`).
- **Sign-in frequency:** if a Conditional Access policy sets a short sign-in frequency, ask for at least 8 hours for the test account.
- **Guest accounts:** if they invite your own Microsoft account as a guest instead of creating a test account, you will see their tenant's MFA prompts plus your home tenant's.

**Apps that use MSAL.js (tokens in `sessionStorage`)**

Many single-page apps on Entra ID use MSAL.js, which by default keeps tokens in `sessionStorage`. Browsers clear `sessionStorage` when the window closes, so the extension copies it separately. When you click **Done**, it reads the app tab's `sessionStorage` and restores it into every audited page before the app's own scripts run.

For this to work:

- Click **Done** only while the **app** tab is open and fully loaded, not the Microsoft login page.
- Keep the app on the same origin as `baseUrl` (same scheme, host and port).

Restored tokens are valid only until they expire. MSAL renews access tokens with its refresh token, but that also expires (typically within 24 hours for single-page apps). If an audit later stops with *Session expired*, run **A11y: Login (SSO)** again.

If it still fails, also check:

1. That the test account is the **only** account you logged in with in that Chrome window, so Microsoft does not show a **"Pick an account"** screen.
2. Whether a staging build can set MSAL `cache: { cacheLocation: "localStorage" }`.

---

## Okta

**Config**

```json
{
  "baseUrl": "https://staging.client.com",
  "pages": ["/dashboard", "/settings"],
  "idpHosts": ["login.client.com"]
}
```

`*.okta.com`, `*.oktapreview.com` and `*.okta-emea.com` are built in. Many companies put Okta on a **custom URL domain** such as `login.client.com` or `sso.client.com`. Add that host to `idpHosts`, or session expiry will not be detected.

**During login**

- Enter the username, then the password (some orgs ask for the password last, or not at all with passwordless login).
- Complete MFA. Okta Verify push or a 6-digit code works.
- If **Okta FastPass** is offered and fails in this Chrome window, choose **Verify with something else** and use push or a code.
- Check **"Keep me signed in"** if the sign-in page shows it.

**What to request from the client's admin**

- **Authentication policies / device assurance:** exclude the test account from rules that require a managed or registered device, or Okta FastPass only.
- **Global session policy:** the maximum Okta session lifetime for the test account should be at least 8 hours.
- **App assignment:** the test account must be assigned to the app in Okta. Otherwise the login succeeds but Okta shows *"User is not assigned to the client application"*.

**Token storage:** Okta's JavaScript SDKs store tokens in `localStorage` by default, and that is saved with the session. Okta apps normally work without extra steps.

---

## Google Workspace / Sign in with Google

**Config**

```json
{
  "baseUrl": "https://staging.client.com",
  "pages": ["/dashboard", "/settings"]
}
```

`accounts.google.com` is built in.

**During login**

- Choose or enter the test account, then the password, then 2-Step Verification.

**Known problem: "This browser or app may not be secure"**

Google sometimes refuses to sign in a browser that is controlled by automation software, and the extension's login window is such a browser. If you see *"Couldn't sign you in. This browser or app may not be secure"*:

1. Ask the client for a test account that logs in **without Google**, for example a username/password login on staging, if the app supports one.
2. If the app also supports another provider (Microsoft, Okta), use that.
3. If neither is possible, audit the public pages with this tool, and audit the pages behind login manually with axe DevTools in your normal Chrome. Track this as a known limitation in the audit report.

**What to request from the client's admin:** context-aware access rules in Google Workspace can block unmanaged devices. Ask them to exclude the test account.

---

## Auth0

**Config**

```json
{
  "baseUrl": "https://staging.client.com",
  "pages": ["/dashboard", "/settings"],
  "idpHosts": ["auth.client.com"]
}
```

`*.auth0.com` (including regional tenants such as `client.us.auth0.com`) is built in. Add the custom domain if the client uses one, which is common.

**During login:** use the Auth0 Universal Login page as usual, plus any MFA the tenant enforces. Auth0 can also forward you to an enterprise provider (Entra ID, Okta, Google). In that case, also follow that provider's section.

**Known problem: tokens kept in memory.** By default the Auth0 SPA SDK keeps tokens in memory only. On a fresh page load it silently asks Auth0 for new tokens using the Auth0 session cookie, which is saved with the session. This usually works when the client uses a custom domain on the same site as the app. If the audit reports *Session expired* on every page right after a successful login, ask the client whether a staging build can use `cacheLocation: "localstorage"` together with refresh tokens.

---

## OneLogin

**Config**

```json
{
  "baseUrl": "https://staging.client.com",
  "pages": ["/dashboard", "/settings"]
}
```

`*.onelogin.com` is built in. Add a custom domain to `idpHosts` if one is used.

**During login:** username, password, then OneLogin Protect push or a code.

**What to request:** the test account must be assigned the app in OneLogin, and its user policy should allow a session of at least 8 hours.

---

## Ping Identity (PingOne, PingFederate)

**Config**

```json
{
  "baseUrl": "https://staging.client.com",
  "pages": ["/dashboard", "/settings"],
  "idpHosts": ["sso.client.com"]
}
```

- **PingOne** (cloud): `*.pingone.com` and `*.pingidentity.com` are built in.
- **PingFederate** (self-hosted): always runs on the client's own domain, for example `sso.client.com` or `federation.client.com`. You **must** add it to `idpHosts`.

**During login:** username, password, then PingID push or a code.

**What to request:** exclude the test account from device-trust policies, and allow a session of at least 8 hours.

---

## AD FS and other self-hosted SAML providers

Some companies still run Active Directory Federation Services (AD FS) or another SAML provider on their own servers. Microsoft Entra ID tenants are often federated to AD FS too, in which case Microsoft forwards you there after you enter your email.

**Config:** add the AD FS host. You can recognise its URLs because they contain `/adfs/ls/`.

```json
{
  "baseUrl": "https://staging.client.com",
  "pages": ["/dashboard", "/settings"],
  "idpHosts": ["adfs.client.com"]
}
```

**What to request:** AD FS is often reachable only from the corporate network or VPN. Ask whether you need VPN access to log in. If you do, connect to the VPN before **A11y: Login (SSO)** and stay connected during the audit.

---

## Any other provider: find the hostname yourself

1. Run **A11y: Login (SSO)**.
2. Before you type anything, look at the address bar in the Chrome window that opened. Note the **hostname** of every login page you pass through. A hostname is the part between `https://` and the next `/`, for example `login.client.com`.
3. Add every hostname that is **not** the app itself to `idpHosts`.

Do **not** add the app's own hostname. Every page would then look like an expired session, and the audit would stop on the first page.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Audit stops on the first page with *Session expired*, right after a successful login | You clicked **Done** on the login page instead of the app, or the app keeps tokens only in memory (Auth0 SPA SDK) | Log in again and click **Done** only when the app has loaded. For memory-only tokens, see [Auth0](#auth0). |
| Audit stops with *Session expired* after some pages | The session really expired, or the IdP's sign-in frequency is short | Choose **Log In**, check **Stay signed in** / **Keep me signed in**, and ask for a longer session lifetime. |
| Audit runs, but every page shows the login screen's issues | The provider uses a custom domain that is not in `idpHosts`, so expiry was not detected | Find the hostname ([above](#any-other-provider-find-the-hostname-yourself)) and add it to `idpHosts`. |
| Audit stops on the first page even with a fresh login | The app's own hostname is in `idpHosts`, or the app is hosted on an IdP domain | Remove the app's hostname from `idpHosts`. |
| *"You can't get there from here"* / `AADSTS53000` (Microsoft) | Conditional Access requires a compliant device | Ask the client to exclude the test account from that policy. |
| *"User is not assigned to the client application"* (Okta) | The test account has no access to the app | Ask the client to assign the app to the account. |
| *"This browser or app may not be secure"* (Google) | Google blocks sign-in from automated browsers | See [Google](#google-workspace--sign-in-with-google). |
| Login page never loads, or times out | AD FS or the app is reachable only on VPN or from an allow-listed IP | Connect to the VPN, or ask the client to allow your IP for staging. |
| You clicked **Done** and got *"The browser is still on the identity provider page"* | Login had not finished yet | Click **Cancel**, finish the login, and run **A11y: Login (SSO)** again. |

### Verify the setup

Do this once per client app, before the real audit:

1. Run **A11y: Login (SSO)**, log in, and click **Done**. The notification says *session saved*.
2. Add one page to `pages` that needs login, and run **A11y: Run Audit**. It must finish without *Session expired*, and the result must not show the login screen's issues.
3. Run **A11y: Clear Session**, then **A11y: Run Audit** again. It must now stop with a prompt to log in. If it does not, the identity provider's hostname is missing from `idpHosts`.

After the audit, run **A11y: Clear Session** and tell the client that the test account can be disabled.
