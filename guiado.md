# 🧪 Laboratorio Guiado — Cloud Financial Management (FinOps)
## Módulo 8 · Sesión 2 | Arquitectura de Soluciones Multinube — UTEC
### 📋 VERSIÓN DOCENTE — Respuestas incluidas

> **Docente:** Aldo Trucios
> **Duración:** 1 hora 15 minutos
> **Modalidad:** Guiado — el docente explica cada ejercicio y revela las respuestas progresivamente. Los alumnos siguen con el documento en blanco (LAB_PROPUESTO_M08S02).

---

## 📋 Tabla de Contenidos

1. [Objetivos del Laboratorio](#1-objetivos-del-laboratorio)
2. [Contexto del Caso — RetailCorp Perú](#2-contexto-del-caso--retailcorp-perú)
3. [Parte 1 — Diagnóstico de Visibilidad y Tagging](#3-parte-1--diagnóstico-de-visibilidad-y-tagging-15-min)
4. [Parte 2 — Análisis de Costos y Alertas](#4-parte-2--análisis-de-costos-y-alertas-15-min)
5. [Parte 3 — Right-Sizing y Optimización de Uso](#5-parte-3--right-sizing-y-optimización-de-uso-15-min)
6. [Parte 4 — Recursos Huérfanos y Quick Wins](#6-parte-4--recursos-huérfanos-y-quick-wins-10-min)
7. [Parte 5 — Estrategia de Storage y Ciclo de Vida](#7-parte-5--estrategia-de-storage-y-ciclo-de-vida-10-min)
8. [Parte 6 — Hoja de Ruta FinOps](#8-parte-6--hoja-de-ruta-finops-10-min)

---

## 1. Objetivos del Laboratorio

Al completar este laboratorio el estudiante será capaz de:

- ✅ Diagnosticar el nivel de madurez FinOps de una organización usando el modelo Crawl-Walk-Run
- ✅ Calcular el Tagging Compliance e identificar su impacto en el Chargeback
- ✅ Diseñar una estrategia de alertas proactivas con AWS Budgets y SNS
- ✅ Evaluar recomendaciones de right-sizing y cuantificar su impacto anual
- ✅ Identificar y valorizar recursos huérfanos como quick wins de optimización
- ✅ Diseñar una política de ciclo de vida S3 y calcular su ahorro
- ✅ Proponer una hoja de ruta FinOps priorizada con impacto en dólares

---

## 2. Contexto del Caso — RetailCorp Perú

**RetailCorp Perú** es una empresa de retail con presencia en 12 ciudades. Tiene 3 equipos que usan AWS:

| Equipo | Descripción | Responsable |
|--------|-------------|-------------|
| **E-Commerce** | Plataforma de ventas online, picos en campañas | Javier (CTO) |
| **Data & Analytics** | Pipelines de datos, reportes de ventas | Patricia (Data Lead) |
| **Backend Core** | APIs internas, bases de datos, ERP | Carlos (Infra Lead) |

El CFO acaba de recibir la factura de AWS del mes pasado: **$47,300**. Nadie puede explicar con precisión a qué corresponde ese gasto. El CEO pide un diagnóstico y un plan de acción.

### 2.1 — Factura AWS del mes

| Servicio AWS | Gasto mensual | % del total |
|--------------|:---:|:---:|
| Amazon EC2 | $21,400 | 45.2% |
| Amazon RDS | $9,200 | 19.5% |
| Amazon S3 | $7,800 | 16.5% |
| AWS Data Transfer | $4,100 | 8.7% |
| Amazon CloudFront | $2,600 | 5.5% |
| Otros servicios | $2,200 | 4.6% |
| **TOTAL** | **$47,300** | **100%** |

### 2.2 — Inventario de recursos EC2

| ID | Nombre | Tipo | vCPU | RAM | CPU prom. | RAM prom. | Equipo | Tags completos |
|----|--------|------|:---:|:---:|:---:|:---:|--------|:---:|
| i-001 | web-prod-01 | m5.4xlarge | 16 | 64 GB | 8% | 12% | E-Commerce | No |
| i-002 | web-prod-02 | m5.4xlarge | 16 | 64 GB | 7% | 11% | E-Commerce | No |
| i-003 | api-backend-01 | m5.2xlarge | 8 | 32 GB | 41% | 55% | Backend Core | Sí |
| i-004 | api-backend-02 | m5.2xlarge | 8 | 32 GB | 38% | 52% | Backend Core | Sí |
| i-005 | data-processing | m5.8xlarge | 32 | 128 GB | 6% | 9% | Data & Analytics | No |
| i-006 | ml-training | p3.2xlarge | 8 | 61 GB | 3% | 5% | Data & Analytics | No |
| i-007 | staging-env | m5.xlarge | 4 | 16 GB | 2% | 4% | E-Commerce | No |
| i-008 | dev-old | t3.medium | 2 | 4 GB | 0% | 1% | Desconocido | No |

### 2.3 — Inventario de recursos huérfanos

| Recurso | Tipo | Detalle | Costo/mes |
|---------|------|---------|:---------:|
| eip-prod-legacy | Elastic IP | Sin asociar | $3.65 |
| eip-test-01 | Elastic IP | Sin asociar | $3.65 |
| eip-data-old | Elastic IP | Sin asociar | $3.65 |
| vol-backup-2023 | EBS gp2 500 GB | Sin adjuntar, último snapshot ene-2023 | $50.00 |
| vol-staging-old | EBS gp2 200 GB | Sin adjuntar, sin dueño | $20.00 |
| lb-ecommerce-v1 | Load Balancer | 0 targets en 45 días | $16.20 |

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

> 💬 **[DOCENTE]** Explica primero: sin tags, la factura de AWS es una caja negra. El CFO no sabe a quién cobrarle. El objetivo de FinOps es asignar el 100% del gasto a un responsable. Eso no es posible sin etiquetas.

**a) Tabla completada:**

| Recurso | Tags completos | Gasto/mes |
|---------|:---:|:---:|
| i-001 web-prod-01 (m5.4xlarge) | ❌ No | $561 |
| i-002 web-prod-02 (m5.4xlarge) | ❌ No | $561 |
| i-003 api-backend-01 (m5.2xlarge) | ✅ Sí | $384 |
| i-004 api-backend-02 (m5.2xlarge) | ✅ Sí | $384 |
| i-005 data-processing (m5.8xlarge) | ❌ No | $1,121 |
| i-006 ml-training (p3.2xlarge) | ❌ No | $2,234 |
| i-007 staging-env (m5.xlarge) | ❌ No | $140 |
| i-008 dev-old (t3.medium) | ❌ No | $30 |

**b) Cálculo del Tagging Compliance:**

```
Total de recursos EC2:                          8
Recursos con tagging completo:                  2  (i-003 e i-004)
Tagging Compliance:                             25%

Gasto EC2 total mensual:                      $ 5,415
Gasto EC2 asignable (con tags):               $   768  (i-003 + i-004)
Gasto EC2 NO asignable (sin tags):            $ 4,647
% del gasto EC2 invisible al CFO:               85.8%
```

> 💬 **[DOCENTE]** Señala el impacto: el CFO ve $21,400 en EC2 pero solo puede justificar $768 (3.6% del total EC2). El 85.8% restante es invisible para el Chargeback. Esto es exactamente el problema que FinOps viene a resolver.

**c) Respuesta para el CFO:**

> *"En este momento solo puedo asignarte con certeza $768 de los $21,400 de EC2, correspondientes a los servidores api-backend del equipo de Backend Core que sí tienen etiquetas. Los $4,647 restantes no tienen dueño asignado porque el 75% de las instancias carecen de las etiquetas obligatorias. Para responder tu pregunta correctamente necesito implementar una taxonomía de tags en los próximos 7 días y aplicarla retroactivamente a todos los recursos existentes."*

---

### Ejercicio 1.2 — Diseñar la taxonomía de tags

> 💬 **[DOCENTE]** La clave no es tener muchos tags, es tener los tags correctos y aplicarlos de forma obligatoria. Muestra que AWS Service Control Policies (SCPs) y AWS Config Rules permiten hacer el tagging obligatorio técnicamente.

| Clave | Valores válidos | ¿Qué pasa si falta? |
|-------|-----------------|---------------------|
| `Environment` | `prod`, `staging`, `dev`, `qa` | El recurso no se puede crear (SCP bloqueante) |
| `CostCenter` | `CC-ECOM`, `CC-DATA`, `CC-BACK`, `CC-SHARED` | El recurso no se puede crear (SCP bloqueante) |
| `ApplicationName` | Nombre del sistema en minúsculas (ej: `ecommerce-web`, `erp-core`) | El recurso no se puede crear (SCP bloqueante) |
| `OwnerEmail` | Email corporativo válido `@retailcorp.pe` | Alerta automática vía AWS Config |
| `Team` | `ecommerce`, `data-analytics`, `backend-core`, `shared` | Alerta automática vía AWS Config |

> 💬 **[DOCENTE]** Explica la diferencia entre SCP bloqueante (impide crear el recurso) y alerta de Config (permite crearlo pero notifica). Para los 3 primeros tags usamos bloqueo total porque son los que habilitan el Chargeback. Para los últimos 2 usamos alerta porque son más propensos a errores de tipeo.

---

### Ejercicio 1.3 — Madurez FinOps

> 💬 **[DOCENTE]** Presenta el modelo Crawl-Walk-Run. Pide a los alumnos que voten antes de revelar la respuesta.

**Fase actual de RetailCorp: CRAWL** 🐛

| Evidencia del caso | Indicador de fase |
|--------------------|-------------------|
| Nadie puede explicar los $47,300 de la factura | Sin visibilidad básica → Crawl |
| 75% de los recursos EC2 sin tags | Tagging Compliance < 50% → Crawl |
| No hay Budgets ni alertas configuradas | Sin alertas proactivas → Crawl |
| Recursos huérfanos activos (EIPs, EBS, LB) | Sin proceso de limpieza → Crawl |
| Sin asignación de costos por equipo | Sin Showback ni Chargeback → Crawl |

> 💬 **[DOCENTE]** Una empresa en fase **Walk** ya tiene Showback funcionando y alertas configuradas. Una empresa en **Run** tiene Chargeback automático integrado en el pipeline de CI/CD. RetailCorp está claramente en Crawl — y eso es normal al inicio. El valor de FinOps está en el camino de madurez.

---

## 4. Parte 2 — Análisis de Costos y Alertas ⏱ 15 min

---

### Ejercicio 2.1 — Distribuir el presupuesto por equipo

> 💬 **[DOCENTE]** No hay una única respuesta correcta para la distribución. Lo importante es que tenga criterio y justificación. Presenta la siguiente como referencia razonada.

| Equipo | Presupuesto mensual | Justificación |
|--------|:---:|---------------|
| E-Commerce | $22,000 | Mayor gasto actual (web-prod x2 + CloudFront + DataTransfer de tienda online). Tiene picos en campañas. |
| Data & Analytics | $14,000 | Instancias grandes (m5.8xlarge, p3.2xlarge) + S3 intensivo. Pipeline batch predecible. |
| Backend Core | $9,000 | APIs estables, bien etiquetadas. RDS compartido asignado aquí por ser owner del ERP. |
| **TOTAL** | **$45,000** | Reducción de $2,300 vs. gasto actual, alcanzable con las optimizaciones de las partes 3-5. |

---

### Ejercicio 2.2 — Estrategia de Budgets

> 💬 **[DOCENTE]** Explica la diferencia entre alerta Actual (reactiva) y Forecasted (proactiva). La Forecasted es la más valiosa porque te da tiempo de actuar antes de que ocurra el problema.

| Equipo | Nombre Budget | Monto | Alerta 1 | Alerta 2 | Canal |
|--------|--------------|:---:|:---:|:---:|-------|
| E-Commerce | `budget-ecommerce-monthly` | $22,000 | 75% Actual | 90% Forecasted | SNS → Slack #finops + email Javier |
| Data & Analytics | `budget-data-monthly` | $14,000 | 80% Actual | 95% Forecasted | SNS → Slack #finops + email Patricia |
| Backend Core | `budget-backend-monthly` | $9,000 | 80% Actual | 95% Forecasted | SNS → Slack #finops + email Carlos |

> 💬 **[DOCENTE]** Nota que E-Commerce tiene umbrales más bajos (75%/90%) porque tiene picos de campaña impredecibles. Los otros dos tienen cargas más estables y pueden tolerar umbrales más altos.

---

### Ejercicio 2.3 — Calcular el Unit Cost actual

> 💬 **[DOCENTE]** El Unit Cost es la métrica que conecta el gasto en cloud con el negocio. Sin esto, FinOps es solo una función de costos. Con esto, es una función estratégica.

**a) Servicios de E-Commerce:**

```
Servicios identificados:
  - EC2: web-prod-01 + web-prod-02 + staging-env = $561 + $561 + $140 = $1,262
  - CloudFront: $2,600 (distribución de contenido del e-commerce)
  - Data Transfer: ~$2,000 (estimado, tráfico de tienda online)
  - S3 assets-cdn: 1,126 GB × $0.023 = ~$26

Gasto estimado E-Commerce mensual:   ≈ $5,888
```

**b) Unit Cost:**

```
Unit Cost = Gasto del equipo / Transacciones del mes
Unit Cost = $5,888 / 380,000 transacciones
Unit Cost actual = $0.0155 por transacción

Si el objetivo es $0.03/transacción:
  Gasto máximo = $0.03 × 380,000 = $11,400/mes
  → E-Commerce tiene margen para crecer sin optimizar urgentemente
```

> 💬 **[DOCENTE]** Aquí viene el punto clave: el Unit Cost de $0.0155 está MUY por debajo del objetivo de $0.03. Eso significa que E-Commerce es eficiente en términos de negocio, aunque sus instancias estén sobreaprovisionadas. El right-sizing que haremos en la Parte 3 bajará aún más ese número. Este es el argumento para que el CTO entienda que optimizar no es "cortar" sino "hacerlo mejor".

**c) Unit Cost vs. gasto total:**

> *El gasto total mensual ($47,300) no le dice nada al CEO sobre si estamos gastando bien o mal. El Unit Cost ($0.0155/transacción) sí lo hace: si el mes que viene el gasto sube a $55,000 pero las transacciones crecen a 600,000, el Unit Cost baja a $0.0092 — eso es una mejora, no un problema. El CEO necesita ver la tendencia del Unit Cost, no la factura absoluta.*

---

## 5. Parte 3 — Right-Sizing y Optimización de Uso ⏱ 15 min

---

### Ejercicio 3.1 — Análisis de right-sizing

> 💬 **[DOCENTE]** Regla práctica: el tipo recomendado debe poder manejar el doble del uso promedio (para absorber picos). CPU 8% promedio → picos esperados ~20% → m5.large alcanza. Nunca se optimiza sin margen de seguridad.

**a) Tabla de recomendaciones:**

| Instancia | Tipo actual | CPU | RAM | Tipo recomendado | Razonamiento | Actual/mes | Nuevo/mes | Ahorro/mes |
|-----------|-------------|:---:|:---:|-----------------|--------------|:---:|:---:|:---:|
| i-001 web-prod-01 | m5.4xlarge | 8% | 12% | m5.large | CPU 8% → pico ~20%, m5.large cubre holgado | $561 | $70 | $491 |
| i-002 web-prod-02 | m5.4xlarge | 7% | 11% | m5.large | Mismo patrón que i-001 | $561 | $70 | $491 |
| i-005 data-processing | m5.8xlarge | 6% | 9% | m5.xlarge | CPU 6% → pico ~15%, 12 GB RAM usados → m5.xlarge (16 GB) | $1,121 | $140 | $981 |
| i-006 ml-training | p3.2xlarge | 3% | 5% | Apagar / Spot | 3% de uso en GPU = desperdicio total. Migrar a Spot o apagar cuando no hay jobs | $2,234 | $0* | $2,234 |
| i-007 staging-env | m5.xlarge | 2% | 4% | t3.medium | Staging no necesita m5. t3.medium cubre ampliamente | $140 | $30 | $110 |
| i-008 dev-old | t3.medium | 0% | 1% | **Eliminar** | 0% CPU = instancia abandonada. Terminar inmediatamente | $30 | $0 | $30 |

*i-006 se analiza con Spot en el ejercicio 3.2

**b) Impacto total:**

