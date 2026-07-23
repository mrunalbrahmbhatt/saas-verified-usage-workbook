# Verified SaaS Usage – Identity Journey (Microsoft Sentinel Workbook)

An Azure Monitor / Microsoft Sentinel workbook that answers one ITAM question for **cba**:

> *For each SaaS application, which users are really using it — and is that usage backed by a verified corporate sign-in (MFA / Conditional Access) or not?*

It presents the same **App × User** question through **four complementary approaches** side by side, so you can compare what Entra sign-in logs say against what Defender for Cloud Apps (MDA) activity says, and reconcile the two.

## Files

| File | Purpose |
|------|---------|
| `Sentinel-Identity-Dashboard.workbook.json` | The workbook. Import via **Workbooks → New → Advanced Editor**. |

## The four approaches

| # | Section | Source | Signal |
|---|---------|--------|--------|
| **A** | Union (Interactive + Non-interactive) · App × User Journey | `SigninLogs` ∪ `AADNonInteractiveUserSignInLogs` | Full journey: Sign-in → MFA → MFA Method → Conditional Access → Recency |
| **B** | Non-interactive sign-ins (direct) | `AADNonInteractiveUserSignInLogs` | Leanest path (no `union`/`mv-expand`); a fallback to confirm data flows |
| **C** | MDA activity (direct) | `CloudAppEvents` | `MDCAActivity` = user performed an in-app action captured by MDCA in range. Covers non-Entra apps |
| **D** | MDA × Entra cross-reference | `CloudAppEvents` ⨝ Entra sign-ins | `AccessType` = **Authenticated** (matching corporate sign-in) vs **Unauthenticated** |

Two **data-availability** tiles (Entra and MDA) show row/user/app counts per source so you can see at a glance which tables are actually ingesting data in the selected workspace.

## Filters

Filters are pills at the top and cascade top-to-bottom. **Select Workspace and Time Range first**; the query-backed pills then populate.

| Filter | Type | Notes |
|--------|------|-------|
| **Workspace** | Azure Resource Graph | Pick the Log Analytics / Sentinel workspace. |
| **Time Range** | Duration | 24h / 7d / 30d / 90d / custom. Scopes every table and the recency logic. |
| **User (type UPN or name)** | Free text | Partial, case-insensitive `contains` match. Free text (not a dropdown) because there are thousands of users. Leave blank for all. |
| **Application** | Multi-select | Populated from Entra `AppDisplayName`. Tick the 5–10 apps of interest, or leave **All**. |
| **Sign-in Status** | All / Success / Failure | Applies to Approaches A/B. |
| **Access Type** | All / Authenticated / Unauthenticated | Applies to Approach D. |

### Cross-taxonomy app matching (A/B vs C/D)

The **Application** list comes from **Entra** sign-in names (e.g. *Microsoft 365 Admin portal*), but MDA (`CloudAppEvents.Application`) often uses different names (e.g. *Microsoft 365*). Approaches **C/D** therefore **fuzzy-match** your selection to MDA app names (case-insensitive substring, either direction) so a specific-app pick still scopes them. Very broad names (e.g. *Office*) may pull in several MDA apps — leave **All** for the full MDA picture.

## Prerequisites

- **Microsoft Entra ID P1/P2** with the **Entra ID** data connector enabled — provides `SigninLogs` and/or `AADNonInteractiveUserSignInLogs` (Approaches A/B).
- **Microsoft Defender for Cloud Apps** connected to Sentinel — provides `CloudAppEvents` (Approaches C/D).
- **Reader** on the target Log Analytics / Sentinel workspace.

If a table shows *no data*, check the matching data-availability tile, widen the **Time Range**, or pick a different **Workspace**.

## Deploy (Advanced Editor import)

1. Open your Log Analytics workspace (or Microsoft Sentinel) in the Azure portal — or Microsoft Defender XDR (unified) if using the Defender portal.
2. Go to **Workbooks → + New**, click **Edit**, then open the **Advanced Editor** (`</>`).
3. Replace the contents with `Sentinel-Identity-Dashboard.workbook.json`, click **Apply**, then **Save** (e.g. *Verified SaaS Usage – Identity Journey*).
4. Select the **Workspace** and **Time Range** pills, then use **User** / **Application** to focus.

## Known limitations

- **Naming taxonomies differ** between Entra and MDA. The fuzzy bridge (above) reconciles most cases; a curated alias map is the follow-up if exact per-app parity is required.
- **Non-Entra IdPs (e.g. PingID)** produce no Entra sign-in, so those apps appear **Unauthenticated** in Approach D even when the user really did authenticate. The workbook's *How the correlation works* section documents how to ingest PingID/other IdP logs and `union` them into the authentication set to correct this.
- **Blank / GUID app names** on non-interactive sign-ins are backfilled from `ResourceDisplayName` then `AppId`, so you get a value instead of a blank.
- Query-backed **parameter dropdowns** require the Log Analytics `queryType`/`resourceType` and bind to the selected **Workspace** — they populate only after Workspace + Time Range are set.
