# Terraform Plan - GitHub Workflow

Este documento describe un workflow reutilizable de GitHub Actions para ejecutar `Terraform Plan`. Este pipeline está diseñado para trabajar con proyectos de Terraform y facilita la validación, formateo, y planificación de cambios en la infraestructura gestionada con Terraform.

---

## 🛠 Funcionalidades del Workflow

Este workflow realiza lo siguiente:

1. Clona el repositorio.
2. Configura credenciales de AWS asumiendo un rol vía OIDC.
3. Instala la versión especificada de Terraform.
4. Ejecuta `terraform fmt` para verificar el formato del código.
5. Inicializa Terraform (`terraform init`) con configuración según el entorno.
6. Valida los archivos con `terraform validate`.
7. Instala y ejecuta `TFLint` para análisis estático.
8. Ejecuta `Checkov` para auditoría de seguridad.
9. Ejecuta `terraform plan`.
10. Comenta automáticamente los resultados en pull requests (si aplica).

---

## 📥 Entradas del Workflow

El workflow acepta las siguientes entradas mediante `workflow_call`:

| Nombre                  | Requerido | Tipo   | Valor por defecto | Descripción                                                                 |
|-------------------------|-----------|--------|--------------------|-----------------------------------------------------------------------------|
| `account-id`            | ✅        | string | —                  | ID de la cuenta de AWS donde se encuentra el rol.                          |
| `aws-role-name`         | ✅        | string | —                  | Nombre del rol de AWS a asumir (sin el prefijo ARN).                       |
| `aws-region`            | ❌        | string | `us-east-1`        | Región de AWS donde se ejecutarán los comandos.                            |
| `environment`           | ✅        | string | —                  | Entorno para seleccionar la configuración (`.conf` y `.tfvars`).           |
| `lint-config-file`      | ❌        | string | `.tflint.hcl`      | Ruta del archivo de configuración de TFLint.                               |
| `terraform-version`     | ❌        | string | `1.10.0`           | Versión de Terraform a instalar.                                           |
| `rules-to-skip-checkov` | ❌        | string | `""`               | Reglas de Checkov a omitir (separadas por comas, ej. `CKV_AWS_1,CKV_AWS_2`).|

---

## 🔐 Secretos Requeridos

Este workflow usa autenticación con AWS vía OIDC y utiliza el token interno `${{ secrets.GITHUB_TOKEN }}` para comentar en pull requests. No se requiere configuración adicional de secretos.

---

## 🧪 Job: `plan`

Este único job realiza todos los pasos clave:

- `checkout`: Clona el repositorio.
- `configure-aws-credentials`: Autenticación con AWS usando el rol indicado.
- `setup-terraform`: Instala la versión definida de Terraform.
- `terraform fmt`: Valida formato del código Terraform.
- `terraform init`: Inicializa la configuración del backend.
- `terraform validate`: Valida sintaxis y estructura del código.
- `tflint`: Ejecuta análisis de linting con TFLint.
- `checkov`: Realiza auditoría de seguridad con Checkov.
- `terraform plan`: Genera el plan de ejecución.
- `github-script`: Publica los resultados del plan en un comentario de pull request.

---

## 🚀 Ejemplo de Uso

Puedes usar este workflow desde otro repositorio llamándolo así:

```yaml
name: Plan infraestructura

on:
  workflow_dispatch:
  pull_request:
    branches:
      - main
      - develop

jobs:
  terraform-plan:
    uses: innvlat/tos-gha-reusable-workflows/.github/workflows/terraform-plan.yaml@main
    with:
      account-id: "123456789012"
      aws-role-name: "gha-terraform-role"
      environment: dev
    secrets: inherit
```
> 🔒 **Recomendación de seguridad:** Usa un tag (`@v1.0.0`) o un commit hash en lugar de `@main` para asegurar versiones estables del workflow.


---

## ✅ Recomendaciones

- Asegúrate de tener estos archivos en tu estructura:
  - `backend/<environment>.conf`
  - `env/<environment>.tfvars`
- Usa este workflow para prevenir errores antes de aplicar cambios a producción.
- Se recomienda separar `terraform plan` y `terraform apply` en diferentes workflows para mantener control de cambios.

---

© 2025 - DevOps Infra Team