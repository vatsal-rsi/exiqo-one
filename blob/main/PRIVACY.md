# Exiqo One — Privacy Policy

**Last updated:** June 2026  
**Publisher:** Exiqo One (testing release)  
**Contact:** vatsal.patel@rsystems.com

## Overview

Exiqo One is an Atlassian Forge app that sends Jira and Confluence event data to a webhook URL you configure on your automation platform.

## Data we collect and store

| Data | Where | Purpose |
|------|-------|---------|
| Webhook URL | Forge app storage | Deliver signed event payloads to your platform |
| Secret Token | Forge app storage | Sign outbound webhooks (HMAC-SHA256) |
| User display name, account ID, email | Event payloads only | Enrich webhook context for the actor who triggered the event |
| Issue, page, project/space metadata | Event payloads only | Enrich webhook context |

## Data we send off Atlassian

Signed JSON event payloads are POSTed **only** to the **Webhook URL** you enter in Exiqo Settings. We do not send data to any other destination.

## Data retention

- Settings (Webhook URL, Secret Token) remain in Forge app storage until you change or uninstall the app.
- Your receiving platform controls retention of webhook payloads.

## Your choices

- You choose the Webhook URL and Secret Token.
- You can update or remove settings anytime from **Exiqo Settings** in Confluence or Jira admin.
- Uninstalling the app removes Forge app storage for your site.

## Security

Payloads are signed with `X-Hub-Signature: sha256=<hex>` using your Secret Token. Your platform should verify signatures before processing events.

## Contact

Questions about privacy or data handling:

**vatsal.patel@rsystems.com**
