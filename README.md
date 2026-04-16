# Codemio Infrastructure

Infraestructura de Docker para la aplicación Codemio, que incluye MySQL, Prometheus, Grafana y MySQL Exporter para monitoreo completo con volúmenes persistentes.

## 📋 Descripción

> **⚠️ IMPORTANTE**: Esta infraestructura funciona de manera idéntica en desarrollo local y en Render usando las mismas imágenes Docker y configuraciones.

---

## 📁 Estructura del Proyecto

```
codemio-infra/
├── mysql/                      # MySQL 8.0
│   ├── Dockerfile             # → Construye imagen (local y Render)
│   └── my.cnf                 # → Config copiada en imagen
├── prometheus/                 # Sistema de monitoreo
│   ├── Dockerfile             # → Construye imagen (local y Render)
│   └── prometheus.yml         # → Config copiada en imagen
├── grafana/                    # Visualización de métricas
│   ├── Dockerfile             # → Construye imagen (local y Render)
│   ├── datasources.yml        # → Config copiada en imagen
│   └── dashboards.yml         # → Config copiada en imagen
│
├── docker-compose.yml          # 🔵 SOLO Local - Orquestación
├── render.yaml                 # 🟢 SOLO Render - Orquestación
├── .env.example                # Template de variables
├── .env                        # 🔵 SOLO Local - Variables (no en Git)
│
└── README.md                   # Este archivo
```

### 📌 Archivos por Entorno

| Archivo | Local | Render | Descripción |
|---------|:-----:|:------:|-------------|
| `docker-compose.yml` | ✅ | ❌ | Orquesta contenedores en tu máquina |
| `render.yaml` | ❌ | ✅ | Orquesta contenedores en Render |
| `.env` | ✅ | ❌ | Variables locales |
| Dashboard Render | ❌ | ✅ | Variables en Render |
| `Dockerfiles` | ✅ | ✅ | Construyen imágenes |
| `*.yml` configs | ✅ | ✅ | Copiados dentro de imágenes |

---

## 🚀 Quick Start

### Desarrollo Local

```bash
# 1. Clonar repositorio
git clone <repository-url>
cd codemio-infra

# 2. Configurar variables de entorno
cp .env.example .env
nano .env  # Cambiar las 3 contraseñas

# 3. Generar contraseñas seguras (ejecutar 3 veces)
openssl rand -base64 32

# 4. Iniciar servicios
docker-compose up -d

# 5. Verificar
docker-compose ps

# 6. Acceder
# - Grafana:    http://localhost:3000
# - Prometheus: http://localhost:9090
# - MySQL:      localhost:3306
```

### Despliegue en Render

```bash
# 1. Push a GitHub
git push origin main

# 2. Crear Blueprint en Render
# Dashboard → New → Blueprint → Seleccionar repo → Apply

# 3. Configurar variables post-deploy:
# - En codemio-mysql-exporter: DATA_SOURCE_NAME
# - En codemio-grafana: GF_SERVER_ROOT_URL

# 4. Acceder a Grafana
# https://codemio-grafana.onrender.com
```

---

## 📋 Variables de Entorno

### Variables Obligatorias

| Variable | Descripción | Ejemplo |
|----------|-------------|---------|
| `MYSQL_ROOT_PASSWORD` | Contraseña del usuario root de MySQL | `xK9mP2nQ7vL4wE8yT6rA5sD3fG1hJ0zX` |
| `MYSQL_DATABASE` | Nombre de la base de datos | `codemio_db` |
| `MYSQL_USER` | Usuario de la aplicación (no root) | `codemio_user` |
| `MYSQL_PASSWORD` | Contraseña del usuario de la aplicación | `aB8cD9eF2gH5iJ7kL3mN1oP4qR6sT0uV` |
| `GRAFANA_ADMIN_USER` | Usuario administrador de Grafana | `admin` |
| `GRAFANA_ADMIN_PASSWORD` | Contraseña del admin de Grafana | `wX5yZ8aB1cD4eF7gH2iJ9kL3mN6oP0qR` |

### Variables Opcionales (solo si necesitas cambiar puertos locales)

