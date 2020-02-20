# SELinux
## Introducción
Security-Enhanced Linux,SELinux, es una aquitectura de seguridad integrada en el kernel Linux y es, por tanto, un módulo de éste que no reemplaza el modelo tradicional de seguridad de los sistemas sino que los complementa. 

En su mayor parte, SELinux es casi invisible para la mayoría de los usuarios. Pero es tremendamente útil para los administradores de sistemas ya que permite un control total y granular sobre el sistema al completo. 

Proporciona un sistema flexible de control de acceso obligatorio, **MAC**, donde se indica cuando un objeto o sujeto puede acceder a otro objeto. Y, bajo el Linux estándar, se utiliza el control de acceso a discreción, **DAC**, donde la seguridad está dividida por niveles. También pueden definirse los controles de acceso a través de roles, **RBAC**.  

Los permisos se registran en una caché de acceso que es consultada cuando un sujeto, aplicación, intenta acceder a un objeto, fichero. Si con estos datos no se puede tomar una decisión, la petición continua al servidor de seguridad.

Puede ejecutarse en modo impositivo o permisivo, donde se verifican los registros, se registran los rechazos, pero no hace cumplir esta política. 

## Contextos de seguridad
Se conoce como **contexto de seguridad** al conjunto de los siguientes atributos:
#### Identidad de usuario
Tiene su propia base de datos de usuarios que está asociada la base de datos de usuarios de Linux. Para listarlos se utiliza **semanage user -l**
#### Role
Los usuarios pueden tener deferentes roles y éstos ser de diferentes tipos. Los principales roles son:
- user_r: mínimo privilegio.
- staff_r: puede alternar a los diferentes roles asignados.
- auditadm_r: encargado de las herramienta de auditoría.
- secadm_r: maneja las herramientas de seguridad, declara nuevos roles, crea nuevas políticas y puede ver los log de auditoría.
- sysadm_r: rol máximo.
#### Tipo
Atributo principal, llamado atributo primario.
#### Nivel

Ejemplo de la lista de usuario en SELinux:
~~~
[centos@salmorejo ~]$ sudo semanage user -l

                Etiquetado MLS/       MLS/                          
Usuario SELinux  Prefijo    Nivel MCS  Rango MCS                      Roles SELinux

guest_u         user       s0         s0                             guest_r
root            user       s0         s0-s0:c0.c1023                 staff_r sysadm_r system_r unconfined_r
staff_u         user       s0         s0-s0:c0.c1023                 staff_r sysadm_r system_r unconfined_r
sysadm_u        user       s0         s0-s0:c0.c1023                 sysadm_r
system_u        user       s0         s0-s0:c0.c1023                 system_r unconfined_r
unconfined_u    user       s0         s0-s0:c0.c1023                 system_r unconfined_r
user_u          user       s0         s0                             user_r
xguest_u        user       s0         s0                             xguest_r
~~~

## Ficheros y comandos
#### /etc/selinux/config
En **/etc/selinux/config** aparece la configuración principal de SELinux:
~~~
[root@salmorejo centos]# cat /etc/selinux/config 

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=enforcing
# SELINUXTYPE= can take one of three two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected. 
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
~~~

A través de este fihcero puede configurarse los atributos principales de SELinux. PEro además pueden usarse los siguientes comandos:
**SELINUX=< enforcing | permissive | disabled >**: estado de SELinux. 
- Enforcing: se impone la política de seguridad de SELinux.
- Permissive: No impone la política de SELinux, pero advierte y registra. 
- Disabled: Desactivado. 
**SELINUXTYPE=< targeted | strict >**: indica qué polícita se está implantando. 
- Targeted: solo se protegen ciertos demonios. Estas políticas para demonios concretos pueden activarse o desactivarse con **system-config-securitylevel**.
- Strict: protección completa.

#### Getenforce
Con el comando **getenforce** se puede saber cómo se están ejecutando las políticas de seguridad:
~~~
[root@salmorejo centos]# getenforce
Enforcing
~~~

#### Setenforce
Para activar o desactivar el modo enforcing se utiliza **setenforce** 1 o 0.

