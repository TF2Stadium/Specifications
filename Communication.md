# Communication

The Server and Client communicate using the socket.io WebSockets protocol. All communication
will be in the form of JSON objects. Lobbies are managed server-side, with the
client sending appropriate messages to the server. For messages that require responses, the server returns a socket.io acknowledgement with the response data. The MVP client can request the server to:

* Create a Lobby.

* Close a Lobby.

* Add players to the lobby.

* Remove/Kick players from the lobby.

* Configure servers for the lobby, setup mumble channels.

* Send short text notifications that clients display.

In return, the Server can message the client to

* Ask players to "Ready Up".

* Connect players to the match server.

* Deliver information about running lobbies and their info (Scores, Time Left, etc)

## Messages

### Server

* [`lobbyCreate`](#lobbycreate) - Creates an lobby.

* [`lobbyClose`](#lobbyclose) - Closes a lobby.

* [`lobbyJoin`](#lobbyjoin) - Adds a player to a lobby.

* [`lobbyJoinSpectator`](#lobbyjoinspectator) - Adds a player to a lobby as a spectator

* [`lobbyRemovePlayer`](#lobbyremoveplayer) - Removes a player from a lobby

* [`playerReady`](#playerready) - Readies the authenticated player.

* [`playerUnready`](#playerunready) - Unreadies the authenticated player.

### Client

* [`chatReceive`](#chatreceive) - Sends the client a chat message

* [`lobbyListData`](#lobbylistdata) - Sends the client the list of lobbies that haven't yet started

* [`lobbyData`](#lobbydata) - Sends the client data about a specific lobby. Only sent if the player is in that lobby.

* [`lobbyReadyUp`](#lobbyreadyup) - Asks a player in a lobby to ready up.

* [`lobbyStart`](#lobbystart) - Notifies a player in a lobby that it started

* [`sendNotification`](#sendnotification) - Sends a short toast notification to a client

## Request format

Requests are sent over socket.io's `emit`, with the event name being the request's name. Request
parameters are sent as an additional JSON argument to `emit
`

Lobby types can be one of `["hl", "6s"]`. This list will be expanded when the project supports more lobby types.

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

## Constants

In the following message types

* `team` is in `['red', 'blu']`
* `type` is in `['sixes', 'highlander']`
* `class` is in
  * if `type == 'sixes'` : [`scout1`, `scout2`, `roamer`, `pocket`, `demoman`, `medic`]
  * if `type == 'highlander'` : `['scout', 'soldier', 'pyro', 'demoman', 'heavy', 'engineer', 'medic', 'sniper', 'spy']`

## Authentication

Client's authentication is verified automatically when the socket.io connection is established, so all the client has to do is be logged into the website the normal http way.

## Server Requests

These requests are sent to the Server.

### lobbyCreate

* `mapName`

* `type` - a `type` constant

* `whitelist` - whitelist ID from [whitelist.tf](http://whitelist.tf/)

* `server` - Server address, with port (`example.org:1234`)

* `rconpwd` - password to the server's RCON

* `mumbleRequired` - true if mumble required for lobby, else false

Returns `id`, the Lobby ID

### lobbyClose

* `id` - Lobby ID

### lobbyJoin

Joins a lobby as a player

* `id` - ID of the lobby the client wants to join

* `team` - a `team` constant

* `class` - a `class` constant

If the player has already joined a lobby, and `lobbyid` is the player's current
lobby, the player's current team/slot will be changed accordingly (provided that
the chosen slot is also empty).

### lobbyJoinSpectator

Joins a lobby as a spectator (in the website, unrelated to tf2 spectating)

* `id` - ID of the lobby the client wants to join


### lobbyRemovePlayer

Removes a player or a spectator from a lobby.

* `steamid` - (optional) ID of the player to kick from a lobby. If empty, removes the authenticated player from their current lobby. If provided, requires admin privileges.

* `ban` - true if player is to be banned from the lobby, else false

### playerReady

Readies the player in the lobby they are in.

No parameters.

### playerUnready

Unreadies the player in the lobby they are in.

No parameters.

### chatSend

* `message` - message string

* `room` - room to which the message should go. If empty or less than 0, the message is sent to the lobby list view chat.

## Client Requests

These requests are sent to the Client.

### chatReceive

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

### lobbyListData

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


### lobbyData

TODO

### lobbyReadyUp
Asks the players to send `playerReady` messages

* `time` - timestamp value

### lobbyStart

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

### sendNotification

* `message` - the text of a notification. 140 characters max.
