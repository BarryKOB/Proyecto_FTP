## Instalacion DNS(BIND9)
1. Primero instalamos bind9 con: `sudo apt install bind9`
 
2. En el fichero `named.conf.local` configuramos la zona centro.loca, para ello entramos con `sudo nano /etc/bind/named.conf.local` ponemos lo siguiente.
<img width="613" height="280" alt="image" src="https://github.com/user-attachments/assets/9f0a5e3c-4b43-49a0-9bb3-d4fa99e93dae" />

3. Luego crearemos el archivo de tablas de la zona, para ello haremos una copia de una archivo y reemplazamos los datos. `sudo cp /etc/bind/db.local /etc/bind/db.centro.local`
   
4. Entramos al fichero y ponermos el siguiente contendio
<img width="740" height="442" alt="image" src="https://github.com/user-attachments/assets/109aa5ad-ff99-465f-b199-f97140c19d1e" />

5. Comprobamos que este todo bien con `named-checkconf`
<img width="596" height="111" alt="image" src="https://github.com/user-attachments/assets/24e0dece-44c5-4970-9372-193ebd108272" />

6. Reiniciamos el bind9 con `sudo systemctl restart bind9`
   
7. Comprobamos con dig y nslookup que funcione todo bien.

   1. Con nslookup
      <img width="553" height="192" alt="image" src="https://github.com/user-attachments/assets/5f4d6644-7e6e-47f3-9b5f-7d6853c937e9" />
<img width="357" height="139" alt="image" src="https://github.com/user-attachments/assets/64f9e44a-741c-46e9-8d30-dd5bed68bce1" />

   2. Con dig
   <img width="392" height="268" alt="image" src="https://github.com/user-attachments/assets/31e9f9ae-b11a-442a-8e30-c762cf82193c" />
   <img width="406" height="196" alt="image" src="https://github.com/user-attachments/assets/e4118986-d25e-4e4b-bcd1-decea144186f" />

## Instalacion LDAP
1- Instalamos Ldap con `sudo apt install slapd`.

2- Nos pedira poner una contraseña en nuestro caso `1234`

3- Cuando este instalado configuraremos el dominio con `sudo dpkg-reconfigure slapd`

   -- Primero nos dira si queremos omitir la configuracion del servidor, le diremos que `NO`
   
   -- Pondremos el nombre del dominio `centro.local`
   
   -- Luego el nomobre de la organizacion `centro`
   
   -- La contraseña de administrador `1234`
   
   -- Le decimos luego que si queremos borrar la bd y que la remplace
   
4. Luego crearemos la estructura ldif, para ello creamos el siguiente fichero con `sudo nano carga_inicial.ldif` dentro pondremos el siguiente contenido
   
```
dn: ou=usuarios,dc=centro,dc=local
objectClass: organizationalUnit
ou: usuarios

dn: ou=grupos,dc=centro,dc=local
objectClass: organizationalUnit
ou: grupos

dn: cn=profesores,ou=grupos,dc=centro,dc=local
objectClass: posixGroup
cn: profesores
gidNumber: 10001

dn: cn=alumnos,ou=grupos,dc=centro,dc=local
objectClass: posixGroup
cn: alumnos
gidNumber: 10002

dn: uid=alumno1,ou=usuarios,dc=centro,dc=local
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: Alumno1
sn: Alumno1
uid: alumno1
userPassword: {SSHA}9xKPhNCZiIVK6gRreBIfmIYQVPMZADt8
uidNumber: 20001
gidNumber: 10002
homeDirectory: /home/alumno1
loginShell: /bin/bash

dn: uid=alumno2,ou=usuarios,dc=centro,dc=local
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: Alumno2
sn: Alumno2
uid: alumno2
userPassword: {SSHA}9xKPhNCZiIVK6gRreBIfmIYQVPMZADt8
uidNumber: 20002
gidNumber: 10002
homeDirectory: /home/alumno2
loginShell: /bin/bash

dn: uid=profalumno1,ou=usuarios,dc=centro,dc=local
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: Profesor1
sn: Profesor1
uid: profalumno1
userPassword: {SSHA}9xKPhNCZiIVK6gRreBIfmIYQVPMZADt8
uidNumber: 20003
gidNumber: 10001
homeDirectory: /home/profalumno1
loginShell: /bin/bash

dn: uid=profalumno2,ou=usuarios,dc=centro,dc=local
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: Profesor2
sn: Profesor2
uid: profalumno2
userPassword: {SSHA}9xKPhNCZiIVK6gRreBIfmIYQVPMZADt8
uidNumber: 20004
gidNumber: 10001
homeDirectory: /home/profalumno2
loginShell: /bin/bash
```

