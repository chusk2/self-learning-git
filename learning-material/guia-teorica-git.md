# Guía teórica de Git

Esta guía explica los conceptos que usarás en los ejercicios prácticos. Léela en orden la primera vez; después úsala como referencia. Cada sección incluye no solo el "cómo" sino el "por qué": qué hace Git por dentro, cuándo aplica cada comando y qué significa cada flag.

---

## 1. Qué es Git y cómo piensa

Git es un sistema de control de versiones **distribuido**: cada copia del repositorio contiene la historia completa, no depende de un servidor central para funcionar. A diferencia de sistemas anteriores (SVN, CVS), Git no guarda diferencias entre versiones, sino **fotos completas** del proyecto en cada commit (técnicamente, snapshots con deduplicación interna por contenido).

### Las tres zonas de Git

Todo cambio que hagas vive en una de estas tres zonas:

1. **Working directory** (directorio de trabajo): tus archivos tal cual los ves en el sistema operativo. Aquí editas.
2. **Staging area** (índice): una zona intermedia donde "preparas" lo que quieres incluir en el próximo commit. Pones cosas aquí con `git add`.
3. **Repositorio** (.git/): el historial inmutable de commits. Pasas del staging al repo con `git commit`.

Entender estas tres zonas es la clave para entender el resto. Casi cada comando de Git mueve datos entre ellas.

### El grafo de commits

El historial de Git no es una lista, es un **grafo dirigido acíclico (DAG)**. Cada commit apunta a su padre (o padres, si es un merge). Las ramas son simplemente punteros móviles a un commit; `HEAD` es un puntero a la rama actual (o a un commit directamente, en estado "detached HEAD").

Cuando entiendes que una rama es solo una etiqueta sobre un commit, comandos como `reset`, `rebase` o `cherry-pick` dejan de parecer mágicos.

---

## 2. Configuración: `git config`

Git lee configuración de tres lugares, en orden de prioridad ascendente:

| Nivel | Ubicación | Flag |
|---|---|---|
| Sistema | `/etc/gitconfig` | `--system` |
| Usuario | `~/.gitconfig` o `~/.config/git/config` | `--global` |
| Repo local | `.git/config` dentro del repo | (sin flag) o `--local` |

Configuración mínima antes de tu primer commit:

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu@email.com"
```

**¿Por qué `--global`?** Sin esa flag, Git intentaría escribir en `.git/config`, pero si aún no estás dentro de un repo, fallaría. Con `--global` queda guardado para tu usuario en cualquier repo.

Otros ajustes útiles:

- `git config --global init.defaultBranch main` — al hacer `git init`, la rama inicial se llamará `main` en vez del antiguo `master`.
- `git config --global core.editor "nano"` — qué editor abre Git cuando necesita uno (mensajes de commit largos, rebases interactivos, etc.).
- `git config --list` — muestra toda la configuración efectiva.
- `git config --list --show-origin` — además, te dice de qué archivo viene cada valor (útil para depurar configuración heredada).

---

## 3. Crear y entender un repositorio: `git init`

```bash
git init
```

Crea una carpeta oculta `.git/` dentro del directorio actual. Esa carpeta es **el repositorio**: contiene los objetos (commits, árboles, blobs), las referencias (ramas, tags) y la configuración local. El resto de archivos del directorio son tu **working directory**.

Borrar `.git/` equivale a "desgitificar" la carpeta: los archivos siguen ahí, pero la historia se pierde.

---

## 4. El ciclo básico: `add`, `commit`, `status`, `log`

### `git status`

El comando más usado. Te dice:

- Qué archivos están modificados pero no añadidos al staging.
- Qué archivos están en staging listos para commitear.
- Qué archivos no están rastreados por Git (untracked).
- En qué rama estás.

Variantes:

- `git status -s` (short) — salida compacta, una línea por archivo con dos columnas: estado en staging | estado en working directory.

### `git add`

Mueve cambios del working directory al staging area.

- `git add archivo.txt` — añade un archivo concreto.
- `git add .` — añade todo lo cambiado y los archivos nuevos del directorio actual hacia abajo.
- `git add -A` — añade todos los cambios del repo entero, incluidos los borrados.
- `git add -p` — modo interactivo: te muestra cada cambio (hunk) y pregunta si añadirlo o no. Excelente para hacer commits limpios cuando has tocado muchas cosas a la vez.

### `git commit`

Toma lo que hay en staging y lo convierte en un commit nuevo.

- `git commit -m "Mensaje"` — commit con mensaje en línea.
- `git commit` — abre el editor configurado para escribir un mensaje (recomendado para mensajes largos).
- `git commit -a -m "Mensaje"` — combina `add` + `commit` para archivos **ya rastreados**. Los archivos nuevos no se incluyen.
- `git commit --amend` — modifica el último commit en lugar de crear uno nuevo (más en sección 8).
- `git commit --amend --no-edit` — igual, pero sin abrir el editor (útil cuando solo añadiste un archivo olvidado).

**Anatomía de un mensaje de commit bien escrito:**

```
Verbo en imperativo y resumen corto (≤50 caracteres)

