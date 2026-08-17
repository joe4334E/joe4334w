---
title: "linux-09 Monitorizacion"
date: 2026-08-07
draft: false
featuredImage: "/images/blog-space.jpg"
tags: [top htop ps sar iostat pidstat procesos]
categories: ["Linux Admin"]
series: ["Practical Linux System Administration"]
weight: 09
bookAuthor: "Kenneth Hess"
bookTitle: "Practical Linux System Administration"
bookYear: "2023"
---

# Tutorial 9: Monitorización del Sistema

**Capítulo 9 del libro** — "Practical Linux System Administration" (pags. 121-135)
**Nivel:** Intermedio

---

## ⚡ Para empezar: `top` cumple 40 años (y sigue siendo el rey)

El comando `top` fue creado por William LeFebvre en 1984 para BSD Unix. Originalmente solo mostraba los procesos que más CPU consumían (de ahí "TOP"). 40 años después, sigue siendo el estándar en cada distribución Linux. Convive con herramientas modernas como `htop`, `atop` y `glances`, pero cuando no tienes nada más instalado, `top` siempre está ahí.

El libro lo dice claro: "Monitoring is not optional. It's essential."

---

## Objetivos

- Comprender por qué la monitorización es esencial
- Usar `top`, `atop`, `htop` y `ps` para monitorizar procesos en tiempo real
- Interpretar estadísticas de CPU, memoria y disco con `sar`
- Generar informes formateados con `sadf`
- Monitorizar E/S de dispositivos con `iostat`
- Recopilar estadísticas de procesadores con `mpstat`
- Rastrear tareas individuales con `pidstat`

---

## 1. `top` — Visión general de procesos

`top` es el comando estándar disponible en todas las distribuciones. Muestra una vista en tiempo real de los procesos que más CPU consumen:

```
$ top
```

Atajos de teclado (Tabla 9-1 del libro):

| Tecla       | Ordenación           |
|-------------|----------------------|
| Mayús + M   | %MEM (uso de memoria)|
| Mayús + P   | %CPU (por defecto)   |
| Mayús + S   | TIME (tiempo ejec.)  |

---

## 2. `atop` — Monitor avanzado

No instalado por defecto. Muestra procesos en el panel inferior y CPU, memoria, disco y red en el panel superior:

```
$ sudo atop
```

> "The atop utility is an essential part of a system administrator's toolbox."

Cuando se ejecuta como root: `Unrestricted view (privileged)`. Como usuario normal: `Restricted view (unprivileged)`.

---

## 3. `htop` — Monitor con color y menú

```
$ sudo htop
```

Colorido, con menú de comandos que evita memorizar atajos de teclado.

### 🖐️ Mini-ejercicio

```bash
$ sudo apt install htop   # o: sudo yum install htop
$ htop
```

**¿Cuántos procesos tienes? ¿Cuál consume más CPU?** Sal de `htop` con `F10` o `q`. Luego ejecuta `top` y compara las vistas. ¿Cuál prefieres?

---

## 4. `ps` — Listar procesos

Ver todos los procesos del usuario actual:

```
$ ps -ux
```

Ver todos los procesos del sistema:

```
$ ps -aux
```

---

## 5. `glances` — Monitor multiplataforma

```
$ glances
```

Teclas: `M` para MEM%, `T` para TIME, `C` para CPU%.

> "If your system is CPU-constrained, don't use glances as your primary CPU performance check because it consumes about 15% of your CPU as opposed to top's 3%–5%."

---

## 6. `sar` — Reportes de actividad del sistema

Parte del paquete `sysstat`. Sin opciones muestra estadísticas de CPU desde medianoche:

```
$ sar
```

Opciones de filtrado:

```
$ sar -u ALL      # Estadísticas completas de CPU
$ sar -B          # Estadísticas de paginación
$ sar -b          # Estadísticas de E/S y tasa de transferencia
```

---

## 7. `sadf` — Formatear datos de sar

Genera datos en múltiples formatos (CSV, XML, etc.):

```
$ sadf -d /var/log/sa/sa21
```

### 🖐️ Mini-ejercicio

```bash
$ sar -u ALL
```

**¿Cuál es tu porcentaje de idle (`%idle`)?** Si es menor de 20%, tu CPU está al límite. Anota los valores de usuario (`%user`) y sistema (`%system`).

---

## 8. `iostat` — Estadísticas de E/S

```
$ iostat -d 2             # Con resumen incluido
$ iostat -y -d 2          # Sin resumen
```

---

## 9. `mpstat` — Estadísticas de multiprocesador

```
$ mpstat                  # Resumen de todos los procesadores
$ mpstat -P 1             # Procesador específico
$ mpstat -P ALL           # Todos los procesadores
$ mpstat 5 3              # Cada 5 segundos, 3 iteraciones
```

---

## 10. `pidstat` — Tareas individuales

