# Email Template Fix for Employee Name Issue

## Problem Diagnosis
The employee name is not appearing in the email subject and content when testing. This indicates the variables are not being properly resolved.

## Current Email Configuration Issues

### Subject Line Issue
**Current:** `"={{`Timesheet Reminder for ${$json[\"name\"]}`}}"`
**Problem:** Mixed template syntax causing variable resolution failure

### HTML Content Issue
**Current:** Uses `{{$json[\"name\"]}}` in HTML
**Problem:** Variable not being resolved properly

## Fixed Email Configuration

### 1. Correct Subject Line
Replace the current subject with:
```json
"subject": "=Timesheet Reminder for {{$json[\"name\"]}}"
```

### 2. Correct HTML Content
Replace the current HTML with:
```html
<p>Dear {{$json["name"]}},</p>

<p>We hope this message finds you well.</p>

<p>As part of our monthly review, we have identified the following dates between <strong>{{$node["Set Date Range"].json["from_date"]}}</strong> and <strong>{{$node["Set Date Range"].json["to_date"]}}</strong> for which timesheet entries have not been recorded under your name:</p>

<ul>
{{#each $json["missing_days"]}}
  <li>{{this}}</li>
{{/each}}
</ul>

<p>If any of these days were covered by approved leave or if you believe this list is inaccurate, kindly double-check your entries and update your timesheet accordingly in the Odoo portal.</p>

<p>We appreciate your cooperation and timely compliance.</p>

<p>Best regards,<br>
HR Department<br>
Morison Qatar</p>
```

## Alternative HTML Content (JavaScript-based)
If the above doesn't work, use this JavaScript-generated HTML:
```html
=`<p>Dear ${$json["name"]},</p>

<p>We hope this message finds you well.</p>

<p>As part of our monthly review, we have identified the following dates between <strong>${$node["Set Date Range"].json["from_date"]}</strong> and <strong>${$node["Set Date Range"].json["to_date"]}</strong> for which timesheet entries have not been recorded under your name:</p>

<ul>
${$json["missing_days"].map(date => `<li>${date}</li>`).join('')}
</ul>

<p>If any of these days were covered by approved leave or if you believe this list is inaccurate, kindly double-check your entries and update your timesheet accordingly in the Odoo portal.</p>

<p>We appreciate your cooperation and timely compliance.</p>

<p>Best regards,<br>
HR Department<br>
Morison Qatar</p>`
```

## Complete Send Reminder Node Configuration
```json
{
  "parameters": {
    "fromEmail": "hr@morisonqatar.com",
    "toEmail": "your-test-email@domain.com",
    "subject": "=Timesheet Reminder for {{$json[\"name\"]}}",
    "html": "=`<p>Dear ${$json[\"name\"]},</p>\n\n<p>We hope this message finds you well.</p>\n\n<p>As part of our monthly review, we have identified the following dates between <strong>${$node[\"Set Date Range\"].json[\"from_date\"]}</strong> and <strong>${$node[\"Set Date Range\"].json[\"to_date\"]}</strong> for which timesheet entries have not been recorded under your name:</p>\n\n<ul>\n${$json[\"missing_days\"].map(date => `<li>${date}</li>`).join('')}\n</ul>\n\n<p>If any of these days were covered by approved leave or if you believe this list is inaccurate, kindly double-check your entries and update your timesheet accordingly in the Odoo portal.</p>\n\n<p>We appreciate your cooperation and timely compliance.</p>\n\n<p>Best regards,<br>\nHR Department<br>\nMorison Qatar</p>`",
    "options": {
      "appendAttribution": false
    }
  }
}
```

## Debugging Steps

### 1. Check Data Flow
Add a "Code" node before the "Send Reminder" node to debug:
```javascript
// Debug node to check data
console.log("Employee data:", $json);
console.log("Employee name:", $json["name"]);
console.log("Employee email:", $json["work_email"]);
console.log("Missing days:", $json["missing_days"]);
console.log("Date range:", $node["Set Date Range"].json);

return [$input.all()];
```

### 2. Test Email Template
Create a simple test email first:
```json
{
  "subject": "Test Email",
  "html": "=`<p>Testing employee: ${$json[\"name\"]}</p><p>Email: ${$json[\"work_email\"]}</p><p>Missing days: ${JSON.stringify($json[\"missing_days\"])}</p>`"
}
```

### 3. Verify "Find Missing Days" Output
Make sure the "Find Missing Days" node is returning the correct data structure:
```javascript
// At the end of Find Missing Days node
return [{
  json: {
    name: $json["name"] || "Unknown Employee",
    work_email: $json["work_email"] || "no-email@domain.com",
    missing_days: missing,
    debug_info: {
      original_name: $json["name"],
      original_email: $json["work_email"],
      from_date: $node["Set Date Range"].json["from_date"],
      to_date: $node["Set Date Range"].json["to_date"]
    }
  }
}];
```

## Common Issues and Solutions

### Issue 1: Employee Name is Undefined
**Cause:** Data not properly passed through workflow nodes
**Solution:** Check the "Loop All Employees" node assignments:
```json
{
  "assignments": {
    "assignments": [
      {
        "name": "id",
        "value": "={{$json[\"id\"]}}",
        "type": "string"
      },
      {
        "name": "name",
        "value": "={{$json[\"name\"]}}",
        "type": "string"
      },
      {
        "name": "work_email",
        "value": "={{$json[\"work_email\"]}}",
        "type": "string"
      }
    ]
  }
}
```

### Issue 2: Missing Days Not Displaying
**Cause:** Array not properly converted to HTML
**Solution:** Use JavaScript template literals as shown above

### Issue 3: Date Range Not Showing
**Cause:** Node reference incorrect
**Solution:** Verify the "Set Date Range" node name matches exactly

## Testing Checklist
- [ ] Employee name appears in subject line
- [ ] Employee name appears in email greeting
- [ ] Date range displays correctly
- [ ] Missing days list is formatted properly
- [ ] Email is sent to your test email address
- [ ] HTML formatting looks correct

## Step-by-Step Implementation

1. **Update Subject Line:**
   - Go to Send Reminder node
   - Change subject to: `=Timesheet Reminder for {{$json["name"]}}`

2. **Update HTML Content:**
   - Replace HTML with the JavaScript template version above
   - Make sure to use backticks (`) for template literals

3. **Test with Debug Node:**
   - Add a debug node before Send Reminder
   - Check the console output for data structure

4. **Verify and Test:**
   - Run the workflow manually
   - Check your test email
   - Verify all variables are populated

## Production Deployment
Once testing is complete and employee names appear correctly:
1. Change `toEmail` back to `={{$json["work_email"]}}`
2. Remove any debug nodes
3. Test with a single employee first
4. Deploy to production schedule