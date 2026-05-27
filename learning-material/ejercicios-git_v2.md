# Ejercicios prácticos de Git

Esta guía contiene una progresión de **25 ejercicios** sobre el mismo repositorio. Cada comando va acompañado de **una explicación de qué hace y por qué se usan las flags indicadas**. Si necesitas conceptos previos, consulta `guia-teorica-git.md`.

**Repositorio único:** crearás una carpeta `git-practica` en el ejercicio 2 y trabajarás dentro de ella durante toda la guía. En el ejercicio 17 se conecta con un remoto en GitHub. A partir de ahí, todo sigue en ese mismo repo local + remoto.

**Cómo leer cada ejercicio:**
- Cada comando viene en su propio bloque, seguido de un comentario corto que explica qué hace.
- Cuando aparecen flags, se indica para qué sirven.
- El símbolo 📌 marca conceptos clave a interiorizar.

---

## Ejercicio 1 — Configuración inicial de Git

**Objetivo:** Configurar tu identidad. Solo se hace una vez por máquina.

```bash
git config --global user.name "Tu Nombre"
# Guarda tu nombre en ~/.gitconfig. `--global` = aplica a todos los repos de tu usuario.
# Sin `--global`, Git intentaría escribir en .git/config, que aún no existe.

git config --global user.email "tu.email@ejemplo.com"
# Tu email. Aparecerá en cada commit.

git config --global init.defaultBranch main
# Que `git init` cree la rama como "main" en lugar de "master" (la convención moderna).

git config --global core.editor "nano"
# Editor que Git abrirá cuando necesite uno (mensajes largos, rebase interactivo).
# Alternativas: "vim", "code --wait" para VS Code, "subl -w" para Sublime.
```

**Verificación:**

```bash
git config --list
# Muestra TODA la configuración efectiva (sistema + global + local).

git config --list --show-origin
# Igual, pero indica de qué archivo viene cada valor. Útil si hay valores inesperados.
```

---

## Ejercicio 2 — Crear el repositorio de prácticas

**Objetivo:** Inicializar el repositorio que usarás durante toda la guía.

```bash
mkdir git-practica
# Crea la carpeta en el directorio actual.

cd git-practica
# Entra en ella. Todos los comandos siguientes se ejecutan aquí.

git init
# Crea la carpeta oculta .git/ que convierte este directorio en un repo Git.
# Sin .git/, esta carpeta sería una carpeta normal sin historial.

git status
# Muestra el estado actual. Debería decir:
#   - On branch main
#   - No commits yet
#   - nothing to commit
```

📌 **A partir de aquí, todos los comandos se ejecutan dentro de `git-practica`.**

---

## Ejercicio 3 — Tu primer commit

**Objetivo:** Crear un archivo, añadirlo al staging y hacer el primer commit.

**Crea el archivo `README.md`** con este contenido:

```markdown
# Mi Primer Repositorio

Este es mi primer proyecto en Git. Estoy aprendiendo control de versiones.
```

**Comandos:**

```bash
git status
# Ahora muestra README.md como "Untracked files": Git lo ve pero no lo rastrea aún.

git add README.md
# Mueve README.md al staging area. Le decimos "esto entrará en el próximo commit".

git status
# Ahora dice "Changes to be committed" — el archivo pasó de untracked a staged.

git commit -m "Initial commit: add README"
# Crea un commit con los cambios en staging.
# `-m "..."` permite poner el mensaje en línea, sin abrir el editor.

git log
# Muestra el historial. Verás tu commit con hash, autor, fecha y mensaje.
```

---

## Ejercicio 4 — Modificar y commitear cambios

**Objetivo:** Practicar el ciclo modificar → stage → commit, y aprender a ver diferencias.

**Reemplaza el contenido de `README.md` por:**

```markdown
# Mi Primer Repositorio

Este es mi primer proyecto en Git. Estoy aprendiendo control de versiones.

## Lo que he aprendido
- Inicializar un repositorio con `git init`
- Añadir archivos con `git add`
- Confirmar cambios con `git commit`
```

**Comandos:**

```bash
git diff
# Muestra las diferencias entre el working directory y el staging.
# Como el archivo NO está en staging todavía, ves todos tus cambios sin guardar.

git add README.md
# Mueve los cambios al staging.

git diff
# Ahora no muestra nada: ya no hay diferencias entre working directory y staging.

git diff --staged
# Esta variante compara staging vs último commit. Muestra lo que se commitearía ahora.
# `--cached` es sinónimo de `--staged`.

git commit -m "Add learning section to README"
# Crea el commit con los cambios stageados.

git log --oneline
# `--oneline` = una línea por commit (hash corto + mensaje). Útil para ver muchos commits.
```

---

