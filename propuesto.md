# 🧪 Laboratorio Propuesto — Cloud Financial Management (FinOps)
## Módulo 8 · Sesión 2 | Arquitectura de Soluciones Multinube — UTEC
### 📋 VERSIÓN ALUMNO

> **Docente:** Aldo Trucios
> **Entrega:** Documento PDF con todas las tablas y respuestas completadas.
> **Modalidad:** Individual. Puedes consultar las diapositivas y los apuntes del laboratorio guiado.

---

## 📋 Tabla de Contenidos

1. [Objetivos](#1-objetivos-del-laboratorio)
2. [Contexto del Caso](#2-contexto-del-caso--retailcorp-perú)
3. [Parte 1 — Visibilidad y Tagging](#3-parte-1--diagnóstico-de-visibilidad-y-tagging)
4. [Parte 2 — Costos y Alertas](#4-parte-2--análisis-de-costos-y-alertas)
5. [Parte 3 — Right-Sizing](#5-parte-3--right-sizing-y-optimización-de-uso)
6. [Parte 4 — Recursos Huérfanos](#6-parte-4--recursos-huérfanos-y-quick-wins)
7. [Parte 5 — Storage y Ciclo de Vida](#7-parte-5--estrategia-de-storage-y-ciclo-de-vida)
8. [Parte 6 — Hoja de Ruta FinOps](#8-parte-6--hoja-de-ruta-finops)
9. [Criterios de Evaluación](#9-criterios-de-evaluación)

---

## 1. Objetivos del Laboratorio

- ✅ Diagnosticar el nivel de madurez FinOps usando el modelo Crawl-Walk-Run
- ✅ Calcular el Tagging Compliance e identificar su impacto en el Chargeback
- ✅ Diseñar una estrategia de alertas proactivas con AWS Budgets y SNS
- ✅ Evaluar recomendaciones de right-sizing y cuantificar su impacto anual
- ✅ Identificar y valorizar recursos huérfanos como quick wins
- ✅ Diseñar una política de ciclo de vida S3 y calcular su ahorro
- ✅ Proponer una hoja de ruta FinOps priorizada con impacto en dólares

---

## 2. Contexto del Caso — RetailCorp Perú

**RetailCorp Perú** es una empresa de retail con presencia en 12 ciudades y 3 equipos en AWS:

| Equipo | Descripción | Responsable |
|--------|-------------|-------------|
| **E-Commerce** | Plataforma de ventas online, picos en campañas | Javier (CTO) |
| **Data & Analytics** | Pipelines de datos, reportes de ventas | Patricia (Data Lead) |
| **Backend Core** | APIs internas, bases de datos, ERP | Carlos (Infra Lead) |

El CFO acaba de recibir la factura de AWS: **$33,900**. Nadie puede explicar con precisión a qué corresponde ese gasto.

### 2.1 — Factura AWS del mes

| Servicio AWS | Gasto mensual | % del total |
|--------------|:---:|:---:|
| Amazon RDS | $9,200 | 27.1% |
| Amazon EC2 | $8,700 | 25.7% |
| Amazon S3 | $7,800 | 23.0% |
| AWS Data Transfer | $4,100 | 12.1% |
| Amazon CloudFront | $2,600 | 7.7% |
| Otros servicios | $1,500 | 4.4% |
| **TOTAL** | **$33,900** | **100%** |

### 2.2 — Inventario de recursos EC2

| ID | Nombre | Tipo | vCPU | RAM | CPU prom. | RAM prom. | Equipo | Tags completos |
|----|--------|------|:---:|:---:|:---:|:---:|--------|:---:|
| i-001 | web-prod-01 | m5.4xlarge | 16 | 64 GB | 8% | 12% | E-Commerce | No |
| i-002 | web-prod-02 | m5.4xlarge | 16 | 64 GB | 7% | 11% | E-Commerce | No |
| i-003 | web-prod-03 | m5.4xlarge | 16 | 64 GB | 9% | 13% | E-Commerce | No |
| i-004 | web-prod-04 | m5.4xlarge | 16 | 64 GB | 8% | 12% | E-Commerce | No |
| i-005 | cache-redis-01 | m5.xlarge | 4 | 16 GB | 22% | 61% | E-Commerce | No |
| i-006 | staging-env | m5.xlarge | 4 | 16 GB | 2% | 4% | E-Commerce | No |
| i-007 | api-backend-01 | m5.2xlarge | 8 | 32 GB | 41% | 55% | Backend Core | Sí |
| i-008 | api-backend-02 | m5.2xlarge | 8 | 32 GB | 38% | 52% | Backend Core | Sí |
| i-009 | api-backend-03 | m5.2xlarge | 8 | 32 GB | 36% | 50% | Backend Core | Sí |
| i-010 | erp-app-01 | r5.2xlarge | 8 | 64 GB | 45% | 62% | Backend Core | Sí |
| i-011 | erp-app-02 | r5.2xlarge | 8 | 64 GB | 43% | 59% | Backend Core | Sí |
| i-012 | data-processing | m5.8xlarge | 32 | 128 GB | 6% | 9% | Data & Analytics | No |
| i-013 | ml-training | p3.2xlarge | 8 | 61 GB | 3% | 5% | Data & Analytics | No |
| i-014 | etl-worker-01 | c5.2xlarge | 8 | 16 GB | 31% | 28% | Data & Analytics | No |
| i-015 | etl-worker-02 | c5.2xlarge | 8 | 16 GB | 28% | 25% | Data & Analytics | No |
| i-016 | etl-worker-03 | c5.2xlarge | 8 | 16 GB | 33% | 30% | Data & Analytics | No |
| i-017 | dev-old | t3.medium | 2 | 4 GB | 0% | 1% | Desconocido | No |
| i-018 | test-loadgen | t3.large | 2 | 8 GB | 1% | 2% | Desconocido | No |

### 2.3 — Inventario de recursos huérfanos

| Recurso | Tipo | Detalle | Costo/mes |
|---------|------|---------|:---------:|
| eip-prod-legacy | Elastic IP | Sin asociar a instancia | $3.65 |
| eip-test-01 | Elastic IP | Sin asociar a instancia | $3.65 |
| eip-data-old | Elastic IP | Sin asociar a instancia | $3.65 |
| vol-backup-2023 | EBS gp2 500 GB | Sin adjuntar, último snapshot ene-2023 | $50.00 |
| vol-staging-old | EBS gp2 200 GB | Sin adjuntar, sin dueño identificado | $20.00 |
| lb-ecommerce-v1 | Load Balancer | 0 targets registrados en 45 días | $16.20 |

### 2.4 — Inventario S3

| Bucket | Tamaño | Acceso promedio | Lifecycle |
|--------|:---:|:---:|:---:|
| retailcorp-sales-data | 4.2 TB | 15 días | No |
| retailcorp-logs-app | 8.7 TB | 94 días | No |
| retailcorp-backups | 12.1 TB | 380 días | No |
| retailcorp-assets-cdn | 1.1 TB | 3 días | No |

### 2.5 — Precios de referencia (us-east-1)

**EC2 On-Demand:**

| Tipo | vCPU | RAM | Precio/hora | Precio/mes (730h) |
|------|:---:|:---:|:---:|:---:|
| t3.medium | 2 | 4 GB | $0.0416 | $30 |
| t3.large | 2 | 8 GB | $0.0832 | $61 |
| m5.large | 2 | 8 GB | $0.096 | $70 |
| m5.xlarge | 4 | 16 GB | $0.192 | $140 |
| m5.2xlarge | 8 | 32 GB | $0.384 | $280 |
| m5.4xlarge | 16 | 64 GB | $0.768 | $561 |
| m5.8xlarge | 32 | 128 GB | $1.536 | $1,121 |
| c5.2xlarge | 8 | 16 GB | $0.34 | $248 |
| r5.2xlarge | 8 | 64 GB | $0.504 | $368 |
| p3.2xlarge | 8 | 61 GB GPU | $3.06 | $2,234 |

**S3 Storage:**

| Clase | Precio/GB/mes |
|-------|:---:|
| Standard | $0.023 |
| Standard-IA | $0.0125 |
| Glacier Instant Retrieval | $0.004 |
| Glacier Flexible Retrieval | $0.0036 |

---

## 3. Parte 1 — Diagnóstico de Visibilidad y Tagging

### Ejercicio 1.1 — Calcular el Tagging Compliance

**a)** Completa la tabla:

| Recurso | ¿Tags completos? | Gasto/mes |
|---------|:---:|:---:|
| i-001 web-prod-01 (m5.4xlarge) | | $ |
| i-002 web-prod-02 (m5.4xlarge) | | $ |
| i-003 web-prod-03 (m5.4xlarge) | | $ |
| i-004 web-prod-04 (m5.4xlarge) | | $ |
| i-005 cache-redis-01 (m5.xlarge) | | $ |
| i-006 staging-env (m5.xlarge) | | $ |
| i-007 api-backend-01 (m5.2xlarge) | | $ |
| i-008 api-backend-02 (m5.2xlarge) | | $ |
| i-009 api-backend-03 (m5.2xlarge) | | $ |
| i-010 erp-app-01 (r5.2xlarge) | | $ |
| i-011 erp-app-02 (r5.2xlarge) | | $ |
| i-012 data-processing (m5.8xlarge) | | $ |
| i-013 ml-training (p3.2xlarge) | | $ |
| i-014 etl-worker-01 (c5.2xlarge) | | $ |
| i-015 etl-worker-02 (c5.2xlarge) | | $ |
| i-016 etl-worker-03 (c5.2xlarge) | | $ |
| i-017 dev-old (t3.medium) | | $ |
| i-018 test-loadgen (t3.large) | | $ |

**b)** Calcula:

```
Total de recursos EC2:                    ______
Recursos con tagging completo:            ______
Tagging Compliance:                       ______  %

Gasto total instancias EC2:            $  ______
Gasto asignable (con tags):            $  ______  ( ______  %)
Gasto NO asignable (sin tags):         $  ______  ( ______  %)
```

**c)** El CFO pregunta: *"¿a qué equipo le cargo los $8,700 de EC2?"* ¿Qué le respondes con los datos actuales? ¿Qué necesitas para poder responderle correctamente? *(3-5 líneas)*

---

### Ejercicio 1.2 — Diseñar la taxonomía de tags

| Clave | Valores válidos | ¿Qué pasa si falta? |
|-------|-----------------|---------------------|
| `Environment` | | |
| `CostCenter` | | |
| `ApplicationName` | | |
| `OwnerEmail` | | |
| `Team` | | |

---

### Ejercicio 1.3 — Madurez FinOps

```
Fase actual de RetailCorp:   ________________

Evidencia 1: _______________________________________________
Evidencia 2: _______________________________________________
Evidencia 3: _______________________________________________
```

---

## 4. Parte 2 — Análisis de Costos y Alertas

### Ejercicio 2.1 — Distribuir el presupuesto

El CFO asigna **$34,000/mes**. Distribúyelo entre los 3 equipos:

| Equipo | Presupuesto mensual | Justificación |
|--------|:---:|---------------|
| E-Commerce | $ | |
| Data & Analytics | $ | |
| Backend Core | $ | |
| **TOTAL** | **$34,000** | |

---

### Ejercicio 2.2 — Estrategia de Budgets

| Equipo | Nombre Budget | Monto | Alerta 1 | Alerta 2 | Canal |
|--------|--------------|:---:|:---:|:---:|-------|
| E-Commerce | | $ | % Actual | % Forecasted | |
| Data & Analytics | | $ | % Actual | % Forecasted | |
| Backend Core | | $ | % Actual | % Forecasted | |

---

### Ejercicio 2.3 — Calcular el Unit Cost

El equipo de E-Commerce procesó **380,000 transacciones** el mes pasado.

**a)**
```
Servicios de E-Commerce identificados:   ________________________________
Gasto estimado E-Commerce mensual:     $ ________________________________
```

**b)**
```
Unit Cost actual:   $ ________________  por transacción

Si el objetivo es $0.03/transacción:
  Gasto máximo permitido:   $ ________________ /mes
```

**c)** ¿Por qué el Unit Cost es más útil que el gasto total para presentar al CEO? *(3-5 líneas)*

---

## 5. Parte 3 — Right-Sizing y Optimización de Uso

### Ejercicio 3.1 — Análisis de right-sizing

> Regla de referencia: el tipo recomendado debe poder manejar al menos el doble del uso promedio para absorber picos.

| Instancia | Tipo actual | CPU | RAM | Tipo recomendado | Razonamiento | Actual/mes | Nuevo/mes | Ahorro/mes |
|-----------|-------------|:---:|:---:|-----------------|--------------|:---:|:---:|:---:|
| i-001 web-prod-01 | m5.4xlarge | 8% | 12% | | | $561 | $ | $ |
| i-002 web-prod-02 | m5.4xlarge | 7% | 11% | | | $561 | $ | $ |
| i-003 web-prod-03 | m5.4xlarge | 9% | 13% | | | $561 | $ | $ |
| i-004 web-prod-04 | m5.4xlarge | 8% | 12% | | | $561 | $ | $ |
| i-005 cache-redis-01 | m5.xlarge | 22% | 61% | | | $140 | $ | $ |
| i-006 staging-env | m5.xlarge | 2% | 4% | | | $140 | $ | $ |
| i-012 data-processing | m5.8xlarge | 6% | 9% | | | $1,121 | $ | $ |
| i-013 ml-training | p3.2xlarge | 3% | 5% | | | $2,234 | $ | $ |
| i-017 dev-old | t3.medium | 0% | 1% | | | $30 | $ | $ |
| i-018 test-loadgen | t3.large | 1% | 2% | | | $61 | $ | $ |

```
Ahorro mensual total:    $ ________________
Ahorro anual:            $ ________________
% reducción sobre EC2:     ________________  %
```

---

### Ejercicio 3.2 — Instancias Spot para i-012 e i-013

**a)**
```
i-012 data-processing (tras right-sizing a m5.2xlarge):
  On-Demand m5.2xlarge:   $ ________________ /mes
  Spot (70% desc.):       $ ________________ /mes
  Ahorro adicional Spot:  $ ________________

i-013 ml-training (p3.2xlarge Spot):
  On-Demand actual:       $ ________________ /mes
  Spot (70% desc.):       $ ________________ /mes
  Ahorro:                 $ ________________
```

**b)** 2 cargas que SÍ pueden usar Spot y 2 que NO:

| ¿Spot? | Carga | Justificación |
|:---:|---|---|
| ✅ SÍ | | |
| ✅ SÍ | | |
| ❌ NO | | |
| ❌ NO | | |

---

## 6. Parte 4 — Recursos Huérfanos y Quick Wins

### Ejercicio 4.1 — Valorizar recursos huérfanos

**a)**

