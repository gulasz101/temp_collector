# Secrets

**This repository is PUBLIC.** There is no plaintext secret in it, and there must never be one.

## `.env` — untracked

The live docker-compose stack reads its configuration from `.env`. That file is **gitignored and
untracked**. It *was* tracked (and therefore public) from 2025-12-23 until 2026-10-11, when every
credential in it was **rotated** — see the homelab-2nd board, ticket **H2-218**.

`.env` must stay untracked. If you ever find yourself doing `git add -f .env`, stop.

## `.env.sops` — the encrypted copy

The **rotated** values are stored here, encrypted with [sops](https://getsops.io/) +
[age](https://age-encryption.org/).

- The age **public** recipient lives in `.sops.yaml`.
- The age **private** key is **not** in this repo (it is in the password manager).

### Decrypt

⚠️ sops **cannot infer the dotenv format from the `.env.sops` filename**, so the type flags are
**required**. The obvious command fails:

```bash
$ sops -d .env.sops
Could not unmarshal input data: invalid character 'I' looking for beginning of value
```

Use this instead:

```bash
SOPS_AGE_KEY_FILE=~/.keys/age-homelab-2nd.txt \
  sops -d --input-type dotenv --output-type dotenv .env.sops > .env
```

Never pipe the output into chat, a log, or a terminal you are screen-sharing. Verify a round-trip by
hash, not by eye:

```bash
sops -d --input-type dotenv --output-type dotenv .env.sops > /tmp/x && diff <(sort /tmp/x) <(sort .env); rm -f /tmp/x
```

### Re-encrypt after changing `.env`

```bash
sops -e --input-type dotenv --output-type dotenv .env > .env.sops
```

## Note for the migration

This legacy stack is being retired into k3s. The replacement stores its secrets as proper
Kubernetes `Secret` manifests (`data`/`stringData`), which the homelab-2nd `.sops.yaml` rule
(`encrypted_regex: ^(data|stringData)$`) encrypts and which decrypt with a plain `sops -d`.