## Ejercicio 5 — El archivo .gitignore

**Objetivo:** Excluir archivos del control de versiones.

**Crea los siguientes archivos en el repo:**

`notas.txt`:
```
Estas son mis notas personales, no deben subirse.
```

`secreto.env`:
```
API_KEY=12345-secreto
```

`app.log`:
```
[INFO] Aplicación iniciada
```

**Crea el archivo `.gitignore`:**

```
# Archivos de entorno (cualquiera con extensión .env)
*.env

# Logs
*.log

# Archivo concreto
notas.txt
```

**Comandos:**

```bash
git status
# Antes del .gitignore verías los 3 archivos como untracked.
# Tras crear .gitignore, solo aparece .gitignore (los demás están ignorados).

git add .gitignore
# Añadimos solo el .gitignore al staging.

git commit -m "Add .gitignore"
# Lo commiteamos.

git status
# Debería decir "nothing to commit, working tree clean".
# Los archivos ignorados ni siquiera aparecen.
```

📌 **`.gitignore` solo afecta a archivos no rastreados aún.** Si un archivo ya estaba commiteado, añadirlo al `.gitignore` no lo elimina; tendrías que hacer `git rm --cached archivo`.

---

## Ejercicio 6 — Deshacer cambios antes de commitear

**Objetivo:** Descartar cambios en el working directory y en staging.

**Modifica `README.md`** añadiendo al final:

```markdown

## Esta línea es un error que voy a deshacer
```

**Comandos para descartar cambios del working directory:**

```bash
git status
# Muestra README.md como modificado (no en staging).

git diff
# Te enseña exactamente qué cambió.

git restore README.md
# Descarta los cambios del working directory.
# El archivo vuelve a estar como en el último commit. ⚠️ DESTRUCTIVO: pierdes esa edición.

git status
# Debería decir "working tree clean".
```

**Ahora practica deshacer en staging.** Vuelve a hacer la modificación, luego:

```bash
git add README.md
# Pasamos los cambios al staging.

git status
# Aparece como "Changes to be committed".

git restore --staged README.md
# `--staged` saca el archivo del staging, PERO mantiene los cambios en el working directory.
# Útil cuando hiciste `git add` por error.

git status
# Ahora aparece como modificado pero no staged.

git restore README.md
# Y finalmente descartamos los cambios del working directory.
```

📌 **Resumen:** `git restore` (sin flag) afecta al working directory; con `--staged` afecta solo al staging.

---

## Ejercicio 7 — Ver el historial e inspeccionar commits

**Objetivo:** Explorar el historial del repositorio.

Antes de los comandos, haz un commit más. Añade al final de `README.md`:

```markdown

## Comandos básicos aprendidos
- git init
- git add
- git commit
- git status
- git log
```

```bash
git add README.md
git commit -m "Document basic commands learned"
```

**Variantes de log:**

```bash
git log
# Vista completa: hash largo, autor, fecha y mensaje completo por commit.

git log --oneline
# Una línea por commit (hash corto + mensaje). Vista compacta.

git log --oneline --graph --all
# `--graph` dibuja líneas ASCII mostrando el grafo de commits.
# `--all` incluye TODAS las ramas (no solo la actual).
# Combinación oro para ver la topología del repo.

git log -p
# `-p` (de "patch") añade el diff de cada commit.

git log --stat
# Resumen estadístico: qué archivos cambiaron y cuántas líneas.

git log -2
# Solo los 2 commits más recientes. Funciona con cualquier número.

git show HEAD
# Muestra el commit actual (mensaje + diff). HEAD = puntero al commit actual.

git show HEAD~1
# El commit anterior. HEAD~N = N commits hacia atrás.
```

---

## Ejercicio 8 — Crear y cambiar entre ramas

**Objetivo:** Trabajar con ramas (branches).

```bash
git branch
# Lista las ramas locales. La actual lleva un asterisco delante.

git branch feature-saludo
# Crea una rama nueva llamada "feature-saludo" en el commit actual.
# Importante: SOLO la crea, NO te cambia a ella.

git branch
# Ahora verás las dos ramas, pero sigues en main.

git switch feature-saludo
# Cambia a la rama feature-saludo. Equivale al antiguo `git checkout`.

git branch
# El asterisco se movió a feature-saludo.
```

**Crea un archivo nuevo `saludo.txt`:**

```
¡Hola mundo desde la rama feature-saludo!
```

```bash
git add saludo.txt
git commit -m "Add saludo.txt on feature branch"
# Este commit existe SOLO en feature-saludo, no en main.

git switch main
# Vuelve a main.

ls
# saludo.txt ha desaparecido: no existe en main, solo en feature-saludo.
```

📌 Las ramas son **independientes**: archivos creados en una no se ven en otra hasta que las mezcles.

