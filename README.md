# Yubal Syncthing

An independent hobby fork of [Yubal](https://github.com/guillevc/yubal), the self-hosted YouTube Music downloader. It downloads and organizes music into a local library and can share that library with devices through Syncthing.

> [!IMPORTANT]
> This repository is independent from the original Yubal repository. The original Git history was not retained when this checkout was made independent. It does not automatically sync with, pull from, or send changes to the original project. Updates to either repository must be handled manually.
>
> Please check out and credit the [original Yubal project](https://github.com/guillevc/yubal) and its creator, [Guillermo Alfonso Varela Chouciño](https://github.com/guillevc). This fork is based on that work and retains its MIT license and copyright notice.

## What it does

- Download YouTube Music tracks, albums, and playlists through the web interface.
- Organize files with tags and album artwork, and create M3U playlists.
- Subscribe to playlists for scheduled syncs.
- Optionally fetch lyrics and apply ReplayGain.
- Share one music directory with Syncthing and an optional Navidrome server.
- Process downloads and metadata lookups concurrently with configurable limits.

This is a personal hobby project. It is maintained independently and may not track upstream changes or releases.

## Run with Docker Compose

The Compose file builds Yubal from this checkout and shares `./data` with Syncthing. Navidrome is optional and starts only when its profile is enabled.

```sh
docker compose up --build -d
```

Open Yubal at `http://localhost:8000`. Syncthing's web interface is available on port `8384` inside the Compose network; use your deployment's reverse proxy if you want to access it remotely. Syncthing device pairing and folder sharing are still configured in Syncthing's interface or app. Yubal provides the shared music directory; it does not pair devices for you.

To start Navidrome as well:

```sh
docker compose --profile media-server up --build -d
```

Set `PUID`, `PGID`, and `TZ` in your environment to match your host. The example Compose file publishes Syncthing's device ports on the host; adjust those port mappings to suit your network and firewall.

For private playlists, Liked Music, age-restricted content, or Premium quality, upload YouTube cookies from Yubal's Settings page. Cookies are sensitive account credentials; do not commit them.

## Configuration

The most relevant environment variables are:

| Variable | Purpose | Default |
| --- | --- | --- |
| `YUBAL_AUDIO_FORMAT` | Output format: `opus`, `mp3`, or `m4a` | `opus` |
| `YUBAL_SCHEDULER_ENABLED` | Enable scheduled playlist sync | `true` |
| `YUBAL_SCHEDULER_CRON` | Sync schedule | `0 0 * * *` |
| `YUBAL_FETCH_LYRICS` | Fetch `.lrc` lyrics | `true` |
| `YUBAL_YTMUSIC_LYRICS_FALLBACK` | Use YouTube Music lyrics when LRCLIB has no match | `true` |
| `YUBAL_REPLAYGAIN` | Apply ReplayGain tags | `true` |
| `YUBAL_DOWNLOAD_CONCURRENCY` | Concurrent track downloads and processing (1–8) | `3` |
| `YUBAL_METADATA_CONCURRENCY` | Concurrent metadata enrichment (1–8) | `3` |
| `YUBAL_TZ` | Scheduler timezone | `UTC` |
| `PUID` / `PGID` | Host user and group IDs for file permissions | `1000` |

See [`.env.example`](.env.example) for additional settings.

## Changes in this fork

Compared with the original project, this fork currently adds or changes:

- A Compose setup that shares the Yubal music directory with Syncthing and optionally Navidrome.
- Bounded parallel track processing, plus configurable concurrent metadata lookups using separate YouTube Music clients per worker.
- Refreshing the playlist metadata client when the uploaded cookies file is added or changed, so the subscription page can use the current credentials.
- A portable Compose example with project-relative data/config directories and configurable concurrency.

### Bug fix

- Fixed stale YouTube Music credentials on the playlist subscription page after cookies are uploaded or replaced.

Parallel track processing and metadata lookup are performance improvements, not bug fixes. They keep duplicate output paths serialized, preserve playlist order for metadata progress, and leave metadata cache access on the main thread.

These changes are documented from this fork's local commit history; they have not all been independently compared against the latest upstream revision.

## Development

This repository contains the Python downloader/API and the web interface. Install the tools listed in [`mise.toml`](mise.toml), then run:

```sh
just install
just check
```

To run the development servers, use `just dev`. To build the Docker image, use `just docker-build`.

## Credits and license

Yubal was created by [Guillermo Alfonso Varela Chouciño](https://github.com/guillevc). This fork is derived from [guillevc/yubal](https://github.com/guillevc/yubal); please visit the original repository for upstream development and releases. The original MIT license and copyright notice are retained in [`LICENSE`](LICENSE).

This fork is maintained separately. It is not an official Yubal release, is not endorsed by the original maintainer, and will not automatically synchronize with the original repository.
