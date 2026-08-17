---
title: "linux-10 Scripting y Automatizacion"
date: 2026-08-10
draft: false
featuredImage: "/images/blog-space.jpg"
tags: [bash scripts cron chrony automatizacion]
categories: ["Linux Admin"]
series: ["Practical Linux System Administration"]
weight: 10
bookAuthor: "Kenneth Hess"
bookTitle: "Practical Linux System Administration"
bookYear: "2023"
---

# Tutorial 10: Scripting y Automatización

**Capítulo 10 del libro** — "Practical Linux System Administration" (pags. 137-144)
**Nivel:** Intermedio

---

## ⚡ Para empezar: "No automatices todo... pero casi"

El libro plantea 5 preguntas sobre automatización. La más importante: **¿Qué tareas no deberían automatizarse nunca?** — Aquellas que requieren almacenar contraseñas en texto plano.

Pero hay un lado psicológico: "sysadmins who automate experience less burnout than those who perform every task manually." Automatizar no solo es eficiente, es saludable.

Y hablando de orígenes: `cron` viene de Chronos, el dios griego del tiempo. Fue creado por Ken Thompson (sí, el mismo de Unix) en 1975. El archivo `crontab` significa "cron table".

---

## Objetivos

- Comprender qué tareas automatizar y cuáles no
- Escribir scripts básicos de shell a partir de un esquema
- Programar tareas con `cron`
- Sincronizar el tiempo entre sistemas con `chrony`

---

## 1. ¿Qué automatizar?

> "Sysadmins automate what they can. But you can't automate everything." — Capítulo 10

La automatización permite centrarse en tareas de alto nivel mientras los sistemas manejan las tareas mundanas sin errores.

### Preguntas comunes sobre automatización (del libro):

1. **¿Qué tareas automatizar?** — Cualquier cosa que pueda escribirse como script de forma fiable
2. **¿Hay tareas que no pueden automatizarse?** — Sí, las que requieren decisiones complejas de múltiples pasos
3. **¿Hay tareas que nunca deberían automatizarse?** — Sí, las que requieren almacenar contraseñas sin cifrar en texto plano
4. **¿Qué tarea automatizar primero?** — Copias de seguridad
5. **¿Comprar solución comercial?** — Empiece con scripts propios antes de gastar dinero

---

## 2. Crear Scripts

### Esquema del script `backup_server1.sh`:

1. Crear un archivo tar del directorio `/etc`
2. Comprimir el archivo tar
3. Transferir el archivo al servidor `archive1`

### Script completo:

```bash
#!/bin/bash
# Create a tar file of /etc.
sudo tar cvf server1_etc.tar /etc
# Compress the tar file
gzip -9 server1_etc.tar
# Transfer the file to archive1 into the /server1/backups directory
scp server1_etc.tar.gz archive1:/server1/backups
```

### 🖐️ Mini-ejercicio

Crea un script simple:

```bash
#!/bin/bash
echo "Hola, mundo!"
echo "Hoy es: $(date)"
```

Guárdalo como `hola.sh`, hazlo ejecutable (`chmod +x hola.sh`) y ejecútalo: `./hola.sh`.

**Modifícalo para que también muestre el usuario actual (`whoami`) y el directorio actual (`pwd`).**

---

## 3. Programar Tareas con `cron`

> "The cron utility schedules commands to run at a specific time."

Formato de cron:

```
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6)
# │ │ │ │ │
# * * * * * command to execute
```

Ejemplos del libro:

```bash
# Cada 5 minutos
0,5,10,15,20,25,30,35,40,45,50,55 * * * * /path/to/script.sh

# Lunes, miércoles y viernes a las 14:00
0 14 * * 1,3,5 /path/to/script.sh

# 6 AM del día 15 de cada mes
0 6 15 * * /path/to/script.sh

# Crontab para el usuario bur (backup diario a las 2 AM)
0 2 * * * /home/bur/backup_server1.sh
```

> "Do not set a schedule in cron for `* * * * * /path/to/script.sh` unless you want your script to run every minute of every day."

### 🖐️ Mini-ejercicio

Abre tu crontab:

```bash
$ crontab -e
```

Añade esta línea para ejecutar un comando cada hora:

```
0 * * * * echo "Hora: $(date)" >> $HOME/cron-test.txt
```

Espera a que cambie la hora o fuerza la ejecución. Luego verifica: `cat ~/cron-test.txt`.

**Elimina la línea del crontab cuando termines la prueba.**

---

## 4. Sincronizar Tiempo con `chrony`

> "cron schedules jobs to be executed at a specific day and time, whereas chrony synchronizes the system clock with an external time server."

### Instalación y configuración básica:

```bash
$ sudo yum -y install chrony      # RHEL/CentOS
$ sudo apt install chrony          # Debian/Ubuntu
$ sudo systemctl enable chronyd
$ sudo systemctl start chronyd
```

### Verificar actividad:

```bash
$ chronyc activity
200 OK
4 sources online
0 sources offline
```

### Configurar como servidor de tiempo local:

Editar `/etc/chrony.conf` y descomentar:

```
# Allow NTP client access from local network.
allow 192.168.0.0/16
# Serve time even if not synchronized to a time source.
local stratum 10
```

Reiniciar:

```bash
$ sudo systemctl restart chronyd
```

### Configurar cliente:

En `/etc/chrony.conf` del cliente:

```
server 192.168.1.80 prefer iburst
```

Verificar fuentes:

```bash
$ chronyc sources
```

---

## 🔧 Laboratorio práctico (en clase)

### Paso 1: Crear un script de backup básico

```bash
$ mkdir -p ~/scripts
$ nano ~/scripts/backup_home.sh
```

Contenido del script:
```bash
#!/bin/bash
BACKUP_DIR="/backup/home"
FECHA=$(date +%Y%m%d_%H%M%S)
USUARIO="$USER"
ORIGEN="/home/$USUARIO"
DESTINO="$BACKUP_DIR/${USUARIO}_${FECHA}.tar.gz"

mkdir -p "$BACKUP_DIR"
tar -czf "$DESTINO" "$ORIGEN" 2>/dev/null

if [ $? -eq 0 ]; then
    echo "[OK] Backup creado: $DESTINO"
    logger -t backup_home "Backup de $USUARIO completado: $(du -sh $DESTINO | cut -f1)"
else
    echo "[ERROR] Falló el backup de $USUARIO"
    logger -t backup_home "ERROR: Falló backup de $USUARIO"
    exit 1
fi
```

### Paso 2: Asignar permisos y probar el script

```bash
$ chmod 755 ~/scripts/backup_home.sh
$ sudo mkdir /backup
$ ~/scripts/backup_home.sh
$ ls -lh /backup/home/
$ tar -tzf /backup/home/*.tar.gz | head -10
```
`chmod 755` da permisos de lectura/ejecución al propietario, lectura/ejecución a grupo y otros. Prueba la ejecución y verifica el contenido del tarball.

### Paso 3: Agregar el script al crontab

```bash
$ crontab -e
```
Añade la siguiente línea para ejecutar el backup cada día a las 2:30 AM:
```
30 2 * * * /home/$USER/scripts/backup_home.sh
```
Guarda y verifica:
```bash
$ crontab -l
```
Puedes forzar una ejecución manual para verificar que crond lo procesa. Revisa los logs del sistema: `grep backup_home /var/log/syslog`.

### Paso 4: Instalar y configurar chrony como cliente

```bash
$ sudo apt install -y chrony
$ sudo systemctl status chrony
$ sudo nano /etc/chrony/chrony.conf
```
Asegúrate de tener un servidor NTP:
```
pool 0.ubuntu.pool.ntp.org iburst
```
Reinicia y verifica:
```bash
$ sudo systemctl restart chrony
$ chronyc tracking
$ chronyc sources -v
```
`chronyc tracking` muestra la deriva del reloj, el último offset y la frecuencia de ajuste. `chronyc sources` lista los servidores NTP consultados.

### Paso 5: Automatizar la sincronización horaria

```bash
$ sudo timedatectl set-ntp true
$ timedatectl status
```
`timedatectl` muestra si NTP está activo, la hora del sistema, la hora RTC y la zona horaria. Asegúrate de que `System clock synchronized: yes`.

**Verificación:** `chronyc tracking` debe mostrar un offset pequeño (< 5ms típicamente).

### 🚀 Desafío individual

Escribe un script que:
1. Verifique el uso de disco de la partición raíz (`/`).
2. Si el uso supera el 90%, envíe un correo de alerta a `root@localhost`.
3. Se registre en el log del sistema con `logger`.
4. Añade el script al crontab para que se ejecute cada hora.

Pista: Usa `df -h / | tail -1 | awk '{print $5}' | tr -d '%'` para obtener el porcentaje. Para enviar correo: `echo "Asunto" | mail -s "Alerta" root@localhost` (instala `mailutils` si es necesario).

---

## 💪 Ejercicios (para casa / laboratorio)

1. Escriba un script que respalde `/etc` y `/home` usando `tar` y `gzip`
2. Programe el script para que se ejecute cada día a las 3 AM usando `cron`
3. Instale `chrony` y verifique que su sistema está sincronizado
4. Cree un usuario `bur` (backup and restore) y configure SSH sin contraseña
5. Configure un servidor chrony local y un cliente que lo use

---


## Curiosidad: El shebang `#!`

La secuencia `#!/bin/bash` al inicio de los scripts se llama "shebang" (por "shell" y "bang" que es como se lee `!` en jerga Unix). Fue introducido en Unix Version 7 (1979) por Dennis Ritchie, uno de los creadores de C y Unix. Sin el shebang, el sistema no sabe qué intérprete usar.

Dato curioso: en sistemas modernos, `#!/bin/sh` no es lo mismo que `#!/bin/bash`. En muchas distribuciones, `/bin/sh` es un enlace a `dash` (Debian Almquist Shell), que es más rápido pero menos feature-rich que bash. Usar `#!/bin/sh` por "compatibilidad" puede romper scripts que usen características específicas de bash como `[[ ]]` o `source`.
