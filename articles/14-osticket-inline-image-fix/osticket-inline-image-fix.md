### *osTicket Inline Image Failure After Server Migration: A Production Incident Analysis*

---

> *A step-by-step troubleshooting guide to diagnosing and fixing broken inline images in osTicket after migrating to a new server — covering CSRF tokens, session cookies, PHP compatibility, and configuration corruption.*

---

### Executive Summary

After migrating an osTicket helpdesk to a new server with an updated operating system, agents could no longer insert images (screenshots, logos) inside ticket replies. Images showed as a broken icon. The same osTicket package worked normally on the old server.

**Business Impact:** Agents could not share visual evidence such as error screenshots inside ticket responses, which slowed troubleshooting and communication with requesters. File attachments, ticket creation and other helpdesk functions continued to work.

**Root Cause (Plain Language):** The migration was incomplete, and the newer software platform enforces stricter browser security rules than the old one. Five issues stacked up:

1. The helpdesk still had the old server address configured.
2. The main configuration file was missing security settings and had a corrupted line.
3. The login cookie was sent in a form that modern browsers reject on non-HTTPS sites.
4. An earlier, unsuccessful manual fix had left broken files on the server.
5. A defect in osTicket 1.18.4 meant the image upload omitted its security token, so the server rejected it.

**Resolution:** All five issues were corrected. Inline image upload now succeeds, with the server-side anti-forgery (CSRF) protection kept fully enabled. Backups of every changed file were retained.

---

### 1. Environment

| Item | Old Server (Working) | New Server (Affected) |
|------|---------------------|----------------------|
| IP / URL | `10.20.30.50` | `10.20.30.40` |
| OS | Ubuntu 20.04 | Ubuntu 24.04 |
| PHP | 7.4.3 | 8.3.6 |
| osTicket | v1.15.1 | v1.18.4 |
| Web Server | Apache | Apache 2.4.58, mod_php, mod_rewrite |
| Install Path | `/var/www/osTicket/upload` | `/var/www/osTicket` (DB: `osticket_db`) |

---

### 2. Symptom and Key Evidence

- Inline image in the response editor showed a broken icon; draft text saved normally ("all changes saved").
- Browser console: Cookie `OSTSESSID` rejected because it has the `SameSite=None` attribute but is missing the `secure` attribute.
- Access log: `POST /scp/ajax.php/draft/14146/attach` returned **400**; the response body `Valid CSRF Token Required` was inserted as the image URL.
- osTicket syslog (log level raised temporarily): `Invalid CSRF token []` on `.../draft/14146/attach` — an empty token was received.
- Early 503 errors in testing came from the server's outbound proxy (`10.20.30.60:8080`), not from osTicket; local tests used `curl --noproxy '*'`.

---

### 3. Root Causes and Fixes

#### Issue 1: Stale Helpdesk URL

**Finding:** `helpdesk_url` in `ost_config` still pointed to the old server (`http://10.20.30.50/`).

**Fix:** Updated to `http://10.20.30.40/osticket/`.

---

#### Issue 2: Corrupted Configuration File

**Finding:** `include/ost-config.php` lacked `OSTINSTALLED` and `SECRET_SALT`. Line 26 was corrupted (`DEFAULT?TOPIC_ID`). PHP 8 makes undefined constants fatal.

**Fix:** Added both constants (salt copied from old server so existing encrypted data stays valid); fixed line 26; permissions set to `640 root:www-data`.

---

#### Issue 3: SameSite Cookie Rejection

**Finding:** `SameSite=None` on session cookie over HTTP. `Config::getAllowIframes()` returns `"'self'"` when empty, so `class.ostsession.php` always chose `None`.

**Fix:** Local patch — `include/class.ostsession.php` line 263 set to `'samesite' => 'Lax'`.

---

#### Issue 4: Broken Manual Patch Files

**Finding:** Failed manual patch of the editor JS left `.orig`, `.rej`, `.bak` files, a GitHub HTML page saved as `csrf_image_fix.diff`, and extra `scp/js/redactor-osticket.js` and `include/js` files.

**Fix:** Moved broken files to `/root/redactor_broken_2026-11-04/`. Core JS verified by MD5 as identical to upstream v1.18.4.

---

#### Issue 5: Empty CSRF Token on Upload

**Finding:** Stock 1.18.4 `scp/staff.inc.php` enforces CSRF on all POST/PUT/PATCH/DELETE, but the Redactor upload path sent no token.

**Fix:** Local patch — XHR wrapper appended to `js/redactor-osticket.js` that adds an `X-CSRFToken` header (from meta `csrf_token`) to non-GET XHRs. `checkCSRFToken()` already accepts this header.

---