---

## Ejercicio 9 — Hacer merge de una rama (fast-forward)

**Objetivo:** Integrar los cambios de una rama en otra.

```bash
git switch main
# Asegúrate de estar en la rama destino del merge.

git log --oneline --graph --all
# Visualiza dónde está cada rama antes del merge.

git merge feature-saludo
# Fusiona feature-saludo dentro de main.
# Como main no avanzó desde que se creó la rama, Git hace un "fast-forward":
# simplemente mueve el puntero de main al último commit de feature-saludo.
# NO se crea commit de merge.

git log --oneline --graph --all
# Ahora ambas ramas apuntan al mismo commit. Historial lineal.

ls
# saludo.txt ya está visible en main.
```

**Eliminar la rama ya fusionada:**

```bash
git branch -d feature-saludo
# `-d` (delete) borra la rama. Solo funciona si ya fue mergeada (protección).

git branch
# La rama desapareció.
```

📌 Para borrar una rama no mergeada usarías `-D` (mayúscula), que fuerza el borrado.

---

## Ejercicio 10 — Merge con conflictos

**Objetivo:** Provocar y resolver un conflicto de merge.

**Paso 1 — Desde `main`, edita `README.md`** y reemplaza la primera línea por:

```markdown
# Mi Primer Repositorio — Edición desde MAIN
```

```bash
git add README.md
git commit -m "Update title from main"
```

**Paso 2 — Crea una rama partiendo del commit ANTERIOR**, no del actual:

```bash
git switch -c feature-titulo HEAD~1
# `-c` (create) crea la rama y cambia a ella en un solo paso.
# `HEAD~1` = el commit anterior al actual. Así la rama nace antes de tu edición de main.
```

**Edita `README.md`** y reemplaza la primera línea por:

```markdown
# Mi Primer Repositorio — Edición desde FEATURE
```

```bash
git add README.md
git commit -m "Update title from feature branch"

git switch main
# Volvemos a main.

git merge feature-titulo
# Intenta mergear. Como ambas ramas modificaron la misma línea del mismo archivo,
# Git no sabe cuál usar y declara CONFLICTO.
```

**Verás algo así en `README.md`:**

```
<<<<<<< HEAD
# Mi Primer Repositorio — Edición desde MAIN
=======
# Mi Primer Repositorio — Edición desde FEATURE
>>>>>>> feature-titulo
```

**Resuelve el conflicto** dejando la línea como:

```markdown
# Mi Primer Repositorio — Edición combinada (main + feature)
```

(Borra también los marcadores `<<<<<<<`, `=======` y `>>>>>>>`.)

```bash
git status
# Te indica que hay un archivo "both modified": el conflicto sigue marcado.

git add README.md
# Al hacer add tras resolver, marcamos el conflicto como resuelto.

git commit -m "Resolve merge conflict in README title"
# Completa el merge. Si no pasas `-m`, Git abre el editor con un mensaje predefinido.

git log --oneline --graph --all
# Ahora ves un commit de merge con dos padres.

git branch -d feature-titulo
# Eliminamos la rama ya fusionada.
```

📌 Si te arrepientes durante un conflicto, `git merge --abort` cancela el merge y deja todo como estaba antes.

---

## Ejercicio 11 — Cambios sin commitear que bloquean un switch

**Objetivo:** Ver en acción la protección de Git al cambiar de rama con cambios sin commitear que entran en conflicto con la rama destino.

📌 **Concepto clave:** Si tienes modificaciones en el working directory y la rama destino tiene una versión distinta del MISMO archivo, Git te impide cambiar de rama para no perder tus cambios.

**Paso 1 — Crea una rama nueva con un cambio en `README.md`:**

```bash
git switch -c experimento-titulo
```

**Edita `README.md`** y reemplaza la primera línea por:

```markdown
# Mi Primer Repositorio — Versión EXPERIMENTAL
```

```bash
git add README.md
git commit -m "Experimental title change"
```

**Paso 2 — Vuelve a `main`:**

```bash
git switch main
```

**Paso 3 — Modifica `README.md` SIN COMMITEAR.** Reemplaza la primera línea por:

```markdown
# Mi Primer Repositorio — Trabajo en progreso sin commitear
```

**NO hagas `git add` ni `git commit`.**

**Paso 4 — Intenta cambiar a `experimento-titulo`:**

```bash
git switch experimento-titulo
```

**Verás el error:**

```
error: Your local changes to the following files would be overwritten by checkout:
        README.md
Please commit your changes or stash them before you switch branches.
Aborting
```

Git te protege: cambiar de rama sobrescribiría tus modificaciones locales porque la otra rama tiene una versión diferente del mismo archivo.

