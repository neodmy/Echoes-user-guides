# Distribución para estación de radiodetección del proyecto Contadores de Estrellas

## Resumen

El objetivo de este documento es proporcionar una guía de usuario para facilitar el proceso de instalación del software necesario para establecer una estación de radiodetección basada en Echoes, un programa de análisis de espectro mediante dispositivos RTL-SDR. Además de este programa, es necesario disponer de otros programas y servicios adicionales para garantizar el buen funcionamiento de este, así como la recuperación y envío de la información generada por Echoes.


Esta guía está sujeta a revisión, por lo que, si encuentra algún problema en alguna de las instrucciones facilitadas, póngase en contacto con el autor de esta. Así mismo, si ha sido capaz de encontrar la solución a un problema durante este proceso, le ruego lo comparta con la comunidad para completar esta guía y facilitar el resto de usuario un proceso más sencillo y completo. Puede contactar conmigo en el siguiente correo electrónico: davidmiguelyusta@gmail.com o bien realizando una PR sobre este repositorio.

Para obtener las información necesaria sobre a qué servidor NTP debe sincronizar, así como credenciales y otros parámetros de configuración relativos a la estación, póngase en contacto con los responsables del proyecto Contadores de Estrellas (Raquel Cedazo: rcedazo@ciclope.info)

