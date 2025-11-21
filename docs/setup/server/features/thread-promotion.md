# Thread Promotion

## Overview

Thread Promotion is an Anticensor feature that allows converting active threads into top-level guild channels. This is useful when a discussion in a thread becomes important enough to warrant its own permanent channel, or when organizing community structure dynamically based on activity.

## Use Cases

1. **Organic Community Growth**: A popular thread about a specific topic can be promoted to a dedicated channel
2. **Event Channels**: Temporary event threads can be promoted to permanent channels if the event becomes recurring
3. **Project Channels**: Discussion threads about projects can be promoted when the project becomes official
4. **Topic Organization**: Reorganize channel structure based on actual usage patterns

## API Endpoint

```
POST /channels/:channel_id/promote
```

Promotes a thread to a top-level channel within its guild.

### Request Body

```json
{
	"position": 5
}
```

**Parameters:**

- `position` (optional, number): The position where the promoted channel should be inserted in the guild's channel list. If omitted, the channel will be placed after its former parent channel.

### Permissions

Requires the `MANAGE_CHANNELS` permission in the guild.

### Response

Returns the updated channel object with its new type and position.

```json
{
	"id": "123456789012345678",
	"type": 0,
	"guild_id": "987654321098765432",
	"name": "promoted-discussion",
	"parent_id": null,
	"position": 5,
	"permission_overwrites": [
		{
			"id": "role_id",
			"type": 0,
			"allow": "1024",
			"deny": "2048"
		}
	]
}
```

## Supported Thread Types

The following thread types can be promoted:

| Thread Type                | Promotes To      | Description                                       |
| -------------------------- | ---------------- | ------------------------------------------------- |
| `GUILD_PUBLIC_THREAD` (11) | `GUILD_TEXT` (0) | Public threads become text channels               |
| `GUILD_NEWS_THREAD` (10)   | `GUILD_NEWS` (5) | Announcement threads become announcement channels |
| `ENCRYPTED_THREAD` (13)    | `ENCRYPTED` (14) | Encrypted threads become encrypted channels       |

### Unsupported Thread Types

- **`GUILD_PRIVATE_THREAD` (12)**: Private threads **cannot be promoted** and will return a 400 error. This is by design, as private threads have restricted membership that doesn't translate well to channel permissions.

## Permission Continuity

One of the most important aspects of thread promotion is maintaining permission continuity. The promotion process ensures that users who could access the thread can still access the promoted channel.

### How Permissions Are Preserved

1. **Role-Based Permissions**: For each role in the guild, the system calculates what permissions that role had in the thread (considering both base role permissions and thread overwrites)

2. **Permission Overwrites**: If a role's effective permissions in the thread differ from its base permissions, a permission overwrite is created on the promoted channel to maintain the same access level

3. **User-Specific Overwrites**: Any user-specific permission overwrites from the thread are preserved on the promoted channel

### Permission Calculation Example

If a role has base permissions `VIEW_CHANNEL | SEND_MESSAGES` (permissions: 3072) but the thread had an overwrite denying `SEND_MESSAGES`, the promoted channel will have an overwrite with:

- `allow`: 0 (no additional permissions)
- `deny`: 2048 (deny SEND_MESSAGES)

This ensures the role can still view the channel but cannot send messages, exactly as in the original thread.

## Channel Positioning

### Default Positioning

If no `position` is specified in the request, the promoted channel is placed immediately after its former parent channel in the guild's channel list. This keeps related channels together and maintains logical organization.

### Custom Positioning

You can specify an exact position using the `position` parameter:

```json
{
	"position": 0
}
```

This will insert the promoted channel at the top of the channel list (position 0), pushing other channels down.

### Position Calculation

After promotion, the channel's position is recalculated using `Channel.calculatePosition()` to ensure consistency with the guild's channel ordering system.

## Channel Ordering

Anticensor uses a channel ordering system where guilds maintain an explicit order of their channels. When a thread is promoted:

1. The channel is removed from being a child of its parent
2. It's inserted into the guild's top-level channel order at the specified position
3. The `parent_id` is set to `null`
4. All other channels' positions are adjusted accordingly

