# Modelo Físico del Portal E-Learning

## **Objetivo del proyecto**

El presente documento describe el **modelo de datos físico** para un portal de e-learning orientado al mundo de la programación.

El objetivo es definir la estructura relacional necesaria para almacenar información sobre **cursos, videos, artículos, autores y temáticas**, aplicando principios de **modelado relacional**, **patrones de diseño de bases de datos** y **formas normales**.

El modelo fue implementado en formato de diagrama físico mediante la herramienta [draw.io](https://app.diagrams.net/).

---

## **Resumen del dominio**

La aplicación consiste en un portal donde los usuarios pueden navegar cursos de programación.

Cada curso está compuesto por **videos y artículos**, tiene **uno o varios autores**, y los videos se clasifican por **temáticas** técnicas.

### **Sitemap funcional**

```
Inicio → Curso → Lección (Video o Artículo)
                         ↓
                       Autor
```

---

## **Entidades principales**

| Tabla         | Descripción                                                               |
| ------------- | ------------------------------------------------------------------------- |
| `curso`       | Contiene la información principal de cada curso del portal.               |
| `video`       | Representa las lecciones en video asociadas a un curso.                   |
| `articulo`    | Representa los artículos complementarios a los videos dentro del curso.   |
| `autor`       | Almacena la información de los autores (nombre, biografía, avatar, etc.). |
| `tematica`    | Define las categorías o áreas técnicas de los videos.                     |
| `curso_autor` | Tabla intermedia que relaciona cursos con autores (relación N:M).         |

---

## **Relaciones y cardinalidades**

| Relación                   | Tipo | Descripción                                                                                                                                           |
| -------------------------- | ---- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `curso (1) — (N) video`    | 1:N  | Un curso puede tener varios videos; cada video pertenece a un solo curso.                                                                             |
| `curso (1) — (N) articulo` | 1:N  | Un curso puede tener varios artículos; cada artículo pertenece a un solo curso.                                                                       |
| `autor (1) — (N) video`    | 1:N  | Cada video tiene un autor; un autor puede haber creado varios videos.                                                                                 |
| `tematica (1) — (N) video` | 1:N  | Cada video pertenece a una sola temática.                                                                                                             |
| `curso (N) — (M) autor`    | N:M  | Un curso puede tener varios autores y un autor puede participar en varios cursos. La relación se gestiona mediante la tabla intermedia `curso_autor`. |

---

## **Estructura resumida de las tablas**

### `curso`

- id (PK)
- titulo
- resumen
- slug
- creado_en
- actualizado_en

### `video`

- id (PK)
- curso_id (FK → curso.id)
- autor_id (FK → autor.id)
- tematica_id (FK → tematica.id)
- titulo
- duracion_seg
- s3_recurso_id
- cms_recurso_id
- posicion
- creado_en

### `articulo`

- id (PK)
- curso_id (FK → curso.id)
- titulo
- cms_recurso_id
- posicion
- creado_en

### `autor`

- id (PK)
- nombre
- biografia
- avatar_url
- slug

### `tematica`

- id (PK)
- nombre
- slug

### `curso_autor`

- curso_id (PK, FK → curso.id)
- autor_id (PK, FK → autor.id)
- rol_en_curso
- creado_en

> Se usa una clave primaria compuesta (curso_id, autor_id).

---

## **Patrones aplicados**

| Patrón                         | Descripción                                                                        |
| ------------------------------ | ---------------------------------------------------------------------------------- |
| **Uno a muchos (1:N)**         | Usado en `curso → video`, `curso → articulo`, `autor → video`, `tematica → video`. |
| **Muchos a muchos (N:M)**      | Implementado mediante la tabla intermedia `curso_autor`.                           |
| **Entidad débil / intermedia** | `curso_autor` depende de `curso` y `autor`; no tiene sentido sin ellos.            |
| **Catálogo / lookup table**    | `tematica` funciona como catálogo de áreas técnicas.                               |
| **Integridad referencial**     | Todas las relaciones están definidas con FKs explícitas                            |

---

## **Normalización aplicada**

### **Primera Forma Normal (1FN)**

- Todos los atributos son **atómicos** (sin listas ni campos multivaluados).
- Los recursos externos (videos y artículos) se referencian mediante **identificadores**, no contenido binario.

### **Segunda Forma Normal (2FN)**

- Todas las tablas tienen **claves primarias simples** (salvo la intermedia).

### **Tercera Forma Normal (3FN)**

- No existen **dependencias transitivas**: ningún atributo no-clave depende de otro atributo no-clave.
- Ejemplo: `video` almacena `tematica_id`, no el nombre de la temática.

---

## **Decisiones de diseño relevantes**

1. Los **artículos** no tienen autor ni temática propios, ya que heredan esos datos del curso al que pertenecen.

   Esto simplifica el modelo y evita redundancia.

2. Se almacenan solo los **identificadores** (`s3_recurso_id`, `cms_recurso_id`) de los contenidos multimedia, no los archivos.
3. La relación **curso–autor** se implementa con tabla intermedia para evitar campos multivaluados (manteniendo 1FN y 3FN).
4. Las reglas de negocio como “todo curso debe tener al menos un autor” se validarían **a nivel aplicación**.

---

## 🧾 **Conclusión**

El modelo resultante cumple con:

- **Estructura relacional sólida y normalizada (1FN–3FN).**
- **Aplicación de patrones** de relación comunes en modelado físico (1:N, N:M, entidad débil, catálogo).
- **Integridad referencial y consistencia** mediante claves foráneas y restricciones.
- **Extensibilidad**, ya que permite ampliaciones futuras (jerarquía de temáticas, visibilidad público/privado, usuarios, progreso, etc.).

---

## 🎨 **Leyenda del diagrama**

Para facilitar la lectura visual, se aplicó una codificación por color según el tipo de entidad:

| Color     | Tipo de entidad                      | Función en el modelo                                                                           |
| --------- | ------------------------------------ | ---------------------------------------------------------------------------------------------- |
| 🟦 Azul   | **Entidad principal**                | Representa los elementos centrales del dominio (por ejemplo: `curso`, `video`, `articulo`).    |
| 🟩 Verde  | **Catálogo o entidad de referencia** | Contiene valores de clasificación o información contextual (por ejemplo: `autor`, `tematica`). |
| 🟪 Morado | **Entidad intermedia o débil**       | Gestiona relaciones N:M entre entidades principales (por ejemplo: `curso_autor`).              |

> Esta convención facilita la comprensión jerárquica del modelo y permite identificar visualmente la función de cada tabla.

---

📁 **Estructura final de entrega**

```
/basico/
 ├── elearning-diagrama-fisico.drawio
 ├── elearning-diagrama-fisico.png

/opcional/
 ├── elearning-diagrama-fisico.drawio
 ├── elearning-diagrama-fisico.png
 ├──README.md

/desafio/
 ├── elearning-diagrama-fisico.drawio
 ├── elearning-diagrama-fisico.png
 └── README.md

README.md
```
