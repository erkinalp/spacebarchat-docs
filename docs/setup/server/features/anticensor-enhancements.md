# Anticensor Feature Enhancements

!!! warning "Anticensor Extensions"
This document covers various enhancements and extensions that Anticensor provides beyond the standard Discord API. These features may not be compatible with standard Discord clients or libraries.

This document covers various enhancements and extensions that Anticensor provides beyond the standard Spacebar implementation.

## Slowmode (Arbitrary Seconds)

!!! info "Discord API Compatibility"
The Discord API supports `rate_limit_per_user` values from 0-21600 seconds. However, Discord.com's UI only exposes preset values (0, 5, 10, 15, 30 seconds, 1, 2, 5, 10 minutes, 1, 2, 6 hours). Anticensor allows setting arbitrary second values within the full API range, which is an extension beyond what Discord.com's interface provides. See [Discord's channel documentation](https://discord.com/developers/docs/resources/channel#channel-object-channel-structure) for the API specification.

### Overview

Anticensor supports setting channel slowmode (`rate_limit_per_user`) to any value between 0 and 21600 seconds (6 hours), while Discord.com's interface restricts users to specific preset values.

### API Usage

Set slowmode to any arbitrary value using the channel modification endpoint:

```http
PATCH /channels/:channel_id
Content-Type: application/json

{
  "rate_limit_per_user": 7
}
```

This sets a 7-second slowmode, which is not available through Discord.com's UI but is valid per the API specification.

### Supported Range

- **Minimum**: 0 seconds (no slowmode)
- **Maximum**: 21600 seconds (6 hours)
- **Granularity**: 1 second

### Discord.com Presets vs Anticensor

| Discord.com UI Presets | Anticensor Support |
| ---------------------- | ------------------ |
| 0, 5, 10, 15, 30s      | Any 0-21600s       |
| 1m, 2m, 5m, 10m        | Any 0-21600s       |
| 1h, 2h, 6h             | Any 0-21600s       |

### Use Cases

1. **Fine-Grained Rate Limiting**: Set precise slowmode values like 3 seconds or 45 seconds
2. **Custom Moderation**: Implement slowmode values tailored to specific channel needs
3. **Gradual Restrictions**: Incrementally adjust slowmode in response to activity levels
4. **Event Management**: Set specific slowmode values for timed events

!!! note "Client Compatibility"
Some Discord clients may not display arbitrary slowmode values correctly in their UI, as they expect only the preset values. However, the slowmode will still function correctly at the API level.

## PIN_MESSAGES Permission

!!! info "Discord Compatibility"
As of the latest version, Anticensor uses bit 51 for PIN_MESSAGES, matching Discord's official permission bit position (0x0008000000000000). This ensures full compatibility with Discord clients and libraries. See [Discord's official permissions documentation](https://discord.com/developers/docs/topics/permissions) for reference.

### Overview

Anticensor implements a dedicated `PIN_MESSAGES` permission at bit position 51, separate from the `MANAGE_MESSAGES` permission. This provides more granular control over who can pin messages in channels.

### Permission Details

- **Bit Position**: 51 (matches Discord standard)
- **Value**: `1 << 51` = `2251799813685248` (0x0008000000000000)
- **Scope**: Channel-level permission
- **Default**: Not included in default permissions

### Rationale

In Discord, pinning messages requires the `MANAGE_MESSAGES` permission, which also grants the ability to delete any message. Anticensor separates this functionality to allow trusted members to pin important messages without giving them full moderation powers.

### Use Cases

1. **Community Curators**: Allow members to pin helpful resources without moderation powers
2. **Event Organizers**: Let event coordinators pin announcements without message deletion rights
3. **FAQ Channels**: Enable helpers to pin common questions and answers
4. **Project Channels**: Allow project members to pin important updates

### Configuration

Grant the `PIN_MESSAGES` permission through role settings or channel overwrites:

```json
{
	"id": "role_id",
	"type": 0,
	"allow": "2251799813685248",
	"deny": "0"
}
```

### Comparison with MANAGE_MESSAGES

| Permission        | Pin Messages | Unpin Messages | Delete Any Message | Edit Any Message |
| ----------------- | ------------ | -------------- | ------------------ | ---------------- |
| `MANAGE_MESSAGES` | ✓            | ✓              | ✓                  | ✓                |
| `PIN_MESSAGES`    | ✓            | ✓              | ✗                  | ✗                |

### Rights System

The `PIN_MESSAGES` right also exists in the global rights system (bit 20) for instance-wide control over pinning capabilities.

## Doubly-Linked Replies

### Overview

Doubly-Linked Replies is an opt-in feature that allows messages to track both their parent (the message they're replying to) and their children (messages that reply to them). This enables building complete reply trees and conversation threads.

### Standard Discord Behavior

In Discord, messages only track their parent via `message_reference`:

- Message A
    - Message B (replies to A, knows about A)
    - Message C (replies to A, knows about A)

Message A doesn't know that B and C replied to it.

### Anticensor Enhancement

With doubly-linked replies enabled, messages also track their children via `reply_ids`:

- Message A (knows B and C replied to it)
    - Message B (replies to A)
    - Message C (replies to A)

### Capability Flag

Clients must opt-in to receive `reply_ids` by including the `DOUBLY_LINKED_REPLIES` capability flag during gateway identification.

**Capability**: `DOUBLY_LINKED_REPLIES`
**Bit Position**: 37
**Value**: `1 << 37` = `137438953472`

### API Response

When the capability is enabled, message objects include a `reply_ids` array:

```json
{
	"id": "123456789012345678",
	"content": "Original message",
	"reply_ids": ["234567890123456789", "345678901234567890"]
}
```

### Endpoint

A dedicated endpoint retrieves all replies to a message:

```
GET /channels/:channel_id/messages/:message_id/replies
```

**Response:**

```json
[
	{
		"id": "234567890123456789",
		"content": "First reply",
		"message_reference": {
			"message_id": "123456789012345678"
		}
	},
	{
		"id": "345678901234567890",
		"content": "Second reply",
		"message_reference": {
			"message_id": "123456789012345678"
		}
	}
]
```

### Use Cases

1. **Conversation Trees**: Build visual reply trees in clients
2. **Thread Analytics**: Analyze which messages generate the most discussion
3. **Notification Systems**: Notify users when their messages receive replies
4. **Content Moderation**: Automatically moderate entire reply chains
5. **Archive Systems**: Export complete conversation threads

### Performance Considerations

The `reply_ids` array is populated on-demand and cached in the database for efficiency. The system uses the `populateForwardLinks()` function to query and cache reply relationships.

### Client Implementation

To enable doubly-linked replies in your client:

1. Include the capability flag during IDENTIFY:

```json
{
  "op": 2,
  "d": {
    "token": "...",
    "capabilities": 137438953472,
    "properties": {...}
  }
}
```

2. Handle the `reply_ids` field in message objects
3. Use the `/replies` endpoint to fetch complete reply chains

## Guild Template Enhancements

!!! info "Discord Compatibility"
Guild templates are an official Discord feature. Anticensor extends this by supporting the `discord:` prefix to fetch templates directly from Discord.com, which is not part of the standard Discord API specification. See [Discord's guild template documentation](https://discord.com/developers/docs/resources/guild-template) for the base feature.

### Overview

Anticensor extends guild template support to include Discord.com templates and external template sources, not just locally-created templates.

### Template Sources

Anticensor supports three template code formats:

1. **Local Templates**: Standard Spacebar template codes

    ```
    POST /guilds
    {
      "guild_template_code": "abc123def456"
    }
    ```

2. **Discord Templates** (Anticensor Extension): Templates from Discord.com using `discord:` prefix

    ```
    POST /guilds
    {
      "guild_template_code": "discord:2TffvPucqHkN"
    }
    ```

3. **External Templates**: Templates from other sources using `external:` prefix
    ```
    POST /guilds
    {
      "guild_template_code": "external:https://example.com/template.json"
    }
    ```

### Configuration

Template functionality is controlled by configuration options:

| Configuration Key                 | Default | Description                                  |
| --------------------------------- | ------- | -------------------------------------------- |
| `templates_enabled`               | `true`  | Whether guild templates are enabled          |
| `templates_allowTemplateCreation` | `true`  | Whether new templates can be created         |
| `templates_allowDiscordTemplates` | `true`  | Whether Discord.com templates can be fetched |
| `templates_allowRaws`             | `true`  | Whether raw template JSON is allowed         |

### Discord Template Fetching

When a `discord:` template code is provided:

1. Anticensor fetches the template from Discord's API
2. The template structure is converted to Anticensor's format
3. A new guild is created based on the template
4. All channels, roles, and settings are replicated

### Use Cases

1. **Migration**: Import guild structures from Discord.com
2. **Standardization**: Use popular Discord templates as starting points
3. **Cross-Platform**: Share templates between Anticensor instances
4. **Backup/Restore**: Export and import guild configurations

### API Version Support

The guild creation endpoint supports Discord API v10 for template-based guild creation, ensuring compatibility with modern Discord clients.

## Version Endpoint

### Overview

Anticensor provides a version information endpoint for clients and monitoring tools.

### Endpoint

```
GET /-/version
```

**Response:**

```json
{
	"server": "anticensor",
	"version": "1.0.0",
	"api_version": 10
}
```

### Use Cases

1. **Client Compatibility**: Clients can check server version before connecting
2. **Monitoring**: Health check systems can verify server identity
3. **Feature Detection**: Determine which features are available
4. **Debugging**: Include version information in bug reports

## Privacy Gating for Third-Party Connections

### Overview

Anticensor implements comprehensive privacy controls for third-party service connections, allowing users to control what information is shared with external services.

### Features

1. **Granular Consent**: Users can consent to specific data sharing activities
2. **Connection Privacy**: Control visibility of third-party connections
3. **Data Minimization**: Only share necessary data with external services
4. **Audit Trail**: Track when and what data was shared

### Integration with Consent System

Privacy gating works in conjunction with the [User Consent Management](./consents.md) system to ensure users have explicitly consented to data sharing before connections are established.

### Configuration

Privacy settings are managed through user settings and connection-specific configurations. When establishing third-party connections, the system checks for appropriate consent records before proceeding.

## Presence Suppression by Rights

### Overview

Anticensor allows controlling presence visibility through the rights system, enabling operators and privileged users to hide their online status.

### Rights Flag

**Right**: `PRESENCE`
**Bit Position**: 35
**Value**: `1 << 35` = `34359738368`

### Behavior

The `PRESENCE` right inverts the default presence confidentiality behavior:

- **OPERATOR without PRESENCE**: Presence is hidden (not routed to other users)
- **OPERATOR with PRESENCE**: Presence is visible (routed normally)
- **Regular user without PRESENCE**: Presence is visible (default behavior)
- **Regular user with PRESENCE**: Presence is hidden

### Use Cases

1. **Staff Privacy**: Allow moderators and administrators to work invisibly
2. **Bot Accounts**: Hide bot presence to reduce clutter
3. **Service Accounts**: Keep system accounts hidden from users
4. **Privacy-Conscious Users**: Allow users to opt-out of presence tracking

### Implementation

The gateway service checks the `PRESENCE` right before routing presence updates. If presence should be suppressed, the updates are not sent to other connected clients.

### Configuration

Grant or revoke the `PRESENCE` right through the user rights system:

```sql
-- Hide an operator's presence (remove PRESENCE bit)
UPDATE users SET rights = rights & ~(1 << 35) WHERE id = 'user_id';

-- Show an operator's presence (add PRESENCE bit)
UPDATE users SET rights = rights | (1 << 35) WHERE id = 'user_id';
```

## OAuth2 Enhancements

### Overview

Anticensor extends OAuth2 authorization to support automatic bot installation and role assignment during the authorization flow.

### Bot Installation

When authorizing an application with bot scope:

1. The bot user is automatically added to the specified guild
2. The bot receives the permissions specified in the authorization request
3. The bot is assigned any roles specified in the authorization

### Role Assignment

The OAuth2 authorize endpoint supports a `role_id` parameter to automatically assign a role to the installed bot:

```
GET /oauth2/authorize?client_id=...&scope=bot&permissions=8&guild_id=...&role_id=...
```

### Use Cases

1. **Simplified Bot Setup**: Users don't need to manually invite bots after authorization
2. **Role-Based Bots**: Automatically assign bots to specific roles during installation
3. **Permission Templates**: Pre-configure bot permissions through OAuth2 flow
4. **Streamlined Onboarding**: Reduce steps required to add bots to guilds

## Categories Endpoint

### Overview

Anticensor provides a dedicated endpoint for querying guild categories (channel categories).

### Endpoint

```
GET /guilds/:guild_id/categories
```

**Response:**

```json
[
	{
		"id": "123456789012345678",
		"type": 4,
		"name": "Text Channels",
		"position": 0,
		"permission_overwrites": []
	},
	{
		"id": "234567890123456789",
		"type": 4,
		"name": "Voice Channels",
		"position": 1,
		"permission_overwrites": []
	}
]
```

### Use Cases

1. **Channel Organization**: Query category structure for UI rendering
2. **Permission Management**: List categories for permission configuration
3. **Guild Analytics**: Analyze guild organization structure
4. **Migration Tools**: Export/import category structures

## Users/@me/guilds Enhancements

### Overview

The `/users/@me/guilds` endpoint is enhanced to support additional query parameters for richer guild information.

### Query Parameters

**`with_counts`**: Include member and presence counts

```
GET /users/@me/guilds?with_counts=true
```

**Response:**

```json
[
	{
		"id": "123456789012345678",
		"name": "My Guild",
		"approximate_member_count": 1500,
		"approximate_presence_count": 450
	}
]
```

**`permissions`**: Include user's permissions in each guild

```
GET /users/@me/guilds?permissions=true
```

**Response:**

```json
[
	{
		"id": "123456789012345678",
		"name": "My Guild",
		"permissions": "2147483647"
	}
]
```

### Use Cases

1. **Guild Selection UI**: Show member counts when selecting guilds
2. **Permission Checks**: Determine user permissions without additional API calls
3. **Activity Indicators**: Display online member counts
4. **Guild Management**: Quick overview of guild statistics

## Related Documentation

- [User Consent Management](./consents.md) - Consent system for privacy controls
- [Lobby System](./lobbies.md) - Ephemeral matchmaking lobbies
- [Thread Promotion](./thread-promotion.md) - Convert threads to channels
- [Volatile Mode](./volatile-mode.md) - In-memory database mode
- [Configuration Options](../configuration/index.md) - Server configuration
- [User Rights](../security/rights.md) - Rights system reference
