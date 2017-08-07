# Rich Presence RPC-Only Implementation

Discord's Rich Presence feature is designed as an obfuscated addition to our existing [RPC infrastructure](https://discordapp.com/developers/docs/topics/rpc). The standalone library and header files make it easy for any dev to drop it into their game.

Our library communicates with Discord over the local Discord RPC socket. We've already done the work in connecting properly, handling disconnects and reconnects, and other RPC intracacies, but those who have done this implementation for our private alpha Voice and Chat SDK can simply make use of the new RPC commands and events to implement Rich Presence.

## General Notes

By committing to an RPC-only integration, you decide to forego the work our library and header file have done for you in the way of error handling, state storage, disconnecting and reconnecting, and other quality of life abstractions. While simply implementing the new RPC command and events will enable Rich Presence for your game, we highly suggest that you do your best to mimic the functionality of the SDK the most that you can. It ensure not only code quality on your part, but also an excellent experience on the part of your players.

## Application Protocol Registration

One thing that cannot be explicitly done over RPC is registering an application protocol for your game. If you choose to do an RPC-only implementation, you will have to register your application protocol yourself in the format of `discord-[your_app_id]://`. You can use `Discord_Register()` from `src/discord-register.cpp` as a good example of how to properly register an application protocol for use with Discord.

## New RPC Command

The new RPC command for Rich Presence is `SET_ACTIVITY`. The fields are similar to what is outlined in the SDK; we've combined similar fields into objects for the sake of less data on the wire.

The one major difference is the `party.size` field. It is an array with a size of two. The first element is the current party size, `partySize` from the main documentation. The second element is the maximum party size, `partyMax` from the main documentation.

Below is a full example of a `SET_ACTIVITY` command. Field restrictions like size are the same as outlined in the main documentation.

```json
{
  "cmd": "SET_ACTIVITY",
  "data": {
    "presence": {
      "state": "In a Group",
      "details": "Competitive | In a Match",
      "timestamps": {
        "start": time(nullptr),
        "end": time(nullptr) + ((60 * 5) + 23)
      },
      "assets": {
        "large_image": "numbani_map",
        "large_text": "Numbani",
        "small_image": "pharah_profile",
        "small_text": "Pharah"
      },
      "party": {
        "id": GameEngine.GetPartyId(),
        "size": [3, 6]
      },
      "secrets": {
        "join": "025ed05c71f639de8bfaa0d679d7c94b2fdce12f",
        "spectate": "e7eb30d2ee025ed05c71ea495f770b76454ee4e0",
        "match": "4b2fdce12f639de8bfa7e3591b71a0d679d7c93f"
      },
      "instance": true
    }
  },
  "nonce": "647d814a-4cf8-4fbb-948f-898abd24f55b"
}
```

## New RPC Events

The two new RPC events for Rich Presence power the ability to join and spectate your friends' games. First is the `GAME_JOIN` event:

```json
{
  "cmd": "DISPATCH",
  "data": {
    "secret": "025ed05c71f639de8bfaa0d679d7c94b2fdce12f"
  },
  "evnt": "GAME_JOIN"
}
```

And second is the `GAME_SPECTATE` event:

```json
{
  "cmd": "DISPATCH",
  "data": {
    "secret": "e7eb30d2ee025ed05c71ea495f770b76454ee4e0"
  },
  "evnt": "GAME_SPECTATE"
}
```