| Variable | Valor por Defecto |
|----------|-------------------|
| `MYSQL_PORT` | `3306` |
| `PROMETHEUS_PORT` | `9090` |
| `GRAFANA_PORT` | `3000` |
| `MYSQL_EXPORTER_PORT` | `9104` |
| `GRAFANA_SERVER_ROOT_URL` | `http://localhost:3000` (local) / `https://codemio-grafana.onrender.com` (Render) |

### Generar Contraseñas Seguras

```bash
# Opción 1: OpenSSL (recomendado)
openssl rand -base64 32

# Opción 2: Python
python3 -c "import secrets; print(secrets.token_urlsafe(32))"

# Opción 3: Node.js
node -e "console.log(require('crypto').randomBytes(24).toString('base64'))"
```

### Configuración para Desarrollo Local

Crea un archivo `.env` en la raíz del proyecto:

```env
# MySQL
MYSQL_ROOT_PASSWORD=<generar-contraseña-1>
MYSQL_DATABASE=codemio_db
MYSQL_USER=codemio_user
MYSQL_PASSWORD=<generar-contraseña-2>

# Grafana
GRAFANA_ADMIN_USER=admin
GRAFANA_ADMIN_PASSWORD=<generar-contraseña-3>
GRAFANA_SERVER_ROOT_URL=http://localhost:3000

# Puertos (opcional - solo si hay conflictos)
# MYSQL_PORT=3306
# PROMETHEUS_PORT=9090
# GRAFANA_PORT=3000
```

### Configuración para Render

1. **Variables Generadas Automáticamente:**
   - `MYSQL_ROOT_PASSWORD` (generada por Render)
   - `MYSQL_PASSWORD` (generada por Render)
   - `GRAFANA_ADMIN_PASSWORD` (generada por Render)
   - `MYSQL_DATABASE=codemio_db` (configurada en render.yaml)
   - `MYSQL_USER=codemio_user` (configurada en render.yaml)
   - `GRAFANA_ADMIN_USER=admin` (configurada en render.yaml)

2. **Variables a Configurar Manualmente:**

   **En `codemio-mysql-exporter`:**
   ```
   Key: DATA_SOURCE_NAME
   Value: codemio_user:[MYSQL_PASSWORD]@(codemio-mysql:3306)/codemio_db
   ```
   (Copia `MYSQL_PASSWORD` del servicio `codemio-mysql`)

   **En `codemio-grafana`:**
   ```
   Key: GF_SERVER_ROOT_URL
   Value: https://codemio-grafana.onrender.com
   ```
   (Usa tu URL real asignada por Render)

---

## 📦 Servicios Incluidos

| Servicio | Puerto | Descripción | Volumen Persistente |
|----------|--------|-------------|---------------------|
| **MySQL** | 3306 | Base de datos MySQL 8.0 | ✅ mysql_data |
| **MySQL Exporter** | 9104 | Exporta métricas de MySQL | ❌ |
| **Prometheus** | 9090 | Sistema de monitoreo | ✅ prometheus_data (opcional) |
| **Grafana** | 3000 | Visualización de métricas | ✅ grafana_data |

### Networking

Todos los servicios se comunican usando nombres consistentes:
- `codemio-mysql:3306`
- `codemio-mysql-exporter:9104`
- `codemio-prometheus:9090`
- `codemio-grafana:3000`

**Estos nombres son idénticos en local (Docker Compose) y en Render.**

---

## 🛠️ Requisitos

### Para Desarrollo Local:
- Docker Engine 20.10+
- Docker Compose 2.0+
- Puertos disponibles: 3306, 3000, 9090, 9104

### Para Render:
- Cuenta de Render
- Repositorio en GitHub
- Plan Starter ($7/mes por servicio) para discos persistentes

---

## 🔧 Configuración

### Archivos de Configuración

Estos archivos se copian **dentro** de las imágenes Docker durante el build:

- `prometheus/prometheus.yml` - Define qué servicios monitorear
- `grafana/datasources.yml` - Configura Prometheus como datasource
- `grafana/dashboards.yml` - Configura provisioning de dashboards
- `mysql/my.cnf` - Configuración personalizada de MySQL

**Importante:** Cambios en estos archivos requieren rebuild:

```bash
# Local
docker-compose up -d --build

# Render
# Push a GitHub → Render rebuild automáticamente
```

