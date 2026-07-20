# SaaS Verified Usage and Flexera Export Workbook

## Purpose
This solution provides trusted, user-level SaaS usage visibility in Microsoft Sentinel and prepares export-ready data for Flexera One SaaS Management. It separates simple URL discovery from verified sign-in and verified in-app activity.

## Architecture
The workbook and KQL model unify evidence from Cloud Discovery, cloud app activity, and identity sign-in sources.

Evidence layers:
1. URL evidence from MDCA Cloud Discovery (McasShadowItReporting).
2. Verified application activity from CloudAppEvents.
3. Verified sign-in from Ping Identity custom logs (PingIdentity_CL) when available.
4. Verified sign-in fallback from Entra sign-in logs (SigninLogs or AADSignInEventsBeta).

Classification precedence:
1. Verified Activity
2. Verified Sign-In
3. URL Access Only
4. Unknown

## Required data sources
- McasShadowItReporting (Cloud Discovery / Shadow IT)

## Optional data sources
- CloudAppEvents (Defender for Cloud Apps activity for connected apps)
- SigninLogs or AADSignInEventsBeta (Entra sign-in evidence)
- PingIdentity_CL (custom Ping Identity logs)
- UserInventory_CL (optional enrichment for name, country, department)

## Table mapping
- McasShadowItReporting: URL access/discovery evidence
- CloudAppEvents: API/activity evidence
- SigninLogs or AADSignInEventsBeta: identity sign-in evidence
- PingIdentity_CL: identity sign-in evidence (custom connector)
- UserInventory_CL: HR/user profile enrichment (optional)

## Workbook sections
1. Executive Summary
2. SaaS Application Summary
3. User-Level SaaS Inventory
4. Verification Evidence
5. Flexera Export
6. Data Quality and Gaps

## Known limitations
- MDCA Cloud Discovery can show app/URL usage but is not proof of successful authentication.
- CloudAppEvents coverage depends on app connector and telemetry availability.
- Third-party app naming may differ between discovery and connected-app telemetry.
- PingIdentity_CL is custom and may require schema mapping updates.
- If country/department attributes are not present in Sentinel, enrich from UserInventory_CL or Entra/HR source.

## Validate with sample applications
Validate behavior for:
- Ping Identity
- Salesforce Marketing Cloud
- Workday
- Docker

Recommended validation checks:
1. Confirm each sample app appears in Cloud Discovery evidence where expected.
2. Confirm verified classification only appears when CloudAppEvents or sign-in evidence exists.
3. Confirm URL-only users are not counted as verified users.
4. Confirm Last active and Total days used are populated correctly.
5. Confirm Flexera export table returns expected columns and user-app records.
