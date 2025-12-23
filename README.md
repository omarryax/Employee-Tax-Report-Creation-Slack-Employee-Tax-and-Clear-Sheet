# Employee Tax Report Creation, Slack Employee Tax and Clear Sheet Automation

## 📋 Project Overview

This project contains a comprehensive **Make.com automation workflow** that automates employee tax report generation, processing, and distribution. The workflow monitors Google Sheets for trigger changes, fetches employee and payroll data from multiple systems, calculates tax information, updates spreadsheets, exports reports to Excel, and distributes them via Slack.

### Key Features

- 📊 **Automated Tax Report Generation**: Creates employee tax reports automatically when triggered
- 🔄 **Multi-System Integration**: Integrates BambooHR, Xero, and Google Sheets
- 💰 **Tax Calculation**: Calculates taxable amounts and tax deductions from payroll data
- 📝 **Employee Data Processing**: Fetches and processes employee information including CNIC, NTN, compensation, and schedules
- 📤 **Slack Distribution**: Automatically uploads tax reports to Slack channels
- 🧹 **Sheet Management**: Clears and prepares sheets for the next reporting period
- 🏢 **Multi-Company Support**: Handles multiple companies (company, Company2, Company3)

**Trigger**: Google Sheets cell change (X1 column: false → true)

---

## 🗂️ Project Files

- **`Employee Tax Report Creation, Slack Employee Tax and Clear Sheet.blueprint.json`** - Main Make.com workflow blueprint (ready to import)
- **`README-Employee-Tax.md`** - This documentation file

---

## 🔄 Workflow Process

### High-Level Flow

```
1. Google Sheets Trigger (Cell X1: false → true)
   ↓
2. Validate Trigger Conditions
   ├─→ Spreadsheet Name = "Employee Tax"
   ├─→ Sheet Name = "Sheet1"
   └─→ Cell Changed = X1
   ↓
3. Route by Cell Location
   ├─→ X1 Changed → Process Tax Report
   │   ├─→ Fetch Employee Data (BambooHR)
   │   ├─→ Get Bank Transactions (Xero)
   │   ├─→ Calculate Compensation & Schedules
   │   ├─→ Update Google Sheets with Tax Data
   │   ├─→ Export to Excel
   │   └─→ Upload to Slack
   │
   └─→ Z1 Changed → Clear Sheet & Update Date
       ├─→ Clear Data Range (A2:J51)
       └─→ Update Date for Next Month
   ↓
4. Complete Workflow Execution
```

### Detailed Steps

1. **Google Sheets Trigger**
   - **Module**: `google-sheets:watchUpdatedCells`
   - **Spreadsheet**: "Employee Tax"
   - **Sheet**: "Sheet1"
   - **Trigger Condition**: Cell X1 changes from `false` to `true`
   - **Webhook ID**: 123456

2. **Initial Validation**
   - Checks if old value = `false` and new value = `true`
   - Verifies spreadsheet name = "Employee Tax"
   - Confirms sheet name = "Sheet1"
   - Validates cell location = "X1"

3. **Company Routing**
   - Routes based on company name in column W (rowValues[].`22`)
   - Supports: "company", "Company2", "Company3"
   - Each company has separate Xero organization IDs

4. **BambooHR Employee Data Fetch**
   - **API Call**: `/v1/reports/custom`
   - **Fields Retrieved**:
     - `status` - Employment status
     - `fullName1` - Employee full name
     - `4388` - Custom field (likely employee ID)
     - `payType` - Payment type (Hourly/Salary)
     - `payRate` - Pay rate
     - `4643` - Custom field
     - `EmploymentHistoryStatus` - Employment history status
   - Filters by company name

5. **Xero Bank Transactions**
   - **Query**: Fetches authorized SPEND transactions
   - **Date Range**: From date in column Y (rowValues[].`24`) + 1 day to current date
   - **Filter**: `Type=="SPEND" && Status="AUTHORISED"`
   - **Account Code**: 825 (Tax account)
   - **Organization**: Switches based on company name

6. **Employee Compensation Data**
   - Fetches compensation table from BambooHR
   - Gets latest compensation record for the period
   - Extracts:
     - Pay Type (Hourly/Salary)
     - Pay Rate & Currency
     - Pay Schedule
     - Compensation Reason
     - Start Date
     - Pay Period
     - Overtime Status & Rate

7. **Employee Schedule Data**
   - Fetches custom schedules from BambooHR
   - Gets schedule information:
     - Schedule Date
     - Schedule Shift Time
     - Schedule Day
     - Bi-Weekly Day
     - Hybrid Day
     - Exclude flag

