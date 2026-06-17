# 🧪 Laboratorio Guiado — Cloud Financial Management (FinOps)
## Módulo 8 · Sesión 2 | Arquitectura de Soluciones Multinube — UTEC
### 📋 VERSIÓN DOCENTE — Con respuestas

> **Docente:** Aldo Trucios
> **Duración:** 1 hora 15 minutos
> **Modalidad:** Guiado — el docente explica cada ejercicio y revela las respuestas progresivamente. Los alumnos trabajan con el documento LAB_PROPUESTO_M08S02 (respuestas en blanco).

---

## 📋 Tabla de Contenidos

1. [Objetivos](#1-objetivos-del-laboratorio)
2. [Contexto del Caso](#2-contexto-del-caso--retailcorp-perú)
3. [Parte 1 — Visibilidad y Tagging](#3-parte-1--diagnóstico-de-visibilidad-y-tagging-15-min)
4. [Parte 2 — Costos y Alertas](#4-parte-2--análisis-de-costos-y-alertas-15-min)
5. [Parte 3 — Right-Sizing](#5-parte-3--right-sizing-y-optimización-de-uso-15-min)
6. [Parte 4 — Recursos Huérfanos](#6-parte-4--recursos-huérfanos-y-quick-wins-10-min)
7. [Parte 5 — Storage y Ciclo de Vida](#7-parte-5--estrategia-de-storage-y-ciclo-de-vida-10-min)
8. [Parte 6 — Hoja de Ruta FinOps](#8-parte-6--hoja-de-ruta-finops-10-min)

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

> 💬 **[DOCENTE]** Los $8,700 de EC2 incluyen $8,537 de instancias On-Demand más ~$163 de EBS volumes adjuntos y snapshots. Ese detalle no es relevante para el ejercicio pero lo puedes mencionar si alguien lo pregunta.

### 2.2 — Inventario de recursos EC2

| ID | Nombre | Tipo | vCPU | RAM | CPU prom. | RAM prom. | Equipo | Tags completos |
|----|--------|------|:---:|:---:|:---:|:---:|--------|:---:|
| i-001 | web-prod-01 | m5.4xlarge | 16 | 64 GB | 8% | 12% | E-Commerce | No |
| i-002 | web-prod-02 | m5.4xlarge | 16 | 64 GB | 7% | 11% | E-Commerce | No |
| i-003 | web-prod-03 | m5.4xlarge | 16 | 64 GB | 9% | 13% | E-Commerce | No |
| i-004 | web-prod-04 | m5.4xlarge | 16 | 64 GB | 8% | 12% | E-Commerce | No |
| i-005 | cache-redis-01 | m5.xlarge | 4 | 16 GB | 22% | 61% | E-Commerce | No |
| i-006 | staging-env | m5.xlarge | 4 | 16 GB | 2% | 4% | E-Commerce | No |
| i-007 | api-backend-01 | m5.2xlarge | 8 | 32 GB | 41% | 55% | Backend Core | **Sí** |
| i-008 | api-backend-02 | m5.2xlarge | 8 | 32 GB | 38% | 52% | Backend Core | **Sí** |
| i-009 | api-backend-03 | m5.2xlarge | 8 | 32 GB | 36% | 50% | Backend Core | **Sí** |
| i-010 | erp-app-01 | r5.2xlarge | 8 | 64 GB | 45% | 62% | Backend Core | **Sí** |
| i-011 | erp-app-02 | r5.2xlarge | 8 | 64 GB | 43% | 59% | Backend Core | **Sí** |
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

---

## 3. Parte 1 — Diagnóstico de Visibilidad y Tagging ⏱ 15 min

---

### Ejercicio 1.1 — Calcular el Tagging Compliance

> 💬 **[DOCENTE]** Explica primero el concepto: sin tags la factura es una caja negra. El CFO ve $8,700 en EC2 pero no sabe a quién cobrarle. El tagging es la infraestructura del Chargeback.

**a) Tabla completada:**

| Recurso | Tags completos | Gasto/mes |
|---------|:---:|:---:|
| i-001 web-prod-01 (m5.4xlarge) | ❌ No | $561 |
| i-002 web-prod-02 (m5.4xlarge) | ❌ No | $561 |
| i-003 web-prod-03 (m5.4xlarge) | ❌ No | $561 |
| i-004 web-prod-04 (m5.4xlarge) | ❌ No | $561 |
| i-005 cache-redis-01 (m5.xlarge) | ❌ No | $140 |
| i-006 staging-env (m5.xlarge) | ❌ No | $140 |
| i-007 api-backend-01 (m5.2xlarge) | ✅ Sí | $280 |
| i-008 api-backend-02 (m5.2xlarge) | ✅ Sí | $280 |
| i-009 api-backend-03 (m5.2xlarge) | ✅ Sí | $280 |
| i-010 erp-app-01 (r5.2xlarge) | ✅ Sí | $368 |
| i-011 erp-app-02 (r5.2xlarge) | ✅ Sí | $368 |
| i-012 data-processing (m5.8xlarge) | ❌ No | $1,121 |
| i-013 ml-training (p3.2xlarge) | ❌ No | $2,234 |
| i-014 etl-worker-01 (c5.2xlarge) | ❌ No | $248 |
| i-015 etl-worker-02 (c5.2xlarge) | ❌ No | $248 |
| i-016 etl-worker-03 (c5.2xlarge) | ❌ No | $248 |
| i-017 dev-old (t3.medium) | ❌ No | $30 |
| i-018 test-loadgen (t3.large) | ❌ No | $61 |

**b) Cálculo:**

```
Total de recursos EC2:                    18
Recursos con tagging completo:             5  (i-007 a i-011, Backend Core)
Tagging Compliance:                       28%

Gasto total instancias EC2:            $8,537
Gasto asignable (con tags):            $1,576   (20.5% — solo Backend Core)

Nota: r5.2xlarge en el precio de referencia = $368/mes (no $456).
  i-010 + i-011 = $368 + $368 = $736
  i-007 + i-008 + i-009 = $280 × 3 = $840
  Total asignable = $736 + $840 = $1,576

Gasto NO asignable (sin tags):         $6,961   (79.5% — invisible al CFO)
```

> 💬 **[DOCENTE]** El impacto es demoledor: el CFO puede justificar solo el 20.5% del gasto EC2. El 79.5% restante no tiene dueño. Esto hace imposible el Chargeback y la toma de decisiones financieras correctas.

**c) Respuesta al CFO:**

> *"En este momento solo puedo asignarte con certeza $1,576 de los $8,700 de EC2, correspondientes a los 5 servidores del equipo de Backend Core que sí tienen etiquetas completas. El 79.5% restante no tiene dueño asignado técnicamente. Para responder tu pregunta necesito implementar una taxonomía de tags obligatoria esta semana y retroaplicarla a los 13 recursos que hoy son invisibles."*

---

### Ejercicio 1.2 — Diseñar la taxonomía de tags

> 💬 **[DOCENTE]** Hay dos niveles de control: SCP bloqueante (el recurso no se puede crear sin el tag) y AWS Config Rule (alerta cuando falta el tag). Los más críticos para el Chargeback van con SCP.

| Clave | Valores válidos | ¿Qué pasa si falta? |
|-------|-----------------|---------------------|
| `Environment` | `prod`, `staging`, `dev`, `qa` | SCP bloquea la creación del recurso |
| `CostCenter` | `CC-ECOM`, `CC-DATA`, `CC-BACK`, `CC-SHARED` | SCP bloquea la creación del recurso |
| `ApplicationName` | nombre en minúsculas (ej: `ecommerce-web`, `erp-core`) | SCP bloquea la creación del recurso |
| `OwnerEmail` | email corporativo `@retailcorp.pe` | AWS Config genera alerta automática |
| `Team` | `ecommerce`, `data-analytics`, `backend-core`, `shared` | AWS Config genera alerta automática |

---

### Ejercicio 1.3 — Madurez FinOps

> 💬 **[DOCENTE]** Pide a los alumnos que voten antes de revelar. La mayoría votará Crawl, que es correcto.

**Fase actual: CRAWL** 🐛

| Evidencia | Por qué indica Crawl |
|-----------|---------------------|
| Nadie puede explicar los $33,900 de factura | Sin visibilidad básica → Crawl |
| 72% de instancias EC2 sin tags | Tagging Compliance < 50% → Crawl |
| Sin Budgets ni alertas configuradas | Sin alertas proactivas → Crawl |
| EIPs, EBS y Load Balancer huérfanos activos | Sin proceso de limpieza → Crawl |
| Sin asignación de costos por equipo | Sin Showback ni Chargeback → Crawl |

> 💬 **[DOCENTE]** En fase **Walk** ya hay Showback y alertas. En **Run** el Chargeback es automático y el Unit Cost se mide en tiempo real. RetailCorp está en Crawl — y eso es el punto de partida normal. El valor está en el camino.

---

## 4. Parte 2 — Análisis de Costos y Alertas ⏱ 15 min

---

### Ejercicio 2.1 — Distribuir el presupuesto por equipo

> 💬 **[DOCENTE]** No hay una única respuesta correcta. Lo importante es que tenga criterio. Presenta esta distribución como referencia razonada.

| Equipo | Presupuesto mensual | Justificación |
|--------|:---:|---------------|
| E-Commerce | $16,000 | Mayor gasto actual: 4 web-prod + CloudFront + DataTransfer + RDS frontend |
| Data & Analytics | $11,000 | Instancias grandes (m5.8xlarge, p3.2xlarge) + S3 intensivo + ETL workers |
| Backend Core | $7,000 | APIs estables y bien etiquetadas. RDS del ERP asignado aquí |
| **TOTAL** | **$34,000** | ~$100 sobre el presupuesto de $33,900 — margen mínimo aceptable |

---

### Ejercicio 2.2 — Estrategia de Budgets

> 💬 **[DOCENTE]** Destaca: el Budget de E-Commerce tiene umbrales más bajos porque tiene picos de campaña impredecibles. Forecasted es siempre más valioso que Actual porque da tiempo de actuar.

| Equipo | Nombre Budget | Monto | Alerta 1 | Alerta 2 | Canal |
|--------|--------------|:---:|:---:|:---:|-------|
| E-Commerce | `budget-ecommerce-monthly` | $16,000 | 75% Actual | 90% Forecasted | SNS → email Javier + Slack #finops |
| Data & Analytics | `budget-data-monthly` | $11,000 | 80% Actual | 95% Forecasted | SNS → email Patricia + Slack #finops |
| Backend Core | `budget-backend-monthly` | $7,000 | 80% Actual | 95% Forecasted | SNS → email Carlos + Slack #finops |

---

### Ejercicio 2.3 — Calcular el Unit Cost actual

> 💬 **[DOCENTE]** El Unit Cost es la métrica que convierte FinOps de una función de costos a una función estratégica. Sin esto, el CEO ve "gasté más" o "gasté menos" sin contexto de negocio.

**a) Servicios de E-Commerce:**

```
Servicios identificados:
  EC2: i-001 + i-002 + i-003 + i-004 (web-prod) = 4 × $561 = $2,244
  EC2: i-005 (cache-redis-01)                    = $140
  EC2: i-006 (staging-env)                       = $140
  CloudFront (CDN del e-commerce):               ≈ $2,600
  Data Transfer (tráfico tienda online):         ≈ $2,000 (estimado)
  S3 assets-cdn:                                 ≈ $26

Gasto estimado E-Commerce mensual:   ≈ $7,150
```

**b) Unit Cost:**

```
Unit Cost = $7,150 / 380,000 transacciones = $0.0188 por transacción

Si el objetivo es $0.03/transacción:
  Gasto máximo = $0.03 × 380,000 = $11,400/mes
  → E-Commerce tiene margen: puede gastar hasta $11,400 y seguir
    cumpliendo el objetivo de negocio
```

> 💬 **[DOCENTE]** Aquí está el matiz clave: el Unit Cost de $0.0188 está por debajo del objetivo de $0.03. Eso significa que aunque el right-sizing que haremos en la Parte 3 reduzca el costo, el negocio ya está dentro del rango aceptable. El right-sizing bajará aún más el Unit Cost, que es una mejora adicional — no una corrección de emergencia.

**c) Por qué el Unit Cost supera al gasto total:**

> *El gasto total no da contexto. Si la factura sube de $33,900 a $40,000 el mes que viene, el CFO puede alarmar. Pero si las transacciones crecieron de 380,000 a 600,000, el Unit Cost baja de $0.019 a $0.013 — la arquitectura se volvió más eficiente. El CEO necesita ver la tendencia del Unit Cost, no la factura absoluta. FinOps convierte el gasto en cloud en una función del crecimiento del negocio.*

---

## 5. Parte 3 — Right-Sizing y Optimización de Uso ⏱ 15 min

---

### Ejercicio 3.1 — Análisis de right-sizing

> 💬 **[DOCENTE]** Regla de oro: el tipo recomendado debe poder manejar el doble del uso promedio para absorber picos. CPU 8% promedio → picos esperados ~20% → m5.large cubre con margen. Nunca se optimiza sin margen de seguridad.
>
> Nota sobre i-005 cache-redis-01: CPU 22% con RAM al 61%. Este es un caso de uso intensivo de memoria — NO se hace right-sizing aquí porque la RAM es el factor limitante. m5.xlarge con 16 GB a 61% de uso = ~10 GB usados. No hay margen seguro para bajar.

**a) Tabla de recomendaciones:**

| Instancia | Tipo actual | CPU | RAM | Tipo recomendado | Razonamiento | Actual/mes | Nuevo/mes | Ahorro/mes |
|-----------|-------------|:---:|:---:|-----------------|--------------|:---:|:---:|:---:|
| i-001 web-prod-01 | m5.4xlarge | 8% | 12% | m5.large | CPU 8% → pico ~20%, m5.large cubre. RAM 12% de 64GB = 7.7GB → m5.large (8GB) suficiente | $561 | $70 | **$491** |
| i-002 web-prod-02 | m5.4xlarge | 7% | 11% | m5.large | Mismo patrón que i-001 | $561 | $70 | **$491** |
| i-003 web-prod-03 | m5.4xlarge | 9% | 13% | m5.large | Mismo patrón. 4 web-prod idénticos sugieren sobreaprovisionamiento sistemático | $561 | $70 | **$491** |
| i-004 web-prod-04 | m5.4xlarge | 8% | 12% | m5.large | Mismo patrón | $561 | $70 | **$491** |
| i-005 cache-redis-01 | m5.xlarge | 22% | 61% | **Mantener** | RAM al 61% es el factor limitante. No se toca | $140 | $140 | $0 |
| i-006 staging-env | m5.xlarge | 2% | 4% | t3.medium | Staging no necesita m5. t3.medium cubre ampliamente | $140 | $30 | **$110** |
| i-012 data-processing | m5.8xlarge | 6% | 9% | m5.2xlarge | CPU 6% → pico ~15%. RAM 9% de 128GB = 11.5GB → m5.2xlarge (32GB) con buen margen | $1,121 | $280 | **$841** |
| i-013 ml-training | p3.2xlarge | 3% | 5% | Spot p3.2xlarge | GPU al 3% = desperdicio total. Migrar a Spot (ver ejercicio 3.2) | $2,234 | $670 | **$1,564** |
| i-017 dev-old | t3.medium | 0% | 1% | **Eliminar** | 0% CPU = instancia abandonada. Terminar de inmediato | $30 | $0 | **$30** |
| i-018 test-loadgen | t3.large | 1% | 2% | **Eliminar** | 1% CPU = también abandonada | $61 | $0 | **$61** |

**b) Impacto total:**

