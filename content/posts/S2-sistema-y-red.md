---
title: "Sesion 2: Sistema y Red"
date: 2026-08-19
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

# Sesion 2: Sistema y Red

**Tutoriales:** 04 — Gestion de Usuarios | 05 — Conectar a la Red | 06 — Instalar y Borrar Software
**Nivel:** Principiante
**Duracion estimada:** 3-4 horas

---

## Objetivos de cada tutorial

### Tutorial 04 — Gestion de Usuarios
- Crear cuentas de usuario con `useradd` y `adduser`
- Modificar cuentas con `usermod`
- Eliminar cuentas con `userdel`
- Forzar cambios de contrasena con `chage`
- Gestionar grupos

### Tutorial 05 — Conectar a la Red
- Entender IP estatica vs dinamica (DHCP)
- Conocer implicaciones de seguridad de red
- Asegurar el demonio SSH
- Restringir acceso por IP, deshabilitar root login, usar claves SSH

### Tutorial 06 — Instalar y Borrar Software
- Actualizar el sistema en RHEL/CentOS y Debian/Ubuntu
- Instalar software desde repositorios
- Instalar paquetes individuales manualmente
- Instalar software desde codigo fuente
- Desinstalar software por cada metodo

---

## Resumen

El Tutorial 04 profundiza en la administracion de usuarios. Aprendes que las cuentas de sistema (UID 1-999) no son humanas sino demonios como `www-data` o `nobody` (UID 65534). El campo de comentarios en `/etc/passwd` se llama GECOS por un sistema operativo de los anos 60. Un error comun es usar `usermod -G` sin `-a`, que borra los grupos suplementarios existentes. Las contraseñas se almacenan hasheadas en `/etc/shadow`, no en `/etc/passwd`.

El Tutorial 05 conecta tu sistema a la red. Un punto critico: un sistema Linux recien conectado es escaneado por bots en minutos. Las IP estaticas son ideales para servidores; DHCP para estaciones de trabajo. SSH debe endurecerse: deshabilitar `PermitRootLogin`, cambiar el puerto 22, usar solo claves publicas. `ifconfig` fue reemplazado por `ip`.

El Tutorial 06 cubre la gestion de software. La diferencia critica: `apt update` solo descarga listas de paquetes; `apt upgrade` los instala. `dpkg -i` no resuelve dependencias (hay que usar `apt install -f` despues). `apt purge` elimina paquetes y configuraciones; `apt remove` solo elimina el paquete. Los repositorios de terceros no son tan seguros como los oficiales.

---

## Comandos clave

| Comando | Descripcion |
|---------|-------------|
| `useradd -c "Nombre" usuario` | Crea un usuario nuevo |
| `usermod -a -G grupo usuario` | Agrega usuario a grupo sin borrar otros |
| `userdel -r usuario` | Elimina usuario y su home |
| `chage -d 0 usuario` | Fuerza cambio de contrasena |
| `id` | Muestra UID, GID y grupos del usuario |
| `ip addr show` | Muestra interfaces y direcciones IP |
| `ssh -p 2222 user@host` | Conexion SSH en puerto no estandar |
| `apt update` | Actualiza listas de paquetes |
| `apt upgrade` | Instala actualizaciones disponibles |
| `apt purge paquete` | Elimina paquete y configuraciones |

---

## Ejercicio integrador A

**Configuracion de usuario con acceso SSH:**
1. Crea un usuario `operador` con grupo suplementario `sudo`
2. Genera un par de claves SSH con `ssh-keygen -t ecdsa -b 521`
3. Copia la clave publica al usuario `operador` en el servidor
4. Deshabilita el login root por SSH en `/etc/ssh/sshd_config`
5. Verifica que puedes conectarte como `operador` y ejecutar `sudo apt update`
6. Documenta cada paso y el resultado

## Ejercicio integrador B

**Setup de un servidor web basico:**
1. Actualiza el sistema con `apt update && apt upgrade`
2. Instala Apache con `apt install apache2`
3. Verifica que el servicio esta activo con `systemctl status apache2`
4. Crea un usuario `webmaster` con acceso al directorio `/var/www/html`
5. Configura un archivo `index.html` personalizado
6. Prueba desde otro equipo en la red con `curl http://IP_DEL_SERVIDOR`

---

## Para recordar

- `useradd` crea la cuenta sin contraseña; `passwd` la asigna
- Siempre usa `-a` con `usermod -G` para no borrar grupos existentes
- `apt update` != `apt upgrade`; se ejecutan en ese orden
- SSH: deshabilita root login, usa claves, cambia el puerto
- Un sistema conectado a la red es escaneado en minutos
