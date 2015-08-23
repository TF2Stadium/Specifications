# Administration system
Each player has an associated role. The roles are currently `roles = ['player', 'moderator', 'administrator']`, where `player` is the standard user role that every user starts with, `administrator` is the most powerful role that is allowed to perform anything, and `moderator` is a moderator role that can perform some administration actions.

## Ban system
There are four types of bans:

`banType = ['join', 'create', 'chat', 'full']`

They prevent a player from joining lobbies, creating lobbies, participating in any chats, and logging in to the site respectively.

## toServer requests

### adminChangeRole
Role required: `['moderator', 'administrator']`

* `steamid` - string - user's steam id
* `role` - string - any `roles` constant except `'administrator'` (that role can only be set manually)

Changes the role of a user. The change takes effect immediately.

### adminBanPlayer
Role required: `['moderator', 'administrator']`

* `steamid` - string - user's steam id
* `type` - string - a `banType` constant
* `reason` - string - the reason for a ban
* `until` - int - a Unix timestamp that represents when the ban will _expire_.

Assigns a timed ban of type `type` to a player. Permanent bans should just set a really big `until` value.

### adminUnbanPlayer

Role required: `['moderator', 'administrator']`

* `steamid` - string - user's steam id
* `type` - string - a `banType` constant

Inactivates all player's active bans of type `type`.