# 🎓 Modelo de Datos Relacional: Portal eLearning (Programación)

Este documento presenta el modelado relacional (basado en tablas y relaciones) para un portal de eLearning. El objetivo principal del modelo es evitar redundancias para que el dato no esté repetido en varios sitios y no existan inconsistencias al actualizarlo. Asimismo, el modelo garantiza la integridad referencial, asegurando que cada conexión entre tablas sea sólida y que no existan registros apuntando a datos inexistentes.

### Alcance de este documento: Parte obligatoria + Parte opcional + Desafío.

---

## 📦 Resumen de Tablas

| #   | Tabla             | Sección                | Descripción                                                              |
| --- | ----------------- | ---------------------- | ------------------------------------------------------------------------ |
| 1   | `Courses`         | Obligatoria            | Tabla central. Contenedor de vídeos y artículos.                         |
| 2   | `Videos`          | Obligatoria            | Lecciones en formato vídeo, con autor y categoría propios.               |
| 3   | `Articles`        | Obligatoria            | Lecciones en formato artículo, simétricas a `Videos`.                    |
| 4   | `Authors`         | Obligatoria            | Información biográfica de los instructores.                              |
| 5   | `Categories`      | Obligatoria / Opcional | Temáticas de clasificación. Jerárquica a partir de la parte opcional.    |
| 6   | `Courses_authors` | Obligatoria            | Tabla intermedia N:M entre `Courses` y `Authors`.                        |
| 7   | `Social_networks` | Obligatoria            | Enlaces a redes sociales del autor (basado en mockup).                   |
| 8   | `Users`           | Desafío                | Usuarios de la plataforma.                                               |
| 9   | `Subscriptions`   | Desafío                | Suscripciones activas de un usuario.                                     |
| 10  | `Users_courses`   | Desafío                | Tabla intermedia: cursos comprados individualmente por un usuario.       |
| 11  | `Payments`        | Desafío                | Registro de pagos, referenciado desde `Subscriptions` y `Users_courses`. |
| 12  | `Tags`            | Desafío                | Etiquetas libres para búsqueda rápida.                                   |
| 13  | `TagsVideos`      | Desafío                | Tabla intermedia N:M entre `Videos` y `Tags`.                            |
| 14  | `TagsCourses`     | Desafío                | Tabla intermedia N:M entre `Courses` y `Tags`.                           |

---

## 🛠️ Detalle del Esquema

### 1. Tabla: `Courses`

Es el eje del portal. A diferencia del modelo documental (que usa _extended reference_ embebiendo autor/categoría), aquí el vínculo con autores se resuelve mediante tabla intermedia, ya que un curso puede tener **varios** autores.

| Campo         | Función relacional   | Descripción / Notas                                                                                              |
| ------------- | -------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `id`          | PK                   | Identificador único del curso.                                                                                   |
| `nombre`      | —                    | Título del curso.                                                                                                |
| `created_at`  | —                    | Fecha de creación. Usado para ordenar "cursos nuevos" en Home.                                                   |
| `category_id` | FK → `Categories.id` | Necesario para resolver el breadcrumb jerárquico ("Front End >> React >> Testing");                              |
| `is_public`   | —                    | Un curso puede ser 100% público.                                                                                 |
| `total_views` | —                    | **Desafío.** Contador cacheado de visualizaciones totales de los vídeos del curso (ver Justificación más abajo). |

---

### 2. Tabla: `Videos`

| Campo         | Función relacional             | Descripción / Notas                                                      |
| ------------- | ------------------------------ | ------------------------------------------------------------------------ |
| `id`          | PK, Not Null                   | Identificador único del vídeo.                                           |
| `name`        | —                              | Título del vídeo.                                                        |
| `course_id`   | FK → `Courses.id`, Not Null    | Curso al que pertenece (1:N, sin compartición entre cursos).             |
| `category_id` | FK → `Categories.id`, Not Null | Un vídeo pertenece a una sola temática.                                  |
| `author_id`   | FK → `Authors.id`, Not Null    | Un vídeo tiene un único autor (1:N, FK en el lado "N").                  |
| `cms_id`      | —                              | Referencia al contenido/metadata en el headless CMS.                     |
| `s3_key`      | —                              | Referencia al archivo de vídeo en S3.                                    |
| `is_public`   | —                              | Marca si este vídeo puntual es de acceso libre dentro de un curso mixto. |
| `views_count` | —                              | **Desafío.** Contador simple, incrementado en cada reproducción.         |