| Recurso | Costo/mes | Costo anual | Acción recomendada | Urgencia |
|---------|:---------:|:-----------:|-------------------|:--------:|
| eip-prod-legacy | $3.65 | $ | | |
| eip-test-01 | $3.65 | $ | | |
| eip-data-old | $3.65 | $ | | |
| vol-backup-2023 (500 GB) | $50.00 | $ | | |
| vol-staging-old (200 GB) | $20.00 | $ | | |
| lb-ecommerce-v1 | $16.20 | $ | | |
| **TOTAL** | $ | $ | | |

**b)**
```
Ahorro mensual:   $ ________________
Ahorro anual:     $ ________________
```

**c)** ¿Qué proceso propones para decidir si apagar o eliminar i-017 e i-018? *(3-5 líneas)*

---

### Ejercicio 4.2 — Apagado programado staging

El entorno `staging` (i-006) solo se usa lunes a viernes de 9am a 7pm.

```
Horas de uso real por semana:    ________________ horas
Horas de uso real por mes:       ________________ horas
% de tiempo necesario:           ________________ %

Costo actual (730 horas):        $140.16/mes
Costo con apagado programado:  $ ________________ /mes
Ahorro mensual:                $ ________________
Ahorro anual:                  $ ________________
```

---

## 7. Parte 5 — Estrategia de Storage y Ciclo de Vida

