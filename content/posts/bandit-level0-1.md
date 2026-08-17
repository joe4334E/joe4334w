---
title: "OverTheWire Bandit: Nivel 0 -> 1"
date: 2026-07-25
draft: false
description: "Solución del nivel 0 -> 1 del wargame OverTheWire Bandit."
featuredImage: "/images/overthewire-tux.png"
level: "Nivel 0 -> 1"
prev: ""
next: ""
tags: ["overthewire", "bandit", "walkthrough", "linux"]
categories: ["OverTheWire"]
---

{{< serie prev >}}

## Login

SSH: `ssh bandit0@bandit.labs.overthewire.org -p 2220`

Password: `bandit0`

## Task

> El objetivo del nivel: *"The password for the next level is stored in a file called **readme** located in the home directory."*

## Teoría

Explica aquí los comandos y conceptos que se necesitan para resolver este nivel:

- `pwd` — muestra el directorio de trabajo actual.
- `ls` — lista los archivos del directorio actual.
- `cat` — imprime el contenido de un archivo.

## Solución

1. Paso 1...
2. Paso 2...

```bash
bandit0@bandit:~$ ls
readme
bandit0@bandit:~$ cat readme
<password_bandit1>
```

La cadena resultante es la contraseña del usuario **bandit1**.

## Enlace oficial

[Página del nivel en OverTheWire](https://overthewire.org/wargames/bandit/bandit1.html)

{{< serie next >}}
