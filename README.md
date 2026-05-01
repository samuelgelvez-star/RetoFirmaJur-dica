
# Reto Firma Jurídica

Base de datos relacional para la gestión de asuntos legales, clientes y procuradores de una firma jurídica.

---

## Estructura del repositorio

```
/diagramas
  └── DER_firma_juridica.png          ← Diagrama Entidad-Relación (drawio)
/modelo_logico
  └── drawSQL-image-export-2026-05-01.jpg  ← Modelo lógico relacional (drawSQL)
README.md
```

---

## 1. Interpretación del problema

La firma jurídica necesita un sistema para gestionar tres elementos centrales de su operación:

- **Clientes**: personas identificadas por su DNI que contratan los servicios de la firma. De cada uno se requiere nombre, dirección, fecha de nacimiento y contacto.
- **Asuntos legales**: los expedientes o casos que la firma gestiona. Cada asunto tiene un número de expediente único, fechas de inicio y finalización, y un estado que indica si está en proceso, cerrado o suspendido.
- **Procuradores**: los profesionales encargados de llevar los asuntos. Se registra su DNI, nombre, apellidos, número de colegiado, casos ganados y contacto.

El reto principal del diseño está en que **un asunto puede requerir varios procuradores**, y **un procurador puede gestionar varios asuntos** al mismo tiempo, lo que genera una relación de muchos a muchos (N:M) que debe resolverse correctamente en el modelo relacional.

---

## 2. Identificación de entidades y atributos

### CLIENTES
| Atributo | Tipo | Descripción |
|---|---|---|
| `DNI` (PK) | VARCHAR(255) | Identificador único del cliente |
| `nombre` | VARCHAR(255) | Nombre completo |
| `direccion` | VARCHAR(255) | Dirección de residencia |
| `fecha_nacimiento` | DATE | Fecha de nacimiento |
| `contacto` | BIGINT | Número de contacto telefónico |

### ASUNTOS LEGALES
| Atributo | Tipo | Descripción |
|---|---|---|
| `numero_expediente` (PK) | VARCHAR(255) | Identificador único del caso |
| `fecha_inicio` | DATE | Fecha de apertura del expediente |
| `fecha_finalizacion` | DATE | Fecha de cierre del caso |
| `estado` | VARCHAR(255) | Estado actual del asunto |
| `DNI_cliente(FK)` | VARCHAR(255) | Cliente al que pertenece el asunto |

### PROCURADOR
| Atributo | Tipo | Descripción |
|---|---|---|
| `DNI` (PK) | VARCHAR(255) | Identificador único del procurador |
| `nombre` | VARCHAR(255) | Nombre del procurador |
| `apellidos` | VARCHAR(255) | Apellidos del procurador |
| `numero_colegiado` | VARCHAR(255) | Número de colegiación profesional |
| `casos ganados` | INT | Historial de casos ganados |
| `contacto` | BIGINT | Número de contacto telefónico |

### ASUNTO_PROCURADOR *(tabla intermedia)*
| Atributo | Tipo | Descripción |
|---|---|---|
| `numero_expediente(FK)` | VARCHAR(255) | Referencia al asunto legal |
| `DNI_procurador(FK)` | VARCHAR(255) | Referencia al procurador |

---

## 3. Justificación de relaciones y cardinalidades

Las relaciones fueron identificadas a partir del análisis del enunciado y se representan en el Diagrama Entidad-Relación con notación de Chen (ver `/diagramas/DER_firma_juridica.png`).

### Relación CLIENTES — ASUNTOS LEGALES: **1 a M**

Un cliente puede tener **muchos** asuntos legales a lo largo del tiempo, pero cada asunto pertenece a **un único** cliente. Por eso la cardinalidad es **1:M** (uno a muchos).

```
CLIENTES (1) ─── Tiene ─── (M) ASUNTOS LEGALES
```

Esta relación se implementa colocando el `DNI` del cliente como clave foránea (`DNI_cliente(FK)`) dentro de la tabla `ASUNTOS LEGALES`.

### Relación ASUNTOS LEGALES — PROCURADORES: **N a M**

Un asunto legal puede ser llevado por **varios procuradores** (cuando el caso es complejo), y un procurador puede encargarse de **varios asuntos** al mismo tiempo. La cardinalidad es **N:M** (muchos a muchos).

```
ASUNTOS LEGALES (N) ─── Lleva ─── (M) PROCURADORES
                              ↕
                      ASUNTO_PROCURADOR
                       (tabla intermedia)
```

Esta relación no puede representarse directamente en SQL, por lo que se resuelve con una tabla intermedia (ver sección 5).

---

## 4. Transformación al modelo relacional

El paso del modelo conceptual (DER) al modelo lógico relacional sigue tres reglas fundamentales:

**Regla 1 — Cada entidad se convierte en una tabla.**
Las entidades `CLIENTES`, `ASUNTOS LEGALES` y `PROCURADOR` se convierten directamente en tablas con sus atributos como columnas. El atributo identificador de cada entidad se convierte en la clave primaria (PK).

**Regla 2 — La relación 1:M se resuelve con una clave foránea.**
La relación entre `CLIENTES` y `ASUNTOS LEGALES` se implementa añadiendo el `DNI` del cliente como clave foránea en la tabla `ASUNTOS LEGALES`. No se crea ninguna tabla adicional.

**Regla 3 — La relación N:M se resuelve con una tabla intermedia.**
La relación entre `ASUNTOS LEGALES` y `PROCURADOR` genera la tabla `ASUNTO_PROCURADOR`, que contiene las claves foráneas de ambas tablas. Esta tabla representa cada asignación de un procurador a un asunto.

El modelo lógico resultante puede verse en `/diagramas/drawSQL-image-export-2026-05-01.jpg`.

---

## 5. Decisiones tomadas

**¿Por qué se creó la tabla `ASUNTO_PROCURADOR`?**

En SQL no es posible representar directamente una relación N:M entre dos tablas. Si se intentara guardar todos los procuradores de un asunto en una sola fila, se violaría la Primera Forma Normal (1FN) y las consultas serían ineficientes. La tabla intermedia `ASUNTO_PROCURADOR` permite registrar cada par asunto-procurador como una fila independiente, manteniendo la integridad referencial desde ambos lados.

**¿Por qué `VARCHAR(255)` en casi todos los campos de texto?**

Se eligió `VARCHAR(255)` como tipo seguro y flexible para campos de texto cuya longitud máxima no está especificada en el enunciado. Esto evita truncamientos inesperados y es una práctica común en diseño inicial de bases de datos.


## Diagrama conceptual (DER — Notación Chen)

![Diagrama Entidad-Relación](./diagramas/DER_firma_juridica.png)

## Modelo lógico relacional

![Modelo relacional drawSQL](./diagramas/drawSQL-image-export-2026-05-01.jpg)

---
