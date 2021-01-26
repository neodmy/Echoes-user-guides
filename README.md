# Distribución para estación de radiodetección del proyecto Contadores de Estrellas

## Resumen

El objetivo de este documento es proporcionar una guía de usuario para facilitar el proceso de instalación del software necesario para establecer una estación de radiodetección basada en Echoes, un programa de análisis de espectro mediante dispositivos RTL-SDR. Además de este programa, es necesario disponer de otros programas y servicios adicionales para garantizar el buen funcionamiento de este, así como la recuperación y envío de la información generada por Echoes.


Esta guía está sujeta a revisión, por lo que, si encuentra algún problema en alguna de las instrucciones facilitadas, póngase en contacto con el autor de esta. Así mismo, si ha sido capaz de encontrar la solución a un problema durante este proceso, le ruego lo comparta con la comunidad para completar esta guía y facilitar el resto de usuario un proceso más sencillo y completo.

Puede contactarme en el siguiente correo electrónico: davidmiguelyusta@gmail.com.

Gracias de antemano.

---

## Table de contenido

1. [Drivers RTL-SDR](#drivers-rtl-sdr)
2. [Echoes](#echoes)
3. [Cliente NTP](#cliente-ntp)
4. [Echoes-watcher](#echoes-watcher)


### Drivers RTL-SDR


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

El protocolo NTP (Network Time Protocol) es un protocolo utilizado para la sincronización de los relojes de los sistemas informáticos mediante enrutamiento de paquetes en la red de acuerdo con una latencia variante. Es decir, permite indicar de forma precisa la hora a los sistemas independiente de la lentitud de la red.

Primero, se debe instalar el demonio NTP (ntpd):

```bash
$ sudo apt-get install ntp
````

A continuación, debe especificar el nombre del servidor NTP con el que se quiere sincronizar en el fichero de configuración /etc/ntp.conf:

```bash
$ sudo nano /etc/ntp.conf
```

Por defecto, el fichero de configuración dispondrá de una serie de servidores preconfigurados, que en el caso de una distribución con Ubuntu serán los propios servidores de Ubuntu:

```
server 0.ubuntu.pool.ntp.org
server 1.ubuntu.pool.ntp.org
server 2.ubuntu.pool.ntp.org
server 3.ubuntu.pool.ntp.org
```

Si se quiere sincronizar con un servidor en concreto, se debe indicar el nombre del servidor NTP modificando las líneas anteriores:

```
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

```
$ pip2 --version
```

El código de Echoes-watcher se puede encontrar en el repositorio de CSLab:

https://github.com/cslab-upm/Echoes-watcher

Mediante el siguiente comando puede descargar el código fuente a su máquina:

```
$ git clone https://github.com/cslab-upm/Echoes-watcher.git
```

Esto generará una carpeta con el nombre Echoes-watcher en el directorio en el que se encontrara cuando ejecuto dicho comando. Antes de ejecutar este programa es necesario configurar cierta información en el fichero de configuración de Echoes ECHOES_CONFIG_FILE. En el apartado [Site%20infos] existe una información similar a la siguiente:

```
Altitude=125
Contact=trustno1@nowhere.org
Latitude=45.516667
Longitude=-9.583333
Notes=trentatre trentini tornavan da trento tutti e trentatre trotterellando
RxSetup=who knows
StationLogo=:/icon128
StationName=None
```

La variable StationName debe ser configurada con el nombre de la estación.

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