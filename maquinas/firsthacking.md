# FirstHacking

**Plataforma:** DockerLabs\
**Dificultad:** Muy fácil\
**Área:** FTP, enumeración de servicios y vulnerabilidades conocidas\
**Material de referencia:** [Writeup utilizado en clase](https://github.com/ian16munoz3nunez1/DockerLabs/blob/main/firsthacking/firsthacking.md)

## Qué trabajamos con esta máquina

FirstHacking permite enseñar un flujo muy claro: identificar un servicio expuesto, obtener su versión, relacionarla con una vulnerabilidad conocida, validar la condición y comprobar el impacto dentro de un entorno controlado.

## Reconocimiento

El writeup de referencia parte comprobando conectividad y después realiza un escaneo de servicios:

```bash
ping -c4 172.17.0.2
nmap -sV -p- -T4 172.17.0.2
```

El hallazgo principal es el **puerto 21/TCP**, donde se identifica **vsftpd 2.3.4**.

## Investigación de vulnerabilidades

A continuación se utilizan scripts de Nmap para revisar vulnerabilidades conocidas asociadas al servicio:

```bash
nmap --script vuln -p 21 172.17.0.2
```

La versión identificada se relaciona con **CVE-2011-2523**, una backdoor conocida asociada a una versión comprometida de vsftpd 2.3.4.

## Comprender la vulnerabilidad

El punto importante de la práctica no es memorizar un payload, sino entender la cadena:

1. detectamos FTP;
2. obtenemos la versión;
3. investigamos si esa versión tiene antecedentes de seguridad;
4. identificamos la condición que activa la backdoor;
5. verificamos si aparece un nuevo servicio;
6. comprobamos el nivel de acceso obtenido.

En el laboratorio, la activación de la backdoor provoca la apertura del **puerto 6200**.

## Validación controlada

Una vez activo el puerto, el writeup utiliza `netcat` para conectarse y confirmar la ejecución remota:

```bash
nc 172.17.0.2 6200
whoami
id
hostname
```

El resultado demuestra el impacto de una vulnerabilidad de servicio que entrega acceso privilegiado sin autenticación normal.

## Aprendizajes principales

* La versión de un servicio puede ser una pieza crítica de información.
* Un CVE debe investigarse y entenderse antes de intentar validarlo.
* Un puerto nuevo después de una acción concreta puede ser evidencia de cambio de estado en el objetivo.
* Una vulnerabilidad conocida puede tener impacto crítico incluso si el servicio parece simple.
* El parcheo y la gestión de versiones son controles defensivos fundamentales.

## Mitigación

* Eliminar versiones comprometidas o vulnerables de vsftpd.
* Utilizar versiones mantenidas y verificadas desde fuentes confiables.
* Deshabilitar servicios FTP que no sean necesarios.
* Restringir exposición mediante firewall y segmentación.
* Supervisar la aparición inesperada de puertos o procesos.
* Mantener inventario y gestión de versiones de servicios.

> Esta práctica se realiza exclusivamente sobre un laboratorio vulnerable y autorizado.
