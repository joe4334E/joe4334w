---
title: "Sesion 1: Fundamentos de Linux"
date: 2026-08-18
draft: false
featuredImage: "/images/blog-space.jpg"
tags: [resumen principiante]
categories: ["Linux Admin"]
series: ["Practical Linux System Administration"]
weight: 50
bookAuthor: "Kenneth Hess"
bookTitle: "Practical Linux System Administration"
bookYear: "2023"
---

# Sesion 1: Fundamentos de Linux

**Tutoriales:** 01 — Tu Primer Sistema Linux | 02 — Permisos y Root | 03 — Personaliza el Sistema
**Nivel:** Principiante
**Duracion estimada:** 3-4 horas

---

## Objetivos de cada tutorial

### Tutorial 01 — Tu Primer Sistema Linux
- Instalar Linux en una maquina virtual
- Conocer el sistema de archivos de Linux
- Aprender los comandos esenciales de la CLI: `pwd`, `cd`, `ls`
- Arrancar, reiniciar y apagar el sistema correctamente

### Tutorial 02 — Permisos y Root
- Distinguir entre usuario regular y root
- Usar `su` y `sudo` para obtener privilegios de root
- Leer y modificar permisos de archivos (simbolico y numerico)
- Configurar `umask`

### Tutorial 03 — Personaliza el Sistema
- Entender la diferencia entre shell login y nonlogin
- Conocer los archivos de personalizacion
- Usar el directorio `/etc/skel`
- Personalizar el prompt del shell

---

## Resumen

Esta sesion sienta las bases de todo lo que viene despues. El Tutorial 01 te instala en una maquina virtual y te hace recorrer el sistema de archivos de Linux: desde `/` (la raiz de todo) hasta `/proc` (un sistema de archivos virtual). Aprendes a mverte con `pwd`, `cd` y `ls`, y a apagar/reiniciar correctamente. Lo mas importante: Linux no usa extensiones para determinar ejecutabilidad y no te pregunta "estas seguro?" antes de borrar archivos.

El Tutorial 02 entra en uno de los conceptos mas criticos: los permisos. Aprendes que root (UID 0) tiene control total del sistema, pero trabajar permanentemente como root es peligroso. La practica segura es usar `sudo` para ejecutar comandos privilegiados con auditoria. Los permisos se representan en formato simbolico (`rwxr-xr--`) o numerico (754), y `umask` controla los permisos por defecto de archivos nuevos.

El Tutorial 03 cierra la sesion con la personalizacion: los archivos `.bashrc`, `.bash_profile` y `.profile` controlan el comportamiento de tu shell. `/etc/skel` es la plantilla para nuevos usuarios. La variable `PS1` controla la apariencia del prompt.

---

## Comandos clave

| Comando | Descripcion |
|---------|-------------|
| `pwd` | Muestra el directorio de trabajo actual |
| `cd` | Cambia de directorio |
| `ls -la` | Lista archivos con metadatos y ocultos |
| `sudo` | Ejecuta un comando como root (con auditoria) |
| `su` | Cambia al usuario root |
| `chmod` | Cambia permisos de archivos |
| `umask` | Establece permisos por defecto |
| `shutdown -h now` | Apaga el sistema inmediatamente |
| `reboot` | Reinicia el sistema |
| `echo $PS1` | Muestra la variable del prompt |

---

## Ejercicio integrador A

**Primer contacto con el sistema:**
1. Crea una maquina virtual con Ubuntu Server o Debian minimal
2. Inicia sesion y ejecuta `pwd`, `ls /`, `ls -la ~` para explorar
3. Crea un directorio `miscript` en tu home, crea un archivo `hola.sh` con `#!/bin/bash` y `echo "Hola desde Linux"`
4. Cambia permisos con `chmod 755 hola.sh` y ejecutalo
5. Investiga con `cat /etc/passwd` cuantos usuarios de sistema existen

## Ejercicio integrador B

**Configuracion de un usuario nuevo:**
1. Crea un usuario nuevo con `sudo useradd -c "Nombre Apellido" nuevo_usuario`
2. Asignale contraseña con `sudo passwd nuevo_usuario`
3. Modifica su `.bashrc` para agregar un alias `ll='ls -la'`
4. Configura el prompt para que muestre usuario@host en color verde
5. Verifica con `su - nuevo_usuario` que todo funciona correctamente
6. Documenta que archivos de configuracion se ejecutaron al iniciar sesion

---

## Para recordar

- Linux distingue mayusculas de minusculas: `Archivo.txt` y `archivo.txt` son diferentes
- El prompt `$` indica usuario regular; `#` indica root
- `sudo` es preferible a `su` por auditoria
- Nunca trabajes permanentemente como root
- `/etc/skel` es la plantilla para nuevos usuarios
