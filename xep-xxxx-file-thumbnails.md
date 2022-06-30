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
<file-thumbnails xmlns="proto:urn:xmpp:eft:0">
	<blurhash xmlns="proto:urn:xmpp:eft:blurhash">LEHV6nWB2yk8pyoJadR*.7kCMdnj</blurhash>
</file-thumbnails>
```

In this example, the thumbnail contains only a blurhash thumbnail, indicated by the `<blurhash/>` element, but an implementation
may add more versions of the same thumbnail, i.e. another `<base64-bob/>` element as a fallback if a client does not support
blurhash. To prevent confusion in the display of a thumbnail, all children of `<file-thumbnail />` must describe a thumbnail
for the same file.

### Thumbnail Types
#### Blurhash

If a sender includes a `<blurhash/>` inside the `<file-thumbnail/>` element, then it must be interpreted as
[Blurhash](https://github.com/woltapp/blurhash) data. As a sender, care must be taken to not make the Blurhash
thumbnail too big as to not loose the advantages of blurhash.

```xml
<file-thumbails xmlns="proto:urn:xmpp:eft:0">
	<blurhash xmlns="proto:urn:xmpp:eft:blurhash">LEHV6nWB2yk8pyoJadR*.7kCMdnj</blurhash>
</file-thumbnails>
```

#### Jingle Content Thumbnail Compatability

In order to enable a pseudo-interoperability with [Jingle Content Thumbnails](https://xmpp.org/extensions/xep-0264.html), inclusion of a
`<thumbnail/>` element is allowed. This is to allow easy reuse of an already existent [Jingle Content Thumbnails](https://xmpp.org/extensions/xep-0264.html)
implementation.

```xml
<file-thumbails xmlns="proto:urn:xmpp:eft:0" type="base64-bob">
	<thumbnail xmlns="urn:xmpp:thumbs:1" uri="cid:sha1+...@bob.xmpp.org" media-type="image/png" width="128" height="96" />
</file-thumbnails>
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
        <file-thumbnails xmlns="proto:urn:xmpp:eft:0">
		  	<blurhash xmlns="proto:urn:xmpp:eft:blurhash">LEHV6nWB2yk8pyoJadR*.7kCMdnj</blurhash>
			<thumbnail xmlns='urn:xmpp:thumbs:1'
                       uri='cid:sha1+ffd7c8d28e9c5e82afea41f97108c6b4@bob.xmpp.org'
                       media-type='image/png'
                       width='128'
                       height='96'/>
		</file-thumbnails>
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

- Change XML namespace from `proto:urn:xmpp:eft:0` to `urn:xmpp:eft:0` once accepted as an experimental XEP

## Changelog
### 0.1.0

- Remove the base64 thumbnail type
- Namespace all thumbnail types
- Remove the type attribute of <file-thumbnail/>
- Rename <file-thumbnail/> to <file-thumbnails/>
- Rename the namespace from file-thumbnails to eft
- Rename from File Thumbnails to Extensible File Thumbnails

## Info

| Key | Value |
| --- | --- |
| Author | PapaTutuWawa |
| Version | 0.1.0 |
| Short name | eft |
