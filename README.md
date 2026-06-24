# GrupoTST DNS Server

Servidor DNS auto-alojado con panel web. Escrito en **JavaScript (Node.js)**, sin TypeScript. Licencia **AGPL-3.0**.

## Funciones

- **Panel web** en el puerto 4000 con login por contraseña (cookies)
- **Servidor DNS** en el puerto 53 (UDP)
- **Upstreams configurables** — Cloudflare por defecto (1.1.1.1 y 1.0.0.1)
- **Reglas DNS locales**: A, AAAA, CNAME, redirect
- **Reglas custom**: dominio, wildcard (`*.ejemplo.com`), regex
- **Listas Pi-hole** (formato hosts) y **AdGuard** (`||dominio^`)
- **Easy Lists**: catálogo preconfigurado activable con un switch o dropdown
- **Modo paranoico**: bloquear todo por defecto
- **Modo oscuro/claro** persistente vía cookie (sin parpadeo)
- **Logs en tiempo real** con filtros
- **Block page** personalizable (color, mensaje, footer)
- **Widget de estado online** que consulta un endpoint remoto
- **Botón "Actualizar datos"** que re-sincroniza todas las listas
- **Página de Créditos** que redirige a GitHub

## Instalación (Debian 13)

```bash
# 1. Instalar Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo bash -
sudo apt install -y nodejs build-essential

# 2. Clonar e instalar
sudo mkdir -p /opt/grupotst-dns && sudo chown $USER:$USER /opt/grupotst-dns
cd /opt/grupotst-dns
git clone https://github.com/TomasYTPro3841/dns-server.git .
npm install

# 3. Configurar
cp credentials.conf.example credentials.conf
nano credentials.conf  # Cambia ADMIN_PASSWORD y SESSION_SECRET

# 4. Build
npm run build

# 5. Probar
sudo node server.js
# Abre http://IP:4000

# 6. Instalar como servicio
sudo cp grupotst-dns.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now grupotst-dns
```

## Configuración

Todo está en `credentials.conf`:

```ini
ADMIN_PASSWORD=tu-contraseña
SESSION_SECRET=usa-openssl-rand-base64-48
WEB_PORT=4000
DNS_LISTEN_PORT=53
DEFAULT_UPSTREAM_DNS_1=1.1.1.1
DEFAULT_UPSTREAM_DNS_2=1.0.0.1
EASY_LISTS_INDEX_URL=https://raw.githubusercontent.com/TomasYTPro3841/dns-server/refs/heads/main/lists.txt
ONLINE_CHECK_URL=https://raw.githubusercontent.com/TomasYTPro3841/dns-server/refs/heads/main/check.txt
CREDITS_URL=https://github.com/TomasYTPro3841/dns-server/blob/main/credits.md
```

## Archivos

- `server.js` — Entry point (web + DNS)
- `dns-server.js` — Servidor DNS UDP en Node.js
- `credentials.conf` — Configuración (no subir a git)
- `config.json` — Datos del usuario (auto-creado)
- `cache.db` — SQLite con cache de listas + logs (auto-creado)
- `src/` — Panel web Next.js en JavaScript

## Comandos

| Comando | Acción |
|---------|--------|
| `sudo systemctl start grupotst-dns` | Iniciar |
| `sudo systemctl stop grupotst-dns` | Detener |
| `sudo systemctl restart grupotst-dns` | Reiniciar |
| `sudo systemctl status grupotst-dns` | Estado |
| `sudo journalctl -u grupotst-dns -f` | Ver logs |
| `dig @localhost example.com` | Probar DNS |

## Licencia

GNU Affero General Public License v3.0 (AGPL-3.0). Ver [LICENSE](LICENSE).

Si modificas el código y lo despliegas en un servidor, debes compartir el código fuente.

## Créditos

Ver [credits.md](https://github.com/TomasYTPro3841/dns-server/blob/main/credits.md).
