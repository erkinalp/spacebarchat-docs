# Configurable Limits System

## Overview

Anticensor provides a comprehensive configurable limits system that allows instance administrators to customize various restrictions and boundaries throughout the platform. This system addresses 48+ configuration issues by providing fine-grained control over limits that were previously hardcoded.

The limits system supports:

- **Global defaults**: Instance-wide limit configurations
- **Per-guild overrides**: Guild-specific limit customization
- **Unlimited values**: Use `null` to represent unlimited
- **Feature disabling**: Use `0` to disable features

## Limit Categories

### User Limits

Controls restrictions on user accounts and profiles.

| Limit                  | Default | Description                              |
| ---------------------- | ------- | ---------------------------------------- |
| `maxGuilds`            | 1048576 | Maximum number of guilds a user can join |
| `maxUsername`          | 32      | Maximum username length in characters    |
| `maxFriends`           | 5000    | Maximum number of friends per user       |
| `maxBio`               | 190     | Maximum biography length                 |
| `maxDisplayNameLength` | 32      | Maximum display name length              |
| `maxPronounsLength`    | 40      | Maximum pronouns field length            |
| `maxStatusTextLength`  | 128     | Maximum custom status text length        |

**Configuration Path**: `limits.user.*`

**Example**:

```json
{
	"limits": {
		"user": {
			"maxGuilds": 100,
			"maxFriends": 1000,
			"maxBio": 500
		}
	}
}
```

### Guild Limits

Controls restrictions on guild (server) configuration and content.

| Limit                    | Default  | Description                                     |
| ------------------------ | -------- | ----------------------------------------------- |
| `maxRoles`               | 1000     | Maximum number of roles in a guild              |
| `maxEmojis`              | 2000     | Maximum number of custom emojis                 |
| `maxMembers`             | 25000000 | Maximum guild member count                      |
| `maxChannels`            | 65535    | Maximum number of channels                      |
| `maxBulkBanUsers`        | 200      | Maximum users in a single bulk ban operation    |
| `maxChannelsInCategory`  | 65535    | Maximum channels per category                   |
| `maxGuildNameLength`     | 100      | Maximum guild name length                       |
| `maxRoleNameLength`      | 100      | Maximum role name length                        |
| `maxMembersRequestLimit` | 100      | Maximum members to request in a single API call |

**Configuration Path**: `limits.guild.*`

**Example**:

```json
{
	"limits": {
		"guild": {
			"maxRoles": 500,
			"maxEmojis": 1000,
			"maxMembers": 10000
		}
	}
}
```

### Channel Limits

Controls restrictions on channel configuration and content.

| Limit                  | Default | Description                         |
| ---------------------- | ------- | ----------------------------------- |
| `maxPins`              | 500     | Maximum pinned messages per channel |
| `maxTopic`             | 1024    | Maximum channel topic length        |
| `maxWebhooks`          | 100     | Maximum webhooks per channel        |
| `maxChannelNameLength` | 100     | Maximum channel name length         |
| `maxStageTopicLength`  | 120     | Maximum stage channel topic length  |
| `maxThreadNameLength`  | 100     | Maximum thread name length          |

**Configuration Path**: `limits.channel.*`

**Example**:

```json
{
	"limits": {
		"channel": {
			"maxPins": 1000,
			"maxTopic": 2048,
			"maxWebhooks": 50
		}
	}
}
```

### Message Limits

Controls restrictions on message content and operations.

| Limit                  | Default    | Description                               |
| ---------------------- | ---------- | ----------------------------------------- |
| `maxCharacters`        | 1048576    | Maximum message content length (1MB)      |
| `maxTTSCharacters`     | 160        | Maximum text-to-speech message length     |
| `maxReactions`         | 2048       | Maximum reactions per message             |
| `maxAttachmentSize`    | 1073741824 | Maximum total attachment size (1GB)       |
| `maxBulkDelete`        | 1000       | Maximum messages in bulk delete operation |
| `maxEmbedDownloadSize` | 5242880    | Maximum embed content download size (5MB) |
| `maxReasonLength`      | 512        | Maximum audit log reason length           |

**Configuration Path**: `limits.message.*`

**Example**:

```json
{
	"limits": {
		"message": {
			"maxCharacters": 4000,
			"maxAttachmentSize": 8388608,
			"maxBulkDelete": 100
		}
	}
}
```

### Embed Limits

Controls restrictions on message embeds.

