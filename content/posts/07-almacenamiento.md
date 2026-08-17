---
title: "linux-07 Almacenamiento"
date: 2026-08-05
draft: false
featuredImage: "/images/blog-space.jpg"
tags: [discos particiones lvm fstab tmpfs]
categories: ["Linux Admin"]
series: ["Practical Linux System Administration"]
weight: 07
bookAuthor: "Kenneth Hess"
bookTitle: "Practical Linux System Administration"
bookYear: "2023"
---

# Tutorial 7: Gestion de almacenamiento

**Capítulo 7 del libro** — "Practical Linux System Administration" (pags. 77-94)
**Nivel:** Intermedio

---

## ⚡ Para empezar: "dd: el Disk Destroyer"

En la comunidad Linux, `dd` tiene un apodo: "disk destroyer". Un solo caracter mal escrito — `dd if=/dev/sda of=/dev/sdb` en vez de `dd if=/dev/sdb of=/dev/sda` — y has borrado el disco equivocado. No hay confirmacion, no hay papelera de reciclaje. Solo silencio.

El libro menciona el **"scream test"** para desmantelar discos: desconectas un sistema de la red durante 2 semanas y esperas a ver si alguien grita. Si nadie reclama, puedes darlo de baja. Asi funciona la administracion de almacenamiento en Linux.

---

## Objetivos

- Comprender discos, sistemas de archivos, montaje y LVM
- Anadir un nuevo disco al sistema
- Crear particiones y sistemas de archivos
- Implementar volumenes logicos (LVM)
- Extender volumenes logicos
- Desmantelar y desechar discos

---

## 1. Conceptos de almacenamiento

### Discos

> "Los discos son dispositivos que llamamos discos duros (HDD), pero tambien pueden referirse a SSD y USB."

### Sistemas de archivos (filesystems)

> "Un sistema de archivos es una construccion organizativa que permite el almacenamiento y recuperacion de archivos. Los sistemas Linux actuales ofrecen ZFS, XFS o ext4."

### Montaje y puntos de montaje

> "Solo el usuario root, o un usuario con privilegios sudo, puede montar un sistema de archivos."

```
$ sudo mount /dev/sdd1 /software
```

El directorio donde se monta se llama **punto de montaje**.

> "Los sistemas Linux proveen un punto de montaje generico, `/mnt`, para montajes temporales."

> "Un punto de montaje debe existir antes de montar un disco."

**Montaje automatico en `/etc/fstab`:**
```
UUID=324ddbc2-353b-4221-a80e-49ec356678dc /opt/software xfs defaults 0 0
```

### Volumenes fisicos y logicos (LVM)

> "Un volumen fisico (PV) es una particion o disco gestionado por LVM. Un grupo de volumenes (VG) contiene los PVs. Un volumen logico (LV) es equivalente a una particion."

Ventajas de LVM: redimensionar en vivo, abarcar multiples discos.

### Verificar espacio

```
$ df -h
$ sudo du -h /var/log
```

### Swap

> "Swap extiende la memoria del sistema mas alla de los limites de RAM. El kernel usa swap para escribir programas inactivos de la memoria al disco."

> "Expandir swap no es un remedio para resolver problemas de memoria. La solucion aceptada es anadir mas RAM."

### tmpfs

> "tmpfs es el sistema de archivos temporal basado en RAM preferido. Tiene limite de capacidad."

```
$ mount | grep tmpfs
```

### ramfs vs tmpfs

Ambos son sistemas de archivos en memoria RAM, pero con diferencias clave:

| Característica      | ramfs                     | tmpfs                        |
|--------------------|---------------------------|------------------------------|
| Límite de tamaño   | Ilimitado (puede agotar RAM) | Configurable por defecto (50% RAM) |
| Swap               | No usa swap               | Puede usar swap              |
| Propósito          | Cachés temporales         | `/tmp`, `/dev/shm`           |
| Peligro            | Puede llenar RAM y colgar | Límite seguro                |

> "If given the choice, always use tmpfs over ramfs. An unconstrained ramfs filesystem can grow to fill all available memory and hang your system."

El kernel usa tmpfs internamente para tres propósitos:
- `/tmp` — Archivos temporales de usuario (si se habilita tmp.mount)
- `/dev/shm` — Memoria compartida POSIX
- `/run` — Archivos de estado de procesos en ejecución

