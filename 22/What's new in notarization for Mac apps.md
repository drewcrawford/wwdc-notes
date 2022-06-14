NotaryTool CLI.  
# important deadlines
* `notarytool` replaces `altool` for notarization
* Xcode 14 replaces Xcode 13 for notarization
* Migrate to these new workflows by Fall 2023

[[Faster and simpler notarization for Mac apps]]

# xcode changes
4x upload times for median-sized submissions

Besides updating, you don't have to change project settings etc.
# Notary REST API
Use a machine-readable format, JSON
Upload from anywhere with an internet connection
Retrieve submission history or past details

 e.g. linux-based CI.  Now you can automate that process via REST.
 
 Authenticate via JWT.  For more details, see docs.
 
 ```python
# Upload file for notarization

def upload_file(token, filepath, sha256):
    data = { "sha256": sha256, "submissionName": os.path.basename(filepath) }
    resp = requests.post(
       "https://appstoreconnect.apple.com/notary/v2/submissions",
        json=data,
        headers={"Authorization": "Bearer " + token})

    output = resp.json()
    aws_info = output["data"]["attributes"]
    submission_id = output["data"]["id"] 

    client = boto3.client(
        "s3",  
        aws_access_key_id=aws_info["awsAccessKeyId"],
        aws_secret_access_key=aws_info["awsSecretAccessKey"],
        aws_session_token=aws_info["awsSessionToken"])
    client.upload_file(filepath, aws_info["bucket"], aws_info["object"])
```

```python
# Wait for completion

def watch_upload(submission_id, token):
    while True:
        resp = requests.get(
            "https://appstoreconnect.apple.com/notary/v2/submissions/" + submission_id, 
            headers={"Authorization": "Bearer " + token})

        output = resp.json()
        current_status = output["data"]["attributes"]["status"]
        
        if current_status != "In Progress":
            return current_status # For example: Accepted or Invalid

        time.sleep(30) # Allow time for submission to progress
```

# Webhooks
details can be found in the docs

 * igrate to xcode14 or notarytool by fall 2023
 * check otu the REST API today!