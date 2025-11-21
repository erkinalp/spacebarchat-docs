# Anticensor Features

Anticensor implements Discord-compatible features and adds extensions beyond the standard Discord API. This page provides an overview of both categories.

## Discord-Compatible Features

These features follow the official Discord API specification and are implemented by Anticensor for compatibility:

- [Lobby System](lobbies.md) - Discord's matchmaking and coordination lobby API (implemented with in-memory storage)

## Anticensor Extensions

These features extend beyond the standard Discord API and are specific to Anticensor:

- [User Consent Management](consents.md) - GDPR-compliant consent tracking system
- [Thread Promotion](thread-promotion.md) - Convert threads to top-level channels
- [Volatile Mode](volatile-mode.md) - In-memory database for testing and development
- [Configurable Limits](configurable-limits.md) - Comprehensive limit configuration system with per-guild overrides
- [Anticensor Enhancements](anticensor-enhancements.md) - Additional feature enhancements including:
    - PIN_MESSAGES permission (uses bit 38 instead of Discord's bit 51)
    - Doubly-linked replies for conversation threading
    - Guild template support for Discord.com templates (discord: prefix)
    - Privacy gating for third-party connections
    - Presence suppression by rights
    - OAuth2 bot installation enhancements
    - Version endpoint (/-/version)
    - Categories endpoint
    - users/@me/guilds enhancements (with_counts and permissions parameters)
