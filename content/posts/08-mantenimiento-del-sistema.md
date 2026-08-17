---
title: "linux-08 Mantenimiento del Sistema"
date: 2026-08-06
draft: false
featuredImage: "/images/blog-space.jpg"
tags: [limpieza tmp home fdupes cuotas sysstat]
categories: ["Linux Admin"]
series: ["Practical Linux System Administration"]
weight: 08
bookAuthor: "Kenneth Hess"
bookTitle: "Practical Linux System Administration"
bookYear: "2023"
---

# Tutorial 8: Mantenimiento de la salud del sistema

**Capítulo 8 del libro** — "Practical Linux System Administration" (pags. 95-119)
**Nivel:** Intermedio

---

## ⚡ Para empezar: "I'm sorry, Dave. I'm afraid I can't do that"

`sudo rm -rf /` es el comando mas peligroso de Linux. Borra todo el sistema sin preguntar. En 2005, un bug en la version 1.1.4 de `coreutils` elimino la proteccion que obligaba a usar `--no-preserve-root`. Desde entonces, `rm -rf /` muestra un error... a menos que ejecutes `rm -rf /*` o `rm -rf .*`. Linux confia en que sabes lo que haces. A veces, no es asi.

Pero este tutorial no trata de romper sistemas, sino de mantenerlos sanos: limpiar `/tmp`, mover `/home`, deduplicar archivos, aplicar cuotas, parchear y monitorizar.

---

## Objetivos

- Limpiar el directorio `/tmp` con tmp.mount
- Mover `/home` a su propia particion
- Deduplicar archivos con `fdupes`
- Implementar cuotas de disco
- Parchear el sistema
- Mantener cuentas de usuario y grupos
- Monitorizar con sysstat

---

## 1. Limpiar `/tmp`

> "El directorio `/tmp` es un directorio compartido con todos los usuarios, aplicaciones y procesos del sistema."

**Solucion recomendada:** Habilitar `tmp.mount` para crear un tmpfs:

```
$ sudo systemctl enable tmp.mount
$ sudo systemctl start tmp.mount
$ df -h /tmp
tmpfs       405M     0  405M   0% /tmp
```

> "Ahora `/tmp` ya no es un subdirectorio de `/`. Habilitar tmp.mount hace que esta configuracion sea persistente."

### 🖐️ Mini-ejercicio

```bash
$ df -h /tmp
```

**¿`/tmp` es una particion separada o parte de `/`?** Si ves `tmpfs`, ya esta limpio. Si ves `/dev/sda`, considera habilitar tmp.mount.

---

## 2. Mover `/home` a su propia particion

Pasos del libro:
1. Anadir un nuevo disco
2. Crear particion con `fdisk`
3. Crear filesystem: `sudo mkfs.ext4 /dev/sdc1`
4. Montar temporalmente: `sudo mount /dev/sdc1 /mnt`
5. Copiar archivos: `sudo cp -a /home/* /mnt`
6. Eliminar contenido de `/home`: `sudo rm -rf /home/*`
7. Desmontar: `sudo umount /mnt`
8. Montar en `/home`: `sudo mount /dev/sdc1 /home`
9. Anadir a `/etc/fstab`:
```
/dev/sdc1 /home ext4 defaults 0 0
```

---

## 3. Deduplicar con `fdupes`

Instalar desde GitHub (version recomendada por el libro):
```
$ git clone https://github.com/tobiasschulz/fdupes
$ cd fdupes
$ make fdupes
$ sudo make install
```

**Listar duplicados:**
```
$ fdupes -rS /opt/shared
```

**Generar reporte:**
```
$ fdupes -mr /opt/shared
8 duplicate files (in 2 sets), occupying 214 bytes.
```

**Reemplazar duplicados con hard links (recomendado):**
```
$ fdupes -rL /opt/shared
```

**Eliminar selectivamente (NO recomendado en produccion):**
```
$ fdupes -rd /opt/shared
```

---

## 4. Cuotas de disco

### Instalar quota
```
$ sudo yum install quota     # RHEL
$ sudo apt install quota     # Debian
```

### Configurar en /etc/fstab
```
/dev/sdc1 /home xfs defaults,usrquota,grpquota 0 0
```

### Crear archivos de cuota
```
$ sudo touch /home/quota.group /home/quota.user
$ sudo quotaon /home
```

### Aplicar cuota a un usuario
```
$ sudo xfs_quota -x -c 'limit -u bsoft=50m bhard=80m isoft=60 ihard=80 djones' /home
```

### Ver la cuota en accion
```
$ su - djones
$ head -c 51MB /dev/urandom > fillit.txt
head: error writing 'standard output': Disk quota exceeded
```

