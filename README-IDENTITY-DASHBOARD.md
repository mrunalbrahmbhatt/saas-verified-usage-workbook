# Microsoft Sentinel Identity & Access Dashboard

**Complete workbook solution for monitoring user access, authentication methods, Conditional Access policies, and risk indicators in Microsoft Sentinel.**

---

## 📦 Package Contents

```
workbook/
├── Sentinel-Identity-Dashboard.workbook.json      # Ready-to-import workbook
├── arm-template-identity-dashboard.json            # ARM template for automation
├── KQL-Queries-Reference.kql                       # All queries with documentation
├── DEPLOYMENT-GUIDE.md                             # Step-by-step deployment
├── VALIDATION-GUIDE.md                             # Testing & troubleshooting
└── README.md                                       # This file
```

---

## 🎯 Features

### 10 Comprehensive Sections

1. **Executive Summary** – KPI tiles (users, apps, success/failure, CA coverage)
2. **Application Access Matrix** – Apps with usage metrics and CA status
3. **High-Risk Unprotected Apps** – Red-highlighted risk table
4. **User Investigation** – Drill-down into individual user activity
5. **Authentication Analysis** – MFA adoption and auth methods
6. **Conditional Access Coverage** – CA protection metrics and trends
7. **User Activity Timeline** – Complete sign-in event log
8. **Geographic Access Map** – Sign-ins by location
9. **Risk Correlation** – Risk distribution and failure root causes
10. **Application Deep Dive** – App-level detail with top users and failures

### Smart Parameters

| Parameter | Type | Purpose |
|-----------|------|---------|
| **Time Range** | Dynamic | Last 24h, 7d, 30d, or custom |
| **User** | Dropdown | Filter by UPN (optional) |
| **Application** | Dropdown | Filter by app name (optional) |
| **Conditional Access** | Static | All / Applied / Not Applied |
| **Auth Status** | Static | All / Success / Failure |

### Visual Elements

- ✅ KPI Tiles (green/orange/red thresholds)
- ✅ Data Grids (sortable, filterable)
- ✅ Pie & Donut Charts (MFA, Risk, CA coverage)
- ✅ Bar Charts (auth methods, top users)
- ✅ Timeline Tables (sign-in activity log)
- ✅ Conditional Formatting (color-coded risk levels)
- ✅ Drill-down Navigation (user → app → specific event)

### Data Sources

**Required:**
- `SigninLogs` – Entra ID authentication events

**Optional (for enhanced features):**
- `CloudAppEvents` – SaaS application access
- `IdentityInfo` – User identity enrichment
- `IdentityLogonEvents` – On-premises logon events
- `SecurityAlert` – Identity Protection alerts
- `BehaviorAnalytics` – UEBA insights

---

## 🚀 Quick Start

### Option 1: Direct Import (5 minutes)

1. Open your Log Analytics workspace in Azure Portal
2. Go to **Workbooks** → **New** → **Advanced Editor**
3. Paste contents of `Sentinel-Identity-Dashboard.workbook.json`
4. Click **Apply** → **Save**
5. Done! Workbook is ready to use.

### Option 2: ARM Template (Automated)

```bash
az deployment group create \
  --resource-group my-rg \
  --template-file arm-template-identity-dashboard.json \
  --parameters workspaceName=my-workspace
```

### Option 3: Import from File

In Azure Portal:
1. Workbooks → New → Import from file
2. Select `Sentinel-Identity-Dashboard.workbook.json`
3. Confirm → Save

---

## 📖 Documentation

### For Deployment
→ See **[DEPLOYMENT-GUIDE.md](DEPLOYMENT-GUIDE.md)**
- Step-by-step import instructions (3 methods)
- ARM template deployment
- Parameter configuration
- Post-deployment checklist

### For Validation & Troubleshooting
→ See **[VALIDATION-GUIDE.md](VALIDATION-GUIDE.md)**
- Pre-deployment schema validation
- Post-deployment smoke tests
- Query optimization tips
- Troubleshooting common issues
- Security & compliance considerations

### For Query Details
→ See **[KQL-Queries-Reference.kql](KQL-Queries-Reference.kql)**
- All 25+ KQL queries documented
- Performance notes
- Optional advanced queries
- Query optimization examples

---

## 🔍 Use Cases

### SOC Analyst – User Threat Investigation
1. Open workbook
2. Select user from **User** parameter
3. Review all activities in Section 4
4. Check for anomalous locations, IPs, or devices
5. Identify failed logons and failure reasons

### Security Manager – Compliance Review
1. Check Section 6 (Conditional Access Coverage)
2. Monitor MFA adoption (Section 5)
3. Review unprotected app access (Section 3)
4. Generate metrics for executive briefing

### Identity Architect – Policy Recommendations
1. Identify unprotected apps in Section 3
2. Analyze failure root causes in Section 9
3. Review geographic access patterns in Section 8
4. Recommend CA policies for high-risk apps

### Executive – Security Posture Overview
1. Review KPI tiles in Section 1
2. Check CA coverage percentage in Section 6
3. View risk distribution in Section 9
4. Use for monthly security briefing

---

## 📊 Example Queries

### Find users with MFA disabled
```kql
SigninLogs
| where TimeGenerated >= ago(7d)
| extend MFARequired = column_ifexists('AuthenticationRequirement', 'Unknown')
| where MFARequired != 'MFA'
| summarize Count = count() by UserPrincipalName
| order by Count desc
```

### Detect credential spray attacks
```kql
SigninLogs
| where TimeGenerated >= ago(1d)
| where ResultType != 0
| summarize UniqueUsers = dcount(UserPrincipalName), FailureCount = count() by AppDisplayName
| where UniqueUsers > 5 and FailureCount > 20
```

