# Workbook Validation & Best Practices

## Pre-Deployment Validation Checklist

### Schema Validation
- [x] JSON is valid (no syntax errors)
- [x] All required properties present (`version`, `items`, `schema`)
- [x] Query syntax is valid KQL
- [x] Parameter names match query placeholders

### Data Availability
```kql
// Run these to validate data sources exist
SigninLogs | limit 1
CloudAppEvents | limit 1
IdentityInfo | limit 1
IdentityLogonEvents | limit 1
SecurityAlert | limit 1
BehaviorAnalytics | limit 1
```

### Query Performance
```kql
// Test each critical query with actual time range
SigninLogs
| where TimeGenerated >= ago(7d)
| summarize Count = count()

// Verify no timeouts (should complete <2 seconds)
```

---

## Post-Deployment Validation

### Step 1: Verify Workbook Loads
1. Open workbook in Azure Portal
2. Confirm all 10 sections render
3. Check no red error badges appear

### Step 2: Validate Parameters Work
1. Change **Time Range** → verify data updates
2. Select **User** from dropdown → drill-down loads
3. Select **Application** → app-specific queries execute
4. Toggle **Conditional Access** filter → results change

### Step 3: Smoke Test Each Section

**Section 1 – Executive Summary**
```
Expected: 
  • Total Users > 0
  • Total Applications > 0
  • Successful Sign-ins > 0
  • At least one KPI has a non-zero value
```

**Section 2 – Application Access Matrix**
```
Expected:
  • Table has at least one row
  • Columns: AppDisplayName, CAStatus, Users, Success, Failed, MFAPercent
  • Color formatting applies (Protected=green, Unprotected=red)
```

**Section 3 – Unprotected Apps (Risk View)**
```
Expected:
  • Shows only CA != "success" records
  • Red highlighting on high-risk rows
  • Sortable by TimeGenerated descending
```

**Section 4 – User Investigation**
```
Expected:
  • User filter populates dropdown with real UPNs
  • Summary shows aggregated user metrics
  • Detail grid shows individual sign-in events
```

**Section 5 – Authentication Analysis**
```
Expected:
  • MFA pie chart sums to 100%
  • Auth methods include recognized types (Password, FIDO2, etc.)
  • At least 2 pie chart segments visible
```

**Section 6 – Conditional Access Coverage**
```
Expected:
  • Protection Percent between 0-100
  • Protected count ≥ 0
  • Unprotected count ≥ 0
  • Total = Protected + Unprotected
```

**Section 7 – Timeline**
```
Expected:
  • Shows TimeGenerated in descending order
  • Max 1000 rows displayed
  • All columns populated: Time, User, App, Result, IP, Description
```

**Section 8 – Geographic Map**
```
Expected:
  • Grouped by Location (country/city)
  • Count aggregates properly
  • Failure threshold highlighting applies (>10 = red)
```

**Section 9 – Risk Correlation**
```
Expected:
  • Risk Distribution pie shows Low/Medium/High
  • Failure Root Causes table is populated
  • Top failure reasons visible
```

**Section 10 – Application Deep Dive**
```
Expected:
  • App overview shows Success Rate % and Protection Rate %
  • Top Users bar chart limits to 20 users
  • Recent Failures shows only ResultType != 0 records
```

---

## Query Optimization Guide

### Performance Baseline Targets

| Metric | Target | Method |
|--------|--------|--------|
| KPI queries (single aggregate) | <1 second | Count(), dcount() |
| Matrix queries (multi-dimension) | <2 seconds | summarize, group by |
| Timeline queries (1000 rows) | <3 seconds | order by TimeGenerated |
| Auth expansion (mv-expand) | <5 seconds | Use where before mv-expand |

### Optimization Tips

1. **Always filter on TimeGenerated first**
   ```kql
   // GOOD - Filter early
   SigninLogs
   | where TimeGenerated >= ago({TimeRange})
   | where UserPrincipalName == "{User}"
   
   // SLOW - Filter late
   SigninLogs
   | where UserPrincipalName == "{User}"
   | where TimeGenerated >= ago({TimeRange})
   ```

2. **Use column_ifexists for optional fields**
   ```kql
   // Avoids breaking on missing columns
   | extend Location = column_ifexists('Location', 'Unknown')
   ```

3. **Limit mv-expand to filtered data**
   ```kql
   // GOOD - Expand only relevant rows
   SigninLogs
   | where isnotempty(column_ifexists('AuthenticationDetails', ''))
   | mv-expand AuthenticationDetailsArray = parse_json(AuthenticationDetails)
   
   // SLOW - Expand all rows first
   SigninLogs
   | mv-expand AuthenticationDetailsArray = ...
   ```

4. **Use `limit` or `top` for result sets**
   ```kql
   // Top 20 users only
   | summarize Count = count() by UserPrincipalName
   | top 20 by Count desc
   ```

---

## Common Customizations

### Add Email Domain Filter
```json
{
  "id": "EmailDomain",
  "label": "Email Domain",
  "type": 2,
  "query": "SigninLogs | where TimeGenerated >= ago({TimeRange}) | extend Domain = tostring(split(UserPrincipalName, '@')[1]) | distinct Domain | order by Domain asc",
  "value": null
}
```

