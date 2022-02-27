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
  <file-upload xmlns="urn:xmpp:fun:0">
    <file xmlns="urn:xmpp:file:metadata:0">
	  <name>vacation.jpg</name>	
	  <media-type>image/jpeg</media-type>
	</file>
	<thumbnail type="blurhash" xmlns="urn:xmpp:thumbnail:0">
	  <payload>LEHV6nWB2yk8pyoJadR*.7kCMdnj</payload>
	</thumbnail>
  </file-upload>
  <origin-id xmlns="urn:xmpp:sid:0" id="ccccc" />
  <no-permanent-store xmlns="urn:xmpp:hints" />
</message>
```

The `<file-upload>` element indicates that a message should be replaced with the actual
file embed once the upload is done. Metadata about the file should be included
as specified by [File metadata element](https://xmpp.org/extensions/xep-0446.html).
The metadata should include only the bare minimum, i.e. the mime type and filename.
Additionally, a thumbnail can be sent with the notification in order to allow clients
to already show a preview. The `<thumbnail />` element is specified by [Thumbnails](https://github.com/PapaTutuWawa/custom-xeps/blob/master/xep-xxxx-thumbnails.md).

Note that [Unique and Stable Origin IDs](https://xmpp.org/extensions/xep-0359.html) must be used when the message is sent to a
groupchat.

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
  <replaces xmlns="urn:xmpp:fun:0" id="ccccc" />
  <origin-id xmlns="urn:xmpp:sid:0" id="ddddd" />
</message>
```

The `id` attribute of the `<replaces />` element refers to either the stanza ID or the
origin ID of the message that contained the original `<file-upload />`.

If sent to a groupchat, the origin ID must be used.

Note the the actual method of communicating a file is of no relevance here, as long as the
method allows a client to show it inline. Examples for such methods are
[Out of Band Data](https://xmpp.org/extensions/xep-0066.html)
and [Stateless Inline Media Sharing](https://xmpp.org/extensions/xep-0385.html).

## Security Considerations

A client receiveing a message with an `<replaces />` element must verify if the message it
is supposed to replace was actually sent by the sender of the `<replaces />` element to
prevent arbitrary messages to be replaced.
