
# Terraform Apply Workflow

Este documento describe un workflow reutilizable de GitHub Actions para ejecutar `Terraform Apply`. Este pipeline está diseñado para trabajar con proyectos de Terraform y facilita la aplicación de cambios en la infraestructura gestionada con Terraform.

---

## 🚀 Descripción

Este flujo realiza lo siguiente:

1. Configura credenciales de AWS.
2. Instala la versión especificada de Terraform.
3. Verifica el formato con `terraform fmt`.
4. Inicializa Terraform con configuración de backend.
5. Valida los archivos de configuración.
6. Ejecuta `terraform apply` con aprobación automática.

---

## 📥 Entradas (Inputs)

| Input                | Requerido | Descripción                                                       | Valor por defecto  |
|----------------------|-----------|-------------------------------------------------------------------|--------------------|
| `aws-role-arn`       | ✅        | ARN del rol de AWS a asumir.                                      | —                  |
| `aws-region`         | ❌        | Región de AWS donde ejecutar Terraform.                           | `us-east-1`        |
| `environment`        | ✅        | Nombre del entorno (dev, staging, prod) usado para cargar config. | —                  |
| `terraform-version`  | ❌        | Versión de Terraform a usar.                                      | `1.10.0`           |

---

## 🔐 Secretos Requeridos

Este workflow usa autenticación con AWS vía OIDC. No se requiere configuración adicional de secretos.

---

## 🧱 Jobs

### `apply`

- **Runner**: `ubuntu-latest`
- **Timeout**: 30 minutos
- **Permisos**:
  - `id-token`: `write`
  - `contents`: `read`

### Pasos principales

1. Clona el repositorio.
2. Configura las credenciales de AWS.
3. Instala la versión deseada de Terraform.
4. Verifica el formato con `terraform fmt`.
5. Inicializa con configuración específica del entorno (`conf/<env>.conf`).
6. Valida la configuración.
7. Aplica los cambios (`terraform apply`) con aprobación automática.

---

## 🧪 Ejemplo de uso

Este workflow debe ser llamado desde otro flujo usando `workflow_call`. Ejemplo:

```yaml
name: Aplicar infraestructura

on:
  workflow_dispatch:
  push:
    branches:
      - main
      - develop

jobs:
  apply:
    uses: innvlat/tos-gha-reusable-workflows/.github/workflows/terraform-apply.yaml@main
    with:
      aws-role-arn: arn:aws:iam::123456789012:role/my-gha-role
      environment: dev
    secrets: inherit
```
> 🔒 **Recomendación de seguridad:** Usa un tag (`@v1.0.0`) o un commit hash en lugar de `@main` para asegurar versiones estables del workflow.


---

## ✅ Recomendaciones

Asegúrate de tener estos archivos en tu estructura:

- `conf/<environment>.conf`
- `env/<environment>.tfvars`

Usa este workflow para aplicar cambios únicamente luego de revisar y aprobar el `terraform plan`.

Mantén separados los workflows de `plan` y `apply` para mayor control sobre los cambios aplicados.

---

© 2025 - DevOps Infra Team
