# 🎓 Modelo de Datos Relacional: Portal eLearning (Programación)

Este documento presenta el modelado relacional (basado en tablas y relaciones) para un portal de eLearning
. A diferencia del modelado documental, donde la información se suele embeber (meter objetos dentro de objetos) para agilizar lecturas específicas
, este diseño se fundamenta en la aplicación de las formas normales
. El objetivo principal es evitar redundancias para que el dato no esté repetido en varios sitios y no existan inconsistencias al actualizarlo
. Asimismo, el modelo garantiza la integridad referencial, asegurando que cada conexión entre tablas sea sólida y que no existan registros apuntando a datos inexistentes
.

### Parte obligatoria del ejercicio.

---

## 📦 Resumen de Tablas

| #   | Tabla             | Descripción                                                |
| --- | ----------------- | ---------------------------------------------------------- |
| 1   | `Courses`         | Tabla central. Contenedor de vídeos y artículos.           |
| 2   | `Videos`          | Lecciones en formato vídeo, con autor y categoría propios. |
| 3   | `Articles`        | Lecciones en formato artículo, simétricas a `Videos`.      |
| 4   | `Authors`         | Información biográfica de los instructores.                |
| 5   | `Categories`      | Temáticas de clasificación.                                |
| 6   | `Courses_authors` | Tabla intermedia N:M entre `Courses` y `Authors`.          |

---

## 🛠️ Detalle del Esquema

### 1. Tabla: `Courses`

Es el eje del portal. A diferencia del modelo documental (que usa _extended reference_ embebiendo autor/categoría), aquí el vínculo con autores se resuelve mediante tabla intermedia, ya que un curso puede tener **varios** autores.

| Campo        | Función relacional | Descripción / Notas                                            |
| ------------ | ------------------ | -------------------------------------------------------------- |
| `id`         | PK                 | Identificador único del curso.                                 |
| `nombre`     |                    | Título del curso.                                              |
| `created_at` |                    | Fecha de creación. Usado para ordenar "cursos nuevos" en Home. |

---

### 2. Tabla: `Videos`

| Campo         | Función relacional             | Descripción / Notas                                          |
| ------------- | ------------------------------ | ------------------------------------------------------------ |
| `id`          | PK, Not Null                   | Identificador único del vídeo.                               |
| `name`        | —                              | Título del vídeo.                                            |
| `course_id`   | FK → `Courses.id`, Not Null    | Curso al que pertenece (1:N, sin compartición entre cursos). |
| `category_id` | FK → `Categories.id`, Not Null | Un vídeo pertenece a una sola temática.                      |
| `author_id`   | FK → `Authors.id`, Not Null    | Un vídeo tiene un único autor (1:N, FK en el lado "N").      |
| `cms_id`      | —                              | Referencia al contenido/metadata en el headless CMS.         |
| `s3_key`      | —                              | Referencia al archivo de vídeo en S3.                        |
| `views_count` | —                              | Contador simple, incrementado en cada reproducción.          |

---

### 3. Tabla: `Articles`

Estructuralmente simétrica a `Videos`.

| Campo         | Funcion relacional             | Descripción / Notas                                    |
| ------------- | ------------------------------ | ------------------------------------------------------ |
| `id`          | PK, Not Null                   | Identificador único del artículo.                      |
| `name`        | —                              | Título del artículo.                                   |
| `course_id`   | FK → `Courses.id`, Not Null    | Artículo asociado a un solo curso.                     |
| `category_id` | FK → `Categories.id`, Not Null | Aplicado por simetría con `Video`.                     |
| `author_id`   | FK → `Authors.id`, Not Null    | Aplicado por simetría con `Video`.                     |
| `cms_id`      | —                              | Referencia al cuerpo del artículo en el CMS.           |
| `s3_key`      | —                              | Referencia a recursos multimedia embebidos, si aplica. |

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

| Campo  | Función relacional | Descripción / Notas                |
| ------ | ------------------ | ---------------------------------- |
| `id`   | PK, Not Null       | Identificador de la categoría.     |
| `name` | —                  | Nombre (ej. "Frontend", "Devops"). |

---

### 6. Tabla intermedia: `Courses_authors`

Resuelve la relación N:M entre `Courses` y `Authors` (un curso puede tener varios autores; un autor puede participar en varios cursos).

| Campo       | Función relacional              | Descripción / Notas     |
| ----------- | ------------------------------- | ----------------------- |
| `course_id` | PK compuesta, FK → `Courses.id` | Identificador del curso |
| `author_id` | PK compuesta, FK → `Authors.id` | Identificador del autor |

---

### 7. Tabla: `Social_networks`

He incluido esta tabla debido a que en los mockups aparece esta posibilidad,la página de biografía del autor muestra enlaces a sus redes sociales. Se modela como relación 1:N (un autor puede tener varias redes: Twitter, LinkedIn, GitHub, etc.), con la FK en el lado "N", siguiendo el mismo principio aplicado en el resto del esquema.

| Campo       | Función relacional | Descripción / Notas                                 |
| ----------- | ------------------ | --------------------------------------------------- |
| `id`        | PK                 | Identificador                                       |
| `platform`  |                    | Nombre de la red social (ej. "GitHub", "LinkedIn"). |
| `url`       |                    | Enlace al perfil.                                   |
| `author_id` | FK → `Authors.id`  | Identificador del autor                             |

---

## 🔗 Diagrama de Relaciones

```
Courses
  ├──◄ course_id ── Videos ──► author_id ──► Authors
  │                     └────► category_id ─► Categories
  │
  ├──◄ course_id ── Articles ─► author_id ──► Authors
  │                     └────► category_id ─► Categories
  │
  └──◄ course_id ── Courses_authors ──► author_id ──► Authors
                     (tabla intermedia N:M)

Authors
  └──◄ author_id ── Social_networks
                     (1:N — un autor, varias redes)
```

---

## 💡 Justificación de Decisiones de Diseño

### 🔀 Clave Foránea (FK) en la tabla con muchas ocurrencias

Ubicación de las FK: Siguiendo la lógica de las relaciones 1:M, hemos situado siempre la clave foránea en la tabla con muchas ocurrencias (ej. en Videos y no en Courses) para evitar la repetición innecesaria de filas y cumplir con la 2FN

### 🧩 Simetría Video↔Article en autoría y categoría

Simetría de Dominio: Aunque el enunciado solo pedía categorías para vídeos, se incluyen en artículos para mantener un esquema rígido y profesional, facilitando futuras consultas de búsqueda y filtrado por temática

### 📦 Solo referencias para recursos pesados (S3 + CMS)

Al igual que el modelo documental usa `descriptionGuid` / `contentGuid` para no almacenar contenido pesado en Mongo, aquí se usan dos campos separados — `cms_id` y `s3_key` — asumiendo que el CMS gestiona metadata/texto y S3 aloja el archivo multimedia de forma independiente.

### 🆕 Normalización frente a "Embeber"

diferencia de MongoDB, aquí no metemos el autor dentro del curso. Esto permite que si un autor cambia su biografía, solo tengamos que actualizar un campo en un solo sitio, evitando que la información quede "descoordinada",
