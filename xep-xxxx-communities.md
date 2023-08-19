# Communities

In different instant messaging services, users have the options to create collections of group chat venues that are logically linked
together.

This XEP is designed in a modular way, similar to [Mediated Information Exchange](https://xmpp.org/extensions/xep-0369.html). As such, this XEP only provided the basics for such
groupings.

## Design Requirements

To aid adoption, the following requirements are considered while creating this XEP:

- Basic communities, i.e. grouped links to other JIDs with additional metadata, should work without server support
- The protocol should be extensible for future evolution

## Discovery

A community in XMPP is identified by a JID that "owns" the community. This may be a bare JID or a server"s JID. This JID must provide
a [PubSub](https://xmpp.org/extensions/xep-0060.html) service on which the community"s metadata is stored. This XEP proposes a couple of basic nodes, but other XEPs may extend
this set and provide custom extensions.

Whether a JID hosts a community is discovery by using a [Service Discovery](https://xmpp.org/extensions/xep-0030.html) items query:

```xml
<iq type="get"
    to="community.example.org"
    id="nodes1">
  <query xmlns="http://jabber.org/protocol/disco#items"/>
</iq>
```

The response MUST contain at least the `proto:urn:xmpp:community:0:info` node:

```xml
<iq type="result"
    from="community.example.org"
    id="nodes1">
  <query xmlns="http://jabber.org/protocol/disco#items">
    <item jid="community.example.org" node="proto:urn:xmpp:community:0:info" />
  </query>
</iq>
```

## Basics

A community may be either public, meaning that anyone who knows the JID may join and query information about, or private, meaning that users
wishing to join MUST be added by an administrator of the community. To accomplish this, public communities SHOULD set all their [PubSub](#) nodes
to have an access model of "open", while private communities SHOULD set all of their nodes to "whitelist". A future XEP may define a way for
administrators to pre-authenticate users such that "invite URLs" to users.

### Description

A community"s information is described by a `<information />` element containing information about the community.

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

```xml
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

## State Synchronization (Maybe new XEP?)

There may be some state that should be synchronized across different channels, like ban lists, ACLs, hats, and so on. To accomplish this, XEPs may define
new [PubSub](#) nodes that supporting channels within a community should subscribe to. Upon receiving a [PubSub](#) notification, the channels should
react appropriately in regard to the payload.

Care must be taken when storing bare or full JIDs in these [PubSub](#) nodes as to not leaking them. Thus, The recommendations of
[Persistent Storage of Private Data via PubSub](https://xmpp.org/extensions/xep-0223.html) apply.

### Linking

Each channel must be configured to listen to [PubSub](#) events for every bit of synchronized state it cares about. The actual mechanism for it is out of scope,
but for MUC, it may include a field in the configuration data form that refers to the community's JID.

Example:

```xml
<iq from='crone1@shakespeare.lit/desktop'
    id='create2'
    to='coven@chat.shakespeare.lit'
    type='set'>
  <query xmlns='http://jabber.org/protocol/muc#owner'>
    <x xmlns='jabber:x:data' type='submit'>
      <field var='FORM_TYPE'>
        <value>http://jabber.org/protocol/muc#roomconfig</value>
      </field>
      <field var='muc#community_link'>
        <value>community.example.org</value>
      </field>
    </x>
  </query>
</iq>
```

### Banning

Every channel inside a space that supports banning users, should be configured to subscribe to the `proto:urn:xmpp:community:0:ban` [PubSub](#) node. When a JID is put there,
the supporting channels will be notified of the new ban and SHOULD apply it locally. This node MUST be configured with an access model of whitelist such that only administrators
can see the bare JIDs of banned users.

Banning a JID:

```xml
<iq type="set" to="community.example.org">
  <pubsub xmlns="http://jabber.org/protocol/pubsub">
    <publish node="proto:urn:xmpp:community:0:ban">
      <item id="malicious-user@example.org" />
    </publish>
  </pubsub>
</iq>
```

### Power Levels

Every channel inside a space that supports power levels, like "Moderator", "Admin", and so on, should subscribe to the `proto:urn:xmpp:community:0:power` [PubSub](#) node. Each item in it
describes a bare JID and its power level. When a notification is received, supporting channels SHOULD keep their internal power levels in sync with the payload.

Making a JID an administrator:

```xml
<iq type="set" to="community.example.org">
  <pubsub xmlns="http://jabber.org/protocol/pubsub">
    <publish node="proto:urn:xmpp:community:0:power">
      <item id="admin@example.org">
        <power xmlns="proto:urn:xmpp:community:0:power" level="admin" />
      </item>
    </publish>
  </pubsub>
</iq>
```

# Notes

- Using just this XEP, one can implement Gajim-like workspaces: Create a private community on your own bare JID and create a channel group per workspace.
  - For pinning, each pinned `<channel />` could get a `<pinned />` child that indicates that it is pinned. The pinned items could then either have a different sorting or the same.

# Info

| Key | Value |
| --- | ---   |
| Author | PapaTutuWawa |
| Version | 0.1.0 |
| Short name | cmt |