```
Ahorro mensual por right-sizing (incl. Spot i-013):   $4,570
Ahorro anual:                                         $54,840
% reducción sobre gasto EC2 instancias ($8,537):       53.5%
```

> 💬 **[DOCENTE]** El 53.5% de reducción en EC2 es el número impactante. Más de la mitad del gasto en servidores puede recuperarse sin cambiar una línea de código de negocio. Esto es exactamente lo que el CFO necesita escuchar para aprobar FinOps.

---

### Ejercicio 3.2 — Instancias Spot para i-012 e i-013

> 💬 **[DOCENTE]** Spot = capacidad sobrante de AWS vendida con descuento de hasta 90%. AWS puede recuperarla con 2 minutos de aviso. Solo sirve para cargas tolerantes a interrupciones.

**a) Cálculo Spot (ya incluido en right-sizing de i-013, aquí el detalle completo):**

```
i-012 data-processing (migrado a m5.2xlarge primero):
  On-Demand m5.2xlarge:    $280 /mes
  Spot m5.2xlarge (70%):   $280 × 0.30 = $84 /mes
  Ahorro adicional Spot:   $196 /mes
  (Los $841 del right-sizing ya contaban migrar a m5.2xlarge)

i-013 ml-training (p3.2xlarge Spot):
  On-Demand actual:        $2,234 /mes
  Spot p3.2xlarge (70%):   $2,234 × 0.30 = $670 /mes
  Ahorro:                  $1,564 /mes  ← ya incluido en right-sizing
```

