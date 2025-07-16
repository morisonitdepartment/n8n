# How to Add Debug Node in n8n

## Step-by-Step Instructions

### 1. Open Your Workflow
- Go to your n8n workflow editor
- Open the "Odoo Timesheet Notification" workflow

### 2. Add a Code Node
1. **Click the "+" button** between "Skip Excluded Emails" and "Send Reminder" nodes
2. **Search for "Code"** in the node search box
3. **Select "Code"** node (it should be `n8n-nodes-base.code`)
4. **Click to add it**

### 3. Configure the Debug Code Node
1. **Click on the new Code node** you just added
2. **In the code editor**, replace any existing code with this:

```javascript
// Debug what data is actually reaching the Send Reminder node
console.log("=== DEBUG: Data reaching Send Reminder ===");
console.log("Full $json:", JSON.stringify($json, null, 2));
console.log("Employee name:", $json["name"]);
console.log("Employee email:", $json["work_email"]);
console.log("Missing days:", $json["missing_days"]);
console.log("From Set Date Range:", $node["Set Date Range"]?.json);

// Log the input data structure
console.log("Input items:", $input.all());

return $input.all();
```

3. **Name the node** "Debug Data" (optional but helpful)
4. **Click "Save"** or close the node editor

### 4. Connect the Debug Node
Make sure the connections are:
- **"Skip Excluded Emails"** → **"Debug Data"** → **"Send Reminder"**

### 5. Run the Workflow
1. **Click "Test workflow"** button (or "Execute Workflow")
2. **Wait for execution to complete**
3. **Check the execution results**

### 6. View Debug Output
1. **Click on the "Debug Data" node** after execution
2. **Look for the console output** in the execution details
3. **OR** check the workflow execution logs

## Alternative: Use Function Node
If Code node doesn't work, you can use **Function node**:

1. **Add "Function" node** instead of Code node
2. **Use the same JavaScript code** as above
3. **Make sure to return the data** with `return items;`

## What You Should See in Debug Output

### Good Output (Working):
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

### Bad Output (Not Working):
```
=== DEBUG: Data reaching Send Reminder ===
Full $json: {}
Employee name: undefined
Employee email: undefined
Missing days: undefined
```

## Finding the Debug Output

### Method 1: Node Execution Details
1. After running workflow, click on the "Debug Data" node
2. Look for "Output" or "Logs" section
3. Console.log messages should appear there

### Method 2: Workflow Execution Log
1. Go to workflow execution history
2. Click on the latest execution
3. Look for log messages in the execution details

### Method 3: Browser Developer Tools
1. Open browser developer tools (F12)
2. Go to Console tab
3. Run the workflow
4. Check console for debug messages

## Quick Visual Guide

```
[Skip Excluded Emails] → [Debug Data] → [Send Reminder]
                           ↑
                    Add this Code node here
```

## Simple Test Version
If you want to start simple, use this minimal debug code:

```javascript
console.log("Employee name:", $json["name"]);
console.log("Employee email:", $json["work_email"]);
return $input.all();
```

Once you add this debug node and run the workflow, **tell me what you see in the console output** and I can help fix the specific data flow issue!