```
Ahorro mensual por right-sizing:    $4,337
Ahorro anual:                       $52,044
% reducción sobre EC2 actual:       20.3% del total EC2
```

> 💬 **[DOCENTE]** Señala i-006: una instancia p3.2xlarge con GPU al 3% de uso cuesta $2,234/mes. Eso son $26,808 al año literalmente desperdiciados. Este es el tipo de hallazgo que hace que un CFO apruebe FinOps en 5 minutos.

---

### Ejercicio 3.2 — Instancias Spot

> 💬 **[DOCENTE]** Spot = capacidad sobrante de AWS. AWS puede recuperarla con 2 minutos de aviso. Por eso solo sirve para cargas tolerantes a interrupciones. El descuento compensa ese riesgo.

**a) Cálculo Spot:**

```
i-005 data-processing:
  On-Demand actual:   $1,121 /mes
  Spot (70% desc.):   $1,121 × 0.30 = $336 /mes
  Ahorro mensual:     $785

i-006 ml-training:
  On-Demand actual:   $2,234 /mes
  Spot (70% desc.):   $2,234 × 0.30 = $670 /mes
  Ahorro mensual:     $1,564

Ahorro total Spot:    $2,349 /mes  →  $28,188 /año
```

**b) Cargas SÍ/NO aptas para Spot:**

