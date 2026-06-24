# Exiqo One

**Signed Jira & Confluence webhooks for your automation platform**

Exiqo One is an Atlassian Forge app that streams **Jira issue** and **Confluence page** events to your external platform. It enriches each payload, signs it with **HMAC-SHA256**, and delivers it to a webhook URL you configure — no manual webhook setup in Jira or Confluence.

[![Atlassian Marketplace](https://img.shields.io/badge/Atlassian-Marketplace-0052CC?style=flat-square&logo=atlassian)](https://marketplace.atlassian.com/)
[![Forge](https://img.shields.io/badge/Built%20with-Forge-0052CC?style=flat-square&logo=atlassian)](https://developer.atlassian.com/platform/forge/)
[![License](https://img.shields.io/badge/License-Commercial-blue?style=flat-square)](https://www.atlassian.com/licensing/marketplace)

---

## Table of contents

- [Overview](#overview)
- [How it works](#how-it-works)
- [Requirements](#requirements)
- [Installation](#installation)
- [Configuration](#configuration)
- [Supported events](#supported-events)
- [Webhook payloads](#webhook-payloads)
- [Signature verification](#signature-verification)
- [Permissions](#permissions)
- [Troubleshooting](#troubleshooting)
- [Data & privacy](#data--privacy)
- [Support](#support)

---

## Overview

| | |
|---|---|
| **Products** | Confluence (required) · Jira (optional) |
| **Delivery** | HTTPS POST with `X-Hub-Signature: sha256=...` |
| **Setup** | Install app → open Exiqo Settings → enter Webhook URL + Secret Token |
| **Platform** | Requires an external webhook receiver (e.g. Exiqo / Optima) |

### Why Exiqo One?

- **No admin webhook wiring** — events are captured via native Forge triggers
- **Secure by default** — GitHub-style HMAC-SHA256 signatures on every payload
- **Rich context** — user, project/space, issue or page, and sprint metadata (Jira)
- **Configure once** — settings sync across Confluence and Jira on the same site

---

## How it works

```
┌─────────────────────────────────────────────────────────────┐
│                 Your Atlassian Cloud Site                    │
│                                                              │
│   Jira (optional)              Confluence (required)         │
│   • Issue created              • Page created                │
│   • Issue updated              • Page updated                │
│              \                      /                        │
│               \   Exiqo One Forge  /                         │
│                • Enrich payload                              │
│                • HMAC-SHA256 sign                            │
│                • POST to your webhook                        │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
              ┌────────────────────────────┐
              │   Your automation platform   │
              │   Verify signature           │
              │   Run workflows / agents     │
              └────────────────────────────┘
```

Each event is sent as:

```http
POST /your/webhook/path HTTP/1.1
Content-Type: application/json
X-Hub-Signature: sha256=<hex-digest>

{ ... JSON payload ... }
```

---

## Requirements

### Atlassian

| Requirement | Details |
|-------------|---------|
| **Confluence Cloud** | Required — app installs here first |
| **Jira Cloud** | Optional — connect for issue events |
| **Site admin** | Needed to install the app and open settings |
| **Edition** | Atlassian Cloud (not Server / Data Center) |

### Your platform

| Requirement | Details |
|-------------|---------|
| **Webhook URL** | Public HTTPS endpoint that accepts `POST` |
| **Secret Token** | Shared secret for HMAC verification (you choose the value) |
| **Auth mode** | Platform must verify `X-Hub-Signature` (e.g. `hmac_sha256`) |

> **Note:** Exiqo One is a **connector**. It does not run workflows on its own — it delivers signed events to your platform.

---

## Installation

### 1. Install from Marketplace

1. Open the **Exiqo One** listing on the [Atlassian Marketplace](https://marketplace.atlassian.com/).
2. Click **Get app** / **Try it free**.
3. Select your Atlassian Cloud site.
4. Complete installation on **Confluence** (required product).
5. Optionally **connect Jira** for issue events.

### 2. Connect Jira (optional)

If Jira was not connected during install:

1. Go to **Atlassian Administration** → **Apps** → **Connected apps**
2. Find **Exiqo One** and connect **Jira**

### 3. Upgrade

When a new version is available:

**Settings** → **Manage apps** → **Exiqo One** → **Update**

---

## Configuration

Configure Exiqo One **once**. Settings automatically sync between Confluence and Jira when both are connected on the same site.

### Open Exiqo Settings

| Product | Path |
|---------|------|
| **Confluence** | ⚙️ Settings → **Apps** → **Exiqo Settings** |
| **Jira** | ⚙️ Settings → **Apps** → **Exiqo Settings** |

### Fields

| Field | Required | Description |
|-------|----------|-------------|
| **Webhook URL** | Yes | HTTPS endpoint provided by your platform vendor |
| **Secret Token** | Yes | Any string you choose — use the **same value** on your platform |

### Example

| Field | Example value |
|-------|----------------|
| Webhook URL | `https://v2.optima-rsystems.com/api/v1/webhooks/atlassian` |
| Secret Token | `my-team-secret-2026-abc123` |

### Steps

1. Paste your **Webhook URL**
2. Enter a **Secret Token**
3. Click **Save settings**
4. Configure the **same Secret Token** in your platform’s Atlassian connector
5. Create or update a test issue/page to confirm delivery

> If you change the Secret Token, update it in **both** Exiqo Settings and your platform.

---

## Supported events

| Product | Event | When it fires |
|---------|-------|----------------|
| Confluence | Page created | A new page is created |
| Confluence | Page updated | An existing page is updated |
| Jira | Issue created | A new issue is created |
| Jira | Issue updated | An existing issue is updated |

---

## Webhook payloads

All payloads include `source`, `event`, `eventName`, `timestamp`, `baseUrl`, `user`, and `project` (space for Confluence).

### Jira — issue created

<details>
<summary>View JSON example</summary>

```json
{
  "source": "jira",
  "event": "jira:issue_created",
  "eventName": "issue_created",
  "timestamp": "2026-06-18T10:00:00.000Z",
  "baseUrl": "https://your-site.atlassian.net",
  "user": {
    "displayName": "Jane Doe",
    "accountId": "abc123",
    "email": "jane@example.com"
  },
  "project": {
    "id": "10001",
    "key": "PROJ",
    "name": "My Project",
    "url": "https://your-site.atlassian.net/browse/PROJ"
  },
  "issue": {
    "id": "10042",
    "key": "PROJ-123",
    "url": "https://your-site.atlassian.net/browse/PROJ-123",
    "summary": "Fix login bug",
    "status": "To Do",
    "priority": "High",
    "assignee": "John Smith"
  },
  "sprint": {
    "id": "5",
    "name": "Sprint 1",
    "state": "active"
  }
}
```

</details>

### Jira — issue updated

Same structure as issue created. The `event` field reflects the update type:

```json
{
  "event": "jira:issue_updated:issue_generic",
  "eventName": "issue_generic"
}
```

### Confluence — page created / updated

<details>
<summary>View JSON example</summary>

```json
{
  "source": "confluence",
  "event": "avi:confluence:created:page",
  "eventName": "created",
  "timestamp": "2026-06-18T10:00:00.000Z",
  "baseUrl": "https://your-site.atlassian.net/wiki",
  "user": {
    "displayName": "Jane Doe",
    "accountId": "abc123",
    "email": "jane@example.com"
  },
  "project": {
    "id": "12345",
    "key": "DOCS",
    "name": "Documentation",
    "url": "https://your-site.atlassian.net/wiki/spaces/DOCS"
  },
  "page": {
    "id": "987654",
    "title": "Release Notes",
    "url": "https://your-site.atlassian.net/wiki/spaces/DOCS/pages/987654",
    "status": ""
  }
}
```

</details>

For page updates: `event` = `avi:confluence:updated:page`, `eventName` = `updated`.

---

## Signature verification

Exiqo One uses the same format as [GitHub webhooks](https://docs.github.com/en/webhooks/using-webhooks/validating-webhook-deliveries):

```http
X-Hub-Signature: sha256=<hex>
```

### Verification steps

1. Read the **raw request body** (before JSON parsing)
2. Compute `HMAC-SHA256(secretToken, rawBody)`
3. Hex-encode the digest
4. Compare with the value after `sha256=` (use constant-time comparison)

### Node.js example

```javascript
import crypto from 'crypto';

function verifySignature(secret, rawBody, signatureHeader) {
  if (!signatureHeader?.startsWith('sha256=')) {
    return false;
  }

  const expected = signatureHeader.slice('sha256='.length);
  const actual = crypto
    .createHmac('sha256', secret)
    .update(rawBody, 'utf8')
    .digest('hex');

  return crypto.timingSafeEqual(
    Buffer.from(actual, 'utf8'),
    Buffer.from(expected, 'utf8')
  );
}
```

### Python example

```python
import hashlib
import hmac


def verify_signature(secret: str, raw_body: bytes, signature_header: str) -> bool:
    if not signature_header.startswith("sha256="):
        return False
    expected = signature_header.removeprefix("sha256=")
    actual = hmac.new(
        secret.encode("utf-8"),
        raw_body,
        hashlib.sha256,
    ).hexdigest()
    return hmac.compare_digest(actual, expected)
```

---

## Permissions

Exiqo One requests permissions to:

| Area | Purpose |
|------|---------|
| Confluence / Jira admin settings | Host the Exiqo Settings page |
| Read issues, pages, users, spaces | Build enriched event payloads |
| App storage | Save webhook URL and secret token |
| External network (your webhook host) | Deliver signed events to your platform |

Event data is sent **only** to the webhook URL you configure.

---

## Troubleshooting

### Events not arriving

| Check | Action |
|-------|--------|
| Settings saved? | Open Exiqo Settings and confirm Webhook URL + Secret Token |
| Jira events missing? | Connect Jira on your site |
| Confluence events missing? | Confirm app is installed on Confluence |
| Secret mismatch? | Use the same token in Exiqo Settings and your platform |
| Endpoint reachable? | Ensure your server accepts HTTPS POST and returns 2xx |

### Signature verification fails

- Verify using the **raw body**, not re-serialized JSON
- Confirm the Secret Token matches in both places
- Header must be exactly `sha256=<lowercase-hex>`

### Settings not syncing

- Confirm both Confluence and Jira are connected on the **same** site
- Save settings again from either product
- Ensure Confluence has at least one space

### License issues

- Confirm your Marketplace license is active
- Upgrade to the latest version from **Manage apps**

---

## Data & privacy

Exiqo One:

- Stores your **Webhook URL** and **Secret Token** in Forge app storage
- May include user display name, account ID, and email in event payloads
- Sends data **only** to the webhook URL you configure

For privacy details, see: [R Systems Privacy Policy](https://www.rsystems.com/privacy-policy)

---

## Support

| Channel | Contact |
|---------|---------|
| **Email** | support@rsystems.com |
| **Documentation** | This repository |
| **Version** | 2.0.0 |

---

## Quick start checklist

- [ ] Install Exiqo One on Confluence (required)
- [ ] Connect Jira (optional, for issue events)
- [ ] Open **Exiqo Settings**
- [ ] Enter Webhook URL from your platform
- [ ] Choose a Secret Token and save
- [ ] Configure the same Secret Token on your platform
- [ ] Trigger a test issue or page update
- [ ] Confirm webhook received and signature verifies

---

<p align="center">
  <sub>Built by R Systems · Exiqo One © 2026</sub>
</p>
