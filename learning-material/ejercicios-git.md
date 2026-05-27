# Ejercicios de Git — De Principiante a Avanzado

Esta guía contiene una progresión completa de ejercicios prácticos de Git que se realizan **todos sobre el mismo repositorio**. Empezarás creándolo localmente y, al llegar al ejercicio 17, lo conectarás con un remoto en GitHub. A partir de ahí, todo el trabajo seguirá en ese mismo repo local + remoto.

**Convención:** Crea una carpeta llamada `git-practica` donde prefieras. Ese será **tu repositorio único** durante toda la guía. No crees carpetas nuevas por ejercicio salvo cuando se indique explícitamente (ejercicio 18, donde se clona en otro directorio para simular un segundo entorno).

---

## Ejercicio 1 — Configuración inicial de Git

**Objetivo:** Configurar tu identidad en Git (solo se hace una vez por máquina).

**Comandos:**

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu.email@ejemplo.com"
git config --global init.defaultBranch main
git config --global core.editor "nano"
```

**Verificación:**

```bash
git config --list
```

Deberías ver tu nombre y email en la salida.

---

## Ejercicio 2 — Crear el repositorio de prácticas

**Objetivo:** Inicializar el repositorio que usarás durante toda la guía.

**Comandos:**

```bash
mkdir git-practica
cd git-practica
git init
git status
```

**Resultado esperado:** Mensaje "On branch main / No commits yet / nothing to commit".

> 📌 **A partir de aquí, todos los comandos se ejecutan dentro de `git-practica`.**

---

## Ejercicio 3 — Tu primer commit

**Objetivo:** Crear un archivo, añadirlo al área de staging y hacer commit.

**Archivo a crear:** `README.md`

```markdown
# Mi Primer Repositorio

Este es mi primer proyecto en Git. Estoy aprendiendo control de versiones.
```

**Comandos:**

```bash
git status
git add README.md
git status
git commit -m "Initial commit: add README"
git log
```

---

## Ejercicio 4 — Modificar y commitear cambios

**Objetivo:** Entender el ciclo modificar → stage → commit.

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
git add README.md
git diff --staged
git commit -m "Add learning section to README"
git log --oneline
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
# Archivos de entorno
*.env

# Logs
*.log

# Notas personales
notas.txt
```

**Comandos:**

```bash
git status
git add .gitignore
git commit -m "Add .gitignore"
git status
```

`git status` ya no debería mostrar los archivos ignorados.

---

## Ejercicio 6 — Deshacer cambios antes de commitear

**Objetivo:** Aprender a descartar cambios en el directorio de trabajo y en staging.

**Modifica `README.md`** añadiendo al final:

```markdown

## Esta línea es un error que voy a deshacer
```

**Comandos para descartar en el directorio de trabajo:**

```bash
git status
git diff
git restore README.md
git status
```

**Ahora practica deshacer en staging.** Vuelve a modificar el archivo igual que antes, luego:

```bash
git add README.md
git status
git restore --staged README.md
git status
git restore README.md
```

---

## Ejercicio 7 — Ver el historial e inspeccionar commits

**Objetivo:** Explorar el historial del repositorio.

Antes de empezar, haz un commit más. Añade este texto al final de `README.md`:

```markdown

## Comandos básicos aprendidos
- git init
- git add
- git commit
- git status
- git log
```

**Comandos:**

```bash
git add README.md
git commit -m "Document basic commands learned"

# Variaciones de log
git log
git log --oneline
git log --oneline --graph --all
git log -p
git log --stat
git show HEAD
git show HEAD~1
```

---

## Ejercicio 8 — Crear y cambiar entre ramas

**Objetivo:** Trabajar con ramas (branches).

**Comandos:**

```bash
git branch
git branch feature-saludo
git branch
git switch feature-saludo
git branch
```

**Crea un nuevo archivo `saludo.txt`:**

```
¡Hola mundo desde la rama feature-saludo!
```

**Comandos:**

```bash
git add saludo.txt
git commit -m "Add saludo.txt on feature branch"
git switch main
ls
```

Observa cómo `saludo.txt` desaparece al volver a `main` (porque solo existe en la otra rama).

---

## Ejercicio 9 — Hacer merge de una rama

