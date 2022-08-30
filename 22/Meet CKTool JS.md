#CloudKit 

# Introduction
* store your app's data in iCloud
* Sync data across devices and on the web

* client access
	* CloudKit framework
	* CloudKit JS
* Automation and tooling
	* cktool CLI
	* CKTool JS

* similar to the cktool CLI tool distributed with xcode
* manage containers and manipulate user data
* used by CloudKit console
Manage CloudKit contianer
perform schema operations
* reset
* import
* export
* validate

* read
	* fetch a specific record
	* query records
* Write
	* create
	* update
Typescript type definitions
* compile-time checking
* code completion

Targets
* node.js
* web browsers

Isntallable as npm packages
Build pipeline
tree-shaking
bundling
release history

`@apple/cktool.*`.  Main package is `@apple/cktool.database`.  Also need `cktool.target.nodejs` or cktool.target.browser.  

databse pulls in `cktool.core`, `.api.base`, and `.api.database`.
* authorization required
* token based auth
	* management token
	* user token
* obtain tokens from CloudKit console
* management tokens
	* enable contianer management
	* schema changes and validation
	* resetting a schema

User tokens
* enabling reading and writing data

[[Automate CloudKit tests with cktool and declarative schema]]

# Configure CKTool JS
Record type of countries.  Name, iso codes, etc.  Relate coins for each country.  

Schema
* record types
* relationships
evolves over time

containers
* data stored in iCloud containers
* each has as chema
* uniquely identified

environments
* development
* production

CKToolJS - gather information os that it knows how to work with the right container and your script is uahuthorized etc.
* team ID and contianer ID
* management tokne or user token
* environment

```js
// Create security object and setup default args

const { CKEnvironment } = require("@apple/cktool.database");

const security = {
    "ManagementTokenAuth": "<YOUR_MANAGEMENT_TOKEN>",
    "UserTokenAuth": "<YOUR_USER_TOKEN>"
};

const defaultArgs = {
    "teamId": "<YOUR_TEAM_ID>",
    "containerId": "<YOUR_CONTAINER_ID>",
    "environment": CKEnvironment.DEVELOPMENT
};
```
```js
// Create configuration and API objects

const { createConfiguration } = require("@apple/cktool.target.nodejs");
const { PromisesApi } = require("@apple/cktool.database");

const configuration = createConfiguration();
const api = new PromisesApi({
    "configuration": configuration,
    "security": security
});
```

`createConfiguration` is platform-specific between nodejs and browser.


# Manage your schema
Separate out coin composition as an independent component entity.
I identified two record types.  Coins.  Components.  Stores a reference to a coin described, etc.
schema file
* used for defining and updating your container's schema
* convention is to use .ckdb file extension

Apply schema from CKTool JS
Reset development schema to match production
* pass `defaultARgs` to api.resetToProduction()
* async
* returns a promise

export/import your container schema.  container=>export schema.  Upload a schema by *importing* to the container.
```js
// Create a function to apply a schema

const { File } = require("@apple/cktool.target.nodejs");
const fs = require("fs/promises");
const path = require("path");

const importMySchema = async () => {
    const schemaPath = "<YOUR_SCHEMA_FILE>.ckdb";
    const buffer = await fs.readFile(schemaPath);
    const file = new File([buffer], schemaPath);
    await api.importSchema({ ...defaultArgs, "file": file });
}

// Chain the calls
api.resetToProduction(defaultArgs)
  .then(() => importMySchema());
```



# Read and write user data
* strict type checking
* range checks
* coercion functions for large numbers taht can't be represented in js

```js
// Create fields with factory functions.

const {
    makeRecordFieldValue
} = require("@apple/cktool.database");

const value = makeRecordFieldValue.int64(2007);
```

If a factory function can't create a record from the value passed in, it throws ane xception
```js
// Create a database arguments object.

const {
    CKDatabaseType, CKEnvironment
} = require("@apple/cktool.database");

const databaseArgs = {
    "containerID": "<YOUR_CONTAINER_ID>",
    "environment": CKEnvironment.DEVELOPMENT,
    "databaseType": CKDatabaseType.PRIVATE,
    "zoneName": "_defaultZone"
};
```


```js
// Define helper function for querying records

const { CKDBQueryFilterType } = require("@apple/cktool.database");
const countryQueryRecordForCountryCode3 = async (countryCode3) => {
    const response = await api.queryRecords({
        ...databaseArgs,
        "body": {
            "query": {
                "recordType": "Countries",
                "filters": [{
                    "fieldName": "isoCode3",
                    "fieldValue": makeRecordFieldValue.string(countryCode3),
                    "type": CKDBQueryFilterType.EQUALS
                }]
            }
        }
    });
    return response.result.records[0];
}
```
To convert raw values, we use a hepler such as
```js
// Define a helper function for creating field values

const {
    makeRecordFieldValue, CKDBRecordReferenceAction
} = require("@apple/cktool.database");

const makeCoinFieldValues = ({ countryRecordName, issueYear, nominalValue }) => ({
    "country": makeRecordFieldValue.reference({
        recordName: countryRecordName,
        action: CKDBRecordReferenceAction.DELETE_SELF
    }),
    "issueYear": makeRecordFieldValue.int64(issueYear),
    "nominalValue": makeRecordFieldValue.double(nominalValue)
});
```

```js
// Define helper method for creating coins

const coinCreateRecord = async (fields) => {
    const response = await api.createRecord({
        ...databaseArgs,
        "body": {
            "recordType": "Coins",
            "fields": fields
        },
    });
    return response.result.record;
}
```
need to fetch a country record first
```js
// Call coin creation method with field values

const countryRecord = await countryQueryRecordForCountryCode3("USA");

const coinRecord1 = await coinCreateRecord(
    makeCoinFieldValues({
        "countryRecordName": countryRecord.recordName,
        "issueYear": 2007,
        "nominalValue": 0.10
    })
);
```

```js
// Define helper method for updating coins.
// Note that recordChangeTag is required

const coinUpdate =
    async (recordName, recordChangeTag, fields) => {
        const response = await api.updateRecord({
            ...databaseArgs,
            "recordName": recordName,
            "body": {
                "recordType": "Coins",
                "recordChangeTag": recordChangeTag,
                "fields": fields
            }
        });
        return response.result.record;
    }
```

```js
// Call coin updating method with field values.
// Note that the recordChangeTag of the record
// to update is passed to the coin update function.

const countryRecord = await countryQueryRecordForCountryCode3("USA");
const updatedCoinRecord1 = await coinUpdate(
    coinRecord1.recordName,
    coinRecord1.recordChangeTag,
    makeCoinFieldValues({
        "countryRecordName": countryRecord.recordName,
        "issueYear": 2010,
        "nominalValue": 0.10
    });
);
```

```js
// Deleting a record

await api.deleteRecord({
    ...databaseArgs,
   "recordName": coinRecord1.recordName
});
```

# Wrap up
* configure
* manage your schemas

See github, etc.

* https://developer.apple.com/forums/tags/wwdc2022-10116
* https://developer.apple.com/forums/create/question?&tag1=321030&tag2=431030
* https://github.com/apple/sample-cloudkit-tooling
* https://developer.apple.com/documentation/cktooljs
* https://developer.apple.com/documentation/cloudkit/integrating_a_text-based_schema_into_your_workflow

