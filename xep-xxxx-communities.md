# Communities

In different instant messaging services, users have the options to create collections of group chat venues that are logically linked
together.

This XEP is designed in a modular way, similar to [Mediated Information Exchange]. As such, this XEP only provided the basics for such
groupings.

## Design Requirements

To aid adoption, the following requirements are considered while creating this XEP:

- Basic communities, i.e. grouped links to other JIDs with additional metadata, should work without server support
- The protocol should be extensible for future evolution

## Basics

A community in XMPP is identified by a JID that "owns" the community. This may be a end-user JID or a server JID. This JID must provide
a [PubSub](https://xmpp.org/extensions/xep-0060.html) service on which the community's metadata is stored. This XEP proposes a couple of basic nodes, but other XEPs may extend
this set and provide custom extensions.

A community may be either public, meaning that anyone who knows the JID may join and query information about, or private, meaning that users
wishing to join MUST be added by an administrator of the community. To accomplish this, public communities SHOULD set all their [PubSub](#) nodes
to have an access model of "open", while private communities SHOULD set all of their nodes to "whitelist". A future XEP may define a way for
administrators to pre-authenticate users such that "invite URLs" to users.

### Description

A community's information is described by a `<information />` element containing information about the community.

```xml
<information xmlns="proto:urn:xmpp:community:0">
  <title>Awesome Community</title>
  <description>...</description>
  <icon url="https://example.org" />
</information>
```

This element is stored in the `proto:urn:xmpp:community:0:info` [PubSub](https://xmpp.org/extensions/xep-0060.html) node at the bare JID of the community.
Implementations MUST use an id of "latest" to prevent multiple revisions of existing.

```xml
<iq type="result" from="community.example.org">
  <pubsub xmlns="http://jabber.org/protocol/pubsub">
    <items node="proto:urn:xmpp:community:0:info">
      <item id="latest">
        <information xmlns="proto:urn:xmpp:community:0">
          <title>Awesome Community</title>
          <description>...</description>
          <icon url="https://example.org" />
        </information>
      </item>
    </items>
  </pubsub>
</iq>
```

### Channel

Within this XEP, a channel refers to each item within a channel group that should be displayable by the client. This XEP provides
the `<channel />` element which community administrators MAY use to link to items within the XMPP network by its JID.

The defined types are:

- `groupchat`: A joinable group chat (MUC, MIX, ...)
- `user`: A JID that a one-on-one conversation can be started with
- `community`: A link to another community as defined per this XEP.

```xml
<channel id="dev" type="groupchat" jid="dev@muc.example.org" title="Development" />
```

The title attribute describes the function of the channel to the user. The id is used for sorting within a
channel group. The id MUST consist of only alpha-numeric characters.

### Channel Groups

A channel group is a list of items that should be logically grouped together. This may include group chat venues (MUC, MIX),
user JIDs (like admins), or other channel groups.

```xml
<group xmlns="proto:urn:xmpp:community:0" id="development">
  <title>Development chats</title>
  <channel id="dev" type="groupchat" jid="dev@muc.example.org" title="Development" />
  <channel id="support" type="groupchat" jid="support@muc.example.org" title="Support" />
  <channel id="user" type="user" jid="user@example.org" title="Random user" />
  <channel id="other-community" type="community" jid="some-other-community.example.org" title="Partner community" />
</group>
```

Each extension item inside the `<group />` must also carry an id. The id must be unique within its surrounding group, but not the parent or child groups.

A community has a single `proto:urn:xmpp:community:0:groups` [PubSub](https://xmpp.org/extensions/xep-0060.html) node that contains one top-level group called the *root group*. It MAY have a title, but it does not
have to.

```
<iq type="result" from="community.example.org">
  <pubsub xmlns="http://jabber.org/protocol/pubsub">
    <items node="proto:urn:xmpp:community:0:groups">
      <item id="latest">
        <group xmlns="proto:urn:xmpp:community:0" id="root">
          <channel id="0000" type="groupchat" jid="welcome@muc.example.org" title="Welcome" />
          <group xmlns="proto:urn:xmpp:community:0" id="0001">
            <title>Development</title>
            <channel id="dev-0000" type="groupchat" jid="development@muc.example.org" title="Development" />
            <channel id="dev-0001" type="groupchat" jid="debugging@muc.example.org" title="Debugging" />
          </group>
          <group xmlns="proto:urn:xmpp:community:0" id="0002">
            <title>Support</title>
            <channel id="support-0000" type="groupchat" jid="support@muc.example.org" title="General Support" />
          </group>
        </group>
      </item>
    </items>
  </pubsub>
</iq>
```

# Info

| Key | Value |
| --- | ---   |
| Author | PapaTutuWawa |
| Version | 0.1.0 |
| Short name | cmt |