### Change KPI Tile Colors
```json
"tileSettings": {
  "leftContent": {
    "formatter": 12,
    "formatOptions": {
      "palette": "redGreen",  // or "greenRed", "blue", "orange"
      "aggregation": "Sum"
    }
  }
}
```

### Add Drill-Through Link
```json
"gridSettings": {
  "sortBy": [{"itemKey": "TimeGenerated", "sortOrder": 2}]
},
"sqlItem": null,
"jsonData": null,
"type": "grid",
"gridFormatters": []
```

### Adjust Row Limits
```json
"gridSettings": {
  "rowLimit": 500  // Change from 100, 500, 1000
}
```

---

## Troubleshooting Query Issues

### Query Returns Empty
1. **Verify data exists:**
   ```kql
   SigninLogs | summarize Count = count() by bin(TimeGenerated, 1d)
   ```

2. **Check filter conditions:**
   ```kql
   SigninLogs
   | where TimeGenerated >= ago(7d)
   | where ResultType == 0
   | summarize Count = count()
   // Expected: > 0
   ```

3. **Test parameters:**
   ```kql
   SigninLogs
   | where UserPrincipalName == "user@domain.com"
   | summarize Count = count()
   ```

### Query Times Out
1. **Reduce time range:** Change 30d to 7d
2. **Remove mv-expand:** Don't parse JSON if not needed
3. **Add early filter:** Reduce rows before aggregation
4. **Use summarize:** Reduces cardinality early

### Format Errors
- Check JSON quotes: `'string'` vs `"string"`
- Verify operator syntax: `==` vs `=`
- Confirm function names: `dcount()` vs `count()`

---

## Security Best Practices

1. **Restrict workbook access**
   - Share only with SOC analysts (not external users)
   - Use Azure RBAC to control read/edit permissions

2. **Data retention**
   - SigninLogs: Default 30 days (can extend to 2 years)
   - Archive older data to ADLS for compliance

3. **Sensitive field masking**
   - Consider hashing IP addresses in reports
   - Obfuscate email domains if needed

4. **Audit trail**
   - Enable Azure Monitor diagnostic logs
   - Track who accesses the workbook (Azure AD Sign-in Logs)

---

## Compliance Considerations

### GDPR
- Ensure IP addresses are handled per policy
- Provide data subject access requests
- Implement data retention policies

### SOC 2
- Maintain audit logs of workbook access
- Document query purposes and approvals
- Regularly review access controls

### HIPAA (if healthcare)
- Encrypt data at rest and in transit
- Implement access controls
- Maintain audit logs for 6+ years

---

## Performance Monitoring

### Monitor Workbook Usage
```kql
AzureActivity
| where OperationNameValue contains "workbook"
| summarize Count = count() by bin(TimeGenerated, 1d), Caller
| render barchart
```

### Monitor Query Performance
```kql
// In Log Analytics workspace diagnostic logs
LAQueryLogs
| where QueryTimeRangeStart >= ago(7d)
| summarize 
    AvgTime = avg(QueryCpuTime_s),
    MaxTime = max(QueryCpuTime_s),
    Count = count()
    by Query
| order by AvgTime desc
```

---

## Version Control

Store workbook JSON in Git:

```bash
# Initialize repo
git init sentinel-workbooks
cd sentinel-workbooks

# Add files
git add Sentinel-Identity-Dashboard.workbook.json
git add arm-template-identity-dashboard.json
git add KQL-Queries-Reference.kql
git add DEPLOYMENT-GUIDE.md
git add VALIDATION-GUIDE.md

# Commit with message
git commit -m "Initial: Sentinel Identity Dashboard workbook v1.0

- 10 sections covering user access, auth, and risk
- 25+ KQL queries with full documentation
- ARM template for automated deployment
- Parameters: Time Range, User, Application, CA Status, Auth Status
- Conditional formatting and drill-down navigation"

# Push to remote
git push origin main
```

---

## Testing Scenarios

### Scenario 1: SOC Analyst Investigates User
1. Open workbook
2. Select User parameter (e.g., john.doe@contoso.com)
3. View all user activity: apps, locations, IPs, failures
4. Identify suspicious patterns (multiple countries, unusual apps)

### Scenario 2: Security Team Reviews Unprotected Apps
1. Go to Section 3 (High-Risk Apps)
2. Identify apps without CA protection
3. Count failures per app
4. Recommend CA policies for high-risk apps

### Scenario 3: Executive Briefing
1. Screenshots of Section 1 (KPIs)
2. Share Section 6 (CA Coverage percentage)
3. Highlight risk metrics from Section 9
4. Present trends over time

---

## Maintenance Schedule

| Frequency | Task |
|-----------|------|
| Daily | Monitor failed sign-in alerts |
| Weekly | Review unprotected app access |
| Monthly | Analyze MFA adoption trends |
| Quarterly | Update CA policies based on workbook insights |
| Annually | Review workbook schema and upgrade if needed |

---

## Success Metrics

✅ **Deployment Success:**
- All 10 sections load without errors
- Parameters filter data correctly
- Queries complete <5 seconds
- At least 100 sign-in events visible

✅ **Operational Success:**
- SOC team uses workbook daily
- Reduced investigation time (5→2 minutes per incident)
- MFA adoption increases 10%+ per quarter
- Unprotected app access decreases

✅ **Security Impact:**
- Faster incident response
- Improved threat visibility
- Better understanding of access patterns
- Data-driven security policies

