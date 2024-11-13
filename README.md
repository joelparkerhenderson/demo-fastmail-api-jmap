# Demo Fastmail API JMAP

Demonstration of Fastmail.com Application Programming Interface (API) for JSON Meta Application Protocol Specification (JMAP)

* JMAP specification: [https://jmap.io/spec-core.html](https://jmap.io/spec-core.html)

## Generate a Fastmail API security token

* [https://app.fastmail.com/settings/security/tokens/new](https://app.fastmail.com/settings/security/tokens/new)

* In the field "Name", fill in any name such as "demo-fastmail-jmap".

* In the area "Scope", check "Email Submission"

Example result:

```txt
fmu1-7c178287-1216c9163795fbefa9702c67571fcc32-0-172da3367b9fc53968fdb1e358e16747
```

For convenience, you can export the API token to your shell:

```sh
export TOKEN="fmu1-7c178287-1216c9163795fbefa9702c67571fcc32-0-172da3367b9fc53968fdb1e358e16747
"
```

## Use curl to get a session account id

The typical Fastmail API JMAP session URL is:

* [https://api.fastmail.com/jmap/session](https://api.fastmail.com/jmap/session)

Verify you can connect via curl and your token:

```sh
curl --silent \
  --header "Content-Type: application/json; charset=utf-8" \
  --header "Authorization: Bearer $TOKEN" \
  "https://api.fastmail.com/jmap/session"
```

The output is JSON and looks like this:

```json
{
  "uploadUrl": "https://api.fastmail.com/jmap/upload/{accountId}/",
  "capabilities": {
    "urn:ietf:params:jmap:submission": {},
    "urn:ietf:params:jmap:mail": {},
    "urn:ietf:params:jmap:core": {
      "maxObjectsInGet": 4096,
      "maxSizeUpload": 250000000,
      "maxCallsInRequest": 50,
      "maxSizeRequest": 10000000,
      "maxObjectsInSet": 4096,
      "maxConcurrentUpload": 10,
      "maxConcurrentRequests": 10,
      "collationAlgorithms": [
        "i;ascii-numeric",
        "i;ascii-casemap",
        "i;octet"
      ]
    }
  },
  "primaryAccounts": {
    "urn:ietf:params:jmap:submission": "u00000000",
    "urn:ietf:params:jmap:mail": "u00000000",
    "urn:ietf:params:jmap:core": "u00000000"
  },
  "state": "cyrus-0;p-76d3225573;s-6733cc77c8e1cd52",
  "downloadUrl": "https://www.fastmailusercontent.com/jmap/download/{accountId}/{blobId}/{name}?type={type}",
  "eventSourceUrl": "https://api.fastmail.com/jmap/event/",
  "accounts": {
    "u00000000": {
      "name": "example@fastmail.com",
      "accountCapabilities": {
        "urn:ietf:params:jmap:core": {},
        "urn:ietf:params:jmap:submission": {
          "maxDelayedSend": 44236800,
          "submissionExtensions": {}
        },
        "urn:ietf:params:jmap:mail": {
          "maxSizeMailboxName": 490,
          "maxMailboxDepth": null,
          "mayCreateTopLevelMailbox": true,
          "maxSizeAttachmentsPerEmail": 50000000,
          "maxMailboxesPerEmail": 1000,
          "emailQuerySortOptions": [
            "receivedAt",
            "from",
            "to",
            "subject",
            "size",
            "header.x-spam-score"
          ]
        }
      },
      "isReadOnly": false,
      "isPersonal": true
    }
  },
  "apiUrl": "https://api.fastmail.com/jmap/api/",
  "username": "example@fastmail.com"
```

Look carefully at the "accounts" item, which lists your account id, which may start with the letter "u" for user then 8 hexadecimal lowercase digits, such as:

```json
  "accounts": {
    "u00000000": {
```      

For convenience, you can export the account id to your shell, such as:

```sh
export ACCOUNT=u00000000
```

Many of the Fastmail API JMAP methods will need the account id parameter.


## What is a typical JMAP request?

A simple JMAP request looks like this:

```txt
Content-Type: application/json; charset=utf-8
Authorization: Bearer …

{
  "using": [ 
    "urn:ietf:params:jmap:core", 
    "urn:ietf:params:jmap:mail" 
  ],
  "methodCalls": [
    [ 
      "Mailbox/get", 
      { 
       "accountId": "u00000000"
      }, 
      "" 
    ]
  ]
}
```

The request is explained in more detail in the next few sections below.


## Provide JMAP HTTP headers

A simple JMAP request uses HTTP headers such as:

```txt
Content-Type: application/json; charset=utf-8
Authorization: Bearer f8621aaabd847d2c5e948d8869b5b628
```


## Format a JMAP "using" list

A JMAP request specifies what URN parameter capabilities it is using, such as:

```json
"using": [ 
  "urn:ietf:params:jmap:core", 
  "urn:ietf:params:jmap:mail"
]
```


## Create a JMAP method call

A JMAP method call uses a tuple:

* A String name of the method to call or of the response.

* A String[*] object containing named arguments for that method or response.
  
* A String method call id, which is an arbitrary string from the client to be echoed back with the responses emitted by that method call. The method call id is because a method may return 1 or more responses, because a method may make implicit calls to other methods; all responses initiated by this method call get the same method call id in the response. For simple needs, we prefer using a blank string. For production needs, we prefer using a ZID (i.e. hexadecimal lowercase secure random 32-character string) or UUID-4.

Example:

```json
[
  "Mailbox/get", 
  {
    "accountId": "u5c18414e"
  },
  ""
]
```

Example with more args and an arbitrary method call id:

```json
[
  "Resource1/method1", 
  { 
    "arg1": "value1", 
    "arg2": "value2"
  }, 
  "b602822814f46a6013c7f0dce6fa028f" 
]
```


## Get the mailbox list by using curl

Combine the JMAP JSON "using" section and a JMAP JSON "methodCalls" section, to give you this complete JMAP JSON data to request the account's mailbox list:

HTTP headers:

```txt
Content-Type: application/json; charset=utf-8
Authorization: Bearer $TOKEN
```

JMAP JSON data:

```json
{
  "using": [ 
    "urn:ietf:params:jmap:core", 
    "urn:ietf:params:jmap:mail" 
  ],
  "methodCalls": [
    [ 
      "Mailbox/get", 
      { 
       "accountId": "$ACCOUNT"
      }, 
      "" 
    ]
  ]
}
```

Complete curl example:

```sh
curl \
--header "Content-Type: application/json; charset=utf-8" \
--header "Authorization: Bearer $TOKEN" \
--request POST \
--data "
{
    \"using\": [
        \"urn:ietf:params:jmap:core\", 
        \"urn:ietf:params:jmap:mail\"
    ],
    \"methodCalls\": [[
        \"Mailbox/get\", 
        {
            \"accountId\": \"$ACCOUNT\"
        },
        \"\"
    ]]
}" \
"https://api.fastmail.com/jmap/api/"
```


## Parse the mailbox list by using jq

If you want to parse the mailbox list JSON, here's an example:

```sh
get-mailbox-list |
jq -r ".methodResponses.[0].[1].list | .[] | [.id, .name] | @tsv"
```

What the command does:

* Use command `jq` to parse the JMAP JSON response.

* Select the inner array that is the mailbox list.

* For each mailbox, get its id and name.

* Print the id and name as tab separated values.

Example output:

```tsv
8c543068-9a50-30bf-8cbb-f77ae411f743 Inbox
3edd8e81-a814-947c-4e22-4c4e9c0b30b1 Archive
98a10f38-82c9-8b3a-75f5-085915113509 Drafts
c7d72bfa-0c4e-3ad4-0d9d-2aeee366faaf Sent
76561826-af17-fca9-c96d-764517ebf9e9 Spam
f74e957b-2c74-1f89-e354-0f9bba0acdea Trash
```