Cuerpo opcional explicando el "por qué" del cambio,
no el "qué" (eso ya se ve en el diff).
```

### `git log`

Muestra el historial de commits. Las flags importantes:

- `--oneline` — un commit por línea, solo hash corto y mensaje.
- `--graph` — dibuja el grafo de ramas con líneas ASCII.
- `--all` — incluye todas las ramas, no solo la actual.
- `--decorate` — muestra nombres de ramas y tags junto a los commits (suele estar activado por defecto en versiones modernas).
- `-p` — muestra el diff de cada commit.
- `--stat` — resumen de qué archivos cambiaron y cuántas líneas.
- `-n` — limita al número de commits indicado (ej. `git log -5`).

Combinación recomendada para una vista panorámica:

```bash
git log --oneline --graph --all --decorate
```

### `git diff`

Te enseña los cambios:

- `git diff` — entre working directory y staging (cambios sin añadir).
- `git diff --staged` (o `--cached`) — entre staging y último commit (lo que se commitearía ahora).
- `git diff HEAD` — entre working directory y último commit (todo lo no commiteado).
- `git diff <commit1> <commit2>` — entre dos commits cualesquiera.

---

## 5. Ignorar archivos: `.gitignore`

El archivo `.gitignore` (texto plano, en la raíz del repo o en subcarpetas) le dice a Git qué archivos no rastrear. Patrones útiles:

```
*.log              # cualquier archivo .log
secreto.env        # archivo exacto
build/             # carpeta entera
!build/keep.txt    # excepción: este sí se rastrea
**/temp            # carpetas 'temp' a cualquier profundidad
```

**Punto importante:** `.gitignore` solo afecta a archivos **no rastreados aún**. Si un archivo ya fue commiteado, añadirlo al `.gitignore` no lo elimina; tienes que sacarlo del índice con `git rm --cached archivo`.

---

## 6. Deshacer cambios: `restore`, `reset`, `revert`

Tres comandos diferentes para deshacer, según en qué fase esté el cambio y si quieres reescribir historia o no.

### `git restore` — para cambios no commiteados

- `git restore archivo` — descarta los cambios del working directory para ese archivo, volviendo al estado del staging (o del último commit si no hay nada en staging). **Destructivo: pierdes los cambios.**
- `git restore --staged archivo` — saca el archivo del staging, pero **mantiene** los cambios en el working directory. Útil cuando hiciste `git add` por error.
- `git restore --staged --worktree archivo` — combinación: lo saca del staging y descarta del working directory.
- `git restore --source=HEAD~2 archivo` — restaura el archivo a como estaba hace dos commits, dejando ese cambio en el working directory.

### `git reset` — mueve el puntero HEAD

`reset` se usa cuando quieres "olvidar" commits recientes. Mueve la rama actual a otro commit y, según la flag, decide qué hacer con los cambios.

- `git reset --soft HEAD~1` — mueve HEAD un commit atrás. Los cambios del commit deshecho quedan **en staging**, listos para recommitear. Útil para rehacer el último commit.
- `git reset --mixed HEAD~1` (es el default) — mueve HEAD atrás y los cambios quedan en el **working directory**, fuera del staging.
- `git reset --hard HEAD~1` — mueve HEAD atrás y **descarta todo**. Working directory y staging quedan como en el commit destino. ⚠️ Pierdes cambios no commiteados sin posibilidad de recuperarlos por las vías normales.

**Importante:** los commits "perdidos" tras un reset no desaparecen inmediatamente. Siguen accesibles vía `git reflog` durante unas semanas (sección 12).

### `git revert` — deshacer creando un nuevo commit

- `git revert <hash>` — crea un commit nuevo que **invierte** los cambios del commit indicado. No reescribe historia, así que es **seguro** en ramas compartidas con otras personas.

Diferencia clave: `reset` borra historia, `revert` la añade. Si ya hiciste push, usa `revert`.

### `git commit --amend` — rehacer el último commit

- `git commit --amend -m "Nuevo mensaje"` — cambia el mensaje del último commit.
- `git commit --amend --no-edit` — añade los cambios actualmente en staging al último commit sin cambiar su mensaje.

⚠️ Como `reset`, **amend reescribe historia**. No lo uses en commits ya pusheados a una rama compartida.

---

## 7. Ramas: `branch` y `switch`

Una rama es un puntero móvil a un commit. Crear una rama es prácticamente gratis en Git (no copia archivos, solo crea una referencia).

### Listar y crear

- `git branch` — lista las ramas locales; la actual aparece con un asterisco.
- `git branch -a` — incluye ramas remotas.
- `git branch -v` — añade el último commit de cada rama.
- `git branch nombre` — crea una rama nueva (sin cambiarte a ella).
- `git branch -d nombre` — borra la rama (solo si ya fue mergeada; protege contra pérdidas).
- `git branch -D nombre` — borra a la fuerza, mergeada o no.

### Cambiar de rama

Históricamente se usaba `git checkout`, que hacía demasiadas cosas. Git moderno divide eso en dos comandos más claros:

- `git switch nombre` — cambia a una rama existente.
- `git switch -c nombre` — crea la rama y cambia a ella (equivalente a `git branch nombre && git switch nombre`).
- `git switch -c nombre commit-base` — crea la rama partiendo de un commit concreto (no del actual).
- `git switch -` — vuelve a la rama anterior.

### La advertencia al cambiar de rama con cambios sin commitear

Si tienes modificaciones sin commitear en archivos que también difieren en la rama destino, Git **se niega** a cambiar:

```
error: Your local changes to the following files would be overwritten by checkout:
        archivo.txt
