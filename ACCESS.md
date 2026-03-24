# Matrix — Access & Delivery

A Matrix bot is addressable by anyone who knows its MXID or shares a room. Without a gate, messages from unknown users would flow straight into your assistant session. The access model described here decides who gets through.

By default, a DM from an unknown sender triggers **pairing**: the bot replies with a 6-character code and drops the message. You run `/matrix:access pair <code>` from your assistant session to approve them. Once approved, their messages pass through.

All state lives in `~/.claude/channels/matrix/access.json`. The `/matrix:access` skill commands edit this file; the server re-reads it on every inbound message, so changes take effect without a restart. Set `MATRIX_ACCESS_MODE=static` to pin config to what was on disk at boot (pairing is unavailable in static mode since it requires runtime writes).

## At a glance

| | |
| --- | --- |
| Default policy | `pairing` |
| Sender ID | Matrix user ID (e.g. `@alice:matrix.org`) |
| Room key | Room ID (e.g. `!abc123:matrix.org`) |
| Config file | `~/.claude/channels/matrix/access.json` |

## DM policies

`dmPolicy` controls how DMs from senders not on the allowlist are handled.

| Policy | Behavior |
| --- | --- |
| `pairing` (default) | Reply with a pairing code, drop the message. Approve with `/matrix:access pair <code>`. |
| `allowlist` | Drop silently. No reply. |
| `disabled` | Drop everything, including allowlisted users and rooms. |

```
/matrix:access policy allowlist
```

## User IDs

Matrix identifies users by **MXIDs** like `@alice:matrix.org`. These are permanent. The allowlist stores MXIDs.

Pairing captures the MXID automatically. To find one manually, ask the person for their Matrix ID (visible in their profile in any Matrix client).

```
/matrix:access allow @alice:matrix.org
/matrix:access remove @alice:matrix.org
```

## Rooms

Rooms are off by default. Opt each one in individually.

```
/matrix:access room add !abc123:matrix.org
```

Room IDs look like `!abc123:matrix.org`. To find one, check Room Settings → Advanced in Element, or use the bot's `/roomid` response.

With the default `requireMention: true`, the bot responds only when @mentioned or replied to. Pass `--no-mention` to process every message, or `--allow id1,id2` to restrict which members can trigger it.

```
/matrix:access room add !abc123:matrix.org --no-mention
/matrix:access room add !abc123:matrix.org --allow @alice:matrix.org,@bob:matrix.org
/matrix:access room rm !abc123:matrix.org
```

## Mention detection

In rooms with `requireMention: true`, any of the following triggers the bot:

- The bot's MXID (`@botname:server.com`) appears in the message body
- The bot's localpart (`botname`) appears in the message body (case-insensitive)
- A reply to one of the bot's recent messages
- A match against any regex in `mentionPatterns`

```
/matrix:access set mentionPatterns '["^hey claude\\b", "\\bassistant\\b"]'
```

## Delivery

Configure outbound behavior with `/matrix:access set <key> <value>`.

**`ackReaction`** reacts to inbound messages on receipt. Matrix supports any Unicode emoji.

```
/matrix:access set ackReaction 👀
/matrix:access set ackReaction ""
```

**`replyToMode`** controls threading on chunked replies. When a long response is split, `first` (default) threads only the first chunk under the inbound message; `all` threads every chunk; `off` sends all chunks standalone.

**`textChunkLimit`** sets the split threshold. Default: 16000. Matrix has no hard message size limit, but very long messages degrade readability.

**`chunkMode`** chooses the split strategy: `length` cuts exactly at the limit; `newline` (default) prefers paragraph boundaries.

## Skill reference

| Command | Effect |
| --- | --- |
| `/matrix:access` | Print current state: policy, allowlist, pending pairings, enabled rooms. |
| `/matrix:access pair a4f91c` | Approve pairing code `a4f91c`. Adds the sender to `allowFrom` and sends a confirmation on Matrix. |
| `/matrix:access deny a4f91c` | Discard a pending code. The sender is not notified. |
| `/matrix:access allow @user:server.com` | Add a user directly. |
| `/matrix:access remove @user:server.com` | Remove from the allowlist. |
| `/matrix:access policy allowlist` | Set `dmPolicy`. Values: `pairing`, `allowlist`, `disabled`. |
| `/matrix:access room add !room:server.com` | Enable a room. Flags: `--no-mention`, `--allow id1,id2`. |
| `/matrix:access room rm !room:server.com` | Disable a room. |
| `/matrix:access set ackReaction 👀` | Set a config key. |

## Config file

`~/.claude/channels/matrix/access.json`. Absent file is equivalent to `pairing` policy with empty lists, so the first DM triggers pairing.

```jsonc
{
  // Handling for DMs from senders not in allowFrom.
  "dmPolicy": "pairing",

  // Matrix user IDs allowed to DM.
  "allowFrom": ["@alice:matrix.org"],

  // Rooms the bot is active in. Empty object = DM-only.
  "rooms": {
    "!abc123:matrix.org": {
      // true: respond only to @mentions and replies.
      "requireMention": true,
      // Restrict triggers to these senders. Empty = any member.
      "allowFrom": []
    }
  },

  // Case-insensitive regexes that count as a mention.
  "mentionPatterns": ["^hey claude\\b"],

  // Emoji to react with on receipt. Empty string disables.
  "ackReaction": "👀",

  // Threading on chunked replies: first | all | off
  "replyToMode": "first",

  // Split threshold. Default 16000.
  "textChunkLimit": 16000,

  // newline = prefer paragraph boundaries. length = cut at limit.
  "chunkMode": "newline"
}
```
