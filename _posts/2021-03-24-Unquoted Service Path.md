---
layout: post
title:  "Vulnerabilidad: Unquoted Service Path"
description: En este post veremos en que consiste la vulnerabilidad Unquoted Service Path, que requísitos tenemos que cumplir para poder explotarla, y como explotarla.

tags: vulnerabilidad
---



# Requisitos

* Disponer de alguna forma de subir archivos , ya sea o bien teniendo una shell (Netcat, Meterpreter, etc...) o bien tener alguna forma de subir ficheros, ya sea RFI o LFI,o como sea que hayais vulnerado la máquina en la que quereís escalar privilegios.
* Si la shell es de un usuario sin privilegios,nos bastaría con poder subir (si no se pueden subir ficheros, pero si crearlos, podría funcionar si somos capaces de crear un ejecutable completamente funcional, sin necesidad de ejecutarlo)
* Tener en la máquina que estamos atacando un servicio el cual ejecuta un fichero. Este fichero puede ser, o bien ejecutado cada cierto tiempo ,o bien un proceso/servicio que se active con cierto evento/trigger.(EJ: Iniciar sesión.)





# Explicación de la vulnerabilidad con pseudocódigo

En este fragmento de código podemos ver como, la función proceso ( que simula un servicio que se ejecuta periodicamente ) ejecuta ruta:
```python
def servicio():  
  ejecutar("C:\CARPETA UNO\CARPETA DOS\CARPETA TRES\sql.exe") 
  #El proceso quiere ejecutar la 
  #ruta "C:\CARPETA UNO\CARPETA DOS\CARPETA TRES\sql.exe" (con comillas)
```

En este caso, al estar la ruta con comillas, no importa que tenga espacios, puesto que es una String.
Por tanto el servicio que se ejecuta periodicamente ejecutara sin problemas "C:\CARPETA UNO\CARPETA DOS\CARPETA TRES\sql.exe".



```python
def servicio():  
  ejecutar(C:\CARPETA UNO\CARPETA DOS\CARPETA TRES\sql.exe) 
  #El proceso quiere ejecutar la 
  #ruta C:\CARPETA UNO\CARPETA DOS\CARPETA TRES\sql.exe (sin comillas)
```
En este otro fragmento de código podemos ver como la función servicio ejecuta la ruta `C:\CARPETA UNO\CARPETA DOS\CARPETA TRES\sql.exe` , (vease que no lleva comillas)



# ¿Cómo ejecutará esto Windows?

Buscará subdirectorio por subdirectorio hasta dar con el fichero que busca(sql.exe). Es decir, que si encuentra el fichero antes, usará ese fichero y dejará de buscarlo.
Pero hay que analizar muy bien la ruta , puesto que , al ser una vulnerabilidad(ERROR) , las rutas que ira comprobando no son muy normales, o intuitivas.


Por tanto, si la ruta del ejecutable que se esta ejecutando es: `C:\CARPETA UNO\CARPETA DOS\CARPETA TRES\sql.exe` , será recorrida de la siguiente manera:
```ps1
"C:\PRIMERA.exe"
"C:\PRIMERA CARPETA/segunda.exe"
"C:\PRIMERA CARPETA/SEGUNDA CARPETA/tercera.exe"
"C:\PRIMERA CARPETA/SEGUNDA CARPETA/TERCERA CARPETA/sql.exe"
```



# <i class="fas fa-bomb"></i> Explotación de la vulnerabilidad

Por tanto, visto lo anteriormente , y si cumplimos las condiciones previamente citadas, si colocamos por ejemplo un payload `segunda.exe` hecho en powershell/cmd en una ruta previa a la ruta final , vease:

```ps1
"C:\PRIMERA CARPETA/segunda.exe"
```

...siendo `segunda.exe` nuestro payload/reverse shell o lo que queraís ejecutar como administrador,
conseguiremos una reverse shell (en este ejemplo) como administrador (o el usuario que este ejecutando el servicio/tarea).



#  <i class="fas fa-bomb"></i> Creacion de un payload en Msfvenom

La creación del payload y del listener pueden hacerse de manera manual, es recomendable hacerlo aunque sea más díficil si se quiere aprender, pero aqui usaremos `metasploit` y `msfvenom` para facilitar que se entienda el proceso.



<a href="https://ibb.co/QnJWCjJ"><img src="https://i.ibb.co/d6fNLJf/ups-msfvenom.png" alt="ups-msfvenom" border="0" /></a>


# <i class="fas fa-headphones-alt"></i> Poniendonos a la escucha con Metasploit
<a href="https://ibb.co/kgH8tFy"><img src="https://i.ibb.co/XZ2sqT4/MSFCONSOLE-ESCUCHAR.png" alt="MSFCONSOLE-ESCUCHAR" border="0" /></a>
<a href="https://ibb.co/0yh4qJj"><img src="https://i.ibb.co/xqJ9h5S/EXPLOIT.png" alt="EXPLOIT" border="0" /></a>




# Creación de esta vulnerabilidad para testear en entornos controlados...
Si queremos probar esta vulnerabilidad,podemos crear un servido llamado ``SERVICIOVULNERABLE``

```ps1
sc create SERVICIOVULNERABLE start= auto binPath= "C:\PRIMERA CARPETA\SEGUNDA CARPETA\TERCERA CARPETA\sql.exe" DisplayName= SERVICIOVULNERABLE
```
<a href="https://ibb.co/8rvBh6B"><img src="https://i.ibb.co/m9ZBx6B/image.png" alt="image" border="0" /></a>

NOTA: Las comillas son necesaria para el uso del comando, pero la creación se esta llevando a cabo sin comillas, se puede comprobar con el siguiente comando,el cúal nos mostrara todos los procesos con rutas sin entrecomillar:

```ps1
wmic service get name,displayname,pathname,startmode |findstr /i /v “C:\Windows\\” | findstr /i /v “””
```
<a href="https://ibb.co/r2ZF9gG"><img src="https://i.ibb.co/4RjKkcZ/image.png" alt="image" border="0" /></a>
Para simular que se ejecuta de manera periodica podemos simplemente encender , parar y reiniciar,respectivamente,el servicio :

```ps1
net start SERVICIOVULNERABLE
net stop SERVICIOVULNERABLE
net restart SERVICIOVULNERABLE
```
# <i class="fas fa-shield-alt"></i> ¿Cómo protegernos de esta vulnerabilidad?

Para protegernos de esta vulnerabilidad,simplemente tenemos que añadir comillas a nuestra rutas.
Al crear servicios, si queremos que estos sean seguros, hay que jugar con los `slash` para introducir comillas dentro de las comillas del propio comando,por ejemplo:
```ps1
sc create ServicioNoVulnerable start= auto binPath= "\"RUTA\DEL\SERVICIO/EJECUTABLE"\" DisplayName= ServicioNoVulnerable
```



# Agradecimientos
[Sumit Verma con su post "Windows Privilege Escalation — Part 1 (Unquoted Service Path)"](https://medium.com/@SumitVerma101/windows-privilege-escalation-part-1-unquoted-service-path-c7a011a8d8ae) 

[@EvilOrez con su Post "MICROSOFT’S UNQUOTED SERVICES"](https://hackmd.io/@EvilOrez/unquotedservices) 


