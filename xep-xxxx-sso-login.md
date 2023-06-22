# Single Sign-On (SSO) Login
## Introduction

XMPP implementations can authenticate against a server with various different
methods. There exist password-based mechanisms, like *SCRAM* or *PLAIN*, and token-based
approaches like [FAST](https://xmpp.org/extensions/inbox/xep-fast.html). However, for certain deployments, where authentications is delagated
to a third-party, *SCRAM* and *PLAIN* cannot be used as they require some access to the user's
raw password (in the case of *SCRAM* the hashed password). [FAST](https://xmpp.org/extensions/inbox/xep-fast.html) can only be used for
reauthentication without access to the user's password.

This specification provides a way for XMPP clients to authenticate with a server using
server-provided single sign-on (SSO) solutions.

## Authentication

When a client connects to the server and the server deems the connection ready for authentication, a
server providing login using SSO provides it as a [SASL2](https://xmpp.org/extensions/xep-0388.html) feature:

```xml
<?xml version='1.0'?>
<stream:stream
  from='example.org'
  id='++TR84Sm6A3hnt3Q065SnAbbk3Y='
  to='user@example.org'
  version='1.0'
  xml:lang='en'
  xmlns='jabber:client'
  xmlns:stream='http://etherx.jabber.org/streams'>
<stream:features>
  <authentication xmlns='urn:xmpp:sasl:2'>
    <mechanism>SCRAM-SHA-1</mechanism>
    <mechanism>SCRAM-SHA-1-PLUS</mechanism>
    <sso xmlns='proto:urn:xmpp:sso:0'>
        <service id='sso-service-1' icon='https://...'>Login with Service 1</service>
        <service id='sso-service-2' icon='https://...'>Login with Service 2</service>
        <service id='sso-service-3' icon='https://...'>Login with Service 3</service>
    </sso>
  </authentication>
</stream:features>
```

Each `<service />` element identifies a SSO service that a client can authenticate against. Each `<service />` element's id attribute
MUST be unique in its listing as it identifies what service the client wants to authenticate against in the next step. It MUST NOT contain
the `,` and `=` characters, An optional icon attribute points to a URL where an icon for the service can be found. Clients MUST ignore non-HTTP URLs. The text of the element
MUST provide a human-readable string identifying the service. It SHOULD idealy be short to pose minimal UI constraints.

While the server in this example also provides authentication using *SCRAM*, a server does not have to. In a business deployment, the only authentication
mechanism MAY be just the company's SSO mechanism.

In the next step, the client specifies that it wants to authenticate using (in this example) `sso-service-1`.

```xml
<authenticate xmlns='urn:xmpp:sasl:2' mechanism='SCRAM-SHA-1-PLUS'>
  <!-- Base64 of: ',,id=sso-service-1' -->
  <initial-response>LCxuPXVzZXIsaWQ9c3NvLXNlcnZpY2UtMQo=</initial-response>
  <user-agent id='d4565fa7-4d72-4749-b3d3-740edbf87770'>
    <software>AwesomeXMPP</software>
    <device>Kiva's Phone</device>
  </user-agent>
</authenticate>
```

In case the client sent an unknown SSO service identifier, it SHOULD respond with an error:

```xml
<failure xmlns='urn:xmpp:sasl:2'>
  <aborted xmlns='urn:ietf:params:xml:ns:xmpp-sasl'/>
  <unknown-service xmlns='proto:urn:xmpp:sso:0'/>
</failure>
```

If the chosen service identifier is known, the server MUST send an HTTPS URL at which the client can authenticate. In case of protocols like OpenID Connect, the sent URL MUST point
to a server-controlled callback URL, so that the server knows when the authentication succeeded. That callback URL SHOULD display a hint to the user that they may return to their
XMPP client.

```xml
<challenge xmlns='urn:xmpp:sasl:2'>
  <!-- Base64 of: 'https://auth.example.org/login?secure-parameter=abc&sid=123' -->
  aHR0cHM6Ly9hdXRoLmV4YW1wbGUub3JnL2xvZ2luP3NlY3VyZS1wYXJhbWV0ZXI9YWJjJnNpZD0xMjMK
</challenge>
```

Next, the client responds with an empty [SASL2](https://xmpp.org/extensions/xep-0388.html) response:

```xml
<response xmlns='urn:xmpp:sasl:2'></response>
```

The user is should then be redirected to the server-provided authorization URL.

## Success Case

If the user's authentication succeeded and the server's callback URL has been called, the server MUST retrieve user information from the authentication server and use it for
the following session. In this example, the returned user information contained the username `user`. As such, the resulting authentication success may look like this (assuming
the server is responsible for the `example.org` domain):

```xml
<success xmlns='urn:xmpp:sasl:2'>
    <authorization-identifier>user@example.org</authorization-identifier>
</success>
```

## Failure Case

If the authentication somehow fails, the server MUST return a [SASL2](https://xmpp.org/extensions/xep-0388.html) authentication failure. One possible failure may be a timeout:

```xml
<failure xmlns='urn:xmpp:sasl:2'>
    <aborted xmlns='urn:ietf:params:xml:ns:xmpp-sasl'/>
    <auth-timeout xmlns='proto:urn:xmpp:sso:0' />
</failure>
```

## Business Rules

This authentication mechanism SHOULD only be used on interactive authentication, i.e. when the user is physically able to perform actions, and not on
non-interactive logins, for example when reconnecting after losing the connection to the server.

When authenticating, it is recommended to also request a token for a token-based authentication mechanism, like [FAST](https://xmpp.org/extensions/inbox/xep-fast.html), so that the user does not have
to reauthenticate to the third-party on every new connection.

## Security Considerations

This specification allows offloading the actual credential handling from the XMPP server to another third-party. Thus, security considerations of that chosen
third-party authentication solution apply.

The server-provided authentication URLs MUST be using a secure protocol, like HTTPS. Clients MUST ignore insecure URLs.

## Info

| Key | Value |
| --- | --- |
| Author | PapaTutuWawa |
| Version | 0.0.1 |
| Short name | sso |