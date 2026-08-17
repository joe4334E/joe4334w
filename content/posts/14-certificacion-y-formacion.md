---
title: "linux-14 Certificacion y Formacion"
date: 2026-08-14
draft: false
featuredImage: "/images/blog-space.jpg"
tags: [rhcsa lpic lfcs certificacion estudio]
categories: ["Linux Admin"]
series: ["Practical Linux System Administration"]
weight: 14
bookAuthor: "Kenneth Hess"
bookTitle: "Practical Linux System Administration"
bookYear: "2023"
---

# Tutorial 14: Certificación y Formación Continua

**Capítulo 14 del libro** — "Practical Linux System Administration" (pags. 195-200)
**Nivel:** Avanzado (orientación profesional)

---

## ⚡ Para empezar: La certificación no lo es todo (pero ayuda)

A finales de los 90 y principios de los 2000, hubo una fiebre de certificaciones. Todos querían ser CCNA, MCSE, RHCE. El libro advierte: "Don't take the exam lightly, and don't assume that you already know everything."

Pero también hay un lado humano: "It doesn't matter how technical you are. You must continue learning throughout your career." La formación continua no es opcional — es el combustible de una carrera tecnológica.

---

## Objetivos

- Comprender la importancia de la formación continua
- Explorar opciones de certificación y preparación de exámenes
- Aprender estrategias de estudio independiente
- Formalizar la educación universitaria
- Usar el trabajo actual como formación

---

## 1. Formación Interna

La formación interna es la más accesible y económica, pero tiene desventajas:

> "When you train at your local facility, you risk having users, colleagues, or your manager interrupt your training to have you fix something that will 'only take a minute.'"

---

## 2. Certificaciones

> "In the late 1990s and early 2000s, there was an industry-wide effort to get technical people certified."

Las certificaciones tienen valor, pero el autor recomienda:

- Buscar certificaciones probadas por el tiempo y el trabajo
- No saltar en la tendencia del momento
- La CISSP es un ejemplo de certificación de alto nivel ampliamente respetada

### Preparación para el examen:

> "Don't take the exam lightly, and don't assume that you already know everything."

- Usar al menos dos fuentes de estudio de diferentes autores
- Tomar exámenes de práctica
- Los exámenes modernos son basados en escenarios, no solo en hechos

### El día del examen:

> "On the day of your exam, arrive at the testing center a little early so you can sign in, calm down, use the restroom, and get into a test-taking mindset."

- El proctor le escoltará a la sala de examen
- Deberá vaciar bolsillos, entregar el móvil
- La mayoría de los exámenes se califican al finalizar
- Si falla: "resume your studies as soon as possible while the exam questions and subject matter are fresh in your mind"

### 🖐️ Mini-ejercicio

Busca en internet **tres certificaciones** que te interesen (ej: RHCSA, LPIC-1, LFCS, CompTIA Linux+).

**¿Cuál es el costo de cada una? ¿Cuánto tiempo de estudio recomiendan?** Anótalo. En la siguiente clase comparte cuál elegirías y por qué.

---

## 3. Autoeducación

> "I have spent thousands of dollars and many, many hours educating myself."

El autor instaló múltiples sistemas en su garaje: Hyper-V, vSphere, docenas de MV con múltiples distribuciones Linux, Microsoft, FreeBSD, FreeDOS.

> "Independent study can make you stand out in a crowd."

Ejemplo del libro: un sistema Solaris antiguo que nadie sabía reparar, el autor lo resolvió gracias a su estudio independiente.

Hoy en día no hace falta gran inversión en hardware:

> "You can sign up for free or inexpensive virtual infrastructure and explore cloud solutions, N-tier database setups, peer-to-peer networking, software development."

---

## 4. Educación Formal

> "During my more than 25 years in the IT industry, I've found that coworkers who don't have degrees are about as common as those who do."

