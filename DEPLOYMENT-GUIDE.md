# Microsoft Sentinel Identity & Access Dashboard – Deployment Guide

## Overview

This guide provides step-by-step instructions to deploy the **Sentinel Identity & Access Dashboard** workbook into your Azure Monitor or Microsoft Sentinel environment.

**Deliverables:**
- `Sentinel-Identity-Dashboard.workbook.json` – Ready-to-import workbook
- `arm-template-identity-dashboard.json` – ARM template for automated deployment
- `KQL-Queries-Reference.kql` – All KQL queries with documentation

---

## Prerequisites

1. **Microsoft Sentinel or Azure Monitor** workspace already configured
2. **SigninLogs** table available (populated via Entra ID Sign-in Logs connector)
3. **Contributor** or **Owner** role on the Log Analytics workspace
4. Optional: **CloudAppEvents**, **IdentityInfo**, **IdentityLogonEvents**, **SecurityAlert** tables for enhanced functionality

---

## Deployment Method 1: Direct Import (5 minutes)

### Via Azure Portal

1. **Navigate to your Log Analytics workspace**
   - Go to [Azure Portal](https://portal.azure.com)
   - Search for **Log Analytics workspaces**
   - Select your workspace

2. **Open Workbooks**
   - In the left sidebar, click **Workbooks**
   - Click **+ New** (top left)
   - Click **Edit** (pencil icon)
   - Go to **Advanced Editor** (top right)

3. **Paste the workbook JSON**
   - Open `Sentinel-Identity-Dashboard.workbook.json`
   - Copy the entire JSON content
   - Paste into the **Advanced Editor**
   - Click **Apply**

4. **Save the workbook**
   - Click **Save** (top left)
   - Enter name: `Sentinel Identity & Access Dashboard`
   - Select location (e.g., "My Reports")
   - Click **Save**

5. **Validate**
   - Click **Done Editing**
   - Verify all sections load without errors
   - Check KQL query results populate

---

## Deployment Method 2: ARM Template (Automated)

### Using Azure CLI

```powershell
# Set variables
$subscriptionId = "your-subscription-id"
$resourceGroupName = "your-resource-group"
$workspaceName = "your-workspace-name"
$templateFile = "arm-template-identity-dashboard.json"
$location = "East US"

# Login to Azure
az login
az account set --subscription $subscriptionId

# Deploy the template
az deployment group create \
  --resource-group $resourceGroupName \
  --template-file $templateFile \
  --parameters \
    workspaceName=$workspaceName \
    workbookDisplayName="Sentinel Identity & Access Dashboard" \
  --location $location

# Verify deployment
az deployment group show \
  --resource-group $resourceGroupName \
  --name $(az deployment group list -g $resourceGroupName --query "[0].name" -o tsv)
```

### Using PowerShell

```powershell
# Set variables
$subscriptionId = "your-subscription-id"
$resourceGroupName = "your-resource-group"
$workspaceName = "your-workspace-name"
$templateFile = "arm-template-identity-dashboard.json"
$location = "East US"

# Login to Azure
Connect-AzAccount -Subscription $subscriptionId

# Deploy the template
$deployment = New-AzResourceGroupDeployment `
  -ResourceGroupName $resourceGroupName `
  -TemplateFile $templateFile `
  -workspaceName $workspaceName `
  -workbookDisplayName "Sentinel Identity & Access Dashboard" `
  -Location $location

# Output deployment status
$deployment | Select-Object ProvisioningState, ResourceGroupName
```

---

## Deployment Method 3: GitHub (Infrastructure as Code)

```bash
# Clone or download repository
git clone https://your-repo-url.git
cd sentinel-workbooks/identity-dashboard

# Deploy using ARM template
az deployment group create \
  --resource-group your-resource-group \
  --template-file arm-template-identity-dashboard.json \
  --parameters workspaceName=your-workspace-name
```

---

## Post-Deployment Configuration

### 1. Verify Data Sources

Ensure the following tables are populated:

```kql
SigninLogs
| summarize count() by TimeGenerated
| order by TimeGenerated desc
| limit 1
```

Expected: At least one row with recent data.

### 2. Configure Parameters

The workbook includes 5 parameters:

| Parameter | Purpose | Type |
|-----------|---------|------|
| **Time Range** | Filter events by time window | Dynamic |
| **User** | Filter by specific user (UPN) | Dynamic dropdown |
| **Application** | Filter by app name | Dynamic dropdown |
| **Conditional Access** | Filter by CA status | Static: All/Applied/Not Applied |
| **Auth Status** | Filter by result | Static: All/Success/Failure |

### 3. Pin Key Tiles to Dashboard

1. Open the workbook
2. Click **Pin to Dashboard** on individual KPI tiles
3. Create a custom dashboard for executive visibility

---

## Workbook Sections Overview

### **Section 1 – Executive Summary** (KPIs)
- Total Users
- Total Applications
- Successful Sign-ins
- Failed Sign-ins
- Apps Without CA Protection

### **Section 2 – Application Access Matrix**
- Apps with user counts, success/failure rates, MFA %
- Filterable by Conditional Access status

### **Section 3 – High-Risk Unprotected Apps**
- Red table highlighting apps accessed without CA
- Shows user, IP, location, auth method, timestamp

### **Section 4 – User Investigation**
- Summary statistics: signins, apps, IPs, failures
- Detailed timeline of recent sign-ins
- Filterable by user

### **Section 5 – Authentication Analysis**
- MFA adoption pie chart
- Authentication methods breakdown (Password, FIDO2, Certificate, etc.)

### **Section 6 – Conditional Access Coverage**
- Protection percentage
- Protected vs unprotected counts
- Threshold indicators (Good ≥90%, Fair ≥70%, Poor <70%)

### **Section 7 – User Activity Timeline**
- Complete sign-in event log (last 1000 records)
- Sortable by time, user, app, result

### **Section 8 – Geographic Map**
- Sign-in volume by location
- User and failure count by country/city

### **Section 9 – Risk Correlation**
- Risk distribution (High/Medium/Low)
- Failed sign-in root cause analysis
- Grouped by ResultType and ResultDescription

### **Section 10 – Application Deep Dive**
- App-level metrics: success rate, protection rate
- Top users for selected app
- Recent failures with details

---

## Customization Examples

### Change Default Time Range

Open the workbook in edit mode and modify the `TimeRange` parameter:

```json
"value": {
  "durationMs": 2592000000
},
"typeSettings": {
  "selectableValues": [
    {"durationMs": 86400000, "displayText": "Last 24 Hours"},
    {"durationMs": 604800000, "displayText": "Last 7 Days"},
    {"durationMs": 2592000000, "displayText": "Last 30 Days"},
    {"durationMs": 5184000000, "displayText": "Last 60 Days"}
  ]
}
```

### Add Custom Query

1. Click **Edit** on the workbook
2. Add a new **Query** item
3. Paste your KQL from `KQL-Queries-Reference.kql`
4. Configure visualization (table, chart, map, etc.)
5. Click **Done**

### Adjust Color Thresholds

For tile visualizations, modify `formatOptions.palette`:
- `"green"` – Success, Protected, Good
- `"orange"` – Warning, Unprotected, Fair
- `"red"` – Failed, High Risk, Poor
- `"blue"` – Informational

Example:
```json
"tileSettings": {
  "leftContent": {
    "columnMatch": "FailedSignins",
    "formatter": 12,
    "formatOptions": {
      "palette": "redGreen"
    }
  }
}
```

---

## Troubleshooting

### Workbook Won't Load
- **Issue:** "Failed to execute query"
- **Solution:** Verify `SigninLogs` table exists: run `SigninLogs | limit 1` in Log Analytics

### Dynamic Parameters Empty
- **Issue:** User/Application dropdowns show no values
- **Solution:** Ensure SigninLogs has data in the selected time range; check time zone settings

### Queries Return No Data
- **Issue:** All sections show empty results
- **Solution:** 
  - Verify Entra ID Sign-in Logs connector is enabled
  - Check data ingestion: `SigninLogs | summarize count()`
  - Confirm time range includes actual events

### Performance Issues
- **Issue:** Workbook loads slowly
- **Solution:** Reduce time range (default 7 days); increase limits on `| limit 1000`

### ARM Deployment Fails
- **Issue:** "Insufficient permissions" or "Invalid workspace"
- **Solution:**
  - Verify Contributor role on resource group
  - Confirm workspace name is correct
  - Check workspace still exists in region

---

## Monitoring & Maintenance

### Recommended Alerts

Create Azure Monitor alerts for:

1. **High Failed Sign-in Rate**
   ```kql
   SigninLogs
   | where TimeGenerated > ago(1h)
   | summarize FailureRate = countif(ResultType != 0) * 100 / count()
   | where FailureRate > 10
   ```

2. **Unprotected Application Access**
   ```kql
   SigninLogs
   | where ConditionalAccessStatus != "success"
   | summarize Count = count()
   | where Count > 100
   ```

3. **Anomalous Sign-in Locations**
   ```kql
   SigninLogs
   | where ResultType == 0
   | extend Hour = floor(TimeGenerated / 1h) * 1h
   | summarize UniqueLocations = dcount(Location) by UserPrincipalName, Hour
   | where UniqueLocations > 3
   ```

### Weekly Review Checklist

- [ ] Review failed sign-in trends
- [ ] Check MFA adoption metrics
- [ ] Identify unprotected applications
- [ ] Investigate anomalous user activity
- [ ] Verify Conditional Access coverage

---

## Performance Notes

- **Data retention:** SigninLogs default is 30 days (configurable to 2 years)
- **Query timeout:** 10 minutes (standard Log Analytics)
- **Typical query time:** <2 seconds for 7-day range
- **Recommended workspace size:** 10GB+ daily ingestion

---

## Support & Feedback

For issues or feature requests:
1. Check workbook JSON for syntax errors
2. Review KQL query logic in advanced editor
3. Verify all required tables are present
4. Consult Microsoft Sentinel documentation

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-07-21 | Initial release; 10 sections, 25+ queries |

---

## License

This workbook is provided as-is for use in Microsoft Sentinel environments.

