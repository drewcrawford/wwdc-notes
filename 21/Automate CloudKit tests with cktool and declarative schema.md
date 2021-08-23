# Testing challenges
* Schema consistency
* Cloud sample data

Solutions
* Schema language for schema consistency
* cktool for sample data and automation


# New command line tool
## Installing
with xc13.  Once you install xc13, you can start using from terminal right away.  Invoked with `xcrun`.

Create records, query records, import/export, etc.  

## Authorization
* Required by server
* Administrative tasks
* Data access

Token types
* Management token
	* Scope: developer account
	* Purpose: cloudKit configuration
* User token
	* Scope: Developer account
	* Purpose: Public and private databas data

[[Meet CloudKit Console]]

```bash
xcrun cktool save-token --type management

xcrun cktool save-token --type user

xcrun cktool get-teams
```

```bash
xcrun cktool export-schema \
  --team-id XYZ1234567 \
  --container-id iCloud.com.WWDC21.Example \
  --environment development \
  --output-file schema.ckdb
```

## Composing a script
* Delete data
* Upload scehma declaration
* Create test data


# CloudKit schema language

```
DEFINE SCHEMA
     RECORD TYPE Book (
        "___createTime" TIMESTAMP,
        "___createdBy"  REFERENCE,
        "___etag"       STRING,
        "___modTime"    TIMESTAMP,
        "___modifiedBy" REFERENCE,
        "___recordID"   REFERENCE QUERYABLE,
        description     STRING,
        pageCount       INT64,
        publishedOn     TIMESTAMP,
        reviewStatus    INT64,
        // A single-line comment, for humans
        title           STRING QUERYABLE,
        GRANT WRITE TO "_creator",
        GRANT CREATE TO "_icloud",
        GRANT READ TO "_world"
     );
```

Each record type has multiple fields, and each field has a name and datatype.

`___` are system fields
others are custom fields

Comments are ignored

Need to ensure CK still understands fields used by earlier version so fyour app.  Can make destructive changes in development, not in production

|                        | Development | Production |
|------------------------|-------------|------------|
| Add record types       | Yes         | Yes        |
| Add custom fields      | Yes         | Yes        |
| remove record types    | Yes         | No         |
| Remove custom fields   | Yes         | No         |
| Add and remove indexes | Yes         | Yes        |
| Modify security tokens | Yes         | Yes        |
# Wrap up
* Authorize cktool and explore
* Add schema to your project
* Script your setup steps
* Add a test pre-action in Xcode

* https://developer.apple.com/icloud/cloudkit/automating/
* https://developer.apple.com/documentation/cloudkit/integrating_a_text-based_schema_into_your_workflow
* https://developer.apple.com/documentation/cloudkit

