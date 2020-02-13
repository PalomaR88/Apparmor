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
- **aa-audit** 