---
title: "linux-05 Conectar a la Red"
date: 2026-08-03
draft: false
featuredImage: "/images/blog-space.jpg"
tags: [red ip ssh dhcp estatica seguridad]
categories: ["Linux Admin"]
series: ["Practical Linux System Administration"]
weight: 05
bookAuthor: "Kenneth Hess"
bookTitle: "Practical Linux System Administration"
bookYear: "2023"
---

# Tutorial 5: Conectar a la red

**Capítulo 5 del libro** — "Practical Linux System Administration" (pags. 45-55)
**Nivel:** Principiante

---

## ⚡ Para empezar: "The cloud is just someone else's computer"

Y antes de la nube, habia que conectar servers a una red fisica. El momento en que conectas un sistema a la red, ya esta siendo escaneado por bots.

**Polemica:** Por que llamamos "spam" al correo basura? Sketch de Monty Python (1970): un restaurante solo sirve "Spam, spam, spam, spam..." y los vikingos cantan "spam, spam, spam" ahogando todo. Exactamente lo que hace el correo basura: satura hasta que no se oye nada mas.

> "Dos tipos de sistemas Linux pueden considerarse seguros: uno apagado y uno encendido pero no conectado a una red."

---

## Objetivos

- Entender IP estatica vs dinamica (DHCP)
- Conocer implicaciones de seguridad de red
- Asegurar el demonio SSH
- Restringir acceso por IP, deshabilitar root login, usar claves SSH

---

## 1. IP Estatica vs DHCP

Red de ejemplo del libro: `192.168.1.0/24`

| Estatica | DHCP |
|----------|------|
| No cambia | Asignada automaticamente |
| Servidores, impresoras | Workstations, laptops |
| Gestion intensiva | Mantenimiento minimo |

> "Reservo `192.168.1.1` a `192.168.1.25` para servidores, equipos de red, impresoras."

### 🖐️ Mini-ejercicio

```bash
$ ip addr show
$ ip route show
```

**Tu IP es estatica o DHCP?** Cual es tu gateway? Comparte con la clase — cuantos estan en la misma subred?

---

## 2. Seguridad en redes

> "Una vez que conectas un sistema a la red, lo has expuesto a ataques. Actores maliciosos escanean continuamente rangos de IP."

**4 medidas basicas:**
1. Minimo privilegio
2. Contrasenas seguras / claves / MFA
3. Parches y actualizaciones
4. Auditorias periodicas

### Podar el sistema
> "Podar significa eliminar cualquier servicio y demonio innecesario."

Minimo necesario: solo SSH.

---

## 3. Asegurar SSH

### hosts.allow y hosts.deny (TCP Wrappers)

> "TCP wrappers provide access control to network services based on the client's IP address or hostname."

TCP wrappers usan dos archivos en `/etc`:
- `/etc/hosts.allow` — Permisos explícitos
- `/etc/hosts.deny` — Denegaciones explícitas

**Orden de evaluación:** `hosts.allow` se evalúa primero. Si una regla en `hosts.allow` coincide, `hosts.deny` no se evalúa.

Sintaxis general:
```
daemon_list : client_list [: shell_command]
```

Ejemplos del libro:
```
# Permitir SSH solo desde la red local
sshd: 192.168.1.

# Permitir múltiples servicios desde una IP específica
sshd, vsftpd: 192.168.1.50

# Denegar todo lo demás
ALL: ALL
```

> "Use ALL: ALL in hosts.deny as your default-deny policy, then add specific allow rules in hosts.allow."

Verificar si un binario usa TCP wrappers:
```bash
$ ldd /usr/sbin/sshd | grep libwrap
```

Si `ldd` no muestra `libwrap`, el servicio fue compilado sin soporte de TCP wrappers y necesitas firewalld o iptables.

### Firewalld (firewall-cmd)

> "Firewalld is the default firewall management tool on RHEL/CentOS 7+ and Fedora."

Firewalld organiza las reglas en **zonas**. Cada zona tiene un nivel de confianza distinto:

| Zona       | Nivel de confianza | Uso típico              |
|------------|-------------------|-------------------------|
| drop       | Mínimo            | Rechazar todo           |
| block      | Bajo              | Rechazar con icmp       |
| public     | Bajo              | Redes públicas          |
| external   | Bajo              | NAT / router            |
| dmz        | Medio             | Zona desmilitarizada    |
| work       | Medio-alto        | Red laboral             |
| home       | Alto              | Red doméstica           |
| internal   | Alto              | Red interna             |
| trusted    | Máximo            | Aceptar todo            |

**Reglas runtime vs permanentes:** Por defecto, los cambios son inmediatos pero no persisten tras reinicio. Usa `--permanent` para hacerlos persistentes.

