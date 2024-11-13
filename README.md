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

A typical JMAP request uses these two HTTP headers:

```txt
Content-Type: application/json; charset=utf-8
Authorization: Bearer …
```

## JMAP using list

A JMAP request specifies what URN capabilities it is using, such as:

Example syntax:

```json
"using": [ 
  "urn:ietf:params:<group1>:<part1>"
  "urn:ietf:params:<group2>:<part2>"
]
```

Example real-world JMAP excerpt:

```json
"using": [ 
  "urn:ietf:params:jmap:core", 
  "urn:ietf:params:jmap:mail"
]
```

## JMAP method call

A JMAP method call uses a tuple:

* A String name of the method to call or of the response.

* A String[*] object containing named arguments for that method or response.
  
* A String method call id, which is an arbitrary string from the client to be echoed back with the responses emitted by that method call. The method call id is because a method may return 1 or more responses, because a method may make implicit calls to other methods; all responses initiated by this method call get the same method call id in the response. For simple needs, we prefer using a blank string. For production needs, we prefer using a ZID (i.e. 32 lowercase secure random hex digits) or UUID-4.

Example syntax:

```json
[
  "Type1/method1", 
  { 
    "arg1": "data1", 
    "arg2": "data2"
  }, 
  "arbitrary method call id" 
]
```

Example real-world JMAP excerpt:

```json
[
  "Mailbox/get", 
  {
    "accountId": "u5c18414e"
  },
  ""
]
```


## JMAP request example

A JMAP request combines the "using" list above, and the method calls.

Example syntax:

```json
{
  "using": [ 
    "urn:ietf:params:group1:part1", 
    "urn:ietf:params:group2:part2" 
  ],
  "methodCalls": [
    [ 
      "Type1/method1", 
      { 
        "arg1": "data1", 
        "arg2": "data2" 
      }, 
      "arbitrary method call id 1" 
    ],
    [ 
      "Type2/method2", 
      { 
        "arg3": "data3", 
        "arg4": "data4" 
      }, 
      "arbitrary method call id 2" 
    ]
  ]
}
```

Example real-world JMAP complete request JSON:

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
       "accountId": "u00000000"
      }, 
      "" 
    ]
  ]
}
```
