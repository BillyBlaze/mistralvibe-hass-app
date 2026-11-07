# Changelog

All notable changes to this app will be documented in this file.

## 1.7.1 — 2026-04-30

### Added
- Extended security documentation: token handling, hassio_role, AppArmor, auto_approve risks

## 1.7.0 — 2026-04-30

### Fixed
- resolve_model() now falls back to hardcoded defaults if model resolution returns empty string

## 1.6.9 — 2026-04-30

### Fixed
- SUPERVISOR_TOKEN guard with 5s timeout before HA version query in init script

## 1.6.8 — 2026-04-30

### Fixed
- ha_list_areas tool name corrected in system prompt
- Dockerfile label type: addon → app
- ha-logs and ha-errors aliases now use direct Supervisor API instead of missing ha CLI

## 1.6.7 — 2026-04-30

### Added
- Shell aliases: v (vibe-launcher), ha-config, ha-logs, ha-errors

## 1.6.6 — 2026-04-30

### Added
- ha.md system prompt generated dynamically at startup including HA version and path mapping

## 1.6.5 — 2026-04-30

### Added
- Configurable terminal_font_size (10-24px) and terminal_theme (dark/light)

## 1.6.4 — 2026-04-30

### Added
- Documentation: Le Chat Pro included budget and pay-as-you-go API key setup

## 1.6.3 — 2026-04-30

### Added
- ingress_stream: true for better streaming performance
- ttyd: scrollback=20000, --ping-interval 30, --max-clients 5
- Docker HEALTHCHECK

## 1.6.2 — 2026-04-30

### Fixed
- HA_TOKEN restored in mcp_servers.env — Vibe does not inherit shell environment to MCP child processes

## 1.6.0 — 2026-04-30

### Changed
- Documentation: renamed add-on to app throughout

## 1.5.9 — 2026-04-30

### Removed
- ha_get_vibe_usage MCP tool — session logs contain no token data

## 1.5.8 — 2026-04-30

### Fixed
- Removed duplicate entries in de.yaml translations

## 1.5.5 — 2026-04-30

### Fixed
- ha.md and VIBE.md copied correctly on startup
- reset_data and session_persistence options restored in config.yaml
- Removed duplicate entries in en.yaml translations

## 1.5.1 — 2026-04-29

### Security
- HA_TOKEN removed from config.toml template — passed via inherited environment only

## 1.4.8 — 2026-04-29

### Fixed
- py3-pip added to apk packages (required for fastmcp/httpx)
- Removed invalid --mouse flag from ttyd

## 1.4.0 — 2026-04-29

### Changed
- Replaced hass-mcp with custom FastMCP server (server.py)
- ha_get_error_log limited to max 200 lines
- ha_list_entities, ha_list_automations limited to 50 results by default

## 1.0.0 — 2026-04-29

### Added
- Initial release
- Mistral Vibe CLI installed via uv tool install
- Custom MCP server for Home Assistant Core API access via Supervisor token
- Browser terminal (ttyd) via HA ingress
- App options: API key, active model, default agent, auto-approve, telemetry, CLI auto-update
- Persistent state under /data/vibe (config, history, session logs)
- AppArmor profile
- Multi-arch builds: amd64, aarch64

## 1.7.4 — 2026-04-30

### Added
- ha CLI binary installed in Dockerfile (from github.com/home-assistant/cli)
- ha-logs and ha-errors aliases restored to use ha CLI

## 1.7.3 — 2026-04-30

### Changed
- CHANGELOG rewritten with full history

## 1.7.2 — 2026-04-30

### Fixed
- Documentation: replaced outdated hass-mcp references with custom MCP server
- Removed addon_config from filesystem layout (not mounted)

## 1.7.5 — 2026-04-30

### Fixed
- SUPERVISOR_TOKEN exported in bashrc so ha CLI authenticates automatically


## 1.7.9 — 2026-05-02

### Fixed
- Increased xterm.js scrollSensitivity for better touch scrolling on iOS HA app


## 1.8.0 — 2026-05-02

### Added
- mobile command: detaches tmux for native iOS scrolling. Use 'v' to re-enter Vibe.


## 1.8.1 — 2026-05-02

### Changed
- m alias detaches tmux (mobile mode), v alias re-attaches to existing tmux session
- Switch freely between Vibe and bash with v and m


## 1.8.2 — 2026-05-02

### Added
- Automatic mobile detection: iOS clients (iPhone/iPad) get tmux-free session for native touch scrolling
- TCP proxy on port 7681 routes to desktop (7682, with tmux) or mobile (7683, no tmux) ttyd

## 2.0.0 — 2026-11-07

### Changed
- **Base image migrated from Alpine/HA base to `debian:bookworm-slim`.**
  The mistral-vibe PyPI package ships Rust extensions and a rusty_v8 prebuilt
  binary that does not exist for musl (`librusty_v8_*_alpine-linux-musl.a.gz`
  returns HTTP 404 on every release), making the Alpine/HA-base build impossible.
  Debian/glibc installs prebuilt manylinux wheels via uv with no Rust toolchain
  and no V8 source build.
- Python runtime now provisioned via uv (managed Python 3.13, tool installs in
  `/opt`), replacing the base-image Python.
- s6-overlay v2.1.0.2 installed from upstream tarballs with a merged-usr-safe
  procedure: extraction to a staging directory, then copying into `/usr/bin`
  so Debian's `/bin → usr/bin` symlink is never clobbered.

### Added
- bashio v0.19.0 installed explicitly (preinstalled in HA base images, missing
  on Debian). Required by all rootfs service scripts.
- ttyd download now maps HA build arch to the correct upstream release asset
  (`amd64 → ttyd.x86_64`, `aarch64 → ttyd.aarch64`, `i386 → ttyd.i686`) and
  fails the build with a clear error on unsupported architectures.

### Fixed
- `/init` is actually present at container start (s6 tarball is now extracted
  into the image root, not a subdirectory).
- s6-overlay installation no longer breaks `/bin` on merged-usr systems
  (previously `/bin/bash` disappeared at build time, causing exit code 126 at
  container start).
- All 11 s6 helper scripts under `/usr/bin` (`printcontenv`, `with-contenv`,
  `justc-envdir`, `fix-attrs`, `logutil-*`, `with-retries`) are installed;
  previously lost to an over-broad tar exclude pattern, which aborted s6 init
  with `unable to spawn printcontenv`.
- AppArmor profile grants execute (`ixr`) on `/etc/s6/**`; previously only
  `/etc/cont-init.d/**`, `/etc/services.d/**` etc. were covered, so the kernel
  denied execution of the s6 init scripts at container start.

### Notes
- Verified working end-to-end on HA OS x86_64/amd64: image builds, container
  starts, ttyd terminal reachable via Ingress, MCP bridge authenticated via
  SUPERVISOR_TOKEN, Vibe CLI runs against a Vibe Pro API key.
- Maintainer label updated for the fork.