**b) Cargas SÍ/NO aptas para Spot:**

| ¿Spot? | Carga | Justificación |
|:---:|---|---|
| ✅ SÍ | `data-processing` (i-012) | Pipeline batch nocturno — si AWS interrumpe, el job se reinicia desde el checkpoint |
| ✅ SÍ | `ml-training` (i-013) | Entrenamiento ML con checkpoints — tolerante a interrupciones por diseño |
| ❌ NO | `web-prod-01/02/03/04` | Si AWS recupera la instancia, la tienda online cae en producción |
| ❌ NO | `api-backend-01/02/03` | APIs síncronas en tiempo real — la interrupción afecta a usuarios activos |

---

## 6. Parte 4 — Recursos Huérfanos y Quick Wins ⏱ 10 min

---

### Ejercicio 4.1 — Valorizar recursos huérfanos

> 💬 **[DOCENTE]** Los recursos huérfanos son la fruta más baja del árbol FinOps. Sin riesgo técnico, impacto inmediato, perfectos para demostrar valor en el día 1.

**a) Tabla de impacto:**

| Recurso | Costo/mes | Costo anual | Acción | Urgencia |
|---------|:---------:|:-----------:|--------|:--------:|
| eip-prod-legacy | $3.65 | $43.80 | Liberar inmediatamente | 🔴 Alta |
| eip-test-01 | $3.65 | $43.80 | Liberar inmediatamente | 🔴 Alta |
| eip-data-old | $3.65 | $43.80 | Liberar inmediatamente | 🔴 Alta |
| vol-backup-2023 (500 GB) | $50.00 | $600.00 | Snapshot final → eliminar | 🔴 Alta |
| vol-staging-old (200 GB) | $20.00 | $240.00 | Identificar dueño en 72h, si nadie reclama → eliminar | 🟡 Media |
| lb-ecommerce-v1 | $16.20 | $194.40 | Eliminar (0 targets en 45 días = seguro) | 🔴 Alta |
| **TOTAL** | **$97.15** | **$1,165.80** | | |

