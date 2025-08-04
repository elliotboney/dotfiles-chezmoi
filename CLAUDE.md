# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Architecture

This is a **chezmoi-managed dotfiles repository** that provides cross-platform configuration management with template-driven configs, LastPass integration for secrets, and modular shell utility loading.

### Core Components

**Chezmoi Template System**: The repository uses Go templates (`.tmpl` files) that render configurations dynamically based on machine type, OS, and user data. Key template variables include:
- `.chezmoi.os` - Operating system (darwin/linux)
- `.chezmoi.arch` - Architecture (arm64/amd64) 
- `.email` - User email from chezmoi config
- Machine type booleans: `.personal`, `.work`, `.development`, `.server`

**LastPass Integration**: SSH keys and sensitive configurations use `lastpassRaw` template functions to retrieve secrets from LastPass secure notes during rendering, eliminating hardcoded secrets in git.

**Modular Shell Loading**: The `dot_dotfiles/` directory contains a legacy-compatible modular loading system:
- `private_early_init.d/` - Core initialization (colors, helper functions, exports)
- `private_utilities.d/` - 25+ utility modules organized by domain (filesystem, git, networking, etc.)  
- `private_environment.d/` - Environment variables, paths, OS-specific settings
- `private_completions.d/` - Shell completions

**Installation Scripts**: `.chezmoiscripts/` contains templated installation scripts that run automatically:
- `run_before_*` - Pre-installation (encryption setup, LastPass login)
- `run_once_*` - One-time setup (package installation, performance optimization)
- `run_after_*` - Post-installation (documentation, health checks)

## Essential Commands

### Initial Setup
```bash
# Fresh machine setup
chezmoi init --apply elliotboney/dotfiles

# Login to LastPass (required for SSH keys)
lpass login elliotboney@gmail.com
```

### Daily Development Workflow
```bash
# Edit configuration files (opens templates in $EDITOR)
chezmoi edit ~/.zshrc
chezmoi edit ~/.gitconfig  
chezmoi edit ~/.ssh/config

# Preview changes before applying
chezmoi diff

# Apply all changes
chezmoi apply

# Add new files to chezmoi management
chezmoi add ~/.new-config
chezmoi add --template ~/.dynamic-config  # For templated files
chezmoi add --encrypt ~/.secret-file      # For age-encrypted files
```

### Template Development and Testing
```bash
# Test template rendering without applying
chezmoi execute-template < template.tmpl

# Show available template data/variables
chezmoi data

# Preview rendered output of specific file
chezmoi cat ~/.zshrc

# Validate template syntax
chezmoi apply --dry-run
```

### Multi-Machine Synchronization
```bash
# Commit and push changes from chezmoi source directory
chezmoi cd  # Navigate to ~/.local/share/chezmoi
git add . && git commit -S -m "Update configs" && git push

# Pull and apply changes on other machines
chezmoi update  # Equivalent to: git pull && chezmoi apply

# Use helper function for quick commits
pushdots "commit message"  # Available after shell loads
```

### SSH Key Management (LastPass)
SSH keys are managed via LastPass secure notes. Required entries:
- "SSH Private Key" - Private key content
- "SSH Public Key" - Public key content  
- "SSH Config" - SSH configuration with hosts
- "SSH Authorized Keys" - Authorized keys (optional)

### GPG Setup for Commit Signing
```bash
# GPG key is stored in encrypted .gnupg directory
# Key ID: F425B86D4EB372D7 (passphrase-free for automation)
git config --global user.signingkey F425B86D4EB372D7
git config --global commit.gpgsign true
```

## Key Files and Their Purposes

- `dot_zshrc.tmpl` - Main shell configuration with platform/machine-type conditionals
- `private_dot_gitconfig.tmpl` - Git config using template variables for email/GPG key
- `private_dot_ssh/*.tmpl` - SSH keys retrieved dynamically from LastPass
- `Brewfile.tmpl` - Package installation based on machine type and OS
- `.chezmoitemplates/helpers.tmpl` - Reusable template functions and helpers
- `dot_dotfiles/private_utilities.d/dotfiles.sh` - Core dotfiles management functions

## Template Helper Functions

Available in `.chezmoitemplates/helpers.tmpl`:
- `{{ template "is_mac" . }}` - Platform detection
- `{{ template "is_personal" . }}` - Machine type detection  
- `{{ template "colors" }}` - Script color definitions
- `{{ template "log_functions" }}` - Logging helpers for scripts
- `{{ template "command_exists" "command" }}` - Command availability check

## Security Model

- **Age Encryption**: GPG keys and some configs use age encryption with key stored in LastPass
- **LastPass Templates**: SSH keys and sensitive configs use `lastpassRaw` functions
- **GPG Signing**: All commits signed with F425B86D4EB372D7
- **Private Files**: Use `private_` prefix for files containing sensitive data

## Machine Configuration

Machine behavior controlled by chezmoi data variables (set during `chezmoi init`):
- **Personal**: GitHub Copilot, personal apps, media tools
- **Work**: Corporate tools, compliance settings, work-specific packages  
- **Development**: Full dev toolchain, language environments, containers
- **Server**: Minimal packages, security-focused, server optimizations

## Performance Notes

- Scripts use `run_once_` prefix to avoid repeated execution
- Shell startup optimized with lazy loading and compiled functions
- Template rendering typically <50ms for all configs
- Installation scripts are idempotent and use `|| true` for non-critical operations