Por favor, siga el proceso de instalación en el order proporcionado en la [Tabla de Contenido](#tabla-de-contenido)

Gracias.

---

## Tabla de contenido

1. [Drivers RTL-SDR](#drivers-rtl-sdr)
2. [Echoes](#echoes)
3. [Cliente NTP](#cliente-ntp)
4. [Echoes-watcher](#echoes-watcher)
5. [Docker](#docker)
6. [Echoes-backup-client](#echoes-backup-client)
7. [Echoes-monitor](#echoes-monitor)


### Drivers RTL-SDR

Para poder recoger los datos del dispositivo RTL-SDR es necesario en primer lugar instalar los drivers.

Primero, se debe actualizar los paquetes de la distribución:

```bash
$ sudo apt-get update
```

En segundo lugar, se deben instalar las herramientas para recuperar el código fuente, compliar, y hacer la build:

```bash
$ sudo apt-get install git
$ sudo apt-get install cmake
$ sudo apt-get install build-essential
```

A continuación es necesario instalar la librería de C que proporciona accesso genérico a dispositivos USB:

```bash
$ sudo apt-get install libusb-1.0-0-dev
```

Posteriomente, se debe descargar el código fuente y hacer la build:

```bash
$ git clone git://git.osmocom.org/rtl-sdr.git
$ cd rtl-sdr/
$ mkdir build
$ cd build
$ cmake ../ -DINSTALL_UDEV_RULES=ON
$ make
$ sudo make install
$ sudo ldconfig
$ sudo cp ../rtl-sdr.rules /etc/udev/rules.d/
```

A continuación se debe deshabilitar el driver por defecto ya que no funciona para dispositivos SDR y genera interferencias con el driver que acabamos de instalar. En el directorio `/etc/modprobe.d` se debe crear un fichero con el nombre de `blacklist-rtl.conf` y añadir la siguiente línea:

```bash
blacklist dvb_usb_rtl28xxu
```

Por último, para comprobar que funciona correctamente, se debe ejecutar:

```bash
$ rtl_test -t
```

La salida indicará que se está utilizando un dispositivo RTL2832U genérico.

### Echoes

Echoes es un software de análisis de espectro mediante dispositivos RTL-SDR, diseñado para analizar y registrar la potencia total de las señales generando imágenes y datos en formato de tabla ante la presencia de determinados picos en un rango de frecuencias determinados.

Echoes puede ser configurado con un rango de frecuencias específico, y en diferentes modos de operación:
-	Continúo: captura de datos únicamente.
-	Periódico: captura de datos e imágenes cada X segundos.
-	Automático: captura de datos e imágenes cuando se ha excedido cierto umbral configurable.

Durante su funcionamiento habitual, Echoes genera una serie de ficheros con los rangos de frecuencias configuradas.

Los enlaces para descargar Echoes se pueden encontrar en:

https://sourceforge.net/projects/echoes/files/Linux%20binaries

En este enlace existen diferentes versiones para cada sistema operativo. Esta guía se centra en sistemas operativos basados en Linux, concretamente la distribución Ubuntu, y Raspbian, en el caso en que la estación se establezca utilizando una Raspberry Pi.

-	Versión para Ubuntu: https://sourceforge.net/projects/echoes/files/Linux%20binaries/ubuntu
-	 Versión para Raspbian: https://sourceforge.net/projects/echoes/files/Linux%20binaries/raspbian

Para la versión de Ubuntu, elija la correspondiente a la versión de su distribución, que puede consultar mediante el siguiente comando:

```bash
$ lsb_release -a
```

Para la versión de Raspbian es necesario incluir los siguientes ficheros en un mismo directorio:

-	libqt5charts5_5.7.1-3_armhf.deb 
-	qt-everywhere-opensource-rpi_5.9.4_armhf.deb
-	echoes_0.25-1_armhf.deb

A continuación, ejecute:

```bash
$ sudo apt-get update
$ sudo apt-get upgrade
```

Una vez realizado los pasos anteriores, ejecute los ficheros en el orden anterior. Tras la finalización del proceso, reinicie el sistema y lance Echoes desde el menú Other en el menú principal.

Si Echoes no se inicia, ejecute los dos primeros ficheros invirtiendo el orden, reinicie y vuelva a lanzar Echoes.

El manual de usuario de Echoes se puede encontrar en el siguiente enlace:

https://sourceforge.net/projects/echoes/files/documentation/echoes-manual.pdf/download

### Cliente NTP

El protocolo NTP (Network Time Protocol) es un protocolo utilizado para la sincronización de los relojes de los sistemas informáticos mediante enrutamiento de paquetes en la red de acuerdo con una latencia variable. Es decir, permite indicar de forma precisa la hora a los sistemas independiente de la lentitud de la red.

Primero, se debe instalar el demonio NTP (ntpd):

```bash
$ sudo apt-get install ntp
```

A continuación, debe especificar el nombre del servidor NTP con el que se quiere sincronizar en el fichero de configuración /etc/ntp.conf:

```bash
$ sudo nano /etc/ntp.conf
```

Por defecto, el fichero de configuración dispondrá de una serie de servidores preconfigurados, que en el caso de una distribución con Ubuntu serán los propios servidores de Ubuntu:

```bash
server 0.ubuntu.pool.ntp.org
server 1.ubuntu.pool.ntp.org
server 2.ubuntu.pool.ntp.org
server 3.ubuntu.pool.ntp.org
```

Si se quiere sincronizar con un servidor en concreto, se debe indicar el nombre del servidor NTP modificando las líneas anteriores:

```bash
server nombre-del-servidor-NTP-elegido
```

Por último, es necesario reiniciar el demonio NTP mediante el siguiente comando:

```bash
$ sudo service ntp reload
```

Una vez reiniciado el demonio, se puede comprobar la lista de servidores NTP especificados en el fichero de configuración con el siguiente comando:

```bash
$ ntpq -p
```

### Echoes-watcher

Echoes-watcher es un programa desarrollado por Samuel Peres del Instituto de Astrofísica de Canarias. La función principal de este programa es recuperar y transformar los datos recogidos por Echoes, para la posterior generación de sonidos en base a las frecuencias. Estos datos son emitidos a un bróker. Con esta información se generará una señal que será emitida, sirviendo como una “radio del cielo”.

Antes de iniciar el programa, se debe instalar (si no se encuentra ya instalada en el sistema) la versión de Python 2.7. Python 2 viene instalado de serie en distribuciones anteriores a Ubuntu 18.04. Sin embargo, a partir de esta versión, se distribuye por defecto Python 3. Para instalar Python 2, se deben realizar los siguientes pasos:

En primer lugar, habilitar el repositorio universo:

```bash
$ sudo add-apt-repository universe
```

A continuación, actualizar los paquetes e instalar Python 2:

```bash
$ sudo apt update 
$ sudo apt install python2
$ curl https://bootstrap.pypa.io/get-pip.py --output get-pip.py
$ sudo python2 get-pip.py
````

Una vez completados los pasos anteriores, se puede comprobar la versión mediante:

```bash
$ pip2 --version
```

El código de Echoes-watcher se puede encontrar en el repositorio de CSLab:

https://github.com/cslab-upm/Echoes-watcher

Mediante el siguiente comando puede descargar el código fuente a su máquina:

```bash
$ git clone https://github.com/cslab-upm/Echoes-watcher.git
```

Esto generará una carpeta con el nombre Echoes-watcher en el directorio en el que se encontrara cuando ejecuto dicho comando. Antes de ejecutar este programa es necesario configurar cierta información en el fichero de configuración de Echoes ECHOES_CONFIG_FILE. En el apartado [Site%20infos] existe una información similar a la siguiente:

```bash
Altitude=125
Contact=trustno1@nowhere.org
Latitude=45.516667
Longitude=-9.583333
Notes=trentatre trentini tornavan da trento tutti e trentatre trotterellando
RxSetup=who knows
StationLogo=:/icon128
StationName=None
```

La variable `StationName` debe ser configurada con el nombre de la estación.

Una vez configurado el nombre de la variable, se procede a instalar las dependencias de de Echoes-watcher. Para ello, en la carpeta descargada del repositorio, se debe ejecutar:

```bash
$ pip2 install -r requirements.txt
```

Una vez instaladas las dependencias, se puede lanzar el programa con:

```bash
$ nohup /usr/bin/python2.7 watcher.py </dev/null >/dev/null 2>&1 &
```

En el caso de que Echoes no este instalado en el home del usuario, es necesario especificar la ruta al fichero default.rts de Echoes al ejecutar el programa:

```bash
$ nohup /usr/bin/python2.7 watcher.py /path/to/defaults.rts /path/to/working/directory/ </dev/null >/dev/null 2>&1 &
```

### Docker

Para ejecutar las siguiente aplicaciones, es necesario disponer de Docker. A modo de resumen, Docker permite ejecutar aplicaciones en un entorno aislado de la máquina local, además de permitir empaquetar todas las dependencias de estas en un contenedor.

En este enlace se proporciona una guía muy completa para Ubuntu: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04-es

Para instalar Docker en Raspbian puede seguir esta guía: https://blog.desdelinux.net/como-instalar-docker-en-raspberry-pi-con-raspbian/

Una vez se disponga de Docker, se puede proceder a instalar el resto de aplicaciones de la distribución que se exponen a continuación.

### Echoes-backup-client

Este servicio se dedica a realizar copias de seguridad de los ficheros generados por Echoes con las detecciones diarias. El servicio se ejecuta una vez al día y enviará a un servidor SFTP del proyecto Contadores de Estrellas los ficheros necesarios para ser procesado. 

En primer lugar, debe descargar el código fuente mediante este comando:

```bash
git clone https://github.com/neodmy/echoes-backup-client.git
```


La aplicación debe ser configurada en el fichero `docker-compose.yml`, que se encuentra en la raíz del proyecto. Los parámetros de configuración son los siguientes:

- Slack: mediante los siguientes parámetros (opcionales) el usuario puede establecer el canal de Slack en el que se proporcionarán alertas sobre fallos durante el procesamiento diario.
  - `SLACK_TOKEN`: token necesario para la autenticación y autorización en el espacio de trabajo de Slack.
  - `SLACK_CHANNEL`: identificador del canal de Slack en el que se reportan las incidencias.
- SFTP: credenciales necesarias para que el cliente pueda realizar el envío de las copias de seguridad.
  - `SFTP_HOSTNAME`: nombre del servidor SFTP.
  - `SFTP_PORT`: puerto en el que el servidor SFTP escucha.
  - `SFTP_USERNAME`: nombre del usuario.
  - `SFTP_PASSWORD`: contraseña del usuario.
- Directorio remoto: ruta completa en el directorio remoto en el que el cliente va a depositar los ficheros a copiar
  - `REMOTE_DIRECTORY`: ruta completa al directorio remoto.
- Parámetros de cliente: estos parámetros incluyen información relativa a la estación:
  - `CLIENT_ID`: nombre de la estación. Este parámetro se añadirá a la ruta del directorio remoto para asegurar que cada estación deposita sus ficheros en una ruta propia y no interfiere con los ficheros enviados por otra estación. 
  - `REMOVAL_OFFSET`: número de días que los directorios diarios generados por Echoes permanecen en el almacenamiento local.
- Base de datos: parámetros de conexión a la base de datos:
  - `MONGO_CONNECTION_STRING`: parámetro de conexión a la base de datos con la que interactúa el cliente: `<protocolo>://<nombre_base_datos>:<puerto>`.
- Email: además de la alternativa de Slack, el cliente ofrece la posibilidad (de manera opcional) de reportar las alertas mediante correo electrónico. Lo ideal es crear una cuenta de correo electrónico para cada estación, ya que este será el remitente de los correos electrónicos:
  - `SENDER_SERVICE`: servidor SMTP de la cuenta que se utiliza para reportar alertas. Puede ser cualquier servicio comercial que permita el envío de correos electrónicos de manera programática como, por ejemplo, Google.
  - `SENDER_USER`: usuario de la cuenta de correo electrónico que se utiliza para reportar alertas.
  - `SENDER_PASSWORD`:  contraseña de la cuenta de usuario de correo electrónico que se utiliza para reportar alertas.
  - `REPORT_EMAILS`: direcciones de correo electrónico a las que se va a reportar las alertas. El formato es una cadena de texto con un conjunto de emails separados por comas: email1@email.com,email2@email.com.
- Inicialización: puesto que el servicio realiza una sincronización inicial del contenido del directorio donde Echoes genera los directorios diarios, se proporciona la posibilidad de desactivar esta funcionalidad mediante:
  - `INITIAL_CSV`: una cadena de texto con el valor active en el caso de que el usuario desee realizar el envío de la línea del fichero CSV que representa el día a procesar, teniendo en cuenta el contenido del directorio.
  - `INITIAL_UPLOAD`: una cadena de texto con el valor active en el caso de que el usuario desee realizar el envío del directorio diario.
- Cron: este parámetro permite establecer la expresión Cron para determinar el momento en el que se realice el procesado.
  - `CRON_SCHEDULE`: Dado que el procesamiento debe ser diario (debido al comportamiento de Echoes), solo se debe modificar los tres primeros valores, que representan el segundo, minuto y hora (de izquierda a derecha) en la que se debe realizar dicho procesamiento. Por ejemplo, la expresión 0 0 8 * * * indica todos los días de la semana a las 8:00 (hay que tener en cuenta la hora local de la máquina en la que se ejecuta el servicio y las diferencias horarias. Esta hora es GMT).


Por último, y aunque no es una variable de entorno, pero dado que el servicio se ejecutará en un contenedor Docker, es necesario disponer de un volumen para que la aplicación que ejecuta dentro del contenedor pueda acceder al sistema de ficheros local de la máquina en la que se ejecuta. Esto se puede hacer a través de los host volume, una funcionalidad que proporciona Docker para realizar un mapping del siguiente modo (modificando el fichero `docker-compose.yml`:

```yml
volumes:
  - /path/a/directorio/local:/echoes
```

Mediante esta construcción, cada vez que la aplicación acceda a la ruta /echoes, estará accediendo a la ruta /home/user del sistema de ficheros local. En este caso, el valor a proporcionar será la ruta absoluta al directorio dónde Echoes genera los directorios diarios.

Para lanzar la aplicación basta con ejecutar el siguiente comando **desde la raíz del proyecto**:

```bash
$ docker-compose --file docker-compose.yml --project-name echoes-backup-client up -d --force-recreate --build
```

o bien con la ruta absoluta al fichero docker-compose.yml que se encuentra en la raíz del proyecto:

```bash
$ docker-compose --file ruta/absoluta/a/fichero/docker-compose.yml --project-name echoes-backup-client up -d --force-recreate --build
```


### Echoes-monitor

Esta aplicación analiza a intervalos regulares el directorio de Echoes indicado en la configuración. Una falta de actividad de Echoes puede deberse a fallos en este, por lo que es recomendable, cada vez que se reciba un aviso de esta aplicación revisar el estado de Echoes.

El código fuente se puede descargar ejecutando:

```bash
git clone https://github.com/neodmy/local-echoesmonitor.git
```

En el fichero `src/config.json` se establece la configuración de la aplicación mediante los siguientes parámetros:

- SenderInfo:
  - `service`: servidor SMTP de la cuenta que se utiliza para reportar alertas. Puede ser cualquier servicio comercial que permita el envío de correos electrónicos de manera programática como, por ejemplo, Google.
  - `user`: usuario de la cuenta de correo electrónico que se utiliza para reportar alertas.
  - `pass`:  contraseña de la cuenta de usuario de correo electrónico que se utiliza para reportar alertas.

- ReportEmails:
  - `reportEmails`: direcciones de correo electrónico a las que se va a reportar las alertas. El formato es una cadena de texto con un conjunto de emails separados por comas: email1@email.com,email2@email.com.

- CheckPeriod: estas variables permiten establecer cada cuando tiempo se ejecutará la revisión del estado de Echoes
  - `unit`: se aceptan dos valores: `minute` o `hour`
  - `value`: un número entero
Mediante estas dos variables, se puede configurar un comportamiento dinámico basado en horas o minutos, de manera que si se configura con:

```js
unit: "minute"
value: 1
```
La comprobación se realizará cada minuto. Básicamente, ambas variables permiten generar una expresión cron, que para el caso anterior sería `*/value * * * *`

- DirectoryPath:
  - `directoryPath`: esta variable es la ruta completa al directorio a comprobar de manera periódica, y debería ser configurada con los mismos valores que se incluyan en el fichero `docker-compose.yml` bajo la propiedad `volumes` (similar a lo comentado para `Echoes-backup-client`):

```yml
volumes:
  - /path/a/comprobar:/path/a/comprobar
```

La aplicación se debe lanzar ejecutando el siguiente comando en la ráiz del proyecto:

```bash
docker-compose up
```