#### Sealert
En Fedora se incluye una potente herramienta para auditar SELinux y con ello una completa descripcion que ayuda a solucionar cualquier problema relacionado con los eventos que infringen las políticas de SELinux. Esta herramienta es **sealert -a /var/log/audit/audit.log**. Si se puede disponer de una consola gráfica se puede ejecutar **setreubleshoot**. Si se quiere permitir el acceso a una aplicación negada por SELInux se puede generaar un módulo de seguridad con el siguiente comando:
~~~
ausearch -c "<aplicacion>" -m avc | audit2allow -M <aplicacion>
~~~

Lo que genera 3 ficheros con las siguientes extensiones:
- .te: conjunto de reglas de seguridad sobre tipos, roles, atributos y dominios.
- .pp: paquete binario que contiene el módulo a cargar par ala nueva política.
-  .mod: módulo de politicas compiladas sin empaquetar. 

Para que, una vez generados los ficheros anteriores, se proceda a la instalación de las reglas se utiliza el siguiente comando:
~~~
semodule -i <fichero>.pp
~~~

#### Getsebool
Para obtener los valores de un booleano se utiliza el comando **getsebool**. Indica si un booleano particular o todos los booleanos de SELinux están activados o desactivados. Existe un estado, diferente a activado o desactivado, que ocurre cuando un booleano está pendiente al cambio de otro estado, se conoce como un cambio pendiente. 

Con la opción **-a** lista todos los booleanos, pero se puede hacer una consulta de un booleano en particular.

Para cambiar el estado de los booleanos de utiliza **setsebool < nombre_booleano > < 0 | 1 >**.

#### Semanage
Semanage se usa para configurar ciertos elementos de la política de SELinux sin requerir modificaciones o recompilación de las fuentes de políticas. 

## Caso práctico
En la máquina salmorejo, CentOS Linux 8, se va a implementar SELinux. Es posible que para utilizar determinadas herramientas de SELinux se necesite el paquete **stroubleshoot:
~~~
[centos@salmorejo ~]$ sudo dnf install setroubleshoot
~~~

Para saber las aplicaciones que se están denegando por SELinux: 
~~~
[centos@salmorejo ~]$ sudo sealert -a /var/log/audit/audit.log
...
100% done
found 40 alerts in /var/log/audit/audit.log
--------------------------------------------------------------------------------

SELinux está negando a /usr/sbin/php-fpm de setattr el acceso a carpeta /usr/share/nginx/nextcloud-data.

*****  El complemento catchall (100. confidence) sugiere**********************

Si cree que de manera predeterminada se debería permitir a php-fpm el acceso setattr sobre  nextcloud-data directory.     
Entoncesdebería reportar esto como un error.
Puede generar un módulo de política local para permitir este acceso.
Hacer
permita el acceso temporalmente ejecutando:
# ausearch -c 'php-fpm' --raw | audit2allow -M mi-phpfpm
# semodule -X 300 -i mi-phpfpm.pp


Información adicional:
Contexto de origen            system_u:system_r:httpd_t:s0
Contexto Destino              unconfined_u:object_r:usr_t:s0
Objetos Destino               /usr/share/nginx/nextcloud-data [ dir ]
Origen                        php-fpm
Dirección de origen           /usr/sbin/php-fpm
Puerto                        <Unknown>
Nombre de Equipo              <Unknown>
Paquetes RPM Fuentes          php-
                              fpm-7.2.11-2.module_el8.1.0+209+03b9a8ff.x86_64
Paquetes RPM Destinos         
RPM de Políticas              selinux-policy-3.14.3-20.el8.noarch
SELinux activado              True
Tipo de política              targeted
Modo impositivo               Enforcing
Nombre de equipo              salmorejo
Plataforma                    Linux salmorejo 5.4.0-1.el8.elrepo.x86_64 #1 SMP
                              Mon Nov 25 09:24:58 EST 2019 x86_64 x86_64
Cantidad de alertas           1
Visto por primera vez         2019-11-17 15:46:43 UTC
Visto por última vez          2019-11-17 15:46:43 UTC
ID local                      b730b210-7ca5-42ff-9a5d-45279d3844f5

Mensajes raw de aviso
type=AVC msg=audit(1574005603.399:1935): avc:  denied  { setattr } for  pid=9228 comm="php-fpm" name="nextcloud-data" dev="vda1" ino=1079393 scontext=system_u:system_r:httpd_t:s0 tcontext=unconfined_u:object_r:usr_t:s0 tclass=dir permissive=1