| ¿Aplica Spot? | Carga de trabajo en RetailCorp |
|:---:|---|
| ✅ SÍ | `data-processing`: pipeline batch de reportes nocturnos (se puede reiniciar) |
| ✅ SÍ | `ml-training`: entrenamiento de modelos ML (jobs idempotentes con checkpoints) |
| ❌ NO | `web-prod-01/02`: si AWS recupera la instancia, la tienda online cae |
| ❌ NO | `api-backend-01/02`: APIs síncronas con clientes en tiempo real |

---

## 6. Parte 4 — Recursos Huérfanos y Quick Wins ⏱ 10 min

---

### Ejercicio 4.1 — Valorizar recursos huérfanos

> 💬 **[DOCENTE]** Los recursos huérfanos son el quick win más fácil de FinOps. No requieren rediseño de arquitectura, no tienen riesgo técnico, y el ahorro es inmediato. Son perfectos para demostrar valor en la primera semana.

**a) Tabla de impacto:**

| Recurso | Costo/mes | Costo anual | Acción | Urgencia |
|---------|:---------:|:-----------:|--------|:--------:|
| eip-prod-legacy | $3.65 | $43.80 | Liberar inmediatamente | 🔴 Alta |
| eip-test-01 | $3.65 | $43.80 | Liberar inmediatamente | 🔴 Alta |
| eip-data-old | $3.65 | $43.80 | Liberar inmediatamente | 🔴 Alta |
| vol-backup-2023 (500 GB) | $50.00 | $600.00 | Crear snapshot final y eliminar | 🔴 Alta |
| vol-staging-old (200 GB) | $20.00 | $240.00 | Identificar dueño, si nadie reclama en 72h → eliminar | 🟡 Media |
| lb-ecommerce-v1 | $16.20 | $194.40 | Eliminar (0 targets en 45 días) | 🔴 Alta |
| **TOTAL** | **$97.15** | **$1,165.80** | | |

