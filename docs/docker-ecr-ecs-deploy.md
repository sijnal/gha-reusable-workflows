# Docker ECR ECS Deploy - GitHub Workflow

Este documento describe un workflow reutilizable de GitHub Actions para construir imágenes Docker, subirlas a Amazon ECR y opcionalmente desplegarlas en Amazon ECS. Este pipeline está diseñado para trabajar con aplicaciones contenerizadas y facilita el proceso completo de CI/CD desde build hasta deploy.

---

## 🛠 Funcionalidades del Workflow

Este workflow realiza lo siguiente:

1. Clona el repositorio.
2. Configura credenciales de AWS asumiendo un rol vía OIDC.
3. Configura Docker Buildx para optimizaciones avanzadas.
4. Autentica con Amazon ECR.
5. Construye la imagen Docker con caché optimizado.
6. Sube la imagen a ECR con tag único basado en run_number.
7. Crea tags de Git (opcional).
8. Renderiza la task definition de ECS con la nueva imagen.
9. Despliega a ECS si está habilitado (`deploy-to-ecs: true`).

---

## 📥 Entradas del Workflow

El workflow acepta las siguientes entradas mediante `workflow_call`:

| Nombre                  | Requerido | Tipo    | Valor por defecto     | Descripción                                                                 |
|-------------------------|-----------|---------|----------------------|-----------------------------------------------------------------------------|
| `account-id`            | ✅        | string  | —                    | ID de la cuenta de AWS donde se encuentra el rol.                          |
| `aws-role-name`         | ✅        | string  | —                    | Nombre del rol de AWS a asumir (sin el prefijo ARN).                       |
| `aws-region`            | ❌        | string  | `us-east-1`          | Región de AWS donde se ejecutarán los comandos.                            |
| `environment`           | ✅        | string  | —                    | Entorno de despliegue (dev, staging, prod).                                |
| `ecr-repository`        | ✅        | string  | —                    | Nombre del repositorio ECR donde subir la imagen.                          |
| `docker-context`        | ❌        | string  | `.`                  | Contexto para el build de Docker.                                          |
| `dockerfile-path`       | ❌        | string  | `Dockerfile`         | Ruta al Dockerfile.                                                         |
| `image-tag-prefix`      | ❌        | string  | `v`                  | Prefijo para el tag de la imagen.                                          |
| `task-definition-path`  | ❌        | string  | `task-definition.json` | Ruta al archivo de task definition de ECS.                               |
| `ecs-service-name`      | ✅        | string  | —                    | Nombre del servicio ECS.                                                   |
| `ecs-cluster-name`      | ✅        | string  | —                    | Nombre del cluster ECS.                                                    |
| `container-name`        | ✅        | string  | —                    | Nombre del contenedor en la task definition.                               |
| `deploy-to-ecs`         | ❌        | boolean | `false`              | **Determina si se debe hacer deploy a ECS**.                               |
| `create-git-tag`        | ❌        | boolean | `true`               | Crear tag de Git con la versión construida.                                |

---

## 📤 Outputs del Workflow

| Output      | Descripción                                  |
|-------------|----------------------------------------------|
| `image-uri` | URI completa de la imagen construida        |
| `image-tag` | Tag de la imagen construida                  |

---

## 🔐 Secretos Requeridos

Este workflow usa autenticación con AWS vía OIDC y utiliza `secrets: inherit` para heredar todos los secretos del workflow padre. No se requiere configuración adicional de secretos.

---

## 🧪 Jobs del Workflow

### Job: `build-and-push`

Este job construye y sube la imagen Docker:

- `checkout`: Clona el repositorio.
- `setup-buildx`: Configura Docker Buildx para funciones avanzadas.
- `configure-aws-credentials`: Autenticación con AWS usando el rol indicado.
- `login-ecr`: Autentica con Amazon ECR.
- `generate-tags`: Genera tag único para la imagen.
- `build-push`: Construye y sube la imagen con optimizaciones de caché.

### Job: `create-git-tag` (Condicional)

Crea tags de Git si está habilitado (`create-git-tag: true`):