**Tres soluciones posibles:**

```bash
# Opción A: Commitear los cambios primero.
git add README.md
git commit -m "WIP: title in progress"
git switch experimento-titulo

# Opción B: Descartar los cambios (los pierdes).
# git restore README.md
# git switch experimento-titulo

# Opción C: Guardarlos temporalmente con stash (siguiente ejercicio).
```

Usa la **Opción A**. Al final, vuelve a `main` y borra la rama experimental para no acumular ramas:

```bash
git switch main
git branch -D experimento-titulo
# `-D` fuerza el borrado aunque la rama no esté mergeada.
```

📌 Si el archivo modificado **no existiera** en la rama destino, Git te dejaría pasar sin protesta: no habría nada que sobrescribir.

---

## Ejercicio 12 — git stash: guardar cambios temporalmente

**Objetivo:** Usar `stash` cuando necesitas cambiar de rama pero no quieres hacer un commit "basura".

**Paso 1 — Desde `main`, modifica `README.md`** añadiendo al final:

```markdown

## Sección en construcción
Esta sección está sin terminar.
```

**Paso 2 — Guarda los cambios temporalmente con stash:**

```bash
git status
# Confirma que tienes cambios sin commitear.

git stash
# Empuja los cambios a una pila temporal y limpia el working directory.
# El archivo vuelve al estado del último commit.

git status
# "working tree clean": no hay nada pendiente.

git stash list
# Lista los stashes guardados. Verás algo como:
#   stash@{0}: WIP on main: <hash> <mensaje del último commit>
```

**Paso 3 — Recupera el stash:**

```bash
git stash pop
# Aplica el stash más reciente Y lo borra de la pila.
# Tus cambios reaparecen en el working directory.

git status
# Verás README.md modificado de nuevo.
```

**Variantes útiles a practicar:**

```bash
git stash push -m "Trabajo en sección README"
# Stashea con un mensaje descriptivo. Útil cuando vas a tener varios stashes.

git stash list
# Verifica que aparece con tu mensaje.

git stash show -p stash@{0}
# `-p` muestra el diff completo del stash, no solo un resumen.

git stash apply stash@{0}
# Aplica un stash concreto SIN borrarlo de la pila (a diferencia de `pop`).

git stash drop stash@{0}
# Borra un stash específico sin aplicarlo.
```

Antes de continuar, deja el README limpio:

```bash
git restore README.md
git status
```

---

## Ejercicio 13 — Modificar el último commit (amend)

**Objetivo:** Corregir el último commit (mensaje o contenido) sin crear uno nuevo.

**Cambiar el mensaje del último commit:**

```bash
git log --oneline
# Anota el mensaje actual del commit más reciente.

git commit --amend -m "Document basic commands and learning notes"
# `--amend` reescribe el último commit en lugar de crear uno nuevo.
# Con `-m` cambias el mensaje directamente.

git log --oneline
# El último commit aparece con el nuevo mensaje y nuevo hash (es un commit distinto).
```

**Añadir un archivo olvidado al último commit:**

Crea un archivo `notas-correccion.txt`:

```
Olvidé añadir este archivo al commit anterior.
```

```bash
git add notas-correccion.txt
# Stageamos el archivo nuevo.

git commit --amend --no-edit
# `--no-edit` = mantén el mensaje del commit anterior sin abrir el editor.
# El archivo se añade al commit existente.

git log --stat -1
# `--stat` muestra qué archivos cambian en cada commit.
# `-1` limita a 1 commit. Confirma que el archivo está incluido.
```

⚠️ **Importante:** `--amend` reescribe historia. Solo es seguro en commits que **NO has pusheado**. Como aún no tenemos remoto, no hay problema.

---

## Ejercicio 14 — Revertir un commit (git revert)

**Objetivo:** Deshacer un commit creando otro que invierta sus cambios. Es seguro incluso si ya pusheaste.

**Paso 1 — Crea un archivo "malo":**

`archivo-erroneo.txt`:
```
Este archivo no debería haber existido nunca.
```

```bash
git add archivo-erroneo.txt
git commit -m "Add archivo-erroneo.txt"

git log --oneline
# Verás el commit del archivo erróneo en la cima del historial.
```

**Paso 2 — Revierte ese commit:**

```bash
git revert HEAD
# `HEAD` = el commit más reciente.
# `revert` crea un NUEVO commit que invierte los cambios del indicado.
# Se abrirá el editor con un mensaje predefinido tipo "Revert ...". Guarda y cierra.
```

```bash
git log --oneline
# Ahora hay 2 commits: el original y el revert. La historia queda íntegra.

ls
# archivo-erroneo.txt ya no existe: el revert lo "deshizo".
```

