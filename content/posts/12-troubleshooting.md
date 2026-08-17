---
title: "linux-12 Troubleshooting"
date: 2026-08-12
draft: false
featuredImage: "/images/blog-space.jpg"
tags: [diagnostico logs dmesg kernel panic hardware]
categories: ["Linux Admin"]
series: ["Practical Linux System Administration"]
weight: 12
bookAuthor: "Kenneth Hess"
bookTitle: "Practical Linux System Administration"
bookYear: "2023"
---

# Tutorial 12: Troubleshooting en Linux

**Capítulo 12 del libro** — "Practical Linux System Administration" (pags. 157-172)
**Nivel:** Avanzado

---

## ⚡ Para empezar: "It's always DNS"

En la comunidad de administradores de sistemas hay un dicho famoso: "It's always DNS" ("siempre es DNS"). Cuando algo no funciona en la red, el 99% de las veces la causa es el DNS. El otro 1%... también es DNS, pero el administrador aún no lo sabe.

El libro lo plantea distinto: "Troubleshooting is a personal process; not everyone approaches it the same way." Y también: "Some have compared Linux troubleshooting to an 'exercise in futility,' but troubleshooting Linux is neither so dramatic nor difficult if you simply take time to investigate what's going on with your system."

---

## Objetivos

- Resolver un kernel panic reconstruyendo initramfs
- Extraer información de logs del sistema con `dmesg`
- Diagnosticar problemas de software con `apachectl configtest`
- Recopilar información de hardware con `hwinfo`, `lshw`, `lspci`, `lsblk`, `lscpu`
- Crear informes de seguridad automatizados

---

## 1. Revivir el Sistema Operativo

### Kernel Panic: Reconstruir initramfs

Mostrar versión del kernel:

```bash
# uname -r
3.10.0-327.el7.x86_64
```

Recrear initramfs con `dracut`:

```bash
# dracut -f /boot/initramfs-3.10.0-327.el7.x86_64.img 3.10.0-327.el7.x86_64
```

Si `dracut` se niega a sobrescribir:

```bash
# mkinitrd --force /boot/initramfs-3.10.0-327.el7.x86_64.img 3.10.0-327.el7.x86_64
```

Si persiste el problema, instalar un kernel diferente desde ISO:

```bash
# cd /opt/mnt/install/repo/Packages
# rpm -Uvh --root=/mnt/sysimage kernel-3.10.0-1127.10.1.el7.x86_64
```

### 🖐️ Mini-ejercicio

```bash
$ uname -r
```

**¿Qué versión de kernel tienes?** Anótala. Si algún día tu sistema no arranca, esta información será tu punto de partida.

---

## 2. Extraer Logs del Sistema

### `dmesg` — Mensajes del kernel

```bash
$ dmesg | grep -i error
$ dmesg | grep -i failed
$ dmesg | grep -i fault
$ dmesg | grep -i undefined
$ dmesg | grep -i unknown
```

### Logs del sistema

```bash
$ cd /var/log
$ sudo grep -ir error *
$ sudo grep -ir error * > ~/errors.txt
$ grep host errors.txt
```

### 🖐️ Mini-ejercicio

```bash
$ dmesg | grep -i error
$ dmesg | grep -i fail
```

**¿Encuentras algún error o fallo en los mensajes del kernel?** Si hay algo, investiga con `journalctl -xe` para más contexto.

---

## 3. Troubleshooting de Software

### Verificar configuración de Apache

```bash
$ apachectl configtest
Syntax OK
```

Cuando hay error:

```bash
$ apachectl configtest
AH00526: Syntax error on line 34 of /etc/httpd/conf/httpd.conf:
Invalid command 'ServerBoot', perhaps misspelled or defined by a module not included
```

### Ver estado del servicio

```bash
$ sudo apachectl restart
Job for httpd.service failed...
$ systemctl status httpd.service
$ sudo journalctl -xe
```

### Firewalls durante troubleshooting

> "Avoid the temptation to disable your firewall during troubleshooting. Instead, add new rules."

---

## 4. Troubleshooting de Hardware

### Comandos de información de hardware

**hwinfo** — Informe completo de hardware:

```bash
# hwinfo --short
```

**lshw** — Información detallada:

```bash
# lshw
# lshw -html          # Reporte HTML
# lshw -sanitize      # Sin información sensible
```

**lspci** — Dispositivos PCI:

```bash
# lspci
00:00.0 Host bridge: Intel Corporation 440FX - 82441FX PMC [Natoma] (rev 02)
00:02.0 VGA compatible controller: VMware SVGA II Adapter
00:03.0 Ethernet controller: Intel Corporation 82540EM Gigabit Ethernet Controller
```

**lsblk** — Dispositivos de bloque:

```bash
# lsblk
NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda             8:0    0    8G  0 disk
├─sda1          8:1    0    1G  0 part /boot
└─sda2          8:2    0    7G  0 part
  ├─cl-root   253:0    0  6.2G  0 lvm  /
  └─cl-swap   253:1    0  820M  0 lvm  [SWAP]

# lsblk -fm       # Información extendida
```

**lscpu** — Arquitectura de CPU:

```bash
# lscpu
Architecture:        x86_64
CPU(s):              1
Model name:          Intel(R) Core(TM) i5-5350U CPU @ 1.80GHz
Hypervisor vendor:   KVM
```

---

## 5. Informe de Seguridad Automatizado

Script `daily_report.sh` del libro:

