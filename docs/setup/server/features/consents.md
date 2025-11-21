# User Consent Management

!!! warning "Anticensor Extension"
This feature is specific to Anticensor and is not part of the official Discord API. Clients and libraries expecting standard Discord behavior will not have built-in support for this feature.

## Overview

The User Consent Management system provides a mechanism for tracking user consent to various services and data processing activities. This feature enables GDPR compliance and allows users to manage their consent preferences for different services integrated with the platform.

## Architecture

The consent system is built around the `UserConsent` entity, which stores consent records in the database with the following structure:

- `user_id`: The ID of the user who provided consent
- `service_id`: A string identifier for the service or data processing activity
- `created_at`: Timestamp when consent was granted

Each user-service pair is unique (enforced by a database index), meaning a user can only have one active consent record per service at any given time.

## API Endpoints

### User Endpoints

Users can manage their own consents through the following endpoints:

#### List User Consents

```
GET /users/@me/consents
```

Returns a list of all services the current user has consented to.

**Response:**

```json
[
	{
		"service_id": "analytics",
		"consented_at": "2024-01-15T10:30:00.000Z"
	},
	{
		"service_id": "marketing",
		"consented_at": "2024-02-20T14:45:00.000Z"
	}
]
```

#### Grant Consent

```
PUT /users/@me/consents/:service_id
```

Grants consent for a specific service. This operation is idempotent - calling it multiple times for the same service will not create duplicate records.

**Parameters:**

- `service_id` (path): The identifier of the service to consent to

**Response:**

```json
{
	"service_id": "analytics",
	"consented_at": "2024-01-15T10:30:00.000Z"
}
```

#### Revoke Consent

```
DELETE /users/@me/consents/:service_id
```

Revokes consent for a specific service. This operation is silent - it will not return an error if the consent record doesn't exist.

**Parameters:**

- `service_id` (path): The identifier of the service to revoke consent from

**Response:** `204 No Content`

### Administrative Endpoints

Administrators with the `MANAGE_USERS` right can manage consents for any user:

#### List User Consents (Admin)

```
GET /users/:id/consents
```

Returns all consent records for a specific user.

**Parameters:**

- `id` (path): The user ID to query consents for

**Response:** Same format as the user endpoint

#### Grant Consent (Admin)

```
PUT /users/:id/consents/:service_id
```

Grants consent on behalf of a user. This should be used carefully and only when legally appropriate.

**Parameters:**

- `id` (path): The user ID
- `service_id` (path): The service identifier

#### Revoke Consent (Admin)

```
DELETE /users/:id/consents/:service_id
```

Revokes a user's consent for a specific service.

**Parameters:**

- `id` (path): The user ID
- `service_id` (path): The service identifier

## Service Identifiers

Service identifiers are arbitrary strings that identify different services or data processing activities. Common examples include:

- `analytics` - User behavior analytics
- `marketing` - Marketing communications
- `third_party_integrations` - External service integrations
- `data_sharing` - Sharing data with partners
- `personalization` - Personalized content and recommendations

Instance administrators should define a consistent set of service identifiers based on their specific use case and legal requirements.

## Implementation Considerations

### GDPR Compliance

The consent system is designed to support GDPR compliance by:

1. **Explicit Consent**: Users must explicitly grant consent through API calls
2. **Granular Control**: Consent is tracked per-service, allowing fine-grained control
3. **Audit Trail**: The `created_at` timestamp provides an audit trail of when consent was granted
4. **Easy Revocation**: Users can revoke consent at any time through the DELETE endpoint

### Integration with Services

When integrating services that require user consent:

1. Define a unique `service_id` for the service
2. Check for consent before processing user data:
    ```typescript
    const consent = await UserConsent.findOne({
    	where: { user_id: userId, service_id: "analytics" },
    });
    if (!consent) {
    	// Skip analytics processing
    	return;
    }
    ```
3. Provide UI elements for users to manage their consents
4. Document what each service identifier means in your privacy policy

### Database Schema

The consent system uses a dedicated `user_consents` table with the following schema:

```sql
CREATE TABLE user_consents (
  id VARCHAR PRIMARY KEY,
  user_id VARCHAR NOT NULL,
  service_id VARCHAR NOT NULL,
  created_at TIMESTAMP NOT NULL,
  UNIQUE(user_id, service_id),
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

The `ON DELETE CASCADE` ensures that when a user is deleted, all their consent records are automatically removed.

## Best Practices

1. **Document Service IDs**: Maintain clear documentation of all service identifiers used in your instance
2. **Privacy Policy**: Link service identifiers to specific sections of your privacy policy
3. **Consent UI**: Provide clear, user-friendly interfaces for managing consents
4. **Regular Audits**: Periodically audit consent records to ensure compliance
5. **Consent Withdrawal**: Ensure that revoking consent immediately stops the associated data processing
6. **Legal Review**: Have your consent implementation reviewed by legal counsel to ensure compliance with applicable regulations

## Related Configuration

There are no specific configuration options for the consent system. It is always available and enabled in Anticensor instances.

## See Also

- [User Rights](../security/rights.md) - The `MANAGE_USERS` right is required for administrative consent management
- [Privacy Gating](./privacy-gating.md) - Related privacy controls for third-party connections
