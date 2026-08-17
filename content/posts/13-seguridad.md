---
title: "linux-13 Seguridad"
date: 2026-08-13
draft: false
featuredImage: "/images/blog-space.jpg"
tags: [lynis aide contrasenas ssh hardening]
categories: ["Linux Admin"]
series: ["Practical Linux System Administration"]
weight: 13
bookAuthor: "Kenneth Hess"
bookTitle: "Practical Linux System Administration"
bookYear: "2023"
---

# Tutorial 13: Seguridad del Sistema

**Capítulo 13 del libro** — "Practical Linux System Administration" (pags. 173-194)
**Nivel:** Avanzado

---

## ⚡ Para empezar: "El sistema más seguro está apagado y bajo llave... pero no sirve para nada"

Esa frase del libro captura la paradoja de la seguridad: cada medida que añades reduce la usabilidad. Tu trabajo como administrador es encontrar el equilibrio.

El capítulo 13 es el más largo del libro (21 páginas) por una razón: "As a system administrator, security is your most important and time-consuming task." Y aunque suene a frase hecha, viene respaldada por ejemplos concretos: proteger root, minimizar superficie de ataque, políticas de contraseñas, SSH por clave, Lynis, Portsentry, AIDE.

---

## Objetivos

- Proteger la cuenta root
- Minimizar la superficie de ataque del sistema
- Crear y aplicar políticas de contraseñas sólidas
- Configurar autenticación SSH por clave pública
- Implementar herramientas de seguridad avanzadas (Lynis, Portsentry, AIDE)
- Responder ante incidentes de seguridad

---

## 1. Proteger la Cuenta Root

> "If someone compromises this account, they can lock you out, destroy the system, steal data, or maintain control of it and use it to pivot to and compromise other systems."

- Nunca escribir ni compartir la contraseña root fuera del grupo de administradores
- Usar un gestor de contraseñas seguro
- Reemplazar contraseñas con archivos de clave sin contraseña

---

## 2. Minimizar la Superficie de Ataque

### Verificar y eliminar GUI:

```bash
$ systemctl get-default
multi-user.target

$ sudo systemctl set-default multi-user.target

$ rpm -qa | grep xorg | grep server
xorg-x11-server-utils-7.7-27.el8.x86_64

$ sudo yum remove xorg-x11-server-Xorg xorg-x11-server-common xorg-x11-server-utils
```

### Crear sistemas de propósito único:

- Instalación mínima (solo SSH)
- Añadir servicios solo cuando sean necesarios
- Auditoría de puertos con `netstat`:

```bash
$ netstat -an | grep LISTEN | grep tcp
tcp   0   0 0.0.0.0:22      0.0.0.0:*      LISTEN
```

### Limitar Apache a localhost:

```
Listen 127.0.0.1:80
#Listen 80
```

### Autoremove:

```bash
$ sudo yum autoremove
$ sudo apt autoremove
```

### 🖐️ Mini-ejercicio

```bash
$ systemctl list-units --type=service --state=running | head -20
```

**¿Cuántos servicios están ejecutándose?** Marca los que no reconozcas. Luego revisa los puertos abiertos:

```bash
$ netstat -an | grep LISTEN
```

**¿Hay algún puerto abierto que no debería estar?**

---

## 3. Política de Contraseñas

`/etc/security/pwquality.conf` — Configuración de calidad de contraseñas:

```ini
# Minimum acceptable size for the new password
# Cannot be set to lower value than 6.
minlen = 8

# The minimum number of required classes of characters
# (digits, uppercase, lowercase, others)
minclass = 3

# Whether the check is enforced by the PAM module
enforcing = 1
```

---

## 4. Autenticación por Clave SSH

> "Using key files, users may connect from one system to another without the need to interactively enter a password."

### Generar par de claves (ECDSA 521 bits):

```bash
$ ssh-keygen -t ecdsa -b 521
```

> "The default encryption algorithm is RSA, but both RSA and DSA algorithms are old, and you shouldn't use them."

> "The three available key sizes are 256, 384, and 521. No, the 521 you see in the command isn't an error."