5. Luego añadimos los usuario con `ldapadd -x -D "cn=admin, dc=centro, dc=local" -w -f carga_inicial.ldif`
<img width="754" height="421" alt="image" src="https://github.com/user-attachments/assets/0d8a7637-9c71-4f3c-9824-b91afa7bc094" />


6. Luego hacemos la prueba con `ldapsearch -x -b "dc=centro,dc=local" "(objectClass=posixAccount)"`
<img width="620" height="375" alt="image" src="https://github.com/user-attachments/assets/570854a3-c889-4d49-a8e4-9eda17611f74" />


1. Instalación de Servicios y Dependencias
En primer lugar, instalamos los paquetes necesarios para el servidor web, el servidor FTP y la comunicación con el servidor LDAP:

# Actualizar repositorios
sudo apt update

# Instalación de Apache2 y módulos LDAP
sudo apt install apache2 libapache2-mod-ldap -y

# Instalación de VSFTPD y librerías de autenticación PAM
sudo apt install vsftpd libpam-ldap -y

# Instalación de utilidades para gestión de AppArmor
sudo apt install apparmor-utils -y


2. Configuración de la Estructura de Archivos y Permisos
Creamos el árbol de directorios para la intranet y asignamos los permisos según los requisitos del proyecto (Grupos LDAP: profesores e alumnos).

# Creación de carpetas
sudo mkdir -p /var/www/html/intranet/public
sudo mkdir -p /var/www/html/intranet/alumnos
sudo mkdir -p /var/www/html/intranet/profesores

# Asignación de grupos y permisos restrictivos (770)
sudo chown -R :10001 /var/www/html/intranet/profesores  # GID profesores
sudo chown -R :10002 /var/www/html/intranet/alumnos     # GID alumnos
sudo chmod -R 770 /var/www/html/intranet/profesores
sudo chmod -R 770 /var/www/html/intranet/alumnos
sudo chmod -R 755 /var/www/html/intranet/public


3. Configuración del Servidor Web (Apache2)
Activamos los módulos de autenticación y configuramos el VirtualHost para validar contra el servidor LDAP (srv-infra).

Paso A: Activar módulos

sudo a2enmod ldap
sudo a2enmod authnz_ldap


Paso B: Configuración del VirtualHost
Editamos /etc/apache2/sites-available/intranet.conf con las directivas AuthLDAPURL para apuntar a nuestro servidor de nombres.

<img width="792" height="436" alt="imgintranet" src="https://github.com/user-attachments/assets/68ded475-223a-41c0-b3fd-708111eed2f2" />


4. Configuración del Servidor FTP (VSFTPD)
Configuramos el acceso para usuarios de red y solucionamos el error de cambio de directorio mediante la creación de "homes" técnicos.

Paso A: Ajustes en vsftpd.conf
Modificamos /etc/vsftpd.conf con los siguientes parámetros clave:

local_enable=YES

chroot_local_user=YES

local_root=/var/www/html/intranet

allow_writeable_chroot=YES

pam_service_name=common-auth (Para heredar la autenticación LDAP del sistema)


Paso B: Creación de directorios Home
Para evitar el error 500 OOPS: cannot change directory, creamos las carpetas físicas para los usuarios de LDAP:

sudo mkdir -p /home/profalumno1 /home/profalumno2 /home/alumno1 /home/alumno2
sudo chown profalumno1 /home/profalumno1
# ... repetir para el resto de usuarios