---

## 📊 Monitoreo y Dashboards

### Importar Dashboard de MySQL en Grafana

1. Accede a Grafana (`http://localhost:3000` o tu URL de Render)
2. Ve a **Dashboards** → **Import**
3. Ingresa el ID: **7362** (MySQL Overview)
4. Selecciona **Prometheus** como datasource
5. Click en **Import**

### Dashboards Recomendados:
- **7362**: MySQL Overview ⭐
- **6239**: MySQL InnoDB Metrics
- **11323**: MySQL Performance Schema

### Métricas Clave:
- `mysql_up` - Estado del servidor MySQL
- `mysql_global_status_threads_connected` - Conexiones activas
- `mysql_global_status_queries` - Consultas por segundo

---

## 🔍 Comandos Útiles

### Desarrollo Local

```bash
# Iniciar servicios
docker-compose up -d

# Ver logs
docker-compose logs -f                    # Todos
docker-compose logs -f codemio-mysql      # Servicio específico

# Detener servicios
docker-compose down

# Detener y eliminar volúmenes (⚠️ PIERDE DATOS)
docker-compose down -v

# Rebuild después de cambios
docker-compose up -d --build

# Ver estado
docker-compose ps

# Ejecutar comando en contenedor
docker exec -it codemio-mysql mysql -u root -p
```

### Backups

```bash
# Backup de MySQL
docker exec codemio-mysql mysqldump -u root -p${MYSQL_ROOT_PASSWORD} ${MYSQL_DATABASE} > backup.sql

# Restaurar MySQL
docker exec -i codemio-mysql mysql -u root -p${MYSQL_ROOT_PASSWORD} ${MYSQL_DATABASE} < backup.sql
```

---

## 🔐 Seguridad

### Checklist de Seguridad

- [ ] Cambiar TODAS las contraseñas de `.env.example`
- [ ] Usar contraseñas diferentes para root, usuario y Grafana
- [ ] Generar contraseñas fuertes (mínimo 16 caracteres)
- [ ] NO subir `.env` a Git (ya está en `.gitignore`)
- [ ] Guardar contraseñas en un gestor seguro
- [ ] En Render: Verificar que variables están configuradas
- [ ] Habilitar 2FA en tu cuenta de Render

### Generar Contraseñas Seguras

```bash
# OpenSSL (recomendado)
openssl rand -base64 32

# Python
python3 -c "import secrets; print(secrets.token_urlsafe(32))"

# Node.js
node -e "console.log(require('crypto').randomBytes(24).toString('base64'))"
```

---

## 🐛 Troubleshooting

### Local: Puerto en uso

```bash
# Ver qué usa el puerto
sudo lsof -i :3306  # o 3000, 9090, 9104

# Cambiar puerto en .env
echo "MYSQL_PORT=3307" >> .env
docker-compose up -d
```

### Local: No puede conectarse a MySQL

```bash
# Verificar que está corriendo
docker-compose ps

# Ver logs
docker-compose logs codemio-mysql

# Verificar credenciales
cat .env | grep MYSQL_
```

### Render: Grafana no conecta a Prometheus

1. Ve a Grafana → **Configuration** → **Data Sources**
2. Click en **Prometheus**
3. Verifica URL: `http://codemio-prometheus:9090`
4. Click **Test** (debería ser verde ✅)

### Render: MySQL Exporter no conecta

1. Ve a `codemio-mysql-exporter` → **Environment**
2. Verifica `DATA_SOURCE_NAME`:
   ```
   codemio_user:[PASSWORD]@(codemio-mysql:3306)/codemio_db
   ```
3. Copia el `MYSQL_PASSWORD` correcto de `codemio-mysql`

---

## 📚 Conexión desde tu Aplicación

### Variables de Entorno para tu App

```env
DB_HOST=codemio-mysql               # Local
DB_HOST=codemio-mysql.onrender.com  # Render
DB_PORT=3306
DB_NAME=codemio_db
DB_USER=codemio_user
DB_PASSWORD=<mismo-que-MYSQL_PASSWORD>
```

### Ejemplo Node.js

```javascript
const mysql = require('mysql2/promise');

const pool = mysql.createPool({
  host: process.env.DB_HOST,
  port: process.env.DB_PORT,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  waitForConnections: true,
  connectionLimit: 10
});
```

