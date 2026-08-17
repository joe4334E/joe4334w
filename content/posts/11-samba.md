---
title: "linux-11 Samba"
date: 2026-08-11
draft: false
featuredImage: "/images/blog-space.jpg"
tags: [samba smb windows linux comparticion]
categories: ["Linux Admin"]
series: ["Practical Linux System Administration"]
weight: 11
bookAuthor: "Kenneth Hess"
bookTitle: "Practical Linux System Administration"
bookYear: "2023"
---

# Tutorial 11: Desplegando Samba para Compatibilidad con Windows

**Capítulo 11 del libro** — "Practical Linux System Administration" (pags. 145-156)
**Nivel:** Intermedio

---

## ⚡ Para empezar: Microsoft odiaba Samba (y con razón)

Samba implementa el protocolo SMB (Server Message Block) de Microsoft mediante ingeniería inversa. Esto siempre fue polémico. En los 90 y 2000, Microsoft intentó "embrace, extend, extinguish" con SMB/CIFS, añadiendo extensiones propietarias que Samba tenía que reimplementar constantemente.

El creador, Andrew Tridgell, necesitaba montar un disco de una DEC Alpha en una PC con Linux. Hizo ingeniería inversa al protocolo SMB. El resultado: Samba, uno de los proyectos de software libre más exitosos. Hoy, es mantenido por el equipo Samba y usado por millones de organizaciones para interoperabilidad Windows-Linux.

---

## Objetivos

- Comprender la interoperabilidad Linux-Windows con Samba
- Planificar un entorno Samba según necesidades del negocio
- Instalar Samba y sus dependencias
- Crear usuarios y grupos Samba
- Configurar directorios compartidos
- Montar recursos compartidos de Windows desde Linux

---

## 1. Conceptos Clave

> "Interoperability is the capability for disparate systems such as Windows, Linux, Unix, and macOS to coexist and work together." — Capítulo 11

Samba permite a sistemas Linux:
- Compartir directorios, sistemas de archivos e impresoras con Windows
- Montar recursos compartidos de Windows
- Participar en autenticación y autorización de dominios Windows

---

## 2. Planificar un Recurso Compartido

Preguntas clave antes de crear un recurso compartido:

1. ¿Debe restringirse a un grupo específico?
2. ¿Todos en el grupo pueden copiar y crear archivos?
3. ¿Hay permisos por defecto para archivos subidos?
4. ¿Se necesitan carpetas de solo lectura?
5. ¿Es permanente o temporal?

### Escenario del libro: Departamento de Facilities

El manager respondió: restringido a Facilities, todos necesitan lectura/escritura, y una carpeta Policies de solo lectura.

---

## 3. Instalación

```bash
$ sudo yum install samba
```

Esto instala 7 paquetes (total 7.4 MB descargados, 25 MB instalados).

Habilitar e iniciar servicios:

```bash
$ sudo systemctl enable smb
$ sudo systemctl enable nmb
$ sudo systemctl start smb
$ sudo systemctl start nmb
```

### 🖐️ Mini-ejercicio

```bash
$ systemctl status smb
$ systemctl status nmb
```

**¿Están ambos servicios activos?** Si Samba no está instalado, ejecuta `sudo apt install samba` (o `sudo yum install samba`) si quieres seguirlo en vivo.

---

## 4. Configuración

### 4.1 Crear grupo y usuarios:

```bash
$ sudo groupadd -g 9001 facilities
$ sudo usermod -g facilities atran
$ sudo usermod -g facilities areed
$ sudo usermod -g facilities akumar
```

### 4.2 Crear y preparar directorio compartido:

```bash
$ sudo mkdir /opt/facilities
$ sudo chgrp facilities /opt/facilities
$ sudo chmod 770 /opt/facilities
$ sudo mkdir /opt/facilities/Policies
$ sudo chmod 740 /opt/facilities/Policies
```

### 4.3 Agregar usuarios Samba:

```bash
$ sudo smbpasswd -a atran
New SMB password:
Retype new SMB password:
Added user atran.
```

> "Samba usernames and passwords must match exactly with those configured on Windows systems."

### 🖐️ Mini-ejercicio

Crea un grupo `prueba-samba` y un directorio `/opt/prueba-samba`:

```bash
$ sudo groupadd prueba-samba
$ sudo mkdir /opt/prueba-samba
$ sudo chgrp prueba-samba /opt/prueba-samba
$ sudo chmod 770 /opt/prueba-samba
```

**Verifica los permisos con `ls -la /opt/prueba-samba`.**

### 4.4 Configurar `/etc/samba/smb.conf`:

Sección [global]:

```ini
[global]
load printers = yes
passdb backend = tdbsam
security = user
cups options = raw
workgroup = YOUR_DOMAIN_NAME
printcap name = cups
printing = cups
os level = 20
netbios name = SERVER1
browseable = yes
interfaces = lo eth0 192.168.1.0/24
hosts allow = 127. 192.168.1.
```

Sección del recurso compartido [Facilities]:

```ini
[Facilities]
create mode = 0660
writeable = yes
path = /opt/facilities
comment = Facilities Group Share
directory mode = 0770
only user = yes
valid users = @facilities
browseable = yes
```

---

## 5. Acceder a Recursos Compartidos