**Objetivo:** Integrar los cambios de una rama en otra (fast-forward merge).

**Comandos (desde la rama `main`):**

```bash
git switch main
git log --oneline --graph --all
git merge feature-saludo
git log --oneline --graph --all
ls
```

**Eliminar la rama ya fusionada:**

```bash
git branch -d feature-saludo
git branch
```

---

## Ejercicio 10 — Merge con conflictos

**Objetivo:** Resolver un conflicto de merge.

**Paso 1 — Desde `main`, edita `README.md`** y reemplaza la primera línea por:

```markdown
# Mi Primer Repositorio — Edición desde MAIN
```

**Comandos:**

```bash
git add README.md
git commit -m "Update title from main"
```

**Paso 2 — Crea una rama desde el commit anterior y modifica la MISMA línea de forma diferente:**

```bash
git switch -c feature-titulo HEAD~1
```

**Edita `README.md`** y reemplaza la primera línea por:

```markdown
# Mi Primer Repositorio — Edición desde FEATURE
```

**Comandos:**

```bash
git add README.md
git commit -m "Update title from feature branch"
git switch main
git merge feature-titulo
```

**Verás un conflicto.** Abre `README.md` y verás algo parecido a:

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

**Comandos para finalizar:**

```bash
git status
git add README.md
git commit -m "Resolve merge conflict in README title"
git log --oneline --graph --all
git branch -d feature-titulo
```

---

## Ejercicio 11 — Cambios sin commitear que bloquean un switch

**Objetivo:** Entender la advertencia de Git al cambiar de rama con cambios sin commitear que entran en conflicto con la rama destino.

> **Concepto clave:** Si tienes modificaciones en el directorio de trabajo y la rama a la que quieres cambiar tiene una versión distinta del MISMO archivo, Git puede negarse a cambiar de rama para no perder tus cambios. Esta es una protección contra pérdida de datos.

**Paso 1 — Crea una rama nueva y modifica `README.md`:**

```bash
git switch -c experimento-titulo
```

**Edita `README.md`** y reemplaza la primera línea por:

```markdown
# Mi Primer Repositorio — Versión EXPERIMENTAL
```

**Commitea ese cambio:**

```bash
git add README.md
git commit -m "Experimental title change"
```

**Paso 2 — Vuelve a `main`:**

```bash
git switch main
```

**Paso 3 — Modifica `README.md` SIN COMMITEAR** (cambia la primera línea por):

```markdown
# Mi Primer Repositorio — Trabajo en progreso sin commitear
```

**NO hagas `git add` ni `git commit`.**

**Paso 4 — Intenta cambiar a `experimento-titulo`:**

```bash
git switch experimento-titulo
```

**Verás un error similar a:**

```
error: Your local changes to the following files would be overwritten by checkout:
        README.md
Please commit your changes or stash them before you switch branches.
Aborting
```

Git te protege: si te dejara cambiar de rama, perderías tus modificaciones locales porque la otra rama tiene una versión diferente del mismo archivo.

**Soluciones posibles:**

```bash
# Opción A: Commitear los cambios antes de cambiar de rama
git add README.md
git commit -m "WIP: title in progress"
git switch experimento-titulo

# Opción B: Descartar los cambios (los pierdes)
# git restore README.md
# git switch experimento-titulo

# Opción C: Guardarlos temporalmente con stash (siguiente ejercicio)
```

Para este ejercicio, usa la **Opción A** y vuelve a `main` al final. Luego borra la rama experimental para no acumular ramas:

```bash
git switch main
git branch -D experimento-titulo
```

---

## Ejercicio 12 — git stash: guardar cambios temporalmente

**Objetivo:** Usar `stash` para guardar cambios sin commitearlos.

**Paso 1 — Desde `main`, modifica `README.md`** añadiendo al final:

```markdown

## Sección en construcción
Esta sección está sin terminar.
```

**Paso 2 — Guarda los cambios temporalmente con stash:**

```bash
git status
git stash
git status
git stash list
```

**Paso 3 — Recupera el stash:**

```bash
git stash pop
git status
```

**Practica también las variantes:**

```bash
git stash push -m "Trabajo en sección README"
git stash list
git stash show -p stash@{0}
git stash drop stash@{0}
```