8. **Employment Status**
   - Fetches employment status history
   - Filters for active employees (not "Terminated")
   - Gets current employment status

9. **Working Days Calculation**
   - Calculates number of working days in the month
   - Excludes Sundays
   - Uses repeater to iterate through days

10. **Tax Amount Calculation**
    - Processes Xero line items
    - Calculates taxable amount based on:
      - Pay type (Hourly vs Monthly Salary)
      - Working days
      - Schedule information
      - Employment status (Full-time = 8 hours, Part-time = 4 hours)
    - Formula: `(Working Days × Pay Rate × Hours)`
    - Extracts tax amount from account code 825

11. **Google Sheets Update**
    - **Row Filter**: Finds empty row (column A doesn't exist)
    - **Data Updated**:
      - Column A (0): "149/3" (Payment Section)
      - Column C (2): CNIC (formatted: XXXXX-XXXXXXX-X)
      - Column D (3): Employee Name
      - Column E (4): (City)
      - Column I (8): Taxable Amount (sum of calculated amounts)
      - Column J (9): Tax Amount (from Xero)

12. **Sheet Export & Slack Upload**
    - **Export**: Converts Google Sheet to Excel (.xlsx)
    - **File Naming**: `{Spreadsheet Name} {Date from Y1}.xlsx`
    - **Slack Upload Process**:
      1. Get upload URL from Slack API
      2. Upload file to Slack
      3. Wait 20 seconds for processing
      4. Complete upload and post to channel
      5. Get permalink to file
    - **Company-Specific Channels**: Different Slack workspaces per company

13. **Sheet Clearing (Z1 Trigger)**
    - When Z1 is changed:
      - Clears data range: `A2:J51`
      - Updates Y1 with next month's date
      - Format: `M/1/YYYY` (e.g., "7/1/2025")

---

## 🚀 Setup Instructions

### Prerequisites

1. **Make.com Account**
   - Active Make.com (formerly Integromat) account
   - Appropriate subscription plan for workflow complexity

2. **Required Services & Credentials**
   - **Google Sheets**: Google account with Sheets API access
   - **BambooHR**: Account with API access
   - **Xero**: Account with API access for multiple organizations
   - **Slack**: Workspace(s) with API tokens for each company

### Installation Steps

1. **Import Workflow**
   ```
   1. Log in to Make.com
   2. Go to Scenarios → Import
   3. Select "Employee Tax Report Creation, Slack Employee Tax and Clear Sheet.blueprint.json"
   4. Click Import
   ```

2. **Configure Google Sheets Connection**
   - Go to Connections → Add Connection
   - Select Google Sheets
   - Authenticate with OAuth2
   - Grant Sheets API permissions
   - Verify access to "Employee Tax" spreadsheet

3. **Configure BambooHR Connection**
   - Add BambooHR connection
   - Enter API key and subdomain
   - Test connection
   - Verify access to employee data and custom fields

4. **Configure Xero Connection**
   - Add Xero connection
   - Authenticate with OAuth2
   - Grant accounting API access
   - Configure organization IDs:
     - Company: `[ORG_ID_1]`
     - Company2: `[ORG_ID_2]`
     - Company3: `[ORG_ID_3]`
   - Update tenant IDs in workflow

5. **Configure Slack Connections**
   - Add Slack connections for each company:
     - Slack company (for "company")
     - Slack Company2 (for "Company2")
     - Slack Company3 (for "Company3")
   - Use API key authentication
   - Update channel IDs: `[CHANNEL_ID]` in workflow

6. **Update Configuration Values**
   - **Xero Organization IDs**: Replace placeholders:
     - `[ORG_ID_1]` - Company organization ID
     - `[ORG_ID_2]` - Company2 organization ID
     - `[ORG_ID_3]` - Company3 organization ID
   - **Slack Channel IDs**: Replace `[CHANNEL_ID]` with actual channel IDs
   - **Google Spreadsheet ID**: Update if using different spreadsheet

7. **Set Up Google Sheets**
   - Create or verify "Employee Tax" spreadsheet
   - Ensure "Sheet1" exists
   - Set up columns:
     - Column A: Payment Section
     - Column B: TaxPayer_NTN
     - Column C: TaxPayer_CNIC
     - Column D: TaxPayer_Name
     - Column E: TaxPayer_City
     - Column F: TaxPayer_Address
     - Column G: TaxPayer_Status
     - Column H: TaxPayer_Business_Name
     - Column I: Taxable_Amount
     - Column J: Tax_Amount
     - Column W: Company name
     - Column X: Trigger flag (boolean)
     - Column Y: Date (format: "Jun 2025")
     - Column Z: Clear trigger (boolean)

8. **Test the Workflow**
   - Use Make.com's "Run once" feature
   - Set cell X1 to `true` in Google Sheets
   - Check execution history for errors
   - Verify data is updated correctly
   - Confirm Slack upload works

---

## ⚙️ Configuration Details

### Key Parameters

#### Google Sheets Trigger
- **Spreadsheet Name**: "Employee Tax"
- **Sheet Name**: "Sheet1"
- **Trigger Cell**: X1
- **Trigger Condition**: `oldValue = false` AND `newValue = true`

#### BambooHR Configuration
- **Custom Report Fields**:
  - `status` - Employment status
  - `fullName1` - Employee name
  - `4388` - Employee ID field
  - `payType` - Payment type
  - `payRate` - Pay rate
  - `4643` - Custom field
  - `EmploymentHistoryStatus` - Employment history
- **API Endpoints Used**:
  - `/v1/reports/custom` - Employee report
  - `/v1/employees/{id}/tables/compensation` - Compensation data
  - `/v1/employees/{id}/tables/customSchedules` - Schedule data
  - `/v1/employees/{id}/tables/employmentStatus` - Employment status

#### Xero Configuration
- **Base URL**: `api.xro`
- **Endpoint**: `/2.0/BankTransactions`
- **Query Filter**: `Type=="SPEND" && Status="AUTHORISED"`
- **Account Code**: 825 (Tax account)
- **Date Range**: From column Y date + 1 day to current date
- **Organizations**:
  - Company: `[ORG_ID_1]`
  - Company2: `[ORG_ID_2]`
  - Company3: `[ORG_ID_3]`

#### Google Sheets Data Mapping
- **Column A (0)**: Payment Section = "149/3"
- **Column C (2)**: CNIC (formatted: XXXXX-XXXXXXX-X)
- **Column D (3)**: Employee Name
- **Column E (4)**: City = "KARACHI"
- **Column I (8)**: Taxable Amount (calculated sum)
- **Column J (9)**: Tax Amount (from Xero)

#### Slack Configuration
- **Upload Process**:
  1. `files.getUploadURLExternal` - Get upload URL
  2. Upload file to URL
  3. Wait 20 seconds
  4. `files.completeUploadExternal` - Complete upload
  5. `chat.getPermalink` - Get file link
- **File Naming**: `{Spreadsheet Name} {Date}.xlsx`
- **Channel IDs**: Company-specific (update placeholders)

#### Sheet Clearing
- **Trigger**: Z1 cell change
- **Clear Range**: `A2:J51`
- **Date Update**: Y1 = Next month (format: `M/1/YYYY`)

---

## 📊 Workflow Structure

### Module Types Used

#### Integration Modules

- **Google Sheets** (`google-sheets:watchUpdatedCells`, `updateRow`, `filterRows`, `updateCell`, `makeAPICall`)
  - Watches for cell changes
  - Updates rows with tax data
  - Filters and finds rows
  - Exports to Excel format

- **BambooHR** (`bamboohr:MakeApiCall`)
  - Fetches employee reports
  - Gets compensation data
  - Retrieves schedule information
  - Gets employment status

- **Xero** (`xero:MakeAPICall`, `GetBankTransaction`)
  - Fetches bank transactions
  - Gets transaction details
  - Filters by account code and date

- **Slack** (`slack:MakeAPICall`, HTTP modules with API key auth)
  - Uploads files to Slack
  - Posts to channels
  - Gets file permalinks

#### Control Flow Modules

- **BasicRouter** (`builtin:BasicRouter`)
  - Routes by cell location (X1 vs Z1)
  - Routes by company name
  - Routes by data conditions

- **BasicFeeder** (`builtin:BasicFeeder`)
  - Iterates through arrays
  - Processes bank transactions
  - Processes compensation records
  - Processes schedule data

- **BasicAggregator** (`builtin:BasicAggregator`)
  - Aggregates compensation data
  - Aggregates schedule data
  - Aggregates employment status
  - Aggregates calculated amounts

- **Set Variables** (`util:SetVariables`)
  - Stores compensation variables
  - Stores schedule variables
  - Stores employment status

- **Function Aggregator** (`util:FunctionAggregator2`)
  - Calculates working days (sum)
  - Excludes Sundays

- **Basic Repeater** (`builtin:BasicRepeater`)
  - Iterates through days of month
  - Calculates working days

- **Sleep** (`util:FunctionSleep`)
  - Waits 2 seconds (employee/sheet check)
  - Waits 20 seconds (Slack file processing)

---

## 🔍 How It Works

### Tax Report Generation Flow

1. **Trigger Activation**
   - User sets cell X1 to `true` in Google Sheets
   - Workflow detects change from `false` to `true`

2. **Data Collection**
   - Fetches employee list from BambooHR (filtered by company)
   - Gets bank transactions from Xero for the period
   - Retrieves employee compensation details
   - Fetches schedule information
   - Gets employment status

3. **Data Processing**
   - Matches Xero transactions with employees by name
   - Calculates taxable amount based on:
     - Pay type (Hourly vs Salary)
     - Working days (excluding Sundays)
     - Hours per day (8 for full-time, 4 for part-time)
     - Pay rate
   - Extracts tax amount from Xero line items (account code 825)

4. **Sheet Update**
   - Finds next empty row
   - Updates row with:
     - CNIC (formatted from nationalId)
     - Employee name
     - City
     - Taxable amount
     - Tax amount

5. **Report Distribution**
   - Exports sheet to Excel format
   - Uploads to Slack
   - Posts to company-specific channel
   - Generates permalink

### Sheet Clearing Flow

1. **Trigger Activation**
   - User sets cell Z1 to `true` in Google Sheets

2. **Sheet Clearing**
   - Clears data range A2:J51
   - Updates Y1 with next month's date
   - Prepares sheet for next reporting period

---

## ⚠️ Important Notes

### Manual Configuration Required

1. **Xero Organization IDs**
   - Replace `[ORG_ID_1]`, `[ORG_ID_2]`, `[ORG_ID_3]` with actual organization IDs
   - Update in all Xero API call modules
   - Verify tenant IDs match your Xero organizations

2. **Slack Channel IDs**
   - Replace `[CHANNEL_ID]` with actual Slack channel IDs
   - Update for each company's Slack workspace
   - Verify channel IDs in `files.completeUploadExternal` calls

3. **Google Spreadsheet ID**
   - Update spreadsheet ID if using different spreadsheet
   - Verify spreadsheet name is "Employee Tax"
   - Ensure sheet name is "Sheet1"

4. **BambooHR Custom Fields**
   - Verify custom field IDs match your BambooHR setup:
     - Field `4388` - Employee ID
     - Field `4643` - Custom field
   - Update if your field IDs are different

5. **Date Formatting**
   - Column Y format: "Jun 2025" (Month Year)
   - Y1 update format: "M/1/YYYY" (e.g., "7/1/2025")
   - Timezone: "Asia/Karachi" for date calculations

6. **CNIC Formatting**
   - Format: `XXXXX-XXXXXXX-X` (5 digits, 7 digits, 1 digit)
   - Removes dashes from nationalId and reformats
   - Handles null values gracefully

### Workflow Behavior

- **Trigger**: Manual (set X1 or Z1 to `true`)
- **Processing**: Processes one employee per execution cycle
- **Data Matching**: Matches employees by name (case-insensitive)
- **Tax Calculation**: Complex logic based on pay type, schedule, and working days
- **Error Handling**: Built-in retry logic and error management
- **Multi-Company**: Supports 3 different companies with separate Xero/Slack configs

---

## 🔍 Troubleshooting

### Common Issues

1. **Workflow Not Triggering**
   - **Problem**: Cell change not detected
   - **Solution**:
     - Verify webhook is active in Make.com
     - Check spreadsheet name is exactly "Employee Tax"
     - Ensure sheet name is "Sheet1"
     - Verify cell X1 or Z1 is being changed
     - Check old value was `false` before setting to `true`

2. **Employee Data Not Found**
   - **Problem**: Employee not matched or data missing
   - **Solution**:
     - Verify employee name matches exactly in BambooHR and Xero
     - Check company name in column W matches filter
     - Verify employee has compensation record for the period
     - Check employment status is not "Terminated"
     - Review BambooHR API connection

3. **Xero Transactions Not Found**
   - **Problem**: No transactions or wrong data
   - **Solution**:
     - Verify date in column Y is correct
     - Check Xero organization ID matches company
     - Verify account code 825 exists and has transactions
     - Review date range calculation
     - Check transaction status is "AUTHORISED"

4. **Tax Calculation Errors**
   - **Problem**: Wrong taxable amount calculated
   - **Solution**:
     - Verify pay type is correct (Hourly vs Salary)
     - Check schedule data is available
     - Review working days calculation
     - Verify employment status (Full-time vs Part-time)
     - Check pay rate and currency

5. **Slack Upload Fails**
   - **Problem**: File not uploaded or posted
   - **Solution**:
     - Verify Slack API key is valid
     - Check channel ID is correct
     - Review file size limits
     - Ensure 20-second wait is sufficient
     - Check Slack workspace permissions

6. **Sheet Not Updating**
   - **Problem**: Data not written to sheet
   - **Solution**:
     - Verify Google Sheets connection
     - Check spreadsheet permissions
     - Review row filter logic (empty row detection)
     - Verify column mappings
     - Check data format matches sheet format

---

## 📝 Usage Examples

### Example 1: Generate Tax Report

1. **Prepare Data**:
   - Ensure column Y has the reporting month (e.g., "Jun 2025")
   - Set column W with company name (e.g., "company")

2. **Trigger Report**:
   - Set cell X1 to `true` in Google Sheets
   - Workflow automatically:
     - Fetches employee data
     - Gets Xero transactions
     - Calculates tax amounts
     - Updates sheet
     - Exports and uploads to Slack

3. **Result**:
   - New row added with employee tax information
   - Excel file uploaded to Slack channel
   - File permalink available

### Example 2: Clear Sheet for Next Month

1. **Trigger Clear**:
   - Set cell Z1 to `true` in Google Sheets

2. **Result**:
   - Data range A2:J51 is cleared
   - Column Y1 updated to next month (e.g., "Jul 2025")

### Example 3: Multi-Company Processing

1. **Company 1**:
   - Column W = "company"
   - Uses Xero Org ID 1
   - Posts to Slack workspace 1

2. **Company 2**:
   - Column W = "Company2"
   - Uses Xero Org ID 2
   - Posts to Slack workspace 2

3. **Company 3**:
   - Column W = "Company3"
   - Uses Xero Org ID 3
   - Posts to Slack workspace 3

---

## 🔧 Customization

### Modify Tax Calculation Logic

- **Working Days**: Adjust repeater logic to exclude different days
- **Hours Calculation**: Change full-time (8) and part-time (4) hours
- **Pay Type Handling**: Modify calculation for different pay types
- **Tax Account Code**: Update account code 825 if different

### Add New Companies

1. Add company name to router filter
2. Add Xero organization ID mapping
3. Add Slack workspace configuration
4. Update channel ID

### Change Data Fields

- **BambooHR Fields**: Update field IDs in custom report
- **Sheet Columns**: Modify column mappings in updateRow
- **CNIC Format**: Adjust formatting logic if needed

### Modify Date Ranges

- **Reporting Period**: Change date calculation in Xero query
- **Date Format**: Update date formatting in sheet updates
- **Timezone**: Adjust timezone settings if needed

---

## 📚 Additional Resources

- [Make.com Documentation](https://www.make.com/en/help)
- [Google Sheets API Documentation](https://developers.google.com/sheets/api)
- [BambooHR API Documentation](https://documentation.bamboohr.com/docs)
- [Xero API Documentation](https://developer.xero.com/documentation)
- [Slack API Documentation](https://api.slack.com/)

---

## 📄 License & Compliance

This workflow is provided as-is for internal use. Ensure compliance with:
- Make.com terms of service
- Google Sheets API usage policies
- BambooHR API usage policies
- Xero API terms of service
- Slack workspace policies
- Data privacy regulations (GDPR, local tax laws, etc.)
- **Tax Compliance**: Ensure tax calculations comply with local tax regulations

---

## 👤 Support

For issues or questions:

1. Check Make.com execution history for detailed logs
2. Review module-specific error messages
3. Verify all connections are active and valid
4. Test individual modules in isolation
5. Consult Make.com community forums
6. Review service API documentation
7. Verify tax calculation formulas match your requirements

---

## 🔄 Version History

- **v1.0** (Current)
  - Initial workflow implementation
  - Google Sheets trigger integration
  - BambooHR employee data fetching
  - Xero transaction processing
  - Tax calculation logic
  - Multi-company support
  - Slack file upload automation
  - Sheet clearing functionality

---

## 📋 Workflow Statistics

- **Total Modules**: 100+ modules
- **Integration Services**: 4 (Google Sheets, BambooHR, Xero, Slack)
- **Companies Supported**: 3 (company, Company2, Company3)
- **Trigger Type**: Google Sheets cell change
- **Data Processing**: Employee compensation, schedules, tax calculations
- **Output Format**: Excel (.xlsx)
- **Distribution**: Slack channels

---

**Last Updated**: January 2025  
**Workflow Version**: 1.0  
**Make.com Compatibility**: Make.com (Integromat)  
**Required Services**: Google Sheets, BambooHR, Xero, Slack  
**Trigger**: Manual (Google Sheets cell change)


