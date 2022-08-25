# Extensible File Thumbnails
## Introduction

When sending media, it is often helpful for users when a thumbnail can be shown before the
actual file is finished downloading.

XMPP already has a protocol for specifying thumbails, [Jingle Content Thumbnails](https://xmpp.org/extensions/xep-0264.html).
However, this protocol requires that the thumbnail is a base64-encoded image published using
[Bits of Binary](https://xmpp.org/extensions/xep-0231.html). This prevents clients from implementing newer technologies, such as
[Blurhash](https://github.com/woltapp/blurhash).

This document specifies are more general and extensible element for specifiying thumbnails in various formats.

## Usage

```xml
<file-thumbnail type="proto:urn:xmpp:eft:0:blurhash" xmlns="proto:urn:xmpp:eft:0">
	<blurhash xmlns="proto:urn:xmpp:eft:0:blurhash">LEHV6nWB2yk8pyoJadR*.7kCMdnj</blurhash>
</file-thumbnail>
```

This example specifies a thumbnail of type `blurhash`, meaning that the child element of the
`<file-thumbnail/>` element must contain a `<blurhash/>` element (See "Thumbnail Types").

### Thumbnail Types
#### Blurhash

If a sender specifies the type of the `<file-thumbnail/>` element to be
`proto:urn:xmpp:eft:0:blurhash`, then the `<blurhash/>` element's value must be interpreted
as [Blurhash](https://github.com/woltapp/blurhash) data. As a sender, care must be taken to
not make the Blurhash thumbnail too big as to not loose the advantages of Blurhash.

```xml
<file-thumbail xmlns="proto:urn:xmpp:eft:0">
	<blurhash xmlns="proto:urn:xmpp:eft:0:blurhash">LEHV6nWB2yk8pyoJadR*.7kCMdnj</blurhash>
</file-thumbnail>
```

#### Jingle Content Thumbnail Compatability

In order to enable a pseudo-interoperability with [Jingle Content Thumbnails](https://xmpp.org/extensions/xep-0264.html), inclusion of a
`<thumbnail/>` element is allowed. This is to allow easy reuse of an already existent [Jingle Content Thumbnails](https://xmpp.org/extensions/xep-0264.html)
implementation. The type of the `<file-thumbnail/>` must then be set to `proto:urn:xmpp:eft:0:base64-bob`.

```xml
<file-thumbail xmlns="proto:urn:xmpp:eft:0" type="proto:urn:xmpp:eft:0:base64-bob">
	<thumbnail xmlns="urn:xmpp:thumbs:1" uri="cid:sha1+...@bob.xmpp.org" media-type="image/png" width="128" height="96" />
</file-thumbnail>
```

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
                <file-thumbnail type="proto:urn:xmpp:eft:0:blurhash" xmlns="proto:urn:xmpp:eft:0">
                    <blurhash xmlns="proto:urn:xmpp:eft:blurhash">LEHV6nWB2yk8pyoJadR*.7kCMdnj</blurhash>
                </file-thumbnail>
                <file-thumbnail type="proto:urn:xmpp:eft:0:base64-bob" xmlns="proto:urn:xmpp:eft:0">
                    <thumbnail xmlns='urn:xmpp:thumbs:1'
                            uri='cid:sha1+ffd7c8d28e9c5e82afea41f97108c6b4@bob.xmpp.org'
                            media-type='image/png'
                            width='128'
                            height='96'/>
                </file-thumbnail>
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

## Todo

- Change XML namespace from `proto:urn:xmpp:eft:0` to `urn:xmpp:eft:0` once we submit this as an XEP

## Changelog
### 0.2.0

- Namespace the thumbnail type
- `<file-thumbnails/>` to `<file-thumbnail/>`

### 0.1.0

- Remove the base64 thumbnail type
- Namespace all thumbnail types
- Remove the type attribute of `<file-thumbnail/>`
- Rename `<file-thumbnail/>` to `<file-thumbnails/>`
- Rename the namespace from file-thumbnails to eft
- Rename from File Thumbnails to Extensible File Thumbnails

## Info

| Key | Value |
| --- | --- |
| Author | PapaTutuWawa |
| Version | 0.2.0 |
| Short name | eft |
