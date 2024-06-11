```bash
while IFS= read -r line; do curl -X POST -L plane.mermaid-gecko.ts.net/api/v1/workspaces/wwdc/projects/1ca8fbbe-0f75-4121-80ba-0765007df32d/issues/ --header 'X-API-Key:  plane_api_key_here' --header 'Content-Type: application/json' --data "{

  \"name\": \"$line\"

}" && sleep 1; done < ~/Code/wwdc24
```