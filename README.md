# fix-zsh-autosuggest-script

One script that repairs an **Oh My Zsh + Powerlevel10k + zsh-autocomplete / zsh-autosuggestions** setup, including on offline machines (e.g. isolated malware-analysis VMs).

## Symptoms it fixes

| Symptom | Cause |
|---|---|
| Tab prints `command not found: _autocomplete__unambiguous` / `_autocomplete__should_add_space` | zsh-autocomplete regression ([#907](https://github.com/marlonrichert/zsh-autocomplete/issues/907), fix pending in PR #903): Oh My Zsh runs `compinit` before the plugin loads, so its helper functions are never registered |
| No live "list as you type" menu | the async helper `zasync` is missing or not autoloaded |
| Powerlevel10k: `Console output during zsh initialization detected` followed by `Cloning into ~/.cache/zsh/zasync ... Could not resolve host: github.com` | zsh-autocomplete `git clone`s zasync at the first prompt, which fails without internet |

## What it does

1. Installs anything missing: Oh My Zsh, `zsh-autosuggestions`, `zsh-syntax-highlighting`, `fast-syntax-highlighting`, `zsh-autocomplete`, `powerlevel10k`, `zasync` (from GitHub, or from an offline bundle).
2. Patches `~/.zshrc` (timestamped backup first; edits live between marker comments so re-runs are idempotent):
   - normalises `plugins=(...)`, with `zsh-autocomplete` last
   - puts `zasync` on `fpath` before Oh My Zsh loads, so the plugin never tries to clone/pull it
   - re-registers `zasync` and the `_autocomplete__*` helpers after Oh My Zsh loads
3. Clears stale `.zcompdump` and p10k instant-prompt caches, then verifies the functions resolve.

## Usage

```sh
# Online machine
./fix-zsh-autocomplete.sh
exec zsh

# Preview changes only
./fix-zsh-autocomplete.sh --dry-run
```

### Offline machines

```sh
# On a machine where everything already works:
./fix-zsh-autocomplete.sh --make-bundle        # -> zsh-bundle.tar.gz next to the script

# Copy fix-zsh-autocomplete.sh + zsh-bundle.tar.gz to the offline machine, then:
./fix-zsh-autocomplete.sh                      # bundle is picked up automatically
# or: ./fix-zsh-autocomplete.sh --from /path/to/zsh-bundle.tar.gz
```

Requires `bash`, `zsh`, `perl`, `tar` (and `git` for online installs). Tested with zsh 5.9 on Arch-based and Ubuntu 24.04 systems.
