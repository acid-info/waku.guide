---
title: Network Interaction Domains
---

Waku is a unified and cohesive entity that offers a rich ecosystem with three distinct network interaction domains. These domains serve specialized purposes and contribute to the robust functionality of Waku, forming its foundation.

## Discovery Domain

Node discovery in Waku facilitates locating other nodes within the network. As a modular protocol, Waku incorporates various discovery mechanisms, such as [`Discv5`](/overview/concepts/node-discovery#discv5) and [`Peer Exchange`](/overview/concepts/node-discovery#peer-exchange). These mechanisms allow developers to choose the most suitable option(s) for their specific use cases and user environments, including mobile phones, desktop browsers, servers, and more.

## Gossip Domain

Gossipsub derives its name from the practice within Pub/Sub networks where peers gossip about the messages they have encountered, thus establishing a message delivery network.

Waku employs gossiping through [`Waku Relay`](/overview/concepts/protocols#waku-relay) to distribute messages across the network. Additionally, Waku introduces [`Waku RLN Relay`](/overview/concepts/protocols#waku-rln-relay), an experimental mechanism that combines privacy preservation and economic spam protection.

## Request/Reply Domain

Waku provides a set of Request/Reply protocols to optimize its performance in resource-restricted environments like low bandwidth or offline scenarios.

These protocols serve multiple purposes. [`Waku Store`](/overview/concepts/protocols#waku-store) enables the retrieval of historical messages, [`Waku Filter`](/overview/concepts/protocols#waku-filter) efficiently retrieves a subset of messages to conserve bandwidth, and [`Waku Light Push`](/overview/concepts/protocols#waku-light-push) facilitates message publication for nodes with limited bandwidth and short connection windows.

## Overview of Protocol Interaction

Here's a diagram illustrating the interaction between different protocols within the Waku network.

```mermaid
sequenceDiagram
participant A as A relay
participant B as B relay(pubtopic1)
participant C as C relay(pubtopic1)
participant D as D relay(pubtopic1), store(pubtopic1), filter
participant E as E relay, store
participant F as F filter

A ->> A: msg1=WakuMessage(contentTopic1, data) (1)
F ->> D: FilterRequest(pubtopic1, contentTopic1) (2)
D ->> D: Subscribe F to filter (2)
A ->> B: Publish msg1 on pubtopic1 (3)
B ->> D: relay msg1 on pubtopic1 (3)
D ->> D: store: saves msg1 (4)
D ->> C: relay msg1 on pubtopic1 (4)
D ->> F: MessagePush(msg1) (5)
E ->> E: E comes online (6)
E ->> D: HistoryQuery(pubtopic1, contentTopic1) (6)
D ->> E: HistoryResponse(msg1, ...) (6)
```

The Pub/Sub topic `pubtopic1` serves as a means of routing messages (the network employs a default pubsub topic) and indicates that it is subscribed to messages on that topic for a relay. In the context of Waku Store, Node D is responsible for persisting these messages.

1. Node A creates a WakuMessage `msg1` with a [ContentTopic](/overview/concepts/content-topics) `contentTopic1`.
2. Node F requests to get messages filtered by Pub/Sub topic `pubtopic1` and ContentTopic `contentTopic1`. Node D subscribes F to this filter and will forward messages that match that filter in the future.
3. Node A publishes `msg1` on `pubtopic1`. The message is sent from Node A to Node B and then forwarded to Node D.
4. Node D, upon receiving `msg1` both stores the message for future retrieval by other nodes and forwards it to Node C.
5. Node D also pushes `msg1` to Node F, informing it about the arrival of a new message.
6. At a later time, Node E comes online and requests messages matching `pubtopic1` and `contentTopic1` from Node D. Node D responds with `msg1` and potentially other messages that match the query.