Please commit your changes or stash them before you switch branches.
```

Esta es una **protección contra pérdida de datos**. Tienes tres salidas:

1. **Commitear** los cambios antes de cambiar de rama.
2. **Stashearlos** (`git stash`, sección 9).
3. **Descartarlos** (`git restore archivo.txt`).

Si los cambios son en archivos que **no** existen en la rama destino, Git te deja pasar sin problema: no hay riesgo de pérdida.

---

## 8. Combinar ramas: `merge` y conflictos

### Tipos de merge

**Fast-forward:** si la rama destino no tiene commits nuevos desde que se creó la rama feature, Git simplemente "avanza" el puntero. No se crea commit de merge.

```
Antes:                       Después de merge fast-forward:
main:    A──B                main:    A──B──C──D
              \                       
feature:       C──D                   
```

**Merge commit (three-way merge):** si ambas ramas avanzaron en paralelo, Git crea un commit especial con **dos padres** que une las dos historias.

```
main:    A──B────────M (merge commit)
              \     /
feature:       C──D
```

Flags relevantes:

- `git merge feature` — merge normal (fast-forward si es posible, si no, merge commit).
- `git merge --no-ff feature` — fuerza un merge commit incluso si pudiera ser fast-forward. Útil para preservar la historia de que existió una rama.
- `git merge --squash feature` — aplica todos los cambios de la rama como un único commit nuevo, sin merge commit ni padres extra. Pierde la granularidad de la rama original.
- `git merge --abort` — cancela un merge en curso (por ejemplo, si los conflictos son demasiados y prefieres no continuar).

### Conflictos

Pasan cuando ambas ramas modifican **las mismas líneas** del mismo archivo. Git marca el conflicto así dentro del archivo:

```
<<<<<<< HEAD
versión de la rama actual
=======
versión de la rama que estás mergeando
>>>>>>> feature
```

Para resolverlo:

1. Edita el archivo dejando solo la versión final (puedes mezclar ambas).
2. Borra los marcadores `<<<<<<<`, `=======`, `>>>>>>>`.
3. `git add archivo` — marca el conflicto como resuelto.
4. `git commit` — completa el merge (Git ya tiene un mensaje predefinido).

Comandos útiles durante un conflicto:

- `git status` — qué archivos están en conflicto.
- `git diff` — muestra los conflictos con contexto.
- `git merge --abort` — si te arrepientes, vuelve al estado previo al merge.

---

## 9. Guardar trabajo temporal: `git stash`

`stash` empuja tus cambios sin commitear a una pila temporal, dejando el working directory limpio. Útil cuando necesitas cambiar de rama rápido pero no quieres hacer un commit "basura".

- `git stash` — guarda los cambios y limpia el working directory.
- `git stash push -m "mensaje"` — igual, con descripción.
- `git stash list` — lista los stashes guardados (`stash@{0}`, `stash@{1}`, ...).
- `git stash show stash@{0}` — muestra qué cambia ese stash; `-p` añade el diff completo.
- `git stash pop` — aplica el stash más reciente y lo borra de la pila.
- `git stash apply stash@{1}` — aplica un stash concreto pero **lo mantiene** en la pila.
- `git stash drop stash@{0}` — borra un stash sin aplicarlo.
- `git stash clear` — borra todos los stashes.
- `git stash -u` — incluye también los archivos no rastreados (untracked).

---

## 10. Tags: marcar versiones

Un tag es un puntero **fijo** a un commit (a diferencia de las ramas, que se mueven). Se usan para marcar releases.

- `git tag v1.0` — tag ligero (lightweight): solo un nombre apuntando a un commit.
- `git tag -a v1.0 -m "Versión 1.0"` — tag anotado: incluye autor, fecha y mensaje. Es el recomendado para releases.
- `git tag` — lista todos los tags.
- `git tag -l "v1.*"` — lista filtrando por patrón.
- `git show v1.0` — muestra info del tag y del commit asociado.
- `git tag -d v1.0` — borra el tag local.
- `git push origin v1.0` — sube un tag concreto al remoto.
- `git push origin --tags` — sube todos los tags locales.
- `git push origin --delete v1.0` — borra el tag en el remoto.

---

## 11. Trabajar con remotos: `remote`, `push`, `pull`, `fetch`, `clone`

### Conceptos

Un **remoto** es otra copia del repositorio, normalmente en un servidor (GitHub, GitLab, Bitbucket, otra máquina por SSH...). Por convención, el remoto principal se llama `origin`.

### Comandos clave

- `git remote add origin <url>` — asocia el repo local con uno remoto.
- `git remote -v` — lista los remotos configurados con sus URLs.
- `git remote remove origin` — elimina la asociación.

- `git clone <url>` — descarga un repo entero a una carpeta nueva, ya configurado con `origin` apuntando a esa URL.
- `git clone <url> nombre-carpeta` — clona en una carpeta con nombre personalizado.

- `git fetch` — descarga commits, ramas y tags del remoto **sin** modificar tu working directory ni tus ramas locales. Solo actualiza las "ramas remotas" (`origin/main`, etc.).
- `git pull` — combina `fetch` + `merge` en un solo comando. Trae cambios y los mezcla con tu rama actual.
- `git pull --rebase` — igual, pero usando rebase en vez de merge.

- `git push` — sube tus commits locales al remoto.
- `git push -u origin main` — la primera vez que pusheas una rama: `-u` (de `--set-upstream`) la asocia con su contraparte remota, así futuros `push` y `pull` ya no requieren argumentos.
- `git push origin nombre-rama` — sube una rama específica.
- `git push origin --delete nombre-rama` — borra una rama en el remoto.
- `git push --force` — fuerza el push, sobrescribiendo el remoto. ⚠️ Peligroso en ramas compartidas.
- `git push --force-with-lease` — versión más segura: solo fuerza si nadie pusheó cambios después de tu último fetch.

- `git fetch --prune` — al traer cambios, también borra las referencias locales a ramas remotas que ya no existen en el servidor.

### Ramas locales vs ramas remotas

Cuando haces `git fetch`, Git **no** modifica tus ramas locales. Solo actualiza referencias del tipo `origin/main`, `origin/feature/xyz`. Esto te permite ver qué tiene el remoto sin afectar tu trabajo. Para mezclar el remoto con tu rama local sí necesitas `merge` o `rebase` (o usar `pull`, que lo hace en un paso).

---

## 12. Reescribir y limpiar historia: `rebase`

`rebase` toma una serie de commits y los reaplica sobre otra base. El resultado es un historial **lineal**.

### Diferencia visual con merge

```
MERGE                          REBASE