- `checkout`: Clona el repositorio.
- `create-tag`: Crea y push del tag de Git con la versión.

### Job: `deploy` (Condicional)

Despliega a ECS si está habilitado (`deploy-to-ecs: true`):

- `checkout`: Clona el repositorio.
- `configure-aws-credentials`: Autenticación con AWS.
- `render-task-definition`: Actualiza la task definition con la nueva imagen.
- `deploy-ecs`: Despliega el servicio ECS y espera estabilidad.

---

## 🚀 Ejemplos de Uso

### 1. Build y Deploy Completo

```yaml
name: Deploy to Production

on:
  push:
    branches:
      - main

jobs:
  deploy-prod:
    uses: innvlat/tos-gha-reusable-workflows/.github/workflows/docker-ecr-ecs-deploy.yaml@main
    with:
      account-id: "123456789012"
      aws-role-name: "Production-Deploy-Role"
      environment: "prod"
      ecr-repository: "mi-app-prod"
      ecs-service-name: "mi-app-service"
      ecs-cluster-name: "production-cluster"
      container-name: "api-container"
      deploy-to-ecs: true
    secrets: inherit
```

### 2. Solo Build (Sin Deploy)

```yaml
name: Build and Push Image

on:
  pull_request:
    branches: [main]

jobs:
  build-only:
    uses: innvlat/tos-gha-reusable-workflows/.github/workflows/docker-ecr-ecs-deploy.yaml@main
    with:
      account-id: "123456789012"
      aws-role-name: "CI-Role"
      environment: "staging"
      ecr-repository: "mi-app-staging"
      ecs-service-name: "dummy"    # Requerido pero no se usa
      ecs-cluster-name: "dummy"    # Requerido pero no se usa
      container-name: "dummy"      # Requerido pero no se usa
      deploy-to-ecs: false         # 🔑 No hacer deploy
      create-git-tag: false        # No crear tags en PRs
    secrets: inherit
```

### 3. Usando Outputs para Tests de Integración

```yaml
name: Deploy and Test

jobs:
  deploy:
    uses: innvlat/tos-gha-reusable-workflows/.github/workflows/docker-ecr-ecs-deploy.yaml@main
    with:
      account-id: "123456789012"
      aws-role-name: "Deploy-Role"
      environment: "dev"
      ecr-repository: "mi-app-dev"
      ecs-service-name: "mi-servicio"
      ecs-cluster-name: "dev-cluster"
      container-name: "api-container"
      deploy-to-ecs: true
    secrets: inherit

  integration-tests:
    needs: deploy
    runs-on: ubuntu-latest
    steps:
    - name: Run integration tests
      run: |
        echo "Testing image: ${{ needs.deploy.outputs.image-uri }}"
        echo "Image tag: ${{ needs.deploy.outputs.image-tag }}"
        # Ejecutar tests de integración aquí
```

> 🔒 **Recomendación de seguridad:** Usa un tag (`@v1.0.0`) o un commit hash en lugar de `@main` para asegurar versiones estables del workflow.

---

## ✅ Recomendaciones

- Asegúrate de tener el archivo `task-definition.json` en la raíz del repositorio (o especifica la ruta correcta).
- Usa `deploy-to-ecs: false` para workflows de CI que solo necesitan construir la imagen.
- El workflow crea un tag único: `{prefix}{run_number}` (compatible con ECR inmutable).
- Para producción, considera usar `create-git-tag: true` para trazabilidad.
- El caché de Docker reduce significativamente los tiempos de build en ejecuciones consecutivas.

## 🔒 Compatibilidad con ECR Inmutable

Este workflow está diseñado para funcionar con repositorios ECR que tienen **inmutabilidad de tags habilitada**. Por esta razón:

- ✅ **Solo genera un tag único** por imagen basado en `github.run_number`
- ✅ **No usa tags "latest"** que podrían causar conflictos
- ✅ **Cada build tiene un identificador único** que nunca se repite

> **Nota**: Si tu repositorio ECR tiene inmutabilidad deshabilitada y necesitas tags "latest", puedes fork este workflow y agregar la lógica adicional.

---

© 2025 - DevOps Infra Team 