**b) Ahorro inmediato:**

```
Ahorro mensual:   $97.15
Ahorro anual:     $1,165.80
```

> 💬 **[DOCENTE]** Parecen montos pequeños, pero hay tres puntos importantes: (1) son ahorro en el día 1, sin ningún riesgo. (2) En empresas grandes con cientos de proyectos, este patrón multiplica. (3) Demuestran a la organización que FinOps entrega valor rápido, lo que genera apoyo para las iniciativas más grandes.

**c) Proceso para i-008 dev-old:**

> 1. Identificar el último usuario que hizo login (via CloudTrail) y el equipo al que pertenecía.
> 2. Enviar notificación al equipo: "Esta instancia será apagada en 72 horas si no hay respuesta."
> 3. Si no hay respuesta → **apagar** (no eliminar aún) y esperar 7 días más.
> 4. Si en 7 días nadie la reclama → **eliminar** y crear snapshot de los volúmenes como respaldo por 30 días adicionales.
> 5. Documentar el proceso como política formal del equipo de FinOps.

---

### Ejercicio 4.2 — Apagado programado staging

> 💬 **[DOCENTE]** El apagado programado es uno de los ahorros más fáciles y consistentes de FinOps. Lunes a viernes, 9am-7pm = 50 horas/semana. El resto del tiempo no tiene valor.