### Desde macOS: Aparecen en el Finder automáticamente al navegar la red.

### Desde Windows: Mapear una unidad (ej. G:) a `\\server1\Facilities`.

### Desde Linux (montar recurso de Windows):

```bash
$ smbclient -U "Ken Hess" -L WIN10
$ sudo mount -t cifs \\\\WIN10\\Files /mnt
```

> "Use four backslashes (\\\\) and two for the UNC mapping command's share name (\\)."

El tipo de sistema de archivos debe ser `cifs`:

```
mount: /mnt: special device \\WIN10\Files does not exist.
```

---

## 6. Archivo `lmhosts`

Similar a `/etc/hosts` pero para NetBIOS:

```
127.0.0.1       localhost
192.168.1.50    testserv1       #PRE
192.168.1.100   DC1             #PRE #DOM:WEST
```

---

## 🔧 Laboratorio práctico (en clase)

### Paso 1: Instalar Samba

```bash
$ sudo apt update && sudo apt install -y samba samba-common smbclient cifs-utils
```

Verifica que los servicios estén corriendo:

```bash
$ sudo systemctl status smbd
$ sudo systemctl status nmbd
```

Samba separa `smbd` (servicio de archivos/impresión SMB/CIFS) de `nmbd` (servicio de nombres NetBIOS).

### Paso 2: Crear usuarios del sistema y usuarios Samba

Crea usuarios sin shell de login (solo para acceso Samba):

```bash
$ sudo useradd -M -s /usr/sbin/nologin ventas
$ sudo useradd -M -s /usr/sbin/nologin admin
```

Asigna contraseña Samba a cada uno:

```bash
$ sudo smbpasswd -a ventas
$ sudo smbpasswd -a admin
```

Habilita los usuarios en Samba:

```bash
$ sudo smbpasswd -e ventas
$ sudo smbpasswd -e admin
```

### Paso 3: Configurar un recurso compartido en smb.conf

Haz backup del archivo original y edita la configuración:

```bash
$ sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
$ sudo nano /etc/samba/smb.conf
```

Agrega al final del archivo:

```ini
[compartido]
   path = /srv/samba/compartido
   browseable = yes
   read only = no
   guest ok = no
   valid users = ventas admin
   create mask = 0660
   directory mask = 0770
```

### Paso 4: Crear el directorio y establecer permisos

```bash
$ sudo mkdir -p /srv/samba/compartido
$ sudo chown -R root:sambashare /srv/samba/compartido
$ sudo chmod 2770 /srv/samba/compartido
$ sudo usermod -aG sambashare ventas
$ sudo usermod -aG sambashare admin
```

Reinicia Samba:

```bash
$ sudo systemctl restart smbd nmbd
```

### Paso 5: Probar con smbclient

Desde el mismo servidor (o desde un cliente en la red):

```bash
$ smbclient -L //localhost -U ventas
$ smbclient //localhost/compartido -U ventas
```

Dentro de smbclient, prueba:

```
smb: \> mkdir test
smb: \> put /etc/hosts prueba.txt
smb: \> ls
smb: \> quit
```

### Paso 6: Montar el recurso CIFS

```bash
$ mkdir ~/mnt-samba
$ sudo mount -t cifs //localhost/compartido ~/mnt-samba -o username=ventas,uid=$(id -u),gid=$(id -g),noexec
$ ls ~/mnt-samba/
```

Verifica el montaje:

```bash
$ mount | grep cifs
$ df -h | grep cifs
```

**Verificación:** El recurso compartido debe ser accesible por ambos usuarios, permitir lectura/escritura, y aparecer en la salida de `smbclient -L`.

### 🚀 Desafío individual

**Escenario:** La empresa "TechCorp" necesita dos recursos compartidos:
1. `[ventas]` — solo accesible por el grupo `ventas` (lectura/escritura)
2. `[directivos]` — solo accesible por el grupo `directivos` (lectura/escritura)

Crea los grupos, usuarios, directorios y la configuración smb.conf necesaria. Usa `valid users = @grupo` para restringir por grupo. Verifica que un usuario de ventas NO pueda acceder al recurso de directivos.

---

## 💪 Ejercicios (para casa / laboratorio)

1. Instale Samba en su sistema
2. Cree un grupo `ventas` y tres usuarios Samba
3. Configure un recurso compartido `/opt/ventas` con permisos 770
4. Agregue una subcarpeta `Plantillas` de solo lectura
5. Verifique la configuración con `testparm`
6. Desde otro sistema Linux, monte el recurso compartido usando `mount -t cifs`

---


## Curiosidad: Samba no es solo un nombre

Samba debe su nombre al protocolo SMB (Server Message Block) que implementa. El creador, Andrew Tridgell, buscó un nombre que incluyera "SMB". La "a" añadida forma "Samba" — como el baile brasileño.

Pero hay más: Tridgell también creó rsync, el estándar de facto para transferencia de archivos. Y en 2005, hizo ingeniería inversa del protocolo BitKeeper (un sistema de control de versiones propietario que Linus Torvalds usaba para Linux). Esto llevó a que BitKeeper retirara su licencia gratuita para Linux, lo que a su vez llevó a Linus a crear Git en 2005. Así que sin Tridgell y Samba... quizás no existiría Git.
