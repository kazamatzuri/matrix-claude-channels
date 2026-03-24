# Matrix Channel for Claude Code

A Matrix messaging bridge for [Claude Code](https://claude.com/claude-code) — two-way communication with built-in access control.

## Prerequisites

- [Bun](https://bun.sh) runtime
- Claude Code v2.1.80+ (channels support)
- A Matrix bot account with an access token

## Setup

### 1. Create a Matrix bot account

Create an account on your homeserver for the bot, or use an existing one. You'll need:
- **Homeserver URL**: e.g., `https://matrix.org` or `https://matrix.example.com`
- **Access token**: obtainable via Element (Settings → Help & About → Access Token) or the login API

### 2. Install the plugin

```bash
# Clone this repo
git clone <repo-url> ~/projects/matrix-claude-channels

# Install as a Claude Code plugin
claude plugins add ~/projects/matrix-claude-channels
```

### 3. Configure credentials

In Claude Code, run:

```
/matrix:configure https://matrix.example.com syt_your_access_token_here
```

Or manually create `~/.claude/channels/matrix/.env`:

```
MATRIX_HOMESERVER_URL=https://matrix.example.com
MATRIX_ACCESS_TOKEN=syt_your_access_token_here
```

### 4. Launch

```bash
claude --channels plugin:matrix@claude-channel-matrix
```

Or for development:

```bash
claude --dangerously-load-development-channels server:matrix
```

### 5. Pair

1. DM your bot on Matrix — it replies with a 6-character code
2. In Claude Code: `/matrix:access pair <code>`
3. Lock down: `/matrix:access policy allowlist`

## Features

- **Two-way messaging**: Send and receive text, images, files
- **Access control**: Pairing flow, allowlists, per-room policies
- **Mention detection**: Respond to @mentions, replies, or custom patterns in rooms
- **Permission relay**: Approve/deny tool use from your Matrix client
- **Message editing**: Update sent messages for progress indicators
- **Reactions**: React to messages with any emoji
- **Message history**: Fetch recent room messages for context

## Environment Variables

| Variable | Required | Description |
| --- | --- | --- |
| `MATRIX_HOMESERVER_URL` | Yes | Matrix homeserver base URL |
| `MATRIX_ACCESS_TOKEN` | Yes | Bot account access token |
| `MATRIX_STATE_DIR` | No | Override state directory (default: `~/.claude/channels/matrix`) |
| `MATRIX_ACCESS_MODE` | No | Set to `static` to pin access config at boot |

## Access Control

See [ACCESS.md](./ACCESS.md) for the full access & delivery reference.

## License

MIT