```
Horas de uso real por semana:     10 horas/día × 5 días = 50 horas
Horas de uso real por mes:        50 × 4.33 semanas = 216.5 horas
Horas totales del mes:            730 horas
% de tiempo necesario:            216.5 / 730 = 29.7%  ≈ 30%

Costo actual (730 horas):         $0.192/hora × 730 = $140.16 /mes
Costo con apagado (216.5 horas):  $0.192/hora × 216.5 = $41.57 /mes
Ahorro mensual:                   $98.59 /mes  ≈ $99
Ahorro anual:                     $1,186.08 /año  ≈ $1,187
```

> 💬 **[DOCENTE]** Implementar esto en AWS es trivial: AWS Instance Scheduler o simplemente un EventBridge Scheduler con dos reglas (start/stop). 30 minutos de trabajo, $1,187 de ahorro anual. ROI inmediato.

---

## 7. Parte 5 — Estrategia de Storage y Ciclo de Vida ⏱ 10 min

---

### Ejercicio 5.1 — Políticas de ciclo de vida

> 💬 **[DOCENTE]** El patrón de acceso determina la política. Acceso frecuente → Standard. Acceso infrecuente → Standard-IA. Archivo → Glacier. Lo importante es que la transición sea automática — sin intervención humana.

**Bucket: `retailcorp-sales-data` (4.2 TB, acceso 15 días)**