---

### 3. Tabla: `Articles`

Estructuralmente simétrica a `Videos`.

| Campo         | Función relacional             | Descripción / Notas                                      |
| ------------- | ------------------------------ | -------------------------------------------------------- |
| `id`          | PK, Not Null                   | Identificador único del artículo.                        |
| `name`        | —                              | Título del artículo.                                     |
| `course_id`   | FK → `Courses.id`, Not Null    | Artículo asociado a un solo curso.                       |
| `category_id` | FK → `Categories.id`, Not Null | Aplicado por simetría con `Video`.                       |
| `author_id`   | FK → `Authors.id`, Not Null    | Aplicado por simetría con `Video`.                       |
| `cms_id`      | —                              | Referencia al cuerpo del artículo en el CMS.             |
| `s3_key`      | —                              | Referencia a recursos multimedia embebidos, si aplica.   |
| `is_public`   | —                              | **Parte opcional.** Misma lógica que `Videos.is_public`. |

---

### 4. Tabla: `Authors`

Se mantiene como tabla independiente, referenciada tanto desde `Videos`/`Articles` (autoría individual, 1:N) como desde `Courses_authors` (colaboración por curso, N:M).

| Campo     | Función relacional | Descripción / Notas                              |
| --------- | ------------------ | ------------------------------------------------ |
| `id`      | PK, Not Null       | ID único del autor.                              |
| `name`    | —                  | Nombre.                                          |
| `surname` | —                  | Apellido.                                        |
| `bio`     | —                  | Biografía profesional, para la página del autor. |

---

### 5. Tabla: `Categories`

Se añade auto-referencia para modelar jerarquía con parent_id

| Campo       | Función relacional              | Descripción / Notas                                                                                     |
| ----------- | ------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `id`        | PK, Not Null                    | Identificador de la categoría.                                                                          |
| `name`      | —                               | Nombre (ej. "Frontend", "Devops").                                                                      |
| `parent_id` | FK → `Categories.id` (nullable) | Auto-referencia para construir la jerarquía (Front End >> React >> Testing). `NULL` en categorías raíz. |

---

### 6. Tabla intermedia: `Courses_authors`

Resuelve la relación N:M entre `Courses` y `Authors` (un curso puede tener varios autores; un autor puede participar en varios cursos).

| Campo       | Función relacional              | Descripción / Notas     |
| ----------- | ------------------------------- | ----------------------- |
| `course_id` | PK compuesta, FK → `Courses.id` | Identificador del curso |
| `author_id` | PK compuesta, FK → `Authors.id` | Identificador del autor |

---

### 7. Tabla: `Social_networks`

He incluido esta tabla debido a que en los mockups aparece esta posibilidad: la página de biografía del autor muestra enlaces a sus redes sociales. Se modela como relación 1:N (un autor puede tener varias redes: Twitter, LinkedIn, GitHub, etc.).

| Campo       | Función relacional | Descripción / Notas                                 |
| ----------- | ------------------ | --------------------------------------------------- |
| `id`        | PK                 | Identificador                                       |
| `platform`  | —                  | Nombre de la red social (ej. "GitHub", "LinkedIn"). |
| `url`       | —                  | Enlace al perfil.                                   |
| `author_id` | FK → `Authors.id`  | Identificador del autor                             |

---

### 8. Tabla: `Users`

| Campo     | Función relacional | Descripción / Notas              |
| --------- | ------------------ | -------------------------------- |
| `id`      | PK                 | Identificador único del usuario. |
| `name`    | —                  | Nombre.                          |
| `surname` | —                  | Apellido.                        |
| `email`   | —                  | Correo, usado para login.        |

---

### 9. Tabla: `Subscriptions`