### Copiar clave al servidor remoto:

```bash
$ ssh-copy-id server2
```

### Verificar:

```bash
$ ssh server2
Last login: Fri Aug 12 14:10:46 2022 from 192.168.1.80
```

### Configuración bidireccional:

Repetir el proceso en `server2` para conectarse a `server1`.

### Endurecer SSHD:

```bash
$ sudo grep -i pubkey /etc/ssh/sshd_config
PubkeyAuthentication yes

$ sudo grep -i password /etc/ssh/sshd_config
PasswordAuthentication yes
```

> "If you change `PasswordAuthentication yes` to `PasswordAuthentication no`, you will exclude users who do not already have their key-based authentication configured."

### 🖐️ Mini-ejercicio

Genera un par de claves ECDSA:

```bash
$ ssh-keygen -t ecdsa -b 521 -f ~/.ssh/id_ecdsa_lab
```

**¿Qué archivos se crearon?** (`ls -la ~/.ssh/`). La clave pública (`.pub`) puede compartirse. La privada (sin extensión) **nunca**.

---

## 5. Herramientas Avanzadas de Seguridad

### Lynis — Auditoría de seguridad

```bash
$ sudo lynis audit system
$ sudo grep Suggestion /var/log/lynis.log > lynis_fixes.txt
```

Recomendaciones típicas: hardening de SSH, banners legales, logs externos.

### Portsentry — Detección de escaneos

```bash
$ sudo portsentry
```

Puertos monitoreados por defecto:

```
TCP_PORTS="1,11,15,79,111,119,143,540,635,1080,1524,2000,5742,6667,12345..."
```

Acción por defecto: bloquear ruta al host atacante.

### AIDE — Integridad de archivos

```bash
$ sudo aide --init                    # RHEL/CentOS
$ sudo aideinit                       # Ubuntu/Debian
$ sudo cp /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz
$ sudo aide --check
```

Salida esperada: `AIDE found NO differences between database and filesystem. Looks okay!!`

Si hay cambios:

```
AIDE found differences between database and filesystem!!
Summary:
  Added entries:      1
  Removed entries:    1
  Changed entries:    5
```

Actualizar base de datos:

```bash
$ sudo aide.wrapper --update          # Ubuntu
```

---

## 6. Responder a Incidentes

1. **Confirmar la brecha** — Retirar sistemas comprometidos de la red
2. **Identificar al actor** — Involucrar a las autoridades si es necesario
3. **Acciones correctivas** — Escanear, limpiar, cifrar, reimaginar

> "A well-written security policy is the best defense against insider threats."

---

## 🔧 Laboratorio práctico (en clase)

### Paso 1: Identificar servicios y puertos abiertos

Lista servicios en ejecución:

```bash
$ systemctl list-units --type=service --state=running
```

Enumera puertos en escucha:

```bash
$ sudo ss -tlnp
$ sudo ss -ulnp
```

Identifica qué está escuchando en cada puerto:

```bash
$ sudo lsof -i -P -n | grep LISTEN
```

### Paso 2: Fortaleza de contraseñas con pwquality

Instala pwquality y configura la política:

```bash
$ sudo apt install -y libpam-pwquality
$ sudo nano /etc/pam.d/common-password
```

Busca la línea que contiene `pam_pwquality.so` y ajústala:

```
password requisite pam_pwquality.so retry=3 minlen=12 minclass=4
```

Prueba la política con un usuario normal:

```bash
$ passwd   # Intenta poner "1234" y luego "MiC0ntr@s3ñ4F1rm3!"
```

### Paso 3: Generar claves SSH ECDSA

Genera un par de claves ECDSA (más seguras que RSA para misma longitud):

```bash
$ ssh-keygen -t ecdsa -b 521 -f ~/.ssh/id_ecdsa -C "mi-clave-ecdsa-$(hostname)"
```

Revisa las claves generadas:

```bash
$ ls -la ~/.ssh/
$ cat ~/.ssh/id_ecdsa.pub
```

Copia la clave pública al servidor remoto (o a localhost para probar):