```
$ pidstat                          # Tareas activas
$ pidstat -d -U khess              # Uso de disco por usuario
```

---

## 11. `cifsiostat` — Estadísticas CIFS/Samba

```
$ cifsiostat
```

---

## 🔧 Laboratorio práctico (en clase)

### Paso 1: top y htop — visión general de procesos

```bash
$ top
# Teclas útiles dentro de top:
#   P = ordenar por CPU
#   M = ordenar por MEM
#   k = matar proceso
#   q = salir
$ htop   # si no está instalado: sudo apt install htop
```
`htop` añade navegación con flechas, colores y la posibilidad de enviar señales sin recordar números de señal. Observa los campos: `%CPU`, `%MEM`, `TIME+`, `RES` (memoria física usada).

### Paso 2: Análisis detallado con ps aux

```bash
$ ps aux --sort=-%cpu | head -20
$ ps aux --sort=-%mem | head -10
$ ps -eo pid,ppid,cmd,%cpu,%mem --sort=-%cpu | head -15
```
`ps aux` muestra todos los procesos del sistema. El `--sort` permite identificar rápidamente los procesos que más consumen. Analiza la columna `STAT`: `R` (running), `S` (sleeping), `Z` (zombie).

### Paso 3: sar — datos históricos de rendimiento

```bash
$ sar -u   # uso de CPU (promedio desde que arrancó la recolección)
$ sar -r   # estadísticas de memoria
$ sar -b   # E/S de bloques
$ sar -n DEV   # tráfico de red por interfaz
$ sar -u -s 08:00:00 -e 09:00:00   # ventana de tiempo específica
```
`sar` lee los datos almacenados por `sadc` en `/var/log/sysstat/`. Si no hay datos históricos, genera una muestra en tiempo real con `sar 1 5` (cada 1 segundo, 5 muestras).

### Paso 4: iostat — rendimiento de disco detallado

```bash
$ iostat -x 2 4
```
Columnas clave:
- `%util` — porcentaje de tiempo que el disco estuvo ocupado (saturación)
- `r/s`, `w/s` — operaciones de lectura/escritura por segundo
- `await` — tiempo promedio de respuesta (ms)
- `svctm` — tiempo de servicio real (ms)

Un `%util` cercano al 100% indica saturación, pero en SSD con NCQ puede ser engañoso — hay que mirar `await`.

### Paso 5: mpstat — estadísticas por CPU

```bash
$ mpstat -P ALL 2 3
```
Muestra el uso individual de cada núcleo. Útil para detectar desbalanceo de carga (un núcleo al 100% y otros ociosos) o procesos monohilo (`%usr` alto en un solo core). La columna `%idle` muestra tiempo ocioso; si está cerca de 0, hay saturación de CPU.

**Verificación:** `sar -u 1 3` debe devolver tres líneas con métricas de CPU en tiempo real.

### 🚀 Desafío individual

Deja `stress` corriendo en una terminal y monitoréalo con `pidstat` en otra:

**Terminal 1:**
```bash
$ sudo apt install -y stress
$ stress --cpu 2 --io 1 --vm 1 --vm-bytes 256M --timeout 120
```

**Terminal 2:**
```bash
$ pidstat -p $(pgrep -d, stress) -u -r -d 2 10
```

Observa cómo `pidstat` reporta por separado CPU, memoria y E/S del proceso `stress`. ¿Qué columna te indica que el proceso está forzando E/S de disco?

---

## 💪 Ejercicios (para casa / laboratorio)

1. Ejecute `top` y use Mayús+M, Mayús+P y Mayús+S para cambiar la ordenación
2. Instale `atop` y `htop` y compare sus vistas con `top`
3. Ejecute `sar` sin opciones y luego con `-u ALL`, `-B` y `-b`
4. Use `sadf` para exportar datos de sar a formato CSV
5. Ejecute `mpstat -P ALL` y observe la diferencia entre procesadores
6. Use `pidstat -d -U $USER` para ver su propia actividad de E/S

---


## Curiosidad: La señal que no avisa

La señal `kill -9` (SIGKILL) mata un proceso sin limpieza — no le da oportunidad de cerrar archivos ni liberar recursos. Es el "apagón nuclear" de Linux. La señal por defecto (`kill` sin número) es `kill -15` (SIGTERM), que pide amablemente al proceso que termine.

Los procesos "zombie" (estado Z en `ps`) son procesos que ya terminaron pero cuyo padre aún no ha recogido su código de salida. No consumen recursos... excepto una entrada en la tabla de procesos. Si se acumulan, pueden llenarla.

La herramienta `glances` fue creada por Nicolas Hennion en 2010 y está escrita en Python. Su nombre viene de "una mirada rápida" al sistema. Consume ~15% de CPU porque carga múltiples plugins: CPU, memoria, disco, red, sensores, procesos, docker, y más.
