# estadisticas-service

Microservicio de **estadísticas / dashboards** del casino (FastAPI, **solo lectura**).
Comparte la base de datos PostgreSQL y el `JWT_SECRET` con `casino-backend`.
Agrega KPIs sobre `transacciones`, `usuarios` y `apuestas`.

- Prefijo de rutas: `/api/estadisticas` · Docs: `/docs`
- Puerto: **8006**

## Endpoints

| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/api/estadisticas/mias` | KPIs, desglose por tipo y línea de saldo del usuario |
| GET | `/api/estadisticas/globales` | Usuarios, GGR, top jugadores, métricas de apuestas |
| GET | `/livez` | Liveness probe (Kubernetes) |
| GET | `/readyz` | Readiness probe — verifica BD (200/503) |

## Variables de entorno

| Variable | Valor por defecto | Descripción |
|----------|------------------|-------------|
| `DB_HOST` | `localhost` | Host de PostgreSQL |
| `DB_PORT` | `5432` | Puerto de PostgreSQL |
| `DB_USER` | `casino` | Usuario de PostgreSQL |
| `DB_PASSWORD` | `casino` | Contraseña de PostgreSQL |
| `DB_NAME` | `casino_db` | Nombre de la BD |
| `JWT_SECRET` | `cambiame` | Clave para validar JWT del backend |

## Ejecutar en local

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
uvicorn app.main:app --reload --port 8006
```

## Construir imagen Docker

```bash
docker build -t estadisticas-service:latest .
docker run --rm estadisticas-service:latest whoami  # debe mostrar: appuser
```

## Desplegar en EKS

```bash
aws eks update-kubeconfig --region us-east-1 --name vidal-casino
kubectl apply -f k8s/
kubectl get pods -l app=estadisticas-service
```

## CI/CD

El pipeline se dispara con push a la rama `deploy`:

```
dev → merge a deploy → GitHub Actions → ECR → EKS
```

## Secrets requeridos en GitHub

`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`, `AWS_REGION`, `AWS_ACCOUNT_ID`, `EKS_CLUSTER`

## Troubleshooting

| Síntoma | Causa | Solución |
|---------|-------|----------|
| `ImagePullBackOff` | Credenciales AWS expiradas | Actualizar secrets en GitHub |
| `/readyz` responde 503 | BD no disponible | Verificar pod de postgres |
| `CrashLoopBackOff` | Error en variables de entorno | `kubectl logs deployment/estadisticas-service` |
