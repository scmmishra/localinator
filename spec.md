## Overview

`localinator` is a Go CLI + daemon that provides **local HTTPS domains** with automatic wildcard certificates.

* Every domain added automatically gets a **wildcard cert** (`*.myapp.local`) that covers the apex (`myapp.local`) and all subdomains (`api.myapp.local`, `www.myapp.local`, etc.).
* Config lives in `~/.localinator/config.yml`.
* Daemon runs persistently as a **reverse proxy** on `:443`, hot-reloading config.

---

## Features

### CLI Commands

* **`localinator add <domain> --to <port>`**

  * Adds `127.0.0.1 <domain>` and `127.0.0.1 *.<domain>` to `/etc/hosts`.
  * Generates a wildcard cert (`*.domain`) using mkcert.
  * Saves cert in `~/.localinator/certs/<domain>/`.
  * Updates `config.yml` with mapping.
  * Signals daemon to reload.

* **`localinator remove <domain>`**

  * Removes domain entry from `config.yml`.
  * Removes `/etc/hosts` entry for both `domain` and `*.domain`.
  * Deletes associated cert directory.
  * Signals daemon to reload.

* **`localinator list`**

  * Shows all active mappings (`domain → port`).
  * Indicates cert type = wildcard.

* **`localinator daemon`**

  * Always running HTTPS reverse proxy.
  * Watches `config.yml` for changes (fsnotify).
  * Reloads mappings + certs automatically.

---

## Config & Filesystem

* **Config file:** `~/.localinator/config.yml`

  ```yaml
  mappings:
    myapp.local:
      port: 3000
    blog.local:
      port: 4000
  ```

* **Certs directory:**

  * `~/.localinator/certs/myapp.local/cert.pem`
  * `~/.localinator/certs/myapp.local/key.pem`
    (cert is valid for `myapp.local` + `*.myapp.local`)

---

## Reverse Proxy Daemon

* Single HTTPS listener on port `443`.
* Uses **TLS SNI** to match incoming domains.
* If cert exists for `myapp.local`, it covers both `myapp.local` and all `*.myapp.local`.
* Request routing via config:

  * `myapp.local` → `http://127.0.0.1:3000`
  * `api.myapp.local` → `http://127.0.0.1:3000`
  * `www.blog.local` → `http://127.0.0.1:4000`

---

## Dependencies

* **mkcert** (run `mkcert "*.myapp.local" "myapp.local"`).
* Go stdlib: `net/http`, `crypto/tls`, `httputil`, `yaml`, `fsnotify`.

---

## Security

* `/etc/hosts` edits require sudo.
* Certs + config stored in `~/.localinator/`.
* Daemon runs as non-root after binding.

---

## Example Usage

```bash
# Add myapp.local → port 3000
localinator add myapp.local --to 3000
# Cert covers myapp.local + *.myapp.local

# Add blog.local → port 4000
localinator add blog.local --to 4000
# Cert covers blog.local + *.blog.local

# Remove mapping
localinator remove myapp.local

# List all mappings
localinator list
```

---

## Stretch Goals

* System service integration (systemd / launchd).
* Auto-renew wildcard certs.
* Support multiple target ports per domain (e.g. subdomain → different port).

---

⚡ This v4 spec makes **wildcard certs the default**, so every domain automatically covers its apex + all subdomains.

Do you also want me to design how to handle **subdomain → different port** mappings? (e.g. `api.myapp.local → 3000`, `admin.myapp.local → 4000` while both share one wildcard cert).
