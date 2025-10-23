## **Objetivo**

Este documento describe las **extensiones y mejoras** aplicadas al modelo físico básico del portal E-Learning.

La versión opcional introduce **nuevas capacidades estructurales** sin romper la compatibilidad ni los principios de normalización establecidos en la versión base.

Los objetivos de esta extensión son:

1. Incorporar una **jerarquía de temáticas** (autorelación).
2. Añadir **niveles de acceso o visibilidad** para cursos y videos.
3. Mantener la **integridad referencial y normalización** (1FN–3FN).
4. Demostrar la aplicación de patrones de modelado relacional.

---

## **Cambios respecto a la versión básica**

| Elemento                     | Versión básica                                              | Versión opcional                                                                                          |
| ---------------------------- | ----------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| **Jerarquía de temáticas**   | Solo un nivel de temáticas (`tematica` simple).             | Se añade `tematica_padre_id` (FK a `tematica.id`) → permite subniveles como _Frontend → React → Testing_. |
| **Visibilidad de contenido** | No contemplada. Todos los cursos/videos se asumen públicos. | Se añade la tabla `nivel_acceso` y FKs en `curso` y `video` (`nivel_acceso_id`).                          |
| **Nuevas relaciones**        | 5 relaciones (1:N y N:M).                                   | +3 relaciones nuevas (`nivel_acceso → curso`, `nivel_acceso → video`, `tematica (self-FK)`).              |
| **Nuevos patrones**          | 1:N, N:M, entidad débil, catálogo.                          | + Autorelación (self-reference) y catálogo compartido para política de acceso.                            |

---

## **Nuevas tablas y campos**

### **Tabla: nivel_acceso**

Define los niveles de visibilidad permitidos para cursos y videos.

| Campo         | Descripción                              |
| ------------- | ---------------------------------------- |
| `id` (PK)     | Identificador del nivel de acceso.       |
| `nombre`      | Ej.: “Público”, “Suscriptores”, “Mixto”. |
| `descripcion` | Detalle opcional.                        |

**Relaciones:**

- `nivel_acceso (1)` → `curso (N)`
- `nivel_acceso (1)` → `video (N)`

> Permite que los cursos y videos definan su nivel de acceso de forma centralizada y consistente.

---

### **Campo nuevo en tematica**

- `tematica_padre_id` (FK → tematica.id)

**Relación:**

`tematica (1)` → `tematica (N)` (self-FK)

> Soporta jerarquías arbitrarias:
>
> “Frontend” → “React” → “Testing React”.

---

### **Campos nuevos en curso y video**

| Tabla   | Campo             | FK                  | Descripción                                   |
| ------- | ----------------- | ------------------- | --------------------------------------------- |
| `curso` | `nivel_acceso_id` | → `nivel_acceso.id` | Nivel de visibilidad general del curso.       |
| `video` | `nivel_acceso_id` | → `nivel_acceso.id` | Permite granularidad dentro de cursos mixtos. |

---

## **Patrones de diseño aplicados**

| Patrón                           | Aplicación                             | Descripción                                                                  |
| -------------------------------- | -------------------------------------- | ---------------------------------------------------------------------------- |
| **Autorelación (Self-FK)**       | `tematica_padre_id → tematica.id`      | Permite representar jerarquías (categorías dentro de categorías).            |
| **Catálogo compartido**          | `nivel_acceso`                         | Centraliza los posibles niveles de acceso en una sola tabla.                 |
| **Herencia funcional de reglas** | `nivel_acceso_id` en `curso` y `video` | Los cursos heredan su política de acceso; los videos pueden sobreescribirla. |
| **Reutilización de estructuras** | Extiende sin alterar tablas existentes | Los nuevos campos no rompen compatibilidad con el modelo anterior.           |

---

## **Normalización y consistencia**

| Forma Normal | Explicación                                                                                                   |
| ------------ | ------------------------------------------------------------------------------------------------------------- |
| **1FN**      | Los nuevos atributos (`nivel_acceso_id`, `tematica_padre_id`) son atómicos.                                   |
| **2FN**      | No hay dependencias parciales: los nuevos campos dependen de la PK completa.                                  |
| **3FN**      | Se evita repetir nombres de niveles de acceso en cada curso/video; se normaliza mediante FK a `nivel_acceso`. |

---

## **Nuevas relaciones (resumen visual)**

```
tematica (1) ───< (N) tematica_hija
nivel_acceso (1) ───< (N) curso
nivel_acceso (1) ───< (N) video
```

Estas relaciones se añaden a las ya existentes:

- curso → video
- curso → articulo
- autor → video
- tematica → video
- curso ↔ autor (N:M vía curso_autor)

---

## **Decisiones de diseño**

1. **Autorelación en temáticas:**

   Se eligirá `ON DELETE SET NULL` para no eliminar temáticas hijas al borrar el padre.

   Esto evitará la pérdida de clasificación y mantendrá flexibilidad para reestructurar la jerarquía.

2. **Tabla `nivel_acceso`:**

   En lugar de un booleano (`es_publico`), se usó un **catálogo explícito**.

   Permitirá representar múltiples estados (público, privado, mixto, beta, etc.) sin alterar el esquema.

3. **Extensión no disruptiva:**

   Las nuevas FKs serán **NULLABLE**, lo que permitirá mantener datos existentes del modelo básico.

4. **Compatibilidad descendente:**

   Toda consulta y relación del modelo base sigue siendo válida.

---

## **Leyenda de colores (sin cambios)**

| Color     | Tipo de entidad            | Ejemplo                             |
| --------- | -------------------------- | ----------------------------------- |
| 🟦 Azul   | Entidades principales      | `curso`, `video`, `articulo`        |
| 🟩 Verde  | Catálogos / referencias    | `autor`, `tematica`, `nivel_acceso` |
| 🟪 Morado | Intermedias / relacionales | `curso_autor`                       |

---

## **Conclusión**

La versión opcional amplía el modelo básico incorporando **jerarquías y políticas de acceso**, demostrando:

- Capacidad de **evolución sin pérdida de normalización**.
- Uso de **patrones relacionales avanzados** (self-reference, catálogo compartido).
- Mantenimiento de la **coherencia e integridad referencial**.

Estas mejoras preparan la base para futuras fases del proyecto, como:

- Modelar **usuarios y suscripciones** (desafío).
- Implementar **consultas jerárquicas**
- Gestionar **niveles de visibilidad dinámica** a nivel de aplicación.

---

**Estructura de entrega**