**b) Ahorro inmediato:**

```
Ahorro mensual:   $97.15
Ahorro anual:     $1,165.80
```

> 💬 **[DOCENTE]** $1,165 parece poco comparado con $54,840 del right-sizing. Pero hay tres argumentos: (1) es el ahorro del día 1, sin ningún riesgo. (2) En empresas grandes este patrón multiplica por cientos. (3) Demuestra a la organización que FinOps entrega valor rápido — genera el apoyo político para las iniciativas más grandes.

**c) Proceso para i-017 dev-old e i-018 test-loadgen:**

> 1. Consultar CloudTrail para identificar el último usuario que accedió y su equipo.
> 2. Enviar notificación formal: *"Esta instancia será apagada en 72 horas sin respuesta."*
> 3. Si no hay respuesta → **apagar** (no eliminar). Esperar 7 días más.
> 4. Si nadie la reclama → **eliminar** y crear snapshot de los volúmenes como respaldo por 30 días.
> 5. Documentar como política formal: toda instancia sin actividad por 30 días entra en proceso de revisión automática.

---

### Ejercicio 4.2 — Apagado programado staging

> 💬 **[DOCENTE]** Implementar esto en AWS es trivial con EventBridge Scheduler: dos reglas (start/stop). 30 minutos de trabajo de un ingeniero. Uno de los mejores ROI de FinOps.

