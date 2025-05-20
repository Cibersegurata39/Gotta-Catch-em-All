# Gotta-Catch-em-All

Máquina resuelta de *TryHackMe* en la que se trabaja la enumeración y *fingerprinting*, la conexión SSH, el descifrado de contraseñas y la escalada de privilegios.

<div>
  <img src="https://img.shields.io/badge/-Kali-5e8ca8?style=for-the-badge&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/-Nmap-6933FF?style=for-the-badge&logo=nmap&logoColor=white" />
  <img src="https://img.shields.io/badge/-Dirsearch-005571?style=for-the-badge&logo=dirsearch&logoColor=white" />
  <img src="https://img.shields.io/badge/-Gobuster-3CBDB1?style=for-the-badge&logo=gobuster&logoColor=white" />
 <img src="https://img.shields.io/badge/-ssh-231F20?style=for-the-badge&logo=ssh&logoColor=white" />
  <img src="https://img.shields.io/badge/-php-777BB4?style=for-the-badge&logo=php&logoColor=white" />
  <img src="https://img.shields.io/badge/-Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white" />
  <img src="https://img.shields.io/badge/-python-3776AB?style=for-the-badge&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/-unzip-000000?style=for-the-badge&logo=unzip&logoColor=white" />
</div>

## Objetivo

Explicar la realización del siguiente _Capture the flag_ perteneciente a la plataforma *TryHackMe*. Esta máquina está basada en la serie animada *Pokemon*, en la cual se deberán intentar encontrar diferentes *Pokemons* en forma de *flags*. Para ello, se deberá penetrar en la máquina por medio de *SSH* y realizar una escalada de privilegios.

## Que hemos aprendido?

- Realizar *fingerprinting* y enumeración de puertos y enumeración web.
- Realizar una conxexión *SSH*.
- Crear un servidor con *python3*.
- Descomprimir archivos.
- Descifrar contraseñas.
- Buscar archivos por comando.
- Escalada de privilegios.

## Herramientas utilizadas

- *Kali Linux*.
- Enumeración: *Nmap*, *Dirsearch*, *Gobuster*.
- Penetración: *SSH*, *Bash*, *Python3*, *Unzip*, diferentes decodificadores web. 

## Steps

### Enumeración y fingerprinting

La máquina a vulnerar pertenece a la plataforma *TryHackMe*, la propia web te proporciona la IP de la máquina víctima. La conexión a esta se hace mediante una VPN que te proporciona THM y asigna una nueva IP para que tu máquina interactúe con la máquina vulnerable.

El primer paso es lanzar **Nmap** sin *ping* (-Pn) y sin resolución DNS inversa (-n) para agilizar el escaneo y ser más silencioso. Además, se quieren conocer las versiones que corren en cada puerto (-sV) y el sistema operativo (-O). No se indica, específicamente, que compruebe todos los puertos, pues en este tipo de máquina no será necesario. Por último, se lanzan los *scripts* por *default* de *nmap* (-sC) para que encuentren posibles vulnerabilidades.

<code>nmap -n -Pn -sV -O 10.10.226.58 -sC</code>

