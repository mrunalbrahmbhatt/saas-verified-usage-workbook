# Deployment Guide

## Import workbook JSON into Microsoft Sentinel
1. Open Microsoft Sentinel.
2. Go to Workbooks.
3. Select Add workbook.
4. Open the new workbook, then select Edit.
5. Select Advanced editor.
6. Paste content from workbook/SaaS-Verified-Usage-Flexera.workbook.json.
7. Save as: SaaS Verified Usage and Flexera Export Workbook.

## Test each query in Logs
Run these files directly in Logs and verify output:
1. queries/01-unified-saas-usage.kql
2. queries/02-flexera-export.kql
3. queries/03-data-quality-gaps.kql

Validation expectations:
- 01-unified-saas-usage.kql returns user/app-level classification and evidence columns.
- 02-flexera-export.kql returns export-ready columns with one row per user/application.
- 03-data-quality-gaps.kql returns KPI metrics, discovery-only apps, and table health.

## Modify custom table names and schema
Common changes:
1. Ping custom table name
   - Replace PingIdentity_CL with your custom table name.
2. Ping column mapping
   - Update AppName_s, UserPrincipalName_s, Result_s, EventType_s mappings to your schema.
3. Cloud Discovery column differences
   - Update Application and user identity mappings in McasShadowItReporting section.
4. User enrichment table
   - Replace UserInventory_CL column mappings if your enrichment source differs.

## Export Flexera table to CSV
1. Open workbook section: Flexera Export.
2. Apply filters and time range.
3. Use Export to CSV from the grid/table control.
4. Confirm CSV columns exactly match Flexera required schema:
   - Name
   - Email address
   - Username
   - Country
   - Department
   - Subscriptions
   - Activity Status
   - Discovery Type
   - Last active
   - Total days used

## Troubleshoot missing data
1. Check data source connectors are enabled and healthy.
2. Verify selected time range is wide enough.
3. Run table health output from queries/03-data-quality-gaps.kql.
4. Confirm app naming consistency across sources (for example, Workday vs Workday Inc).
5. If Ping logs are absent, expect Verified Sign-In to rely on Entra sign-ins only.
6. If Entra and Ping are both absent, records may remain URL Access Only.
7. If country/department are missing, onboard UserInventory_CL or enrich from Entra/HR source.

## Important accuracy notes
- MDCA Cloud Discovery can show app/URL usage, but should not be treated as proof of successful authentication.
- CloudAppEvents is populated by Defender for Cloud Apps records and is useful for cloud app activity where available.
- Third-party discovered apps may appear differently depending on whether the app is only discovered or connected through an app connector.
- If Ping Identity logs are not onboarded, verified sign-in should use Entra ID where available, otherwise usage remains URL Access Only.
- If HR attributes such as country and department are not in Sentinel, use a watchlist called UserInventory_CL or enrich from Entra ID/HR source.
