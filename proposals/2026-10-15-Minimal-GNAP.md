# Minimal GNAP Support

## Metadata

|Tag |Value |
|---- | ---------------- |
|Proposal |[2026-10-15-Minimal-GNAP](https://github.com/OAI/OpenAPI-Specification/tree/main/proposals/{2026-10-15-Minimal-GNAP.md})|
|Authors | [Henry Andrews](https://github.com/handrews) |
|Review Manager | TBD |
|Status | Proposal |
|Implementations | n/a |
|Issues | [#7](https://github.com/OAI/sig-security/issues/7) |
|Previous Revisions | n/a |

## Change Log

|Date |Responsible Party |Description |
|---- | ---------------- | ---------- |
| 2026-10-15 | Henry Andrews | Initial version |

## Introduction

[RFC9635 Grant Negotiation and Authorization Protocol (GNAP)](https://www.rfc-editor.org/info/rfc9635/#resource-access-rights)
is a next-generation authorization system intended as a replacement
for OAuth 2.  Some of its features align with newer OAuth 2 extensions,
but it takes a fundamentally different approach emphasizing runtime
negotiation with minimal pre-flight configuration.

## Motivation

GNAP is an emerging standard with institutional backing, including
grants from the Interledger Foundation to add support to the OAS.
Full disclosure: the author of this proposal is one of the grantees.

## Proposed solution

Minimal GNAP support using access reference strings can be enabled
by adding a Security Scheme Object `type: gnap` with an optional
`gnapAsUri' field, and allowing GNAP access strings in the Security
Requirement Object in place of OAuth scopes.

Whether the endpoint is defined statically in `gnapAsUri` or discovered
through a `WWW-Authenticate` challenge, an `OPTIONS` request to that
endpoint will provide further configuration information.

In many if not most cases, support is also needed for GNAP's Resource
Access Rights (RAR).  OAS support for these is addressed by a companion
proposal, as they share structure and behavior with OAuth 2 Rich
Authorization Requests (also RAR, which was intentional).

### Examples

This example shows four possible configurations, the details of which
are explained after the example:

* Fully dynamic, where both the AS URI and the access string are
  discovered via `WWW-Authenticate`.
* Static AS, dynamic access string, where the `WWW-Authenticate`
  challenge needs to provide `access` but might omit `as\_uri`.
* Dynamic AS, static access, where the `WWW-Authenticate` challenge needs
  to provide `as\_uri` but is expected to omit the `access` parameter.
* Fully static, where both the AS URI and the access string are
  provided in the OAD.

```yaml
components:
  securitySchemes:
    gnapDynamic:
      type: gnap
    gnapStatic:
      type: gnap
      gnapAsUri: https://as.example.com/grant
security:
- gnapDynamic: []   # Default is fully dynamic.
paths:
  /staticAsUri/dynamicAccess:
    security:
    - gnapStatic: []
  /dynamicAsUri/staticAccess:
    security:
    - gnapDynamic: [read]
  /staticAsUri/staticAccess:
    security:
    - gnapStatic: [read]
```

Fully dynamic discovery can be either
[Resource Server-first](https://www.rfc-editor.org/info/rfc9635/#section-9.1)
(RS-first), which is defined by the GNAP RFC, or an extension process.
As no extension processes are yet defined, they are outside the scope
of this proposal.

With RS-first usage, a request to a protected resource results in
a `401 Unauthorized` response with a `WWW-Authenticate` that at least
includes the `as\_uri` parameter that provides the grant request endpoint.
Such a response MAY also include `access` and `referrer` parameters
that, together with `as\_uri`, provide everything needed to request
authorization.

It is possible for a request using statically configuerd GNAP
information to fail with a new `WWW-Authenticate` challenge, in which
case the challenge SHOULD be considered more up-to-date and therefore
authoritative at that moment.

See the "Alternatives considered" section below for additional
features that could be added later (before or after the next release).

## Detailed design

Below are the exact OAS changes under the relevant section headers.
The only changes to the introductory paragraph and the `type` field
are to add GNAP to the relevant lists and to split up the long-ish
paragraph. I also moved much of the text out of the field table for
the Security Requirement Object as it became far too long to be
readable there.

The resolution rules for a relative value of `gnapGrantRequestEndpointUrl`
are the same as for other Security Scheme URLs.  There is some ambiguity
around this, but this will be resolved by addressing OAI/sig-security#31.

We currently do not provide explanatory text for Security Scheme examples,
so I have not included any below.  We might want to reconsider that.

### Security Scheme Object

Defines a security scheme that can be used by the operations.

Supported schemes are HTTP authentication, an API key (either as a header, a cookie parameter or as a query parameter), mutual TLS (use of a client certificate), OAuth2’s common flows (implicit, password, client credentials and authorization code) as defined in [RFC6749], OAuth2 device authorization flow as defined in [RFC8628], [OpenID-Connect-Core], and GNAP as defined in [RFC9635].

Please note that as of 2020, the implicit flow is about to be deprecated by OAuth 2.0 Security Best Current Practice. Recommended for most use cases is Authorization Code Grant flow with PKCE.

#### Fixed Fields

| Field Name | Type | Applies To | Description |
| ---- | :----: | ---- | ---- |
| <a name="security-scheme-type"></a>type | `string` | Any | **REQUIRED**. The type of the security scheme. Valid values are `"apiKey"`, `"http"`, `"mutualTLS"`, `"oauth2"`, `"openIdConnect"`, `"gnap"`. |
| <a name="security-scheme-gnap-grant-request-endpoint-url"></a>grantRequestEndpointUrl | `string` | `gnap` | The URL of the Authorization Server's grant request endpoint.  Per [RFC9635] an HTTP `OPTIONS` request to this endpoint will provide additional configuration information. |

### Security Scheme Object Examples

##### Resource Server-First GNAP Example

```yaml
type: gnap
```

##### Explicit Authorization Server GNAP Example

```yaml
type: gnap
gnapGrantRequestEndpointUrl: https://as.example.com/grant
```

### Security Requirement Object

#### Patterned Fields

| Field Pattern | Type | Description |
| ---- | :----: | ---- |
| <a name="security-requirements-name"></a>{name} | [`string`] | Each name or URI MUST correspond to a security scheme as described above. See below for how to use the values. |

If the security scheme is of type `"oauth2"` or `"openIdConnect"`, then
the value is a list of scope names required for the execution, and the list
MAY be empty if authorization does not require a specified scope.

If the security scheme is of type `"gnap"`, the value is a list of GNAP
reference strings (not related to `$ref` references).  If the list is
empty, operations are expected to respond with a `WWW-Authenticate`
challenge that includes an `"access"` parameter providing the required
reference string.  If the list is non-empty but a `WWW-Authenticate`
challenge provides an `"access"` parameter that is not in the list,
clients SHOULD assume that the challenge is more up-to-date and
therefore correct.

For other security scheme types, the array MAY contain a list of role
names which are required for the execution, but are not otherwise defined
or exchanged in-band.

#### Security Requirement Object Examples

##### GNAP with dynamic access strings

```yaml
security:
- gnap: []
```

##### GNAP with static access strings

```yaml
security:
- gnap: [read, write]
```

## Backwards compatibility

This proposal is purely additive.

## Alternatives considered

### SHOULD vs MUST for WWW-Authenticate conflicts

I chose a SHOULD for treating the `WWW-Authenticate` challenge as
authoritative as there may be good reason to instead stop attempting
to access the resource, particularly if the change in configuration
was unexpected.

This could be weakened to a MAY, but I do not think a MUST is appropriate.

### Possible additional fields and features

Not including RAR which is in a companion proposal to this one.
The possiblity of pre-declaring GNAP access reference strings
the way we pre-declare OAuth 2 scopes is addressed in the RAR proposal.

#### Static configuration

The possibility of manually specifying such configuration information
goes against the design of GNAP, but may have valid use cases.
The same considerations exist with discoverable OAuth 2 and OIDC,
so configuration concerns will be addressed separately to span all
three systems.  Such configuration enhancements are not necesary
for GNAP to be useful.

### Client identification

While a GNAP client also needs a public key to make grant requests, per
[Section 2.3 of RFC9635](https://www.rfc-editor.org/info/rfc9635/#section-2.3-9):

    If the same public key is sent by value on different access requests,
    the AS MUST treat these requests as coming from the same client
    instance for purposes of identification, authentication, and policy
    application.

Therefore, an OAD is not an appropriate place for client public keys.

The other by-value client object fields are optional.  The `class\_id`
field is optional and AS-defined.  It is similarly client-specific.
The `display` field is used to display information to a Resource Owner,
and the use case for including this in the OAD is not clear to me.

By-reference (as opposed to by-value as mentioned in the quote) identifiers
are analogous to OAuth 3 client IDs, which we also do not include in OADs.

### Bikeshedding

#### Grant request endpoint field name

The `gnapAsUri` field was chosen to match the `WWW-Authenticate` challenge
parameter `as\_uri`, using the OAS's established convention of treating
acronyms like words for the purpose of camelCasing.

The drawback is that in the OAS, "URI" usually refers to OAD URIs, while
API-related endpoints are described as "URL"s.  As these two sets of
URIs have different resolution rules, this could be confusing, and
might require a note in the relative resolution sections.  However,
as noted in the detailed design, there is already work that needs to
be done to clarify this area for Security Scheme and OAuth Flow Objects.

Alternatives coudl be based on the longer standard name returned in
the response to an `OPTIONS` request, `grant\_request\_endpoint`.
I considered `gnapGrantRequestEndpoint` (closest match),
`gnapGrantRequestEndpointUrl` (match but follow our URL convention,
which as noted above contradicts GNAP's usage), or `gnapGrantRequestUrl`
(further emphasizing our conventions over GNAP's).  All of these are
annoyingly lenghty.

#### Unified discovery URL

We could deprecate `oauth2MetadataUrl` and `openIdConnectUrl` and
have a single field for the entry point of whatever system is indicated
by the `type` field.

However, this would break down if a future `type` requires multiple
initial URLs, and it feels like a conceptual optimization of a problem
no one has complained about.  There are too many fields in the Security
Scheme Object, but as we can't fully remove any until 4.0, we would
add potential confusion without truly reducing the set of fields.
