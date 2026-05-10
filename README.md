# OK

## Paso a paso

### Paso 1
Esto lo haces una vez nomass
Primero cambiar usuario y la otra wbd cambiar lo que esta entre comillas

```powershell
git config --global user.name "nombre_ususario" 
```
pon tu correo pe
```powershell
git config --global user.email "correo@utec.edu.pe"
```

### Paso 2
hacer add y commit de tus cambios :)
```powershell
git add archivo_modificado.py
```
o
```powershell
git add .
```

luego

```powershell
git commit -m "mensaje"
```

correr este comando, cambiando la fecha a la que quieras
```powershell
GIT_COMMITTER_DATE="2026-05-08T12:00:00 -0500" git commit --amend --no-edit --reset-author --date="2026-05-08T12:00:00 -0500"
```
si te da error pq tienes windows, usa este

```powershell
$env:GIT_COMMITTER_DATE="2026-05-08T12:00:00 -0500"; git commit --amend --no-edit --reset-author --date="2026-05-08T12:00:00 -0500"
```

finalmente, haz el push con force
```powershell
git push origin main --force
```
