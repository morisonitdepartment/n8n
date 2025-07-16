# Debug Data Flow Issue

## The Real Problem
The issue isn't the email address - it's that the employee data (name, missing days) isn't flowing through the workflow correctly to the Send Reminder node.

## Quick Debug Steps

### 1. Add Debug Node Before Send Reminder
Add a **Code node** right before the "Send Reminder" node:

```javascript
// Debug what data is actually reaching the Send Reminder node
console.log("=== DEBUG: Data reaching Send Reminder ===");
console.log("Full $json:", JSON.stringify($json, null, 2));
console.log("Employee name:", $json["name"]);
console.log("Employee email:", $json["work_email"]);
console.log("Missing days:", $json["missing_days"]);
console.log("From Set Date Range:", $node["Set Date Range"].json);

// Log all available node data
console.log("Available nodes:");
console.log("- Get Employees result:", $node["Get Employees"]?.json);
console.log("- Find Missing Days result:", $node["Find Missing Days"]?.json);

return [$input.all()];
```

### 2. Check Each Node's Output

#### A. Test "Get Employees" Node
Run just this node and check the output:
- Should return: `[{"id": 123, "name": "John Doe", "work_email": "john@company.com"}, ...]`

#### B. Test "Loop All Employees" Node
Check if this node properly assigns the data:
```json
{
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
```

#### C. Test "Find Missing Days" Node
Make sure it's returning the correct structure:
```javascript
// At the end of Find Missing Days node, make sure you return:
return [{
  json: {
    name: $json["name"],
    work_email: $json["work_email"], 
    missing_days: missing
  }
}];
```

### 3. Common Issues and Fixes

#### Issue 1: "Loop Over Items" Breaking Data Flow
**Problem**: The "Loop Over Items" (splitInBatches) node might be causing data loss.

**Fix**: Make sure it's configured correctly:
- **Batch Size**: Set to 1 for testing
- **Reset**: Should be false
- **Connection**: Make sure it loops back to itself properly

#### Issue 2: "Find Missing Days" Not Preserving Employee Data
**Problem**: The JavaScript code might not be passing through the employee data correctly.

**Fix**: Update the "Find Missing Days" node to ensure it preserves all employee data:
```javascript
try {
    // ... existing logic for finding missing days ...
    
    // Make sure to return ALL employee data
    return [{
        json: {
            id: $json["id"],
            name: $json["name"],
            work_email: $json["work_email"],
            missing_days: missing,
            from_date: $node["Set Date Range"].json["from_date"],
            to_date: $node["Set Date Range"].json["to_date"]
        }
    }];
} catch (error) {
    return [{
        json: {
            id: $json["id"],
            name: $json["name"] || "Unknown Employee",
            work_email: $json["work_email"] || "no-email",
            missing_days: [],
            error: error.message
        }
    }];
}
```

#### Issue 3: Node References Not Working
**Problem**: `$node["Set Date Range"].json` might not be accessible from the Send Reminder node.

**Fix**: Pass the date range through the data flow:
```javascript
// In Find Missing Days node, include date range in output:
return [{
    json: {
        name: $json["name"],
        work_email: $json["work_email"],
        missing_days: missing,
        from_date: $node["Set Date Range"].json["from_date"],
        to_date: $node["Set Date Range"].json["to_date"]
    }
}];
```

Then in the email template use:
```
=`<p>Dear ${$json["name"]},</p>
<p>Missing dates between ${$json["from_date"]} and ${$json["to_date"]}:</p>
<ul>
${$json["missing_days"].map(date => `<li>${date}</li>`).join('')}
</ul>`
```

### 4. Test with Single Employee First

**Temporary Fix for Testing**:
1. Add a **Filter** node after "Get Employees" 
2. Set condition: `={{$json["name"]}} === "Specific Employee Name"`
3. This will test with just one employee

### 5. Check Workflow Execution Order

Make sure the workflow flows correctly:
1. Schedule Trigger → Set Date Range → Get Employees
2. Get Employees → Loop All Employees → Loop Over Items
3. Loop Over Items → Get Timesheets → Get Holidays → Get Leaves
4. Get Leaves → Find Missing Days → Check if Missing > 0
5. Check if Missing > 0 → Skip Excluded → Send Reminder

## Quick Test Steps

1. **Add the debug node** before Send Reminder
2. **Run the workflow manually** (don't wait for schedule)
3. **Check the execution logs** to see what data is actually flowing through
4. **Look for the console.log output** in the workflow execution details

## Expected Debug Output
You should see something like:
```
=== DEBUG: Data reaching Send Reminder ===
Full $json: {
  "name": "John Doe",
  "work_email": "john@company.com", 
  "missing_days": ["2024-01-15", "2024-01-16"]
}
Employee name: John Doe
Employee email: john@company.com
Missing days: ["2024-01-15", "2024-01-16"]
```

## If Debug Shows Empty Data
If the debug shows empty or undefined data, the issue is in the workflow flow, not the email template. Check:
1. "Loop Over Items" configuration
2. "Find Missing Days" return statement
3. Node connections

Let me know what the debug output shows and I can help fix the specific issue!