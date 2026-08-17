---
title: "linux-04 Gestion de Usuarios"
date: 2026-07-31
draft: false
featuredImage: "/images/blog-space.jpg"
tags: [usuarios grupos useradd usermod userdel]
categories: ["Linux Admin"]
series: ["Practical Linux System Administration"]
weight: 04
bookAuthor: "Kenneth Hess"
bookTitle: "Practical Linux System Administration"
bookYear: "2023"
---

# Tutorial 4: Gestion de usuarios

**Capítulo 4 del libro** — "Practical Linux System Administration" (pags. 33-44)
**Nivel:** Principiante

---

## ⚡ Para empezar: El usuario fantasma

Cada sistema Linux tiene **mas de 30 cuentas de servicio** que nadie creo manualmente. Aparecen solas al instalar paquetes. La cuenta `nobody` (UID 65534) es la mas misteriosa: no tiene shell, no tiene home, no puede iniciar sesion. Existe para que los servicios se ejecuten sin ningun privilegio.

**Polemica:** Por que el campo de comentarios en `/etc/passwd` se llama GECOS? Por "General Electric Comprehensive Operating System" — un sistema de los anos 60. Unix heredo el formato y el nombre jamas se actualizo. 60 anos despues, ahi sigue.

> "El UID y GID para root son siempre 0. Ningun otro usuario del sistema tiene estos IDs."

---

## Objetivos

- Crear cuentas de usuario con `useradd` y `adduser`
- Modificar cuentas con `usermod`
- Eliminar cuentas con `userdel`
- Forzar cambios de contrasena con `chage`
- Gestionar grupos

---

## 1. Convenciones UID/GID

| UID | GID | Tipo |
|-----|-----|------|
| 0 | 0 | Root |
| 1-999 | 1-999 | Sistema/servicio |
| 1000+ | 1000+ | Usuarios |

---

## 2. Crear usuarios

```bash
# useradd -c "Jane Smith" jsmith
```

Esto crea: home `/home/jsmith`, archivos desde `/etc/skel`, entrada en `/etc/passwd`.