### Ejercicio 5.1 — Políticas de ciclo de vida

**`retailcorp-sales-data` (4.2 TB, acceso 15 días)**
```
Patrón: _______________________________________________

  Día   0:  Standard
  Día ___:  ________________   (justificación: _______________)
  Día ___:  ________________   (justificación: _______________)
  Día ___:  Expiración         (justificación: _______________)
```

**`retailcorp-logs-app` (8.7 TB, acceso 94 días)**
```
Patrón: _______________________________________________

  Día   0:  Standard
  Día ___:  ________________   (justificación: _______________)
  Día ___:  ________________   (justificación: _______________)
  Día ___:  Expiración         (justificación: _______________)
```

**`retailcorp-backups` (12.1 TB, acceso 380 días)**
```
Patrón: _______________________________________________

  Día   0:  Standard
  Día ___:  ________________   (justificación: _______________)
  Día ___:  ________________   (justificación: _______________)
  Día ___:  Expiración         (justificación: _______________)
```

**`retailcorp-assets-cdn` (1.1 TB, acceso 3 días)**
```
¿Necesita lifecycle?   Sí / No
Justificación: ________________________________________________
```

---

### Ejercicio 5.2 — Calcular el ahorro de storage

**a) Estado actual:**
```
retailcorp-sales-data:    4,301 GB × $0.023 = $ ________________ /mes
retailcorp-logs-app:      8,909 GB × $0.023 = $ ________________ /mes
retailcorp-backups:      12,390 GB × $0.023 = $ ________________ /mes
retailcorp-assets-cdn:    1,126 GB × $0.023 = $ ________________ /mes
Gasto S3 storage actual:                      $ ________________ /mes
```