Paso C: Ajuste de AppArmor
Para permitir que el servicio FTP acceda a la ruta de la intranet:
sudo aa-complain /usr/sbin/vsftpd

5. Pruebas de Funcionamiento
Verificación DNS

<img width="792" height="367" alt="imgns" src="https://github.com/user-attachments/assets/994ea5ec-f30c-4449-b5f0-8575238ab3e7" />


Prueba de Acceso Web (Apache)
Acceder a http://intranet.centro.local/profesores.

Validar con usuario profalumno1. Resultado: Éxito.

Intentar acceder con usuario alumno1. Resultado: Acceso Denegado (403).

<img width="592" height="352" alt="imgpro" src="https://github.com/user-attachments/assets/86abd93f-9893-458e-a0d2-62bf658d863b" />

<img width="860" height="527" alt="imgpro1" src="https://github.com/user-attachments/assets/8ffbc15a-27f0-4020-a5bf-e85eeada8c8c" />

<img width="852" height="547" alt="imgpro12" src="https://github.com/user-attachments/assets/000ee357-c305-44db-a1a2-50d6283be659" />

<img width="862" height="492" alt="imgal" src="https://github.com/user-attachments/assets/791d0788-ba41-4767-8159-11ef310331af" />

<img width="682" height="291" alt="imgal1" src="https://github.com/user-attachments/assets/76ddcec9-e53b-491d-b78a-c7b806e274a5" />

entramos en alumnos

<img width="682" height="421" alt="imgal2" src="https://github.com/user-attachments/assets/40d419a5-d6e9-4b96-a757-da6ff18eca45" />

<img width="837" height="432" alt="imgal3" src="https://github.com/user-attachments/assets/8ce49d57-eef3-459c-af01-2fce6f48368e" />

intentamos entrar en la seccion de alumnos con el usuario 		profalumno1 

<img width="842" height="502" alt="imgalprof" src="https://github.com/user-attachments/assets/5deabcb4-6efb-4343-a495-0106c9122e10" />

<img width="852" height="401" alt="imgres" src="https://github.com/user-attachments/assets/217dedbf-aa96-4699-80d3-0a39fa4333ed" />

Prueba de Publicación (FTP)
Conectar vía FileZilla a 192.168.1.60 con el usuario alumno1.

Subir un archivo prueba.txt a la carpeta /alumnos. Resultado: Transferencia exitosa.

Intentar subir el archivo a la carpeta /profesores. Resultado: Error (Permiso denegado).

<img width="302" height="507" alt="ftp1" src="https://github.com/user-attachments/assets/740d68a4-6aa8-425b-8c77-e3e54a35ba58" />

escribirnos en el archivo 

<img width="687" height="457" alt="ftp2" src="https://github.com/user-attachments/assets/846f8e74-e1a9-4de9-be02-067a888009c8" />

<img width="856" height="396" alt="ftp3" src="https://github.com/user-attachments/assets/a7a6c2ea-a75f-4234-a555-3dfbc6efb2d6" />

<img width="837" height="691" alt="ftp4" src="https://github.com/user-attachments/assets/e285e90e-f987-460a-b3fa-7c9acbee217c" />

y subimos el fichero Prueba_FTP a la carpeta profesores 

<img width="842" height="667" alt="ftp5" src="https://github.com/user-attachments/assets/6dfc8355-889c-41d1-b820-efb0096da8d3" />

en el servidor 

<img width="835" height="451" alt="ftp6" src="https://github.com/user-attachments/assets/3607ebdf-bcd2-492f-a50c-e1fbc99a9aaa" />

<img width="877" height="401" alt="ftp7" src="https://github.com/user-attachments/assets/43bc4853-563a-4225-be48-7792f7c91a31" />

como vemos si picamos en la carpeta de profesores siendo alumnos nos dara un error 

<img width="757" height="622" alt="ftp8" src="https://github.com/user-attachments/assets/79731f65-0499-4206-88b5-e2670cce6310" />



   