## Gateway Events

After successful promotion, a `CHANNEL_UPDATE` event is emitted to all guild members:

```json
{
  "t": "CHANNEL_UPDATE",
  "d": {
    "id": "123456789012345678",
    "type": 0,
    "guild_id": "987654321098765432",
    "name": "promoted-discussion",
    "parent_id": null,
    "position": 5,
    "permission_overwrites": [...]
  }
}
```

Clients should handle this event to update their channel lists and move the channel from the thread list to the main channel list.

## Error Responses

| Status Code | Error                                    | Description                                             |
| ----------- | ---------------------------------------- | ------------------------------------------------------- |
| 400         | "Only guild threads can be promoted"     | The channel is not in a guild (e.g., DM channel)        |
| 400         | "Channel is not a thread"                | The channel has no parent (already a top-level channel) |
| 400         | "Private threads cannot be promoted"     | Attempted to promote a private thread                   |
| 400         | "Unsupported channel type for promotion" | The channel type is not supported for promotion         |
| 403         | Forbidden                                | Missing `MANAGE_CHANNELS` permission                    |
| 404         | Not Found                                | Channel does not exist                                  |

## Implementation Details

### Thread Metadata

When a thread is promoted, thread-specific metadata is preserved:

- Thread name becomes the channel name
- Thread topic (if any) becomes the channel topic
- Thread rate limit settings are preserved

### Message History

All messages in the thread remain accessible after promotion. The message history is preserved and continues to be available in the promoted channel.

### Member List

Thread member tracking is removed when promoting to a channel, as channels don't have explicit member lists (they use permission-based access instead).

## Best Practices

1. **Communicate Before Promoting**: Notify thread participants before promoting to avoid confusion
2. **Review Permissions**: Check the resulting permission overwrites to ensure they match your intent
3. **Consider Alternatives**: Sometimes creating a new channel and linking to the thread is better than promotion
4. **Archive First**: Consider archiving the thread before promotion if you want to preserve its "thread" state
5. **Update Channel Settings**: After promotion, you may want to adjust the channel's topic, slowmode, or other settings

## Example Usage

### Basic Promotion

```javascript
// Promote a thread to a channel at its default position
const response = await fetch("/channels/123456789012345678/promote", {
	method: "POST",
	headers: {
		Authorization: "Bearer YOUR_TOKEN",
		"Content-Type": "application/json",
	},
	body: JSON.stringify({}),
});

const promotedChannel = await response.json();
console.log(
	`Thread promoted to channel at position ${promotedChannel.position}`,
);
```

### Promotion with Custom Position

```javascript
// Promote a thread and place it at the top of the channel list
const response = await fetch("/channels/123456789012345678/promote", {
	method: "POST",
	headers: {
		Authorization: "Bearer YOUR_TOKEN",
		"Content-Type": "application/json",
	},
	body: JSON.stringify({
		position: 0,
	}),
});
```

### Error Handling

```javascript
try {
	const response = await fetch("/channels/123456789012345678/promote", {
		method: "POST",
		headers: {
			Authorization: "Bearer YOUR_TOKEN",
			"Content-Type": "application/json",
		},
		body: JSON.stringify({ position: 5 }),
	});

	if (!response.ok) {
		const error = await response.json();
		if (
			response.status === 400 &&
			error.message.includes("Private threads")
		) {
			console.error("Cannot promote private threads");
		} else if (response.status === 403) {
			console.error("Missing MANAGE_CHANNELS permission");
		}
		return;
	}

	const promotedChannel = await response.json();
	console.log("Thread promoted successfully:", promotedChannel);
} catch (error) {
	console.error("Failed to promote thread:", error);
}
```

## Related Features

- [Threads](../../threads.md) - Thread creation and management
- [Permissions](../security/permissions.md) - Permission system details
- [Channel Types](../../channel-types.md) - Different channel types in Anticensor

## Comparison with Discord

Discord does not have a thread promotion feature. This is an Anticensor-specific enhancement that provides more flexibility in community organization and structure evolution.
