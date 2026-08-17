---
title: "linux-06 Instalar y Borrar Software"
date: 2026-08-04
draft: false
featuredImage: "/images/blog-space.jpg"
tags: [apt dpkg rpm repositorios software]
categories: ["Linux Admin"]
series: ["Practical Linux System Administration"]
weight: 06
bookAuthor: "Kenneth Hess"
bookTitle: "Practical Linux System Administration"
bookYear: "2023"
---

# Tutorial 6: Instalar y desinstalar software

**Referencia:** Capitulo 6 - "Installing and Uninstalling Software" (pags. 57-75)
**Nivel:** Principiante

---

## ⚡ Para empezar: La guerra de los gestores de paquetes

**APT vs YUM vs DNF vs Pacman vs emerge.** Cada distribucion Linux tiene su propio gestor de paquetes, y sus usuarios defienden el suyo a capa y espada. Los de Debian/Ubuntu dicen "APT es elegante y simple". Los de Red Hat responden "DNF resuelve dependencias mejor". Los de Arch se rien con Pacman. Los de Gentoo... bueno, ellos compilan TODO desde codigo fuente.

**Easter egg:** APT tiene una vaca escondida. Ejecuta `apt moo` y veras.

> "No hay comandos especiales para parches, actualizaciones de seguridad o actualizaciones de version; este solo comando se encarga de todo."

---

## Objetivos

- Actualizar el sistema en RHEL/CentOS y Debian/Ubuntu
- Instalar software desde repositorios
- Instalar paquetes individuales manualmente
- Instalar software desde codigo fuente
- Desinstalar software por cada metodo

---

## 1. Actualizar el sistema

**RHEL/CentOS:**
```bash
$ sudo yum update
```
> "YUM/DNF responde `N` por defecto. Debes responder `y` explicitamente."

**Debian/Ubuntu:**
```bash
$ sudo apt update
$ sudo apt upgrade
```
> "En Debian, `Y` es la respuesta predeterminada. Presionar Enter INSTALA."

---

## 🖐️ Mini-ejercicio

```bash
$ sudo apt update
```

**Cuantos paquetes actualizables tienes?** Si usas RHEL: `sudo dnf check-update`. El que mas tenga, cuenta cual fue el ultimo que actualizo.

---

## 2. Instalar desde repositorios

> "Instalar desde un repositorio es el metodo mas facil porque satisface automaticamente tus dependencias."

```bash
$ sudo dnf install lynx        # RHEL
$ sudo apt install lynx        # Debian
```

### Dependencias
```bash
$ dnf deplist lynx              # RHEL
$ sudo apt show lynx            # Debian (linea Depends:)
```

---

## 3. Desinstalar

```bash
$ sudo rpm -e lynx              # RHEL
$ sudo dnf autoremove

$ sudo apt purge lynx           # Debian
$ sudo apt autoremove
```

> "La opcion `purge` elimina el paquete pero no las dependencias. Debes ejecutar `sudo apt autoremove`."

---

## 🖐️ Mini-ejercicio

```bash
$ sudo apt install sl -y
$ sl
```

**Que acaba de pasar?** Ahora desinstalalo: `sudo apt purge sl -y && sudo apt autoremove -y`. Por que crees que existe este paquete?

---

## 4. Instalar paquetes manualmente

### RPM (RHEL)
```bash
$ sudo dnf --downloadonly install lynx
$ sudo rpm -i lynx-2.8.9-2.el8.x86_64.rpm
```

### DPKG (Debian)
```bash
$ sudo apt install --download-only lynx
$ sudo dpkg -i lynx_2.9.0dev.5-1_amd64.deb
```

---

## 5. Instalar desde codigo fuente

> "Instalar desde source te permite personalizar la instalacion para tus necesidades."

```bash
$ sudo dnf groupinstall "Development Tools"   # RHEL
$ sudo apt install build-essential            # Debian

$ wget https://ejemplo.com/programa.tar.gz
$ tar zxvf programa.tar.gz
$ cd programa
$ ./configure
$ make
$ sudo make install
$ make clean
```

