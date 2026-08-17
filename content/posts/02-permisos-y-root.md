---
title: "linux-02 Permisos y Root"
date: 2026-07-29
draft: false
featuredImage: "/images/blog-space.jpg"
tags: [permisos root sudo chmod umask seguridad]
categories: ["Linux Admin"]
series: ["Practical Linux System Administration"]
weight: 02
bookAuthor: "Kenneth Hess"
bookTitle: "Practical Linux System Administration"
bookYear: "2023"
---

# Tutorial 2: Permisos y cuentas privilegiadas

**Capítulo 2 del libro** — "Practical Linux System Administration" (pags. 11-23)
**Nivel:** Principiante

---

## ⚡ Para empezar: El poder absoluto corrompe absolutamente

En Linux, el usuario root (UID 0) puede hacer **absolutamente todo**. No hay "¿Estas seguro?" No hay papelera de reciclaje. `rm -rf /` ejecutado como root destruye el sistema en segundos.

**Polemica:** En los primeros Unix, la contrasena de root se compartia entre todos los administradores. Si alguien cometia un error, no habia forma de saber quien fue. Por eso se invento `sudo` — para tener auditoria. Aun hoy, muchas empresas tienen la contrasena de root en un post-it pegado al monitor. No seas esa empresa.

> "La regla general y mas consciente de la seguridad es que siempre debes trabajar como un usuario regular a menos que alguna tarea requiera acceso privilegiado (root)."

---

## Objetivos

- Distinguir entre usuario regular y root
- Usar `su` y `sudo` para obtener privilegios de root
- Leer y modificar permisos de archivos (simbolico y numerico)
- Configurar `umask`

---

## 1. Usuario regular vs root

**Usuario regular:** Acceso limitado a su home y `/tmp`.

**Usuario root:** Control total del sistema.

> "El usuario root puede crear, editar, mover o eliminar cualquier archivo del sistema."

---

## 2. Metodos para ser root

### Iniciar sesion como root (NO recomendado)
> "No se recomienda SSH a un sistema e iniciar sesion como root."

### su (Substitute User)
```bash
$ su -
Password:
#
```
> "El prompt `#` te informa que ahora has iniciado sesion como root."

### sudo (Substitute User Do) — RECOMENDADO
```bash
$ sudo env
[sudo] password for bjones:
```

### 🖐️ Mini-ejercicio

```bash
$ whoami
$ id
```

**Eres root o usuario regular?** Cual es tu UID? Si no eres root, `sudo whoami` te muestra algo diferente?

---

## 3. Crear un sudoer

> "Debes tener acceso root para editar el archivo `/etc/sudoers` y usar la utilidad `visudo`."

```bash
# visudo
```

```ini
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
khess   ALL=(ALL)       ALL
```

> "Configurar una cuenta de usuario para usar sudo sin emitir una contrasena no es recomendado."

---

## 4. Leer permisos

| Permiso | Letra | Valor | En archivos | En directorios |
|---------|-------|-------|-------------|----------------|
| Read | r | 4 | Ver contenido | Listar contenidos |
| Write | w | 2 | Crear/modificar | Copiar, mover |
| Execute | x | 1 | Ejecutar | Entrar (cd) |

**Grupos:** `u`(user), `g`(group), `o`(others), `a`(all)

### Leer permisos con `ls -l`

```bash
$ touch file.txt
$ ls -l
-rw-rw-r--. 1 khess khess 0 Jun 19 17:35 file.txt
```
- Posicion 1: tipo (`-`=archivo, `d`=directorio)
- Posiciones 2-4: user (rw-)
- Posiciones 5-7: group (rw-)
- Posiciones 8-10: others (r--)

**Ejemplo:** `-rw-rw-r--` = 664

### 🖐️ Mini-ejercicio

```bash
$ ls -l /etc/passwd /etc/shadow /etc/hosts
```

**Que permisos tiene cada uno?** Por que `/etc/shadow` tiene permisos tan restrictivos? (`----------` en RHEL)

---

## 5. Cambiar permisos con `chmod`

### Modo simbolico
```bash
$ chmod o-r file.txt          # Quitar lectura a others
$ chmod u+x script.sh          # Anadir ejecucion al user
$ chmod ug+x,o+w file.txt     # Multiples cambios
```

> "Si no especificas a que grupos, el sistema asume 'all'. Esto puede ser peligroso."

### Modo numerico
```bash
$ chmod 660 file.txt           # -rw-rw----
```

### 🖐️ Mini-ejercicio

