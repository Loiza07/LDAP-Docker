README – Proyecto Reto3 (LDAP + Docker)

Este proyecto contiene la infraestructura necesaria para desplegar un servidor OpenLDAP configurado manualmente dentro de contenedores Docker, así como archivos auxiliares para la gestión y modificación de usuarios dentro del directorio.
La estructura está pensada para entornos de prácticas de ciberseguridad y autenticación centralizada (SSO).

🎯 Objetivo del proyecto

Este repositorio es parte del Reto 3, cuyo propósito es:

Construir OpenLDAP desde cero en Docker.

Gestionar usuarios mediante LDIF.

Evitar imágenes preconfiguradas (osixia/openldap).

Preparar el entorno para integrarlo más adelante con Keycloak para SSO.

📁 Estructura del Proyecto
1. .env

Archivo con variables de entorno utilizadas por Docker Compose.
Suele incluir parámetros como dominios LDAP, usuarios admin, contraseñas o rutas.

2. docker-compose.yml

Define la infraestructura principal del proyecto.
Incluye el servicio del servidor LDAP construido desde el Dockerfile incluido en ldap-server/.

3. ldap-server/

Carpeta con todo lo necesario para construir el contenedor del servidor LDAP:

🔹 Dockerfile

Construye una imagen personalizada de OpenLDAP sin usar imágenes preconfiguradas, siguiendo los requisitos del proyecto.

🔹 entrypoint.sh

Script que se ejecuta al iniciar el contenedor.
Generalmente inicializa la base de datos, aplica LDIF iniciales y configura el servidor.

🔹 slapd.conf.template

Plantilla del archivo de configuración principal de OpenLDAP.
Se completa con valores del entorno o variables durante el build/entrypoint.

🔹 ldif/

Contiene LDIFs de creación de usuarios iniciales:

aimar_costana.ldif

aritz_loizate.ldif

ortzi_gonzalez.ldif

patxi_chocan.ldif

Cada archivo incluye los atributos necesarios para añadir usuarios al directorio LDAP.

4. ldapmodifi/

Carpeta con LDIFs para modificaciones posteriores:

add-mail.ldif → Añade un correo electrónico a usuarios.

remove-mail.ldif → Elimina el atributo mail de usuarios.

Sirve para aplicar cambios usando ldapmodify.

5. volumes/ldap-data/

Carpeta utilizada por Docker como volumen persistente del servidor LDAP.

Incluye archivos de la base de datos real:

data.mdb

lock.mdb

No se deben editar manualmente.

6. home/asir2/certs/

🔹phpldap/

Contiene los certificados necesarios para el uso de certificados SSL de phpldapadmin

🔹ldap/

Contiene los certificados necesarios para el uso de certificados SSL de ldap

🔹keycloak/

Contiene los certificados necesarios para el uso de certificados SSL de keycloak

🚀 Uso del Sistema LDAP
  1. Construir la imagen

	  docker compose build
  
  2. Levantar el servicio

	  docker compose up -d

👤 Gestión de Usuarios LDAP

✅ Listar usuarios
  1. Ver todos los usuarios
	docker exec -it ldap ldapsearch -x -H ldap://localhost \
	-D "cn=admin,dc=hashimodos,dc=local" -w admin123 \
	-b "ou=People,dc=hashimodos,dc=local"
  
  2. Ver solo usuarios (objectClass inetOrgPerson)
	docker exec -it ldap ldapsearch -x -H ldap://localhost \
	-D "cn=admin,dc=hashimodos,dc=local" -w admin123 \
	-b "ou=People,dc=hashimodos,dc=local" "(objectClass=inetOrgPerson)"
  
  3. Listar solo los UID
	docker exec -it ldap ldapsearch -x -LLL -H ldap://localhost \
	-D "cn=admin,dc=hashimodos,dc=local" -w admin123 \
	-b "ou=People,dc=hashimodos,dc=local" uid

🔄 Modificar usuarios
  Ejemplo para añadir correo
	dn: uid=usuario1,ou=People,dc=hashimodos,dc=local
	changetype: modify
	add: mail
	mail: usuario1@hashimodos.local


  Aplicación:
	docker exec -it ldap ldapmodify -x -H ldap://localhost \
	-D "cn=admin,dc=hashimodos,dc=local" -w admin123 -f /usuario1.ldif

🗑️ Eliminar usuarios
  Método recomendado (ldapdelete)
	docker exec -it ldap ldapdelete -x -H ldap://localhost \
	-D "cn=admin,dc=hashimodos,dc=local" -w admin123 \
	"uid=usuario1,ou=People,dc=hashimodos,dc=local"

  Alternativa con LDIF
	dn: uid=usuario1,ou=People,dc=hashimodos,dc=local
	changetype: delete


  Aplicación:
	docker exec -it ldap ldapmodify -x -H ldap://localhost \
	-D "cn=admin,dc=hashimodos,dc=local" -w admin123 \
	-f /delete-user.ldif

🔍 Verificación

✔️ Comprobar que un usuario existe
	docker exec -it ldap ldapsearch -x -H ldap://localhost \
	-D "cn=admin,dc=hashimodos,dc=local" -w admin123 \
	-b "ou=People,dc=hashimodos,dc=local" "(uid=usuario1)"


Si no aparece ningún resultado → el usuario ha sido eliminado.
