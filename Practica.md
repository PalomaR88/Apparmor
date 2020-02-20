# AppArmor
## Introducción
Apparmor es un sistema de control obligatorio de acceso donde el núcleo pregunta a AppArmor antes de cada llamada para saber si un proceso está autorizado a realizar la operación. De esta forma, Apparmor limita los recursos de un programa.

### Perfiles
Al conjunto de reglas que se le aplican a cada programa se le llama perfil. Al contrario que en SELinux, las reglas que se aplican no dependen del usuario sino que a todos los usuarios se les aplican las mimsas reglas. Estos perfiles se guardan en **/etc/apparmor.d/**. Para compilar y cargar los perfiles se utiliza el comando **apparmor_parse**. Hay dos modos para cargar los perfíles:
- Modo estricto (enforcing): cuando se viola una regla se bloquea la acción y se registra la incidencia.
- Modo relajado (compleining): cuando se viola una regla solo se registra la incidencia pero se lleva a cabo la acción.

### Instalación
Para activar AppAmor basta con instalar el paquete **apparmor**. Algunos de sus paquetes adicionales son **apparmor-profiles** (conjunto de perfiles desarrollados por la comunidad AppArmor) y **apparmor-utils** (herramientas). Otro paquete es **apparmor-profiles-extra** (contiene perfiles adicionales desarrollados por Ubuntu y Debian).

Otros comandos de utilidad son:
- **aa-status**: estado del servicio. 
- **aa-enforce** <ruta>: para modificar al modo estricto.
- **aa-complain** <ruta>: para modificar al modo relajado.
- **aa-desable** <ruta>: para desactivar.
- **aa-audit**: modo auditoría.
- **aa-unconfined**: muestra la lista de programas que expone al menos un zócalo de red.
- **aa-genprof** <servicio>: para crear un perfil. 
- **aa-autodep**:crea un perfil en blanco.

## Puesta en funcionamiento
A continuación se va a configurar AppArmor en la máquina Tortilla, con sistema operativo Debian Buster. En la máquina hay 15 perfiles activos en modo enforce:
~~~
ubuntu@tortilla:~$ sudo apparmor_status
apparmor module is loaded.
15 profiles are loaded.
15 profiles are in enforce mode.
   /sbin/dhclient
   /usr/bin/lxc-start
   /usr/bin/man
   /usr/lib/NetworkManager/nm-dhcp-client.action
   /usr/lib/NetworkManager/nm-dhcp-helper
   /usr/lib/connman/scripts/dhclient-script
   /usr/lib/snapd/snap-confine
   /usr/lib/snapd/snap-confine//mount-namespace-capture-helper
   /usr/sbin/tcpdump
   lxc-container-default
   lxc-container-default-cgns
   lxc-container-default-with-mounting
   lxc-container-default-with-nesting
   man_filter
   man_groff
0 profiles are in complain mode.
0 processes have profiles defined.
0 processes are in enforce mode.
0 processes are in complain mode.
0 processes are unconfined but have a profile defined.
~~~

Los servicios que se deben configurar son:
~~~
ubuntu@tortilla:~$ sudo aa-unconfined
2892 /usr/lib/postfix/sbin/master not confined
3021 /usr/sbin/bacula-fd not confined
9904 /usr/sbin/apache2 not confined
9905 /usr/sbin/apache2 not confined
9906 /usr/sbin/apache2 not confined
9907 /usr/sbin/apache2 not confined
9908 /usr/sbin/apache2 not confined
11805 /lib/systemd/systemd-networkd not confined
11810 /lib/systemd/systemd-resolved not confined
15496 /usr/sbin/sshd not confined
26954 /usr/sbin/mysqld not confined
31297 /usr/sbin/apache2 not confined
~~~


ubuntu@tortilla:~$ cd /etc/apparmor.d/
ubuntu@tortilla:/etc/apparmor.d$ sudo aa-autodep nginx
Writing updated profile for /usr/sbin/nginx.
ubuntu@tortilla:/etc/apparmor.d$ sudo service nginx restart
ubuntu@tortilla:/etc/apparmor.d$ sudo aa-logprof
Reading log entries from /var/log/syslog.
Updating AppArmor profiles in /etc/apparmor.d.
Complain-mode changes:

Profile:    /usr/sbin/nginx
Capability: dac_override
Severity:   9

 [1 - #include <abstractions/lxc/container-base>]
  2 - #include <abstractions/lxc/start-container> 
  3 - capability dac_override, 
(A)llow / [(D)eny] / (I)gnore / Audi(t) / Abo(r)t / (F)inish
Adding #include <abstractions/lxc/container-base> to profile.
Deleted 2 previous matching profile entries.

= Changed Local Profiles =

The following local profiles were changed. Would you like to save them?

 [1 - /usr/sbin/nginx]
(S)ave Changes / Save Selec(t)ed Profile / [(V)iew Changes] / View Changes b/w (C)lean profiles / Abo(r)t
Writing updated profile for /usr/sbin/nginx.