```bash
#!/bin/bash
#Daily Report Script
today=`date +%m-%d-%Y`
touch /opt/note/$today.html
echo "<pre>" >> /opt/note/$today.html
echo "Last Log " >> /opt/note/$today.html
last | grep root >> /opt/note/$today.html
echo "Non-privileged accounts in the Last Log " >> /opt/note/$today.html
last | grep -v root >> /opt/note/$today.html
echo " " >> /opt/note/$today.html
echo "Root Accounts " >> /opt/note/$today.html
grep :0 /etc/passwd >> /opt/note/$today.html
echo " " >> /opt/note/$today.html
echo "Files modified since yesterday " >> /opt/note/$today.html
find /etc -mtime -1 >> /opt/note/$today.html
echo "</pre>" >> /opt/note/$today.html
```

Comparar informes para detectar brechas:

```bash
# diff 10-20-2022.html 10-21-2022.html
4a5
> jamd:x:0:0:root:/root:/bin/sh
```

Esto revela una cuenta root ilegítima creada por un intruso.

> "Intruders rarely lock out system administrators because doing so will cause action and remediation."

---

## 🔧 Laboratorio práctico (en clase)

### Paso 1: Diagnóstico con dmesg

Explora mensajes del kernel:

```bash
$ dmesg | less
$ dmesg | grep -i error
$ dmesg | grep -i fail
$ dmesg | tail -50
```

Busca errores de hardware:

```bash
$ dmesg | grep -E "(ata|scsi|sd\s|nvme)" | grep -i error
```

Usa `dmesg -w` para monitorear en tiempo real (déjalo corriendo en una terminal mientras haces otros pasos).

### Paso 2: Diagnóstico con journalctl

Lista los boots disponibles:

```bash
$ journalctl --list-boots
```

Revisa logs del boot actual:

```bash
$ journalctl -b 0 -p err
```

Examina servicios específicos:

```bash
$ journalctl -u sshd --since "1 hour ago"
$ journalctl -u apache2 --since today
$ journalctl -u networking --no-pager | tail -30
```

Sigue los logs en vivo de un servicio:

```bash
$ journalctl -fu apache2
```

### Paso 3: Verificación de Apache

Prueba la sintaxis de la configuración de Apache:

```bash
$ sudo apachectl configtest
$ sudo apache2ctl configtest   # Debian/Ubuntu
```

Verifica que el servicio esté activo:

```bash
$ sudo systemctl status apache2
$ curl -I http://localhost
```

### Paso 4: Información de hardware

Comando para listar dispositivos PCI:

```bash
$ lspci -nn
$ lspci -v | grep -E "(VGA|Ethernet|Controller)"
```

Información de CPU:

```bash
$ lscpu
$ cat /proc/cpuinfo | grep "model name"
```

Información de memoria y sistema:

```bash
$ sudo lshw -short
$ sudo lshw -class disk -class storage
```

Resumen de hardware:

```bash
$ lshw -short 2>/dev/null | head -30
```

### Paso 5: Crear script de reporte diario

Crea el archivo `daily_report.sh`:

```bash
$ nano ~/daily_report.sh
```

Contenido del script:

```bash
#!/bin/bash
echo "=== REPORTE DIARIO: $(date) ==="
echo ""
echo "--- Mensajes de error del kernel ---"
dmesg | grep -i error | tail -10
echo ""
echo "--- Servicios fallidos ---"
systemctl --failed --no-pager
echo ""
echo "--- Uso de disco ---"
df -h | grep -v tmpfs
echo ""
echo "--- Memoria ---"
free -h
echo ""
echo "--- Carga del sistema ---"
uptime
echo ""
echo "--- Conexiones de red activas ---"
ss -tun | tail -20
```

Hazlo ejecutable y pruébalo:

```bash
$ chmod +x ~/daily_report.sh
$ ~/daily_report.sh
```

**Verificación:** El script debe ejecutarse sin errores y mostrar información coherente del sistema. Los comandos `dmesg`, `journalctl` y las herramientas de hardware deben devolver datos legibles.

### 🚀 Desafío individual

**Escenario:** Un colega te dice "Apache no arranca, pero no sé por qué". El archivo `/etc/apache2/sites-available/mi-sitio.conf` tiene un error tipográfico (por ejemplo, `DocumentRoot /var/www/html` escrito como `DocumentRoots /var/www/html`).

1. Introduce el error deliberadamente en el archivo de configuración.
2. Usa `apachectl configtest` para detectarlo.
3. Revisa `journalctl -u apache2 --since today` para ver el mensaje de error.
4. Corrige el error y verifica que el servicio arranque correctamente.
5. Responde: ¿qué tan rápido pudiste encontrar y solucionar el problema?

---

## 💪 Ejercicios (para casa / laboratorio)

1. Ejecute `dmesg | grep -i error` en su sistema e interprete los resultados
2. Use `apachectl configtest` si tiene Apache instalado, o instálelo para probar
3. Ejecute `hwinfo --short`, `lshw`, `lspci`, `lsblk` y `lscpu` y compare la información
4. Cree el script `daily_report.sh` y ejecútelo manualmente
5. Programe el script con cron para que se ejecute diariamente
6. Use `diff` para comparar dos informes consecutivos

---


## Curiosidad: "Kernel panic — not syncing"

El kernel panic de Linux muestra el mensaje: "Kernel panic — not syncing". En palabras de Linus Torvalds: "I'm sorry, but I have to stop this system because something really bad happened. I'm going to print some information that might help you diagnose the problem."

El comando `dmesg` debe su nombre a "diagnostic messages" (mensajes de diagnóstico). Antes de `systemd`, los logs del kernel se almacenaban en un buffer circular en memoria que `dmesg` leía. Hoy en día `journalctl -k` hace lo mismo, pero `dmesg` sigue funcionando igual que siempre.

La reconstrucción de initramfs con `dracut` es una de esas tareas que parecen de otro siglo, pero cuando tu sistema no arranca por un kernel corrupto, saber hacerlo a mano marca la diferencia entre 5 minutos de downtime y una reinstalación completa.