A──B──────────M (merge)        A──B──C'──D' (lineal)
      \──C──D─/
```

Los commits `C'` y `D'` tras un rebase tienen el mismo contenido que `C` y `D` originales, pero son **commits nuevos con distinto hash**. Rebase reescribe historia.

### Rebase básico

Estando en la rama feature:

```bash
git rebase main
```

Mueve tus commits para que parezca que partieron del último commit de `main`. Si hay conflictos durante el rebase:

1. Git pausa y te muestra el conflicto.
2. Editas y resuelves los archivos.
3. `git add archivo`.
4. `git rebase --continue` — **no uses `git commit` aquí**; el `--continue` se encarga internamente.
5. Si quieres salir y cancelar todo el rebase, `git rebase --abort`.

### Rebase interactivo

`git rebase -i HEAD~3` abre un editor con los últimos 3 commits y te deja modificarlos. Las operaciones más usadas:

- `pick` — mantener el commit tal cual (default).
- `reword` (`r`) — cambiar el mensaje del commit.
- `squash` (`s`) — fusionarlo con el commit anterior (suma cambios y combina mensajes).
- `fixup` (`f`) — como squash, pero descarta el mensaje del commit fusionado.
- `edit` (`e`) — pausa el rebase en ese commit para modificarlo (cambiar archivos, dividirlo, etc.).
- `drop` (`d`) — eliminar el commit.
- Reordenar las líneas reordena los commits en la historia resultante.

### La regla de oro del rebase

> **Nunca hagas rebase de commits que ya hayas pusheado a una rama compartida.**

Como rebase reescribe historia, los demás colaboradores tendrán hashes distintos a los tuyos y el repo se romperá. El rebase es para limpiar **tu trabajo local** antes de pushearlo.

---

## 13. Operaciones puntuales: `cherry-pick` y `bisect`

### `git cherry-pick <hash>`

Aplica un commit concreto de otra rama sobre la rama actual, creando un commit nuevo con los mismos cambios. Útil cuando quieres llevarte solo un cambio sin mergear la rama entera (por ejemplo, un hotfix).

- `git cherry-pick <hash1> <hash2>` — varios commits seguidos.
- `git cherry-pick A..B` — rango (todos los commits entre A y B, sin incluir A).
- `git cherry-pick --abort` — cancela si hay conflictos y te arrepientes.
- `git cherry-pick --continue` — tras resolver conflictos.

### `git bisect` — búsqueda binaria del bug

Cuando sabes que algo "antes funcionaba y ahora no", bisect te ayuda a encontrar el commit culpable en tiempo logarítmico.

```bash
git bisect start
git bisect bad HEAD            # commit actual: tiene el bug
git bisect good <hash-antiguo> # commit antiguo: funcionaba

