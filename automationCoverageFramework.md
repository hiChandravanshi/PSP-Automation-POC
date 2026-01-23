# QA Automation Coverage Framework for Azure DevOps

> A comprehensive framework to track, measure, and report QA automation coverage feature-wise without manual calculations.

## 🎯 Overview

This framework enables automatic tracking of QA automation coverage across your Azure DevOps project with zero manual calculation. It provides:

- **Automated Coverage Calculation**: Scripts run on schedule to update coverage metrics
- **Feature-wise Tracking**: See coverage percentage for each feature via queries
- **Complete Traceability**: Epic → Feature → Work Item → Test Case → Automation Spec
- **RTM Generation**: Auto-generated Requirements Traceability Matrix
- **Real-time Dashboards**: Visual representation of coverage metrics
- **No Manual Work**: Everything updates automatically via scheduled pipelines

### Key Benefits

✅ Find automation coverage feature-wise using Azure DevOps queries  
✅ Automatic coverage percentage calculation and updates  
✅ Proper linking between Epics, Features, Work Items, and Test Cases  
✅ Integration with automation test specs via metadata  
✅ One-click RTM generation  
✅ Real-time dashboards and reports  

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Azure DevOps                             │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐ │
│  │   Epic   │───▶│ Feature  │───▶│Work Item │───▶│Test Case │ │
│  │          │    │          │    │  (PBI)   │    │          │ │
│  │ Coverage │    │ Coverage │    │ Coverage │    │Automation│ │
│  │   %      │    │   %      │    │   %      │    │ Status   │ │
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              ▲
                              │
                    ┌─────────┴──────────┐
                    │  Scheduled Pipeline │
                    │  (trigger on main)  │
                    └─────────┬──────────┘
                              │
                    ┌─────────▼──────────┐
                    │  Node.js Scripts   │
                    ├────────────────────┤
                    │ • Coverage Updater │
                    │ • RTM Generator    │
                    │ • Spec Sync        │
                    └─────────┬──────────┘
                              │
                    ┌─────────▼──────────┐
                    │ Test Automation         │
                    │ Specs (Playwright/wdio) │
                    │ + test-mapping.json│
                    └────────────────────┘
```

---


## ⚙️ Configuration

### 1. Azure DevOps Custom Fields Setup

#### Epic Custom Fields
```
Field Name: Custom.TotalFeatures
Type: Integer
Default: 0

Field Name: Custom.FeaturesWithAutomation
Type: Integer
Default: 0

Field Name: Custom.EpicAutomationCoveragePercent
Type: Decimal
Default: 0.00
```

#### Feature Custom Fields
```
Field Name: Custom.TotalTestCases
Type: Integer
Default: 0

Field Name: Custom.AutomatedTestCases
Type: Integer
Default: 0

Field Name: Custom.ManualTestCases
Type: Integer
Default: 0

Field Name: Custom.AutomationCoveragePercent
Type: Decimal
Default: 0.00

Field Name: Custom.AutomationStatus
Type: Picklist
Values: Not Started, In Progress, Completed

Field Name: Custom.Priority
Type: Picklist
Values: P0, P1, P2, P3
```

#### Work Item (PBI/User Story) Custom Fields
```
Field Name: Custom.TestCasesCount
Type: Integer
Default: 0

Field Name: Custom.AutomatedCount
Type: Integer
Default: 0

Field Name: Custom.AutomationCoveragePercent
Type: Decimal
Default: 0.00

Field Name: Custom.TestSuiteID
Type: String
```

#### Test Case Custom Fields
```
Field Name: Custom.AutomationStatus
Type: Picklist
Values: Not Automated, Automated, Automation In Progress
Default: Not Automated

Field Name: Custom.TestType
Type: Picklist
Values: Functional, Regression, Smoke, Integration, E2E

Field Name: Custom.AutomationFramework
Type: Picklist
Values: playwright, wdio

Field Name: Custom.TestSpecPath
Type: String

Field Name: Custom.TestSpecID
Type: String

Field Name: Custom.AutomationPriority
Type: Picklist
Values: High, Medium, Low
```

### 2. Work Item Linking Strategy

```
Epic (Parent)
  └── Feature (Child via System.LinkTypes.Hierarchy-Forward)
        └── Work Item/PBI (Child via System.LinkTypes.Hierarchy-Forward)
              └── Test Case (Linked via Microsoft.VSTS.Common.TestedBy-Forward)
                    └── Automation Spec (Via Custom.TestSpecID field)
