---
title: Match delivery
sections:
  - Polling
  - Websocket
---

Matchmore comes with a variety of push channels to deliver *matches*. They are integrated into our *SDKs*.

## Polling

You can periodically call the [GetMatches API call](https://matchmore.io/documentation/api#operation/getMatches) to retrieve the matches for a device.

## Websocket

WebSockets are available for use. You can create a connection on wss://api.matchmore.io/pusher/v5/ws/{deviceId}?key={apiKey}.

### Control Messages

The socket will periodically send a `PING` or `PONG` messages. 

These can be discarded or responded with respective `PONG` and `PING` messages.

### Matches

Matches come in a form of the match ID which is a [UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier). Whenever you get such messages, you can pass this to the SDKs `GetMatch` method or call the [endpoint on the API](https://matchmore.com/documentation/api#operation/getMatch).



 



