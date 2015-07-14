# Communication

The Server and Client communicate using the WebSockets protocol. All communication
will be in the form of JSON objects. Lobbies are managed server-side, with the
client sending appropriate requests to the server. The MVP client can request the
server to:

* Create a Lobby.

* Close a Lobby.

* Add players to the lobby.

* Remove/Kick players from the lobby.

* Configure servers for the lobby, setup mumble channels.

* Send short text notifications that clients display.

In return, the Server can request the client to

* Ask players to "Ready Up".

* Connect players to the match server.

Additionally, the client queries the match server to get the game status, which
is sent to the client to convey match details (Scores, Time Left, etc) to players.

# Requests

* `createLobby` - Creates an lobby.

* `closeLobby` - Closes a lobby.

* `addPlayer` - Adds a player to a lobby.

* `removePlayer` - Removes a player from a lobby

* `startMatch` - Creates the appropriate mumble channel, configures the server with the correct match, whitelist, etc.

* `askReady` - Ask all players for a lobby to ready up.

# Request format

Requests are sent over socket.io's `emit`, with the event name being the request's name. Request
parameters are sent as an additional JSON argument to `emit
`
## Response format

If the request is successful, the returned object is
```
{
  success: true,
  data: ...
}
```

If the request is unsuccessful, the returned object is
```
{
  success: false,
  message: ...
  code: ...
}
```

`data`'s type varies by request sent.
## Server Requests

These requests are sent to the Server.

### `createLobby`

* `mapName`

* `format` - sixes, HL (lol), etc

* `whitelist` - whitelist ID from [whitelist.tf](http://whitelist.tf/)

* `server` - Server address, with port (`example.org:1234`)

* `rconpwd` - password to the server's RCON

Returns `id`, the Lobby ID

### `closeLobby`

* `id` - Lobby ID

### `addPlayer`

* `lobbyid` - ID of the lobby the client wants to join

* `team` - (optional) `0` for RED, `1` for BLU

* `slot` - (optional) class slot. 0-5 in 6s, 0-9 in hl, in the usual order.

If the player has already joined a lobby, and `lobbyid` is the player's current
lobby, the player's current team/slot will be changed accordingly. (Provided that
the chosen slot is also empty)

### `removePlayer`

* `steamid` - (optional) ID of the player to kick from a lobby. If empty, kicks the authenticated player. If provided, requires admin privileges.

* `lobbyid` - ID of the lobby the client wants to leave

## Client Requests

These requests are sent to the Client.

### `askReady`

### `startMatch`

### `sendNotification`

* `message` - the text of a notification. 140 characters max.

### `getLobbyList`

TODO

### `getLobbyDetails`

TODO
