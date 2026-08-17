---
title: "Introducción a OverTheWire Bandit"
date: 2026-07-24T17:35:14-0400
draft: false
description: "Qué es el wargame Bandit de OverTheWire, cómo conectarte por SSH y consejos para seguir esta serie de niveles."
featuredImage: "/images/overthewire-tux.png"
tags: ["overthewire", "linux", "seguridad"]
categories: ["OverTheWire"]
---

## ¿Qué es OverTheWire?

[OverTheWire](https://overthewire.org/wargames/) es una plataforma gratuita con **wargames**: juegos de captura de la bandera pensados para aprender seguridad informática y administración de sistemas. Cada reto te da una máquina real a la que te conectas por **SSH** y debes resolver un objetivo para obtener la contraseña del siguiente nivel.

## ¿Qué es Bandit?

**Bandit** es el wargame de iniciación: está pensado para quien empieza desde cero con Linux. A lo largo de los niveles irás usando comandos como `ls`, `cat`, `find`, `grep`, `sort`, `base64` o `ssh` para encontrar contraseñas escondidas en archivos, directorios y procesos.

Cada nivel se escribe como `N -> N+1`: inicias sesión como el usuario `banditN` y al resolverlo obtienes la contraseña del usuario `banditN+1`.

## Cómo conectarte

Todos los niveles se juegan en el mismo servidor, por el puerto `2220`:

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

La contraseña inicial del nivel 0 es `bandit0`.

## Estructura de esta serie

Cada artículo de la serie sigue el mismo formato:

- **Login** — el comando SSH y la contraseña del nivel actual.
- **Task** — el objetivo que debes cumplir.
- **Teoría** — los comandos y conceptos necesarios.
- **Solución** — los pasos para resolverlo.
- **Enlace oficial** — la página del nivel en OverTheWire.
- Enlaces a **nivel anterior** y **siguiente nivel** para no perderte.

Niveles publicados:

- [Nivel 0 -> 1](/posts/bandit-level0-1/)

## Consejos

- **Guarda las contraseñas** en un archivo de notas local: Bandit no las guarda por ti y si las pierdes tienes que volver a empezar.
- **Toma notas** de cómo resolviste cada nivel: te servirá para repasar y para los niveles más avanzados.
- Si no entiendes un comando, usa `man <comando>` para leer su documentación.
- Los usuarios `banditN` se desconectan solos tras un rato inactivo; no pasa nada, vuelve a entrar.

¡Buena suerte y a resolver niveles!
