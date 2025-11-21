# Discord Lobby System

## Overview

The Lobby system in Anticensor provides ephemeral, in-memory matchmaking and group coordination functionality similar to Discord's lobby feature. Lobbies are temporary gathering spaces that allow users to coordinate activities, find teammates, or organize groups before moving to permanent channels.

Unlike most other Anticensor features, lobbies are **not persisted to the database**. They exist only in memory and are automatically cleaned up when they expire or when the server restarts.

## Architecture

### In-Memory Storage

Lobbies are stored in the `LobbyStore`, an in-memory data structure that manages lobby lifecycle:

- **Creation**: Lobbies are created with a unique Snowflake ID
- **Expiration**: Lobbies automatically expire after a configurable idle timeout (default: 300 seconds)
- **Activity Tracking**: Each lobby access updates the last activity timestamp
- **Automatic Cleanup**: Expired lobbies are periodically removed from memory

### Lobby Structure

Each lobby contains:

- `id`: Unique Snowflake identifier
- `application_id`: The user ID of the lobby creator
- `metadata`: Key-value pairs for custom lobby information (max 1000 characters total)
- `members`: Array of lobby members (max 25 members)
- `idle_timeout_seconds`: Time in seconds before the lobby expires (5-604800 seconds)
- `linked_channel`: Optional channel ID that the lobby is associated with
- `created_at`: Timestamp of lobby creation
- `last_activity`: Timestamp of last lobby access

### Member Structure

Each lobby member has:

- `id`: User ID
- `metadata`: Member-specific metadata
- `flags`: Permission flags (bit 0 = owner/admin permissions)

## API Endpoints

### Create Lobby

```
POST /lobbies
```

Creates a new lobby with the specified configuration.

**Request Body:**

```json
{
	"metadata": {
		"game": "valorant",
		"rank": "gold",
		"region": "na-west"
	},
	"members": [
		{
			"id": "123456789012345678",
			"metadata": {
				"role": "duelist"
			},
			"flags": 1
		}
	],
	"idle_timeout_seconds": 600
}
```

**Validation Rules:**

- `metadata`: Total length of all keys and values cannot exceed 1000 characters
- `members`: Maximum 25 members
- `idle_timeout_seconds`: Must be between 5 and 604800 (7 days)

**Response:**

```json
{
	"id": "987654321098765432",
	"application_id": "123456789012345678",
	"metadata": {
		"game": "valorant",
		"rank": "gold",
		"region": "na-west"
	},
	"members": [
		{
			"id": "123456789012345678",
			"metadata": {
				"role": "duelist"
			},
			"flags": 1
		}
	],
	"idle_timeout_seconds": 600,
	"created_at": "2024-01-15T10:30:00.000Z"
}
```

### Get Lobby

```
GET /lobbies/:lobby_id
```

Retrieves information about a specific lobby. This operation updates the lobby's last activity timestamp, preventing it from expiring.

**Response:** Same format as create response

**Errors:**

- `404`: Lobby not found (may have expired or never existed)

### Update Lobby

```
PATCH /lobbies/:lobby_id
```

Updates lobby configuration. All fields are optional - only provided fields will be updated.

**Request Body:**

```json
{
	"metadata": {
		"game": "valorant",
		"rank": "platinum"
	},
	"members": [
		{
			"id": "123456789012345678",
			"metadata": {
				"role": "duelist"
			},
			"flags": 1
		},
		{
			"id": "234567890123456789",
			"metadata": {
				"role": "controller"
			},
			"flags": 0
		}
	],
	"idle_timeout_seconds": 900
}
```

**Validation:** Same rules as create endpoint

**Response:** Updated lobby object

### Delete Lobby

```
DELETE /lobbies/:lobby_id
```

Immediately deletes a lobby, removing it from memory.

**Response:** `204 No Content`

### Link Lobby to Channel

```
PATCH /lobbies/:lobby_id/channel-linking
```

Associates a lobby with a specific channel. This is useful for coordinating lobby members to join a voice or text channel.

**Request Body:**

```json
{
	"channel_id": "345678901234567890"
}
```

To unlink a channel, send an empty `channel_id` or omit it:

```json
{
	"channel_id": ""
}
```

**Permissions:**

- The requesting user must be a member of the lobby
- The requesting user must have the owner flag (bit 0 set in member flags)
- If linking to a guild channel, the user must be a member of that guild

**Response:** Updated lobby object with `linked_channel` field

### Leave Lobby

```
DELETE /lobbies/:lobby_id/members/@me
```

Removes the current user from the lobby.

**Response:** `204 No Content`

**Errors:**

- `404`: Lobby not found or user is not a member

## Use Cases

### Matchmaking

Lobbies can be used to implement matchmaking systems:

1. Create a lobby with metadata describing the game mode, skill level, etc.
2. Other users search for lobbies matching their criteria
3. Users join the lobby by having the lobby owner update the members list
4. Once full, the lobby owner links the lobby to a voice channel
5. All members join the linked channel to play together

### LFG (Looking for Group)

Users can create lobbies to find teammates:

```json
{
	"metadata": {
		"activity": "raid",
		"game": "destiny2",
		"experience": "experienced",
		"spots_available": "3"
	}
}
```

### Event Coordination

Organize temporary events or activities:

```json
{
	"metadata": {
		"event": "movie_night",
		"movie": "The Matrix",
		"start_time": "2024-01-15T20:00:00Z"
	},
	"idle_timeout_seconds": 7200
}
```

## Implementation Details

### Automatic Cleanup

The `LobbyStore` periodically checks for expired lobbies based on their `last_activity` timestamp and `idle_timeout_seconds`. When a lobby expires:

1. It is immediately removed from memory
2. No notification is sent to members
3. Any subsequent requests to the lobby will return 404

### Activity Updates

The following operations update a lobby's last activity timestamp:

- `GET /lobbies/:lobby_id`
- `PATCH /lobbies/:lobby_id`
- `PATCH /lobbies/:lobby_id/channel-linking`

Operations that do **not** update activity:

- `DELETE` operations (they remove the lobby entirely)

### Member Management

Member management is currently done by updating the entire members array via `PATCH /lobbies/:lobby_id`. There is no dedicated endpoint for adding/removing individual members (except for leaving via `DELETE /lobbies/:lobby_id/members/@me`).

To add a member:

```json
{
  "members": [
    ...existing_members,
    {
      "id": "new_member_id",
      "metadata": {},
      "flags": 0
    }
  ]
}
```

### Metadata Best Practices

Lobby metadata is flexible and can store any key-value pairs. Recommended practices:

1. **Keep it Small**: Total length limit is 1000 characters
2. **Use Consistent Keys**: Define standard keys for your application (e.g., `game`, `mode`, `region`)
3. **Store Searchable Data**: Include information users might filter by
4. **Avoid Sensitive Data**: Lobbies are not encrypted and may be visible to other users
5. **Use Member Metadata**: Store user-specific information in member metadata rather than lobby metadata

## Limitations

1. **No Persistence**: Lobbies are lost on server restart
2. **No Cross-Server**: Lobbies exist only on the server instance that created them
3. **No Search API**: There is no built-in endpoint to search or list lobbies (must be implemented separately)
4. **No Events**: Lobby changes do not emit gateway events to members
5. **Manual Member Management**: No automatic invitation or join request system

## Server Restart Considerations

Since lobbies are stored in memory:

- **All lobbies are lost** when the server restarts
- Applications should handle 404 errors gracefully
- Consider implementing a reconnection strategy for critical lobbies
- For persistent coordination, use guild channels instead

## Comparison with Channels

| Feature        | Lobbies                   | Channels                 |
| -------------- | ------------------------- | ------------------------ |
| Persistence    | In-memory only            | Database-backed          |
| Lifetime       | Temporary (expires)       | Permanent                |
| Member Limit   | 25                        | Configurable (thousands) |
| Messages       | No                        | Yes                      |
| Permissions    | Simple flags              | Full permission system   |
| Gateway Events | No                        | Yes                      |
| Use Case       | Matchmaking, coordination | Communication, community |

## Related Features

- [Channels](../../channels.md) - Permanent communication channels
- [Voice Channels](../voice.md) - Voice communication that lobbies can link to
- [Guild Features](../configuration/guildFeatures.md) - Guild-level features

## Example: Complete Matchmaking Flow

```javascript
// 1. Create a lobby
const lobby = await fetch("/lobbies", {
	method: "POST",
	body: JSON.stringify({
		metadata: {
			game: "valorant",
			mode: "competitive",
			rank: "gold",
		},
		members: [
			{
				id: currentUserId,
				flags: 1, // Owner
			},
		],
		idle_timeout_seconds: 600,
	}),
});

// 2. Other users find and join the lobby
// (Search implementation not shown - would need custom endpoint)

// 3. Owner updates members list as users join
await fetch(`/lobbies/${lobby.id}`, {
	method: "PATCH",
	body: JSON.stringify({
		members: [...existingMembers, { id: newUserId, flags: 0 }],
	}),
});

// 4. When full, link to voice channel
await fetch(`/lobbies/${lobby.id}/channel-linking`, {
	method: "PATCH",
	body: JSON.stringify({
		channel_id: voiceChannelId,
	}),
});

// 5. Members join the linked channel
// 6. Lobby expires after idle timeout or is manually deleted
await fetch(`/lobbies/${lobby.id}`, {
	method: "DELETE",
});
```
