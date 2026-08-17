---
title: "Sesion 3: Almacenamiento y Mantenimiento"
date: 2026-08-20
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

# Sesion 3: Almacenamiento y Mantenimiento

**Tutoriales:** 07 — Almacenamiento | 08 — Mantenimiento del Sistema | 09 — Monitorizacion
**Nivel:** Intermedio
**Duracion estimada:** 4-5 horas

---

## Objetivos de cada tutorial

### Tutorial 07 — Almacenamiento
- Comprender discos, sistemas de archivos, montaje y LVM
- Anadir un nuevo disco al sistema
- Crear particiones y sistemas de archivos
- Implementar volumenes logicos (LVM)
- Extender volumenes logicos
- Desmantelar y desechar discos

### Tutorial 08 — Mantenimiento del Sistema
- Limpiar el directorio `/tmp` con tmp.mount
- Mover `/home` a su propia particion
- Deduplicar archivos con `fdupes`
- Implementar cuotas de disco
- Parchear el sistema
- Mantener cuentas de usuario y grupos
- Monitorizar con sysstat

### Tutorial 09 — Monitorizacion
- Comprender por que la monitorizacion es esencial
- Usar `top`, `atop`, `htop` y `ps` para monitorizar procesos en tiempo real
- Interpretar estadisticas de CPU, memoria y disco con `sar`
- Generar informes formateados con `sadf`
- Monitorizar E/S de dispositivos con `iostat`
- Recopilar estadisticas de procesadores con `mpstat`
- Rastrear tareas individuales con `pidstat`

---

## Resumen

El Tutorial 07 es fundamental para cualquier administrador. La jerarquia LVM (PV → VG → LV) permite redimensionar volumenes sin reiniciar, combinar discos y crear snapshots. XFS no se puede reducir (solo crecer); ext4 si permite shrink. Siempre usa UUID en `/etc/fstab` en lugar de nombres de dispositivo, porque los UUID persisten aunque cambien los discos. `tmpfs` es mas seguro que `ramfs` porque tiene limite de tamaño configurable.

El Tutorial 08 cubre el mantenimiento diario. `fdupes` encuentra archivos duplicados por contenido (no por nombre). Las cuotas de disco (usrquota/grpquota en fstab) tienen dos tipos: soft (puede excederse temporalmente) y hard (absoluto). `tmp.mount` hace que `/tmp` sea un tmpfs en RAM, mas rapido y seguro. sysstat es la suite de monitorizacion historica.

El Tutorial 09 profundiza en herramientas de monitoreo. `top` ordena por CPU por defecto (no por memoria como muchos creen). `glances` consume ~15% de CPU vs 3-5% de `top`. Los procesos zombie (estado Z) no consumen recursos pero ocupan entradas en la tabla de procesos. `iostat -x` muestra `%util` para saturacion de disco y `await` para latencia. Un `await` alto con `%util` bajo sugiere problemas de controlador, no saturacion.

---

## Comandos clave

| Comando | Descripcion |
|---------|-------------|
| `lsblk` | Muestra discos y particiones |
| `pvcreate / dev/sdb1` | Crea un Physical Volume |
| `vgcreate vg_datos /dev/sdb1` | Crea un Volume Group |
| `lvcreate -L 10G -n lv_data vg_datos` | Crea un Logical Volume |
| `lvextend -l +100%FREE lv_data` | Extiende LV con todo el espacio libre |
| `mkfs.xfs /dev/sdb1` | Crea sistema de archivos XFS |
| `blkid /dev/sdb1` | Muestra UUID del dispositivo |
| `mount -a` | Monta todo lo definido en fstab |
| `fdupes -rS /directorio` | Busca duplicados recursivamente |
| `iostat -x 3` | Estadisticas de disco cada 3 seg |
| `sar -u 2 5` | Uso de CPU cada 2 seg, 5 veces |
| `ps aux --sort=-%mem | head -10` | Top 10 procesos por memoria |

---

## Ejercicio integrador A

**Particionado y montaje persistente:**
1. Agrega un disco virtual de 10GB a tu maquina
2. Crea una particion con `fdisk` y un sistema de archivos ext4
3. Monta el disco en `/mnt/datos`
4. Obten el UUID con `blkid` y agrega una entrada en `/etc/fstab`
5. Ejecuta `mount -a` y verifica que persiste tras reiniciar
6. Crea un usuario `datos` y configura cuotas de disco (50MB soft, 100MB hard)

## Ejercicio integrador B

**Disco LVM con monitoreo:**
1. Agrega dos discos virtuales de 5GB cada uno
2. Crea PVs, un VG `vg_prod` y un LV `lv_datos` de 8GB
3. Formatea con XFS y monta en `/srv/datos`
4. Monitorea el uso de disco con `iostat -x 1 5`
5. Extiende el LV con `lvextend` y `xfs_growfs`
6. Usa `sar` para capturar estadisticas de E/S durante 30 segundos

---

## Para recordar

- LVM: PV → VG → LV; XFS no se reduce
- Siempre usa UUID en `/etc/fstab`
- `fdupes` compara contenido, no nombres
- Cuotas: soft (temporal) vs hard (absoluto)
- `top` ordena por CPU por defecto, no por memoria
- `%util` alto = disco saturado; `await` alto = latencia