Para crear un tmpfs personalizado:
```
$ sudo mount -t tmpfs -o size=1G tmpfs /mnt/mi-ramdisk
$ df -h /mnt/mi-ramdisk
```

### 🖐️ Mini-ejercicio

Ejecuta estos comandos en tu terminal:

```bash
$ df -h
$ lsblk
```

**¿Cuantos discos tienes? ¿Que sistemas de archivos usan?** Compara con tu companero de al lado.

---

## 2. Anadir un nuevo disco

### Identificar el nuevo disco

```
$ sudo fdisk -l
```

O con `lsblk`:
```
$ lsblk
```

### Crear una particion con fdisk

```
$ sudo fdisk /dev/sdb
```

Comandos dentro de fdisk:
- `n` - nueva particion
- `p` - primaria
- `w` - escribir cambios

### Crear el sistema de archivos

```
$ sudo mkfs.xfs /dev/sdb1
```

### Montar la particion

```
$ sudo mkdir /opt/software
$ sudo mount /dev/sdb1 /opt/software
```

Verificar montaje:
```
$ mount | grep sdb1
```

### Montaje persistente en /etc/fstab

Obtener el UUID:
```
$ sudo blkid /dev/sdb1
```

Anadir a `/etc/fstab`:
```
UUID=ca2701e0-3e75-4930-b14e-d83e19a5cffb /opt/software xfs defaults 0 0
```

---

## 3. Implementar LVM (Logical Volume Manager)

### Paso 1: Crear el Volumen Fisico (PV)

```
$ sudo pvcreate /dev/sdb
$ sudo pvs
$ sudo pvdisplay /dev/sdb
```

### Paso 2: Crear el Grupo de Volumenes (VG)

```
$ sudo vgcreate vgsw /dev/sdb
$ sudo vgs
$ sudo vgdisplay vgsw
```

### Paso 3: Crear el Volumen Logico (LV)

```
$ sudo lvcreate -L 1G -n software-lv vgsw
$ sudo lvs
$ sudo lvdisplay /dev/vgsw/software-lv
```

### Paso 4: Crear el sistema de archivos

```
$ sudo mkfs.xfs /dev/vgsw/software-lv
```

### Paso 5: Montar

```
$ sudo mkdir /sw
$ sudo mount /dev/vgsw/software-lv /sw
$ df -h /sw
```

### Paso 6: Anadir a /etc/fstab

```
/dev/vgsw/software-lv /sw xfs defaults 0 0
```

> "No uses UUID para volumenes logicos en /etc/fstab. Usa el nombre del dispositivo."

### Extender un volumen logico

```
$ sudo lvextend -l +100%FREE /dev/vgsw/software-lv
$ sudo xfs_growfs /dev/vgsw/software-lv
$ df -h /sw
```

> "No puedes reducir (shrink) un volumen XFS."

### 🖐️ Mini-ejercicio

En una VM de pruebas, crea un volumen logico de 500MB:

```bash
$ sudo lvcreate -L 500M -n prueba-lv vgsw
$ sudo mkfs.xfs /dev/vgsw/prueba-lv
$ sudo mkdir /mnt/prueba-lvm
$ sudo mount /dev/vgsw/prueba-lv /mnt/prueba-lvm
$ df -h /mnt/prueba-lvm
```

**Desmontalo y elimina el LV cuando termines:** `sudo umount /mnt/prueba-lvm && sudo lvremove /dev/vgsw/prueba-lv`.

---

## 4. Desmantelamiento y eliminacion de discos