📌 Diferencia clave: `reset` borra historia (no seguro en ramas compartidas); `revert` la añade (siempre seguro).

---

## Ejercicio 15 — git reset: soft, mixed y hard

**Objetivo:** Entender las tres variantes de `reset`.

**Prepara tres commits de prueba:**

```bash
echo "linea 1" > prueba-reset.txt
# `>` sobrescribe el archivo creándolo si no existe.

git add prueba-reset.txt
git commit -m "reset test: commit 1"

echo "linea 2" >> prueba-reset.txt
# `>>` añade al final (no sobrescribe).

git add prueba-reset.txt
git commit -m "reset test: commit 2"

echo "linea 3" >> prueba-reset.txt
git add prueba-reset.txt
git commit -m "reset test: commit 3"

git log --oneline
# Ahora hay 3 commits de prueba.
```

**Reset --soft (mantiene los cambios en staging):**

```bash
git reset --soft HEAD~1
# Mueve HEAD un commit atrás. El commit "commit 3" desaparece del historial.
# Pero sus cambios siguen en STAGING, listos para recommitear.

git status
# Verás "Changes to be committed" con las modificaciones del commit 3.

git log --oneline
# Solo quedan 2 commits.
```

**Reset --mixed (mantiene cambios en el working directory, no en staging):**

```bash
git reset --mixed HEAD~1
# `--mixed` es el default si no pones flag.
# Los cambios del commit deshecho vuelven al working directory, sin stage.

git status
# Verás "Changes not staged for commit".

git log --oneline
# Solo queda 1 commit.
```

**Reset --hard (descarta todo: el más peligroso):**

```bash
# Primero recomponemos para tener algo que perder:
git add prueba-reset.txt
git commit -m "reset test: recompose"

git reset --hard HEAD~1
# `--hard` borra el commit Y descarta los cambios. Working directory y staging
# quedan exactamente como en el commit destino. ⚠️ PIERDES los cambios.

git status
# "working tree clean".

cat prueba-reset.txt
# El archivo está como antes, o ni siquiera existe (depende de hasta dónde retrocediste).
```

⚠️ **`--hard` es destructivo.** Pero recuerda: los commits "perdidos" siguen accesibles por `git reflog` durante ~90 días (ejercicio 22).

Limpia antes de continuar:

```bash
git rm prueba-reset.txt
# `git rm` quita el archivo del repo Y del working directory en una sola acción.

git commit -m "Remove reset test file"
```

---

## Ejercicio 16 — Etiquetas (tags)

**Objetivo:** Marcar versiones específicas del proyecto.

```bash
git tag v1.0
# Crea un tag LIGHTWEIGHT: solo un nombre que apunta al commit actual.
# Simple, pero sin metadatos.

git tag -a v1.1 -m "Versión 1.1 con sección de comandos básicos"
# `-a` (annotated) crea un tag CON metadatos: autor, fecha y mensaje.
# `-m` define el mensaje del tag.
# Es el tipo recomendado para releases reales.

git tag
# Lista todos los tags.

git show v1.1
# Muestra info del tag (autor, fecha, mensaje) + info del commit asociado.

git tag -d v1.0
# `-d` (delete) borra un tag.

git tag -l "v1.*"
# `-l` lista filtrando por patrón. Útil con muchos tags.
```

📌 A diferencia de las ramas, los tags **no se mueven**. Una vez creados, apuntan siempre al mismo commit.

---

## Ejercicio 17 — Conectar el repositorio con un remoto (GitHub)

**Objetivo:** Subir tu repositorio local a GitHub. A partir de aquí trabajamos con el mismo repo, pero ya con remoto.

**Paso 1:** Crea un repositorio vacío en GitHub llamado `git-practica` (**sin** README, **sin** .gitignore, **sin** licencia, para evitar conflictos iniciales).

**Paso 2 — Conecta local y remoto:**

```bash
git remote add origin https://github.com/TU_USUARIO/git-practica.git
# `remote add` registra un remoto.
# `origin` es el nombre convencional para el remoto principal.
# La URL es la que GitHub te muestra tras crear el repo.

git remote -v
# `-v` (verbose) lista los remotos con sus URLs.
# Verás "origin" con dos líneas: una para fetch y otra para push.

git branch -M main
# `-M` renombra la rama actual a "main".
# Solo necesario si tu rama por defecto se llamaba diferente. Inofensivo si ya era main.

git push -u origin main
# Sube la rama main al remoto origin.
# `-u` (--set-upstream) asocia la rama local con la remota.
# Tras esto, futuros `git push` y `git pull` saben qué rama tocar sin argumentos.

git push origin --tags
# `--tags` sube TODOS los tags locales. Por defecto, `push` no los incluye.
```

