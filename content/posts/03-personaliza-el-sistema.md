---
title: "linux-03 Personaliza el Sistema"
date: 2026-07-30
draft: false
featuredImage: "/images/blog-space.jpg"
tags: [shell bashrc profile prompt personalizacion]
categories: ["Linux Admin"]
series: ["Practical Linux System Administration"]
weight: 03
bookAuthor: "Kenneth Hess"
bookTitle: "Practical Linux System Administration"
bookYear: "2023"
---

# Tutorial 3: Personalizar la experiencia de usuario

**Capítulo 3 del libro** — "Practical Linux System Administration" (pags. 25-32)
**Nivel:** Principiante

---

## ⚡ Para empezar: La guerra de los shells

Bash vs Zsh vs Fish. La guerra santa de los shells. Los usuarios de Bash dicen que "si no usas Bash, no eres un verdadero sysadmin". Los de Zsh responden con "autocompletado infernal". Los de Fish solo dicen "miren, colores".

**Polemica:** Por que diablos los archivos de configuracion empiezan con punto? Por accidente. En Unix, `ls` simplemente omitia los directorios `.` y `..` por redundantes. Los programadores de Bell Labs empezaron a poner punto a sus configs para que `ls` no los mostrara. 50 anos despues, seguimos arrastrando ese accidente historico.

> "Ten cuidado con que programas, scripts y mensajes colocas en tus archivos de inicio porque si estan corruptos, danados o medio abiertos, podrias encontrar que tu inicio de sesion se retrasa o que no puedes iniciar sesion en absoluto."

---

## Objetivos

- Entender la diferencia entre shell login y nonlogin
- Conocer los archivos de personalizacion
- Usar el directorio `/etc/skel`
- Personalizar el prompt del shell

---

## 1. Login vs Nonlogin shells

**Login shell:** SSH o consola con usuario+contrasena.
**Nonlogin shell:** `bash` desde la terminal (shell hijo).

```bash
$ echo $SHLVL
1
$ bash
$ echo $SHLVL
2
```

> "`$SHLVL` es una variable que rastrea tu nivel de shell."

### 🖐️ Mini-ejercicio

```bash
$ echo $SHLVL
$ bash
$ echo $SHLVL
$ exit
```

**Cuantos niveles puedes alcanzar?** Prueba a anidar 5 shells. Que pasa?

---

## 2. Archivos de personalizacion

| Archivo | Cuando se ejecuta |
|---------|------------------|
| `.bashrc` | Shells login y nonlogin |
| `.bash_profile` | Solo login (despues de .bashrc) |
| `.bash_logout` | Al cerrar sesion |

### Archivos globales (no editar directamente)
- `/etc/bashrc` — funciones y alias del sistema
- `/etc/profile` — variables de entorno
- `/etc/profile.d/` — **aqui** hacer cambios personalizados

---

## 3. Archivos del usuario

### `.bashrc`
```bash
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
alias ll='ls -l'
```

> "El flag `-i` significa interactivo y te pide verificar antes de que el comando ejecute su accion."

### `.bash_profile`
```bash
if [ -f ~/.bashrc ]; then
    . ~/.bashrc
fi
```

### 🖐️ Mini-ejercicio

```bash
$ cat ~/.bashrc
```

**Cuantos alias tienes definidos?** Anade `alias ll='ls -l'` y recarga con `source ~/.bashrc`. `ll` funciona ahora?

---

## 4. El directorio `/etc/skel`

> "`/etc/skel` es un directorio especial para contener archivos que quieres que cada usuario reciba en su directorio home al crear sus cuentas."

```bash
# ls -la /etc/skel
-rw-r--r--. 1 root root   18 .bash_logout
-rw-r--r--. 1 root root  141 .bash_profile
-rw-r--r--. 1 root root  376 .bashrc
```

> "Los usuarios existentes NO recibiran archivos colocados en `/etc/skel` despues de haber creado sus cuentas."

---

## 5. Personalizar el prompt (`PS1`)

Prompt predeterminado: `[khess@server1 ~]$`

```bash
PS1="[\u@\h \W]\\$ "
```

| Escape | Descripcion |
|--------|-------------|
| `\u` | Usuario |
| `\h` | Hostname |
| `\W` | Directorio actual |
| `\d` | Fecha |
| `\t` | Hora 24h |
| `\\$` | `#` (root) o `$` |

