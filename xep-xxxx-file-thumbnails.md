# File Thumbnails
## Introduction

When sending media, it is often helpful for users when a thumbnail can be shown before the
actual file is finished downloading.

XMPP already has a protocol for specifying thumbails, [Jingle Content Thumbnails](https://xmpp.org/extensions/xep-0264.html).
However, this protocol requires that the thumbnail is a base64-encoded image published using
[Bits of Binary](https://xmpp.org/extensions/xep-0231.html). This prevents clients from implementing newer technologies, such as
[Blurhash](https://github.com/woltapp/blurhash).

This document specifies are more general and extensible element for specifiying thumbnails.

## Usage

```xml
<file-thumbnail type="blurhash" xmlns="proto:urn:xmpp:file-thumbnails:0">
	<blurhash>LEHV6nWB2yk8pyoJadR*.7kCMdnj</blurhash>
</thumbnail>
```

In this example, the thumbnail is specified of type `proto:urn:xmpp:file-thumbnails:0:blurhash`, which means that a supporting
client should decode the `<blurhash/>` element using the [Blurhash](https://github.com/woltapp/blurhash) algorithm.

Other extensions may define similar types. Note that the content of the `<file-thumbnail />` element
is dependent on the type specified in the `<file-thumbnail />` element.

### Thumbnail Types
#### Blurhash

If a sender specifies the type to be of `blurhash`, then the `<file-thumbnail />` element must contain the
image data as specified by the [Blurhash](https://github.com/woltapp/blurhash) algorithm. Care should be taken to prevent
the blurhash data from growing to large in order to prevent hitting the stanza size limit.

```xml
<file-thumbail xmlns="proto:urn:xmpp:file-thumbnails:0" type="blurhash">
	<blurhash>LEHV6nWB2yk8pyoJadR*.7kCMdnj</blurhash>
</file-thumbnail>
```

#### Jingle Content Thumbnail Compatability

In order to enable a pseudo-interoperability with [Jingle Content Thumbnails](https://xmpp.org/extensions/xep-0264.html), a type
`base64-bob` is defined.

If a sender specified the type to be of `base64-bob`, then the `<file-thumbnail />`
element must contain exactly one `<base64-bob />` element. The attributes of the element are the same
as the `<thumbnail />` element specified in [Jingle Content Thumbnails](https://xmpp.org/extensions/xep-0264.html).

```xml
<file-thumbail xmlns="proto:urn:xmpp:file-thumbnails:0" type="base64-bob">
	<base64-bob uri="cid:sha1+...@bob.xmpp.org" media-type="image/png" width="128" height="96" />
</file-thumbnail>
```

This allows clients that already support [Jingle Content Thumbnails](https://xmpp.org/extensions/xep-0264.html) to reuse
the same logic they use for retrieving thumbnail data.

#### Base64 Data

If a sender specified the type to be of `base64`, then the `<file-thumbnail />` element must
contain exactly one `<base64 />` element. The element must posess an `media-type` attribute, whose value
contains the mime type of the thumbnail data, while the inner text of the element contains the base64-encoded
binary thumbnail data.

```xml
<file-thumbail xmlns="proto:urn:xmpp:file-thumbnails:0" type="base64">
	<base64 media-type="image/png" width="128" height="96">
		[base64 data]
	</base64>
</file-thumbnail>
```

Note that this thumbnail type is essentially the same as the `base64-bob` type, but allowing the thumbnail to be used even if
the sender is not available on a server that does not cache [Bits of Binary](https://xmpp.org/extensions/xep-0231.html) data or has it otherwise not available.

If using this thumbnail type, care has to be taken in order to not hit any existing stanza size limits.

## Usage Example with Stateless Inline Media Sharing

NOTE: This example is taken from [Stateless Inline Media Sharing](https://xmpp.org/extensions/xep-0385.html) and modified for this protocol.

```xml
<message to='julient@shakespeare.lit' from='romeo@montague.lit'>
  <body>Look at the nice view from the summit.</body>
  <reference xmlns='urn:xmpp:reference:0' begin='17' end='20' type='data'>
    <media-sharing xmlns='urn:xmpp:sims:1'>
      <file xmlns='urn:xmpp:jingle:apps:file-transfer:5'>
        <media-type>image/jpeg</media-type>
        <name>summit.jpg</name>
        <size>3032449</size>
        <hash xmlns='urn:xmpp:hashes:2' algo='sha3-256'>2XarmwTlNxDAMkvymloX3S5+VbylNrJt/l5QyPa+YoU=</hash>
        <hash xmlns='urn:xmpp:hashes:2' algo='id-blake2b256'>2AfMGH8O7UNPTvUVAM9aK13mpCY=</hash>
        <desc>Photo from the summit.</desc>
        <file-thumbnail type="base64-bob" xmlns='urn:xmpp:thumbnail:0'>
			<base64-bob uri="cid:sha1+ffd7c8d28e9c5e82afea41f97108c6b4@bob.xmpp.org" media-type="image/png" width="128" height="96" />
		</thumbnail>
        <file-thumbnail type="blurhash" xmlns="proto:urn:xmpp:file-thumbnails:0">
		  	<blurhash>LEHV6nWB2yk8pyoJadR*.7kCMdnj</blurhash>
		</thumbnail>
      </file>
      <sources>
        <reference xmlns='urn:xmpp:reference:0' type='data' uri='https://download.montague.lit/4a771ac1-f0b2-4a4a-9700-f2a26fa2bb67/summit.jpg' />
        <reference xmlns='urn:xmpp:reference:0' type='data' uri='xmpp:romeo@montague.lit/resource?jingle;id=9559976B-3FBF-4E7E-B457-2DAA225972BB' />
      </sources>
    </media-sharing>
  </reference>
</message>
```

## Security Considerations

The same considerations apply as specified by [Jingle Content Thumbnails](https://xmpp.org/extensions/xep-0264.html).

## Info

| Key | Value |
| --- | --- |
| Author | PapaTutuWawa |
| Version | 0.0.3 |