- Programas de grado completos ahora cubren desarrollo de juegos, ciencia de datos, redes, seguridad y cloud
- Muchas universidades ofrecen estudio remoto y a ritmo propio
- Investigar antes de gastar: no todos los programas son bien recibidos en la industria

---

## 5. El Trabajo como Educación

La regla aceptada en la industria: 1 año de educación = 2 años de experiencia.

Para ganar experiencia sin tenerla:
- Voluntariado (iglesias, teatros, escuelas)
- Pasantías y aprendizajes
- Documentar trabajo e investigación personal

> "Once employed, try to learn everything you can about the business. Learn its business cycles, profitable areas, weaknesses, vulnerabilities."

---

## 6. Comparativa de certificaciones Linux

| Certificacion | Nivel | Costo aprox. | Temario principal |
|--------------|-------|-------------|-------------------|
| **RHCSA** | Inicial | $400 | RHEL: administracion basica, SElinux, LVM |
| **RHCE** | Avanzado | $400 | RHEL: automatizacion, Ansible, redes |
| **LPIC-1** | Inicial | $200 | Distribucion neutral: CLI, permisos, paquetes |
| **LPIC-2** | Intermedio | $200 | Redes, kernel, almacenamiento, DNS |
| **LFCS** | Inicial | $300 | Administracion practica en linea de comandos |
| **CompTIA Linux+** | Inicial | $350 | Conceptos generales, mantenimiento, seguridad |

> "Choose certifications that have been proven by time and by the job."

### 🖐️ Mini-ejercicio

Compara RHCSA y LPIC-1. Investiga que temarios cubren y cual se alinea mas con tu trabajo actual o deseado. ¿Cual ofrece mejor retorno de inversion?

---

## 7. Recursos de estudio recomendados

El libro enumera multiples fuentes para preparacion:

