# File Upload Notification

## Introduction

When sending files relevant to a current discussion, it sometimes happens that the conversation
moves on while waiting for the file to be uploaded, for example by using [HTTP-Upload](https://xmpp.org/extensions/xep-0363.html).
This causes the file to be sent at a later date.

File Upload Notifications are a way to allow supporting clients to place a pseudo-message
that will be replaced once the file has been uploaded.

## Pre-Upload

```xml
<message from="some.user@some.server/abc123" to="someone.else@other.server" id="aaaaa">
    <file-upload xmlns="proto:urn:xmpp:fun:0">
        <file xmlns="urn:xmpp:file:metadata:0">
            <name>vacation.jpg</name>	
            <media-type>image/jpeg</media-type>
            <thumbnail type="blurhash" xmlns="proto:urn:xmpp:eft:0">
                <blurhash>LEHV6nWB2yk8pyoJadR*.7kCMdnj</blurhash>
            </thumbnail>
            <thumbnail type="base64-bob" xmlns="proto:urn:xmpp:eft:0">
                <base64-bob uri="cid:sha1+...@bob.xmpp.org" media-type="image/png" width="128" height="96" />
            </thumbnail>
        </file>
    </file-upload>
    <no-permanent-store xmlns="urn:xmpp:hints" />
</message>
```

The `<file-upload>` element indicates that a message should be replaced with the actual
file embed once the upload is done. Metadata about the file should be included
as specified by [File metadata element](https://xmpp.org/extensions/xep-0446.html).
The metadata should include only the bare minimum, i.e. the mime type and filename.
Additionally, zero or more thumbnails can be sent with the notification in order to allow clients
to already show a preview. The `<file-thumbnail />` element is specified by [File Thumbnails](https://github.com/PapaTutuWawa/custom-xeps/blob/master/xep-xxxx-file-thumbnails.md).

Since this message carries no meaning to anyone retrieving it after the file upload has been
completed, a `<no-permanent-store />` element should be added (See [Message Processing Hints](https://xmpp.org/extensions/xep-0334.html)).

## Post-Upload

After the file upload has been completed, the sending client includes a `<replaces />` element
in the message to inform clients which messages should be replaced.

```xml
<message from="some.user@some.server/abc123" to="someone.else@other.server" id="bbbbb">
  <body>...</body>
  <x xmlns="jabber:x:oob">
	<url>...</url>
  </x>
  <replaces xmlns="proto:urn:xmpp:fun:0" id="aaaaa" />
</message>
```

The `id` attribute of the `<replaces />` element refers to either the stanza ID of the
message that contained the original `<file-upload />`.

Note that the actual method of communicating a file is of no relevance here, as long as the
method allows a client to show it inline. Examples for such methods are
[Out of Band Data](https://xmpp.org/extensions/xep-0066.html)
and [Stateless Inline Media Sharing](https://xmpp.org/extensions/xep-0385.html).

In case the sender uses methods like [Stateless Inline Media Sharing](https://xmpp.org/extensions/xep-0385.html)
or [Stateless File Sharing](https://xmpp.org/extensions/xep-0447.html) which allow specifying
a thumbnail, then the client should do here as well to allow clients catching up using
[Message Archive Management](https://xmpp.org/extensions/xep-0313.html) to also provide a thumbnail to their users.

## Cancelled Upload

If the uploading entity has cancelled the upload, then it should indicate so to the other communicating entities.

```xml
<message from="some.user@some.server/abc123" to="someone.else@other.server" id="bbbbb">
  <cancelled xmlns="proto:urn:xmpp:fun:0" id="aaaaa" />
</message>
```

In this example, the uploading entity just sends a message containing a `<cancelled />` tag to indicate the
cancellation, allowing receiving clients to perhaps stop showing loading spinners and the
like. Its `id` attribute refers to the message containing the original `<file-upload />`
element.

## Security Considerations

- A client receiving a File Upload Notification must ensure that only messages containing a `<file-upload />` are replaced. This is to ensure arbitrary messages being replaced by file uploads.
- A client receiving a File Upload Notification MUST ensure that replacements and cancellations are only accepted from the JID that sent the original message containing the `<file-upload />` element. This means that as long as the two JIDs are equal when bare, then the replacement or cancellation is valid.

## Info

| Key | Value |
| --- | --- |
| Author | PapaTutuWawa |
| Version | 0.0.5 |
| Short name | fun |