```bash
$ ssh-copy-id -i ~/.ssh/id_ecdsa.pub localhost
$ ssh localhost -i ~/.ssh/id_ecdsa
```

### Paso 4: Auditoría con Lynis

Instala Lynis:

```bash
$ sudo apt install -y lynis
```

Ejecuta una auditoría:

```bash
$ sudo lynis audit system
```

Analiza la salida — presta atención a estas secciones:

- `[WARNING]` — advertencias que necesitan acción
- `[SUGGESTION]` — sugerencias de mejora
- `[TEST]` — cada prueba individual con su resultado

Genera un reporte:

```bash
$ sudo lynis audit system --report-file /tmp/lynis-report.txt
$ grep -E "(Warning|Suggestion)" /tmp/lynis-report.txt | head -20
```

### Paso 5: Configurar AIDE

Instala AIDE:

```bash
$ sudo apt install -y aide
```

Inicializa la base de datos (esto puede tomar un minuto):

```bash
$ sudo aideinit
$ sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db
```

Ejecuta una verificación:

```bash
$ sudo aide --check
```

Agrega una tarea programada para verificación diaria:

```bash
$ sudo nano /etc/cron.daily/aide-check
```

```bash
#!/bin/bash
/usr/bin/aide --check | mail -s "AIDE Daily Check" root
```

```bash
$ sudo chmod +x /etc/cron.daily/aide-check
```

### Paso 6: Explorar controles STIG

Instala la herramienta de referencia STIG si está disponible:

```bash
$ sudo apt install -y openscap-scanner scap-security-guide
```

Ejecuta un escaneo de línea base STIG:

```bash
$ sudo oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_stig \
  --results /tmp/stig-results.xml \
  /usr/share/xml/scap/ssg/content/ssg-ubuntu2204-ds.xml 2>/dev/null || true
```

**Verificación:** Debes poder identificar servicios y puertos abiertos, generar claves ECDSA funcionales, obtener un reporte Lynis con sugerencias concretas, tener AIDE configurado con una base de datos inicializada, y haber visto los resultados de un escaneo STIG.

### 🚀 Desafío individual

**Escenario:** Recibes un servidor Ubuntu recién instalado con configuración por defecto. Tu tarea es:

1. Endurecer SSH (`/etc/ssh/sshd_config`): deshabilita root login, cambia puerto, deshabilita autenticación por contraseña (solo claves).
2. Ejecuta `sudo lynis audit system`.
3. Lee las sugerencias de Lynis e implementa las **3 más importantes** (prioriza las marcadas como `[HIGH]`).
4. Vuelve a ejecutar Lynis y compara la puntuación inicial vs final.

Anota los cambios que hiciste y cómo cambiaron el score de Lynis.

---

## 💪 Ejercicios (para casa / laboratorio)

1. Verifique que su sistema no tenga GUI instalada
2. Revise los puertos de escucha con `netstat`
3. Cree un par de claves ECDSA de 521 bits y configure SSH sin contraseña entre dos sistemas
4. Instale y ejecute Lynis, revise las sugerencias
5. Inicialice AIDE y ejecute una verificación
6. Cree el script `daily_report.sh` del capítulo 12 y póngalo en cron

---


## Curiosidad: El backdoor del compilador C

Ken Thompson (co-creador de Unix) demostró en 1984 el backdoor más ingenioso de la historia. Modificó el compilador de C para que:
1. Cuando compilara el programa `login`, insertara una puerta trasera
2. Cuando compilara el compilador, insertara esa misma vulnerabilidad en el nuevo compilador

El resultado: podías eliminar el código malicioso del fuente, pero al recompilar el compilador, este seguía generando el backdoor automáticamente. Solo se podía detectar analizando el binario del compilador. Su paper se titula "Reflections on Trusting Trust".

El comando `sudo` significa "substitute user do" (originalmente "superuser do"). Fue creado por Robert Coggeshall y Cliff Spencer en la Universidad Estatal de Nueva York en 1980 para permitir que usuarios específicos ejecutaran comandos como root sin compartir la contraseña.