```bash
# Listar zonas y configuración
$ sudo firewall-cmd --get-active-zones
$ sudo firewall-cmd --list-all

# Servicios predefinidos
$ sudo firewall-cmd --get-services
$ sudo firewall-cmd --info-service=ssh

# Añadir reglas
$ sudo firewall-cmd --permanent --add-service=http
$ sudo firewall-cmd --permanent --add-port=8080/tcp
$ sudo firewall-cmd --permanent --remove-service=telnet

# Recargar configuración permanente
$ sudo firewall-cmd --reload

# Crear zona personalizada para SSH
$ sudo firewall-cmd --permanent --new-zone=ssh_zone
$ sudo firewall-cmd --permanent --zone=ssh_zone --add-source=192.168.1.50
$ sudo firewall-cmd --permanent --zone=ssh_zone --add-service=ssh
$ sudo firewall-cmd --reload
```

### iptables y nftables

> "iptables has been the standard Linux firewall for decades, but nftables is its modern replacement."

**iptables** — El firewall clásico de Linux. Organiza reglas en tablas y cadenas (chains):

| Tabla   | Cadenas incorporadas       | Propósito            |
|---------|---------------------------|----------------------|
| filter  | INPUT, OUTPUT, FORWARD    | Filtrado de paquetes |
| nat     | PREROUTING, POSTROUTING   | NAT / forwarding     |
| mangle  | PREROUTING, OUTPUT        | Modificar paquetes   |

Ejemplo del libro: Restringir SSH a una IP específica:

```bash
# Permitir SSH solo desde 192.168.1.50
$ sudo iptables -A INPUT -p tcp --dport 22 -s 192.168.1.50 -j ACCEPT

# Denegar SSH desde cualquier otra IP
$ sudo iptables -A INPUT -p tcp --dport 22 -j DROP

# Ver reglas
$ sudo iptables -L -n -v

# Política por defecto: denegar todo el tráfico entrante
$ sudo iptables -P INPUT DROP
$ sudo iptables -P FORWARD DROP
$ sudo iptables -P OUTPUT ACCEPT
```

> "Create a rule set that allows SSH from only the management station and drops all other SSH connection attempts."

Guardar reglas persistentemente:
```bash
# RHEL/CentOS
$ sudo service iptables save

# Debian/Ubuntu
$ sudo apt install iptables-persistent
$ sudo netfilter-persistent save
```

**nftables** — El reemplazo moderno. Unifica iptables, ip6tables, arptables y ebtables en un solo framework:

```bash
# Ver reglas nftables
$ sudo nft list ruleset

# Regla equivalente para SSH
$ sudo nft add rule inet filter input tcp dport 22 ip saddr 192.168.1.50 accept

# nftables es el backend por defecto de firewalld en RHEL 8+ y Debian 10+
$ sudo systemctl status nftables
```

> "nftables is the default backend for firewalld in modern distributions. When you add a rule with firewall-cmd, it creates nftables rules under the hood."

### Denegar root SSH

> "El usuario root nunca debe iniciar sesion en ningun sistema via SSH."

Editar `/etc/ssh/sshd_config`:
```
PermitRootLogin no
```

### Claves en lugar de contrasenas
```bash
$ ssh-keygen -t rsa
$ ssh-copy-id tux@192.168.1.99
$ ssh tux@192.168.1.99
```

### 🖐️ Mini-ejercicio

```bash
$ grep PermitRootLogin /etc/ssh/sshd_config
$ ssh-keygen -t rsa
```

**Root SSH esta permitido en tu sistema?** Genera un par de claves. Donde se guardaron? (`~/.ssh/id_rsa` y `~/.ssh/id_rsa.pub`)

---

## 4. Diagnostico de red basico

Cuando un sistema no tiene conectividad, estos comandos ayudan a identificar el problema:

```bash
$ ping -c 4 8.8.8.8          # Prueba conectividad basica
$ traceroute 8.8.8.8          # Ruta hasta el destino
$ ss -tulpn                   # Puertos en escucha
$ dig +short google.com       # Resolucion DNS
```

> "The network configuration of any system is the most critical component of getting that system working on the network."

Si `ping` falla, revisar primero la interfaz con `ip link set dev eth0 up` y luego el gateway con `ip route show default`.

### dig — DNS lookup avanzado

`dig` (Domain Information Groper) es la herramienta más potente para consultas DNS. Soporta múltiples tipos de registro:

```bash
# Tipos de registro DNS
$ dig google.com A          # Registro IPv4
$ dig google.com AAAA       # Registro IPv6
$ dig google.com MX         # Servidores de correo
$ dig google.com NS         # Servidores de nombres
$ dig google.com TXT        # Registros de texto (SPF, DKIM)

# Consultar un servidor DNS específico
$ dig @8.8.8.8 google.com

# Resolución inversa (PTR)
$ dig -x 8.8.8.8

# Salida breve (solo la respuesta)
$ dig +short google.com
```

### nslookup — DNS lookup interactivo

```bash
$ nslookup google.com
Server:         192.168.1.1
Address:        192.168.1.1#53
Non-authoritative answer:
Name:   google.com
Address: 142.250.80.46

# Especificar servidor DNS
$ nslookup google.com 8.8.8.8

# Modo interactivo (útil para múltiples consultas)
$ nslookup
> server 8.8.8.8
> set type=mx
> google.com
> exit
```