---

## 🖐️ Mini-ejercicio

```bash
$ which make gcc g++
```

**Tienes herramientas de compilacion instaladas?** Si no: `sudo apt install build-essential`. Si si: cuantos paquetes tiene ese meta-paquete?

---

## 💪 Ejercicios (para casa / laboratorio)

1. **Compilacion completa:** Descarga `lynx` (navegador web CLI) desde codigo fuente, compilalo e instalalo siguiendo los 7 pasos del tutorial. Documenta cada error de dependencia y como lo resolviste. Al final, ejecuta `lynx google.com`.

2. **Crear tu propio paquete:** Crea un script `hola.sh`, empaquetalo como `.deb` con `dpkg-deb` (o `.rpm` con `rpmbuild`). Instalalo, ejecutalo, y luego desinstalalo. El paquete debe tener: nombre, version, dependencias y el script en `/usr/local/bin/`.

3. **Automatizacion:** Crea un script `instalar-entorno.sh` que automatice la instalacion de:
   - `build-essential` / "Development Tools"
   - `git`, `curl`, `wget`, `vim`, `htop`, `tree`
   - Un servidor web (nginx o apache2)
   - Que verifique si cada paquete ya esta instalado antes de intentarlo
   - Que genere un log de todo lo instalado

4. **Gestion de repositorios:** Anade el repositorio de Docker oficial a tu sistema, instala Docker Desktop, ejecuta `hello-world`. Investiga la diferencia entre repos oficiales de la distro, PPAs y repos de terceros en terminos de seguridad.

5. **Rollback:** Investiga como hacer downgrade de un paquete en APT (`apt-cache showpkg`) y en DNF (`dnf downgrade`). Simula un escenario donde una actualizacion rompe algo y tienes que volver a la version anterior.

---

## 🔧 Laboratorio práctico (en clase)

### Paso 1: Actualizar el sistema
```bash
sudo apt update
sudo apt upgrade -y
```
Observen cuántos paquetes se actualizan. En RHEL: `sudo dnf upgrade -y`.

### Paso 2: Instalar y explorar un paquete
```bash
sudo apt install lynx -y
lynx --version
which lynx
dpkg -L lynx
```
`dpkg -L` muestra todos los archivos instalados por el paquete.

### Paso 3: Ver dependencias
```bash
apt show lynx | grep Depends
apt-cache depends lynx
```
Observen qué paquetes necesita lynq para funcionar.

### Paso 4: Desinstalar con purge
```bash
sudo apt purge lynx -y
sudo apt autoremove -y
which lynx
```
Verificar que `lynx` ya no existe. `autoremove` elimina dependencias huérfanas.

### Paso 5: Instalar desde código fuente (simulación)
```bash
which make gcc
apt list --installed | grep build-essential
```
Verificar si las herramientas de compilación están instaladas.

**Verificación:** Cada operación debe completarse sin errores. Los comandos `which` y `dpkg -L` confirman instalación/desinstalación.

### 🚀 Desafío individual

Crea un script `instalar-herramientas.sh` que:
1. Verifique si cada herramienta está instalada antes de intentar instalarla
2. Instale: `git`, `curl`, `wget`, `vim`, `htop`, `tree`, `lynx`
3. Genere un log en `/tmp/instalacion.log` con fecha y resultado de cada operación
4. Al final, muestre un resumen de lo instalado vs lo que ya existía

---


## Curiosidad: `apt moo` y el tren `sl`

El gestor APT tiene una vaca:
```
$ apt moo
         (__)
         (oo)
   /------\/
  / |    ||
 *  /\---/\
    ~~   ~~
..."Have you mooed today?"...
```

Y si alguien escribe `sl` en vez de `ls`, aparece un tren a vapor atravesando la terminal. Creado como broma por un desarrollador japones. Un clásico.