**b) Con lifecycle:**

| Bucket | Standard (GB) | Standard-IA (GB) | Glacier (GB) | Costo nuevo/mes |
|--------|:---:|:---:|:---:|:---:|
| sales-data | | | | $ |
| logs-app | | | | $ |
| backups | | | | $ |
| assets-cdn | | | | $ |
| **Total** | | | | **$** |

```
Gasto S3 storage actual:          $ ________________ /mes
Gasto S3 storage con lifecycle:   $ ________________ /mes
Ahorro mensual:                   $ ________________ /mes
Ahorro anual:                     $ ________________ /año
```

---

## 8. Parte 6 — Hoja de Ruta FinOps

### Ejercicio 6.1 — Consolidar el ahorro total

| Iniciativa | Ahorro/mes | Ahorro/año | Esfuerzo | Plazo |
|-----------|:---:|:---:|:---:|:---:|
| Right-sizing EC2 | $ | $ | | |
| Recursos huérfanos | $ | $ | | |
| Apagado staging | $ | $ | | |
| Lifecycle S3 | $ | $ | | |
| **TOTAL** | **$** | **$** | | |

```
Factura actual:                        $33,900/mes
Ahorro total identificado:             $ ________________ /mes
Factura proyectada post-optimización:  $ ________________ /mes
Reducción porcentual:                    ________________  %
```