type=SYSCALL msg=audit(1574005603.399:1935): arch=x86_64 syscall=chmod success=yes exit=0 a0=7f536ed47680 a1=1f8 a2=7f538b225960 a3=0 items=0 ppid=9227 pid=9228 auid=4294967295 uid=997 gid=994 euid=997 suid=997 fsuid=997 egid=994 sgid=994 fsgid=994 tty=(none) ses=4294967295 comm=php-fpm exe=/usr/sbin/php-fpm subj=system_u:system_r:httpd_t:s0 key=(null)

Hash: php-fpm,httpd_t,usr_t,dir,setattr
--------------------------------------------------------------------------------
...
~~~

Devuelve 40 alertas, muchas de ellas relacionadas con nextclud, un programa cliente-servidor para alojamiento de ficheros en la nube, y php. Como indica el mensaje, se va a crear una política que permita a php accede al directorio de nextcloud. Con el comando **ausearch** se busca los eventos relacionados con php-fpm y con **audit2allow** se generan los ficheros necesarios para crear la nueva política. Con **semodule** se carga el módulo con extensión .pp que se ha creado con el comando aterior.
~~~
[centos@salmorejo ~]$ sudo ausearch -c 'php-fpm' --raw | audit2allow -M mi-phpfpm
******************** IMPORTANTE **********************
Para activar este paquete de políticas, ejecute:

semodule -i mi-phpfpm.pp

[centos@salmorejo ~]$ sudo semodule -X 300 -i mi-phpfpm.pp
~~~

A continuación, se le va a agregar un nuevo contexto a la ruta donde se ubica nextcloud. Para ello se utiliza la herramienta de adminitración de políticas, **semanage**, seguido de la opción **fcontext**, que sirve para la gestión de las definiciones del contexto. El nuevo contexto de la ruta de nextcloud será **httpd_sys_rw_content_t**.
~~~
[centos@salmorejo ~]$ sudo semanage fcontext -a -t httpd_sys_rw_content_t '/usr/share/nginx/nextcloud-data'
~~~

Y se restaura el contexto de seguridad predeterminado del fichero con **restorecon**:
~~~
[centos@salmorejo ~]$ sudo restorecon -v '/usr/share/nginx/nextcloud-data'
Relabeled /usr/share/nginx/nextcloud-data from unconfined_u:object_r:usr_t:s0 to unconfined_u:object_r:httpd_sys_rw_content_t:s0
~~~

## Otras configuraciones de SELinux
Para la configuración de ftp en esta misma máquina, ejercicio sobre la creación de un [Hosting](https://github.com/PalomaR88/Hosting/blob/master/Practica.md#configuraci%C3%B3n-de-selinux), se utilizó la siguiente sentencia:
~~~
[centos@salmorejo-3 html]$ sudo setsebool -P allow_ftpd_full_access=1
~~~

El comando **setsebool** establece el estado actual de un booleano a un valor dado. La sentencia anterior indica al servidor vsftpd permitir acceder a los usuarios locales a sus directorios de inicio. 

En este mismo ejercicio se activa también el permiso para que los script y los módulos del servidor web puedan conectarse a servidores de bases de datos:++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
[centos@salmorejo ~]$ sudo setsebool -P httpd_can_network_connect_db 1
~~~

Y se cambió el contexto de seguridad del siguiente directorio:
~~~
[centos@salmorejo ~]$ sudo semanage fcontext -a -t httpd_sys_content_t /usr/share/nginx/html/phpmyadmin
~~~

Otra ocasión donde se utilizaron herramientas de SELinux fue en la práctica de la creación de un [sitio Wordpress](https://github.com/PalomaR88/Instalacion_aplicaciones_web/blob/master/Practica-aplicaciones.md#configuraci%C3%B3n-de-selinux-para-wordpress). Para ello se modificaron los contextos de los directorios de configuración de wordpress:
~~~
[centos@salmorejo ~]$ sudo semanage fcontext -a -t httpd_sys_content_t /usr/share/nginx/html/wordpress
[centos@salmorejo ~]$ sudo semanage fcontext -a -t httpd_sys_rw_content_t /usr/share/nginx/html/wordpress/wp-config-sample.php 
[centos@salmorejo ~]$ sudo semanage fcontext -a -t httpd_sys_rw_content_t /usr/share/nginx/html/wordpress/wp-content
~~~