Verifica en GitHub: deberías ver todos tus commits y el tag `v1.1`.

---

## Ejercicio 18 — Clonar, fetch y pull

**Objetivo:** Simular trabajar desde otro ordenador clonando el repo en una carpeta hermana.

**Paso 1 — Clona en una carpeta nueva:**

```bash
cd ..
# Sal temporalmente de git-practica.

git clone https://github.com/TU_USUARIO/git-practica.git git-practica-clon
# `clone` descarga el repo completo, configura `origin` automáticamente y crea la carpeta.
# El último argumento es opcional: nombre de la carpeta destino (sin él se llamaría "git-practica").

cd git-practica-clon
git log --oneline
# Verás todo el historial del repo original.
```

**Simula trabajo desde el clon.** Edita `README.md` añadiendo al final:

```markdown

## Edición desde el clon
Este cambio se hizo en el segundo entorno.
```

```bash
git add README.md
git commit -m "Edit from clone"
git push
# Sin argumentos extra porque `clone` ya configuró el upstream.
```

**Paso 2 — Vuelve al repositorio original y trae los cambios:**

```bash
cd ../git-practica

git fetch
# Descarga los nuevos commits del remoto SIN modificar tu rama local.
# Solo actualiza la referencia "origin/main".

git status
# Te dirá "Your branch is behind 'origin/main' by 1 commit": el remoto tiene algo que tú no.

git pull
# Equivale a `fetch` + `merge` en un solo paso.
# Trae los cambios y los mezcla con tu rama actual.

git log --oneline
# Ya tienes el commit que se hizo desde el clon.
```

📌 **`fetch` vs `pull`:** `fetch` solo descarga (puedes inspeccionar antes); `pull` descarga y mergea de golpe.

📌 La carpeta `git-practica-clon` queda solo como simulación; el repo principal sigue siendo `git-practica`.

---

## Ejercicio 19 — Rebase básico

**Objetivo:** Reescribir el historial para tenerlo lineal.

**Paso 1 — Crea una rama desde un commit ANTERIOR al actual:**

```bash
git switch -c feature-rebase HEAD~2
# Crea la rama y se posiciona dos commits antes del HEAD actual.
```

**Crea `feature-rebase.txt`:**

```
Trabajo hecho en la rama feature-rebase.
```

**Commits:**

```bash
git add feature-rebase.txt
git commit -m "Add feature-rebase.txt"

echo "más trabajo" >> feature-rebase.txt
git add feature-rebase.txt
git commit -m "Continue work on feature-rebase"
```

**Paso 2 — Rebase sobre main:**

```bash
git log --oneline --graph --all
# Verás la rama divergiendo de main hace varios commits.

git rebase main
# Toma los commits de feature-rebase y los REAPLICA sobre el último commit de main.
# Los commits resultantes tienen el mismo contenido pero distintos hashes.

git log --oneline --graph --all
# Ahora feature-rebase aparece "encima" de main sin divergencia.
```

**Merge fast-forward de vuelta a main y sube:**

```bash
git switch main
git merge feature-rebase
# Tras un rebase, el merge es fast-forward: simplemente mueve el puntero.

git branch -d feature-rebase
git push
# Subimos los nuevos commits al remoto.
```

⚠️ **Regla de oro del rebase:** NO hagas rebase de commits ya pusheados a una rama compartida. Aquí rebaseamos commits locales antes de pushearlos, que sí es seguro.

---

## Ejercicio 20 — Rebase interactivo: limpiar el historial

**Objetivo:** Combinar, reordenar o editar commits con `rebase -i` antes de pushearlos.

**Crea varios commits "sucios" en una rama de feature:**

```bash
git switch -c feature-limpieza

echo "a" > limpieza.txt
git add limpieza.txt
git commit -m "wip 1"

echo "b" >> limpieza.txt
git add limpieza.txt
git commit -m "wip 2"

echo "c" >> limpieza.txt
git add limpieza.txt
git commit -m "wip 3"
```

**Inicia el rebase interactivo:**

```bash
git rebase -i HEAD~3
# `-i` (interactive) abre el editor con los últimos 3 commits.
# Te permite decidir qué hacer con cada uno.
```

**En el editor verás:**

```
pick abc1234 wip 1
pick def5678 wip 2
pick ghi9012 wip 3
```

**Cámbialo a:**

```
pick abc1234 wip 1
squash def5678 wip 2
squash ghi9012 wip 3
```

`squash` (o `s`) fusiona ese commit con el anterior. Guarda y cierra.

Se abre otro editor para escribir el mensaje del commit fusionado. Reemplaza el contenido por:

```
Add limpieza.txt with lines a, b, c
```

Guarda y cierra.

**Verifica, mergea y sube:**

