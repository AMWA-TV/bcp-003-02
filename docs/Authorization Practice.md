# AMWA BCP-003-02: Authorization in NMOS Systems
{:.no_toc}

* A markdown unordered list which will be replaced with the ToC, excluding the "Contents header" from above
{:toc}

Please see the IS-10 Specification at [NMOS Authorization][IS-10] for further details regarding implementing
authorization with the NMOS suite of APIs.

## Use of Normative Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in [RFC 2119][RFC-2119].

## Scope

The [AMWA IS-10 Specification](https://specs.amwa.tv/is-10) describes how to implement a general OAuth
2.0 authorization system, in line with current best practices. This document outlines additional requirements placed
upon Resource Servers and Clients which implement or interact with specific NMOS Interface Specifications.

## Authorization Grant Constraints

As identified in IS-10, Authorization Clients are expected to use the Authorization Code OAuth grant type unless they
are a confidential client which needs to peform machine to machine operations with no user interaction. Client
interactions which MAY use the Client Credentials grant are detailed below.

* IS-04 Node interactions with an IS-04 registry using the 'registration' scope
* IS-07 WebSocket Receivers which are required to present a token to an IS-07 WebSocket Sender using the 'events' scope

The above list will be added to in the future if additional use cases for client credentials are identified.

Alternative uses of the Client Credentials grant MUST be arranged on a deployment specific basis and MUST NOT form the
default mode of operation.

Authorization Servers SHOULD be configured to enforce the scopes which are permitted to use Client Credentials during
the dynamic client registration or token exchange process where supported.

## NMOS APIs

### IS-04 - Discovery and Registration

#### Node Manifest Href Authorization

Nodes which support IS-04 and IS-05 and operate in authorized mode SHOULD use the IS-05 `transportfile` paths matching
their Sender IDs when publishing IS-04 `manifest_href` Sender attributes.

Control systems which identify an IS-05 path in the `manifest_href` (identified via the presence of the base path
`/x-nmos/connection`) MUST determine whether an authorization token is required to access this file via the IS-04 Device
controls `authorization` attribute. Where the Node does not use an IS-05 path for the `manifest_href` the control system
SHOULD assume that authorization is not required for this path unless this requirement and relevant claims have been
established out of band.

#### Registry Client Authorization

When registering resources with an instance of an [IS-04 Registry][], the Registry MUST capture the Client ID of the
client performing the initial 'node' resource type registration. The Client ID can be found within the `client_id` claim
of the valid JWT access token used in the HTTP request. Where the `client_id` is not specified the same identifier can
be found in the `azp` claim.

Subsequent requests to register, modify or delete resources which belong to Node ID(s) which were registered by this
client MUST be validated to ensure that the Client ID used matches the `client_id` or `azp` claim in the JWT access
token used in the HTTP request making the modifications. This is to ensure that clients do not, maliciously or
incorrectly, alter resources belonging to other Nodes. Unsuccessful validation of a matching Client ID MUST result in
the Resource server rejecting the request with a '403' HTTP response code and a `WWW-Authenticate` header using the
error value of `insufficient_scope` as specified in [RFC 6750][RFC-6750]. All IS-04 resources can be related back to a
single Node ID by following the [IS-04 Referential Integrity][] guide.

#### Registry WebSocket Authorization

When creating an [IS-04 Registry][] WebSocket subscription, whether by using a POST to the `/subscriptions` resource
or by using an `Upgrade` HTTP header, the JSON Web Token used MUST be validated to ensure it contains a write claim
matching the subscriptions path, for example:

```
"x-nmos-query": {
  "write": ["subscriptions"],
  "read": ["*"]
}
```

When connecting to an [IS-04 Registry][] WebSocket subscription, the JSON Web Token used MUST be validated to ensure it
contains a read claim matching all data which may be returned according to the subscription's `resource_path`
configuration. Note that the `resource_path` includes a leading slash where token claims do not. The `ws_href` path
MUST NOT be validated against the token claims.

For example, access to the following subscription would require token claims as follows:

```
{
  "max_update_rate_ms": 100,
  "resource_path": "/receivers",
  "params": {},
  "persist": false,
  "secure": false,
  "ws_href": "wss://172.29.80.52:8870/ws/?uid=6a52dbd5-a737-4c4e-823f-909ade8f8bf4",
  "id": "6a52dbd5-a737-4c4e-823f-909ade8f8bf4"
}
```

```
"x-nmos-query": {
  "read": ["receivers/*"]
}
```

### IS-05 - Device Connection Management

#### Transport File Authorization

Where an IS-05 Sender exposes a transport file via the `/single/senders/{sender_id}/transportfile` path, this MUST NOT
use HTTP redirects when the API is operating in authorized mode. The API MUST return an HTTP 200 code as opposed to a
3xx code when the request is successful.

Requests to the transport file path MUST be checked for a valid authorization token as specified in IS-10, including
checks that the requested path is included in the private claims. An example of claims from such a token is shown below.

```
"x-nmos-connection": {
  "read": ["single/senders/*/transportfile"]
}
```

### IS-07 - Event and Tally

#### WebSocket Authorization

When connecting to an [IS-07][] WebSocket, the JSON Web Token used MUST be validated to ensure it contains a read claim
matching all Source data which may be returned by the WebSocket. These Source paths match those used by the IS-07 Events API.

For example, access to the IS-07 Source IDs '9f463872-9621-4939-aa3a-dc3c82d8578b' and
'7f87027c-ebb4-4640-b878-14952915249a' only would require token claims as follows.

```
"x-nmos-events": {
  "read": ["sources/9f463872-9621-4939-aa3a-dc3c82d8578b",
           "sources/7f87027c-ebb4-4640-b878-14952915249a"]
}
```

Alternatively, access to all IS-07 Source IDs may be requested with a token claim as follows.

```
"x-nmos-events": {
  "read": ["sources/*"]
}
```

Specific token claims are not required in order to send IS-07 WebSocket commands such as 'subscription' and 'health',
provided the token covers the 'events' scope and includes claims for any Sources which are being requested.

### IS-12 - Control Protocol

When connecting to an [IS-12][] WebSocket control endpoint, the JSON Web Token used MUST be validated to ensure it contains a read/write claim
matching resource role paths. These represent device model role paths and MUST be created by appending [NcObject roles](https://specs.amwa.tv/ms-05-02/latest/docs/NcObject.html) starting with the [root block](https://specs.amwa.tv/ms-05-02/latest/docs/Blocks.html) and using `.` as the delimiter. Consequently the `.` character MUST NOT be used inside individual object roles.

For example, offering read access to the IS-12 receiver monitor object identified by a role path of 'root.receiver-monitors.monitor-01' only would require the following token claim.

```
"x-nmos-ncp": {
  "read": ["root.receiver-monitors.monitor-01"]
}
```

Alternatively, offering read access to all IS-12 device model objects may be given with the following token claim which uses the wildcard '*'.

```
"x-nmos-ncp": {
  "read": ["root.*"]
}
```

Access to modify object properties MUST only be given if the token claim includes a write claim with a role path that includes the object.

Access to invoke object methods MUST only be given if the token claim includes a write claim with a role path that includes the object.

The following is a token claim example which offers access to modify properties and invoke methods on the receiver monitor object identified by a role path of 'root.receiver-monitors.monitor-01'.

```
"x-nmos-ncp": {
  "read": ["root.receiver-monitors.monitor-01"],
  "write": ["root.receiver-monitors.monitor-01"]
}
```

The following is a token claim example which uses the wildcard '*' and offers access to modify properties and invoke methods on any device model object.

```
"x-nmos-ncp": {
  "read": ["root.*"],
  "write": ["root.*"]
}
```

A token claim could also offer asymmetrical access like in the following example which only allows write access to a specific path, whilst allowing the entire device model to be read.

```
"x-nmos-ncp": {
  "read": ["root.*"],
  "write": ["root.receiver-monitors.monitor-01"]
}
```

Devices MUST only allow subscriptions for objects which are associated with a role path that has been deemed to have read access through a token claim as shown in previous examples.

### IS-14 - Device Configuration

When processing requests sent to an [IS-14][] control endpoint, the JSON Web Token used MUST be validated to ensure it contains a read/write claim
matching resource role paths. These represent device model role paths and MUST be created by appending [NcObject roles](https://specs.amwa.tv/ms-05-02/latest/docs/NcObject.html) starting with the [root block](https://specs.amwa.tv/ms-05-02/latest/docs/Blocks.html) and using `.` as the delimiter. Consequently the `.` character MUST NOT be used inside individual object roles.

For example, offering read access to an IS-14 object identified by a role path of 'root.example-controls.control-01' only would require the following token claim.

```
"x-nmos-configuration": {
  "read": ["root.example-controls.control-01"]
}
```

Alternatively, offering read access to all IS-14 device model objects may be given with the following token claim which uses the wildcard '*'.

```
"x-nmos-configuration": {
  "read": ["root.*"]
}
```

Access to modify object properties MUST only be given if the token claim includes a write claim with a role path that includes the object.

Access to invoke object methods MUST only be given if the token claim includes a write claim with a role path that includes the object.

The following is a token claim example which offers access to modify properties and invoke methods on an object identified by a role path of 'root.example-controls.control-01'.

```
"x-nmos-configuration": {
  "read": ["root.example-controls.control-01"],
  "write": ["root.example-controls.control-01"]
}
```

The following is a token claim example which uses the wildcard '*' and offers access to modify properties and invoke methods on any device model object.

```
"x-nmos-configuration": {
  "read": ["root.*"],
  "write": ["root.*"]
}
```

A token claim could also offer asymmetrical access like in the following example which only allows write access to a specific path, whilst allowing the entire device model to be read.

```
"x-nmos-configuration": {
  "read": ["root.*"],
  "write": ["root.example-controls.control-01"]
}
```

Devices MUST check that PATCH requests against URLs ending in `/bulkProperties` have the necessary write access. This allows for permission errors to be identified at the validation stage before a client attempts to perform a restore by using the PUT verb.

[IS-10]: https://specs.amwa.tv/is-10 "AMWA IS-10 NMOS Authorization Specification"
[RFC-2119]: https://tools.ietf.org/html/rfc2119 "Key words for use in RFCs to Indicate Requirement Levels"
[RFC-6750]: https://tools.ietf.org/html/rfc6750 "The OAuth 2.0 Authorization Framework: Bearer Token Usage"
[IS-04 Registry]: https://specs.amwa.tv/is-04 "AMWA IS-04 NMOS Discovery and Registration Specification"
[IS-04 Referential Integrity]: https://specs.amwa.tv/is-04/v1.3/docs/4.1._Behaviour_-_Registration.html#referential-integrity "AMWA IS-04 Resource Referential Integrity"
[IS-07]: https://specs.amwa.tv/is-07 "AMWA IS-07 NMOS Event and Tally Specification"
[IS-12]: https://specs.amwa.tv/is-12 "AMWA IS-12 NMOS Control Protocol"
[IS-14]: https://specs.amwa.tv/is-14 "AMWA IS-14 NMOS Device Configuration"
