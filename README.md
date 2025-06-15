# Reusable GitHub Actions Workflows

Este repositorio contiene un conjunto de **workflows reutilizables para GitHub Actions**, diseñados para simplificar y estandarizar procesos comunes de CI/CD en proyectos que usan Terraform, Docker y despliegue en ECS.

## Workflows disponibles

1. **Terraform Plan**  
   `terraform-plan.yaml`  
   Genera y valida el plan de ejecución de Terraform.

2. **Terraform Apply**  
   `terraform-apply.yaml`  
   Aplica los cambios de infraestructura definidos en Terraform.

3. **Docker CI**  
   `ci-ecr.yaml`  
   Compila, analiza y prueba imágenes Docker en el proceso de CI.

4. **ECS Deploy**  
   `cd-ecs.yaml`  
   Realiza el despliegue de imágenes Docker a Amazon ECS.

5. **ECS Rollback**  
   `rollback-ecs.yaml`  
   Revierte un despliegue en ECS a una versión anterior estable.

---

Cada uno de estos workflows está diseñado para ser **reutilizado desde múltiples repositorios**, facilitando la consistencia y eficiencia en tus pipelines de DevOps.

> La documentación detallada de cada workflow estará disponible próximamente.