```bash
$ echo "echo Hola" > script.sh
$ ./script.sh                  # Error? Por que?
$ chmod u+x script.sh
$ ./script.sh                  # Ahora funciona?
```

**Que permisos tenia script.sh antes y despues del chmod?**

---

## 6. `umask` — Permisos predeterminados

> "Un `umask` (user file-creation mask) enmascara o filtra ciertos permisos."

```bash
$ umask
0002
```

Con umask 002: archivos nuevos = `-rw-rw-r--` (664). El permiso de escritura se filtra de "others".

El primer digito (0) es para permisos especiales: **setuid, setgid y sticky**.

---

## 🔧 Laboratorio práctico (en clase)

### Paso 1: Crear usuarios de prueba
```bash
sudo useradd -m -s /bin/bash alice
sudo useradd -m -s /bin/bash bob
sudo passwd alice
sudo passwd bob
```
Crean dos usuarios con home directory (`-m`) y shell bash. Establecen contraseñas.

### Paso 2: Probar su y sudo
```bash
su - alice
whoami
exit
sudo -u alice whoami
```
Observen la diferencia: `su -` abre una sesión completa; `sudo -u` ejecuta un solo comando.

### Paso 3: Practicar chmod con notación simbólica
```bash
touch prueba.txt
ls -l prueba.txt
chmod u+x prueba.txt
chmod g-w prueba.txt
chmod o=r prueba.txt
ls -l prueba.txt
```
Paso a paso: agregar ejecución para dueño, quitar escritura para grupo, asignar solo lectura para otros.

### Paso 4: Practicar chmod con notación octal
```bash
chmod 751 prueba.txt
ls -l prueba.txt
chmod 644 prueba.txt
ls -l prueba.txt
```
Desglose: 7=rwx, 5=r-x, 1=--x. Luego 6=rw-, 4=r--, 4=r--.

### Paso 5: Explorar umask
```bash
umask
touch test-umask.txt
ls -l test-umask.txt
umask 0027
umask
touch test-umask2.txt
ls -l test-umask2.txt
```
La umask resta permisos de la base 666 (archivos) o 777 (directorios). La umask por defecto suele ser 022 (resultado: 644 y 755).

**Verificación:** `ls -l` debe mostrar permisos que cambian después de cada `chmod`. Verifiquen que `umask` altera los permisos por defecto de archivos nuevos.

### 🚀 Desafío individual

Crea un directorio compartido `/home/compartido` que pertenezca al grupo `proyecto`. Los usuarios `alice` y `bob` deben ser miembros de ese grupo. El directorio debe tener permisos `rwxrwx---` (770) para que ambos puedan leer, escribir y ejecutar, pero nadie más pueda acceder. Verifica que `bob` puede crear archivos allí y que un tercer usuario (ej. `nadie`) no puede.

---

## 💪 Ejercicios (para casa / laboratorio)

1. **Analisis de permisos:** Crea 5 archivos con diferentes permisos (644, 755, 700, 666, 600). Explica que significa cada uno en terminos de `rwx`. Para cada archivo, indica quien puede leer, escribir y ejecutar.

2. **Escenario real:** Eres administrador de un sistema con 3 usuarios (ana, bob, carla). Crea un directorio `/opt/proyecto` donde:
   - ana y bob puedan leer y escribir
   - carla solo pueda leer
   - nadie mas pueda hacer nada
   Documenta cada comando usado.

3. **Umask profundidad:** Cambia tu umask a 027, crea un archivo, explica los permisos resultantes (pista: 027 = ---/-w-/rwx filtrado). Luego cambialo a 077 y repite. Que aplicacion practica tendria un umask 077?

4. **setuid investigation:** Busca todos los binarios con setuid en tu sistema (`find / -perm -4000`). Identifica 3 de ellos y explica por que necesitan setuid. Investiga el riesgo de seguridad.

5. **Caso real:** Una empresa tuvo un incidente donde un desarrollador ejecuto `chmod -R 777 /etc` por error. Investiga que consecuencias tendria y como recuperarias el sistema.

---


## Curiosidad: El `rm -rf /` mas famoso

En 2021, un desarrollador de Node.js ejecuto un script mal escrito que borro `/node_modules` pero se llevo archivos del sistema. Los memes exageraron el desastre, pero el incidente sirve como recordatorio: **siempre revisa tu script antes de ejecutarlo como root**.

El nombre "root" viene de la raiz `/`. UID 0 es poder absoluto — no importa si la cuenta se llama "root", "admin" o "toor".