```
Horas de uso real por semana:   10h/día × 5 días = 50 horas
Horas de uso real por mes:      50 × 4.33 semanas = 216.5 horas
% de tiempo necesario:          216.5 / 730 = 29.7% ≈ 30%

Costo actual (730 horas):       $0.192/h × 730 = $140.16/mes
Costo con apagado (216.5 h):    $0.192/h × 216.5 = $41.57/mes
Ahorro mensual:                 $98.59/mes ≈ $99
Ahorro anual:                   $1,186/año
```

---

## 7. Parte 5 — Estrategia de Storage y Ciclo de Vida ⏱ 10 min

---

### Ejercicio 5.1 — Políticas de ciclo de vida

> 💬 **[DOCENTE]** El patrón de acceso determina la política. Recuerda que Standard-IA cobra por retrieval ($0.01/GB). Si el acceso es frecuente, el costo de retrieval puede superar el ahorro. Siempre hay que analizar el patrón antes de decidir.

**`retailcorp-sales-data` (4.2 TB, acceso 15 días)**
```
Patrón: consultas diarias los primeros 30 días, luego solo auditorías.
  Día  0:   Standard           (reportes diarios activos)
  Día 30:   Standard-IA        (acceso infrecuente, solo auditorías)
  Día 180:  Glacier Instant    (cumplimiento, acceso muy ocasional)
  Día 730:  Expiración         (2 años: política de retención de datos)
```

