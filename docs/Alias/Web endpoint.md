<!--
SPDX-FileCopyrightText: 2021 Andre 'Staltz' Medeiros

SPDX-License-Identifier: CC-BY-4.0
-->

## Web endpoint

Once an alias is [registered](Registration.md), it enables any web user to visit a web endpoint on the room server dedicated to that alias, for the purpose of telling the visitor what SSB ID does the alias resolve to, and with instructions on how to install an SSB app if the visitor doesn't have it yet.

The goal of this endpoint is to help any SSB user *locate and identify* the alias' owner by resolving the alias to: (1) the room's [multiserver address](https://github.com/ssb-js/multiserver), (2) the owner's SSB ID, and (3) a cryptographic signature that proves the owner associated themselves with that alias. This web endpoint is valuable to onboard new SSB users being invited by an [internal user](../Stakeholders/Internal%20user.md).

**Prior art:** This endpoint should be in many ways similar to the [Telegram](https://telegram.org/) `https://t.me/example` service for the username `@example`, also capable of redirecting the web visitor to a scheme `tg` URI `tg://resolve?domain=example`, which Telegram apps know how to parse and open the target user's profile screen.

### Specification

This specification does not apply if the [privacy mode](../Setup/Privacy%20modes.md) is *Restricted*. This web endpoint is available only if the privacy mode is *Open* or *Community*.

If the alias `${alias}` is registered at the room `${roomHost}` for a certain `${userId}`, then the room's HTTP endpoint reserved for the alias **SHOULD** be the wildcard subdomain URL `https://${alias}.${roomHost}` but it **MAY** be `https://${roomHost}/${alias}`.

The HTML response then:

- **MAY** inform users how to install an SSB app that can correctly consume room aliases
- **SHOULD** render a "Connect with me" button linking to an SSB URI (see below)
- The page **MAY** automatically redirect (when the browser supports it) to an SSB URI (see below)
- The alias SSB URI **MUST** be `ssb:experimental?action=consume-alias&alias=${alias}&userId=${userId}&signature=${signature}&roomId=${roomId}&multiserverAddress=${roomMsAddr}`, in other words there are 6 query components:
  - `action=consume-alias`, a constant string to identify the purpose of this URI
  - `alias=${alias}`, the [alias string](Alias%20string.md)
  - `userId=${userId}`, the SSB ID of the alias's owner
  - `roomId=${roomId}`, the room's SSB ID
  - `signature=${signature}`, the alias's owner signature as described in the [alias database](Alias%20database.md)
  - `multiserverAddress=${roomMsAddr}`, the room's [multiserver address](https://github.com/ssb-js/multiserver) using percent encoding for URIs

As an additional endpoint for programmatic purposes, if the query parameter `encoding=json` is added to the alias endpoint (for illustration: `https://${alias}.${roomHost}?encoding=json`), then, in successful responses, the JSON body **MUST** conform to the following schema:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "https://github.com/ssb-ngi-pointer/rooms2#alias-json-endpoint-success",
  "type": "object",
  "properties": {
    "status": {
      "title": "Response status tag",
      "description": "Indicates the completion status of this response",
      "type": "string",
      "pattern": "^(successful)$"
    },
    "multiserverAddress": {
      "title": "Multiserver address",
      "description": "Should conform to https://github.com/ssbc/multiserver-address",
      "type": "string"
    },
    "roomId": {
      "title": "Room ID",
      "description": "SSB ID for the room server",
      "type": "string"
    },
    "userId": {
      "title": "User ID",
      "description": "SSB ID for the user owning the alias",
      "type": "string"
    },
    "alias": {
      "title": "Alias",
      "description": "A domain 'label' as defined in RFC 1035",
      "type": "string"
    },
    "signature": {
      "title": "Signature",
      "description": "Cryptographic signature covering the roomId, the userId, and the alias",
      "type": "string"
    }
  },
  "required": [
    "status",
    "multiserverAddress",
    "roomId",
    "userId",
    "alias",
    "signature"
  ]
}
```

In failed responses, the JSON body **MUST** conform to the following schema:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "https://github.com/ssb-ngi-pointer/rooms2#alias-json-endpoint-error",
  "type": "object",
  "properties": {
    "status": {
      "title": "Response status tag",
      "description": "Indicates the completion status of this response",
      "type": "string"
    },
    "error": {
      "title": "Response error",
      "description": "Describes the specific error that occurred",
      "type": "string"
    }
  },
  "required": [
    "status",
    "error"
  ]
}
```

### Example

Suppose the alias is `bob`, registered for the user ID `@yVQxFxzeRQ13DQ813hf8G20U5z5I/nkNDliKeSs/IpU=.ed25519` at the room with host name `scuttlebutt.eu`. Then the alias endpoint `https://bob.scuttlebutt.eu` responds with HTML containing the following SSB URI:

[ssb:experimental?action=consume-alias&multiserverAddress=net%3Ascuttlebutt.eu%3A8008~shs%3Azz%2Bn7zuFc4wofIgKeEpXgB%2B%2FXQZB43Xj2rrWyD0QM2M%3D&alias=bob&roomId=%40zz%2Bn7zuFc4wofIgKeEpXgB%2B%2FXQZB43Xj2rrWyD0QM2M%3D.ed25519&userId=%40yVQxFxzeRQ13DQ813hf8G20U5z5I%2FnkNDliKeSs%2FIpU%3D.ed25519&signature=EiEgn%2Fh2lKoaz28ggKBod6havJNKapRKCmXQ%2Ft%2F4KS1gY4T6zPXWhw6kTaglt8vDJZW%2BjJRJvfB4Rryhl0njCg%3D%3D.sig.ed25519](ssb:experimental?action=consume-alias&multiserverAddress=net%3Ascuttlebutt.eu%3A8008~shs%3Azz%2Bn7zuFc4wofIgKeEpXgB%2B%2FXQZB43Xj2rrWyD0QM2M%3D&alias=bob&roomId=%40zz%2Bn7zuFc4wofIgKeEpXgB%2B%2FXQZB43Xj2rrWyD0QM2M%3D.ed25519&userId=%40yVQxFxzeRQ13DQ813hf8G20U5z5I%2FnkNDliKeSs%2FIpU%3D.ed25519&signature=EiEgn%2Fh2lKoaz28ggKBod6havJNKapRKCmXQ%2Ft%2F4KS1gY4T6zPXWhw6kTaglt8vDJZW%2BjJRJvfB4Rryhl0njCg%3D%3D.sig.ed25519)

The JSON endpoint `https://bob.scuttlebutt.eu/?encoding=json` would respond with the following JSON:

```json
{
  "status": "successful",
  "multiserverAddress": "net:scuttlebutt.eu:8008~shs:zz+n7zuFc4wofIgKeEpXgB+/XQZB43Xj2rrWyD0QM2M=",
  "roomId": "@zz+n7zuFc4wofIgKeEpXgB+/XQZB43Xj2rrWyD0QM2M=.ed25519",
  "userId": "@yVQxFxzeRQ13DQ813hf8G20U5z5I/nkNDliKeSs/IpU=.ed25519",
  "alias": "bob",
  "signature": "EiEgn/h2lKoaz28ggKBod6havJNKapRKCmXQ/t/4KS1gY4T6zPXWhw6kTaglt8vDJZW+jJRJvfB4Rryhl0njCg==.sig.ed25519"
}
```

### Security considerations

#### Malicious web visitor

A web visitor, either human or bot, could attempt brute force visiting all possible alias endpoints, in order to build a dataset of all SSB IDs and claimed aliases gathered at this room, potentially tracking profiles of these SSB IDs. Malicious web visitors can also attempt to connect with these target IDs as victims, and may use social engineering or impersonation tactics during [tunneled authentication](../Participation/Tunneled%20authentication.md).

#### Malicious [room admin](../Stakeholders/Room%20admin.md)

The room admin could tamper with the [alias database](Alias%20database.md) and provide fake information on this web endpoint, e.g. that a certain alias was claimed by a certain users. Although the [database signature](Alias%20database.md) exists to prevent this type of tampering, it is only verified when performing [alias consumption](Alias%20consumption.md). For web visitors who only want to know which SSB ID corresponds to an alias, and only that, these visitors must trust the room administrator, who could provide inauthentic information.
