---
layout: post
title:  "TryHackMe Overpass 2 - Wireshark"
description: En este post aprenderemos a movernos de manera básica por Wireshark y a filtrar los paquetes.

tags: tryhackme, wireshark
---


# Máquina TryHackMe: Overpass 2

Máquina de la plataforma de TryHackMe que trata sobre análisis de paquetes.

Aprenderemos: 
* A movernos por Wireshark.
* A leer Wireshark de una forma eficiente.
* A aplicar filtros a Wireshark.


# ¿Cúal era la URL de la página que se uso para subir la reverse shell?

Para responder a esta pregunta lo primero que tenemos que entender es que para subir y bajar ficheros se usa el protocolo HTTP, con sus distintos métodos.
Por eso, para responder a esta cuestión, filtraremos todos los paquetes por el protocolo HTTP, de la siguiente manera:


<a href="https://ibb.co/9qdsLsv"><img src="https://i.ibb.co/BgYqxqP/HTTP.png" alt="HTTP" border="0" /></a>


Así conseguiremos reducir drastricamente el numero de paquetes a revisar . ( De 3896 a 15 paquetes, eso es ahorrar muchisimo tiempo, analizaremos solo un 0,3 % de los paquetes totales del fichero .pcap)

Con estos paquetes, iremos uno por uno siguiendo la trama TCP, para analizarlo más en profundidad.

Si nos fijamos, en la pestaña "INFO" de cada paquete, nos pone que se ha realizado una peitición HTTP post a /upload.php, lo cual nos da a entender que se ha subido un fichero, lo cual es interesante, asi que lo analizaremos siguiendo la trama TCP...

<a href="https://ibb.co/9qdsLsv"><img src="https://i.ibb.co/BgYqxqP/HTTP.png" alt="HTTP" border="0" /></a>

Aqui es donde descubriremos la segunda respuesta: El payload.

Seguiremos la trama TCP y nos daremos cuenta de que ahí se encontrará el payload.

<a href="https://ibb.co/LN7wjvB"><img src="https://i.ibb.co/J3JL6C8/payload.png" alt="payload" border="0" /></a>

# Q2: ¿Qué contraseña uso el atacante para conseguir escalar privilegios?

Aquí, tras haber leído todas las peticiones HTTP previamente filtrada, me puse a pensar... 

¿Cómo habra realizado la escalada de privilegios, si usó una reverse shell con PHP?

Y llegué a la conclusión de que la habría conseguido mediante NETCAT (aka nc). 

Por tanto me puse a investigar los paquetes TCP, el problema es que había muchisimos. Y otra vez tocaba pensar...¿Cómo hago un vistazo rápido, que estoy buscando? Y la respuesta fue: La escalada de privilegios, son comandos, mucho texto, datos, data... Basicamente estabamos buscando una trama "grande", con muchos datos, comandos, input(comandos) y output(salida de esos comandos)...

Asi que simplemente me puse a ver las peticiones las cuales su lenght era superior a la media (que rondaba entre 60-70, las cuales son muchísimas, por tanto ordené los paquetes según su length.

Tras apenas un minuto siguiendo las tramas de estas peticiones, llegue a una que contenía un LENGTH de 121. Lo que me llamo la atención de esta petición no fue el LENGTH , fue que contenía "DATA".

<a href="https://ibb.co/h25HBTP"><img src="https://i.ibb.co/7rcVn0w/image.png" alt="image" border="0" /></a>

Me puse a leerlo (``Follow TCP Stream, Seguir trama TCP...``), y lo que contenía resultó ser todo lo que pasaba por NC (netcat). Ahí se encontraba varias respuestas sobre los hashes, simplemente hay que leerlo y encuentras tanto el hash como la salt.

<a href="https://ibb.co/kGBt4FV"><img src="https://i.ibb.co/Prjdc8s/image.png" alt="image" border="0" /></a>


# ¿Cómo crackear la contraseña hasheada?

Antes de seguir adelante hay que entender que un hash tiene salt.
Por tanto,crackear el "hash" de la pregunta de ``THM`` que contestamos antes, darias error. Esto es debido a que el formato requerido para crackear es "HASH:SALT".

<a href="https://imgbb.com/"><img src="https://i.ibb.co/SB6n98N/hashwithsalt.png" alt="hashwithsalt" border="0" /></a>

Cuando lo tengamos guardado en un fichero ``P.EJ: hash`` deberemos lanzar alguna herramientas para crackearla.
En mi caso usé hashcat con la wordlist recomendada por ´´THM´´ : fasttrack.

<a href="https://ibb.co/V2YnghZ"><img src="https://i.ibb.co/pJjVfs8/image.png" alt="image" border="0" /></a>


# Conseguir la flag User.txt

Primero que todo realize un escaneo de puertos con nmap. 
El escaneo de nmap arrojó como resultado, entre otro muchos, el puerto 22 abierto, por tanto , al tener ya alguna contraseña.

Así que lo primero que se me ocurrió fue lanzar un ataque de fuerza bruta por SSH con hydra.
En los usuarios puse todos los del fichero shadow que encontramos en el fichero Wireshark, 


# Conseguir la flag root.txt

Al entrar como user , comprobe "ls -la"

Encontre el fichero ".suid_bash"

Intente leerlo pero era ilegible, pero si nos penemos a pensar en la situación,el atacante puede haber dejado algun archivo para escalar privilegios facilmente, (se sobreentiende por el nombre del fichero y el contexto de la maquina de TryHackMe)

Ejecutamos el ficheros, comprobamos permisos, y vemos que nada ha pasado , seguimos teniendo el euid de James ( se comprueba con el comando ``id``), aunque el prompt cambio. Esto nos confirma que el script está jugando con la shell y los permisos...

<a href="https://ibb.co/0hZ3gf5"><img src="https://i.ibb.co/SR0phKS/suid-james.png" alt="suid-james" border="0" /></a>

Por suerte se me ocurrió probar con la flag -p (para mantener los permisos de ejecuccion del script) y funcionó.

<a href="https://ibb.co/27sVtFB"><img src="https://i.ibb.co/VWQnSL8/image.png" alt="image" border="0" /></a>

Como vemos se nos mantuvo el euid=0, que corresponde a root, aunque seguimos siendo james, tenemos permisos de root.

Por tanto ya solo nos queda sacar la flag de root:

<a href="https://imgbb.com/"><img src="https://i.ibb.co/XjB9sdp/image.png" alt="image" border="0" /></a>



# Agradecimientos
[John Hammond (TryHackMe! Overpass 2 Recovering from THE HACK)"](https://www.youtube.com/watch?v=XtySdRYCbiY) 



