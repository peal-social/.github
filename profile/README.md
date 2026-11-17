<div align="center">

<a href="https://peal.social">
  <img src="https://raw.githubusercontent.com/peal-social/peal/main/.github/logo/peal-cube.svg" alt="Peal" width="128" height="128">
</a>

# Peal

### Schedule social posts with your team.

You can also schedule them from the public API, SDKs in 8 languages, a command
line or an AI agent.<br>
Peal is **fair source**, so you can read the code and run it yourself. Each release becomes Apache-2.0 two years after it ships.

[Website](https://peal.social) •
[Docs](https://docs.peal.social) •
[API reference](https://docs.peal.social/reference) •
[MCP](https://docs.peal.social/developers/mcp) •
[Self-hosting](https://docs.peal.social/self-hosting) •
[Roadmap](https://github.com/orgs/peal-social/projects/1) •
[Status](https://status.peal.social)

</div>

> [!NOTE]
> Peal is being built in the open and hasn't been released yet. The SDKs are published, and the hosted service at [app.peal.social](https://app.peal.social) is an early preview. You can [self-host](#self-host) it today. The [roadmap](https://github.com/orgs/peal-social/projects/1) shows what's next.

Install the Node.js SDK:

```bash
npm install peal-sdk
```

Then schedule a post on every channel your key can reach:

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

[![peal-sdk on npm](https://img.shields.io/npm/v/peal-sdk?label=peal-sdk&color=ffca00)](https://www.npmjs.com/package/peal-sdk)
[![peal-sdk on PyPI](https://img.shields.io/pypi/v/peal-sdk?label=peal-sdk&color=ff98d3)](https://pypi.org/project/peal-sdk/)
[![OpenAPI 3.1](https://img.shields.io/badge/OpenAPI-3.1-968efb)](https://docs.peal.social/reference)
[![Licence: FSL-1.1-ALv2](https://img.shields.io/badge/licence-FSL--1.1--ALv2-415eee)](https://github.com/peal-social/peal/blob/main/LICENSE.md)

You don't need an SDK. Any HTTP client works with a bearer token and JSON:

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

Peal posts to 11 social networks and 3 chats, publishes articles to 4 kinds of website, and reads ad accounts on 2 ad networks. A post's text, media and options are checked against each platform's rules before it's scheduled, and you get every problem back at once. Each channel also says what it accepts (`GET …/channels/{channelId}/capabilities`), so your client doesn't have to hard-code any limit.

### Social networks

Every network takes posts, and most give their counts back. The Inbox column says what Peal reads there and what you can answer:

| Network | `platform` | Publish | Analytics | Inbox |
|---|---|:---:|:---:|---|
| X | `x` | ✓ | operator opt-in | – |
| LinkedIn (profiles and pages) | `linkedin` | ✓ | company pages | page comments (awaiting LinkedIn) |
| Bluesky | `bluesky` | ✓ | ✓ | comments, direct messages |
| Mastodon (any instance) | `mastodon` | ✓ | ✓ | comments, direct messages |
| Threads | `threads` | ✓ | ✓ | – |
| Instagram | `instagram` | ✓ | ✓ | comments, messages, private replies (awaiting Meta) |
| Facebook Pages | `facebook` | ✓ | ✓ | comments, messages, private replies (awaiting Meta) |
| TikTok | `tiktok` | ✓ | ✓ | – |
| YouTube | `youtube` | ✓ | ✓ | comments (awaiting Google) |
| Pinterest | `pinterest` | ✓ | ✓ | – |
| Google Business Profile | `googleBusiness` | ✓ | – | reviews (awaiting Google) |

"Awaiting" means the feature is built but stays off until the platform has reviewed the server's app. For Facebook and Instagram that's Meta's App Review. YouTube needs Google's verification, Google Business Profile needs Google's API access, and LinkedIn pages need LinkedIn's comment scopes.

Direct messages are private, so Peal reads none until whoever runs the server turns them on. An owner or an admin then turns them on one channel at a time, and Peal keeps each message for 30 days ([Inbox](https://docs.peal.social/guides/engage/inbox)).

### Chats

A chat takes posts, but Peal reads nothing back from it, so there are no counts and no inbox. Here's how each one takes a post:

| Chat | `platform` | Connects with | A thread's parts | Media | Edit and delete |
|---|---|---|---|---|---|
| Discord | `discord` | A channel's webhook | Messages in a row, or one forum post | Images and videos | Both |
| Telegram | `telegram` | Your own bot, in a channel or a group | Replies, each to the part before | Photos and MP4 videos, or one GIF alone | Edit, and delete within 48 hours of sending |
| Slack | `slack` | Signing in to Slack, one channel each | Replies in the thread of the first part | Images, no video | Both, unless the workspace stops edits after a while |

### Websites

A website takes articles rather than posts. An article has a title, a formatted body, an excerpt, tags, categories and a featured image. Here's what each kind of site takes:

| Website | Kind | Connects with | Tags, at most | Categories, at most | On the site | Unpublish |
|---|---|---|:---:|:---:|---|---|
| A WordPress site of your own | `wordpress` | Its address, a user and an app password | 20 | 20 | Published, a draft, or scheduled | Moves it to the site's trash |
| WordPress.com | `wordpressCom` | Signing in to WordPress.com | 20 | 20 | Published, a draft, or scheduled | Moves it to the site's trash |
| DEV | `devto` | An API key | 4 | – | Published, a draft, or scheduled | Turns it back into a draft |
| Hashnode, on its Pro plan | `hashnode` | A personal access token | 15 | – | Published or a draft | Removes it from the site |

Peal never sends an article twice. When a site's answer is lost and Peal can't find the article there by its title, it marks the article to check, and you look on the site before sending it again.

### Ads

Ad accounts connect next to your channels. They stay off until whoever runs the server turns ads on, and nothing spends until an owner lets an account spend and sets its monthly ceiling. Here's what Peal does on each network:

| Network | What Peal reads | How often | Next to your posts | Boost a post |
|---|---|---|---|---|
| Meta (awaiting Meta) | Campaigns, ad sets, ads and each ad's numbers by day | Every hour | Yes, on the page of the post it promotes | On a Facebook Page or Instagram, within the account's monthly ceiling |
| Google Ads | Campaigns and each one's daily cost, impressions and clicks | Every 6 hours | No, its numbers stand on their own | No, Peal changes nothing at Google |

Meta has to grant the server's app its App Review before Peal can read or spend on anyone else's ad account ([Ads](https://docs.peal.social/guides/engage/ads)).

## What you can do

Here's the rest, by area:

| Area | What you can do |
|---|---|
| **Posts** | Write drafts with their own text on each channel, schedule them in your time zone on a calendar, move or cancel them, and post again where an attempt failed |
| **Media** | Add images, videos and PDF documents with alt text, uploaded straight to storage or imported from a URL |
| **Review** | Share a post through a link for comments and approval from outside the workspace, and make posts wait for an approval before they go out |
| **Analytics** | Read each post's likes, reposts, replies, quotes and views every hour for 48 hours, then every day until it's 30 days old, with followers by day and a CSV export |
| **Inbox** | Answer comments, direct messages and Google Business Profile reviews as the connected account |
| **Automations** | Answer comments that hold a keyword, 25 times a day at most per rule, and comment under or repost your own post once it has the likes you set |
| **Webhooks** | Hear about posts, articles, boosts, channels to reconnect, and comments, messages and reviews received. Each delivery is signed as Standard Webhooks says, with 8 attempts over about a day and a log |
| **Assistant** | Rewrite, shorten or translate a post and suggest hashtags, with Anthropic or any OpenAI-compatible endpoint |
| **Workspaces** | Invite members with one of 5 roles, make scoped API keys, and pick a plan. Sign in with a password or Google, or with your own OpenID Connect provider on a server you run |

## SDKs

The SDKs are generated from one contract and released together. They're under Apache-2.0, unlike the rest of Peal:

| Language | Install | Version |
|---|---|---|
| [TypeScript / Node.js](https://github.com/peal-social/peal/tree/main/packages/peal-sdk/node) | `npm install peal-sdk` | [![npm](https://img.shields.io/npm/v/peal-sdk?label=&color=ffca00)](https://www.npmjs.com/package/peal-sdk) |
| [Python](https://github.com/peal-social/peal/tree/main/packages/peal-sdk/python) | `pip install peal-sdk` | [![PyPI](https://img.shields.io/pypi/v/peal-sdk?label=&color=ffb696)](https://pypi.org/project/peal-sdk/) |
| [Go](https://github.com/peal-social/peal/tree/main/packages/peal-sdk/go) | `go get github.com/peal-social/peal/packages/peal-sdk/go` | [![Go](https://img.shields.io/badge/pkg.go.dev-reference-ffa2be)](https://pkg.go.dev/github.com/peal-social/peal/packages/peal-sdk/go) |
| [Ruby](https://github.com/peal-social/peal/tree/main/packages/peal-sdk/ruby) | `gem install peal` | [![Gem](https://img.shields.io/gem/v/peal?label=&color=da9ff4)](https://rubygems.org/gems/peal) |
| [Java](https://github.com/peal-social/peal/tree/main/packages/peal-sdk/java) | `social.peal:peal` | [![Maven Central](https://img.shields.io/maven-metadata/v?metadataUrl=https%3A%2F%2Frepo1.maven.org%2Fmaven2%2Fsocial%2Fpeal%2Fpeal%2Fmaven-metadata.xml&label=&color=aa9bfd)](https://central.sonatype.com/artifact/social.peal/peal) |
| [PHP](https://github.com/peal-social/peal-php) | `composer require peal-social/peal` | [![Packagist](https://img.shields.io/packagist/v/peal-social/peal?label=&color=8b87f9)](https://packagist.org/packages/peal-social/peal) |
| [.NET](https://github.com/peal-social/peal/tree/main/packages/peal-sdk/dotnet) | `dotnet add package Peal` | [![NuGet](https://img.shields.io/nuget/v/Peal?label=&color=6973f5)](https://www.nuget.org/packages/Peal) |
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

An agent or a script can work in Peal without the web app:

| | What it is |
|---|---|
| **MCP server** | Built into Peal at `/api/mcp` (Streamable HTTP, OAuth sign-in). An agent works in the one workspace you pick. You decide whether it only reads, writes drafts, or publishes. [MCP docs](https://docs.peal.social/developers/mcp) · [Posting from an agent](https://docs.peal.social/developers/agents) |
| **`peal` CLI** | Drafts, posts, analytics, billing, members and invitations from the terminal, signed in with OAuth. Builds from source today. [CLI docs](https://docs.peal.social/developers/cli) |

## Self-host

A server of your own needs Postgres and an S3-compatible store. Email, billing, captcha, AI and telemetry are optional, and stay off until you set them up.

Clone the repository, copy the settings file outside it and fill it in, then start the stack:

```bash
git clone https://github.com/peal-social/peal.git
cd peal/packages/peal-docker
cp self-host.env.example /srv/peal/peal.env   # your domain and secrets, outside the clone
docker compose --project-name peal --file compose.production.yaml \
  --file compose.self-host.yaml --env-file /srv/peal/peal.env up --build --detach --wait
```

Caddy then serves Peal on your domain, with a Let's Encrypt certificate. The [self-hosting guide](https://docs.peal.social/self-hosting) explains how to register your app on each platform.

## Repositories

Peal lives in two repositories:

| Repo | What it is |
|---|---|
| **[peal](https://github.com/peal-social/peal)** | The monorepo: server, web app, contract, SDKs, CLI, docs, site and infrastructure. Issues and pull requests go here |
| [peal-php](https://github.com/peal-social/peal-php) | Read-only mirror of the PHP SDK, for Packagist. Open issues in `peal` |

## Licensing

Peal has two licences, and this table says which part is under which:

| Part | Licence |
|---|---|
| Peal itself | [FSL-1.1-ALv2](https://github.com/peal-social/peal/blob/main/LICENSE.md): any use except offering a competing service, and Apache-2.0 two years after each release |
| The API contract and the SDKs | [Apache-2.0](https://github.com/peal-social/peal/blob/main/packages/peal-sdk/LICENSE) |

If you find a security issue, write to [security@peal.social](mailto:security@peal.social) and don't open an issue, because issues are public.

<div align="center">

<br>

<a href="https://peal.social"><img src="https://img.shields.io/badge/-Visit_peal.social-ffca00?style=for-the-badge" alt="Visit peal.social" height="36"></a>
&nbsp;
<a href="https://docs.peal.social"><img src="https://img.shields.io/badge/-Read_the_docs-b9a6ff?style=for-the-badge" alt="Read the docs" height="36"></a>
&nbsp;
<a href="https://github.com/peal-social/peal"><img src="https://img.shields.io/badge/-Star_on_GitHub-415eee?style=for-the-badge&logo=github&logoColor=white" alt="Star on GitHub" height="36"></a>

<sub><a href="https://peal.social">peal.social</a></sub>

</div>