**`retailcorp-logs-app` (8.7 TB, acceso 94 días)**
```
Patrón: debugging activo primeros 30 días, luego solo incidentes.
  Día  0:   Standard           (debugging activo)
  Día 30:   Standard-IA        (solo incidentes)
  Día 90:   Glacier Flexible   (archivo, recuperación en horas OK)
  Día 365:  Expiración         (logs de más de 1 año sin valor operativo)
```

**`retailcorp-backups` (12.1 TB, acceso 380 días)**
```
Patrón: casi nunca accedidos, solo en disaster recovery.
  Día  0:   Standard           (backup reciente, posible restauración rápida)
  Día  7:   Glacier Instant    (backup verificado → archivo inmediato)
  Día 90:   Glacier Flexible   (largo plazo, recuperación en horas aceptable)
  Día 1095: Expiración         (3 años: política corporativa de retención)
```

**`retailcorp-assets-cdn` (1.1 TB, acceso 3 días)**
```
¿Necesita lifecycle? NO.
Justificación: acceso cada 3 días = altamente frecuente (imágenes, CSS, JS).
Standard-IA cobra $0.01/GB por retrieval. Con acceso tan frecuente,
el costo de retrieval superaría el ahorro de storage. Standard es correcto.
```

> 💬 **[DOCENTE]** El caso de assets-cdn es el más importante pedagógicamente: lifecycle no siempre es la respuesta. Primero analizar el patrón de acceso. Si el retrieval es frecuente, quedarse en Standard es la decisión correcta de FinOps.