# Git te coloca en un commit intermedio. Pruebas la app:
git bisect good   # si funciona
git bisect bad    # si tiene el bug

# Repite hasta que Git te diga "<hash> is the first bad commit"
git bisect reset  # vuelve al estado original
```

Con cada paso, Git divide el rango por la mitad. Con 1000 commits, encuentras el culpable en unos 10 pasos.

---

## 14. Red de seguridad: `git reflog`

Cuando un comando "destruye" commits (`reset --hard`, rebase fallido, amend, etc.), no desaparecen inmediatamente: Git los mantiene durante ~90 días (configurable). El **reflog** registra cada movimiento de HEAD:

```bash
git reflog
```

Salida típica:

```
abc1234 HEAD@{0}: reset: moving to HEAD~1
def5678 HEAD@{1}: commit: Algo importante
```

Para recuperar lo que perdiste:

```bash
git reset --hard HEAD@{1}
```

Es **el seguro de vida de Git**. Mientras no hayas borrado el repo entero, casi nada se pierde de verdad.

---

## 15. Buenas prácticas

1. **Commits pequeños y atómicos.** Un commit, un cambio lógico. Facilita revisión, revertir y bisect.
2. **Mensajes claros, en imperativo.** "Add login form", no "added" ni "adding".
3. **Branch por feature.** Nunca trabajes directo en `main` para cambios no triviales.
4. **Pull antes de push.** Especialmente en ramas compartidas.
5. **No reescribas historia ya compartida** (no `rebase`, no `amend`, no `push --force` en ramas públicas).
6. **Usa `.gitignore` desde el día 1.** Evita commitear archivos sensibles o generados.
7. **`git status` es tu amigo.** Mírale la cara antes de cualquier comando que modifique estado.
8. **Aprende a leer el log.** `git log --oneline --graph --all` te ahorra muchos sustos.

---

## 16. Resumen de comandos por categoría

| Categoría | Comandos clave |
|---|---|
| Configuración | `git config`, `git init`, `git clone` |
| Inspección | `git status`, `git log`, `git diff`, `git show`, `git reflog` |
| Cambios | `git add`, `git commit`, `git restore` |
| Ramas | `git branch`, `git switch`, `git merge`, `git rebase` |
| Deshacer | `git restore`, `git reset`, `git revert`, `git commit --amend` |
| Temporal | `git stash`, `git stash pop`, `git stash list` |
| Remotos | `git remote`, `git fetch`, `git pull`, `git push`, `git clone` |
| Versionado | `git tag` |
| Avanzado | `git cherry-pick`, `git bisect`, `git rebase -i` |

---

Cuando hayas terminado los ejercicios, vuelve a esta guía como referencia. Casi nunca recordarás todas las flags de memoria, y está bien: lo importante es saber **qué buscar**.
