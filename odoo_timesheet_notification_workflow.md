# Odoo Timesheet Notification Workflow

## Overview
This n8n workflow automatically monitors employee timesheet submissions in Odoo and sends email reminders to employees who have missing timesheet entries for the current month.

## Workflow Schedule
- **Trigger**: Monthly on the 26th at 9:00 AM
- **Date Range**: Checks timesheets from the 1st to 25th of the current month
- **Working Days**: Excludes Fridays and Saturdays (weekend days)

## Workflow Components

### 1. Schedule Trigger
- **Type**: `n8n-nodes-base.scheduleTrigger`
- **Schedule**: Monthly on the 26th at 9:00 AM
- **Purpose**: Initiates the workflow automatically

### 2. Set Date Range
- **Type**: `n8n-nodes-base.code`
- **Function**: Sets the date range for timesheet checking
- **Logic**: 
  - From: 1st of current month
  - To: 25th of current month
- **Output**: `from_date` and `to_date` in YYYY-MM-DD format

### 3. Get Employees
- **Type**: `n8n-nodes-base.httpRequest`
- **Method**: POST to Odoo JSON-RPC API
- **Endpoint**: `https://morisonqatar.odoo.com/jsonrpc`
- **Purpose**: Retrieves all employees with their ID, name, and work email
- **Model**: `hr.employee`
- **Fields**: `["id", "name", "work_email"]`

### 4. Loop All Employees
- **Type**: `n8n-nodes-base.set`
- **Purpose**: Formats employee data for processing
- **Assignments**:
  - `id`: Employee ID
  - `name`: Employee name
  - `work_email`: Employee email address

### 5. Loop Over Items
- **Type**: `n8n-nodes-base.splitInBatches`
- **Purpose**: Processes employees in batches to prevent overwhelming the system

### 6. Get Timesheets per Employee
- **Type**: `n8n-nodes-base.httpRequest`
- **Method**: POST to Odoo JSON-RPC API
- **Purpose**: Retrieves timesheet entries for each employee
- **Model**: `account.analytic.line`
- **Filters**:
  - `employee_id = current_employee_id`
  - `date >= from_date`
  - `date <= to_date`
- **Fields**: `["date"]`

### 7. Get Public Holidays
- **Type**: `n8n-nodes-base.httpRequest`
- **Method**: POST to Odoo JSON-RPC API
- **Purpose**: Retrieves public holidays in the date range
- **Model**: `resource.calendar.leaves`
- **Filters**:
  - `date_from >= from_date`
  - `date_to <= to_date`
- **Fields**: `["date_from", "date_to"]`

### 8. Get Approved Leaves for Employee
- **Type**: `n8n-nodes-base.httpRequest`
- **Method**: POST to Odoo JSON-RPC API
- **Purpose**: Retrieves approved leave days for each employee
- **Model**: `hr.leave`
- **Filters**:
  - `employee_id = current_employee_id`
  - `date_from >= from_date`
  - `date_to <= to_date`
  - `state = "validate"`
- **Fields**: `["date_from", "date_to"]`

### 9. Find Missing Days
- **Type**: `n8n-nodes-base.code`
- **Purpose**: Calculates missing workdays using JavaScript
- **Logic**:
  1. Gets working date range (1st to 25th)
  2. Extracts timesheet dates
  3. Extracts public holiday dates
  4. Expands leave date ranges to individual dates
  5. Combines all excluded dates
  6. Loops through working days, skipping Fridays and Saturdays
  7. Identifies missing days not covered by timesheets, holidays, or leaves
- **Output**: Employee details with missing days array

### 10. Check if Missing Days > 0
- **Type**: `n8n-nodes-base.if`
- **Purpose**: Filters employees who have missing timesheet days
- **Condition**: `missing_days.length > 0`

### 11. Skip Excluded Emails
- **Type**: `n8n-nodes-base.if`
- **Purpose**: Excludes specific email addresses from notifications
- **Excluded Emails**:
  - `kurian@morisonqatar.com`
  - `george@morisonqatar.com`
  - `sajan@morisonqatar.com`
  - `kevin@morisonqatar.com`

### 12. Send Reminder
- **Type**: `n8n-nodes-base.emailSend`
- **From**: `hr@morisonqatar.com`
- **To**: Employee's work email
- **Subject**: "Timesheet Reminder for {Employee Name}"
- **Content**: HTML email with:
  - Personalized greeting
  - Date range information
  - List of missing days
  - Instructions to update timesheet
  - Professional closing

## Odoo API Configuration
- **Database**: `morisonqatar`
- **User ID**: `2`
- **API Key**: `aa199d2f588e9aa4981f582cd8335c5270e6d3f2`
- **URL**: `https://morisonqatar.odoo.com/jsonrpc`

## Email Configuration
- **SMTP Credential**: `hr@morisonqatar`
- **Sender**: `hr@morisonqatar.com`
- **Override To**: `it@morisonqatar.com` (for testing)

## Working Days Logic
- **Excluded Days**: Friday (day 5) and Saturday (day 6)
- **Working Days**: Sunday through Thursday
- **Holiday Handling**: Public holidays are excluded from missing day calculations
- **Leave Handling**: Approved leaves are excluded from missing day calculations

## Data Flow
1. **Schedule Trigger** → **Set Date Range** → **Get Employees**
2. **Get Employees** → **Loop All Employees** → **Loop Over Items**
3. **Loop Over Items** → **Get Timesheets per Employee** → **Get Public Holidays**
4. **Get Public Holidays** → **Get Approved Leaves for Employee** → **Find Missing Days**
5. **Find Missing Days** → **Check if Missing Days > 0** → **Skip Excluded Emails**
6. **Skip Excluded Emails** → **Send Reminder**

## Potential Improvements

### 1. Error Handling
- Add error handling nodes for API failures
- Implement retry logic for failed requests
- Add logging for debugging

### 2. Performance Optimization
- Implement parallel processing for API calls
- Add caching for public holidays
- Optimize batch processing size

### 3. Flexibility Enhancements
- Make excluded emails configurable
- Add support for different working day patterns
- Allow customizable date ranges

### 4. Monitoring and Alerts
- Add success/failure notifications
- Implement audit trail logging
- Add metrics collection

### 5. Security Improvements
- Use environment variables for API credentials
- Implement proper authentication handling
- Add rate limiting for API calls

## Maintenance Notes
- **API Key**: Should be rotated regularly
- **Email Template**: Can be customized in the Send Reminder node
- **Working Days**: Currently hardcoded for Friday/Saturday weekends
- **Excluded Emails**: Manually maintained in the workflow
- **Date Range**: Fixed to 1st-25th of each month

## Testing
- **Test Mode**: Email is currently set to send to `it@morisonqatar.com`
- **Production Mode**: Remove the email override in the Send Reminder node
- **Manual Testing**: Can be triggered manually from the n8n interface