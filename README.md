# GrupoTST DNS Server

Servidor DNS auto-alojado con panel web de administración, construido en **Node.js puro** (sin TypeScript). Soporta listas en formato Pi-hole y AdGuard, reglas custom de bloqueo/permiso, reglas DNS locales (A, AAAA, CNAME, redirect), **Easy Lists** preconfiguradas activables con un switch, y upstreams DNS configurables (Cloudflare por defecto).

> **Licencia**: GNU Affero General Public License v3.0 (AGPL-3.0). Ver [LICENSE](LICENSE).

## ✨ Características

### Panel web (puerto 4000)
- **Dashboard con widgets**: estado online, consultas totales, bloqueadas, permitidas, reglas custom, listas activas, upstreams, dominios cacheados, gráfico de actividad 24h
- **Botón "Actualizar datos"**: re-sincroniza el índice de Easy Lists + todas las listas activas
- **Widget de estado online**: consulta `check.txt` → HTTP 200 = online, otro = offline
- **Modo oscuro** integrado con toggle en el sidebar
- **Página de Créditos**: redirige a `credits.md` en GitHub

### Servidor DNS (puerto 53, UDP)
- **Node.js puro** con `better-sqlite3` (sin Bun, sin Docker, sin TypeScript)
- **Upstreams configurables** en orden de prioridad (Cloudflare 1.1.1.1 y 1.0.0.1 por defecto)
- **Modo paranoico**: opción para bloquear todo por defecto (allow-list mode)

### Reglas DNS
- **Reglas custom** de bloqueo/permiso con 3 tipos de match:
  - Dominio (con subdominios): `google.com` bloquea también `www.google.com`
  - Wildcard: `*.google.com` bloquea solo subdominios
  - Regex: `\.doubleclick\.(net|com)$`
- **Reglas locales** (máxima prioridad):
  - `A` — Resuelve a IP IPv4 fija
  - `AAAA` — Resuelve a IP IPv6 fija
  - `CNAME` — Apunta a otro dominio
  - `redirect` — Redirige a IP o dominio
- **Las reglas allow siempre tienen prioridad sobre las block**

### Listas DNS
- **Formato adblock** (`||dominio^`) — por defecto
- **Formato Pi-hole** (hosts: `0.0.0.0 dominio`)
- **Formato custom** (un dominio por línea)
- **Campo comentario** para información interna
- **Sincronización automática** desde URLs remotas
- **Subida de archivo** y bulk-add para listas custom

### Easy Lists (catálogo preconfigurado)
- Lee el índice desde `lists.txt` en el repo de GitHub
- **Listas simples**: activar/desactivar con un switch
- **Listas con grados**: popup desplegable (ej: Protección Básica, Media, Alta, Máxima...)
- **Soporta info en paréntesis** en los nombres (ej: "(No Bloquea Whatsapp)")
- **Cachea los dominios** en `cache.db` al activar
- **Re-escaneo automático cada 1 hora** + botón manual de refresh

### Almacenamiento
- **`credentials.conf`** — Contraseña y configuración de puertos
- **`config.json`** — Toda la configuración del usuario (upstreams, reglas, listas, easy lists, settings)
- **`cache.db`** — SQLite con cache de listas + logs DNS

## 📦 Requisitos

- Debian 13 (Trixie) o similar
- Node.js 20+ (recomendado 22)
- npm
- Permisos de root (para el puerto 53)

## 🚀 Instalación rápida (Debian 13)

### 1. Instalar Node.js 22

```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo bash -
sudo apt install -y nodejs build-essential
```

### 2. Clonar el repositorio

```bash
sudo mkdir -p /opt/grupotst-dns
sudo chown $(whoami):$(whoami) /opt/grupotst-dns
cd /opt/grupotst-dns
git clone https://github.com/TomasYTPro3841/dns-server.git .
```

### 3. Instalar dependencias

```bash
npm install
```

### 4. Configurar credenciales

```bash
cp credentials.conf.example credentials.conf
nano credentials.conf
```

Edita al menos:
- `ADMIN_PASSWORD` — tu contraseña del panel web
- `SESSION_SECRET` — genera una con `openssl rand -base64 48`

### 5. Build de producción

```bash
npm run build
```

### 6. Probar manualmente

```bash
sudo node server.js
```

Abre `http://IP-DE-TU-SERVIDOR:4000` e inicia sesión con tu contraseña.

