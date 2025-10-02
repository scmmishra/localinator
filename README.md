# Localinator

**A Go-based CLI + daemon that maps local domains to ports with automatic wildcard HTTPS certificates and a built-in reverse proxy.**

---

## ✨ Features

* 🔒 Automatic **wildcard TLS certs** (`myapp.local`, `*.myapp.local`) via [mkcert](https://github.com/FiloSottile/mkcert).
* ⚡ Simple CLI: add/remove/list local domains.
* 🌍 Updates `/etc/hosts` automatically.
* 🔁 Persistent **HTTPS reverse proxy daemon** running on port 443.
* 🗂 Config stored in `~/.localinator/config.yml`.
* 🔄 Hot reload when config changes.

---

## 🚀 Installation

```bash
# Clone & build
git clone https://github.com/you/localinator.git
cd localinator
go build -o localinator .
```

You’ll also need [`mkcert`](https://github.com/FiloSottile/mkcert) installed.

---

## ⚡ Usage

### Add a domain

```bash
sudo ./localinator add myapp.local --to 3000
```

* Adds `myapp.local` + `*.myapp.local` to `/etc/hosts`.
* Generates certs (valid for `myapp.local` + subdomains).
* Updates config.
* Signals daemon to reload.

### Remove a domain

```bash
sudo ./localinator remove myapp.local
```

### List all mappings

```bash
./localinator list
```

### Run the daemon

```bash
./localinator daemon
```

---

## 🔧 Config

Stored at `~/.localinator/config.yml`:

```yaml
mappings:
  myapp.local:
    port: 3000
  blog.local:
    port: 4000
```

Certs are stored at:

```
~/.localinator/certs/<domain>/cert.pem
~/.localinator/certs/<domain>/key.pem
```

---

## 🛠 How it Works

1. CLI updates `/etc/hosts` and config.
2. Wildcard cert generated for each domain (`*.domain`).
3. Daemon listens on port 443, handles **TLS SNI**, and proxies requests to the mapped local service.

   * `https://myapp.local` → `http://127.0.0.1:3000`
   * `https://api.myapp.local` → `http://127.0.0.1:3000`

---

## 📦 Roadmap

* Subdomain-specific port mapping (e.g. `api.myapp.local → 3000`, `admin.myapp.local → 4000`).
* System service integration (systemd / launchd).
* Web dashboard for managing domains.

---

## ⚖️ License

MIT
