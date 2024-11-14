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
export token="fmu1-7c178287-1216c9163795fbefa9702c67571fcc32-0-172da3367b9fc53968fdb1e358e16747"
```


## Use curl to get your session account id

Source code:

* [bin/get-session](bin/get-session)


The typical Fastmail API JMAP session URL is:

* [https://api.fastmail.com/jmap/session](https://api.fastmail.com/jmap/session)

Run:

```sh
curl --silent \
  --header "Content-Type: application/json; charset=utf-8" \
  --header "Authorization: Bearer '"$token" \
  'https://api.fastmail.com/jmap/session'
```

The output is JSON, such as:

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

Look carefully at the "accounts" item, which lists your accounts. Each account item starts with an account id, which may start with the letter "u" (meaning user) then 8 hexadecimal lowercase digits, such as:

```txt
"accounts": {
  "u00000000": {
    …
  }
}
```

For convenience, you can export the account id to your shell, such as:

```sh
export account_id="u00000000"
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
Authorization: Bearer fmu1-7c178287-1216c9163795fbefa9702c67571fcc32-0-172da3367b9fc53968fdb1e358e16747
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

* A String method call id, which is an arbitrary string from the client to be echoed back with the responses emitted by that method call. 
Example:

```txt
[
  "Identity/get",
  {
    "accountId": "u00000000"
  },
  ""
]
```

The method call id is because a method may return 1 or more responses, because a method may make implicit calls to other methods; all responses initiated by this method call get the same method call id in the response. 

* If the request has one method call, then we prefer the method call id to be a blank string, because this helps make it clear that the method call id doesn't matter.
  
* If the request has multiple method calls, then we prefer the method call id to be the method call index number, such as "0", "1", "2", etc.

* If the request is for an important system, such as a multi-threaded application, or a production mail service, then we prefer the method call id to be a totally unique id. We prefer using a ZID i.e. hexadecimal 32-character lowercase secure random string, or using a UUID-4.


## Get your identity by using Identity/get

Source code:

* [bin/get-identity-list](bin/get-identity-list)

Get your identity list by using curl with JMAP HTTP headers (explained above) and JMAP JSON data:

```sh
curl \
--header "Content-Type: application/json; charset=utf-8" \
--header "Accept: application/json" \
--header "Authorization: Bearer $token" \
--request POST \
--data '
{
    "using": [
        "urn:ietf:params:jmap:core",
        "urn:ietf:params:jmap:submission"
    ],
    "methodCalls": [[
        "Identity/get", {
            "accountId": "'"$account_id"'"
        },
        ""
    ]]
}' \
'https://api.fastmail.com/jmap/api/'
```

The output is JSON, such as:

```json
{
  "latestClientVersion": "",
  "sessionState": "cyrus-0;p-6e2e3bf4ae;s-67362d7d426ad71b",
  "methodResponses": [
    [
      "Identity/get",
      {
        "list": [
          {
            "id": "00000000",
            "email": "alice.adams@example.com",
            "mayDelete": true,
            "displayName": "j@jph",
            "verificationCheckTime": "2021-06-09T23:09:41Z",
            "addBccOnSMTP": false,
            "replyTo": null,
            "name": "Alice Adams",
            "showInCompose": true,
            "server": "",
            "verificationState": "autoverified",
            "externalCredentialId": null,
            "bcc": null,
            "saveOnSMTP": false,
            "ssl": "starttls",
            "htmlSignature": "",
            "useForAutoReply": false,
            "enableExternalSMTP": false,
            "saveSentToMailboxId": "d12b886e-a5c0-20fa-9f16-081d8788b05b",
            "warnings": [],
            "isAutoConfigured": false,
            "textSignature": "",
            "port": 587
          }
        ],
        "accountId": "u00000000",
        "notFound": [],
        "state": ""
      },
      ""
    ]
  ]
}
```

The "list" item is an array of identities. It's possible to have multiple identities. Each identity has an "id" property.

For convenience, you can export your identity id to your shell, such as:

```sh
export identity_id="00000000"
```

Some of the Fastmail API JMAP methods will need the identity id parameter.


## Parse the identity list by using jq

If you want to parse the identity list JSON, here's an example:

```sh
get-mailbox-list |
jq -r ".methodResponses.[0].[1].list | .[] | [.id, .email] | @tsv"
```

What the command does:

* Use command `jq` to parse the JMAP JSON response.

* Select the inner array that is the identity list.

* For each identity, get its id and email address.

* Print the id and email as tab separated values.

Example output for a user who has 3 identities:

```tsv
00000000 alice.adams@example.com
00000001 alice@example.com
00000002 adams@example.com
```


## Get your mailbox list by using Mailbox/get

Source code:

* [bin/get-mailbox-list](bin/get-mailbox-list)

Combine the JMAP JSON "using" section and a JMAP JSON "methodCalls" section, to give you this complete JMAP JSON data to request the account's mailbox list:

HTTP headers:

```txt
Content-Type: application/json; charset=utf-8
Authorization: Bearer $token
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
       "accountId": "u00000000"
      },
      ""
    ]
  ]
}
```

Run:

```sh
curl \
--header "Content-Type: application/json; charset=utf-8" \
--header "Accept: application/json" \
--header "Authorization: Bearer '"$token" \
--request POST \
--data '
{
    "using": [
        "urn:ietf:params:jmap:core",
        "urn:ietf:params:jmap:mail"
    ],
    "methodCalls": [[
        "Mailbox/get",
        {
            "accountId": "'"$account_id"'"
        },
        ""
    ]]
}" \
'https://api.fastmail.com/jmap/api/'
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


## Get the Inbox mailbox by using Mailbox/query

Source code:

* [bin/get-mailbox-by-name](bin/get-mailbox-by-name)

To get a mailbox by name, use the JMAP method call "Mailbox/query" capability then a filter with a name parameter:

JMAP JSON data:

```json+shell
{
  "using": [
    "urn:ietf:params:jmap:core",
    "urn:ietf:params:jmap:mail"
  ],
  "methodCalls": [
    [
      "Mailbox/query",
      {
       "accountId": "'"$account_id"'",
        "filter": {
            "name": "Inbox"
        }
      },
      ""
    ]
  ]
}
```

Run:

```sh
curl \
--header "Content-Type: application/json; charset=utf-8" \
--header "Accept: application/json" \
--header "Authorization: Bearer '"$token" \
--request POST \
--data '
{
    "using": [
        "urn:ietf:params:jmap:core",
        "urn:ietf:params:jmap:mail"
    ],
    "methodCalls": [[
        "Mailbox/query",
        {
            "accountId": "'"$account_id"'",
            "filter": {
                "name": "$name"
            }
        },
        ""
    ]]
}" \
'https://api.fastmail.com/jmap/api/'
```

Example response that matches three mailboxes:

```json
{
  "methodResponses": [
    [
      "Mailbox/query",
      {
        "accountId": "u0000000",
        "ids": [
          "83616aa1-e433-bc13-2227-9400cb39f7ab",
          "3ee25744-47d2-06b3-de9d-f1e973612d66",
          "013cdf48-e6b0-2e79-dca9-baf70407ce40"
        ],
        "queryState": "000000",
        "total": 3,
        "filter": {
          "name": "Inbox"
        },
        "position": 0,
        "canCalculateChanges": true
      },
      ""
    ]
  ],
  "sessionState": "cyrus-0;p-ca43e86d48;s-6735173d254d20c1",
  "latestClientVersion": ""
}
```


## Send email using Email/set then EmailSubmission/set

Source code:

* [bin/send-email](bin/send-email)

The Fastmail API JMAP requires each email to be created as a draft, then sent.

Steps:

1. Use Email/set to create an email in our drafts folder.

2. Use EmailSubmission/set to send the email.

Run:

```sh
curl \
--header 'Content-Type: application/json; charset=utf-8' \
--header 'Accept: application/json' \
--header 'Authorization: Bearer '"$token" \
--request POST \
--data '
{
    "using": [
        "urn:ietf:params:jmap:core",
        "urn:ietf:params:jmap:mail",
        "urn:ietf:params:jmap:submission"
    ],
    "methodCalls": [
        [
            "Email/set", {
                "accountId": "'"$account_id"'",
                "create": {
                    "draft": {
                        "from": [{
                            "email": "'"$from_email"'"
                        }],
                        "to": [{
                            "email": "'"$to_email"'"
                        }],
                        "subject": "'"$subject"'",
                        "mailboxIds": {
                            "'"$mailbox_id"'": true
                        },
                        "keywords": {
                            "$draft": true
                        },
                        "textBody": [{
                            "partId": "body",
                            "type": "text/plain"
                        }],
                        "bodyValues": {
                            "body": {
                                "charset": "utf-8",
                                "value": "'"$body_value"'"
                            }
                        }
                    }
                }
            },
            "0"
        ],
        [
            "EmailSubmission/set", {
                "accountId": "'"$account_id"'",
                "onSuccessDestroyEmail": ["#sendIt"],
                "create": {
                    "sendIt": {
                        "emailId": "#draft",
                        "identityId": "'"$identity_id"'"
                    }
                }
            },
            "1"
        ]
    ]
}' \
'https://api.fastmail.com/jmap/api/'
```

The output is JSON, such as:

```json
{
    "methodResponses": [
        [
            "Email/set",
            {
                "accountId": "u00000000",
                "notUpdated": null,
                "destroyed": null,
                "updated": null,
                "created": {
                    "draft": {
                        "id": "Me4672cc07651aaf91ab4cee6",
                        "blobId": "Ge4672cc07651aaf91ab4cee6f275cea0d26ce481",
                        "threadId": "T7594f5217d38a1eb",
                        "size": 285
                    }
                },
                "notDestroyed": null,
                "oldState": "1102978",
                "newState": "1102980",
                "notCreated": null
            },
            "0"
        ],
        [
            "EmailSubmission/set",
            {
                "oldState": "1102956",
                "newState": "1102981",
                "notCreated": null,
                "accountId": "u00000000",
                "notDestroyed": null,
                "destroyed": null,
                "notUpdated": null,
                "created": {
                    "sendIt": {
                        "undoStatus": "final",
                        "id": "S3650",
                        "sendAt": "2024-11-14T19:14:25Z"
                    }
                },
                "updated": null
            },
            "1"
        ],
        [
            "Email/set",
            {
                "notCreated": null,
                "oldState": "1102980",
                "newState": "1102982",
                "created": null,
                "updated": null,
                "notUpdated": null,
                "destroyed": [
                    "Me4672cc07651aaf91ab4cee6"
                ],
                "notDestroyed": null,
                "accountId": "u00000000"
            },
            "1"
        ]
    ],
    "sessionState": "cyrus-0;p-6ead906769;s-67364bcfd71fd239",
    "latestClientVersion": ""
}
```
