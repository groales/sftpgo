# SFTPGo

Servidor de transferencia de archivos seguro con soporte para SFTP, FTP/S y WebDAV. Incluye panel de administración web y portal self-service para usuarios.

> 📖 **Documentación oficial Docker**: [https://docs.sftpgo.com/latest/docker/](https://docs.sftpgo.com/latest/docker/)

## Caracteristicas

- SFTP, FTP/S y WebDAV en un único contenedor
- Panel de administración web (puerto 8080 publicado)
- Portal web para usuarios finales (webclient)
- Autenticación 2FA
- Carpetas virtuales y cuotas por usuario
- Backends de almacenamiento: disco local, S3, GCS, Azure Blob
- Sin base de datos externa requerida (SQLite por defecto)

## Puertos

| Puerto | Protocolo | Descripcion |
|--------|-----------|-------------|
| 2022   | SFTP      | Transferencia de archivos (publicado directamente) |
| 8080   | HTTP      | Panel web de administracion (publicado directamente) |

## Requisitos Previos

- Docker Engine
- Docker Compose plugin
- Puerto 2022 abierto en el firewall del host (SFTP)
- Red Docker externa `proxy` (si usas proxy inverso para el panel web)
- Dominio con DNS apuntando al servidor (para acceso web por HTTPS)

## Archivos del Repositorio

- `compose.yaml` — definicion del servicio SFTPGo
- `README.md` — documentacion de despliegue y operacion

Los directorios `data/` y `config/` se crean automáticamente al arrancar y quedan ignorados por git.

## compose.yaml actual

```yaml
services:
  sftpgo:
    container_name: sftpgo
    image: drakkan/sftpgo:latest
    user: "1000:1000"
    restart: unless-stopped
    ports:
      - "2022:2022" # SFTP
      - "8080:8080" # Panel de administracion
    volumes:
      - ./data:/srv/sftpgo
      - ./config:/var/lib/sftpgo

# Red externa para proxy inverso
networks:
  default:
    external: true
    name: proxy
```

**Volumenes**:
- `./data` → `/srv/sftpgo`: datos persistentes (home de usuarios, backups)
- `./config` → `/var/lib/sftpgo`: configuracion, host keys SSH

## Despliegue rapido

1. Clona el repositorio:

```bash
git clone https://github.com/groales/sftpgo.git
cd sftpgo
```

2. Crea la red proxy si todavia no existe:

```bash
docker network create proxy
```

3. Crea los directorios de datos y ajusta permisos (este despliegue usa UID/GID 1000):

```bash
mkdir -p data config
chown -R 1000:1000 data config
```

4. Inicia el servicio:

```bash
docker compose up -d
```

5. Verifica el arranque:

```bash
docker compose logs -f sftpgo
```

## Primer acceso al panel de administracion

Accede al panel web:

1. Abre `http://IP_SERVIDOR:8080/web/admin`.
2. Crea el primer usuario administrador.
3. Opcional: configura proxy inverso y usa `https://sftpgo.tudominio.com/web/admin`.

Con proxy inverso configurado accede directamente a `https://sftpgo.tudominio.com/web/admin`.

## Configuracion del proxy inverso (Nginx Proxy Manager)

Para el panel de administracion (`/web/admin`) y el webclient (`/web/client`):

- Forward Hostname/IP: `sftpgo`
- Forward Port: `8080`
- Websockets Support: habilitado
- SSL: certificado valido y Force SSL habilitado

> El puerto SFTP (2022) va directo al host, no pasa por el proxy inverso.

## Crear usuarios SFTP

Desde el panel de administracion:

1. **Users** → **Add user**
2. Configura:
   - **Username**: nombre del usuario
   - **Password** o **Public keys** (clave SSH)
   - **Home directory**: se crea automaticamente en `/srv/sftpgo/data/<usuario>`
   - **Permissions**: lectura, escritura, listado, etc.
3. **Save**

### Conectar por SFTP

```bash
sftp -P 2022 usuario@IP_SERVIDOR
```

O con cliente grafico como FileZilla:
- Host: `IP_SERVIDOR` / Protocolo: SFTP
- Puerto: `2022`
- Usuario y contraseña configurados en el panel

## Backup y restauracion

Los datos persisten en `./data` y la configuracion en `./config`.

Backup:

```bash
mkdir -p backup
tar -czf backup/sftpgo-$(date +%Y%m%d-%H%M%S).tar.gz data config
```

Restauracion:

```bash
docker compose down
rm -rf data config
tar -xzf backup/sftpgo-FECHA.tar.gz
chown -R 1000:1000 data config
docker compose up -d
```

## Actualizacion

```bash
docker compose pull sftpgo
docker compose up -d sftpgo
docker compose logs -f sftpgo
```

## Solucion de Problemas

Permisos denegados en arranque:

```bash
chown -R 1000:1000 data config
```

No conecta por SFTP:
- Verifica que el puerto 2022 está abierto en el firewall del host
- Verifica que el usuario existe y tiene contraseña o clave SSH configurada

Panel web no carga:
- Verifica que el proxy enruta a `sftpgo:8080`
- Verifica que Websockets está habilitado en el proxy

Ver logs detallados:

```bash
docker compose logs --tail 200 sftpgo
```

## Variables de entorno

Este compose no define variables de entorno personalizadas. SFTPGo arranca con los valores por defecto de la imagen.

La configuracion avanzada se realiza desde el panel web o montando un archivo de configuracion en `./config`.

Consulta la referencia completa de variables en: [https://docs.sftpgo.com/latest/env-vars/](https://docs.sftpgo.com/latest/env-vars/)

## Recursos

- Documentacion Docker oficial: [https://docs.sftpgo.com/latest/docker/](https://docs.sftpgo.com/latest/docker/)
- Documentacion general: [https://docs.sftpgo.com/latest/](https://docs.sftpgo.com/latest/)
- Docker Hub: [https://hub.docker.com/r/drakkan/sftpgo](https://hub.docker.com/r/drakkan/sftpgo)
- Repositorio del proyecto: [https://github.com/drakkan/sftpgo](https://github.com/drakkan/sftpgo)

## Licencia

Este repositorio de configuracion: MIT
SFTPGo: AGPL-3.0
