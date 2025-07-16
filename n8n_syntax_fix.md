# n8n Expression Syntax Fix

## Error Analysis
**Error**: `invalid syntax`
**Problem**: Extra quotes around the expression causing syntax error

## Current Wrong Syntax
```json
"subject": "=Timesheet Reminder for {{$json[\"name\"]}}"
```

## Correct n8n Syntax

### Subject Line (Correct)
In the Subject field, enter exactly this (without any outer quotes):
```
=Timesheet Reminder for {{$json["name"]}}
```

### HTML Content (Correct)
In the HTML field, enter exactly this (without any outer quotes):
```
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

## Alternative Simple Syntax (If above doesn't work)

### Subject Line (Simple)
```
={{`Timesheet Reminder for ${$json["name"]}`}}
```

### HTML Content (Simple)
```
={{`<p>Dear ${$json["name"]},</p>

<p>We hope this message finds you well.</p>

<p>As part of our monthly review, we have identified the following dates between <strong>${$node["Set Date Range"].json["from_date"]}</strong> and <strong>${$node["Set Date Range"].json["to_date"]}</strong> for which timesheet entries have not been recorded under your name:</p>

<ul>
${$json["missing_days"].map(date => `<li>${date}</li>`).join('')}
</ul>

<p>If any of these days were covered by approved leave or if you believe this list is inaccurate, kindly double-check your entries and update your timesheet accordingly in the Odoo portal.</p>

<p>We appreciate your cooperation and timely compliance.</p>

<p>Best regards,<br>
HR Department<br>
Morison Qatar</p>`}}
```

## Step-by-Step Instructions

### 1. Open Send Reminder Node
- Click on the "Send Reminder" node in your workflow

### 2. Update Subject Field
- Clear the current Subject field completely
- Type (or paste) exactly: `=Timesheet Reminder for {{$json["name"]}}`
- DO NOT add any quotes around it

### 3. Update HTML Field
- Clear the current HTML field completely
- Copy and paste the HTML content from above
- Make sure it starts with `=` and backticks
- DO NOT add any quotes around it

### 4. Other Fields
- From Email: `hr@morisonqatar.com`
- To Email: `your-test-email@domain.com` (for testing)

## Common n8n Expression Rules

1. **Start with `=`**: All expressions must start with `=`
2. **No outer quotes**: Don't wrap the entire expression in quotes
3. **Use backticks for templates**: Use `` ` `` for JavaScript template literals
4. **Variable access**: Use `$json["field"]` or `$node["NodeName"].json["field"]`

## Testing Order

### Test 1: Simple Subject First
Try just the subject line first:
```
=Timesheet Reminder for {{$json["name"]}}
```

### Test 2: Simple HTML
If subject works, try simple HTML:
```
=`Dear ${$json["name"]}, your missing days are: ${$json["missing_days"].join(", ")}`
```

### Test 3: Full HTML
Once simple versions work, use the full HTML template

## Debug Data Structure

Add a Code node before Send Reminder to check data:
```javascript
// Check what data is available
console.log("Full data:", JSON.stringify($json, null, 2));
console.log("Name:", $json["name"]);
console.log("Email:", $json["work_email"]);
console.log("Missing days:", $json["missing_days"]);

return [$input.all()];
```

## Expected Results After Fix

- **Subject**: "Timesheet Reminder for John Doe"
- **Email Body**: "Dear John Doe, ..."
- **Missing Days**: Listed as bullet points
- **Date Range**: Properly formatted dates

## If Still Getting Errors

Try this minimal working example first:

**Subject**: `=Test for {{$json["name"]}}`

**HTML**: `=Test email for ${$json["name"]}`

Once this works, gradually add more complexity.