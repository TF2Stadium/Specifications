# Communication

The Server and Client communicate using the [wsevent](https://github.com/TF2Stadium/wsevent) library over WebSockets. All communication will be in the form of JSON objects. Lobbies are managed server-side, with the client sending appropriate messages to the server. The server may directly reply to these messages, and may also send unsolicited messages to the client.

A client can request the server to:

* Create a Lobby.

* Close a Lobby.

* Join a slot in a lobby.

* Remove/Kick players from a lobby. (with proper permissions)

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

* [`lobbySpectatorJoin`](#lobbyspectatorjoin) - Adds a player to a lobby as a spectator

* [`lobbyRemovePlayer`](#lobbyremoveplayer) - Removes a player from a lobby

* [`playerReady`](#playerready) - Readies the authenticated player.

* [`playerUnready`](#playerunready) - Unreadies the authenticated player.

* [`chatSend`](#chatsend) - Sends a message to the chat.

* [`playerSettingsGet`](#playersettingsget) - Gets player's settings.

* [`playerSettingsSet`](#playersettingsset) - Sets a player setting.

### Client

* [`chatReceive`](#chatreceive) - Sends the client a chat message

* [`lobbyListData`](#lobbylistdata) - Sends the client the list of lobbies that haven't yet started

* [`lobbyData`](#lobbydata) - Sends the client data about a specific lobby. Only sent if the player is in that lobby.

* [`lobbyReadyUp`](#lobbyreadyup) - Asks a player in a lobby to ready up.

* [`lobbyStart`](#lobbystart) - Notifies a player in a lobby that it started

* [`sendNotification`](#sendnotification) - Sends a short toast notification to a client

## Request format

Requests are sent over wsevent's `Emit`, with the event name being the request's name. Request parameters are sent as an additional JSON argument.

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
* `state` (player state) is in `['ready', 'notReady']`
* `lobbyState` is in `['initializing, waiting, inProgress, ended']`
* `class` is in
  * if `type == 'sixes'` : [`scout1`, `scout2`, `roamer`, `pocket`, `demoman`, `medic`]
  * if `type == 'highlander'` : `['scout', 'soldier', 'pyro', 'demoman', 'heavy', 'engineer', 'medic', 'sniper', 'spy']`

## Authentication

Client's authentication is verified automatically when the connection is established, so all the client has to do is be logged into the website the normal http way.

## Server Requests

These requests are sent to the Server.

### lobbyCreate

* `mapName`

* `type` - string - a `type` constant

* `whitelist` - integer - whitelist ID from [whitelist.tf](http://whitelist.tf/)

* `server` - string - Server address, with port (`example.org:1234`)

* `rconpwd` - string - password to the server's RCON

* `mumbleRequired` - bool - true if mumble required for lobby, else false

Returns `id`, the Lobby ID. The created lobby is in state `initializing`, and the player starts receiving its `lobbyData` notifications. If server initialization succeeds, the server's type changes to `waiting`. If it fails, the type changes to `ended` and a notification (`sendNotification`) is sent.

### lobbyClose

* `id` - integer - Lobby ID

### lobbyJoin

Joins a lobby as a player

* `id` - integer - ID of the lobby the client wants to join

* `team` - string - a `team` constant

* `class` - string - a `class` constant

If the player has already joined a lobby, and `lobbyid` is the player's current
lobby, the player's current team/slot will be changed accordingly (provided that
the chosen slot is also empty).

### lobbySpectatorJoin

Joins a lobby as a spectator (in the website, unrelated to tf2 spectating)

* `id` - integer - ID of the lobby the client wants to join


### lobbyRemovePlayer

Removes a player or a spectator from a lobby.

* `id` - int - lobby id.

* `steamid` - string - (optional) ID of the player to kick from a lobby. If empty, removes the authenticated player from their current lobby. If provided, requires admin privileges.

* `ban` - boolean - true if player is to be banned from the lobby, else false


### playerReady

Readies the player in the lobby they are in.

No parameters.

### playerUnready

Unreadies the player in the lobby they are in.

No parameters.

### chatSend

* `message` - string -  message string

* `room` - string - room to which the message should go. Each lobby has a chat room with a name 'lobby_id'. Otherwise, if the `room` parameter is not an integer string, the message will be sent to the lobby list menu chat.

### playerSettingsGet

* `key` - string - optional

Returns a player setting json object where each key holds a value string. If `key` is provided, only that key and its value is returned (the object will have one key). Otherwise, every player settings entry is returned (the object will have many keys)

### playerSettingsSet

* `key` - string
* `value` - string

Sets or updates a setting identified by `key` to hold value `value`

### playerProfile

* `steamid` - string - the steamid of a player

Returns player data in a format
```
{
  stats:
    {
      playedHighlanderCount: integer,
      playedSixesCount: integer,
    }
  
	createdAt: integer // unix seconds
	gameHours: integer
  
	id: integer
	steamid: string
	avatar: string // the avatar url	
	name: string
  
  bans: [{ // only contains active bans
    type: string // banType constant, in Admin.md
    until: integer  // unix seconds
    }]
}
```

## Client Requests

These requests are sent to the Client.

### chatReceive

* `createdAt` - integer - timestamp of message creation. Unix timestamp in seconds.

* `message` - string - message string

* `room` - string - room which the message should go to. See `#chatSend`.  

* `user` - json map with the fields:
```
{
	id: string   //steamid of player
	name: string //name of player
}
```

* `colorize` - string - hex code denoting color for the message text, e.g. "#AB11EF"

### lobbyListData

Returns a list of `LobbySummary`s, where a LobbySummary object is

```
{
  `id`: integer //lobby numeric ID, incremental, server-side generated

  `type`: string // lobby type. `type` constant.

  `createdAt`: integer // timestamp of lobby creation. Unix timestamp in seconds.

  `whitelist`: integer // whitelist ID from whitelist.tf

  `players`: integer // number of players currently in the lobby

  `owner`: {
      `id`: string // owner steamid
      `name`: string // owner name
  }

  `state`: string // state of lobby. A `lobbyState` constant

  `classes`: //JSON array in the following format:
  [
  	<class constant>: {
  		'red' : {
    		'id': string //steamid. If "", slot is unoccupied.
    		'name': string //player name. If "", slot is unoccupied.
    		'state': string //player state constant, string
  		},

  		'blu': {
    		'id': string
    		'name': string
    		'state': string
  		}
  	}
  ]
}
```


### lobbyData

TODO

### lobbyReadyUp
Asks the players to send `playerReady` messages

* `time` - integer - timestamp value, Unix timestamp in seconds

### lobbyStart

* `id` - integer - lobby ID, integer

* `time` - integer - timestamp value, Unix timestamp in seconds

* `game` - a json object with the following fields:
```
{
	host: 'example.org:xyz'
}
```

* `mumble` - a json object with the following fields:
```
{
	'ip': "x.x.x.x" //string
	'port': "1234" //string
	'channel': "match13" //string
}
```

### sendNotification

* `message` - string - the text of a notification. 140 characters max.
