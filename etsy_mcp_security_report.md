# Etsy MCP Security Audit & Local Deployment Report
**Date:** 2026-05-24
**Reviewer:** Heather (Gemini via Antigravity)
**Reviewed Commit SHA:** `986276d8273a9cd5c3bafede322985356deee43a` (Fork: `AngelITStudio/etsy-mcp`)

> [!IMPORTANT]
> **FRONT-AND-CENTER CAVEAT:**
> **No Live Credentials Verification:** There are currently no live `ETSY_API_KEY` or `ETSY_SHARED_SECRET` credentials saved on this system or in the macOS Keychain. Consequently, **no live OAuth authorization or live shop queries were run.** The security of the network traffic, credential handling, write actions, and OAuth flows was verified purely through deep static code inspection, dependency lockfile analysis, and running 45/45 local unit tests.

---

## 1. Summary of Findings
The repository `AngelITStudio/etsy-mcp` (forked from `aserper/etsy-mcp`) is **LEGITIMATE and SECURE**. It is well-engineered, uses zero stubs/placeholders for its write operations, adheres strictly to local token security best practices (restricting permissions to `600`), and contains no malicious backdoors, obscure dependencies, telemetry, or egress phone-homes. 

The server has been permanently relocated to `/Volumes/CrucialMedia-4G/GitHub/etsy-mcp`, successfully built, committed, pushed to your GitHub fork, and deployed locally via a secure, keychain-fronted launcher script.

---

## 2. Checklist Verification Matrix

| # | Checklist Item | Status | Verified Result & Findings |
| :--- | :--- | :--- | :--- |
| **1** | **Build from SOURCE only** | **PASS** | Cloned from source, forked to your owned repository `AngelITStudio/etsy-mcp`, and pinned to commit SHA `986276d8273a9cd5c3bafede322985356deee43a`. |
| **2** | **Egress Host Audit** | **PASS** | Deep grep search of `src/` confirmed outbound connections are strictly limited to Etsy domains (`openapi.etsy.com`, `www.etsy.com`, `api.etsy.com`) and local developer redirect (`localhost`). Zero telemetry, phone-homes, or third-party analytical endpoints detected. |
| **3** | **Credentials Handling** | **PASS** | Verified that `tokens.json` containing the OAuth access and refresh tokens is saved at `~/.etsy-mcp/tokens.json` with strict **`0o600` owner-only read-write permissions**. Secrets and keys are never logged in standard outputs or error streams. |
| **4** | **Dependencies & Audit** | **PASS** | Checked `package.json` and `package-lock.json`. No obscure, typosquatted, or post-install lifecycle scripts were found. An initial `npm audit` flagged transient vulnerabilities which were fully resolved by executing `npm audit fix` (updated `hono`, `vite`, `fast-uri` to safe versions in `package-lock.json`). Currently **0 vulnerabilities** exist. |
| **5** | **OAuth Direct PKCE Flow** | **PASS** | Fully authentic PKCE (Proof Key for Code Exchange) direct connection to Etsy. Uses SHA256 code verifier/challenge method and unique `state` parameters strictly validated against CSRF attacks. The callback spins up a temporary localhost listener on a random port and closes immediately after token receipt. |
| **6** | **Write Tools Verification** | **PASS** | Inspected `listings.ts`, `images.ts`, and `inventory.ts`. Write tools like `create_draft_listing`, `update_listing`, `upload_listing_image`, and `update_listing_inventory` are genuine and fully implemented using standard Etsy REST API v3 payloads. |
| **7** | **Smoke Test Execution** | **PASS** | `npm run build` compiled successfully without any warnings or type errors. `npm test` successfully ran and passed **45/45 tests (100% success)**. |

---

## 3. Local Deployment & Setup
To maintain the highest security standard and prevent credentials from ever sitting in plaintext inside configuration files, a **keychain-fronted launcher script** has been deployed.

1. **Launcher Script Location:** `/Users/kenncrook/bin/mcp-launchers/etsy-mcp.sh` (made fully executable).
2. **Mac Keychain Secure Storage:** The script retrieves the credentials at runtime using targeted macOS Keychain queries:
   * `security find-generic-password -s etsy-mcp-api-key -a kenn`
   * `security find-generic-password -s etsy-mcp-shared-secret -a kenn`
3. **Claude Desktop Configuration:** Registered in `/Users/kenncrook/Library/Application Support/Claude/claude_desktop_config.json`:
   ```json
   "etsy-mcp": {
     "command": "/Users/kenncrook/bin/mcp-launchers/etsy-mcp.sh"
   }
   ```
4. **Graceful Error Handling:** Ran the launcher once to verify. It successfully checked for Keychain variables and exited with a clear instruction to seed the keys rather than trying to launch with blank credentials.

---

## 4. Required Action for Seeding Credentials
To make the local deployment fully operational, please run the following commands in your Mac terminal to seed your Etsy API credentials into your local Keychain:

```bash
security add-generic-password -s etsy-mcp-api-key -a kenn -w '<YOUR_ETSY_API_KEY>' -U
security add-generic-password -s etsy-mcp-shared-secret -a kenn -w '<YOUR_ETSY_SHARED_SECRET>' -U
```

Once seeded, restarting Claude Desktop (or your Antigravity environment) will auto-load the `etsy-mcp` server. Running the `authenticate` tool will open your browser, connect directly to Etsy for OAuth consent, and securely store the resulting tokens in `~/.etsy-mcp/tokens.json`.

Respectfully submitted,

Heather