Representa el modelo de suscripción recurrente (acceso a todo el contenido no público).

| Campo        | Función relacional | Descripción / Notas                                                                                                    |
| ------------ | ------------------ | ---------------------------------------------------------------------------------------------------------------------- |
| `id`         | PK                 | Identificador único.                                                                                                   |
| `date`       | —                  | Fecha de alta.                                                                                                         |
| `status`     | —                  | Estado de la suscripción (activa, cancelada, pendiente). Conjunto pequeño y estable de valores — podría ser un `ENUM`. |
| `user_id`    | FK → `Users.id`    | Usuario titular de la suscripción.                                                                                     |
| `payment_id` | FK → `Payments.id` | Pago asociado a esta suscripción.                                                                                      |

---

### 10. Tabla intermedia: `Users_courses`

Representa la compra puntual de un curso concreto, independiente de la suscripción. También registra el progreso del alumno.

| Campo        | Función relacional              | Descripción / Notas                  |
| ------------ | ------------------------------- | ------------------------------------ |
| `user_id`    | PK compuesta, FK → `Users.id`   | Usuario que compró el curso.         |
| `course_id`  | PK compuesta, FK → `Courses.id` | Curso comprado.                      |
| `payment_id` | FK → `Payments.id`              | Pago asociado a esta compra puntual. |
| `progress`   | —                               | Progreso del alumno en el curso.     |

---

### 11. Tabla: `Payments`

Tabla genérica de pagos, referenciada tanto desde `Subscriptions` como desde `Users_courses`. Cada fila de compra/suscripción "sabe" a qué pago corresponde, sin que `Payments` necesite saber para qué fue.

| Campo         | Función relacional | Descripción / Notas                                                                                                                                                                                                                     |
| ------------- | ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`          | PK                 | Identificador único del pago.                                                                                                                                                                                                           |
| `tipo`        | —                  | Tipo de pago (tarjeta, etc.).                                                                                                                                                                                                           |
| `card_number` | —                  | En un sistema productivo sería un token del procesador de pagos (Stripe, Redsys...), no el número real de la tarjeta. Se simplifica aquí por estar fuera del alcance del ejercicio.                                                     |
| `expired_at`  | —                  | Fecha de expiración del método de pago.                                                                                                                                                                                                 |
| `amount`      | —                  | Importe efectivamente pagado. Se guarda aquí, además de en `Course`/`Subscription`, para preservar el histórico: si el precio del curso cambia después de la compra, el registro de lo que la persona pagó realmente no debe alterarse. |

---

### 12. Tabla: `Tags`

Conjunto abierto y creciente de etiquetas libres — a diferencia de `Categories`, no se modela como `ENUM` porque cualquier usuario/autor puede necesitar un tag nuevo sin que eso implique una migración de esquema.

| Campo  | Función relacional | Descripción / Notas                    |
| ------ | ------------------ | -------------------------------------- |
| `id`   | PK                 | Identificador único del tag.           |
| `name` | —                  | Texto del tag (ej. "react", "docker"). |

---

### 13. Tabla intermedia: `TagsVideos`

| Campo      | Función relacional             | Descripción / Notas |
| ---------- | ------------------------------ | ------------------- |
| `video_id` | PK compuesta, FK → `Videos.id` | Vídeo etiquetado.   |
| `tag_id`   | PK compuesta, FK → `Tags.id`   | Tag aplicado.       |

---

### 14. Tabla intermedia: `TagsCourses` _(Desafío)_

El enunciado del desafío menciona explícitamente tags en **curso o vídeo** — de ahí esta tabla en lugar de una para artículos.

| Campo       | Función relacional              | Descripción / Notas |
| ----------- | ------------------------------- | ------------------- |
| `course_id` | PK compuesta, FK → `Courses.id` | Curso etiquetado.   |
| `tag_id`    | PK compuesta, FK → `Tags.id`    | Tag aplicado.       |

---

## 🔗 Diagrama de Relaciones

```
Courses
  ├──◄ course_id ── Videos ──► author_id ──► Authors
  │                     └────► category_id ─► Categories (◄ parent_id, auto-ref)
  │
  ├──◄ course_id ── Articles ─► author_id ──► Authors
  │                     └────► category_id ─► Categories
  │
  ├──◄ course_id ── Courses_authors ──► author_id ──► Authors
  │                  (tabla intermedia N:M)
  │
  ├──◄ course_id ── Users_courses ──► user_id ──► Users
  │                  (tabla intermedia N:M)          └──► has ──► Subscriptions
  │                     └──► payment_id ──► Payments ◄──────────────┘
  │
  └──◄ course_id ── TagsCourses ──► tag_id ──► Tags ◄── tag_id ── TagsVideos ──► video_id ──► Videos

