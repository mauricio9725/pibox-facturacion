# Despliegue en Producción — Pibox Portal de Prefacturación

## Opción A: Railway (recomendado)

### 1. Preparar repositorio en GitHub

1. Crea un repo en GitHub (puede ser privado).
2. Asegúrate de que `.env` y `users.json` estén en `.gitignore` (ya están).
3. Sube el código:
   ```bash
   cd pibox_facturacion
   git init
   git add .
   git commit -m "chore: inicial"
   git remote add origin https://github.com/TU_USUARIO/pibox-facturacion.git
   git push -u origin main
   ```

### 2. Crear servicio en Railway

1. Ve a [railway.app](https://railway.app) → **New Project → Deploy from GitHub repo**.
2. Selecciona el repositorio.
3. Railway detectará el `railway.toml` automáticamente.

### 3. Variables de entorno en Railway

Ve a tu servicio → **Variables** y agrega:

| Variable         | Valor                        |
|------------------|------------------------------|
| `CH_HOST`        | `clickhouse.picap.io`        |
| `CH_PORT`        | `8443`                       |
| `CH_USER`        | `mbustos`                    |
| `CH_PASSWORD`    | `ieCvq9KRBReGEM0qdgIvlQ`     |
| `CH_DATABASE`    | `picapmongoprod`             |
| `ADMIN_USER`     | `admin` (o el que prefieras) |
| `ADMIN_PASSWORD` | contraseña segura para admin |
| `USERS_DIR`      | `/data` (ver nota abajo)     |

> **Nota `USERS_DIR`:** Railway no tiene filesystem persistente por defecto.
> Para persistir `users.json` agrega un **Volume** en Railway montado en `/data`
> y establece `USERS_DIR=/data`. Sin volume, los usuarios se resetean en cada deploy.

### 4. Deploy

Railway hace deploy automático en cada push a `main`. El healthcheck apunta a
`/_stcore/health` (configurado en `railway.toml`).

---

## Opción B: Render.com

Usa el `render.yaml` ya incluido. En el dashboard de Render:

1. **New → Web Service → Connect a repository**.
2. Render detecta el `render.yaml`.
3. Agrega las mismas variables de entorno en **Environment**.

---

## Variables de entorno — resumen

| Variable         | Requerida | Descripción                                         |
|------------------|-----------|-----------------------------------------------------|
| `CH_HOST`        | Sí        | Host de ClickHouse                                  |
| `CH_PORT`        | Sí        | Puerto (8443 para HTTPS)                            |
| `CH_USER`        | Sí        | Usuario ClickHouse                                  |
| `CH_PASSWORD`    | Sí        | Contraseña ClickHouse                               |
| `CH_DATABASE`    | Sí        | Base de datos                                       |
| `ADMIN_USER`     | No        | Nombre de usuario del admin inicial (default: admin)|
| `ADMIN_PASSWORD` | No        | Contraseña del admin inicial (se fuerza cambio)     |
| `USERS_DIR`      | No        | Directorio para `users.json` (default: carpeta app) |

---

## Logs

Los logs se emiten con el logger `pibox` en formato:
```
2024-01-15 10:23:45,123 [INFO] pibox — Login exitoso: admin (admin)
2024-01-15 10:23:46,456 [WARNING] pibox — Login fallido: usuario_inexistente
```

En Railway: **Deployments → Logs** para ver en tiempo real.
En Render: **Logs** en el panel del servicio.

---

## Primer acceso en producción

1. El admin inicial se crea automáticamente con `ADMIN_USER` / `ADMIN_PASSWORD`.
2. Al primer login, el sistema fuerza un cambio de contraseña.
3. Desde **Gestión de Usuarios** puedes crear más usuarios con los roles:
   - `admin` — acceso total
   - `operaciones` — prefacturas + data
   - `financiero` — prefacturas + data (sin gestión de usuarios)
   - `cliente` — solo Prefactura Cliente
