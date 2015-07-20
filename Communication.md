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

* `lobbyCreate` - Creates an lobby.

* `lobbyClose` - Closes a lobby.

* `lobbyJoin` - Adds a player to a lobby.

* `lobbyKickPlayer` - Removes a player from a lobby

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

### `lobbyCreate`

* `mapName`

* `format` - sixes, HL (lol), etc

* `whitelist` - whitelist ID from [whitelist.tf](http://whitelist.tf/)

* `server` - Server address, with port (`example.org:1234`)

* `rconpwd` - password to the server's RCON

* `mumbleRequired` - true if mumble required for lobby, else false

Returns `id`, the Lobby ID

### `lobbyClose`

* `id` - Lobby ID

### `lobbyJoin`

* `lobbyid` - ID of the lobby the client wants to join

* `team` - (optional) `0` for RED, `1` for BLU

* `slot` - (optional) class slot. 0-5 in 6s, 0-9 in hl, in the usual order.

If the player has already joined a lobby, and `lobbyid` is the player's current
lobby, the player's current team/slot will be changed accordingly. (Provided that
the chosen slot is also empty)

### `lobbyRemovePlayer`

* `steamid` - (optional) ID of the player to kick from a lobby. If empty, kicks the authenticated player. If provided, requires admin privileges.

* `lobbyid` - ID of the lobby the client wants to leave

* `ban` - true if player is to be banned, else false

### `chatSend`:

* `message` - message string

* `room` - room to which the message should go

## Client Requests

These requests are sent to the Client.

### `chatReceive`:

* `createdAt` - timestamp of message creation

* `message` - message string

* `room` - room to which the message should go

* `user` - json map with the fields:
```
{
	id: ...   //steamid of player
	name: ... //name of player
}
```

* `colorize` - hex code denoting color for the message text

### `lobbyListData`

* `id` - lobby numeric ID, incremental, server-side generated

* `format` - lobby format

* `createdAt` - string, timestamp of lobby creation

* `whitelist` - whitelist ID from whitelist.tf

* `players` - number of players currently in the lobby

* `owner` - a JSON map with the fields `id` (steamid), and `name`

* `state` - state of lobby

* `classes` - JSON array in the following format:
```
[
	'class_name': { //if id == "", then slot if empty, else taken
		'red' : {
		'id': 'xxx'      //steamid string
		'name': 'foo'    //player name
		'state': "ready" //or unready
		},
		
		'blu': {
		'id': 'xyz'
		'name': 'foo'
		'state': "unready"
		}
	}
	...
]
```

### `lobbyReady`:

* `id` - lobby ID, integer

* `time` - timestamp value

# `lobbyStart`

* `id` - lobby ID, integer

* `time` - timestamp value

* `game` - a json object with the following fields:
```
{
	'ip': "x.x.x.x" //string
	'port': 1233 //integer
	'password': "lobbypass"
}
```

* `mumble` - a json object with the following fields:
```
{
	'ip': "x.x.x.x"
	'port': 1234
	'channel': "match13"
	'password': "foo"
}
```

### `sendNotification`

* `message` - the text of a notification. 140 characters max.