- **Documentacion oficial** de Red Hat y LPIC
- **Libros de practicas** con laboratorios guiados
- **Videos formativos** (O'Reilly Safari, LinkedIn Learning, Pluralsight)
- **Laboratorios virtuales** (Cloud Academy, A Cloud Guru, Katacoda)
- **Comunidades** (Reddit r/linuxadmin, Stack Exchange, foros de certificacion)

> "I have spent thousands of dollars and many, many hours educating myself."

La clave es combinar varias fuentes y practicar en laboratorios reales. Ningun libro o video reemplaza haber configurado un servidor web desde cero o haber recuperado un sistema con el kernel panic.

---

## 🔧 Laboratorio práctico (en clase)

### Paso 1: Investigar RHCSA (Red Hat)

Abre el navegador y busca los objetivos del examen EX200:

```bash
$ firefox https://www.redhat.com/en/services/training/ex200-red-hat-certified-system-administrator-rhcsa-exam &
```

Alternativa desde terminal — descarga la guía de objetivos:

```bash
$ curl -s https://www.redhat.com/rhba/learning-objectives/ex200.pdf -o /tmp/ex200.pdf 2>/dev/null; echo "Descargado"
```

Lista los dominios principales del examen:

- Comprender y usar las herramientas esenciales
- Operar sistemas en ejecución
- Configurar almacenamiento local
- Crear y configurar sistemas de archivos
- Desplegar, configurar y mantener sistemas
- Gestionar usuarios y grupos
- Gestionar seguridad

### Paso 2: Investigar LPIC-1

Busca los objetivos del LPIC-1 (exámenes 101 y 102):

```bash
$ curl -sL https://www.lpi.org/our-certifications/lpic-1-overview | grep -i "exam" | head -10
```

Temas principales de LPIC-1:

- Arquitectura del sistema
- Instalación de Linux y gestión de paquetes
- Comandos GNU y Unix
- Dispositivos, sistemas de archivos y FHS
- Shells, scripting y gestión de datos
- Interfaces de usuario y escritorios
- Tareas administrativas
- Servicios esenciales del sistema
- Fundamentos de redes
- Seguridad

### Paso 3: Investigar LFCS (Linux Foundation)

```bash
$ curl -sL https://www.linuxfoundation.org/certification/fcs | grep -i "objectives\|domains" | head -10
```

Temas LFCS:

- Gestión del sistema operativo Linux
- Seguridad del sistema
- Servicios del sistema y redes
- Contenedores y orquestación básica

### Paso 4: Comparar costos y requisitos

Crea una tabla comparativa:

```bash
$ cat << 'EOF'
| Certificación | Costo examen | Tipo | Válida por | Prerrequisitos |
|---|---|---|---|---|
| RHCSA | ~$400 USD | Práctico (RHEL) | 3 años | Ninguno |
| LPIC-1 | ~$200 USD c/u (2 exámenes) | Teórico | Indefinida | Ninguno |
| LFCS | ~$300 USD | Remoto práctico | 5 años | Ninguno |
EOF
```

### Paso 5: Identificar brechas personales de habilidad

Crea un archivo de autoevaluación:

```bash
$ nano ~/skill-gaps.md
```

Evalúa tu nivel en cada área (1-5):

```markdown
# Autoevaluación - Brechas de habilidad

## RHCSA Domains
- Herramientas esenciales: [1-5]
- Operación de sistemas: [1-5]
- Almacenamiento: [1-5]
- Sistemas de archivos: [1-5]
- Despliegue/configuración: [1-5]
- Usuarios/grupos: [1-5]
- Seguridad: [1-5]

## Áreas prioritarias a reforzar (puntuación < 3):
1. 
2. 
3. 
```

### Paso 6: Crear un plan de estudio

```bash
$ nano ~/study-plan.md
```

```markdown
# Plan de estudio: RHCSA

## Semana 1-2: Fundamentos
- Comandos esenciales, bash, redirecciones, pipes
- Gestión de procesos y servicios

## Semana 3-4: Almacenamiento
- LVM, particionado, sistemas de archivos
- Montaje y automontaje

## Semana 5-6: Usuarios y seguridad
- Usuarios, grupos, permisos ACL
- SELinux básico, firewalld

## Semana 7-8: Red y automatización
- Configuración de red, bonding, equipo
- Scripting avanzado, systemd

## Semana 9-10: Práctica intensiva
- Laboratorios cronometrados
- Exámenes de práctica
```

**Verificación:** Debes tener una tabla comparativa de certificaciones, una autoevaluación de brechas de habilidad, y un plan de estudio personalizado de al menos 4 semanas.

### 🚀 Desafío individual

**Escenario:** Tienes 30 días para preparar el examen RHCSA. Ya trabajas 8 horas al día y solo puedes dedicar 1 hora diaria entre semana y 3 horas los sábados.

Crea un plan de estudio realista día por día (calendario). Incluye:
- What to study each day
- Which labs to practice
- Which topics to review
- 2 full mock exams los domingos

Escribe el plan en `~/30-day-rhcsa-plan.md`. Prioriza los temas con más peso en el examen real.

---

## 💪 Ejercicios (para casa / laboratorio)

1. Identifique tres certificaciones relevantes para su carrera e investigue sus requisitos
2. Cree un plan de estudio independiente para los próximos 6 meses
3. Explore opciones de formación gratuita (cursos online, laboratorios virtuales)
4. Documente sus proyectos personales de IT para añadirlos a su currículum
5. Investigue si su empleador tiene programa de reembolso de matrícula

---


## Curiosidad: GNU = "GNU's Not Unix"

GNU es el acrónimo recursivo más famoso de la informática: "GNU's Not Unix" ("GNU No es Unix"). Fue creado por Richard Stallman en 1983 como un sistema operativo completamente libre. El proyecto GNU desarrolló herramientas esenciales: el compilador GCC, el depurador GDB, el editor Emacs, Bash, Coreutils. Linux aportó el kernel que faltaba.

Stallman insiste en llamar al sistema "GNU/Linux" no solo "Linux", argumentando que Linux es solo el kernel y GNU es el sistema operativo. Linus Torvalds no está del todo de acuerdo. El debate sigue activo 30 años después.
