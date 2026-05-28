---
title: "Manually Installing OpenAI's Codex CLI Agent on Linux"
modified_date: "2026-05-28"
last_modified_at: "2026-05-28"
---
**Update:** OpenAI [has released a curl-based install script](https://github.com/openai/codex/commit/75b7e06621ac38c2b26a73832e797d49a7307f2b) for Codex CLI, so the instructions below are now deprecated. The native Linux installation methods for both Codex CLI and Claude Code are now effectively identical.

---

OpenAI's current [official installation instructions for Codex CLI](https://developers.openai.com/codex/cli#cli-setup) recommend using [npm](https://www.npmjs.com/) to install the agent on Linux. This is painless if you're already using npm, but if you're not, this introduces a ridiculous amount of dependency bloat for a tool that doesn't actually depend on Node.js.

The solution to this problem, as referenced in the [README from the official GitHub repository](https://github.com/openai/codex?tab=readme-ov-file#installing-and-running-codex-cli), is to manually install the agent. OpenAI is shipping prebuilt binaries for a bunch of different platforms, so for a modern, mainstream Linux distribution this is as simple as downloading and extracting the correct `.tar.gz` file from GitHub.

## Sample Installation Script

Since the official documentation for this installation method isn't particularly detailed, and since Codex CLI doesn't seem to have a built-in update mechanism at the moment, I wrote a simple, proof-of-concept Bash script to automate the process of manually installing and updating Codex CLI:
```bash
#!/bin/bash
set -x
mkdir -p ~/.local/bin
wget -q https://github.com/openai/codex/releases/latest/download/codex-x86_64-unknown-linux-musl.tar.gz
tar -xzf codex-x86_64-unknown-linux-musl.tar.gz
mv codex-x86_64-unknown-linux-musl ~/.local/bin/codex
rm codex-x86_64-unknown-linux-musl.tar.gz
```

If demand for this installation method increases on our end, I may go back and add some useful features to this script, such as:
1. Support for different CPU architectures, since OpenAI is currently providing binaries for both `x86_64` and `aarch64` systems.
2. Automatic selection of `musl` versus `gnu` binaries based on platform support, instead of defaulting to `musl`.
3. Automatic creation of a download directoy with `mktemp`, instead of downloading to the current working directory.
4. A sanity check to make sure `~/.local/bin` is in `$PATH` and/or a fallback to `~/bin` as a backup option.

## Closing Thoughts

I'm surprised OpenAI doesn't already provide an official installation script for Codex CLI, but I assume they will in the future. In contrast, Anthropic recommends their [script-based "native install" method](https://code.claude.com/docs/en/setup#install-claude-code) for Claude Code by default, and have already deprecated their npm installation method.
