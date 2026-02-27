# Máquina Buff

Lanzamos la máquina y seteamos la IP en el target de la polybar como hacemos siempre.

![](<Pasted image 20260220125912.png>)

Lanzamos un ping para ver si tenemos conexión y para tener información acerca del SO, TTL cerca de 64 (Linux) y de 128 (Windows).

![](<Pasted image 20260220130047.png>)

Parece una máquina Windows. Ahora procedemos a lanzar nuestro primer comando nmap para identificar puertos abiertos de una manera óptima.

![](<Pasted image 20260220130300.png>)


Lanzaremos el comando con los scripts para sacar mas información. (Tuve que poner el parámetro -Pn para que no realizase host discovery ya que estaba fallando).

![](<Pasted image 20260220131104.png>)

Vemos que el puerto 7680 ahora aparece filtered, he buscado información y parece que es un socket a no tener en cuenta (concretamente Windows Delivery Optimization, que abre y cierra dinámicamente según la actividad de actualizaciones P2P de Windows).

Parece ser que hostea una página web, vamos a  ver de que se trata.

![](<Pasted image 20260220131846.png>)


Vemos lo que parece la página d eun gimnasio y vemos un login pero sin posibilidad de registro.
Vamos a lanzar un whatweb para ver que corre por detrás.


![](<Pasted image 20260220131725.png>)

Vemos que corre Apache 2.4.43 y PHP 7.4.6. En versiones antiguas de PHP existe RCE, pero no nos sirve en esta.

![](<Pasted image 20260220132305.png>)

Antes de lanzar escaneo de directorios o capturar el login con burpsuite vemos que la página nos da cierta info, posible usuario mrb3n, sale por todos lados, y en la página de contacto tenemos una versión.

![](<Pasted image 20260220132724.png>)


Hacemos una pequeña busqueda y bingo, en ExploitDB existe una vulnerabilidad, un RCE en concreto.

![](<Pasted image 20260220132826.png>)


Revisándolo un poco vemos que sube una webshell utilizando el endpoint upload.php y bypasseando el archivo malicioso como una imagen, asi que lo primero es ver si nuestra web lo tiene, por lo que lanzaremos un gobuster con la extension php y o hubiera algo interesante también.txt por si acaso.

![](<Pasted image 20260220133844.png>)

Efectivamente existe por lo que nos descargaremos el script y lo probaremos. 
Al principio me fallaba ya que es viejo y estaba hecho para python2, inexistente desde 2020, asi que lo tuve que cambiar para python3. Una vez ejecutado nos da la webshell.

![](<Pasted image 20260220135055.png>)

Vamos a ver ante que nos enfrentamos.

![](<Pasted image 20260220135228.png>)

Listo usuarios y miro si la flag esta en el típico sitio y así es.

![](<Pasted image 20260220135427.png>)

Antes de pasar a la escalada voya obtener una shell con nc.exe, lo primero es llevarlo al directorio actual y lanzar un servidor para poder traerlo desde la maquina victima. 

![](<Pasted image 20260220140612.png>)

Tuve que hace las peticiones con curl ya que los one liner largos dan problemas en web shells, y utilizar Invoke-WebRequest ya que cerutil no me estaba funcionando. Tambien me fallaba al guardarlo en Windows/Temp por lo que utilice el directorio public.

![](<Pasted image 20260220142150.png>)

Mientras tanto tenia un nc en escucha en el puerto 4444.


![](<Pasted image 20260220145316.png>)

Hago una pequeña enumeración del sistema, usuarios y grupos.

![](<Pasted image 20260220145512.png>)

Por alguna razón no tenia permisos para traerme winPEAS, seguramente estaban limitadas las conexiones salientes, por lo que seguí enumerando.

![](<Pasted image 20260220150902.png>)

*Sin acabar*