---

### Ejercicio 6.2 — Priorizar la hoja de ruta

```
Prioridad 1:   ______________________
  Justificación: ___________________

Prioridad 2:   ______________________
  Justificación: ___________________

Prioridad 3:   ______________________
  Justificación: ___________________

Prioridad 4:   ______________________
  Justificación: ___________________
```

---

### Ejercicio 6.3 — Mensaje al CFO

*(Máximo 8 líneas: diagnóstico del problema, ahorro identificado, primera acción concreta)*

---

## 9. Criterios de Evaluación

| Criterio | Peso |
|----------|------|
| Correctitud de los cálculos numéricos (Partes 1-5) | 35% |
| Calidad del análisis y justificaciones escritas | 30% |
| Hoja de ruta priorizada con criterio claro (Parte 6) | 20% |
| Conexión con conceptos de la sesión (Ciclo FinOps, Madurez, Unit Economics) | 15% |

---

## 📚 Referencias

| Recurso | URL |
|---------|-----|
| FinOps Foundation | https://www.finops.org/framework/ |
| AWS EC2 Pricing | https://aws.amazon.com/ec2/pricing/on-demand/ |
| AWS S3 Storage Classes | https://aws.amazon.com/s3/storage-classes/ |
| AWS Budgets | https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-create.html |
| AWS Compute Optimizer | https://docs.aws.amazon.com/compute-optimizer/latest/ug/what-is-compute-optimizer.html |
| AWS Trusted Advisor | https://docs.aws.amazon.com/awssupport/latest/user/trusted-advisor-check-reference.html |

---

*Laboratorio elaborado por el docente Aldo Trucios · Programa Arquitectura de Soluciones Multinube · UTEC Posgrado*
*Módulo 8 · Sesión 2 — Cloud Financial Management (FinOps) · **VERSIÓN ALUMNO***
