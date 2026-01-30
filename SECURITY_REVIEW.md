# Security Review: offline-memos

**Repository:** https://github.com/MountComb/offline-memos  
**Review Date:** January 30, 2026  
**Reviewer:** Automated Security Analysis

## Executive Summary

**VERDICT: SAFE TO USE** ✓

After a thorough code review, this application appears to be **safe for production use**. The code does NOT send your credentials to any third party. All network communication goes exclusively to the Memos API endpoint that YOU configure.

---

## Detailed Analysis

### 1. Network Requests Review

All network requests in the application are made through two files:

#### `src/lib/memos-api.ts`
- Contains a single `testConnection()` method
- Only makes requests to `${settings.apiEndpoint}/api/v1/auth/sessions/current`
- Uses the user-provided `apiEndpoint` (your own server)
- No hardcoded external URLs

#### `src/lib/sync.ts`
- Contains sync functionality (`createRemoteNote`, `updateRemoteNote`, `syncWithRemote`)
- All requests go to `${apiUrl}/api/v1/memos` endpoints
- `apiUrl` is sourced from `settings.apiEndpoint` (user-configured)
- No external services contacted

**Finding:** ✅ All network calls use only the user-configured endpoint. No third-party servers are contacted.

---

### 2. Credential Storage

#### `src/lib/settings.ts`
- Credentials stored in browser `localStorage` under key `memos-offline-settings`
- Credentials remain local to your browser
- No transmission of settings to external servers
- No encoding/encryption (stored as plain JSON locally)

**Finding:** ✅ Credentials are stored locally and never transmitted to third parties.

---

### 3. Hardcoded URLs/Endpoints

A comprehensive search found:

- **No hardcoded API endpoints**
- **No analytics services** (Google Analytics, Mixpanel, etc.)
- **No tracking pixels**
- **No external service integrations**
- Only placeholder text: `https://your-memos-instance.com` (in input field as example)

**Finding:** ✅ No suspicious hardcoded URLs detected.

---

### 4. Dependencies Analysis

All dependencies in `package.json` are well-known, reputable packages:

| Package | Purpose | Risk |
|---------|---------|------|
| react, react-dom | UI framework | ✅ Safe |
| react-router-dom | Routing | ✅ Safe |
| @radix-ui/* | UI components | ✅ Safe |
| dexie | IndexedDB wrapper | ✅ Safe |
| tailwindcss | CSS framework | ✅ Safe |
| lucide-react | Icons | ✅ Safe |
| react-markdown | Markdown rendering | ✅ Safe |
| sonner | Toast notifications | ✅ Safe |
| vite-plugin-pwa | PWA support | ✅ Safe |

**Note:** `workbox-google-analytics` appears in `pnpm-lock.yaml` as a transitive dependency of Workbox/vite-plugin-pwa, but is NOT used or configured in the application code.

**Finding:** ✅ All dependencies are standard, widely-used packages with no known security concerns.

---

### 5. Suspicious Patterns Check

Searched for and found NONE of the following:

- ❌ `eval()` - Not found
- ❌ `Function()` constructor - Not found
- ❌ `atob()`/`btoa()` encoding - Not found
- ❌ `WebSocket` connections - Not found
- ❌ `navigator.sendBeacon` - Not found
- ❌ `postMessage` - Not found
- ❌ Obfuscated code - Not found
- ❌ Base64 encoded strings - Not found
- ❌ External analytics/tracking - Not found

**Finding:** ✅ No suspicious code patterns detected.

---

### 6. Data Flow Summary

```
┌─────────────────┐
│   Your Browser  │
│                 │
│  ┌───────────┐  │
│  │localStorage│ │  Credentials stored locally
│  └───────────┘  │
│                 │
│  ┌───────────┐  │
│  │ IndexedDB │  │  Notes stored locally (via Dexie)
│  └───────────┘  │
│                 │
└────────┬────────┘
         │
         │ API calls (only when you sync)
         │
         ▼
┌─────────────────┐
│ YOUR Memos      │
│ Server          │  The endpoint YOU configure
│ (your-domain)   │
└─────────────────┘

NO DATA SENT TO:
- The app developer
- Any third-party services
- Any analytics providers
```

---

## Risk Assessment

| Category | Risk Level | Notes |
|----------|------------|-------|
| Credential Theft | ✅ NONE | Credentials only sent to your server |
| Data Exfiltration | ✅ NONE | Notes only sync to your server |
| Tracking/Analytics | ✅ NONE | No tracking code present |
| Malicious Dependencies | ✅ NONE | All deps are standard packages |
| Code Obfuscation | ✅ NONE | Clear, readable source code |

---

## Recommendations

1. **Safe to use** - This application is suitable for production use with your personal Memos server.

2. **Self-host if desired** - Since it's a static PWA, you can build and host it yourself for additional assurance:
   ```bash
   git clone https://github.com/MountComb/offline-memos
   cd offline-memos
   pnpm install
   pnpm build
   # Deploy the 'dist' folder to your own hosting
   ```

3. **Optional security hardening**:
   - Use HTTPS for your Memos server
   - Generate a unique access token for this app
   - Regularly rotate your access tokens

---

## Conclusion

This offline-memos application is a straightforward PWA client for the Memos API. The code is clean, well-organized, and contains no backdoors, tracking, or credential stealing mechanisms. **You can safely use this application** - your credentials and notes will only be sent to the Memos server endpoint that you configure in the settings.