### Identify apps without Conditional Access
```kql
SigninLogs
| where ConditionalAccessStatus != "success"
| summarize Count = count() by AppDisplayName
| order by Count desc
```

---

## ⚡ Performance

| Metric | Target |
|--------|--------|
| KPI queries | <1 second |
| Matrix queries | <2 seconds |
| Timeline queries | <3 seconds |
| Full workbook load | <10 seconds |

**Note:** Performance depends on:
- SigninLogs table size (recommend <500GB)
- Query time range (7 days = optimal)
- Workspace compute tier (Premium = faster)

---

## 🔒 Security & Compliance

### Access Control
- Use Azure RBAC to restrict workbook access
- Share only with authorized SOC analysts
- Consider read-only role for executives

### Data Retention
- SigninLogs: Default 30 days (configurable to 2 years)
- Archive older data for compliance
- Implement purge policies for GDPR/HIPAA

### Audit Trail
- Enable Azure Monitor diagnostic logs
- Track workbook access via Azure AD Sign-in Logs
- Document query purposes and approvals

---

## 🛠️ Customization

### Add New KPI Tile
1. Edit workbook
2. Add new KQL query section
3. Configure visualization as "tiles"
4. Set color thresholds in `formatOptions.palette`

### Extend with Additional Tables
Replace `column_ifexists()` placeholders with actual columns:
```kql
// Original (safe)
| extend Location = column_ifexists('Location', 'N/A')

// Optimized (if table has Location)
| extend Location = Location
```

### Add Drill-Through Navigation
```json
"actions": {
  "url": "https://portal.azure.com/#@{TenantId}/resource/subscriptions/{SubscriptionId}/resourcegroups/{ResourceGroup}/providers/microsoft.insights/workbooks/{WorkbookId}/edit"
}
```

---

## 📋 System Requirements

- **Azure subscription** with Log Analytics workspace
- **Microsoft Sentinel** enabled (or standalone Azure Monitor)
- **SigninLogs** data ingestion (Entra ID connector)
- **Contributor** role on workspace (for deployment)
- **Reader** role (for viewing workbook)

---

## 🐛 Troubleshooting

### Workbook won't load
- Check `SigninLogs` table exists: `SigninLogs | limit 1`
- Verify JSON syntax in Advanced Editor
- Clear browser cache and reload

### Queries return no data
- Confirm SigninLogs has recent events (last 24h)
- Check time zone settings align
- Verify Entra ID Sign-in Logs connector is running

### Slow query performance
- Reduce time range (try 7 days instead of 30)
- Add WHERE filter to reduce row count
- Use `limit 1000` on large result sets

→ See **[VALIDATION-GUIDE.md](VALIDATION-GUIDE.md)** for detailed troubleshooting.

---

## 📝 Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-07-21 | Initial release: 10 sections, 25+ queries, full ARM template |

---

## 📧 Support

For issues or feature requests:
1. Check [VALIDATION-GUIDE.md](VALIDATION-GUIDE.md) for troubleshooting
2. Review [KQL-Queries-Reference.kql](KQL-Queries-Reference.kql) for query logic
3. Verify all prerequisites are met
4. Consult Microsoft Sentinel documentation

---

## 📄 License

This workbook is provided as-is for use in Microsoft Sentinel environments.

---

## 🎓 Learning Resources

- [Microsoft Sentinel Documentation](https://docs.microsoft.com/en-us/azure/sentinel/)
- [KQL Query Language Reference](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/)
- [Azure Monitor Workbooks](https://docs.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-overview)
- [Entra ID Sign-in Logs](https://docs.microsoft.com/en-us/azure/active-directory/monitoring-health/howto-analyze-sign-in-logs)

---

## 📊 Dashboard Snapshot

```
┌─────────────────────────────────────────────────────────────────────┐
│ Microsoft Sentinel Identity & Access Dashboard                      │
├─────────────────────────────────────────────────────────────────────┤
│  Time Range: [Last 7 Days ▼]  User: [all ▼]  App: [all ▼]         │
├─────────────────────────────────────────────────────────────────────┤
│  [👥 Total Users: 450]  [📱 Total Apps: 87]  [✅ Successful: 45K]   │
│  [❌ Failed: 2.3K]      [🚫 Unprotected Apps: 12]                   │
├─────────────────────────────────────────────────────────────────────┤
│ Application Access Matrix (sortable)                                │
│ │ Application │ Users │ Success │ Failed │ CA Status │ MFA % │      │
│ │─────────────┼───────┼─────────┼────────┼───────────┼───────│      │
│ │ Microsoft365│  245  │  24.5K  │   650  │ Protected │ 78%   │      │
│ │ Salesforce  │  165  │  16.8K  │   420  │ Protected │ 92%   │      │
│ │ GitHub      │   45  │   3.2K  │   240  │ Unprot.   │ 45%   │ ⚠️ │
│ └─────────────┴───────┴─────────┴────────┴───────────┴───────┘      │
├─────────────────────────────────────────────────────────────────────┤
│ High-Risk Unprotected Apps                                         │
│ [Red Table: 23 recent unprotected access attempts]                 │
├─────────────────────────────────────────────────────────────────────┤
│ [User Investigation] [Auth Analysis] [Risk Correlation] [More...]   │
└─────────────────────────────────────────────────────────────────────┘
```

---

**Ready to deploy? Start with [DEPLOYMENT-GUIDE.md](DEPLOYMENT-GUIDE.md)**