### Eliminar cuota
```
$ sudo xfs_quota -x -c 'limit -u bsoft=0 bhard=0 isoft=0 ihard=0 djones' /home
```

---

## 5. Parchear el sistema

> "Parchear es una tarea esencial. Un parche de software corrige un problema especifico."

### RHEL/CentOS
```
$ sudo yum update
```

> "El comportamiento predeterminado de YUM/DNF es NO instalar. `N` es la respuesta predeterminada. Debes responder `y` explicitamente."

### Debian/Ubuntu
```
$ sudo apt update
$ sudo apt upgrade
```

> "En Debian, `Y` es la respuesta predeterminada (mayuscula). Presionar Enter INSTALA las actualizaciones."

**Actualizaciones automaticas en Debian:**
```
$ sudo apt install unattended-upgrades
```

> "unattended-upgrades is a package you can install that will allow you to update your Debian-based system automatically."

### Programa de parcheo recomendado:
- **Sistemas de prueba:** Actualizacion manual cada martes
- **Sistemas de desarrollo:** Cada dos jueves
- **Produccion:** Ultimo domingo del mes

### 🖐️ Mini-ejercicio

```bash
$ sudo apt update          # Debian/Ubuntu
$ sudo yum check-update    # RHEL/CentOS
```

**¿Cuantas actualizaciones hay disponibles?** Anota el numero. Revisalo cada semana y observa como cambia.

---

## 6. Mantener cuentas de usuario

### Convencion de nombres (Tabla 8-2 del libro)

| Nombre | Apellido | Usuario |
|--------|----------|---------|
| Jose | Alvarez | jalvarez |
| Paula | Anderson | panderso |
| Vivek | Kundra | vkundra |
| Sylvia | Goldstein | sgoldste |

### Politica de retencion de cuentas

Establecer inactividad de 15 dias:
```
$ sudo useradd -D -f 15
```

Verificar estado de cuenta:
```
$ sudo passwd -S ndavis
ndavis PS 2022-02-13 0 99999 7 15 (Password set, SHA512 crypt.)
$ sudo passwd -S asmith
asmith LK 2022-02-12 0 99999 7 15 (Password locked.)
```

Desbloquear cuenta:
```
$ sudo passwd -u asmith
```

Configurar parametros de contrasena:
```
$ sudo chage -m 1 -M 90 --inactive 15 asmith
```

Script del libro para aplicar a todos los usuarios:
```bash
#!/bin/bash
egrep ^[^:]+:[^\!*] /etc/shadow | cut -d: -f1 | grep -v root > user-list.txt
for user in `more user-list.txt`
do
    chage -m 1 -M 90 -I 15 $user
done
```

### Retirar grupos vacios

Listar miembros:
```
$ sudo groupmems -g operations -l
```

Buscar archivos del grupo:
```
$ sudo find / -group engineering
```

Eliminar grupo:
```
$ sudo groupdel engineering
```

---

## 7. Monitorizar con sysstat

### Instalar y configurar

En Debian/Ubuntu, editar `/etc/default/sysstat`:
```
ENABLED="true"
```
Luego:
```
$ sudo service sysstat restart
```

### Binarios de sysstat (Tabla 8-5 del libro)

| Comando | Descripcion |
|---------|-------------|
| `sar` | System activity reporter |
| `sadf` | Formateador de datos sar |
| `iostat` | Estadisticas de CPU y I/O |
| `mpstat` | Estadisticas de procesadores |
| `pidstat` | Estadisticas por proceso |
| `cifsiostat` | Estadisticas CIFS/Samba |
| `tapestat` | Estadisticas de cintas |

### Ejemplos de sar

```
$ sar                        # CPU (por defecto)
$ sar -b                     # I/O statistics
$ sar 5 5                    # Cada 5 seg, 5 muestras
$ sadf -d                    # Formato CSV
$ sadf -d /var/log/sa/sa10   # Datos de un dia especifico
```

### Ejemplo de iostat
```
$ iostat 5 2
```

### `vmstat` — Estadisticas del sistema (precursor de sysstat)

> "If you're familiar with the vmstat command, which isn't part of sysstat, you know how the other 'stat' commands work."

Formato: `vmstat [options] [delay [count]]`

```
$ vmstat 5 5
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 3  0  44616 343888    244 334676    0    0     1     1    1   24  0  0 100  0  0
 0  0  44616 343768    244 334676    0    0     0     0   48  91  0  0 100  0  0
 1  0  44616 343768    244 334676    0    0     0     2   53 104  0  0 100  0  0
 0  0  44616 343768    244 334676    0    0     0     4   78 141  0  0 100  0  0
 0  0  44616 343768    244 334676    0    0     0     0   75 141  0  0 100  0  0
```