```
Patrón: datos de ventas consultados frecuentemente los primeros 30 días,
        luego solo para auditorías, raramente después de 1 año.

Política:
  Día  0:   Standard              (consultas diarias de reportes)
  Día 30:   Standard-IA           (acceso infrecuente, solo auditorías)
  Día 180:  Glacier Instant       (acceso muy ocasional, cumplimiento)
  Día 730:  Expiración            (datos de más de 2 años, política de retención)
```

**Bucket: `retailcorp-logs-app` (8.7 TB, acceso 94 días)**

```
Patrón: logs consultados para debugging los primeros 30 días,
        luego casi nunca. Retención legal requerida: 1 año.

Política:
  Día  0:   Standard              (debugging activo)
  Día 30:   Standard-IA           (acceso solo para incidentes)
  Día 90:   Glacier Flexible      (archivo, acceso en horas aceptable)
  Día 365:  Expiración            (logs de más de 1 año, sin valor)
```

**Bucket: `retailcorp-backups` (12.1 TB, acceso 380 días)**

```
Patrón: backups casi nunca accedidos (solo en caso de recuperación de desastres).
        Retención: 3 años por política corporativa.

Política:
  Día  0:   Standard              (backup reciente, posible restauración rápida)
  Día  7:   Glacier Instant       (backup ya verificado, archivo inmediato)
  Día 90:   Glacier Flexible      (archivo de largo plazo, recuperación en horas OK)
  Día 1095: Expiración            (3 años cumplidos, política de retención)
```

