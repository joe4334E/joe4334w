---
title: "linux-01 Tu Primer Sistema Linux"
date: 2026-07-28
draft: false
featuredImage: "/images/blog-space.jpg"
tags: [fundamentos instalacion filesystem comandos basicos]
categories: ["Linux Admin"]
series: ["Practical Linux System Administration"]
weight: 01
bookAuthor: "Kenneth Hess"
bookTitle: "Practical Linux System Administration"
bookYear: "2023"
---

# Tutorial 1: Tu primer sistema Linux

**Capítulo 1 del libro** — "Practical Linux System Administration" (pags. 1-10)
**Nivel:** Principiante

---

## ⚡ Para empezar: Linux no es Windows

"Linux (como Unix) no es 'hablador' como los sistemas Windows. Los sistemas Linux no te preguntan 'Esta seguro?' cuando eliminas archivos."

Linus Torvalds creo Linux en 1991 como hobby. Su mensaje original: *"I'm doing a (free) operating system (just a hobby, won't be big and professional like GNU"*) — ironico, considerando que hoy Linux domina servidores, moviles y la nube.

**Dato polemico:** Muchos insisten en llamarlo "GNU/Linux" en vez de "Linux". Stallman (creador de GNU) dice que sin GNU, Linux seria solo un kernel inutil. Torvalds no esta de acuerdo. 30 anos de debate y contando.

---

## Objetivos

- Instalar Linux en una maquina virtual
- Conocer el sistema de archivos de Linux
- Aprender los comandos esenciales de la CLI: `pwd`, `cd`, `ls`
- Arrancar, reiniciar y apagar el sistema correctamente

---

## 1. Instalar Linux

El libro recomienda instalar Linux en una **maquina virtual (VM)** — VirtualBox (gratuito, multiplataforma).

**Distribuciones sugeridas:** Debian, OpenSUSE, RHEL, Ubuntu

> "Cuando tu sistema arranque, puedes aceptar la configuracion predeterminada. Crea una cuenta de usuario cuando se te solicite."

Pasos: Nueva VM → nombre/tipo → 2GB RAM → disco virtual → montar ISO → arrancar.

---

## 2. El sistema de archivos

> "En Linux, solo hay un directorio raiz, `/`. Todos los demas directorios son subdirectorios del directorio raiz."

```
/
├── bin → /usr/bin
├── etc    (configuracion)
├── home   (usuarios)
├── var    (logs, colas)
├── tmp    (temporales)
├── proc   (procesos, virtual)
└── ...
```

### 🖐️ Mini-ejercicio

Abre tu terminal y escribe:

```bash
$ ls /
```

**Cuantos directorios hay en la raiz?** Compara con tus companeros.

---

## 3. Aprender la CLI

> "Los administradores de sistemas Linux raramente instalan o usan GUIs en sus servidores."

### Reglas de oro:
1. Linux no usa extensiones — `file.exe` no significa nada especial
2. Mayusculas importan — `File.txt` ≠ `file.txt`
3. Linux asume que sabes lo que haces — no hay "¿Esta seguro?"
4. La ortografia importa

### pwd — Donde estoy?

```bash
$ pwd
/home/student1
```

### cd — Moverse

```bash
$ cd /etc
$ pwd
/etc
$ cd         # Vuelve a home
$ pwd
/home/student1
```

### 🖐️ Mini-ejercicio

```bash
$ cd /var/log
$ pwd
$ ls
```

**Que archivos hay en /var/log?** Prueba `ls -la` para ver los ocultos.

### ls — Listar

```bash
$ cd
$ pwd
/home/student1
$ ls -a
.  ..  .bash_history  .bash_logout  .bash_profile  .bashrc  .gnupg  .zshrc
```

> "Todos los directorios y archivos cuyos nombres comienzan con un punto estan ocultos de las listas de archivos estandar usando el comando `ls` sin opciones."

---

## 4. Arrancar, reiniciar y apagar

> "Reiniciar o rearrancar un sistema es una practica estandar de administracion. No hay nada malo en reiniciar tu sistema."

> "Cualquier problema resuelto con un reinicio debe investigarse mas a fondo despues de que el sistema este estable."

> "El apagon del sistema debe reservarse para mantenimiento de hardware, reubicacion o desmantelamiento."

### 🖐️ Mini-ejercicio

```bash
$ uptime
```

**Cuanto tiempo lleva tu sistema encendido?** El que mas tiempo tenga... cuenta una anecdota de por que no lo apaga.

---

## 🔧 Laboratorio práctico (en clase)

### Paso 1: Explorar tu ubicación actual
```bash
pwd
```
Muestra el directorio actual. Por defecto estás en `/home/tu_usuario` (tu home).

### Paso 2: Listar contenidos
```bash
ls
ls -l
ls -la
```
`-l` da formato largo (permisos, dueño, tamaño, fecha). `-a` muestra archivos ocultos (punto). Comparen las diferencias entre los tres.

### Paso 3: Navegar por el sistema de archivos
```bash
cd /
ls -la
cd /usr
ls
cd share
pwd
```
Naveguen desde root `/` hasta `/usr/share`. Observen cómo `pwd` confirma su ubicación en cada paso.

### Paso 4: Explorar con man
```bash
man ls
```
Usen las teclas `j`/`k` para desplazarse y `q` para salir. Busquen la bandera `-h` (human-readable) dentro de `man ls`.

### Paso 5: Archivos ocultos en el home
```bash
cd ~
ls -la
cat .bashrc
```
El home contiene archivos de configuración que empiezan con punto. El `.bashrc` es el archivo que configura tu shell.

**Verificación:** Deben ver su prompt cambiar de directorio a medida que navegan, y notar que `ls -la` revela archivos que `ls` solo no muestra.

### 🚀 Desafío individual

Partiendo desde `/`, navega hasta `/usr/share/doc` usando **solo rutas relativas** (sin usar `cd /usr/share/doc` directamente). Luego regresa a `/` usando `cd ..` la menor cantidad de veces posible.

Pista: desde `/`, `cd usr` (sin la barra inicial) es una ruta relativa.

---

## 💪 Ejercicios (para casa / laboratorio)

1. **Instala una VM** con Ubuntu Server o CentOS usando VirtualBox. Documenta cada paso con capturas.
2. **Navegacion completa:** Desde tu home, llega a `/usr/share/doc` usando solo rutas relativas. Vuelve con `cd`. Escribe todos los pasos.
3. **Arbol de directorios:** Dibuja el arbol completo de `/` hasta 3 niveles de profundidad usando `ls -R` o `tree`. Identifica que contiene cada directorio.
4. **Script de investigacion:** Crea un script que ejecute `pwd`, `ls /`, `ls -la ~` y `uptime`, y guarde la salida en `informe-sistema.txt`. Explica que hace cada linea.
5. **Comparativa:** Escribe un documento de 1 pagina comparando Linux vs Windows en: sistema de archivos, permisos, extensiones, CLI vs GUI. Presentalo en la siguiente clase.

---


## Curiosidad: Tux y el nombre Linux

El pinguino Tux fue elegido mascota en 1996. Linus confeso que le pico un pinguino en un zoo australiano. Tux = Torvalds UniX.

> "Some people have told me they don't think a fat penguin really embodies the grace of Linux, which just tells me they have never seen a penguin explode with righteous indignation at 200 mph." — Linus Torvalds
