---
layout: post
title:  "Ataque Active Directory: Envenenamiento del tráfico & servicio 'LLMNR' y como defendernos de estos ataques."
description: En este post entenderemos en que consiste el ataque LLMNR y como hacer uso de el, envenenando el tráfico con la herramienta "responder". También aprenderemos a como defendernos.

tags: PENTESTING, AD
---


[Vídeo en mi canal de YT explicando como escuchar & envenenar,obtener hashes & crackear hashes.](https://www.youtube.com/watch?v=s5KtUur8Mzs&t=7s) 

# ¿En que consiste el ataque y la herramienta "RESPONDER" ?

Cuando un equipo Windows no es capaz de "resolver" `(encontrar)` otro equipo usando DNS... usá el protocolo LLMNR protocol (Link-Local Multicast Name Resolution) para preguntar a los equipos cercanos si alguien sabe "resolverlo" o llegar a el...                                                  

Cuando se usa LLMNR/NBT-NS para "resolver" un nombre, cualquier equipo en la red puede responder.


Aqui es donde entra el `"responder"`... una herramienta que envenena el tráfico.

<a href="https://ibb.co/FzXbX1y"><img src="https://i.ibb.co/w47N7qP/image.png" alt="image" border="0" /></a>



# ¿Cómo llevarlo a cabo?

Aclaración: Este ejemplo se va a llevar mediante una red de active directory local montada con máquinas virtuales.

Primero necesitamos , al menos, un Windows Server y un Windows Cliente (ya sea W7,8,10.., en este caso un W10).

Luego, como atacantes en la red, pondremos el responder a capturar cuanto antes. Es una especie de MITM explotando DNS.


<a href="https://imgbb.com/"><img src="https://i.ibb.co/D4JpQK7/image.png" alt="image" border="0" /></a>

* Con `-i` indicamos la interfaz de red en la que queremos escuchar.
* Con `-d` indicamos que haga peticiones al dominio usando nombres de NETBIOS
* Con `-w` indicamos que se inicie el servidor WPAD 'rogue proxy server'.


Como vemos nos empezará a llegar mucho tráfico , generalmente sin importancia. Este tráfico será envenenado por nuestra herramienta "responder".

<a href="https://ibb.co/0Vvmxzt"><img src="https://i.ibb.co/S7G5zZr/image.png" alt="image" border="0" /></a>

De tal modo que si desde cualquier equipo que esté dentro de la red en la que estamos escuchando alguien intenta acceder a algo que "supuestamente" no existe o el DNS no logra resolver, el <b>"RESPONDER" envenenará el tráfico </b> de tal forma que creeran que si existe y por tanto tendrán que introducir las credenciales como se ve en esta captura, intentando acceder a un supuesto share llamado "produccion".

<b>Así se accedería normalmente.</b>

<a href="https://imgbb.com/"><img src="https://i.ibb.co/QPD5KMg/image.png" alt="image" border="0" /></a>

<b>Normalmente, sin la intervención del atacante, al buscar algo que no existe, saldría esto:</b>

<a href="https://imgbb.com/"><img src="https://i.ibb.co/LvGLNsY/image.png" alt="image" border="0" /></a>

<b>Pero como hay un atacante escuchando y "envenenando" el tráfico, el equipo de la víctima pensará que si existe y pedirá credenciales. Estás credenciales son las que nos llegarán.</b>

<a href="https://imgbb.com/"><img src="https://i.ibb.co/QF8h7Zj/image.png" alt="image" border="0" /></a>

<b>Y desde la máquina del atacante...</b>

<a href="https://ibb.co/KKXpVkH"><img src="https://i.ibb.co/sW2dsGr/image.png" alt="image" border="0" /></a>

<b>Con un hash ya se puede hacer mucho pero encima estos hashes se pueden crackear....</b>

<a href="https://ibb.co/Kxfkr0r"><img src="https://i.ibb.co/nMYvLgL/image.png" alt="image" border="0" /></a>


<b>Y ya con unas credenciales, ya sean de ADMIN o no, podemos intentar multitud de otros escaneos.</b>
 

# Como defenderse de esto.

* La forma más fácil es seleccionar crear una GPO que deshabilite el servicio LLMNR (Configuración del equipo>Directivas>Plantillas administrativas de Windows>Red>Cliente DNS>...) Vease la imágen del final.
* Si nuestra empresa necesita, por lo que sea, este servicio, lo único que podemos hacer es cercionarnos que todas las contraseñas de la empresa son complejas. De esta manera, por mucho que roben los hashes, no podrán crackearla ( al menos con fuerza bruta )
* Que una contraseña sea compleja quiere decir:
    

--Maýusculas 


--Minúsculas


--Símbolos

--Y lo más importante: Que no sea común. Todo lo anterior da igual si la contraseña es común. Vease,de ejemplo, mi video en Youtube, que la contraseña era "password_123" (con simbolos y números) pero al ser una contraseña común que esta en las listas de cracking, pudimos crackearla.


<a href="https://ibb.co/GWz2hfp"><img src="https://i.ibb.co/gSkTnb9/image.png" alt="image" border="0" /></a>
