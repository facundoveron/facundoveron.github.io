# Recuperando tareas perdidas: cuando Git sobrescribió mi "base de datos" JSON

**Fecha:** 13/07/2026  
**Tecnologías:** Next.js, TypeScript, Git, Linux, JSON  
**Categorías:** Git, Debugging, Deploy, Versionado

---

# Contexto

Este incidente ocurrió durante el desarrollo de un proyecto personal: una aplicación web desarrollada con **Next.js** y **TypeScript** para gestionar mis tareas personales mediante un **diagrama de Gantt**.

La aplicación utiliza, por simplicidad, un archivo `tasks.json` como almacenamiento de datos. Cada vez que agrego o modifico una tarea, el archivo se actualiza y la aplicación lo utiliza como fuente de información.

Para organizar el desarrollo sigo un flujo de trabajo basado en Git:

- `main`: versiones estables en producción.
- `develop`: integración de nuevas funcionalidades.
- `feature/*`: desarrollo de nuevas características.
- `fix/*`: corrección de errores.
- Tags para identificar las versiones publicadas.

El problema apareció durante un despliegue en el servidor.

---

# Problema

Después de finalizar una nueva funcionalidad, seguí el flujo habitual:

1. Finalicé la rama `feature/gantt-improvements`.
2. Integré los cambios en `develop`.
3. Realicé el merge hacia `main`.
4. Me conecté al servidor para desplegar la nueva versión.

En el servidor ejecuté un `git pull` para obtener la última versión del proyecto.

Al abrir la aplicación noté inmediatamente que **habían desaparecido todas las tareas que había registrado durante varios días**.

El archivo `tasks.json` había sido reemplazado por la versión existente en el repositorio remoto, sobrescribiendo la información almacenada localmente en el servidor.

Afortunadamente, los datos todavía existían en el historial de Git del servidor.

---

# Investigación

La primera hipótesis fue que el despliegue había fallado o que existía un problema de lectura del archivo JSON.

Comencé revisando:

- El historial reciente de commits.
- Las diferencias del archivo `tasks.json`.
- El estado del repositorio (`git status`).
- El historial (`git log`).

## Historial simplificado

| Commit | Descripción |
|---------|-------------|
| `f3135cb` | Merge de `feature/gantt-improvements` |
| `9a7d1fe` | Actualización del frontend |
| `6c28e41` | Refactor del componente Gantt |
| `2d8bf0a` | Estado anterior del archivo `tasks.json` |

Comparando versiones descubrí que el `git pull` había reemplazado completamente el archivo por la versión almacenada en GitHub.

La causa fue sencilla pero importante:

> Nunca había agregado `tasks.json` al archivo `.gitignore`.

Por lo tanto, Git interpretaba cada modificación del archivo como un cambio que debía versionarse.

---

# Acciones

## 1. Restauré la última versión correcta del archivo

Utilicé Git para recuperar la versión anterior del archivo antes de que fuera sobrescrito.

```bash
git restore --source=2d8bf0a -- tasks.json
```

Con esto recuperé todas las tareas registradas anteriormente.

---

## 2. Verifiqué la información

Comprobé que:

- Todas las tareas estuvieran nuevamente disponibles.
- Las fechas fueran correctas.
- No existieran inconsistencias en el JSON.

---

## 3. Evité que Git vuelva a modificar ese archivo

Actualicé el archivo `.gitignore` agregando:

```gitignore
tasks.json
```

---

## 4. Eliminé el archivo del control de versiones

Como el archivo ya estaba siendo seguido por Git, también fue necesario dejar de versionarlo.

```bash
git rm --cached tasks.json
```

Luego realicé un commit específico para este cambio.

```text
commit 74d3ab1
Author: Facundo Veron

fix(git): stop tracking tasks.json

- Remove tasks.json from Git index
- Add tasks.json to .gitignore
- Preserve local task data
```

---

## 5. Actualicé el repositorio

La corrección fue desarrollada siguiendo el flujo habitual de ramas.

```text
fix/gitignore-tasks-json
        │
        ▼
develop
        │
        ▼
main
        │
        ▼
tag v1.2.1
```

Desde esa versión, `tasks.json` permanece únicamente en cada entorno donde se ejecuta la aplicación y ya no forma parte del repositorio.

---

# Post-mortem

La causa raíz del incidente no fue Git ni el proceso de despliegue.

Fue una decisión de diseño que pasó desapercibida al inicio del proyecto.

Durante las primeras versiones utilicé un archivo JSON como una base de datos temporal. Sin embargo, olvidé que ese archivo contenía **datos dinámicos**, no código fuente.

Al estar versionado:

- Cada commit podía modificarlo.
- Cualquier merge podía reemplazar su contenido.
- Un `git pull` podía sobrescribir información generada en producción.

En retrospectiva, el error ocurrió porque traté un archivo de datos como si fuera parte del código de la aplicación.

También comprendí que revisar el contenido del `.gitignore` debería formar parte del checklist inicial de cualquier proyecto.

---

# Aprendizajes

- Los archivos que representan datos de ejecución no deberían versionarse.
- Configurar correctamente `.gitignore` desde el inicio evita muchos problemas futuros.
- Antes de realizar un despliegue conviene revisar qué archivos serán modificados por el `git pull`.
- Conocer comandos de recuperación de Git permite resolver incidentes rápidamente sin perder información.
- Mantener un flujo de ramas (`feature`, `fix`, `develop`, `main`) facilita aislar y documentar este tipo de correcciones.

---

# Próximos pasos

A partir de este incidente implementé varias mejoras:

- Crear una checklist para nuevos proyectos que incluya la revisión del `.gitignore`.
- Separar claramente el código fuente de los datos generados por la aplicación.
- Realizar una copia de seguridad de `tasks.json` antes de cada despliegue mientras continúe utilizándolo.
- Evaluar reemplazar el archivo JSON por una base de datos ligera como SQLite para almacenar información persistente.
- Documentar el procedimiento de despliegue para reducir la probabilidad de errores similares.

---

# Recursos

## Comandos utilizados

```bash
git log --oneline

git status

git diff tasks.json

git restore --source=2d8bf0a -- tasks.json

git rm --cached tasks.json

git add .gitignore

git commit -m "fix(git): stop tracking tasks.json"

git push origin fix/gitignore-tasks-json
```

## Commits relacionados (ficticios)

| Hash | Descripción |
|------|-------------|
| `f3135cb26dc0d0340825a7150f12487999fe5d56` | Merge de `feature/gantt-improvements` |
| `9a7d1fe1a78d4e7c24b3e1fa8c4fdb9ab73c5d91` | Mejora del renderizado del Gantt |
| `6c28e41fd4d72bb8b4d2aef6e4fd1f0aa95ab623` | Refactor del manejo de tareas |
| `2d8bf0a5f18fdd96bfa8d85c4efbc0d2f32d1c77` | Última versión válida de `tasks.json` |
| `74d3ab1e9a4b66bfe29b6ab70df15d8cb8410ef4` | Dejar de versionar `tasks.json` |

---

# Conclusión

Aunque el incidente fue sencillo, dejó una enseñanza importante: **no todo archivo dentro del proyecto debe formar parte del repositorio**. El código fuente y los datos de la aplicación tienen ciclos de vida diferentes y deben gestionarse de manera distinta.

La experiencia también reforzó la importancia de comprender cómo Git maneja los archivos versionados y de establecer buenas prácticas desde el inicio del desarrollo. Gracias a este error, el proceso de despliegue quedó más robusto y el proyecto mejor preparado para seguir evolucionando.