### 7. Instalar como servicio systemd

```bash
sudo cp grupotst-dns.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now grupotst-dns

# Verificar
sudo systemctl status grupotst-dns
sudo journalctl -u grupotst-dns -f
```

> 💡 Si el puerto 53 está en uso por `systemd-resolved`, desactívalo primero:
> ```bash
> sudo systemctl disable --now systemd-resolved
> sudo rm /etc/resolv.conf
> echo "nameserver 1.1.1.1" | sudo tee /etc/resolv.conf
> sudo systemctl restart grupotst-dns
> ```

## 📋 Configuración

### credentials.conf

Archivo de configuración de secrets y puertos. **No lo subas a git** (está en `.gitignore`).

```ini
# Admin password for the web panel
ADMIN_PASSWORD=changeme

# Secret used to sign session cookies (use: openssl rand -base64 48)
SESSION_SECRET=change-me-to-a-long-random-string

# Web panel settings
WEB_HOST=0.0.0.0
WEB_PORT=4000

# DNS server settings (port 53 requires root)
DNS_LISTEN_HOST=0.0.0.0
DNS_LISTEN_PORT=53

# Default upstream DNS (Cloudflare)
DEFAULT_UPSTREAM_DNS_1=1.1.1.1
DEFAULT_UPSTREAM_DNS_2=1.0.0.1

# Easy Lists index URL (scanned every 1h)
EASY_LISTS_INDEX_URL=https://raw.githubusercontent.com/TomasYTPro3841/dns-server/refs/heads/main/lists.txt

# Online check URL (HTTP 200 = online)
ONLINE_CHECK_URL=https://raw.githubusercontent.com/TomasYTPro3841/dns-server/refs/heads/main/check.txt

# Credits URL (the Credits page redirects here)
CREDITS_URL=https://github.com/TomasYTPro3841/dns-server/blob/main/credits.md
```

### Estructura de archivos

```
/opt/grupotst-dns/
├── server.js              # Entry point (web + DNS coordinator)
├── dns-server.js          # DNS server UDP (puerto 53)
├── credentials.conf       # Secrets y puertos (NO subir a git)
├── config.json            # Configuración del usuario (auto-creada)
├── cache.db               # SQLite: cache de listas + logs (auto-creada)
├── grupotst-dns.service   # Servicio systemd
├── package.json
├── credentials.conf.example
├── README.md
├── INSTALL.md
├── LICENSE                # GNU AGPL v3
├── public/
└── src/                   # Next.js app (React + JS)
    ├── app/
    │   ├── api/           # API routes (Next.js Route Handlers)
    │   ├── dashboard/     # Panel web
    │   └── credits/       # Redirect a GitHub
    ├── components/
    │   ├── dashboard/     # Componentes del panel
    │   └── ui/            # shadcn/ui components
    └── lib/               # storage, auth, parser, etc.
```

## 🎯 Cómo usar

### Acceder al panel

1. Abre `http://IP-DE-TU-SERVIDOR:4000` en el navegador
2. Inicia sesión con la contraseña de `credentials.conf`

### Configurar el DNS en tus dispositivos

Una vez el servidor corriendo en el puerto 53:

- **Router**: configura el servidor DNS DHCP para repartir la IP de tu servidor
- **Linux**: `/etc/resolv.conf` o NetworkManager
- **macOS**: Preferencias del Sistema → Red → Avanzado → DNS
- **Windows**: Panel de Control → Red e Internet → Conexiones de red → Propiedades → IPv4 → DNS

### Activar Easy Lists

1. Ve a la pestaña **Easy Lists** en el panel
2. Pulsa **"Actualizar índice"** para descargar el catálogo desde `lists.txt`
3. Activa las listas con el switch (simples) o selecciona un grado del desplegable (popup)
4. Los dominios se cachean automáticamente en `cache.db`

### Botón "Actualizar datos"

En el Dashboard, este botón:
1. Re-descarga el índice de Easy Lists (`lists.txt`)
2. Re-descarga el contenido de todas las Easy Lists activas
3. Re-descarga el contenido de todas las listas custom activas

## 📊 Verificación