**Bucket: `retailcorp-assets-cdn` (1.1 TB, acceso 3 días)**

```
¿Necesita lifecycle? NO.
Justificación: acceso promedio de 3 días significa que los assets son
consultados constantemente (imágenes, CSS, JS del sitio web).
Moverlos a Standard-IA generaría un costo de recuperación mayor
al ahorro de storage. Standard es correcto aquí.
```

> 💬 **[DOCENTE]** Este último punto es importante: no todo necesita lifecycle. Standard-IA cobra por cada lectura ($0.01/GB recuperado). Si los datos se acceden frecuentemente, el costo de retrieval puede superar el ahorro de storage. Siempre hay que analizar el patrón antes de decidir.

---

### Ejercicio 5.2 — Calcular el ahorro de storage

**a) Estado actual (todo en Standard):**

```
retailcorp-sales-data:    4,301 GB × $0.023 = $  98.92 /mes
retailcorp-logs-app:      8,909 GB × $0.023 = $ 204.91 /mes
retailcorp-backups:      12,390 GB × $0.023 = $ 284.97 /mes
retailcorp-assets-cdn:    1,126 GB × $0.023 = $  25.90 /mes

Gasto S3 storage actual:                      $ 614.70 /mes
```

> 💬 **[DOCENTE]** Nota: la factura de S3 total es $7,800/mes. La diferencia con $614 corresponde a requests, Data Transfer y otros cargos de S3 que no cambian con el lifecycle. El lifecycle solo optimiza el costo de almacenamiento.

**b) Con políticas de lifecycle aplicadas:**

| Bucket | Standard | Standard-IA | Glacier | Costo/mes |
|--------|:---:|:---:|:---:|:---:|
| sales-data | 500 GB | 2,000 GB | 1,801 GB | $11.50 + $25.00 + $7.20 = **$43.70** |
| logs-app | 300 GB | 1,500 GB | 7,109 GB | $6.90 + $18.75 + $25.59 = **$51.24** |
| backups | 100 GB | 0 GB | 12,290 GB | $2.30 + $0 + $44.24 = **$46.54** |
| assets-cdn | 1,126 GB | 0 GB | 0 GB | **$25.90** (sin cambio) |
| **Total** | | | | **$167.38** |

```
Gasto S3 storage actual:          $ 614.70 /mes
Gasto S3 storage con lifecycle:   $ 167.38 /mes
Ahorro mensual:                   $ 447.32 /mes
Ahorro anual:                     $ 5,367.84 /año  ≈ $5,368
```

---

## 8. Parte 6 — Hoja de Ruta FinOps ⏱ 10 min

---

### Ejercicio 6.1 — Consolidar el ahorro total

> 💬 **[DOCENTE]** Este es el momento más impactante del lab. Pide a los alumnos que sumen sus números antes de mostrar el consolidado.