---

### Ejercicio 5.2 — Calcular el ahorro de storage

**a) Estado actual (todo en Standard):**

```
retailcorp-sales-data:    4,301 GB × $0.023 = $  98.92/mes
retailcorp-logs-app:      8,909 GB × $0.023 = $ 204.91/mes
retailcorp-backups:      12,390 GB × $0.023 = $ 284.97/mes
retailcorp-assets-cdn:    1,126 GB × $0.023 = $  25.90/mes
Gasto S3 storage actual:                      $ 614.70/mes
```

**b) Con lifecycle aplicado:**

| Bucket | Standard | Standard-IA | Glacier | Costo/mes |
|--------|:---:|:---:|:---:|:---:|
| sales-data | 500 GB ($11.50) | 2,000 GB ($25.00) | 1,801 GB ($7.20) | **$43.70** |
| logs-app | 300 GB ($6.90) | 1,500 GB ($18.75) | 7,109 GB ($25.59) | **$51.24** |
| backups | 100 GB ($2.30) | 0 GB ($0) | 12,290 GB ($44.24) | **$46.54** |
| assets-cdn | 1,126 GB ($25.90) | 0 GB | 0 GB | **$25.90** |
| **Total** | | | | **$167.38** |

```
Gasto S3 storage actual:          $ 614.70/mes
Gasto S3 storage con lifecycle:   $ 167.38/mes
Ahorro mensual:                   $ 447.32/mes
Ahorro anual:                     $ 5,367.84/año ≈ $5,368
```

> 💬 **[DOCENTE]** La factura de S3 total es $7,800/mes. El lifecycle solo optimiza el storage ($614 → $167). La diferencia ($7,186) corresponde a requests, Data Transfer y otros cargos de S3 que no cambian con el lifecycle.

---

## 8. Parte 6 — Hoja de Ruta FinOps ⏱ 10 min

---

### Ejercicio 6.1 — Consolidar el ahorro total

> 💬 **[DOCENTE]** Pide a los alumnos que sumen sus propios números antes de mostrar el consolidado. El ejercicio es ver si llegaron a resultados similares.

| Iniciativa | Ahorro/mes | Ahorro/año | Esfuerzo | Plazo |
|-----------|:---:|:---:|:---:|:---:|
| Right-sizing EC2 (incl. Spot) | $4,570 | $54,840 | Medio | 2-4 semanas |
| Recursos huérfanos | $97 | $1,166 | Bajo | Esta semana |
| Apagado staging | $99 | $1,187 | Bajo | Esta semana |
| Lifecycle S3 | $447 | $5,368 | Bajo | 1 semana |
| **TOTAL** | **$5,213** | **$62,556** | | |

```
Factura actual:                        $33,900/mes
Ahorro total identificado:             $ 5,213/mes
Factura proyectada post-optimización:  $28,687/mes
Reducción:                              15.4%
```

> 💬 **[DOCENTE]** $62,556 de ahorro anual identificado en 75 minutos de análisis. Sin cambiar una sola línea de código de negocio. Solo ajustando los recursos a su uso real. Esto es FinOps.

---

### Ejercicio 6.2 — Priorizar la hoja de ruta

> 💬 **[DOCENTE]** La matriz Impacto vs. Esfuerzo es la herramienta estándar. Pide a los alumnos que propongan el orden antes de mostrar este.

