# MotorsCars - Mercado de Vehículos

Aplicación web full-stack de marketplace de vehículos, desarrollada con FastAPI, PostgreSQL y HTML/CSS/JavaScript, desplegada en Google Cloud Run.

---

## Dominio

Sistema de marketplace de vehículos con dos entidades relacionadas:
- **Brands** (marcas de vehículos)
- **Cars** (vehículos en venta, pertenecen a una marca)

## Integrantes y Responsabilidades

| Integrante | Responsabilidad |
|------------|-----------------|
| Sebastian Blanco | Database (PostgreSQL) |
| Oscar Alejandro | Frontend y Backend (HTML/CSS/JS con Tailwind, API REST) |
| Oscar Laiton | Despliegue cloud |

## Stack Tecnológico

- **Backend:** Python 3.12+ con FastAPI
- **Base de datos:** PostgreSQL
- **Frontend:** HTML, CSS (Tailwind CSS), JavaScript
- **ORM:** SQLAlchemy
- **Validación:** Pydantic
- **Cloud:** Google Cloud Platform (Cloud Run + Cloud SQL + Artifact Registry)

---

## Estructura del Proyecto

```
MotorsCars/
├── README.md
├── Dockerfile
├── .gitignore
├── .dockerignore
├── backend/
│   ├── app/
│   │   ├── main.py          # App FastAPI + CORS + Servidor frontend
│   │   ├── database.py      # Conexión a PostgreSQL
│   │   ├── models.py        # Modelos SQLAlchemy
│   │   ├── schemas.py       # Esquemas Pydantic
│   │   └── routers/
│   │       ├── brands.py    # Endpoints de marcas
│   │       └── cars.py      # Endpoints de autos
│   ├── requirements.txt
│   └── .env.example
├── frontend/
│   ├── index.html           # Listado de autos
│   ├── car-detail.html      # Detalle de un auto
│   ├── car-form.html        # Crear/editar auto (con subida de fotos)
│   ├── brands.html          # Gestión de marcas
│   ├── app.js               # Cliente API
│   └── style.css
├── database/
│   ├── schema.sql           # Creación de tablas
│   └── seed.sql             # Datos de prueba
├── docs/
│   └── api-documentation.md
└── video/
```

## Diagrama de Arquitectura

```
┌─────────────────────┐
│   Frontend (HTML)    │
│   Tailwind CSS + JS  │
└──────────┬──────────┘
           │ HTTP (fetch /api/*)
           ▼
┌─────────────────────┐
│   Backend FastAPI    │
│   Python 3.12       │
└──────────┬──────────┘
           │ SQLAlchemy ORM
           ▼
┌─────────────────────┐
│   PostgreSQL         │
│   (Cloud SQL)        │
└─────────────────────┘
```

---

## Endpoints de la API

| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | /api/brands | Listar todas las marcas |
| POST | /api/brands | Crear una marca |
| DELETE | /api/brands/{id} | Eliminar marca (y sus autos) |
| GET | /api/brands/{id}/cars | Autos de una marca |
| GET | /api/cars | Listar todos los autos |
| GET | /api/cars/{id} | Detalle de un auto |
| POST | /api/cars | Crear auto |
| PUT | /api/cars/{id} | Actualizar auto |
| DELETE | /api/cars/{id} | Eliminar auto |

---

## Instalación Local

### Requisitos

- Python 3.12+
- PostgreSQL 14+
- Navegador web moderno
- Cliente de base de datos (DBeaver, pgAdmin o similar)

### Base de datos

1. Crear la base de datos en DBeaver o pgAdmin:
```sql
CREATE DATABASE motorscars_db;
```

2. Ejecutar los scripts SQL en orden:
   - Primero `database/schema.sql` (crea las tablas)
   - Luego `database/seed.sql` (inserta datos de prueba)

O por línea de comandos:
```bash
psql -U postgres -d motorscars_db -f database/schema.sql
psql -U postgres -d motorscars_db -f database/seed.sql
```

### Backend

1. Ir a la carpeta backend:
```bash
cd backend
```

2. Instalar dependencias:
```bash
pip install -r requirements.txt
```
> Si `pip` no se reconoce, usar: `py -m pip install -r requirements.txt`

3. Configurar variables de entorno:
```bash
copy .env.example .env
```
Editar `.env` con las credenciales de tu PostgreSQL:
```
DATABASE_URL=postgresql://postgres:TU_PASSWORD@localhost:5432/motorscars_db
```

4. Ejecutar el servidor:
```bash
uvicorn app.main:app --reload
```
> Si `uvicorn` no se reconoce, usar: `py -m uvicorn app.main:app --reload`

5. Verificar la API en: http://localhost:8000/docs

### Frontend

El frontend se sirve automáticamente desde FastAPI. Acceder a:
- **http://localhost:8000** — Página principal
- **http://localhost:8000/car-form.html** — Publicar auto
- **http://localhost:8000/brands.html** — Gestionar marcas

---

## Despliegue en Google Cloud Run

### Parte 1 — Archivos de configuración

Antes de trabajar con Google Cloud, se necesitan tres archivos en la raíz del proyecto.

#### .gitignore

Le indica a Git qué archivos **no** debe subir al repositorio. Lo más importante es excluir el `.env` que contiene las credenciales de la base de datos.