![Captura de pantalla 2025-05-15 171616](https://github.com/user-attachments/assets/e2ad0251-ddf8-4c77-a94c-cc004aea621b)

El comando devuelve 2 puertos TCPs abiertos:

- En el puerto 22 corre la versión Openssh 7.2p2, en un sistema Ubuntu, servicio SSH.
- En el puerto 80 corre el servidor Apache 2.4.18, en un sistema Ubuntu, servicio HTTP.

No ha sido capaz de detectar el sistema operativo y los *scripts* no han devuelto ninguna vulnerabilidad.

Así pues, primero nos dirigimos desde el navegador al puerto 80 de la IP dada y esta nos lleva a la página por defecto del servidor de *Apache*. Al analizar el código fuente de la página, se encuentra un comentario donde se incita a interactuar con la consola para obtener información. En otra parte del código se encuentra una lista de *pokemons*, la cual aparece en la consola a través de la variable 'original'.

![Captura de pantalla 2025-05-15 172413](https://github.com/user-attachments/assets/cfe2840a-818a-4b94-b058-eb2a56c9b651)

Lo siguiente es hacer una enumeración web con **Dirsearch** para encontrar posibles directorios y archivos.  Por la parte de directorios, únicamente aparece '/server-status' el cual no es alcanzable. *Dirsearch* es lanzado con **Python3** indicando la IP a atacar (-u) y la lista a utilizar para la enumeración (-w); en este caso la 'raft-small-directories.txt'.

![Captura de pantalla 2025-05-15 175230](https://github.com/user-attachments/assets/02e63ef5-95ec-4349-9d69-63bd7fbb1097)

Por medio de la herramienta **gobuster** han aparecido más directorios, los cuales tampoco se pueden alcanzar. Esta búsqueda se ha especificado en el parámetro 'dir', añadiendo la IP destinto con '-u' y el diccionario 'common.txt' con '-w'.

<code>gobuster dir -u 10.10.226.58 -w /usr/share/wordlists/dirb/common.txt</code>

![Captura de pantalla 2025-05-15 175353](https://github.com/user-attachments/assets/9921205d-45a2-42db-9673-1c6d134568ec)

### Vulnerabilidades explotadas

Volviendo al código de la web, donde estaba el comentario que sugería revisar la consola, se puede observar dos etiquetas dispuestas de tal manera que parecen ser un usuario y contraseña (pokemon:hack_the_pokemon). Por lo que se prueba la conexión *SSH* con estas credenciales con resultado positivo. Una vez dentro, se comprueba el usuario alcanzado (pokemon) y los grupos a los que pertenece.

<code>ssh pokemon@10.10.226.58</code>

![Captura de pantalla 2025-05-15 175648](https://github.com/user-attachments/assets/a6f58264-4d21-4db7-b96e-59fdd71ab669)

Se inspecciona el directorio '/var/www/html', el cual se suele alcanzar al penetrar los servidores *Apache* y encontramos al primero de los *pokemon*, aunque codificado en *ROT* (codificación por rotación, es decir, cifrado por desplazamiento de cada uno de los caracteres N posiciones).

<code>cd /var/www/html</code>

<code>cat water-type.txt</code>

![Captura de pantalla 2025-05-15 180244](https://github.com/user-attachments/assets/a7e20069-00fa-4e05-9cd2-65805a4b295e)

Gracias a un [rot_cypher](https://www.dcode.fr/rot-cipher), se encuentra la *flag* descrifrada que había sido rotada 14 posiciones.

**Flag: Squirtle_SqUaD{Squirtle}**

Analizando el directorio '/home' se averigua que además del usuario *pokemon*, también existe *ash* (esto se puede comprobar accediendo a '/etc/passwd') y un documento '.txt', cuyo contenido no se puede leer. Al entrar en '/pokemon/Desktop', se encuentra el archivo 'P0kEmOn.zip', el cual es descargado montando un servidor con **Python3** en el puerto 8080.

<code>python3 -m http.server 8080</code>

![Captura de pantalla 2025-05-15 181229](https://github.com/user-attachments/assets/631e7a4c-f584-44fe-b503-b93e696c5c67)

Desde la máquina atacante se lanza el comando <code>wget</code> indicando la IP y puerto del servidor creados y el archivo a descargar. Una vez en posesión del *zip*, se descomprime con <code>unzip</code> y aparece el archivo 'P0kEmOn/grass-type.txt' con un nuevo *pokemon*.

<code>wget 10.10.226.58:8080/P0kEmOn.zip</code>

<code>unzip P0kEmOn.zip</code>

<code>cat P0kEmOn/grass-type.txt</code>

Esta nueva *flag* está en código hexadecimal (50 6f 4b 65 4d 6f 4e 7b 42 75 6c 62 61 73 61 75 72 7d), el cual se descifra con [CyberChef](https://gchq.github.io/CyberChef/). 

![Captura de pantalla 2025-05-15 181912](https://github.com/user-attachments/assets/46c7763c-de89-4c58-8aa2-132bfa6ecdc6)

**Flag: PoKeMoN{Bulbasaur}**

Listando de manera recursiva se encuentra un nuevo archivo en la dirección './Videos/Gotta/Catch/Them/ALL!' y se comprueba que es un archivo C. Al leer su contenido aparece el usuario y contraseña 'ash : pikapika'.

<code>ls -R</code>
<code>file Could_this_be_what_Im_looking_forª?.cplusplus</code>

![Captura de pantalla 2025-05-15 182623](https://github.com/user-attachments/assets/7451aac9-2d94-44d1-8e84-b40f609a777f)

Al intentar cambiar de usuario con las credenciales obtenidas, se obtiene acceso a 'ash'. Se comprueba que forma parte del grupo 'sudo' y esto nos permite pivotar hacia *root* también. Ahora se puede acceder a un archivo encontrado previamente en '/home', como es 'roots-pokemon.txt'. En él se encuentra la bandera de *root*.

**Flag: Pikachu!**

![Captura de pantalla 2025-05-15 183352](https://github.com/user-attachments/assets/431f4d7f-b950-4316-a777-e62fe8cdc545)

Para encontrar la última de las *flags*, perteneciente al *pokemon* de tipo fuego, ha sido necesario realizar una búsqueda con <code>find</code>. Basándonos en el nombre del archivo que contenía el *pokemon* de tipo agua, se filtra la búsqueda por archivos que contuvieran 'fire-type' en el nombre. Se obtiene el documento de texto 'fire-type.txt', cuya *flag* está codificada en Base64. Con la ayuda de nuevo de *Cyberchef* se recupera el valor original de la bandera.

<code>find / -type f -name *fire-type* 2>/dev/null</code>

![image](https://github.com/user-attachments/assets/0eeb9ed4-032b-4c91-9c63-401588e8feaf)

**Flag: P0k3m0n{Charmander}**