```bash
git log --oneline
# Los 3 commits son ahora 1 solo, con el mensaje limpio.

cat limpieza.txt
# El archivo tiene las tres líneas: el contenido se preservó.

git switch main
git merge feature-limpieza
git branch -d feature-limpieza
git push
```

📌 Otras operaciones del rebase interactivo:
- `reword` (r): cambia solo el mensaje.
- `fixup` (f): como squash pero descarta el mensaje del commit fusionado.
- `edit` (e): pausa en ese commit para modificarlo.
- `drop` (d): elimina el commit.

---

## Ejercicio 21 — Cherry-pick: traer commits sueltos

**Objetivo:** Aplicar un commit concreto de otra rama sin mergearla entera.

**Crea una rama con un commit interesante:**

```bash
git switch -c rama-fuente

echo "función útil" > util.txt
git add util.txt
git commit -m "Add util.txt with useful content"

git log --oneline
# Anota el hash corto del commit que acabas de crear.
```

**Vuelve a main y aplica ese commit:**

```bash
git switch main

git cherry-pick <HASH_DEL_COMMIT>
# Reemplaza <HASH_DEL_COMMIT> por el hash real (los 7 primeros caracteres bastan).
# Crea un commit NUEVO en main con los mismos cambios.

ls
# util.txt ahora está también en main.

git log --oneline
# Verás el commit aplicado al final del historial de main.

git branch -D rama-fuente
# `-D` borra a la fuerza (la rama no estaba mergeada formalmente).

git push
```

📌 Útil para llevar un hotfix de una rama a otra sin arrastrar el resto de cambios.

---

## Ejercicio 22 — git reflog: recuperar lo "perdido"

**Objetivo:** Recuperar commits aparentemente borrados.

⚠️ Este ejercicio reescribe historia local. **No hagas push** hasta haber recuperado el commit.

**Provoca una pérdida controlada:**

```bash
echo "muy importante" > importante.txt
git add importante.txt
git commit -m "Add importante.txt"

git log --oneline
# Confirma que existe.

git reset --hard HEAD~1
# Borramos el commit a la fuerza.

git log --oneline
# Ya no está.

ls
# El archivo importante.txt tampoco está.
```

**Recupera con reflog:**

```bash
git reflog
# Muestra el LOG DE MOVIMIENTOS de HEAD, incluidos los "borrados".
# Verás algo como:
#   abc1234 HEAD@{0}: reset: moving to HEAD~1
#   def5678 HEAD@{1}: commit: Add importante.txt
# El que necesitamos es HEAD@{1} (justo antes del reset).

git reset --hard HEAD@{1}
# Volvemos al estado anterior al reset.

ls
# importante.txt está de vuelta.

git log --oneline
# El commit también.

git push
```

📌 El reflog guarda movimientos locales durante ~90 días. Es **el seguro de vida de Git**: casi nada se pierde de verdad si haces commits frecuentes.

---

## Ejercicio 23 — Bisect: encontrar el commit que rompió algo

**Objetivo:** Localizar el commit que introdujo un bug mediante búsqueda binaria.

**Trabaja en una rama dedicada:**

```bash
git switch -c rama-bisect
```

**Prepara commits, uno de ellos "malo":**

```bash
echo "v1" > app.txt
git add app.txt
git commit -m "v1 working"

echo "v2" > app.txt
git add app.txt
git commit -m "v2 working"

echo "v3 BUG" > app.txt
git add app.txt
git commit -m "v3"

echo "v4" > app.txt
git add app.txt
git commit -m "v4"

echo "v5" > app.txt
git add app.txt
git commit -m "v5"
```

**Inicia bisect:**

```bash
git bisect start
# Activa el modo bisect.

git bisect bad HEAD
# Marca el commit actual como malo (tiene el bug).

git bisect good HEAD~4
# Marca un commit antiguo como bueno (no tenía el bug).
# Git te coloca en un commit intermedio para que pruebes.
```

**Inspecciona `app.txt`:**

```bash
cat app.txt
```

Si contiene "BUG", el commit es malo; si no, es bueno:

```bash
# Si es malo:
git bisect bad

# Si es bueno:
git bisect good
```

Repite hasta que Git anuncie el commit culpable con un mensaje como `<hash> is the first bad commit`.

**Termina y limpia:**

```bash
git bisect reset
# Sale del modo bisect y vuelve a la rama y commit donde estabas al iniciarlo.

git switch main
git branch -D rama-bisect
```

📌 Con 1000 commits, bisect encuentra el culpable en ~10 pasos (logarítmico).

---

## Ejercicio 24 — Trabajar con múltiples ramas remotas

**Objetivo:** Crear, subir y eliminar ramas remotas.