```bash
# Estado del servicio
sudo systemctl status grupotst-dns

# Probar el DNS
dig @localhost example.com      # Debería devolver IP
dig @localhost google.com       # Depende de tus reglas

# Ver puertos
sudo ss -ulnp | grep :53        # DNS UDP
sudo ss -tlnp | grep :4000      # Web panel

# Ver logs
sudo journalctl -u grupotst-dns -f
```

## 🔄 Actualizar

```bash
cd /opt/grupotst-dns
git pull
npm install
npm run build
sudo systemctl restart grupotst-dns
```

## 💾 Backup

```bash
sudo cp /opt/grupotst-dns/config.json ~/dns-config-backup.json
sudo cp /opt/grupotst-dns/cache.db ~/dns-cache-backup.db
sudo cp /opt/grupotst-dns/credentials.conf ~/dns-credentials-backup.conf
```

Para restaurar, copia los archivos de vuelta y reinicia el servicio.

## 🐛 Troubleshooting

### El puerto 53 está en uso

```bash
# Ver qué usa el puerto 53
sudo ss -ulnp | grep :53

# Si es systemd-resolved, desactívalo:
sudo systemctl disable --now systemd-resolved
sudo rm /etc/resolv.conf
echo "nameserver 1.1.1.1" | sudo tee /etc/resolv.conf
```

### Error: better-sqlite3 not found

```bash
npm install better-sqlite3
npm run build
sudo systemctl restart grupotst-dns
```

### El DNS no resuelve

1. Verifica que el servicio esté corriendo: `sudo systemctl status grupotst-dns`
2. Verifica que el puerto 53 esté escuchando: `sudo ss -ulnp | grep :53`
3. Prueba una consulta: `dig @127.0.0.1 example.com`
4. Revisa los logs: `sudo journalctl -u grupotst-dns -f`

## 📝 Formato de lists.txt

El archivo `lists.txt` (índice de Easy Lists) usa este formato:

```
-- Nombre de la Sección
ID : Nombre (info opcional) : "https://url.com/lista.txt"
ID : Nombre : "Grado1"-"https://url1.txt"_"Grado2"-"https://url2.txt"
```

- Las secciones empiezan con `-- `
- Las listas simples tienen una sola URL entre comillas
- Las listas popup tienen varios pares `"Grado"-"URL"` separados por `_`
- Los nombres pueden contener paréntesis con información (se mantienen)
- Los nombres pueden contener dos puntos (se hace split solo en el primero)

Ejemplo completo: https://raw.githubusercontent.com/TomasYTPro3841/dns-server/refs/heads/main/lists.txt

## 📜 Licencia

Copyright (C) 2025 TomasYTPro3841

Este programa es software libre: puedes redistribuirlo y/o modificarlo bajo los términos de la **GNU Affero General Public License v3.0** publicada por la Free Software Foundation.

Este programa se distribuye con la esperanza de que sea útil, pero **SIN NINGUNA GARANTÍA**; sin siquiera la garantía implícita de COMERCIABILIDAD o IDONEIDAD PARA UN PROPÓSITO PARTICULAR. Ver [LICENSE](LICENSE) para más detalles.

Como esta licencia es AGPL (no GPL), si modificas el código y lo despliegas en un servidor accesible por red, **debes poner a disposición de los usuarios el código fuente completo de tus modificaciones**.

## 🙏 Créditos

Ver [credits.md](https://github.com/TomasYTPro3841/dns-server/blob/main/credits.md)

## 📦 Comandos útiles

| Comando | Descripción |
|---------|-------------|
| `sudo systemctl start grupotst-dns` | Iniciar |
| `sudo systemctl stop grupotst-dns` | Detener |
| `sudo systemctl restart grupotst-dns` | Reiniciar |
| `sudo systemctl status grupotst-dns` | Estado |
| `sudo journalctl -u grupotst-dns -f` | Ver logs en vivo |
| `dig @localhost example.com` | Probar DNS |
| `sudo ss -ulnp \| grep :53` | Ver puerto 53 |

## 🗺️ Roadmap

- [ ] DNS-over-HTTPS (DoH) endpoint
- [ ] DNS-over-TLS (DoT) endpoint
- [ ] API pública para integración con otros sistemas
- [ ] Autenticación con 2FA
- [ ] Soporte para múltiples usuarios con permisos
- [ ] Estadísticas avanzadas por dominio
- [ ] Exportación de logs a CSV/JSON

---

Hecho con ❤️ por [TomasYTPro3841](https://github.com/TomasYTPro3841)