Antes de continuar, deshaz los cambios pendientes en `README.md` para dejarlo limpio:

```bash
git restore README.md
git status
```

---

## Ejercicio 13 — Modificar el último commit (amend)

**Objetivo:** Corregir el último commit (mensaje o contenido).

**Cambia el mensaje del último commit:**

```bash
git log --oneline
git commit --amend -m "Document basic commands and learning notes"
git log --oneline
```

**Añadir un archivo olvidado al último commit:**

Crea un archivo `notas-correccion.txt`:

```
Olvidé añadir este archivo al commit anterior.
```

**Comandos:**

```bash
git add notas-correccion.txt
git commit --amend --no-edit
git log --stat -1
```

> ⚠️ **Importante:** `--amend` solo es seguro en commits que NO has compartido (no has hecho push). Reescribe el historial. Como en este punto aún no tenemos remoto, no hay problema.

---

## Ejercicio 14 — Revertir un commit (git revert)

**Objetivo:** Deshacer un commit creando otro commit que lo invierta. Es seguro incluso con commits ya compartidos.

**Paso 1 — Crea un archivo "malo":**

`archivo-erroneo.txt`:
```
Este archivo no debería haber existido nunca.
```

**Comandos:**

```bash
git add archivo-erroneo.txt
git commit -m "Add archivo-erroneo.txt"
git log --oneline
```

**Paso 2 — Revierte ese commit:**

```bash
git revert HEAD
```

Se abrirá el editor con un mensaje de commit predefinido. Guarda y cierra. El archivo desaparecerá y habrá un nuevo commit del tipo "Revert ...".

```bash
git log --oneline
ls
```

---

## Ejercicio 15 — git reset: soft, mixed y hard

**Objetivo:** Entender las tres variantes de reset.

**Prepara tres commits de prueba:**

```bash
echo "linea 1" > prueba-reset.txt
git add prueba-reset.txt
git commit -m "reset test: commit 1"

echo "linea 2" >> prueba-reset.txt
git add prueba-reset.txt
git commit -m "reset test: commit 2"

echo "linea 3" >> prueba-reset.txt
git add prueba-reset.txt
git commit -m "reset test: commit 3"

git log --oneline
```

**Reset --soft (deshace commit pero mantiene los cambios staged):**

```bash
git reset --soft HEAD~1
git status
git log --oneline
```

**Reset --mixed (por defecto: deshace commit y unstage):**

```bash
git reset --mixed HEAD~1
git status
git log --oneline
```

**Reset --hard (deshace todo, ¡PIERDES los cambios!):**

```bash
git add prueba-reset.txt
git commit -m "reset test: recompose"
git reset --hard HEAD~1
git status
cat prueba-reset.txt
```

> ⚠️ `--hard` es destructivo. Úsalo con cuidado.

Limpia el archivo de prueba antes de continuar:

```bash
git rm prueba-reset.txt
git commit -m "Remove reset test file"
```

---

## Ejercicio 16 — Etiquetas (tags)

**Objetivo:** Marcar versiones específicas del proyecto.

**Comandos:**

```bash
git tag v1.0
git tag -a v1.1 -m "Versión 1.1 con sección de comandos básicos"
git tag
git show v1.1
git tag -d v1.0
```

**Listar y filtrar:**

```bash
git tag -l "v1.*"
```

---

## Ejercicio 17 — Conectar el repositorio con un remoto (GitHub)

**Objetivo:** Subir tu repositorio local a GitHub. **A partir de este ejercicio trabajaremos con el mismo repo, pero ahora con un remoto asociado.**

**Paso 1:** Crea un repositorio vacío en GitHub llamado `git-practica` (**sin** README, **sin** .gitignore, **sin** licencia, para que no haya conflicto inicial).

**Paso 2 — Conecta el repositorio local con el remoto y sube todo:**

```bash
git remote add origin https://github.com/TU_USUARIO/git-practica.git
git remote -v
git branch -M main
git push -u origin main
git push origin --tags
```

**Verifica en GitHub** que aparecen todos tus commits y la etiqueta `v1.1`.

---

## Ejercicio 18 — Clonar, fetch y pull