### host — DNS lookup simplificado

```bash
$ host google.com
google.com has address 142.250.80.46
google.com mail is handled by 10 smtp.google.com

$ host -t mx google.com
$ host -t ns google.com
```

> "The `dig` command is the most flexible and powerful of the DNS lookup utilities. Use `dig` for troubleshooting and `host` for quick lookups."

### 🖐️ Mini-ejercicio

Tu sistema puede resolver nombres DNS? Ejecuta `dig +short google.com`. Que pasa si desactivas la resolucion editando `/etc/resolv.conf`? Reactivalo despues.

---

## 🔧 Laboratorio práctico (en clase)

### Paso 1: Verificar configuración de red
```bash
ip addr show
ip route show
ss -tuln
hostname -I
```
`ip addr` muestra las interfaces y direcciones IP. `ip route` muestra la tabla de enrutamiento (puerta de enlace). `ss -tuln` lista puertos TCP/UDP en escucha.

### Paso 2: Configurar SSH hardening básico
```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
sudo nano /etc/ssh/sshd_config
```
Modificar las siguientes líneas en `sshd_config`:
```
Port 2222
PermitRootLogin no
PasswordAuthentication yes
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
```

### Paso 3: Probar SSH hardening
```bash
sudo systemctl restart sshd
ssh -p 2222 localhost
ss -tuln | grep 2222
```
Verificar que SSH ahora escucha en el puerto 2222 y que conexiones como root son rechazadas.

### Paso 4: Escanear con nmap
```bash
nmap -p- localhost
nmap -sV localhost
```
Escanean todos los puertos TCP en localhost. Identifican servicios y versiones. Luego limitan a puertos específicos:
```bash
nmap -p 22,80,443,2222 localhost
```

### Paso 5: Configurar firewall con ufw
```bash
sudo ufw status
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2222/tcp
sudo ufw allow 80/tcp
sudo ufw enable
sudo ufw status verbose
```
Configuran reglas básicas de firewall: denegar todo el tráfico entrante excepto SSH (puerto 2222) y HTTP (puerto 80).

**Verificación:** `ss -tuln` debe mostrar solo los servicios configurados. `nmap localhost` debe reportar solo los puertos abiertos esperados. `sudo ufw status` debe mostrar las reglas activas.

### 🚀 Desafío individual

Realiza un hardening completo de SSH con los siguientes requisitos:

1. Cambia el puerto SSH a 2222 (ya hecho en el lab)
2. **Deshabilita la autenticación por contraseña** (`PasswordAuthentication no`) — solo acceso por clave SSH
3. Deshabilita el login de root (`PermitRootLogin no`)
4. Genera un par de claves SSH con `ssh-keygen -t ed25519`
5. Copia la clave pública a `~/.ssh/authorized_keys`
6. Verifica que puedes conectarte con la clave y que el login con contraseña falla

Comando útil: `ssh-copy-id -p 2222 usuario@localhost`

---

## 💪 Ejercicios (para casa / laboratorio)

1. **Mapa de red:** Escanea tu red local con `nmap -sn 192.168.1.0/24` (ajusta la subred). Identifica cuantos dispositivos hay, sus IPs y sus puertos abiertos con `nmap -sV <ip>`. Documenta TODO lo que encuentres. Que dispositivos no esperabas encontrar?

2. **Hardening de SSH:** Crea una guia paso a paso para endurecer un servidor SSH que incluya:
   - Denegar root login
   - Usar solo autenticacion por clave (deshabilitar passwords)
   - Cambiar el puerto por defecto (p.ej. 2222)
   - Limitar acceso por IP
   - Configurar MaxAuthTries a 3
   - Deshabilitar protocolos antiguos
   Pruebalo en una VM y verifica que funciona.

3. **Firewall desde cero:** Sin usar firewalld o ufw, crea reglas `iptables` que:
   - Permitan SSH solo desde tu IP
   - Permitan HTTP/HTTPS a todos
   - Bloqueen todo lo demas
   - Hagan las reglas persistentes
   Documenta cada regla.

4. **Ataque de fuerza bruta:** En una VM de pruebas, instala `fail2ban`. Configuralo para SSH (3 intentos, baneo 10 minutos). Simula un ataque con `ssh` fallando 5 veces. Verifica que la IP queda baneada en `iptables -L -n`.

5. **Investigacion:** Que es `ssh-audit`? Instalalo y ejecutalo contra tu propio servidor SSH. Que recomendaciones da? Implementa al menos 3.

---


## Curiosidad: 127.0.0.1

Por que localhost es 127.0.0.1? Los ingenieros de Internet asignaron `127.0.0.0/8` como red de loopback. `127` fue el ultimo prefijo de Clase A disponible. `127.0.0.1` es solo la primera direccion de esa red.

Antes de SSH existia `rsh` y `rlogin` — enviaban contrasenas en texto plano. Cualquier persona con un sniffer en la red podia capturarlas. SSH los hizo obsoletos.
