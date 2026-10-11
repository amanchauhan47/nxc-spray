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
- **Flexible creds** — single username/password, wordlist files, or NTLM hashes (`-H`), same as NetExec.
- **Pass-the-hash support** — spray NTLM hashes instead of passwords when you've got hashes but no cleartext.
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
nxc-spray.sh <protocols|all> <targets> -u <username> -H <nthash>
```

| Argument | Description |
|---|---|
| `<protocols\|all>` | Comma-separated list of protocols, or `all`. Supported: `smb,winrm,ssh,mssql,ldap,rdp` |
| `<targets>` | Single IP, space-separated IPs (quoted), or a file with one IP per line |
| `-u` | Username, or a file of usernames |
| `-p` | Password, or a file of passwords |
| `-H` | NTLM hash, or a file of hashes — use **instead of** `-p`, not alongside it |

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

# Pass-the-hash spray with a wordlist of NTLM hashes
nxc-spray.sh all targets.txt -u users.txt -H hashes.txt

# Single hash against specific protocols
nxc-spray.sh smb,winrm 10.10.10.10 -u amxn -H aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0
```

Running the script with fewer than 4 arguments prints the full usage block automatically.

> **Note on pass-the-hash:** `-H` relies on NTLM authentication, so it's meaningful for `smb`, `winrm`, `mssql`, and `rdp`. It generally won't produce results against `ssh` (no NTLM concept) or `ldap` (Kerberos/simple bind). The script won't block you from trying, but don't expect hits there.

## How it works

For every protocol selected, the script runs two passes:

1. **Domain accounts** — `nxc <proto> <target> -u <user> -p/-H <cred> --continue-on-success`
2. **Local accounts** — `nxc <proto> <target> -u <user> -p/-H <cred> --continue-on-success --local-auth`

Each pass is labeled and color-coded in the terminal so you can scan long output and immediately tell which block is which.

The local-auth pass is automatically **skipped** for protocols that don't support it (`ssh`, `ldap`), since NetExec rejects `--local-auth` outright for those.

## Known caveats

- **LDAP anonymous bind false positive:** if your username/password lists contain (or get padded with) an empty entry, NetExec may report `[+] domain\:` as a success against LDAP. This is just an anonymous LDAP bind succeeding — a normal, unauthenticated operation most DCs allow — not a cracked credential. Don't mistake it for a real hit.
- **Credential wordlists must exist relative to where you run the script.** If `users.txt`/`passwords.txt`/`hashes.txt` live in a different directory than your current shell, NetExec may silently treat the filename as a literal username/password/hash instead of reading it as a wordlist. Use correct relative or absolute paths.

## Disclaimer

This tool is intended for use in authorized penetration tests, CTFs, and training environments (such as OSCP labs) only. Do not use it against systems you do not have explicit permission to test.

## License

MIT