### 4. Verification

```text
POST /scp/ajax.php/draft/ticket.staff.response/attach → HTTP 200
ost_syslog: no new "Invalid CSRF Token" entries after the fix
diff against clean tree: only .htaccess and the two documented local patches differ
```

---

### 5. Change Record and Backups

| File / Setting | Change | Backup |
|----------------|--------|--------|
| `ost_config.helpdesk_url` | Old IP replaced with new URL | n/a (DB value) |
| `include/ost-config.php` | +`OSTINSTALLED`, +`SECRET_SALT`, line 26 fix, chmod 640 | `/root/ost-config.php.bak.2026-11-04` |
| `include/class.ostsession.php` | SameSite forced to Lax (L263) | `/root/class.ostsession.php.bak.*` |
| `js/redactor-osticket.js` | Upstream restored + CSRF header snippet appended | `/root/redactor-osticket.js.upstream-1.18.4` |
| Leftover patch files | Removed from web root | `/root/redactor_broken_2026-11-04/` |
| `ost_config.log_level` | Temporarily 3 (debug) for diagnosis | Reset to 1 after verification |

---

### 6. Key Recommendations

| # | Recommendation | Priority |
|---|---------------|----------|
| 1 | Move the helpdesk to HTTPS. Staff logins currently travel unencrypted, and HTTPS removes the need for the cookie patch. | High |
| 2 | Rotate the osTicket database password and set a MySQL root password. | High |
| 3 | Record the two local code patches in the change register and re-apply them after any osTicket upgrade. | Medium |
| 4 | Ask all agents to clear their browser cache once to load the corrected script. | Medium |
| 5 | Use a migration checklist (config, URL, PHP compatibility, cookie/HTTPS) for future server moves. | Low |

---

### 7. Open Actions

| Action | Detail | Priority |
|--------|--------|----------|
| Enable HTTPS | Install a certificate on the new server; then `SameSite=None; Secure` works and the `class.ostsession.php` patch can be reverted. | High |
| Credential hygiene | Rotate the DB password for user `osticket` (exposed during troubleshooting); set a MySQL root password; consider rotating `SECRET_SALT` only with a planned re-encryption. | High |
| Reset log level | `UPDATE ost_config SET value='1' WHERE \`key\`='log_level'` (if not done). | Medium |
| Upgrade runbook | Re-apply both local patches after any osTicket upgrade, or confirm upstream has fixed them. | Medium |
| Tidy duplicates | Remove the unused `/var/www/osTicket/config/ost-config.php` and stale backup folders under `/var/www`. | Low |
| Upload limits | Consider raising `upload_max_filesize` (2M) / `post_max_size` (8M) to match helpdesk policy. | Low |

---

### Troubleshooting Principles

**1. Read the error, don't guess.** The browser console and access logs told the exact story — a rejected cookie and a missing CSRF token.

**2. Fix the root cause, not the symptom.** Restarting Apache or clearing caches would not have helped. The issues were in configuration, session handling, and a code defect.

**3. Clean up after failed manual fixes.** Broken `.orig`, `.rej`, and `.bak` files from earlier attempts can cause confusion and unexpected behavior.

**4. Verify with evidence.** HTTP 200 on upload, no new CSRF errors in syslog, and a clean diff against upstream — three independent confirmations.

**5. Document every change.** File backups, change records, and a clear incident report make future upgrades and audits straightforward.

---

*Disclaimer: This incident report is based on a real production troubleshooting scenario. IP addresses, usernames, and sensitive configuration values have been anonymized. The database password and SECRET_SALT are intentionally excluded.*

---

**Tags:** #osTicket #Helpdesk #CSRF #ServerMigration #PHP #Linux #DevOps #IncidentResponse #Troubleshooting #Ubuntu

---

## 📂 GitHub Repository

📋 **Explore the complete article and troubleshooting resources:**

👉 **[GitHub Repository: Medium-Blog](https://github.com/iqbalbd37/Medium-Blog)**

📄 **Read the full osTicket Incident article in Markdown:**

👉 [osTicket Inline Image Failure After Server Migration — Full Article](https://github.com/iqbalbd37/Medium-Blog/blob/main/articles/14-osticket-inline-image-fix/osticket-inline-image-fix.md)

### What You'll Find in This Repository

- ✅ Complete osTicket migration troubleshooting guide
- ✅ CSRF token and session cookie configuration fixes
- ✅ PHP 8 compatibility and corrupted config file recovery
- ✅ Step-by-step verification and change record documentation
- ✅ Production incident report with anonymized infrastructure

⭐ **If you find these resources useful, consider starring the repository on GitHub!**

*Learn, troubleshoot, and share your DevOps knowledge.*

---