#macOS 

To make notarizing mac apps faster and simpler
# What is notary?
For mac developers distributing software outside of app store
Using altool to interact with Notary Service.
If you're notarizing with xcode only, your workflow isn't changing.

Submit your software before distribution
Apple checks for malicious components
Helping protect customers since macOS 10.14
Now faster and simpler to use
[[all about notarization]]

1.  Build software
2.  send to notary API
3.  service runs analysis
4.  Publishes ticket ot be retrieved by macs who run the software
5.  Return results

Completed within 15 minutes for 98% of submissions.

Provide app for users to download.  We care about...
# Introducing notarytool
Notarytool provides a 1 line command
```bash
notarytool submit path/to/submission.zip --wait
```

Notarizing without altool's appstore support
faster and easier
protects customer from malware

As reliable as possible
High availability
Dedicated new service backend

4x faster uploads for median-sized submissions
Simpler client.
# Using notarytool

```bash
// with altool
xcrun altool --notarize-app -f path/to/submission.zip 
    --primary-bundle-id "$BUNDLE_ID"
    --apiKey "$KEY_ID" --apiIssuer "$ISSUER"
while true; do
  INFO_OUT=$(2>&1 xcrun altool --notarization-info "$SUBMISSION_ID" -u "$USER" 
      --apiKey "$KEY_ID" --apiIssuer "$ISSUER")
  STATUS=$(echo "$INFO_OUT" | grep "Status:" | sed -Ee "s|.*: (.*)$|\1|" )
  if [[ "$STATUS" != "in progress" ]]; then 
    break
  fi
  sleep 30
done
```

Now with notarytool just use `--wait`.

```bash
notarytool log $SUBMISSION_ID notary-log.json
```

Conveniently integrated into the client tool

Webhook notifications
```bash
notarytool submit path/to/submission.zip --webhook https://www.example.com
```

Ad dnotarization into your CI workflow
Using notarytool
We look forward to receiving your feedback

Now distributed with xcode
`altool` is now deprecated for use with Notary.  Only for appstore use.

# Wrap up
faster and simpler workflow
`notarytool` is available now

