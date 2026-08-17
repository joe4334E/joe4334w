---
title: "Sesion 5: Seguridad y Carrera"
date: 2026-08-24
draft: false
featuredImage: "/images/blog-space.jpg"
tags: [resumen avanzado]
categories: ["Linux Admin"]
series: ["Practical Linux System Administration"]
weight: 50
bookAuthor: "Kenneth Hess"
bookTitle: "Practical Linux System Administration"
bookYear: "2023"
---

# Sesion 5: Seguridad y Carrera

**Tutoriales:** 13 — Seguridad | 14 — Certificacion y Formacion | 15 — Carrera Profesional
**Nivel:** Avanzado
**Duracion estimada:** 3-4 horas

---

## Objetivos de cada tutorial

### Tutorial 13 — Seguridad
- Proteger la cuenta root
- Minimizar la superficie de ataque del sistema
- Crear y aplicar politicas de contrasenas solidas
- Configurar autenticacion SSH por clave publica
- Implementar herramientas de seguridad avanzadas (Lynis, Portsentry, AIDE)
- Responder ante incidentes de seguridad

### Tutorial 14 — Certificacion y Formacion
- Comprender la importancia de la formacion continua
- Explorar opciones de certificacion y preparacion de examenes
- Aprender estrategias de estudio independiente
- Formalizar la educacion universitaria
- Usar el trabajo actual como formacion

### Tutorial 15 — Carrera Profesional
- Explorar el autoempleo y la consultoria
- Comprender los desafios de la gestion de personas
- Navegar portales de empleo online
- Trabajar de forma remota efectivamente
- Comunicarse profesionalmente por email, videoconferencia y mensajeria
- Dejar un puesto de forma elegante y profesional

---

## Resumen

El Tutorial 13 es el mas critico para la seguridad del sistema. La regla fundamental: minimizar la superficie de ataque. Lynis realiza auditorias completas clasificadas por severidad. AIDE monitorea integridad de archivos con una base de datos de checksums (flujo: init → DB → check → compare → report). `pwquality` controla politicas de contraseñas: `minclass=4` exige 4 clases de caracteres (no 4 caracteres). Los binarios con setuid se encuentran con `find / -perm /4000 -type f`. El backdoor del compilador C de Ken Thompson (1984) demuestra que la confianza en binarios compilados nunca es absoluta.

El Tutorial 14 orienta la formacion. RHCSA (Red Hat Certified System Administrator) cuesta ~$400 USD. LFCS es vendor-neutral. LPIC-1 es el nivel basico. La regla de la industria: 1 ano de formacion = 2 anos de experiencia. El libro recomienda al menos dos fuentes de estudio de diferentes autores y no seguir tendencias momentaneas.

El Tutorial 15 cierra con la carrera profesional. El elevator pitch dura 30-60 segundos. Los emails son prueba legal en tribunales. "No responder a todos" es una regla de email profesional. La carta de renuncia es un registro formal, no una critica. Documentar todo durante el preaviso facilita la transicion. El trabajo remoto tiene desafios propios: disponibilidad 24/7, falta de enfoque, jornadas excesivas.

---

## Comandos clave

| Comando | Descripcion |
|---------|-------------|
| `lynis audit system` | Auditoria completa de seguridad |
| `aide --check` | Verificar integridad de archivos |
| `find / -perm /4000 -type f` | Buscar binarios con setuid |
| `ssh-keygen -t ecdsa -b 521` | Generar par de claves ECDSA |
| `yum repolist` | Listar repositorios RHEL/CentOS |
| `cat /etc/os-release` | Version de la distribucion |
| `dpkg -l paquete` | Verificar si paquete esta instalado |
| `arecord -d 30 -f cd audio.wav` | Grabar 30 segundos de audio |

---

## Ejercicio integrador A

**Endurecimiento de un servidor:**
1. Ejecuta `lynis audit system` y documenta los hallazgos criticos
2. Aplica las sugerencias mas importantes (hardening)
3. Configura SSH: deshabilita root login, usa solo claves ECDSA, cambia puerto
4. Instala AIDE, inicializa la base de datos y ejecuta un check
5. Instala Portsentry y configuralo para monitorear puertos
6. Ejecuta Lynis nuevamente y compara el puntaje antes/despues

## Ejercicio integrador B

**Auditoria completa con reporte:**
1. Crea un script que genere un reporte de seguridad automatico que incluya:
   - Resultado de `lynis audit system` (resumen)
   - Lista de usuarios con UID 0 (solo root deberia tenerlo)
   - Puertos abiertos con `ss -tln`
   - Estado de servicios criticos con `systemctl`
   - Integridad de archivos con `aide --check`
2. Programa el script con cron para ejecutarse semanalmente
3. Guarda el reporte en `/var/log/auditoria/` con fecha
4. Documenta el proceso como si fuera para una auditoria real

---

## Para recordar

- Lynis: auditoria completa; AIDE: integridad de archivos
- `minclass=4` = 4 clases de caracteres, no 4 caracteres
- Nunca deshabilites `PasswordAuthentication` sin tener claves configuradas
- RHCSA ~$400 USD; LFCS es vendor-neutral; LPIC-1 es basico
- 1 ano de formacion = 2 anos de experiencia (regla de la industria)
- Emails son prueba legal; "no responder a todos" siempre
- Documentar todo durante preaviso es obligatorio profesionalmente