Proceso recomendado por el libro:
1. **Notificacion** a stakeholders (3-4 semanas)
2. **"Scream test"** - desconectar de red 2+ semanas, ver si alguien reclama
3. **Apagar** sistema (2+ semanas)
4. **Borrado de discos** - usar DBAN (Darik's Boot and Nuke) para HDDs
5. **Desmontaje** y paletizado
6. **Eliminacion** - reciclaje, trituracion o reventa

---

## 🔧 Laboratorio práctico (en clase)

### Paso 1: Agregar un disco virtual y examinarlo

```bash
$ lsblk
$ sudo fdisk -l /dev/sdb
```
`lsblk` lista todos los dispositivos de bloque. Si tu sistema no tiene un `/dev/sdb`, puedes crear un archivo de imagen que simule un disco:

```bash
$ dd if=/dev/zero of=/tmp/disco_virtual.img bs=1M count=512
$ sudo losetup -fP /tmp/disco_virtual.img
$ lsblk  # buscar /dev/loop0
```

### Paso 2: Particionar con fdisk

```bash
$ sudo fdisk /dev/loop0
```
Dentro de fdisk:
- `n` → nueva partición primaria
- `p` → tipo primaria
- Acepta valores por defecto para usar todo el espacio
- `w` → escribe los cambios

Verifica con `sudo fdisk -l /dev/loop0`.

### Paso 3: Formatear con XFS

```bash
$ sudo mkfs.xfs /dev/loop0p1
```
XFS es el sistema de archivos por defecto en RHEL/CentOS. Para Debian/Ubuntu instala `xfsprogs` si no está disponible: `sudo apt install xfsprogs`.

### Paso 4: Montar y agregar a fstab

```bash
$ sudo mkdir /mnt/disco_prueba
$ sudo mount /dev/loop0p1 /mnt/disco_prueba
$ sudo blkid /dev/loop0p1  # obtener UUID
```
Añade al final de `/etc/fstab`:
```
UUID=<el-uuid> /mnt/disco_prueba xfs defaults 0 2
```
Prueba con `sudo mount -a` para validar que no hay errores.

### Paso 5: Crear una estructura LVM completa

```bash
$ sudo pvcreate /dev/loop0p1
$ sudo vgcreate vg_datos /dev/loop0p1
$ sudo lvcreate -n lv_apps -L 200M vg_datos
$ sudo mkfs.ext4 /dev/vg_datos/lv_apps
$ sudo mkdir /mnt/lvm_apps
$ sudo mount /dev/vg_datos/lv_apps /mnt/lvm_apps
```
Comandos de verificación: `sudo pvs`, `sudo vgs`, `sudo lvs`.

**Verificación:** `df -h | grep mnt` muestra ambos montajes operativos.

### 🚀 Desafío individual

Extiende el Logical Volume `lv_apps` para que ocupe el 100% del espacio libre del VG y redimensiona el sistema de archivos sobre la marcha:

```bash
# Asumiendo que aún hay espacio libre en el VG
$ sudo lvextend -l +100%FREE /dev/vg_datos/lv_apps
$ sudo resize2fs /dev/vg_datos/lv_apps   # ext4
# Si fuera XFS: sudo xfs_growfs /mnt/lvm_apps
$ df -h /mnt/lvm_apps  # verificar nuevo tamaño
```

¿Qué comando usarías para ver cuánto espacio libre queda realmente en el VG?

---

## 💪 Ejercicios (para casa / laboratorio)

1. Ejecuta `df -h` y `lsblk` para ver los discos de tu sistema
2. Crea un directorio `/mnt/prueba` y monta un disco temporal
3. Anade un nuevo disco virtual a una VM, crea una particion con `fdisk`, formateala con `mkfs.xfs` y montala
4. Crea un PV, VG y LV usando LVM (en una VM de pruebas)
5. Extiende el LV al 100% del espacio libre y redimensiona el filesystem con `xfs_growfs`
6. Anade una entrada en `/etc/fstab` para un montaje persistente
7. Explica la diferencia entre PV, VG y LV

---


## Curiosidad: "NVIDIA, FUCK YOU"

En 2012, Linus Torvalds dio una conferencia en Finlandia. Al ser preguntado sobre el soporte de NVIDIA en Linux, miro a la camara y dijo: "So NVIDIA, fuck you!" ("Asi que NVIDIA, que les jodan"). Luego explico que NVIDIA era la peor compania con la que Linux habia tenido que lidiar. Las camaras captaron su carcajada inmediatamente despues. Desde entonces, el driver NVIDIA de codigo abierto (Nouveau) ha mejorado, pero Linus no ha vuelto a disculparse.

Sobre ext4: el pase de ext2 a ext3 fue simplemente anadir journaling. De ext3 a ext4 fue extender ext3 con extents y mayor tamanio de archivo. La regla no escrita: "cada version de ext necesita tener el doble de letras que la anterior" (broma de los desarrolladores).
