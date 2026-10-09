# BreakMySSH

**Plataforma:** DockerLabs\
**Dificultad:** Muy fácil\
**Área:** Reconocimiento, SSH y credenciales débiles\
**Material de referencia:** [Writeup utilizado en clase](https://github.com/PabloSanchezIglesias/DockerLabs/blob/main/Muy%20facil/BreakMySSH.md)

## Qué trabajamos con esta máquina

BreakMySSH fue una de las máquinas utilizadas para introducir una metodología sencilla de pentesting sobre servicios de red. El objetivo principal fue aprender a pasar de un escaneo inicial a una hipótesis concreta de ataque.

## Reconocimiento

El writeup de referencia comienza con un escaneo completo de puertos mediante `nmap`:

```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG escaneo
```

El resultado muestra un único puerto expuesto: **22/TCP**, correspondiente a SSH. A partir de ahí, la superficie de ataque queda claramente acotada.

Después se realiza enumeración específica del servicio:

```bash
sudo nmap -sCV -p22 172.17.0.2 -oN allPorts
```

La máquina expone **OpenSSH 7.7**.

## Investigación de la versión

A partir de la versión detectada, el laboratorio introduce el uso de `searchsploit` para investigar vulnerabilidades conocidas:

```bash
searchsploit OpenSSH 7.7
```

El writeup utilizado en clase relaciona esta versión con técnicas de enumeración de usuarios y con **CVE-2018-15473**. Esta parte es útil para enseñar que encontrar una versión no implica automáticamente que exista una explotación remota directa: primero hay que investigar qué condición, impacto y alcance tiene cada vulnerabilidad.

## Validación de usuarios

La práctica continúa probando la existencia de un usuario válido sobre SSH. En el writeup de referencia se valida el usuario `root` antes de avanzar al siguiente paso.

## Credenciales débiles

Con un usuario válido identificado, se utiliza Hydra con un diccionario para demostrar el impacto de una contraseña débil en un servicio SSH expuesto:

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt 172.17.0.2 ssh -t 15
```

El laboratorio de DockerLabs utiliza una credencial deliberadamente vulnerable para fines educativos.

## Acceso y verificación

Una vez obtenida la credencial de laboratorio, el acceso se valida mediante SSH y comandos básicos de identificación:

```bash
ssh root@172.17.0.2
whoami
id
hostname
```

## Aprendizajes principales

* Un escaneo inicial sirve para reducir la superficie de ataque.
* La enumeración de versiones debe ir seguida de investigación y validación.
* La existencia de un usuario válido puede reducir drásticamente el espacio de búsqueda de un ataque de credenciales.
* SSH no es inseguro por sí mismo; el problema del laboratorio está en la combinación de exposición, usuario privilegiado y contraseña débil.
* Obtener acceso no cierra el ejercicio: hay que entender qué control defensivo habría evitado el compromiso.

## Mitigación

* Deshabilitar acceso SSH directo como `root` cuando no sea estrictamente necesario.
* Utilizar autenticación por clave y restringir el uso de contraseñas.
* Aplicar políticas de contraseñas robustas.
* Implementar rate limiting, fail2ban u otros mecanismos contra intentos repetidos.
* Mantener OpenSSH actualizado.
* Restringir el servicio a redes o usuarios que realmente lo necesiten.

> Esta página documenta una práctica deliberadamente vulnerable y autorizada. Los comandos deben utilizarse únicamente en laboratorios o sistemas donde exista permiso explícito.