**Ejemplo navideno del libro:**
```bash
PS1=" \[\e[33;41m\][\[\e[m\]\[\e[32m\]\u\[\e[m\]..."
```

### 🖐️ Mini-ejercicio

```bash
$ PS1=">>> "
$ # Ahora escribe comandos
$ PS1="\d \t > "
$ # Que cambio?
$ PS1="[\u@\h \W]\\$ "   # Restaurar
```

**Disena tu propio prompt** con fecha, hora y usuario. El mas creativo gana.

---

## 🔧 Laboratorio práctico (en clase)

### Paso 1: Inspeccionar tu .bashrc actual
```bash
cat ~/.bashrc
echo $PS1
echo $PATH
```
Revisen el contenido actual del archivo y las variables de entorno activas.

### Paso 2: Agregar alias útiles
```bash
echo 'alias ll="ls -la"' >> ~/.bashrc
echo 'alias ..="cd .."' >> ~/.bashrc
echo 'alias gs="git status"' >> ~/.bashrc
source ~/.bashrc
ll
```
Agregan alias permanentes al `.bashrc` y recargan con `source`. Prueben que `ll` funciona.

### Paso 3: Personalizar el prompt (PS1)
```bash
PS1='[\u@\h \W]\$ '
export PS1
```
Prueben temporalmente. Luego agréguenlo al `.bashrc` para hacerlo permanente:
```bash
echo 'export PS1="[\u@\h \W]\$ "' >> ~/.bashrc
```

### Paso 4: Explorar /etc/skel
```bash
ls -la /etc/skel/
sudo cat /etc/skel/.bashrc
```
`/etc/skel` contiene los archivos por defecto que se copian al home de un usuario nuevo. Todo lo que pongas aquí se heredará.

### Paso 5: Simular un nuevo usuario con personalización
```bash
sudo useradd -m -k /etc/skel -s /bin/bash testuser
sudo ls -la /home/testuser/
```
La bandera `-k` define el directorio esqueleto. Por defecto es `/etc/skel`.

**Verificación:** El prompt debe cambiar inmediatamente después de `export PS1=...`. Los alias deben estar disponibles tras `source ~/.bashrc`.

### 🚀 Desafío individual

Crea un prompt personalizado que muestre:
- La fecha y hora actual en formato `YYYY-MM-DD HH:MM:SS`
- El usuario y el hostname
- La ruta completa del directorio actual

Ejemplo de cómo debe verse:
```
[2026-07-19 14:30:00] alice@server:/home/alice/projects$
```

Hazlo permanente en tu `.bashrc`.

---

## 💪 Ejercicios (para casa / laboratorio)

1. **Personalizacion completa:** Crea un archivo `.bashrc` personalizado desde cero que incluya:
   - Alias para `rm -i`, `cp -i`, `mv -i`, `ll`, `la` (ls -a), `..` (cd ..)
   - Una funcion `mkcd` que cree un directorio y entre en el
   - Una variable `EDITOR=nano`
   - Un prompt personalizado con colores
   - Un mensaje de bienvenida al abrir la terminal

2. **Exploracion de `/etc/skel`:** Anade un archivo `README.txt` en `/etc/skel`, crea un nuevo usuario, verifica que el archivo se copio a su home. Luego anade otro archivo y verifica si el usuario existente lo recibe (spoiler: no).

3. **Script de salida:** Crea un script `.bash_logout` que al cerrar sesion guarde la fecha y duracion de la sesion en un archivo `~/sesiones.log`. Pista: usa `date` y una variable al inicio de `.bashrc` que registre el inicio.

4. **Investigacion:** Averigua que son PS2, PS3 y PS4. En que situaciones se usa cada uno? Crea ejemplos practicos.

5. **Compatibilidad:** Configura un usuario nuevo con Zsh en lugar de Bash. Investiga que archivos de configuracion usa Zsh (.zshrc, .zprofile, etc.). Compara con Bash.

---


## Curiosidad: Los archivos "dot"

`.bashrc` significa "Bash Run Commands". `PS1` es "Prompt String 1" (hay hasta PS4: PS2 para comandos multi-linea, PS3 para `select`, PS4 para `set -x`).

El prompt `$` vs `#` es una convencion de seguridad: si ves `#`, estas como root. Un descuido y `rm -rf *` en el lugar equivocado... todos hemos pasado por eso.
