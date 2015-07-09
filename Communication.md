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

In return, the Server can request the client to

* Ask players to "Ready Up".

* Connect players to the match server.

Additionally, the server queries the match server to get the game status, which
is sent to the client to convey match details (Scores, Time Left, etc) to players.

# Server Requests

A request is a JSON object, having a mandatory field `request`. `request` can be one of:

* `createLobby` - Creates an lobby.

* `closeLobby` - Closes a lobby. 

* `addPlayer` - Adds a player to a lobby.

* `removePlayer` - Removes a player from a lobby

* `startMatch` - Creates the appropriate mumble channel, configures the server with the correct match, whitelist, etc.

* `askReady` - Ask all players for a lobby to ready up.

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

* `steamid` - the player's steam ID

* `name` - the player's name

* `team` - `0` for RED, `1` for BLU

* `slot` - class slot.

### `removePlayer`

* `steamid` - the player's steam ID

### `startMatch`

* `id` - Lobby ID

## Client Requests

These requests are sent to the Client.

### `askReady`

* `id` - Lobby ID