```

**Linking Steps:**
1. Create Epic
2. Create Features and link to Epic (Parent-Child)
3. Create Work Items/PBIs and link to Features (Parent-Child)
4. Create Test Cases and link to Work Items (Tested By relationship)
5. Update Test Case with `test case ID` pointing to automation spec with same `test case ID`

---

## 📝 Script Logic (Pseudo Code)

### 1. Update Coverage Metrics Script

**File**: `src/scripts/update-coverage-metrics.ts`

```
FUNCTION UpdateCoverageMetrics():
    
    // Initialize Azure DevOps connection
    adoClient = CreateAzureDevOpsClient(orgUrl, project, pat)
    
    // STEP 1: Update Work Item Coverage
    FUNCTION CalculateWorkItemCoverage():
        workItems = GetAllWorkItems(type = "PBI" OR "User Story")
        
        FOR EACH workItem IN workItems:
            testCases = GetLinkedTestCases(workItem)
            totalTests = testCases.length
            automatedTests = COUNT(testCases WHERE automationStatus = "Automated")
            
            coverage = (automatedTests / totalTests) * 100
            
            UPDATE workItem SET:
                Custom.TestCasesCount = totalTests
                Custom.AutomatedCount = automatedTests
                Custom.AutomationCoveragePercent = coverage
            
            LOG "Updated Work Item {workItem.id}: {automatedTests}/{totalTests} = {coverage}%"
    
    // STEP 2: Update Feature Coverage
    FUNCTION CalculateFeatureCoverage():
        features = GetAllWorkItems(type = "Feature")
        
        FOR EACH feature IN features:
            childWorkItems = GetChildWorkItems(feature)
            allTestCases = []
            
            FOR EACH child IN childWorkItems:
                testCases = GetLinkedTestCases(child)
                allTestCases.APPEND(testCases)
            
            totalTests = allTestCases.length
            automatedTests = COUNT(allTestCases WHERE automationStatus = "Automated")
            manualTests = totalTests - automatedTests
            
            coverage = (automatedTests / totalTests) * 100
            
            UPDATE feature SET:
                Custom.TotalTestCases = totalTests
                Custom.AutomatedTestCases = automatedTests
                Custom.ManualTestCases = manualTests
                Custom.AutomationCoveragePercent = coverage
            
            LOG "Updated Feature {feature.id}: {coverage}%"
    
    // STEP 3: Update Epic Coverage
    FUNCTION CalculateEpicCoverage():
        epics = GetAllWorkItems(type = "Epic")
        
        FOR EACH epic IN epics:
            childFeatures = GetChildWorkItems(epic, type = "Feature")
            totalFeatures = childFeatures.length
            featuresWithAutomation = COUNT(childFeatures WHERE coverage > 0)
            
            avgCoverage = AVERAGE(childFeatures.coverage)
            
            UPDATE epic SET:
                Custom.TotalFeatures = totalFeatures
                Custom.FeaturesWithAutomation = featuresWithAutomation
                Custom.EpicAutomationCoveragePercent = avgCoverage
            
            LOG "Updated Epic {epic.id}: {featuresWithAutomation}/{totalFeatures} features"
    
    // Execute in order
    TRY:
        CalculateWorkItemCoverage()
        CalculateFeatureCoverage()
        CalculateEpicCoverage()
        LOG "Coverage calculation completed successfully"
    CATCH error:
        LOG "Error calculating coverage: {error}"
        THROW error