| Iniciativa | Ahorro/mes | Ahorro/año | Esfuerzo | Plazo |
|-----------|:---:|:---:|:---:|:---:|
| Right-sizing EC2 | $4,337 | $52,044 | Medio | 2-4 semanas |
| Migración a Spot | $2,349 | $28,188 | Medio | 1-2 semanas |
| Recursos huérfanos | $97 | $1,166 | Bajo | Esta semana |
| Apagado staging | $99 | $1,187 | Bajo | Esta semana |
| Lifecycle S3 | $447 | $5,368 | Bajo | 1 semana |
| **TOTAL** | **$7,329** | **$87,953** | | |

```
Factura actual:                       $ 47,300 /mes
Ahorro total identificado:            $  7,329 /mes
Factura proyectada post-optimización: $ 39,971 /mes
Reducción porcentual:                   15.5%
```

> 💬 **[DOCENTE]** $87,953 de ahorro anual identificado en 75 minutos de análisis. Sin cambiar una sola línea de código de negocio. Solo ajustando los recursos a su uso real. Esto es FinOps.

---

### Ejercicio 6.2 — Priorizar la hoja de ruta

> 💬 **[DOCENTE]** La matriz Impacto vs. Esfuerzo es la herramienta estándar de priorización. Alto impacto + bajo esfuerzo = primera semana. Alto impacto + alto esfuerzo = planificar con cuidado.

```
Prioridad 1 — ESTA SEMANA (alto impacto, bajo esfuerzo):
  Iniciativa: Eliminar recursos huérfanos + apagado staging
  Justificación: Riesgo cero, impacto inmediato, demuestra valor rápido al CFO.
  Ahorro inmediato: $196/mes sin ningún riesgo técnico.

Prioridad 2 — SEMANA 2 (alto impacto, esfuerzo medio):
  Iniciativa: Migración i-005 e i-006 a Spot Instances
  Justificación: $2,349/mes de ahorro. Solo requiere validar que los jobs
  tienen checkpoints y toleran reinicio. Esfuerzo técnico bajo-medio.

Prioridad 3 — MES 1 (mayor impacto, requiere planificación):
  Iniciativa: Right-sizing EC2 (i-001, i-002, i-005, i-007, i-008)
  Justificación: $4,337/mes pero requiere ventana de mantenimiento,
  pruebas de rendimiento y aprobación del CTO para producción.

Prioridad 4 — MES 1 (impacto medio, bajo esfuerzo):
  Iniciativa: Lifecycle S3
  Justificación: $447/mes, configuración en consola sin impacto en aplicaciones.
  Puede hacerse en paralelo con el right-sizing.

Prioridad 5 — TRIMESTRE 1 (base estructural):
  Iniciativa: Implementar taxonomía de tags + SCPs
  Justificación: Sin esto, en 6 meses volvemos al mismo problema.
  Es la base que habilita el Chargeback y el Unit Cost por equipo.
```

---

### Ejercicio 6.3 — Mensaje al CFO

> 💬 **[DOCENTE]** Pide a 2-3 alumnos que lean su versión antes de mostrar esta. El punto clave: el CFO no quiere jerga técnica. Quiere números y fechas.

**Respuesta modelo:**

> *"Diego, analizamos la factura de $47,300 de AWS del mes pasado. El problema principal es que el 86% del gasto en servidores no tiene responsable asignado porque falta etiquetado en los recursos — el equipo de TI no puede decirte a quién cobrarle ese gasto.*
>
> *Identificamos $7,329 de ahorro mensual ($87,953 anuales) sin cambiar ninguna funcionalidad del negocio: servidores sobredimensionados que usan el 6% de su capacidad, 6 recursos abandonados que nadie apagó, y datos en almacenamiento caro que podrían estar en clases más económicas.*
>
> *Esta semana podemos ejecutar las acciones de bajo riesgo ($196/mes de ahorro inmediato). En el mes 1 completamos el right-sizing con aprobación del CTO. Necesito tu autorización para implementar una política de etiquetado obligatorio que nos dará visibilidad total del gasto en 30 días."*

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
*Módulo 8 · Sesión 2 — Cloud Financial Management (FinOps) · VERSIÓN DOCENTE*
