# ssh-bootstrap-new-key

Small helper script to bootstrap a new SSH key onto a server
using one of several existing (old) SSH private keys.

The script is intentionally simple, explicit, and non-idempotent.
It is designed for controlled, one-off key rotation.

---

## What it does

- Tries a list of **existing (old) SSH private keys** in a defined order
- As soon as one key works, **appends a new public key** to `~/.ssh/authorized_keys`
- Verifies that login using the **new key** works
- Prints which old key succeeded, or a clear error if none worked

---

## Typical use case

### Migrating to a new laptop / workstation

You have:
- many servers
- existing SSH access via old keys
- a new machine with a newly generated SSH key

Instead of manually logging into each server and editing `authorized_keys`,
this script lets you:

1. Temporarily keep your old private keys
2. Use them once to install the new key
3. Verify the new key works
4. Remove the old keys afterwards

No agent magic, no implicit fallback, no guessing.

---

## Usage

```bash
ssh_bootstrap_new_key <user> <host> [port]
```

### Examples

```bash
ssh_bootstrap_new_key ubuntu server.example.com
ssh_bootstrap_new_key ubuntu 203.0.113.10 2222
```

---

## How it works (short)

1. The script tries each configured **old private key** in order
2. When a key successfully authenticates:
   - the new public key is appended to `~/.ssh/authorized_keys`
3. The script then verifies that login with the **new private key** works
4. Execution stops after the first successful old key

If none of the old keys work, the script exits with an error.

---

## Assumptions & design choices

- SSH key authentication already works on the target system
- The script is **not idempotent**
  - running it multiple times will append the key multiple times
- Intended for **manual, conscious execution**
- No batch mode, no fleet automation
- No root fallback

This is a bootstrap tool, not a configuration management system.

---

## Configuration

Edit the script to adjust paths if needed.

### New key (the future default)

```bash
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

### Old keys (tried in order)

```bash
~/.ssh/old/id_ed25519
~/.ssh/old/id_rsa
~/.ssh/old/otc_pk
```

---

## Exit codes

- `0` – new key successfully installed and verified
- `1` – none of the old keys worked
- `2` – invalid usage or missing key files
- `3` – new key appended, but verification with new key failed

---

## Installation

### Local (recommended)

```bash
mkdir -p ~/bin
cp bin/ssh_bootstrap_new_key ~/bin/
chmod +x ~/bin/ssh_bootstrap_new_key
```

Ensure `~/bin` is in your `PATH`.

---

## Security notes

- Old private keys should only be kept temporarily
- Remove old keys from servers after successful migration
- This script makes **direct changes** to `authorized_keys`
- Review before use on production systems

---

## License

MIT
