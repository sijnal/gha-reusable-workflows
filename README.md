# Reusable GitHub Actions Workflows

Este repositorio contiene un conjunto de **workflows reutilizables para GitHub Actions**, diseñados para simplificar y estandarizar procesos comunes de CI/CD en proyectos que usan Terraform y aplicaciones con Docker.

## Workflows disponibles

1. **Terraform Plan**  
   `terraform-plan.yaml`  
   Genera y valida el plan de ejecución de Terraform con análisis de seguridad y calidad.

2. **Terraform Apply**  
   `terraform-apply.yaml`  
   Aplica los cambios de infraestructura definidos en Terraform de forma segura.

3. **Docker ECR ECS Deploy**  
   `docker-ecr-ecs-deploy.yaml`  
   Construye imágenes Docker, las sube a ECR y despliega en ECS de forma condicional.

---

Cada uno de estos workflows está diseñado para ser **reutilizado desde múltiples repositorios**, facilitando la consistencia y eficiencia en tus pipelines de DevOps.

> La documentación detallada de cada workflow está disponible en `docs/`.

## Características principales

### Terraform Workflows
- ✅ **Seguridad**: Autenticación AWS via OIDC
- ✅ **Calidad**: Integración con TFLint y Checkov
- ✅ **Flexibilidad**: Configuración por ambiente
- ✅ **Trazabilidad**: Comentarios automáticos en PRs
- ✅ **Buenas prácticas**: Separación plan/apply

### Docker ECR ECS Workflow
- ✅ **Construcción optimizada**: Docker Buildx con caché
- ✅ **Despliegue condicional**: Control via parámetro `deploy-to-ecs`
- ✅ **Task Definition flexible**: Ruta parametrizable
- ✅ **Outputs disponibles**: URI e imagen tag para composición