END FUNCTION
```

### 2. Generate RTM Script

**File**: `src/scripts/generate-rtm.ts`

```
FUNCTION GenerateRTM(epicId = null):
    
    adoClient = CreateAzureDevOpsClient(orgUrl, project, pat)
    rtmData = []
    
    // Get Epics to process
    IF epicId IS NOT NULL:
        epics = [GetWorkItem(epicId)]
    ELSE:
        epics = GetAllWorkItems(type = "Epic", state != "Removed")
    
    // Build RTM data structure
    FOR EACH epic IN epics:
        features = GetChildWorkItems(epic, type = "Feature")
        
        FOR EACH feature IN features:
            workItems = GetChildWorkItems(feature, type = "PBI" OR "User Story")
            
            FOR EACH workItem IN workItems:
                testCases = GetLinkedTestCases(workItem)
                
                IF testCases.length > 0:
                    FOR EACH testCase IN testCases:
                        rtmData.APPEND({
                            epicId: epic.id,
                            epicTitle: epic.title,
                            featureId: feature.id,
                            featureTitle: feature.title,
                            featureCoverage: feature.customFields.AutomationCoveragePercent,
                            workItemId: workItem.id,
                            workItemTitle: workItem.title,
                            workItemState: workItem.state,
                            testCaseId: testCase.id,
                            testCaseTitle: testCase.title,
                            testCaseState: testCase.state,
                            automationStatus: testCase.customFields.AutomationStatus,
                            testType: testCase.customFields.TestType,
                            testSpecPath: testCase.customFields.TestSpecPath,
                            testSpecID: testCase.customFields.TestSpecID,
                            priority: testCase.customFields.AutomationPriority
                        })
                ELSE:
                    // Include work items without test cases
                    rtmData.APPEND({
                        ...workItemInfo,
                        testCaseId: "N/A",
                        testCaseTitle: "No Test Cases Linked"
                    })
    
    // Calculate summary metrics
    summary = {
        totalEpics: COUNT(UNIQUE rtmData.epicId),
        totalFeatures: COUNT(UNIQUE rtmData.featureId),
        totalWorkItems: COUNT(UNIQUE rtmData.workItemId),
        totalTestCases: COUNT(rtmData WHERE testCaseId != "N/A"),
        automatedTests: COUNT(rtmData WHERE automationStatus = "Automated"),
        overallCoverage: (automatedTests / totalTestCases) * 100
    }
    
    // Generate HTML report
    html = RenderHTMLTemplate(rtmData, summary)
    SaveFile(html, outputPath)
    
    // Generate CSV export
    csv = ConvertToCSV(rtmData)
    SaveFile(csv, outputPath.replace('.html', '.csv'))
    
    LOG "RTM generated: {outputPath}"
    RETURN { htmlPath: outputPath, csvPath: csvPath, summary: summary }

END FUNCTION
```

### 3. Sync Test Specs Script

**File**: `src/scripts/sync-test-specs.ts`

```
FUNCTION SyncTestSpecs(mappingFile):
    
    adoClient = CreateAzureDevOpsClient(orgUrl, project, pat)
    
    // Load test mapping file
    testMapping = ReadJSONFile(mappingFile)
    stats = { updated: 0, failed: 0 }
    
    FOR EACH suite IN testMapping.testSuites:
        LOG "Processing suite: {suite.suiteName}"
        
        FOR EACH spec IN suite.specs:
            FOR EACH testCase IN spec.testCases:
                
                TRY:
                    // Update test case in Azure DevOps
                    UPDATE TestCase(testCase.testCaseId) SET:
                        Custom.TestSpecPath = spec.specPath
                        Custom.TestSpecID = testCase.testCaseId
                        Custom.AutomationStatus = testCase.automationStatus
                        Custom.LastAutomationRunDate = NOW()
                    
                    LOG "Updated Test Case {testCase.testCaseId}"
                    stats.updated++
                    
                CATCH error:
                    LOG "Failed to update Test Case {testCase.testCaseId}: {error}"
                    stats.failed++
    
    LOG "Sync complete - Updated: {stats.updated}, Failed: {stats.failed}"
    RETURN stats

END FUNCTION
```


## 🎯 Usage

### Running Scripts Manually

#### Update Coverage Metrics
```bash
npm run update-coverage
```

This will:
1. Calculate coverage for all Work Items
2. Roll up coverage to Features
3. Roll up coverage to Epics
4. Update all custom fields in Azure DevOps

#### Generate RTM
```bash
# Generate RTM for all epics
npm run generate-rtm

# Generate RTM for specific epic
npm run generate-rtm -- --epic-id 12345
```

Output files:
- `./reports/rtm.html` - HTML report
- `./reports/rtm.csv` - CSV export

#### Sync Test Specs
```bash
npm run sync-specs
```

This syncs automation spec metadata from `test-mapping.json` to Azure DevOps test cases.


## 📊 Queries & Reports

### Saved Queries to Create

Navigate to **Boards → Queries** and create these shared queries:

#### 1. Feature-wise Coverage Overview
```sql
SELECT
    [System.Id],
    [System.Title],
    [System.State],
    [Custom.TotalTestCases],
    [Custom.AutomatedTestCases],
    [Custom.ManualTestCases],
    [Custom.AutomationCoveragePercent],
    [Custom.AutomationStatus]
