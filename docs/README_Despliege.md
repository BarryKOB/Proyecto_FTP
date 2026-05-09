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









   