```
.env
__pycache__/
*.pyc
venv/
.venv/
*.log
video/
```

#### .dockerignore

Funciona igual que el `.gitignore` pero para Docker. Le dice qué archivos no debe copiar dentro del contenedor al construir la imagen.

```
.env
__pycache__/
*.pyc
.git/
venv/
*.log
video/
docs/
database/
README.md
```

#### Dockerfile

Le indica a Docker cómo construir la imagen de la aplicación.

```dockerfile
FROM python:3.12-slim

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app

COPY backend/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY backend/ ./backend/
COPY frontend/ ./frontend/

ENV PORT=8080
EXPOSE 8080

CMD uvicorn app.main:app --host 0.0.0.0 --port ${PORT} --workers 1 --app-dir backend
```

Con los tres archivos listos, se hace push a GitHub:

```bash
cd MotorsCars
git init
git add .
git commit -m "feat: initial commit"
git remote add origin https://github.com/OscarLaiton2105/Motocars-Api.git
git branch -M main
git push -u origin main
```

### Parte 2 — Crear la instancia de Cloud SQL

**Cloud SQL** es el servicio de PostgreSQL gestionado de GCP. Google se encarga del mantenimiento, los backups y la disponibilidad.

```bash
# Crear la instancia PostgreSQL (tarda entre 5 y 10 minutos)
gcloud sql instances create motorscars-db \
  --database-version=POSTGRES_15 \
  --tier=db-f1-micro \
  --region=us-central1

# Crear la base de datos
gcloud sql databases create motorscars_db --instance=motorscars-db

# Crear el usuario
gcloud sql users create motorscars_user \
  --instance=motorscars-db \
  --password=Admin1234@

# Verificar que está activa
gcloud sql instances describe motorscars-db --format="value(state)"
# Respuesta esperada: RUNNABLE
```

### Parte 3 — Definir variables y habilitar APIs

```bash
PROJECT_ID=$(gcloud config get-value project)
CONNECTION_NAME=$(gcloud sql instances describe motorscars-db --format='value(connectionName)')

echo $PROJECT_ID
echo $CONNECTION_NAME
```

| Variable | Descripción |
|----------|-------------|
| `PROJECT_ID` | Identificador del proyecto en GCP |
| `CONNECTION_NAME` | Nombre único de la instancia Cloud SQL con formato `proyecto:región:instancia` |

Habilitar las APIs necesarias:

```bash
gcloud services enable \
  run.googleapis.com \
  sqladmin.googleapis.com \
  cloudbuild.googleapis.com \
  artifactregistry.googleapis.com
```

### Parte 4 — Artifact Registry y permisos

**Artifact Registry** es el servicio de Google para almacenar imágenes Docker.

```bash
gcloud artifacts repositories create motorscars \
    --repository-format=docker \
    --location=us-central1

gcloud auth configure-docker us-central1-docker.pkg.dev
```

Dar permisos a Cloud Run para conectarse a Cloud SQL:

```bash
PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')

gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:${PROJECT_NUMBER}-compute@developer.gserviceaccount.com" \
    --role="roles/cloudsql.client"
```

### Parte 5 — Construir la imagen Docker

Clonar el repositorio en Cloud Shell y construir la imagen:

```bash
cd ~
git clone https://github.com/OscarLaiton2105/Motocars-Api.git
cd Motocars-Api/MotorsCars

gcloud builds submit --tag us-central1-docker.pkg.dev/$PROJECT_ID/motorscars/motorscars
```

Cloud Build ejecuta el Dockerfile paso a paso: descarga la imagen base, instala las dependencias y copia el código. Al finalizar, la imagen queda publicada y lista para desplegar.

### Parte 6 — Desplegar en Cloud Run

```bash
gcloud run deploy motorscars \
    --image us-central1-docker.pkg.dev/$PROJECT_ID/motorscars/motorscars:latest \
    --region us-central1 \
    --allow-unauthenticated \
    --memory 512Mi \
    --add-cloudsql-instances=$CONNECTION_NAME \
    --set-env-vars="DATABASE_URL=postgresql://motorscars_user:Admin1234%40@localhost/motorscars_db?host=/cloudsql/$CONNECTION_NAME"
```

### Parte 7 — Cargar las tablas y datos

Conectarse a Cloud SQL y ejecutar los scripts de base de datos:

```bash
gcloud sql connect motorscars-db --user=motorscars_user --database=motorscars_db
```

Dentro del prompt de PostgreSQL se ejecutan:

- `schema.sql` → crea las tablas `brands`, `cars` y `car_images` con sus índices
- `seed.sql` → inserta 8 marcas y 12 vehículos de prueba

```sql
\q
```

---

## Resultado del Despliegue

- **App en producción:** https://motorscars-584814093483.us-central1.run.app
- **API Docs:** https://motorscars-584814093483.us-central1.run.app/docs

---

## Problemas Encontrados y Soluciones

| Problema | Solución |
|----------|----------|
| Solo funciona con Python 3.12 | Verificar versión con `py --version` |
| `python` o `pip` no se reconocen | Usar `py` en vez de `python` y `py -m pip` en vez de `pip` |
| Error CORS al abrir HTML como archivo | El frontend se sirve desde FastAPI en http://localhost:8000 |
| Error 500 al subir imagen grande | Se cambió `image_url` de VARCHAR(500) a TEXT para soportar base64 |
