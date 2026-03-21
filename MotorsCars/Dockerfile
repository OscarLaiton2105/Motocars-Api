# ---- Etapa de build ----
FROM python:3.12-slim
 
# Evita que Python escriba archivos .pyc y activa logs en tiempo real
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
 
# Directorio de trabajo dentro del contenedor
WORKDIR /app
 
# Copiar e instalar dependencias primero (mejor uso de caché de Docker)
COPY backend/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
 
# Copiar el código fuente completo
COPY backend/ ./backend/
COPY frontend/ ./frontend/
 
# Cloud Run inyecta el puerto via la variable PORT (por defecto 8080)
ENV PORT=8080
 
# Exponer el puerto
EXPOSE 8080
 
# Comando de inicio para producción
# --host 0.0.0.0 es obligatorio en contenedores
# --workers 1 es recomendado en Cloud Run (escala por instancias, no por workers)
CMD uvicorn backend.app.main:app --host 0.0.0.0 --port ${PORT} --workers 1