Authors
  └──◄ author_id ── Social_networks
                     (1:N — un autor, varias redes)
```

---

## 💡 Justificación de Decisiones de Diseño

### 🔀 Clave Foránea (FK) en la tabla con muchas ocurrencias

Ubicación de las FK: siguiendo la lógica de las relaciones 1:M, hemos situado siempre la clave foránea en la tabla con muchas ocurrencias (ej. en `Videos` y no en `Courses`, en `Videos` y no en `Authors`) para evitar la repetición innecesaria de filas y cumplir con la 2FN.

### 🧩 Simetría Video↔Article en autoría y categoría

Aunque el enunciado solo pedía autor y categoría para vídeos, se incluyen también en artículos para mantener un esquema rígido y profesional, facilitando futuras consultas de búsqueda y filtrado por temática.

### 📦 Solo referencias para recursos pesados (S3 + CMS)

Al igual que el modelo documental usa `descriptionGuid`/`contentGuid` para no almacenar contenido pesado en Mongo, aquí se usan dos campos separados — `cms_id` y `s3_key` — asumiendo que el CMS gestiona metadata/texto y S3 aloja el archivo multimedia de forma independiente.

### 🌳 Jerarquía de categorías: auto-referencia, no niveles fijos

La auto-referencia (`Categories.parent_id`) permite una jerarquía de profundidad arbitraria (Front End >> React >> Testing >> ...), construyendo el breadcrumb con una consulta recursiva.

### 🔒 Contenido público/privado a nivel de contenido, no solo de curso

El enunciado permite que un curso tenga "una parte inicial 100% pública, y otra sólo para suscriptores" — por eso `is_public` vive en `Videos` y `Articles` de forma independiente, y no únicamente en `Courses`. Un curso puede ser público en bloque (`Courses.is_public = true`) o mixto, resuelto pieza por pieza.

### 💳 Precio duplicado: `Payments.amount` vs. precio "actual" del curso

Se guarda el importe pagado en `Payments`, independientemente de cuál sea el precio actual del curso/suscripción. Esto preserva el historial: si el precio sube o baja después de la compra, el registro de lo que la persona realmente pagó en su momento no cambia.

### 📈 Visualizaciones: contador simple + contador cacheado, con consistencia eventual

`Videos.views_count` se incrementa en cada reproducción — no se necesita una tabla de eventos por-visualización, porque el enunciado solo pide un total, no un historial detallado. `Courses.total_views` **no** se recalcula en cada request, ni se actualiza en la misma escritura que `views_count` — es un contador cacheado, actualizado mediante un proceso periódico en segundo plano (ej. cada hora), aceptando que puede quedar levemente desactualizado. El enunciado no exige tiempo real, así que el coste de mantenerlo exacto al segundo no se justifica frente al beneficio.

### 🏷️ Tags como tabla, no como `ENUM`

A diferencia de `status` en `Subscriptions` (conjunto pequeño, cerrado y estable → candidato a `ENUM`), los tags son un conjunto abierto y en crecimiento constante — cualquier autor puede necesitar un tag nuevo en cualquier momento. Modelarlos como `ENUM` obligaría a una migración de esquema cada vez que aparece un tag nuevo, así que se usa una tabla `Tags` real, con tablas intermedias (`TagsVideos`, `TagsCourses`) en lugar de una única tabla polimórfica que rompería la integridad referencial nativa de SQL.
