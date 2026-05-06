Add a task to Microsoft Planner. Argument: natural language including assignee (e.g. `/planner_add NAME: clean up Make automations`).

## Instructions

1. Parse `$ARGUMENTS` for: task title, assignee name, and optionally which board. Default to whatever board is configured for the assignee in `tools.md` § IDs Reference.

2. **Read `tools.md`** § Planner to get plan IDs, bucket IDs, and AAD IDs. Do not hardcode — always read from `tools.md`.

3. Resolve the assignee's AAD ID from `tools.md`. If not listed there, look up via Graph:
   ```bash
   m365 request --url "https://graph.microsoft.com/v1.0/users?\$filter=startswith(displayName,'<Name>')&\$select=displayName,id" --output json 2>/dev/null
   ```

4. Create the task in the "To do" bucket:
   ```python
   python3 << 'EOF'
   import subprocess, json
   body = json.dumps({
       "planId": "<plan-id>",
       "bucketId": "<todo-bucket-id>",
       "title": "<task title>",
       "assignments": {
           "<aad-id>": {"@odata.type": "#microsoft.graph.plannerAssignment", "orderHint": " !"}
       }
   })
   r = subprocess.run(['m365','request','--method','post','--url','https://graph.microsoft.com/v1.0/planner/tasks','--body',body,'--content-type','application/json','--output','json'], capture_output=True, text=True)
   print('OK' if r.returncode == 0 else r.stderr[:200])
   EOF
   ```

5. If extra detail was provided, update task details via PATCH (get `etag` from the create response first).

## Output

```
✓ Planner task created
  Board: <board name>
  Assigned to: <name>
  Title: <task title>
```

One confirmation block. No extra commentary.
