# Changelog

All notable changes to the Experimental plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2026-02-24

### Added
- Initial release of experimental plugin
- `/mvp start` command for brainstorming and project scaffolding
- `/mvp build` command for autonomous building with parallel AI agents
- `/mvp status` command for progress dashboard
- `/mvp summary` command for HTML analytics page generation
- JavaScript stack support (Vite + TypeScript + React 19 + TailwindCSS 4 + SQLite)
- Elixir stack support (Phoenix Framework + LiveView + SQLite)
- Persistent state management via `.mvp/` directory
- Lock-based agent concurrency control
- Quality review agent pattern for completed tasks
- Auto git commits at task completion checkpoints