```bash
git switch -c feature/nueva-funcion
# Las barras en nombres de rama son convención común (feature/, bugfix/, hotfix/).

echo "código nuevo" > funcion.txt
git add funcion.txt
git commit -m "Add new function"

git push -u origin feature/nueva-funcion
# Sube la rama Y establece upstream con `-u`. La primera vez es necesario.

git branch -a
# `-a` (all) lista ramas locales Y remotas.
# Verás algo como:
#   * feature/nueva-funcion
#     main
#     remotes/origin/feature/nueva-funcion
#     remotes/origin/main

git switch main

git push origin --delete feature/nueva-funcion
# Borra la rama en el REMOTO. Equivalente a `git push origin :feature/nueva-funcion`.

git branch -d feature/nueva-funcion
# Borra la rama LOCAL.

git fetch --prune
# `--prune` borra las referencias locales a ramas remotas que ya no existen.
# Sin esto, `git branch -a` seguiría mostrando "remotes/origin/feature/nueva-funcion".

git branch -a
# Confirma que la rama desapareció en todas partes.
```

---

## Ejercicio 25 — Flujo completo simulando un equipo

**Objetivo:** Integrar todo lo aprendido en un flujo realista de equipo, sobre el mismo repo.

**Escenario:** Añades una funcionalidad mientras "otro desarrollador" trabaja en `main`.

**Paso 1 — Asegúrate de estar actualizado:**

```bash
git switch main
git pull
# Trae lo último del remoto antes de empezar. Siempre.
```

**Paso 2 — Crea una rama de feature:**

```bash
git switch -c feature/lista-tareas
```

**Crea `tareas.md`:**

```markdown
# Lista de tareas

- [ ] Aprender Git
- [ ] Practicar branches
- [ ] Dominar rebase
```

**Commit:**

```bash
git add tareas.md
git commit -m "Add initial task list"
```

**Paso 3 — Simula que un compañero modifica main.** Cambia a main y añade al final de `README.md`:

```markdown

## Actualización del compañero
Se ha añadido información mientras trabajabas en tu feature.
```

```bash
git switch main
# Aquí harías la edición manual del README.md.

git add README.md
git commit -m "Teammate update on main"
git push

git switch feature/lista-tareas
# Vuelve a tu rama.
```

**Paso 4 — Pon tu rama al día con main usando rebase:**

```bash
git rebase main
# Reaplica tus commits sobre el main actualizado. Historial lineal.

git log --oneline --graph --all
```

**Paso 5 — Añade más cambios y limpia el historial antes de mergear:**

```bash
echo "- [ ] Aprender cherry-pick" >> tareas.md
git add tareas.md
git commit -m "Add cherry-pick task"

echo "- [ ] Aprender bisect" >> tareas.md
git add tareas.md
git commit -m "Add bisect task"

git rebase -i HEAD~2
# En el editor, marca el segundo commit como `squash` para fusionarlos.
```

**Paso 6 — Sube y mergea:**

```bash
git push -u origin feature/lista-tareas
# Sube la rama al remoto. Útil para revisión de código (PRs en GitHub).

git switch main

git merge --no-ff feature/lista-tareas -m "Merge feature/lista-tareas into main"
# `--no-ff` fuerza un merge commit aunque podría ser fast-forward.
# Preserva el hecho de que existió una rama, útil para la trazabilidad.

git log --oneline --graph --all
git push

git push origin --delete feature/lista-tareas
# Borra la rama en el REMOTO primero. Si borras la local antes, `-d` fallará
# porque Git seguirá viendo origin/feature/lista-tareas con commits no integrados en el remoto.

git branch -d feature/lista-tareas
# Ahora sí permite `-d`: la remota ya no existe, no hay riesgo de pérdida.
```

**Paso 7 — Marca la versión final como release:**

```bash
git tag -a v2.0 -m "Release 2.0 — guía de Git completada"
# Tag anotado con mensaje.

git push origin v2.0
# Sube SOLO este tag al remoto.
```

---

## Consejos finales

1. **Antes de cualquier comando de impacto** (`reset`, `rebase`, `checkout`, `merge`), ejecuta `git status` y `git log --oneline --graph --all`. Saber exactamente dónde estás evita el 90% de los líos.
2. **Nunca hagas `rebase`, `amend` ni `push --force`** en ramas compartidas.
3. **El reflog es tu amigo.** Casi nada se pierde de verdad si haces commits frecuentes.
4. **Commits pequeños y atómicos.** Un commit, un cambio lógico. Mensajes claros en imperativo.
5. **Usa ramas para todo cambio no trivial.** Mantén `main` siempre estable.
6. **Lee la salida de Git.** A veces te dice exactamente qué hacer; otras veces, qué NO hacer.

¡Feliz versionado!