La primera ejecucion da promedios desde el ultimo reinicio.

> "Los logs de sysstat residen en `/var/log/sa` o `/var/log/sysstat`."

---

## 🔧 Laboratorio práctico (en clase)

### Paso 1: Configurar tmp.mount para usar tmpfs

```bash
$ sudo systemctl status tmp.mount
$ sudo systemctl edit tmp.mount
```
Añade:
```
[Mount]
Options=size=512M,noexec,nosuid
```
Luego:
```bash
$ sudo systemctl daemon-reload
$ sudo systemctl restart tmp.mount
$ df -h /tmp
```
Verifica que `/tmp` ahora es un `tmpfs` de 512 MB con opciones de seguridad `noexec,nosuid`.

### Paso 2: Instalar fdupes y deduplicar archivos

```bash
$ sudo apt install -y fdupes
$ mkdir -p ~/prueba_dup && cd ~/prueba_dup
$ echo "contenido duplicado" > a.txt && cp a.txt b.txt && echo "otro" > c.txt
$ fdupes -r ~/prueba_dup
$ fdupes -r -d ~/prueba_dup   # interactivo: conserva uno, elimina el resto
```
`fdupes -d` te pide confirmación para cada grupo de duplicados. Usa `-N` para no preguntar (¡con cuidado!).

### Paso 3: Configurar cuotas de disco

```bash
$ sudo apt install -y quota
$ sudo nano /etc/fstab
```
Busca la línea de la partición raíz (o la que quieras) y añade `usrquota,grpquota` en el campo de opciones:
```
UUID=... / ext4 defaults,usrquota,grpquota 0 1
```
Remonta y activa:
```bash
$ sudo mount -o remount /
$ sudo quotacheck -cugm /
$ sudo quotaon -v /
$ sudo edquota -u $USER
```
Asigna límites de 50M soft / 60M hard. Prueba superando el límite con `dd if=/dev/zero of=~/test bs=1M count=100`.

### Paso 4: Configurar unattended-upgrades

```bash
$ sudo apt install -y unattended-upgrades
$ sudo dpkg-reconfigure --priority=low unattended-upgrades
$ sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```
Asegúrate de que esté descomentado:
```
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";
};
```
Simula una ejecución:
```bash
$ sudo unattended-upgrade --dry-run -d
```

### Paso 5: Ejecutar sysstat para recopilar métricas

```bash
$ sudo systemctl enable --now sysstat
$ sudo nano /etc/default/sysstat
# Cambiar ENABLED="true"
$ sudo systemctl restart sysstat
$ sar -u 1 3   # CPU cada 1s, 3 muestras
$ sar -r       # memoria (histórico)
$ sar -S       # swap (histórico)
```

**Verificación:** `systemctl status sysstat`, `sar -u` muestra métricas de CPU.

### 🚀 Desafío individual

Crea una política de retención de cuentas para usuarios inactivos. Implementa un script que:
1. Identifique cuentas de usuario (UID ≥ 1000) sin login en los últimos 90 días (`lastlog -b 90`).
2. Bloquee esas cuentas con `passwd -l <usuario>`.
3. Archive los home directories en `/backup/usuarios_inactivos/`.
4. Envíe un reporte por correo al root.

Pista: Ordena el script con `lastlog | awk '$3 > 90 {print $1}'` para filtrar.

---

## 💪 Ejercicios (para casa / laboratorio)

1. Revisa tu `/tmp` con `df -h /tmp` - es una particion separada o parte de `/`?
2. Instala `fdupes` y busca archivos duplicados en un directorio de prueba
3. Configura una cuota de 100MB para un usuario de prueba
4. Ejecuta `sudo apt update` o `sudo yum update` para ver paquetes disponibles
5. Crea un script que liste los usuarios con contrasenas vencidas
6. Instala sysstat, habilitalo y ejecuta `sar -b` para ver I/O
7. Revisa los logs de actividad en `/var/log/sa/`

---


## Curiosidad: `/dev/null` — el agujero negro de Linux

`/dev/null` es el "basurero" de Linux. Todo lo que se envia a `/dev/null` desaparece para siempre. Es el archivo mas famoso del sistema. Los administradores usan `2>/dev/null` para silenciar errores que no quieren ver. Pero tiene un origen serio: en Unix, todo es un archivo, asi que hasta "nada" tiene que ser un archivo.

El comando `yes` (parte de coreutils) imprime "y" sin parar. Su unico proposito es responder automaticamente a prompts. `yes | sudo apt install paquete` funciona, aunque `-y` es mas elegante.

Las cuotas de disco se inventaron en los primeros Unix de los 80 para evitar que un usuario llenara el disco compartido. Siguen siendo casi identicas 40 anos despues.