### Ejemplo Python

```python
import pymysql
import os

connection = pymysql.connect(
    host=os.getenv('DB_HOST'),
    port=int(os.getenv('DB_PORT', 3306)),
    user=os.getenv('DB_USER'),
    password=os.getenv('DB_PASSWORD'),
    database=os.getenv('DB_NAME')
)
```

---

## ✅ Checklist Post-Instalación

### Desarrollo Local
- [ ] `.env` creado con contraseñas fuertes
- [ ] `docker-compose up -d` exitoso
- [ ] Todos los servicios "Up"
- [ ] Grafana accesible (`localhost:3000`)
- [ ] Prometheus accesible (`localhost:9090`)
- [ ] Prometheus muestra target "mysql" como "up"
- [ ] Dashboard de MySQL importado en Grafana

### Render
- [ ] Blueprint aplicado exitosamente
- [ ] Todos los servicios "Live"
- [ ] `DATA_SOURCE_NAME` configurado en mysql-exporter
- [ ] `GF_SERVER_ROOT_URL` configurado en grafana
- [ ] Grafana accesible (tu URL de Render)
- [ ] Prometheus muestra target "mysql" como "up"
- [ ] Dashboard de MySQL importado
- [ ] Contraseñas guardadas en gestor seguro

---

## 🤝 Contribuir

1. Fork el repositorio
2. Crea una rama: `git checkout -b feature/nueva-funcionalidad`
3. Commit cambios: `git commit -m 'feat: Agregar funcionalidad'`
4. Push: `git push origin feature/nueva-funcionalidad`
5. Abre un Pull Request

---

## 👥 Equipo

Desarrollado por el equipo de CodeMio - UTEZ

### Integrantes del Proyecto

<table>
  <tr>
    <th>Nombre</th>
    <th>Rol(es)</th>
    <th>GitHub</th>
  </tr>
  <tr>
    <td><strong>Miguel Angel Sanchez Perez</strong></td>
    <td>
      🔐 Administrador de Bases de Datos y Seguridad<br/>
      🧪 Lead de Integración y QA
    </td>
    <td>
      <a href="https://github.com/MiguelSP040">
        <img src="https://img.shields.io/badge/GitHub-MiguelSP040-181717?style=flat-square&logo=github" alt="MiguelSP040"/>
      </a>
    </td>
  </tr>
  <tr>
    <td><strong>Martin Antonio Joaquín Landa</strong></td>
    <td>⚙️ Desarrollador Backend</td>
    <td>
      <a href="https://github.com/M4ltin12">
        <img src="https://img.shields.io/badge/GitHub-M4ltin12-181717?style=flat-square&logo=github" alt="M4ltin12"/>
      </a>
    </td>
  </tr>
  <tr>
    <td><strong>Daniela Carrate Bahena</strong></td>
    <td>🎨 Frontend & UX</td>
    <td>
      <a href="https://github.com/danielita05">
        <img src="https://img.shields.io/badge/GitHub-danielita05-181717?style=flat-square&logo=github" alt="danielita05"/>
      </a>
    </td>
  </tr>
  <tr>
    <td><strong>Luis David Rojas Vargas</strong></td>
    <td>💻 Desarrollador JS</td>
    <td>
      <a href="https://github.com/DavidReds7">
        <img src="https://img.shields.io/badge/GitHub-DavidReds7-181717?style=flat-square&logo=github" alt="DavidReds7"/>
      </a>
    </td>
  </tr>
  <tr>
    <td><strong>Luis Ignacio Valera Aguilar</strong></td>
    <td>📊 Business Intelligence</td>
    <td>
      <a href="https://github.com/IgnacioValera">
        <img src="https://img.shields.io/badge/GitHub-IgnacioValera-181717?style=flat-square&logo=github" alt="IgnacioValera"/>
      </a>
    </td>
  </tr>
</table>

---

## 📞 Soporte

Para reportar problemas o solicitar características, abre un issue en el [repositorio de GitHub](https://github.com/MiguelSP040/codemio_infra/issues).

---

## 📄 Licencia

Este proyecto es parte del trabajo académico en UTEZ y está desarrollado con fines educativos.
