<div align="center">

<a href="https://peal.social">
  <img src="https://raw.githubusercontent.com/peal-social/peal/main/packages/peal-site/public/favicon.svg" alt="Peal" width="96" height="96">
</a>

# Peal

### Write a post once. Schedule it everywhere. See how it did.

A social media scheduler for teams, with a public API, SDKs in eight languages,
a command line and an MCP server for agents.<br>
**Fair source**: read it, run it, self-host it. Each release turns Apache-2.0 two years after it ships.

[Website](https://peal.social) •
[Docs](https://docs.peal.social) •
[API reference](https://docs.peal.social/api-overview) •
[MCP](https://docs.peal.social/mcp) •
[Self-hosting](https://docs.peal.social/self-hosting) •
[Status](https://status.peal.social)

</div>

> [!NOTE]
> Peal is in preview and not released yet. The SDKs are published; the hosted service at `app.peal.social` is on its way. You can [self-host](#self-host) today.

```bash
npm install peal-sdk
```

```typescript
import { PealClient } from "peal-sdk";

const peal = new PealClient({ token: process.env.PEAL_API_KEY }); // a workspace key, peal_…

const channels = await peal.channels.list({ workspaceId });

const post = await peal.posts.send({
  workspaceId,
  channelIds: channels.map((channel) => channel.id),
  content: { text: "Hello from Peal!", media: [], overrides: [] },
  when: { _tag: "at", at: "2027-01-15T08:00:00Z", timeZone: "Europe/Paris" },
});
```

[![peal-sdk on npm](https://img.shields.io/npm/v/peal-sdk?label=peal-sdk&color=415eee)](https://www.npmjs.com/package/peal-sdk)
[![peal-sdk on PyPI](https://img.shields.io/pypi/v/peal-sdk?label=peal-sdk&color=415eee)](https://pypi.org/project/peal-sdk/)
[![OpenAPI 3.1](https://img.shields.io/badge/OpenAPI-3.1-415eee)](https://docs.peal.social/api-overview)
[![Licence: FSL-1.1-ALv2](https://img.shields.io/badge/licence-FSL--1.1--ALv2-555)](https://github.com/peal-social/peal/blob/main/LICENSE.md)

No SDK needed. A bearer token and JSON over HTTPS are the whole contract:

```bash
# Which channels can this key post to?
curl https://app.peal.social/api/workspaces/$WORKSPACE_ID/channels \
  -H "Authorization: Bearer $PEAL_API_KEY"

# Publish now to one of them. The Idempotency-Key makes the request safe to retry.
curl -X POST https://app.peal.social/api/workspaces/$WORKSPACE_ID/posts/send \
  -H "Authorization: Bearer $PEAL_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: $(uuidgen)" \
  -d '{
    "channelIds": ["CHANNEL_ID"],
    "content": { "text": "Hello from Peal!", "media": [], "overrides": [] },
    "when": { "_tag": "now" }
  }'
```

## Platforms

One composer and one request shape for all of them. Text, media and options are checked against each platform's rules before a post is scheduled, and every violation comes back at once.

| Platform | `platform` | Publishing | Analytics |
|---|---|:---:|:---:|
| X | `x` | ✓ | soon |
| LinkedIn (profiles and pages) | `linkedin` | ✓ | company pages |
| Bluesky | `bluesky` | ✓ | ✓ |
| Mastodon (any instance) | `mastodon` | ✓ | ✓ |
| Threads | `threads` | ✓ | soon |
| Instagram | `instagram` | ✓ | ✓ |
| Facebook Pages | `facebook` | ✓ | soon |
| TikTok | `tiktok` | ✓ | ✓ |
| YouTube | `youtube` | ✓ | ✓ |
| Pinterest | `pinterest` | ✓ | ✓ |
| Google Business Profile | `googleBusiness` | ✓ | – |

Each channel answers what it accepts (`GET …/channels/{channelId}/capabilities`), so a client never has to hard-code a platform's limits.

## What you can do

| Area | Does |
|---|---|
| **Posts** | Drafts, per-channel text, scheduling in the author's time zone, a calendar, cancel, reschedule, post again where an attempt failed |
| **Media** | Images, videos and PDF documents, uploaded straight to storage or imported from a URL, with alt text |
| **Review** | Share a post through a link for outside comments and approval; have posts wait for approval before they go out |
| **Analytics** | Likes, reposts, replies, quotes and views, read back hourly for two days then daily for a month |
| **Webhooks** | Posts scheduled, published, failed or cancelled; channels to reconnect; signed per Standard Webhooks, retried for about a day, with a delivery log |
| **Assistant** | Rewrite, shorten or translate a post, suggest hashtags, with Anthropic or any OpenAI-compatible endpoint |
| **Workspaces** | Members, roles, invitations, scoped API keys, plans and billing |

## SDKs

Generated from one contract, released together, Apache-2.0.

| Language | Install | Version |
|---|---|---|
| [TypeScript / Node.js](https://github.com/peal-social/peal/tree/main/packages/peal-sdk/node) | `npm install peal-sdk` | [![npm](https://img.shields.io/npm/v/peal-sdk?label=&color=415eee)](https://www.npmjs.com/package/peal-sdk) |
| [Python](https://github.com/peal-social/peal/tree/main/packages/peal-sdk/python) | `pip install peal-sdk` | [![PyPI](https://img.shields.io/pypi/v/peal-sdk?label=&color=415eee)](https://pypi.org/project/peal-sdk/) |
| [Go](https://github.com/peal-social/peal/tree/main/packages/peal-sdk/go) | `go get github.com/peal-social/peal/packages/peal-sdk/go` | [![Go](https://img.shields.io/badge/pkg.go.dev-reference-415eee)](https://pkg.go.dev/github.com/peal-social/peal/packages/peal-sdk/go) |
| [Ruby](https://github.com/peal-social/peal/tree/main/packages/peal-sdk/ruby) | `gem install peal` | [![Gem](https://img.shields.io/gem/v/peal?label=&color=415eee)](https://rubygems.org/gems/peal) |
| [Java](https://github.com/peal-social/peal/tree/main/packages/peal-sdk/java) | `social.peal:peal` | [![Maven Central](https://img.shields.io/maven-metadata/v?metadataUrl=https%3A%2F%2Frepo1.maven.org%2Fmaven2%2Fsocial%2Fpeal%2Fpeal%2Fmaven-metadata.xml&label=&color=415eee)](https://central.sonatype.com/artifact/social.peal/peal) |
| [PHP](https://github.com/peal-social/peal-php) | `composer require peal-social/peal` | [![Packagist](https://img.shields.io/packagist/v/peal-social/peal?label=&color=415eee)](https://packagist.org/packages/peal-social/peal) |
| [.NET](https://github.com/peal-social/peal/tree/main/packages/peal-sdk/dotnet) | `dotnet add package Peal` | [![NuGet](https://img.shields.io/nuget/v/Peal?label=&color=415eee)](https://www.nuget.org/packages/Peal) |
| [Rust](https://github.com/peal-social/peal/tree/main/packages/peal-sdk/rust) | `cargo add peal-sdk` | [![crates.io](https://img.shields.io/crates/v/peal-sdk?label=&color=415eee)](https://crates.io/crates/peal-sdk) |

<details>
<summary><strong>Python quickstart</strong></summary>

<br>

```python
import os

from peal import Peal, PostContent, PostTiming_At

peal = Peal(token=os.environ["PEAL_API_KEY"])

channels = peal.channels.list(workspace_id=workspace_id)

post = peal.posts.send(
    workspace_id=workspace_id,
    channel_ids=[channel.id for channel in channels],
    content=PostContent(text="Hello from Peal!", media=[], overrides=[]),
    when=PostTiming_At(at="2027-01-15T08:00:00Z", time_zone="Europe/Paris"),
)
```

</details>

## Agents and the terminal

| | What it is |
|---|---|
| **MCP server** | Built into Peal at `/api/mcp` (Streamable HTTP, OAuth sign-in). An agent works in the one workspace you pick, with the scopes you allow: read, write drafts, or publish. [MCP docs](https://docs.peal.social/mcp) · [Posting from an agent](https://docs.peal.social/agents) |
| **`peal` CLI** | Drafts, posts, analytics, billing, members and invitations from the terminal, signed in with OAuth. Builds from source today. [CLI docs](https://docs.peal.social/cli) |

## Self-host

Peal needs Postgres and an S3-compatible store; everything else (email, billing, captcha, AI, telemetry) is optional and off until you configure it.

```bash
git clone https://github.com/peal-social/peal.git
cd peal/packages/peal-docker
cp self-host.env.example /srv/peal/peal.env   # your domain and secrets, outside the clone
docker compose --project-name peal --file compose.production.yaml \
  --file compose.self-host.yaml --env-file /srv/peal/peal.env up --build --detach --wait
```

Caddy terminates TLS in front of the stack, with a Let's Encrypt certificate for your domain. The [self-hosting guide](https://docs.peal.social/self-hosting) covers each platform's app registration.

## Repositories

| Repo | What it is |
|---|---|
| **[peal](https://github.com/peal-social/peal)** | The monorepo: server, web app, contract, SDKs, CLI, docs, site and infrastructure. Issues and pull requests go here |
| [peal-php](https://github.com/peal-social/peal-php) | Read-only mirror of the PHP SDK, for Packagist. Open issues in `peal` |

## Licensing

| Part | Licence |
|---|---|
| Peal itself | [FSL-1.1-ALv2](https://github.com/peal-social/peal/blob/main/LICENSE.md): any use except offering a competing service, and Apache-2.0 two years after each release |
| The API contract and the SDKs | [Apache-2.0](https://github.com/peal-social/peal/blob/main/packages/peal-sdk/LICENSE) |

Found a security issue? Write to [security@peal.social](mailto:security@peal.social) rather than opening an issue.

<div align="center">

[<kbd> <br> Read the docs ➜ <br> </kbd>](https://docs.peal.social)
&nbsp;&nbsp;
[<kbd> <br> Star the repo ➜ <br> </kbd>](https://github.com/peal-social/peal)

</div>