FROM workitems
WHERE
    [System.WorkItemType] = 'Feature'
    AND [System.State] <> 'Removed'
ORDER BY [Custom.AutomationCoveragePercent] ASC
```

#### 2. Features with Low Coverage
```sql
SELECT
    [System.Id],
    [System.Title],
    [Custom.AutomationCoveragePercent],
    [Custom.TotalTestCases]
FROM workitems
WHERE
    [System.WorkItemType] = 'Feature'
    AND [Custom.AutomationCoveragePercent] < 50
    AND [Custom.TotalTestCases] > 0
    AND [System.State] <> 'Removed'
ORDER BY [Custom.AutomationCoveragePercent] ASC
```

#### 3. Epic Coverage Summary
```sql
SELECT
    [System.Id],
    [System.Title],
    [Custom.TotalFeatures],
    [Custom.FeaturesWithAutomation],
    [Custom.EpicAutomationCoveragePercent]
FROM workitems
WHERE
    [System.WorkItemType] = 'Epic'
    AND [System.State] <> 'Removed'
ORDER BY [Custom.EpicAutomationCoveragePercent] ASC
```

#### 4. Test Cases Pending Automation
```sql
SELECT
    [System.Id],
    [System.Title],
    [Custom.AutomationStatus],
    [Custom.TestType],
    [Custom.AutomationPriority]
FROM workitems
WHERE
    [System.WorkItemType] = 'Test Case'
    AND [Custom.AutomationStatus] <> 'Automated'
    AND [System.State] = 'Active'
ORDER BY [Custom.AutomationPriority] DESC, [System.CreatedDate] ASC
```

### How to Use Queries

1. **Find Coverage for Specific Feature**:
   - Run "Feature-wise Coverage Overview" query
   - Filter by Feature ID or Title
   - View coverage percentage directly

2. **Identify Automation Gaps**:
   - Run "Features with Low Coverage" query
   - Prioritize features with high test count but low coverage

3. **Track Epic Progress**:
   - Run "Epic Coverage Summary" query
   - See overall epic automation health

4. **Plan Automation Work**:
   - Run "Test Cases Pending Automation" query
   - Filter by Priority = "High"
   - Assign to automation engineers

---

## 📈 Dashboard Setup

### Create Azure DevOps Dashboard

1. Navigate to **Overview → Dashboards**
2. Click **+ New Dashboard**
3. Name: "QA Automation Coverage"
4. Add these widgets:

#### Widget 1: Coverage KPI
- Type: **Markdown**
- Content:
```markdown
# QA Automation Coverage

**Overall Coverage**: 68.5%  
**Target**: 80%  
**Gap**: -11.5%  

📊 **Progress This Month**: +5.2%
```

#### Widget 2: Feature Coverage Chart
- Type: **Chart for Work Items**
- Query: Feature-wise Coverage Overview
- Chart Type: **Bar Chart**
- Group By: Custom.AutomationCoveragePercent (ranges)

#### Widget 3: Feature Coverage Table
- Type: **Query Results**
- Query: Feature-wise Coverage Overview
- Columns: ID, Title, Coverage %, Automated, Total

#### Widget 4: Epic Summary
- Type: **Query Results**
- Query: Epic Coverage Summary

#### Widget 5: Automation Backlog
- Type: **Query Results**
- Query: Test Cases Pending Automation

#### Widget 6: Coverage Trend
- Type: **Chart for Work Items**
- Query: Feature-wise Coverage Overview
- Chart Type: **Pie Chart**
- Segments: Coverage ranges (0-25%, 26-50%, 51-75%, 76-100%)

---

## 📄 RTM Generation

### Manual Generation

```bash
# Generate RTM for all epics
npm run generate-rtm