```bash
# passwd jsmith
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

> "Aunque la cuenta de Jane Smith esta creada, Jane NO puede iniciar sesion. Por que? Porque la cuenta no tiene contrasena."

### 🖐️ Mini-ejercicio

```bash
$ cat /etc/passwd | cut -d: -f1 | head -20
```

**Cuantos usuarios de servicio hay antes del UID 1000?** Identifica 3 cuentas de servicio y adivina para que sirven.

---

## 3. Modificar cuentas con `usermod`

```bash
# usermod -a -G 8020 jsmith        # Grupo suplementario
```

> "Debes usar `-a` (append) y `-G` juntos. Sin `-a`, el usuario pierde todos los otros grupos."

```bash
# usermod -c "Jane R Smith" jsmith  # Comentario
# usermod -e 2021-07-23 rsmith      # Expiracion
# usermod -s /bin/sh jsmith         # Shell
```

Usuario puede cambiar su shell con `chsh`:
```bash
$ chsh -s /bin/zsh
```

---

## 4. Eliminar usuarios

```bash
# userdel jsmith         # Deja home intacto
# userdel -r jsmith      # Elimina tambien home
```

> "Los comandos destructivos de Linux, como `userdel` y `rm`, son irreversibles. Asegurate siempre de tener la cuenta correcta."

### 🖐️ Mini-ejercicio

```bash
$ sudo useradd test99
$ sudo passwd test99
$ sudo chage -l test99
$ sudo userdel -r test99
```

**Crea un usuario, examina su configuracion y eliminalo.** Que campos de `chage -l` consideras mas importantes para seguridad?

---

## 5. Forzar cambios de contrasena con `chage`

```bash
# chage -l rsmith
Last password change           : Jul 17, 2021
Password expires               : never
Account expires                : never
```

> "Esta cuenta tiene la contrasena que nunca expira, lo cual es una violacion de seguridad."

```bash
# chage -m 1 -M 90 rsmith     # Min 1 dia, max 90 dias
```

> "Establecer un numero minimo de dias asegura que los usuarios no cambien sus passwords 10 veces para volver a la original."

---

## 6. Cuentas de servicio

```bash
$ cat /etc/passwd | grep nologin | head -10
```

Ejemplo: `nobody:x:65534:65534:Kernel Overflow User:/:/sbin/nologin`

```bash
# su - nobody
This account is currently not available.
```

---

## 7. Gestionar grupos

```bash
# mkdir /opt/finance
# chgrp finance /opt/finance
# chmod 770 /opt/finance
```

> "Cuando gestionas permisos para un conjunto de usuarios, es mas conveniente definir y gestionar un grupo que gestionar cada usuario por separado."

### 🖐️ Mini-ejercicio

```bash
$ groups
$ id
```

**A cuantos grupos perteneces?** Cual es tu GID primario? Para que serviria crear un grupo por proyecto?

---

## 🔧 Laboratorio práctico (en clase)

### Paso 1: Crear usuarios con opciones específicas
```bash
sudo useradd -m -s /bin/bash -c "Carlos Garcia, Dept IT" carlos
sudo useradd -m -s /bin/bash -c "Maria Lopez, Dept Ventas" -G ventas maria
sudo useradd -m -s /usr/sbin/nologin -c "Usuario FTP" ftpuser
```
Crean tres usuarios: uno con nombre real y departamento en GECOS, otro con grupo secundario, y uno sin shell interactivo.

### Paso 2: Establecer y gestionar contraseñas
```bash
sudo passwd carlos
sudo chage -d 0 carlos
sudo chage -M 90 carlos
sudo chage -l carlos
```
`-d 0` fuerza cambio de contraseña en el próximo login. `-M 90` establece expiración a 90 días.

### Paso 3: Modificar usuarios existentes
```bash
sudo usermod -aG ventas carlos
sudo usermod -c "Carlos Garcia, Dept IT y Ventas" carlos
sudo usermod -L carlos
sudo usermod -U carlos
```
`-aG` agrega a un grupo sin sacar de otros. `-L` bloquea la cuenta. `-U` la desbloquea.

### Paso 4: Gestionar grupos
```bash
sudo groupadd ventas
sudo groupadd it
sudo groupadd admin
sudo gpasswd -a carlos it
sudo gpasswd -M carlos,maria ventas
```
Crean grupos, agregan miembros. `gpasswd -M` establece la lista de miembros del grupo.

### Paso 5: Eliminar usuarios
```bash
sudo userdel -r ftpuser
```
La bandera `-r` elimina también el home directory y el spool de correo. Sin `-r`, la cuenta se elimina pero los archivos quedan huérfanos.

**Verificación:** `cat /etc/passwd` y `cat /etc/group` deben reflejar todos los cambios. `sudo chage -l carlos` debe mostrar expiración a 90 días.

### 🚀 Desafío individual

Diseña y crea una estructura para una empresa con 3 departamentos:

| Departamento | Usuarios | Grupo |
|---|---|---|
| Ventas | alicia, bruno, clara | ventas |
| IT | diana, ernesto | it |
| RH | fernanda, gabriel | rh |

Requisitos:
- Cada usuario pertenece a su departamento como grupo primario.
- Todos los departamentos comparten un directorio `/home/compartido/oficina` con permisos `rwxrwx---` (770).
- Cada departamento tiene un directorio `/home/compartido/<depto>` accesible solo por miembros de ese departamento.
- Las contraseñas expiran cada 60 días para todos.

---

## 💪 Ejercicios (para casa / laboratorio)

1. **Automatizacion de altas:** Crea un script `crear-usuario.sh` que:
   - Tome como parametro nombre, grupo y departamento
   - Cree el usuario con `useradd`
   - Le asigne una contrasena temporal
   - Le fuerce cambio de contrasena en el primer login (`chage -d 0`)
   - Lo anada al grupo `sudo` o `wheel`
   - Muestre un resumen final

2. **Politica de passwords:** Crea un script que aplique a TODOS los usuarios humanos (UID ≥ 1000) la siguiente politica:
   - Contrasena expira cada 90 dias
   - Minimo 1 dia entre cambios
   - Alerta 7 dias antes de expirar
   - Cuenta se desactiva tras 15 dias de inactividad
   Usa `chage` y un bucle sobre `/etc/passwd`.

3. **Auditoria de cuentas:** Escribe un script que genere un reporte HTML con:
   - Usuarios con contrasena expirada
   - Cuentas sin contrasena
   - Usuarios con UID 0 (aparte de root)
   - Cuentas bloqueadas
   - Fecha del ultimo cambio de contrasena

4. **Escenario de integracion:** Crea los siguientes grupos y usuarios:
   - Grupo `ventas` con usuarios `v1`, `v2`
   - Grupo `it` con usuarios `admin1`, `admin2`
   - Directorio `/opt/ventas` accesible solo para ventas (770)
   - Directorio `/opt/it` accesible solo para IT (770)
   - Un directorio `/opt/compartido` accesible para ambos grupos (770)
   Documenta los permisos y resuelve: que pasa si `admin1` necesita leer un archivo en `/opt/ventas`?

5. **Reto:** Sin usar `userdel`, como hariias para desactivar temporalmente una cuenta sin perder sus datos? (Pista: `/etc/shadow`, `usermod -L`, `usermod -e 1`). Prueba los 3 metodos.

---


## Curiosidad: GECOS

El campo de comentarios en `/etc/passwd` se llama GECOS por "General Electric Comprehensive Operating System". Un sistema de los 60. 60 anos despues, ahi sigue.

El usuario `nobody` (UID 65534) existe porque en los primeros Unix, si un servicio no tenia un usuario asignado, se ejecutaba como "nadie". Sigue siendo el estandar de seguridad para procesos sin privilegios.
