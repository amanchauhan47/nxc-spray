# nxc-spray

A wrapper around [NetExec (nxc)](https://github.com/Pennyw0rth/NetExec) that automates full-coverage credential spraying: every chosen protocol, against **both domain and local accounts**, without stopping at the first valid login.

## Why

Running NetExec manually for a thorough spray means remembering to:
- test every relevant protocol (SMB, WinRM, SSH, MSSQL, LDAP, RDP)
- test both domain accounts **and** local SAM accounts (`--local-auth`)
- add `--continue-on-success` so it doesn't stop after the first hit

`nxc-spray` does all of that in one command, in an order weighted toward protocols most likely to give fast, actionable access.

## Features

- **Dual auth mode** — every protocol is sprayed against domain accounts, then local accounts, automatically.
- **Continues past first hit** — `--continue-on-success` is always applied, so you get every valid credential per host, not just the first.
- **Sensible protocol order** — `smb → winrm → ssh → mssql → ldap → rdp`, prioritizing protocols with the fastest, clearest signal and leaving slow/lockout-prone protocols (RDP) for last.
- **Flexible targets** — single IP, space-separated IPs, or a file of IPs (one per line).
- **Flexible creds** — single username/password or wordlist files, same as NetExec.
- **Color-coded output** — each spray block is printed in a distinct color per auth mode, so domain vs. local results are easy to tell apart at a glance, without clashing with NetExec's own red/green/orange output.
- **Built-in usage/help** — run with no arguments to see full usage and examples.

## Requirements

- [NetExec](https://github.com/Pennyw0rth/NetExec) installed and available as `nxc` on your `$PATH`
- Bash

## Installation

```bash
git clone https://github.com/<your-username>/nxc-spray.git
cd nxc-spray
chmod +x nxc-spray.sh
```

## Usage

```bash
nxc-spray.sh <protocols|all> <targets> -u <username> -p <password>
```

| Argument | Description |
|---|---|
| `<protocols\|all>` | Comma-separated list of protocols, or `all`. Supported: `smb,winrm,ssh,mssql,ldap,rdp` |
| `<targets>` | Single IP, space-separated IPs (quoted), or a file with one IP per line |
| `-u` | Username, or a file of usernames |
| `-p` | Password, or a file of passwords |

### Examples

```bash
# Single target, single credential, all protocols
nxc-spray.sh all 10.10.10.10 -u amxn -p password

# Specific protocols only, targets from file
nxc-spray.sh smb,ldap,winrm hosts.txt -u amxn -p password

# Multiple targets inline
nxc-spray.sh all "10.10.10.10 10.10.10.11" -u amxn -p password

# Full wordlist spray across all protocols, all targets
nxc-spray.sh all targets.txt -u users.txt -p passwords.txt
```

Running the script with fewer than 4 arguments prints the full usage block automatically.

## How it works

For every protocol selected, the script runs two passes:

1. **Domain accounts** — `nxc <proto> <target> -u <user> -p <pass> --continue-on-success`
2. **Local accounts** — `nxc <proto> <target> -u <user> -p <pass> --continue-on-success --local-auth`

Each pass is labeled and color-coded in the terminal so you can scan long output and immediately tell which block is which.

## Disclaimer

This tool is intended for use in authorized penetration tests, CTFs, and training environments (such as OSCP labs) only. Do not use it against systems you do not have explicit permission to test.

## License

MIT
