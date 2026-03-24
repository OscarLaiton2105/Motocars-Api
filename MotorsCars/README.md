# MotorsCars - Mercado de Vehículos

Aplicación web full-stack de marketplace de vehículos, desarrollada con FastAPI, PostgreSQL y HTML/CSS/JavaScript.

## Dominio

Sistema de marketplace de vehículos con dos entidades relacionadas:
- **Brands** (marcas de vehículos)
- **Cars** (vehículos en venta, pertenecen a una marca)

## Integrantes y Responsabilidades

| Integrante | Responsabilidad |
|------------|-----------------|
| Sebastian Blanco | Database (Postgress) |
| Oscar Alejandro | Frontend  y Backend (HTML/CSS/JS con Tailwind, Api Rest) |
| Oscar Laiton | Despliegue cloud |

## Stack Tecnológico

- **Backend:** Python 3.12+ con FastAPI
- **Base de datos:** PostgreSQL
- **Frontend:** HTML, CSS (Tailwind CSS), JavaScript
- **ORM:** SQLAlchemy
- **Validación:** Pydantic

## Servicios Cloud

- GCP (App Engine + Cloud SQL + Cloud Storage)

## Estructura del Proyecto

```
MotorsCars/
├── README.md
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
- **http://localhost:8000** - Página principal
- **http://localhost:8000/car-form.html** - Publicar auto
- **http://localhost:8000/brands.html** - Gestionar marcas

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

## Diagrama de Arquitectura

```
┌─────────────────────┐
│   Frontend (HTML)    │
│   Tailwind CSS + JS  │
│   Puerto 8000        │
└──────────┬──────────┘
           │ HTTP (fetch /api/*)
           ▼
┌─────────────────────┐
│   Backend FastAPI    │
│   Python 3.12       │
│   Puerto 8000       │
└──────────┬──────────┘
           │ SQLAlchemy ORM
           ▼
┌─────────────────────┐
│   PostgreSQL         │
│   Base de datos      │
│   Puerto 5432       │
└─────────────────────┘
```

## Despliegue

### Variables de Entorno (producción)
```
DATABASE_URL=postgresql://usuario:password@host:5432/motorscars_db
```

### Comandos de Despliegue
```bash
# Instalar dependencias
pip install -r requirements.txt

# Ejecutar servidor en producción
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

## Problemas Encontrados y Soluciones

| Problema | Solución |
|----------|----------|
| Solo funciona con Python 3.12 | Verificar versión con `py --version` |
| `python` o `pip` no se reconocen | Usar `py` en vez de `python` y `py -m pip` en vez de `pip` |
| Error CORS al abrir HTML como archivo | El frontend se sirve desde FastAPI en http://localhost:8000 |
| Error 500 al subir imagen grande | Se cambió `image_url` de VARCHAR(500) a TEXT para soportar base64 |

## Capturas de Pantalla

<img width="1430" height="906" alt="image" src="https://github.com/user-attachments/assets/299b1b7b-9bd4-480c-8c7a-dd0cc2af7151" />