# Generate RTM for specific epic
npm run generate-rtm -- --epic-id 12345
```

### Output Files

- **HTML Report**: `./reports/rtm.html`
  - Interactive table with all traceability
  - Summary metrics
  - Color-coded automation status
  - Filterable/sortable

- **CSV Export**: `./reports/rtm.csv`
  - Raw data for analysis
  - Import to Excel/Power BI

### RTM Report Contents

The generated RTM includes:

| Epic | Feature | Work Item | Test Case | Status | Coverage | Spec ID |
|------|---------|-----------|-----------|--------|----------|---------|
| E-001 | F-123 | PBI-456 | TC-789 | Automated | 85% | login.spec.ts |
| E-001 | F-123 | PBI-457 | TC-790 | Not Automated | 85% | N/A |

Plus summary metrics:
- Total Epics
- Total Features
- Total Work Items
- Total Test Cases
- Automated Tests
- Overall Coverage %

---

## 🔗 Automation Integration

### Test Spec Metadata

Add metadata to your test specs to link them with Azure DevOps for use adding this in each it makes more sense:

#### Playwright Example

```typescript
import { test, expect } from '@playwright/test';

/**
 * @suiteId 4555
 * @suiteName "Authentication Tests"
 * @testCaseId 54321
 * @workItemId 67890
 * @featureId 12345
 * @testType Functional
 * @priority High
 */
test('User can login with valid credentials', async ({ page }) => {
  await page.goto('/login');
  await page.fill('#username', 'testuser');
  await page.fill('#password', 'password123');
  await page.click('#login-button');
  await expect(page).toHaveURL('/dashboard');
});
```

### Test Mapping File

**File**: `test-mapping.json`

```json
{
  "testSuites": [
    {
      "suiteId": "TS-AUTH",
      "suiteName": "Authentication Tests",
      "featureId": "12345",
      "specs": [
        {
          "specPath": "tests/auth/login.spec.ts",
          "testCases": [
            {
              "testCaseId": "54321",
              "workItemId": "67890",
              "testName": "Valid Login",
              "automationStatus": "Automated"
            },
            {
              "testCaseId": "54322",
              "workItemId": "67890",
              "testName": "Invalid Login",
              "automationStatus": "Automated"
            }
          ]
        },
        {
          "specPath": "tests/auth/logout.spec.ts",
          "testCases": [
            {
              "testCaseId": "54323",
              "workItemId": "67891",
              "testName": "User Logout",
              "automationStatus": "Automated"
            }
          ]
        }
      ]
    },
    {
      "suiteId": "TS-CART",
      "suiteName": "Shopping Cart Tests",
      "featureId": "12346",
      "specs": [
        {
          "specPath": "tests/cart/add-to-cart.spec.ts",
          "testCases": [
            {
              "testCaseId": "54324",
              "workItemId": "67892",
              "testName": "Add Product to Cart",
              "automationStatus": "Automated"
            }
          ]
        }
      ]
    }
  ]
}
```

### Syncing Process

1. **Update test-mapping.json** when:
   - New test spec is created
   - Test case is automated
   - Test case ID changes

2. **Run sync script**:
```bash
npm run sync-specs
```

3. **Verify in Azure DevOps**:
   - Open test case
   - Check `Custom.TestSpecPath` field
   - Verify `Custom.AutomationStatus` updated

---

### required scripts for checks

- scripts to check metaData is added on all test cases
- script to check each it block has single test case ID
- script to check each test cases on the azure test plan is linked to a single PBI
- script to check that each PBI is linked to a single feature PBI 
- script to check that each feature PBI is linked with an epic
- script to generate `test-mapping.json`
- script to sync `test-mapping` with azure test plan test cases
- script to calculate and update Feature PBI, PBIs and Epics with automation coverage values
- script to create RTM with the ado client
- script to create HTML report with RTM data
- script to create a Power BI dashboard with the RTM data (csv)

## 🔧 Maintenance

### Daily Tasks
- ✅ Monitor pipeline execution logs
- ✅ Review coverage metrics
- ✅ Update test case automation status when tests are automated

### Weekly Tasks
- ✅ Review RTM report
- ✅ Identify features with low coverage (<50%)
- ✅ Update `test-mapping.json` with new specs
- ✅ Run sync script
- ✅ Plan automation work for next sprint

### Monthly Tasks
- ✅ Audit work item linkages
- ✅ Clean up orphaned test cases (no work item link)
- ✅ Review custom field usage
- ✅ Generate executive summary report
- ✅ Update automation priorities

### Quarterly Tasks
- ✅ Review and optimize queries
- ✅ Update dashboard if needed
- ✅ Analyze coverage trends
- ✅ Set new coverage targets

---
