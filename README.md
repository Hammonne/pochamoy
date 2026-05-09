# pochamoy

Registro paso a paso de los comandos de Git que se usaron para cambiar la fecha de los commits en este repositorio.

## Contexto

Cuando haces un `git commit`, Git guarda dos fechas:

- **AuthorDate**: cuándo se escribió el cambio originalmente.
- **CommitDate**: cuándo se creó el commit en el repositorio.

Por defecto ambas son la fecha y hora del sistema. Para “mover” la fecha de un commit hay que sobrescribir esas dos variables.

## Estado inicial

El repositorio se clonó el `2026-05-09 00:14:36` y, según el reflog, los commits originales tenían estas fechas:

| Commit  | Mensaje              | Fecha original              |
|---------|----------------------|-----------------------------|
| a3dba30 | Add files via upload | 2024-04-13 08:31:50 -0500   |
| 9e5646c | tu papa              | 2026-05-08 23:16:22 -0500   |
| 65bca24 | ajajajaj vete alv    | 2026-05-09 01:05:48 -0500   |

El objetivo era dejar el último commit con fecha `2026-05-08 12:00:00 -0500`.

## Paso a paso

### 1. Verificar el estado actual

Antes de tocar nada, se revisó el log para ver qué commit tenía la fecha que se quería cambiar.

```powershell
git log --pretty=format:"%h | %an | %ad | %s" --date=iso
```

Salida (resumida):

```
65bca24 | Hammonne | 2026-05-09 01:05:48 -0500 | ajajajaj vete alv
9e5646c | Hammonne | 2026-05-08 23:16:22 -0500 | tu papa
a3dba30 | Jimena   | 2024-04-13 08:31:50 -0500 | Add files via upload
fe7565c | Jimena   | 2024-04-13 08:25:45 -0500 | Initial commit
```

### 2. Cambiar la fecha del último commit con `--amend`

Para reescribir la fecha del commit que está en `HEAD` se usa `git commit --amend` junto con la flag `--date` (que cambia el **AuthorDate**) y la variable de entorno `GIT_COMMITTER_DATE` (que cambia el **CommitDate**). Si solo usas `--date`, el `CommitDate` se queda con la hora actual del sistema.

En PowerShell se setea la variable de entorno antes del comando:

```powershell
$env:GIT_COMMITTER_DATE = "2026-05-08 12:00:00 -0500"
git commit --amend --no-edit --date="2026-05-08 12:00:00 -0500"
```

Qué hace cada parte:

- `$env:GIT_COMMITTER_DATE = "..."` → fija el **CommitDate** que Git va a usar.
- `git commit --amend` → reescribe el commit que está en `HEAD` (no crea uno nuevo encima).
- `--no-edit` → mantiene el mensaje del commit tal cual estaba (`ajajajaj vete alv`).
- `--date="..."` → fija el **AuthorDate**.

> Importante: `--amend` cambia el **hash** del commit. Por eso en el reflog el commit pasa de `65bca24` a `fc53f61`, aunque el mensaje sea el mismo.

### 3. Verificar el cambio

Se vuelve a correr el log para confirmar que la fecha quedó como se quería.

```powershell
git log --pretty=format:"%h | %ad | %s" --date=iso
```

Salida después del amend:

```
fc53f61 | 2026-05-08 12:00:00 -0500 | ajajajaj vete alv
9e5646c | 2026-05-08 23:16:22 -0500 | tu papa
a3dba30 | 2024-04-13 08:31:50 -0500 | Add files via upload
fe7565c | 2024-04-13 08:25:45 -0500 | Initial commit
```

El commit `65bca24` ya no aparece porque fue reemplazado por `fc53f61` con la nueva fecha.

### 4. Confirmar en el reflog

El reflog deja la huella de la operación:

```powershell
git reflog --date=iso
```

```
fc53f61 HEAD@{2026-05-08 12:00:00 -0500}: commit (amend): ajajajaj vete alv
65bca24 HEAD@{2026-05-09 01:05:48 -0500}: commit: ajajajaj vete alv
9e5646c HEAD@{2026-05-08 23:16:22 -0500}: commit: tu papa
a3dba30 HEAD@{2026-05-09 00:14:36 -0500}: clone: from https://github.com/Hammonne/pochamoy.git
```

Ahí se ve claro que `commit (amend)` reemplazó al `commit` original.

### 5. Subir el cambio al remoto

Como `--amend` reescribe historia, un `git push` normal lo rechaza. Hay que forzarlo:

```powershell
git push --force-with-lease origin main
```

Se usa `--force-with-lease` (no `--force` a secas) porque aborta el push si alguien más subió commits al remoto desde el último `fetch`, evitando pisar trabajo ajeno.

## Notas y advertencias

- **Solo en commits no compartidos**: reescribir la fecha de un commit que ya usa otra gente rompe su historia local. Hacerlo solo en ramas propias.
- **AuthorDate vs CommitDate**: si solo cambias una de las dos, GitHub puede mostrar fechas “raras” (la de autor en una vista y la de commit en otra). Por eso se cambian las dos.
- **Formato de fecha**: Git acepta formatos ISO 8601 (`2026-05-08T12:00:00-05:00`) y formatos tipo RFC (`Fri, 8 May 2026 12:00:00 -0500`). El usado aquí es `YYYY-MM-DD HH:MM:SS ±HHMM`.

## Comando para cambiar fechas de commits anteriores (referencia)

Si quisieras cambiar la fecha de un commit que **no** está en `HEAD`, `--amend` no sirve. Habría que usar un rebase interactivo:

```powershell
git rebase -i <commit-anterior-al-que-quieres-cambiar>
# marcar el commit como "edit"
$env:GIT_COMMITTER_DATE = "2026-05-08 12:00:00 -0500"
git commit --amend --no-edit --date="2026-05-08 12:00:00 -0500"
git rebase --continue
```

Este caso no se usó en este repo, pero queda anotado por si hace falta más adelante.
