Empezamos lanzando nuestro nmap de reconocimiento de puertos abiertos.

![](evidences/Pasted%20image%2020260227194905.png)

Vemos que por la pinta se trata de un AD y mas concreto del Domain Controller. Ejecutamos los scripts en los mas importantes.

![](evidences/Pasted%20image%2020260227195536.png)

Podemos ya ver el nombre del dominio y vemos que el ldap sin autenticación puede estar activado, vamos a comprobarlo.

![](evidences/Pasted%20image%2020260227200149.png)

Efectivamente lo esta, por lo que vamos a enumerar usuarios:

![](evidences/Pasted%20image%2020260227200504.png)

Vemos los usuarios que nos interesan un poco mas abajo:

![](evidences/Pasted%20image%2020260227200544.png)

Vemos que ninguno tiene el bit 4194304 (0x400000) que es UF_DONT_REQUIRE_PREAUTH, por lo que en principio no hay AS-REP Roasting, lo comprobamos por si acaso.

![](evidences/Pasted%20image%2020260227201026.png)

Al enumerar usuarios con rpcclient, vemos adicional a parte del administrador, el de kerberos y el guest el usuario svc-alfresco, lo añadiremos a la lista.


![](evidences/Pasted%20image%2020260227201305.png)

Lo guardamos en un archivo hash.txt y con hashcat y el modo 18200 (Kerberos 5, etype 23, AS-REP) lo crackeamos.

![](evidences/Pasted%20image%2020260227201526.png)

Usuario svc-alfresco y contraseña s3rvice. Ya que tenemos el puerto 5985 abierto vamos a probar con evilWinRM. Obtenemos acceso y la flag de usuario.

![](evidences/Pasted%20image%2020260227201844.png)

Al tratarse de un AD vamos a utilizar bloodhound con Neo4j para ver mejor todo el esquema del AD, los usuarios y los dominios.


Recolectamos la información del AD con bloodhound y a través de Neo4j, subiremos el zip para verla representada.

![](evidences/Pasted%20image%2020260227204659.png)

Ejecutamos la query "Shortest Path To Domain Admins" y vemos el grafo.

![](evidences/Pasted%20image%2020260227204714.png)

Al seleccionar el nodo que hemos pwnedado (svc-alfresco) y ver a que podemos acceder vemos que es miembro Account Operators.

![](evidences/Pasted%20image%2020260227205430.png)

Que a su vez tiene un Generic All sobre Exchange Windows Permissions que llega al DC con WriteDacl, el propio bloodhound nos indica como explotarlo.

![](evidences/Pasted%20image%2020260227205506.png)

![](evidences/Pasted%20image%2020260227210013.png)


Vemos como se explota, pero primero necesitamos asignarle el privilegio DCSync. Lo primero creamos un usuario nuevo.

![](evidences/Pasted%20image%2020260227210304.png)

Y lo añadimos al grupo.

![](evidences/Pasted%20image%2020260227210451.png)

Seguimos los pasos que nos indica bloodhound y creamos un objeto
 credencial.
 
 ![](evidences/Pasted%20image%2020260227210911.png)

Vemos que no reconoce la función.

![](evidences/Pasted%20image%2020260227211347.png)

Por lo que necesitamos el script PowerView https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1

![](evidences/Pasted%20image%2020260227212143.png)

Y ya podremos dumpear los hashes de todos los usuarios incluyendo el Administrator.

![](evidences/Pasted%20image%2020260227212305.png)


Y con ese hash NTLM sacamos la parte NTHASH para hacer un pass the hash.

![](evidences/Pasted%20image%2020260227212633.png)

Y tenemos la flag.


![](evidences/Pasted%20image%2020260227212712.png)
