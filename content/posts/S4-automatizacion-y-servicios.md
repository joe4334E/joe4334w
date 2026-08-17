---
title: "Sesion 4: Automatizacion y Servicios"
date: 2026-08-21
draft: false
featuredImage: "/images/blog-space.jpg"
tags: [resumen intermedio]
categories: ["Linux Admin"]
series: ["Practical Linux System Administration"]
weight: 50
bookAuthor: "Kenneth Hess"
bookTitle: "Practical Linux System Administration"
bookYear: "2023"
---

# Sesion 4: Automatizacion y Servicios

**Tutoriales:** 10 — Scripting y Automatizacion | 11 — Samba | 12 — Troubleshooting
**Nivel:** Intermedio
**Duracion estimada:** 4-5 horas

---

## Objetivos de cada tutorial

### Tutorial 10 — Scripting y Automatizacion
- Comprender que tareas automatizar y cuales no
- Escribir scripts basicos de shell a partir de un esquema
- Programar tareas con `cron`
- Sincronizar el tiempo entre sistemas con `chrony`

### Tutorial 11 — Samba
- Comprender la interoperabilidad Linux-Windows con Samba
- Planificar un entorno Samba segun necesidades del negocio
- Instalar Samba y sus dependencias
- Crear usuarios y grupos Samba
- Configurar directorios compartidos
- Montar recursos compartidos de Windows desde Linux

### Tutorial 12 — Troubleshooting
- Resolver un kernel panic reconstruyendo initramfs
- Extraer informacion de logs del sistema con `dmesg`
- Diagnosticar problemas de software con `apachectl configtest`
- Recopilar informacion de hardware con `hwinfo`, `lshw`, `lspci`, `lsblk`, `lscpu`
- Crear informes de seguridad automatizados

---

## Resumen

El Tutorial 10 inicia con el shebang `#!/bin/bash`, introducido en Unix Version 7 (1979) por Dennis Ritchie. Un dato importante: `#!/bin/sh` no es lo mismo que `#!/bin/bash` en distribuciones modernas (sh puede ser `dash`). Los 5 campos de crontab son: minuto, hora, dia del mes, mes, dia de la semana. `chrony` y `cron` son herramientas completamente distintas: una sincroniza el reloj, la otra programa tareas.

El Tutorial 11 cubre Samba, creado por Andrew Tridgell mediante ingeniería inversa del protocolo SMB. El puerto SMB directo es 445 TCP. La seccion `[global]` es obligatoria en `smb.conf`. Samba es preferible cuando hay clientes Windows; NFS es mejor para entornos puros Linux. `testparm` verifica la sintaxis de la configuracion. `hosts allow` restringe el acceso por red.

El Tutorial 12 es critico para resolver problemas. Los 5 problemas mas comunes: DNS/red, llenado de disco, servicios que no inician, permisos/autenticacion, y fallos de hardware. Nunca deshabilites el firewall durante troubleshooting (agrega reglas especificas). `dmesg` y `journalctl -k` muestran logs del kernel, pero `journalctl` integra mejor con el resto del sistema. Reconstruir initramfs con `dracut -f` puede resolver un kernel panic sin reinstalar.

---

## Comandos clave

| Comando | Descripcion |
|---------|-------------|
| `#!/bin/bash` | Shebang para scripts Bash |
| `crontab -e` | Editar el crontab del usuario |
| `*/5 * * * *` | Ejecutar cada 5 minutos |
| `chronyc tracking` | Verificar sincronizacion NTP |
| `testparm` | Verificar sintaxis de smb.conf |
| `smbclient -L //servidor -U user` | Listar recursos compartidos |
| `dmesg -w` | Mensajes del kernel en tiempo real |
| `journalctl -u servicio` | Logs de un servicio especifico |
| `systemctl --failed` | Servicios que fallaron |
| `dracut -f` | Reconstruir initramfs |
| `apachectl configtest` | Verificar configuracion de Apache |

---

## Ejercicio integrador A

**Script de backup con cron:**
1. Escribe un script `backup.sh` que copie `/etc` a `/backup/` con fecha en el nombre
2. Usa `logger` para registrar cada ejecucion en syslog
3. Programa el script con `cron` para que se ejecute cada dia a las 2:00 AM
4. Configura `chrony` como servidor NTP para tu red local
5. Verifica con `chronyc tracking` que la sincronizacion funciona
6. Documenta cada paso y resultado

## Ejercicio integrador B

**Mini-servidor de archivos con Samba:**
1. Instala Samba y verifica con `testparm`
2. Crea un grupo `shared` y dos usuarios del grupo
3. Configura un directorio compartido `[documents]` con acceso solo al grupo
4. Monta el recurso compartido desde otro equipo con `mount -t cifs`
5. Prueba lectura y escritura desde ambos sistemas
6. Configura `hosts allow` para restringir a tu subred

---

## Para recordar

- `#!/bin/sh` != `#!/bin/bash` en sistemas modernos
- cron: 5 campos (min, hora, dia_mes, mes, dia_semana)
- `chrony` (reloj) != `cron` (tareas programadas)
- Nunca deshabilites el firewall durante troubleshooting
- Los 5 problemas mas comunes: DNS, disco, servicios, permisos, hardware
- Samba: ingeniería inversa de SMB por Andrew Tridgell
