## Objetivo

Extender el modelo (básico + opcional) para cubrir:

- **Acceso**: suscripciones y compras por curso.
- **Aprendizaje**: progreso por video y por artículo.
- **Métricas**: visualizaciones de videos (y agregados por curso).
- **Descubrimiento**: etiquetas (tags) en curso y video.

Manteniendo **1FN–3FN**, FKs y reglas claras de opcionalidad en el diagrama.

---

## Leyenda del diagrama (misma convención)

- 🟦 **Principales/actividad**: `usuario`, `suscripcion`, `compra`, `progreso_*`, `video_vista`
- 🟩 **Catálogos**: `plan_suscripcion`, `etiqueta`
- 🟪 **Intermedias**: `curso_etiqueta`, `video_etiqueta` (+ `curso_autor`)

---

## Tablas añadidas

### Principales / actividad ( 🟦 )

- **usuario**: `id`, `email`, `nombre`, `creado_en`, `actualizado_en`.
- **suscripcion**: **PK compuesta** `(usuario_id, plan_id)`, `inicio_en`, `fin_en`, `estado`.
  - FKs: `usuario_id → usuario.id (CASCADE)`, `plan_id → plan_suscripcion.id` .
- **compra**: `id`, `usuario_id (FK)`, `curso_id (FK)`, `precio_centavos`, `moneda`, `comprado_en`.
- **progreso_video**: **PK compuesta** `(usuario_id, video_id)`, `estado`, `posicion_seg`, `actualizado_en`.
  - FKs: `usuario_id → usuario.id`, `video_id → video.id`.
- **progreso_articulo**: **PK compuesta** `(usuario_id, articulo_id)`, `estado`, `actualizado_en`.
  - FKs: `usuario_id → usuario.id`, `articulo_id → articulo.id`.
  - Para artículos: estado manual (pendiente/en_progreso/completado).
- **video_vista**: `id`, `video_id (FK)`, `usuario_id (FK)`, `visto_en`, `origen`.
  - Índices: `(video_id, visto_en)`, `(usuario_id, visto_en)`.

### Catálogos ( 🟩 )

- **plan_suscripcion**: `id`, `nombre`, `precio_mensual_centavos`, `moneda`, `descripcion`.
- **etiqueta**: `id`, `nombre`, `slug`.

### Intermedias ( 🟪 )

- **curso_etiqueta**: **PK compuesta** `(curso_id, etiqueta_id)`.
- **video_etiqueta**: **PK compuesta** `(video_id, etiqueta_id)`.

> Se mantiene la intermedia curso_autor con PK compuesta (curso_id, autor_id).

---

## Relaciones y opcionalidad (lo reflejado en el diagrama)

- `usuario o< suscripcion` y `plan_suscripcion o< suscripcion` (cada suscripción **debe** tener usuario y plan; usuario/plan **pueden** no tener suscripciones)
- `usuario o< compra` y `curso o< compra`
- `usuario o< progreso_video` y `video |< progreso_video`
- `usuario o< progreso_articulo` y `articulo |< progreso_articulo`
- `video o< video_vista` (y `usuario o< video_vista` si la vista es autenticada; `usuario_id`)
- `curso o< curso_etiqueta >o etiqueta` (N:M)
- `video o< video_etiqueta >o etiqueta` (N:M)

Convención visual en crow’s feet:

- **Barra `|`** = participación **obligatoria** (p. ej., todo progreso **debe** apuntar a una lección).
- **Círculo `o`** = participación **opcional** (p. ej., un usuario **puede** no tener suscripciones).

---

## Normalización e integridad

- **1FN**: datos atómicos; eventos de vistas separados por fila; progresos por usuario–recurso.
- **2FN**: PKs compuestas donde la identidad es el par (usuario, recurso).
- **3FN**: catálogos (`plan_suscripcion`, `etiqueta`) evitan literales repetidos.

---

## Decisiones destacadas (para el revisor)

- **`suscripcion` con PK compuesta `(usuario_id, plan_id)`**: no manejamos histórico de renovaciones; refuerza unicidad lógica y evita `id` artificial.
- **Progresos con PK compuesta** `(usuario_id, video_id)` / `(usuario_id, articulo_id)`: “una fila por usuario y recurso”.
- **Opción A para artículos**: simplicidad (estado manual). Se documenta como decisión consciente y extensible.

---

## 📁 Entrega

```
/desafio/
 ├── elearning-diagrama-fisico.drawio
 ├── elearning-diagrama-fisico.png
 └── README.md
```