| Limit                  | Default | Description                          |
| ---------------------- | ------- | ------------------------------------ |
| `maxTitleLength`       | 256     | Maximum embed title length           |
| `maxDescriptionLength` | 4096    | Maximum embed description length     |
| `maxFieldCount`        | 25      | Maximum number of fields in an embed |
| `maxFieldNameLength`   | 256     | Maximum field name length            |
| `maxFieldValueLength`  | 1024    | Maximum field value length           |
| `maxFooterLength`      | 2048    | Maximum footer text length           |
| `maxAuthorNameLength`  | 256     | Maximum author name length           |

**Configuration Path**: `limits.embed.*`

### Attachment Limits

Controls restrictions on file attachments.

| Limit      | Default    | Description                          |
| ---------- | ---------- | ------------------------------------ |
| `maxSize`  | 1073741824 | Maximum single attachment size (1GB) |
| `maxCount` | 10         | Maximum attachments per message      |

**Configuration Path**: `limits.attachment.*`

### Application Limits

Controls restrictions on OAuth2 applications and bots.

| Limit                             | Default | Description                            |
| --------------------------------- | ------- | -------------------------------------- |
| `maxApplicationNameLength`        | 32      | Maximum application name length        |
| `maxApplicationDescriptionLength` | 400     | Maximum application description length |
| `maxBotUsernameLength`            | 32      | Maximum bot username length            |

**Configuration Path**: `limits.application.*`

### Component Limits

Controls restrictions on message components (buttons, select menus, etc.).

| Limit                  | Default | Description                      |
| ---------------------- | ------- | -------------------------------- |
| `maxActionRows`        | 5       | Maximum action rows per message  |
| `maxButtonsPerRow`     | 5       | Maximum buttons per action row   |
| `maxSelectMenuOptions` | 25      | Maximum options in a select menu |

**Configuration Path**: `limits.component.*`

### Automod Limits

Controls restrictions on automod rules and actions.

| Limit                | Default | Description                             |
| -------------------- | ------- | --------------------------------------- |
| `maxRulesPerGuild`   | 10      | Maximum automod rules per guild         |
| `maxKeywordLength`   | 60      | Maximum keyword length in automod rules |
| `maxKeywordsPerRule` | 1000    | Maximum keywords per automod rule       |

**Configuration Path**: `limits.automod.*`

## Special Values

### Unlimited (`null`)

Setting a limit to `null` represents unlimited/no restriction:

```json
{
	"limits": {
		"guild": {
			"maxMembers": null,
			"maxChannels": null
		}
	}
}
```

This allows guilds to have unlimited members and channels (subject to system resources).

### Feature Disabled (`0`)

Setting a limit to `0` disables the feature entirely:

```json
{
	"limits": {
		"channel": {
			"maxPins": 0
		}
	}
}
```

This prevents any messages from being pinned in channels.

## Per-Guild Overrides

Guild administrators can override global limits for their specific guild (if the instance configuration allows it).

### Configuration Structure

Per-guild limits are stored in the guild's configuration:

```json
{
	"guild_id": "123456789012345678",
	"limits": {
		"maxMembers": 50000,
		"maxChannels": 100,
		"maxRoles": 250
	}
}
```

### Limit Resolution

When determining the effective limit for a guild:

1. Check if the guild has a specific override for the limit
2. If yes, use the guild-specific value
3. If no, fall back to the global default

### Example Implementation

```typescript
function getEffectiveLimit(guild_id: string, limitName: string): number | null {
	const guildConfig = await GuildConfig.findOne({ where: { guild_id } });

	if (guildConfig?.limits?.[limitName] !== undefined) {
		return guildConfig.limits[limitName];
	}

	const globalConfig = Config.get();
	return globalConfig.limits[limitCategory][limitName];
}
```

## Pagination Limits

Pagination limits control how many items can be requested in a single API call.

### Common Pagination Limits

- `maxMembersRequestLimit`: Maximum members to fetch in `/guilds/:id/members`
- `maxMessagesRequestLimit`: Maximum messages to fetch in `/channels/:id/messages`
- `maxGuildsRequestLimit`: Maximum guilds to fetch in `/users/@me/guilds`

### Unlimited Pagination

Setting pagination limits to `null` allows unlimited results (use with caution):

```json
{
	"limits": {
		"guild": {
			"maxMembersRequestLimit": null
		}
	}
}
```

**Warning**: Unlimited pagination can cause performance issues and memory exhaustion.

### Disabled Pagination

Setting pagination limits to `0` disables the endpoint:

```json
{
	"limits": {
		"guild": {
			"maxMembersRequestLimit": 0
		}
	}
}
```

This returns an error when attempting to fetch members.

## Rate Limits

Rate limits are a special category of limits that control request frequency. See [Rate Limits](../security/limits.md) for detailed information.

## Use Cases

### Small Community Instance

For a small, trusted community:

```json
{
	"limits": {
		"user": {
			"maxGuilds": 50,
			"maxFriends": 500
		},
		"guild": {
			"maxMembers": 1000,
			"maxChannels": 50,
			"maxRoles": 100
		},
		"message": {
			"maxCharacters": 4000,
			"maxAttachmentSize": 10485760
		}
	}
}
```

### Large Public Instance

For a large public instance with more restrictions:

```json
{
	"limits": {
		"user": {
			"maxGuilds": 100,
			"maxFriends": 1000
		},
		"guild": {
			"maxMembers": 100000,
			"maxChannels": 500,
			"maxRoles": 250
		},
		"message": {
			"maxCharacters": 2000,
			"maxAttachmentSize": 8388608
		}
	}
}
```

### Premium Guild Override

Give a premium guild higher limits:

```json
{
	"guild_id": "premium_guild_id",
	"limits": {
		"maxMembers": null,
		"maxChannels": 1000,
		"maxEmojis": 5000,
		"maxRoles": 500
	}
}
```

## Configuration Methods

### Database Configuration

Update limits through the configuration table:

```sql
UPDATE config
SET value = '{"maxGuilds": 100, "maxFriends": 1000}'
WHERE key = 'limits_user';
```

### JSON Configuration File

If using `CONFIG_PATH`, update the JSON file:

```json
{
	"limits": {
		"user": {
			"maxGuilds": 100,
			"maxFriends": 1000
		},
		"guild": {
			"maxMembers": 50000
		}
	}
}
```

### Environment Variables

Some limits can be set via environment variables (check documentation for specific variables).

## Validation and Enforcement

### Server-Side Validation

All limits are enforced server-side:

1. API requests validate input against configured limits
2. Requests exceeding limits return `400 Bad Request` with error details
3. Database constraints prevent storing data exceeding limits

### Error Responses

When a limit is exceeded:

```json
{
	"code": 50035,
	"message": "Invalid Form Body",
	"errors": {
		"content": {
			"_errors": [
				{
					"code": "BASE_TYPE_MAX_LENGTH",
					"message": "Must be 2000 or fewer in length."
				}
			]
		}
	}
}
```

### Client-Side Validation

Clients should validate against limits before sending requests to provide better user experience:

1. Fetch instance limits from `/api/policies/instance/limits`
2. Validate user input against limits
3. Show helpful error messages before submission

## Performance Considerations

### Memory Usage

Higher limits increase memory usage:

- Large message limits increase memory per message
- High member limits increase guild state size
- Unlimited pagination can exhaust memory

### Database Performance

Limits affect database performance:

- Large bulk operations (bulk delete, bulk ban) increase database load
- High pagination limits increase query time
- Unlimited values can cause slow queries

### Recommendations

1. **Start Conservative**: Begin with lower limits and increase as needed
2. **Monitor Resources**: Track memory and database usage
3. **Use Pagination**: Always paginate large result sets
4. **Set Reasonable Defaults**: Balance usability with resource constraints
5. **Per-Guild Overrides**: Use overrides for trusted guilds rather than raising global limits

## Migration from Hardcoded Limits

If migrating from a version with hardcoded limits:

1. Review current hardcoded values
2. Set global defaults matching previous behavior
3. Gradually adjust limits based on usage patterns
4. Communicate changes to users
5. Monitor for issues after changes

## Related Documentation

- [Configuration Options](../configuration/index.md) - General configuration
- [Rate Limits](../security/limits.md) - Request rate limiting
- [Guild Features](../configuration/guildFeatures.md) - Guild feature flags
- [User Rights](../security/rights.md) - User permission system

## Comparison with Discord

| Feature             | Discord          | Anticensor   |
| ------------------- | ---------------- | ------------ |
| Configurable Limits | No               | Yes          |
| Per-Guild Overrides | Limited (boosts) | Full control |
| Unlimited Values    | No               | Yes (null)   |
| Feature Disabling   | No               | Yes (0)      |
| Pagination Control  | Fixed            | Configurable |

Anticensor provides significantly more flexibility in limit configuration compared to Discord's fixed limits and boost-based system.
