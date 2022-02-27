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
<file-thumbnail type="proto:urn:xmpp:file-thumbnails:0:blurhash" xmlns="proto:urn:xmpp:file-thumbnails:0">
	<blurhash>LEHV6nWB2yk8pyoJadR*.7kCMdnj</blurhash>
</thumbnail>
```

In this example, the thumbnail is specified of type `proto:urn:xmpp:file-thumbnails:0:blurhash`, which means that a supporting
client should decode the `<blurhash/>` element using the [Blurhash](https://github.com/woltapp/blurhash) algorithm.

In order to enable a sort-of interoperability with [Jingle Content Thumbnails](https://xmpp.org/extensions/xep-0264.html), a type
`proto:urn:xmpp:file-thumbnails:0:jct` is defined.

```xml
<file-thumbnail type="proto:urn:xmpp:file-thumbnails:0:jct" xmlns="proto:urn:xmpp:file-thumbnails:0">
	<thumbnail xmlns="urn:xmpp:thumbs:1" uri="..." media-type="image/png" width="128" height="96" />
</thumbnail>
```

In the case of [Jingle Content Thumbnails](https://xmpp.org/extensions/xep-0264.html) compatibility,
the `<file-thumbnail />` element must contain exactly one `<thumbnail xmlns="urn:xmpp:thumbs:1" />` element.

Other extensions may define similar types. Note that the content of the `<file-thumbnail />` element
is dependent on the type specified in the `<file-thumbnail xmlns="proto:urn:xmpp:file-thumbnails:0" />`
element.

### Thumbnail Types
#### Blurhash

If a sender specifies the type to be of `proto:urn:xmpp:file-thumbnails:0:blurhash`, then the `<file-thumbnail />` element must contain the
image data as specified by the [Blurhash](https://github.com/woltapp/blurhash) algorithm. Care should be taken to prevent
the blurhash data from growing to large in order to prevent hitting the stanza size limit.

#### Jingle Content Thumbnail Compatability

If a sender specified the type to be of `proto:urn:xmpp:file-thumbnails:0:jct`, then the `<file-thumbnail />`
element must contain exactly one `<thumbnail xmlns="urn:xmpp:thumbs:1" />` element.

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
        <file-thumbnail type="proto:proto:urn:xmpp:file-thumbnails:0:jct" xmlns='urn:xmpp:thumbnail:0'>
			<thumbnail xmlns='urn:xmpp:thumbs:1'uri='cid:sha1+ffd7c8d28e9c5e82afea41f97108c6b4@bob.xmpp.org' media-type='image/png' width='128' height='96'/>
		</thumbnail>
        <file-thumbnail type="proto:urn:xmpp:file-thumbnails:0:blurhash" xmlns='proto:urn:xmpp:file-thumbnails:0'>
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
| Version | 0.0.2 |