```
Prioridad 1 — ESTA SEMANA:
  Iniciativa: Recursos huérfanos + apagado staging
  Justificación: Riesgo cero, impacto inmediato ($196/mes), demuestra
  valor rápido al CFO. Son las acciones que generan confianza
  organizacional para aprobar el resto del plan.

Prioridad 2 — SEMANA 2:
  Iniciativa: Right-sizing i-013 ml-training a Spot
  Justificación: $1,564/mes de ahorro, solo requiere validar checkpoints
  en los jobs de ML. Esfuerzo técnico bajo, impacto muy alto.

Prioridad 3 — MES 1:
  Iniciativa: Right-sizing web-prod-01/02/03/04 (m5.4xlarge → m5.large)
  Justificación: $1,964/mes de ahorro. Requiere ventana de mantenimiento,
  pruebas de carga y aprobación del CTO para producción.

Prioridad 4 — MES 1 (en paralelo):
  Iniciativa: Lifecycle S3
  Justificación: $447/mes, configuración en consola sin impacto en apps.
  Se puede hacer en paralelo con el right-sizing.

Prioridad 5 — TRIMESTRE 1 (base estructural):
  Iniciativa: Taxonomía de tags + SCPs obligatorias
  Justificación: Sin esto, en 6 meses volvemos al punto de partida.
  Es la base que habilita el Chargeback y el Unit Cost real por equipo.
```

---

### Ejercicio 6.3 — Mensaje al CFO

> 💬 **[DOCENTE]** Pide 2-3 alumnos que lean su versión antes de mostrar esta. El punto clave: no jerga técnica, solo números y fechas.

**Respuesta modelo:**

> *"Diego, analizamos la factura de $33,900 del mes pasado. El problema principal es que el 79.5% del gasto en servidores no tiene responsable asignado — el equipo de TI no puede decirte a quién cobrarle ese gasto porque el 72% de las instancias no están etiquetadas correctamente.*
>
> *Identificamos $5,213 de ahorro mensual ($62,556 anuales) sin cambiar ninguna funcionalidad del negocio: servidores que usan menos del 10% de su capacidad, 6 recursos abandonados que nadie apagó, y datos de backup en almacenamiento caro que podrían estar en clases 10 veces más baratas.*
>
> *Esta semana ejecutamos las acciones de cero riesgo: $196/mes de ahorro inmediato. En el mes 1 completamos el right-sizing de servidores con aprobación del CTO. Solo necesito tu autorización para implementar el etiquetado obligatorio, que en 30 días nos dará visibilidad total de a quién le estamos cobrando cada dólar."*

---

## 🗺️ Mapa de Conceptos Cubiertos

| Ejercicio | Fase FinOps | Concepto |
|-----------|-------------|---------|
| 1.1 Tagging Compliance | Informar | Asignación de costos, Chargeback |
| 1.2 Taxonomía de tags | Informar | Gobernanza, SCPs |
| 1.3 Madurez FinOps | Transversal | Modelo Crawl-Walk-Run |
| 2.1-2.2 Budgets | Operar | Alertas proactivas, Forecasted |
| 2.3 Unit Cost | Informar | Unit Economics |
| 3.1 Right-sizing | Optimizar (Uso) | Sobreaprovisionamiento |
| 3.2 Spot Instances | Optimizar (Tarifas) | Modelos de descuento AWS |
| 4.1 Huérfanos | Informar + Operar | Quick wins |
| 4.2 Apagado programado | Optimizar (Uso) | Gasto ocioso |
| 5.1-5.2 S3 Lifecycle | Optimizar (Uso) | Tiering automático |
| 6.1-6.3 Hoja de ruta | Transversal | Valor de negocio, priorización |

---

*Laboratorio elaborado por el docente Aldo Trucios · Programa Arquitectura de Soluciones Multinube · UTEC Posgrado*
*Módulo 8 · Sesión 2 — Cloud Financial Management (FinOps) · **VERSIÓN DOCENTE***
