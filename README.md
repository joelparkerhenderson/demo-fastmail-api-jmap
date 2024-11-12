# Demo Fastmail API JMAP

Demonstration of Fastmail.com API for JMAP, which is a JSON protocol for email accesss.


## Generate a Fastmail API security token

* [https://app.fastmail.com/settings/security/tokens/new](https://app.fastmail.com/settings/security/tokens/new)

* In the field "Name", fill in any name such as "demo-fastmail-jmap".

* In the area "Scope", check "Email Submission"

Example result:

```txt
fmu1-7c178287-1216c9163795fbefa9702c67571fcc32-0-172da3367b9fc53968fdb1e358e16747
```

For convenience, you can export it to your shell:

```sh
export TOKEN="fmu1-7c178287-1216c9163795fbefa9702c67571fcc32-0-172da3367b9fc53968fdb1e358e16747
"
```

## Connect via curl

The typical Fastmail API URL is:

* [https://api.fastmail.com/jmap/session](https://api.fastmail.com/jmap/session)

Verifiy you can connect via curl and your token:

```sh
curl "https://api.fastmail.com/jmap/session" \
 -H "Authorization: Bearer $TOKEN"
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