**Objetivo:** Simular trabajar desde otro ordenador clonando el repo en una segunda carpeta.

**Paso 1 — Clona en una carpeta hermana** (sal temporalmente del repo):

```bash
cd ..
git clone https://github.com/TU_USUARIO/git-practica.git git-practica-clon
cd git-practica-clon
git log --oneline
```

**Simula trabajo desde el clon.** Edita `README.md` añadiendo al final:

```markdown

## Edición desde el clon
Este cambio se hizo en el segundo entorno.
```

**Sube los cambios:**

```bash
git add README.md
git commit -m "Edit from clone"
git push
```

**Paso 2 — Vuelve al repositorio principal y trae los cambios:**

```bash
cd ../git-practica
git fetch
git status
git pull
git log --oneline
```

> 📌 De ahora en adelante seguimos trabajando en `git-practica`. La carpeta `git-practica-clon` queda solo como simulación, no es necesario volver a usarla.

---

## Ejercicio 19 — Rebase básico

**Objetivo:** Reescribir el historial para tenerlo lineal.

**Paso 1 — Crea una rama desde un commit anterior:**

```bash
git switch -c feature-rebase HEAD~2
```

**Crea un archivo `feature-rebase.txt`:**

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
git rebase main
git log --oneline --graph --all
```

**Merge fast-forward de nuevo a main y sube:**

```bash
git switch main
git merge feature-rebase
git branch -d feature-rebase
git push
```

> ⚠️ **Regla de oro del rebase:** NO hagas rebase de commits ya pusheados a una rama compartida. Reescribe el historial y rompería el de los demás. Aquí lo hicimos sobre commits locales antes de pushearlos, que sí es seguro.

---

## Ejercicio 20 — Rebase interactivo: limpiar el historial

**Objetivo:** Combinar, reordenar o editar commits con `rebase -i` **antes de pushearlos**.

**Crea varios commits "sucios" en una rama de feature (no en main):**

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

Guarda y cierra. Se abrirá otro editor para escribir el mensaje del commit fusionado. Reemplaza por:

```
Add limpieza.txt with lines a, b, c
```

**Verifica, mergea a main y sube:**

```bash
git log --oneline
cat limpieza.txt
git switch main
git merge feature-limpieza
git branch -d feature-limpieza
git push
```

---

## Ejercicio 21 — Cherry-pick: traer commits sueltos

**Objetivo:** Aplicar un commit concreto de otra rama.

**Crea una rama con un commit interesante:**

```bash
git switch -c rama-fuente
echo "función útil" > util.txt
git add util.txt
git commit -m "Add util.txt with useful content"
git log --oneline
```

**Anota el hash del commit y vuelve a main:**

```bash
git switch main
git cherry-pick <HASH_DEL_COMMIT>
ls
git log --oneline
git branch -D rama-fuente
git push
```

---

## Ejercicio 22 — git reflog: recuperar lo "perdido"

**Objetivo:** Usar reflog para recuperar commits aparentemente perdidos.

> ⚠️ Este ejercicio reescribe el historial local. **No hagas push al final** hasta que hayas recuperado el commit.

**Provoca una pérdida controlada:**

```bash
echo "muy importante" > importante.txt
git add importante.txt
git commit -m "Add importante.txt"
git log --oneline

git reset --hard HEAD~1
git log --oneline
ls
```

El archivo y el commit parecen perdidos. **Recupéralos con reflog:**

```bash
git reflog
git reset --hard HEAD@{1}
ls
git log --oneline
git push
```

> 💡 `reflog` guarda referencias locales durante ~90 días por defecto. Es tu red de seguridad.

---

## Ejercicio 23 — Bisect: encontrar el commit que rompió algo

**Objetivo:** Localizar el commit que introdujo un bug mediante búsqueda binaria.

**Trabaja en una rama dedicada para no ensuciar el historial de main:**

```bash
git switch -c rama-bisect
```

**Prepara una serie de commits, algunos "buenos" y uno "malo":**

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
git bisect bad HEAD
git bisect good HEAD~4
```

Git te pondrá en un commit intermedio. **Inspecciona `app.txt`:**

```bash
cat app.txt
```

Si contiene "BUG", márcalo como malo; si no, como bueno:

```bash
# Si es malo:
git bisect bad
# Si es bueno:
git bisect good
```

Repite hasta que Git anuncie el commit culpable. **Termina y limpia:**

```bash
git bisect reset
git switch main
git branch -D rama-bisect
```

---

## Ejercicio 24 — Trabajar con múltiples ramas remotas

**Objetivo:** Crear, subir y eliminar ramas remotas.

**Comandos:**

```bash
git switch -c feature/nueva-funcion
echo "código nuevo" > funcion.txt
git add funcion.txt
git commit -m "Add new function"
git push -u origin feature/nueva-funcion

git branch -a
git switch main
git push origin --delete feature/nueva-funcion
git branch -d feature/nueva-funcion
git fetch --prune
git branch -a
```

---

## Ejercicio 25 — Flujo completo simulando un equipo

**Objetivo:** Integrar todo lo aprendido en un flujo realista de trabajo en equipo, usando el repositorio que llevas construyendo desde el ejercicio 2.

**Escenario:** Tienes que añadir una funcionalidad mientras "otro desarrollador" trabaja en `main` (simularemos ese otro dev haciendo cambios desde `git-practica-clon` o directamente en main).

**Paso 1 — Asegúrate de estar actualizado:**

```bash
git switch main
git pull
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

**Paso 3 — Simula que otro desarrollador hizo cambios en main.** Cambia a `main` y modifica `README.md` añadiendo al final:

```markdown

## Actualización del compañero
Se ha añadido información mientras trabajabas en tu feature.
```

```bash
git switch main
# (edita README.md como se indica arriba)
git add README.md
git commit -m "Teammate update on main"
git push
git switch feature/lista-tareas
```

**Paso 4 — Pon tu rama al día con main usando rebase:**

```bash
git rebase main
git log --oneline --graph --all
```

**Paso 5 — Añade más cambios y limpia el historial:**

```bash
echo "- [ ] Aprender cherry-pick" >> tareas.md
git add tareas.md
git commit -m "Add cherry-pick task"

echo "- [ ] Aprender bisect" >> tareas.md
git add tareas.md
git commit -m "Add bisect task"

git rebase -i HEAD~2
```

(En el editor, marca el segundo como `squash`.)

**Paso 6 — Sube y mergea:**

```bash
git push -u origin feature/lista-tareas
git switch main
git merge --no-ff feature/lista-tareas -m "Merge feature/lista-tareas into main"
git log --oneline --graph --all
git push
git branch -d feature/lista-tareas
git push origin --delete feature/lista-tareas
```

**Paso 7 — Marca la versión final como release:**

```bash
git tag -a v2.0 -m "Release 2.0 — guía de Git completada"
git push origin v2.0
```

---

## Resumen de comandos clave

| Categoría | Comandos |
|---|---|
| **Configuración** | `git config`, `git init`, `git clone` |
| **Cambios** | `git status`, `git add`, `git commit`, `git diff`, `git restore` |
| **Historial** | `git log`, `git show`, `git reflog`, `git blame` |
| **Ramas** | `git branch`, `git switch`, `git merge`, `git rebase` |
| **Remoto** | `git remote`, `git fetch`, `git pull`, `git push` |
| **Deshacer** | `git restore`, `git revert`, `git reset`, `git commit --amend` |
| **Temporal** | `git stash`, `git stash pop`, `git stash list` |
| **Avanzado** | `git cherry-pick`, `git bisect`, `git rebase -i`, `git tag` |

---

## Consejos finales

1. **Antes de cambiar de rama**, revisa siempre `git status`. Si tienes cambios sin commitear que tocan los mismos archivos que la rama destino, Git te lo advertirá para evitar pérdida de datos. Usa `commit`, `stash` o `restore` según el caso.
2. **Nunca hagas rebase ni amend de commits ya pusheados** a una rama compartida.
3. **El reflog es tu amigo:** rara vez algo se pierde de forma irrecuperable en Git si haces commit con frecuencia.
4. **Mensajes de commit claros:** verbo en imperativo + qué cambia ("Add login form", no "added login form").
5. **Commits pequeños y atómicos:** un commit, un cambio lógico.

¡Buena suerte y feliz versionado!
