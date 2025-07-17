# dns-recursivo-pihole-unbound

Guía práctica para desplegar un servidor DNS recursivo y privado usando Pi-hole + Unbound en Docker sobre OpenMediaVault 7. El sistema permite bloquear publicidad, resolver dominios sin depender de servidores externos y mejorar la privacidad en toda la red local. El tutorial está orientado a usuarios con OpenMediaVault 7, pero los conceptos y configuraciones aplican para otros sistemas basados en Linux que usen Docker.


# 🧠 ¿Qué hace esta configuración y por qué te conviene?

Este proyecto te permite montar un **servidor DNS privado y recursivo** en tu red local usando **Pi-hole** y **Unbound** sobre OpenMediaVault.

🔒 **Privacidad total**:  
Normalmente, cuando abrís una página web, tu dispositivo le pregunta a los servidores de tu proveedor de internet (o a Google, Cloudflare, etc.) cómo llegar. Eso significa que todos tus movimientos en internet quedan registrados por terceros (menos privacidad), y que no podés filtrar ni bloquear nada desde tu red (menos control).  
➡️ Con esta configuración, tu servidor **resuelve los dominios directamente desde la raíz de internet**, sin intermediarios. Es como preguntarle a directo a la fuente en lugar de a un intermediario (menos dependencia), pero además luego elegís con qué te quedas del contenido que encontras (dejando atrás publicidad, rastreadores, etc.).

🚫 **Bloqueo de anuncios y rastreadores**:  
Pi-hole actúa como un filtro: cuando un dispositivo de tu red pide acceso a una web conocida por mostrar publicidad o espiar, simplemente bloquea la solicitud.  
➡️ Así, toda tu red queda protegida sin instalar nada en cada dispositivo.

🚀 **Velocidad y control**:  
Unbound guarda en caché las respuestas DNS, así que las segundas veces que se consultan son casi instantáneas.  
Además, todo el tráfico DNS pasa por tu propio servidor, y vos decidís qué permitir o bloquear.

🧱 **Sistema autónomo y persistente**:  
Una vez configurado, el sistema se mantiene estable incluso después de reinicios. No depende de configuraciones frágiles ni de conexiones externas para funcionar.

En resumen: esta configuración te da **más privacidad, menos publicidad, mayor velocidad y control total** sobre el tráfico DNS de tu red. Y todo corre desde tu propio servidor local.

🤔 **¿Por qué usaría esta configuración si no tengo nada que ocultar?**:
Todos saben lo que vamos a hacer al baño, y aun así cerramos la puerta. La privacidad no es un delito ni una sospecha: es un derecho básico.

# ⚙️ Configuración paso a paso

### 🧱 1. Preparación del sistema anfitrión
El primer paso es asegurarse de que **ningún servicio del sistema esté ocupando el puerto 53** y que la configuración DNS del servidor sea **estable y persistente** tras los reinicios.
#### 🔻 Deshabilitar systemd-resolved (común en OMV y otros sistemas Debian/Ubuntu)
Este servicio local de DNS suele usar el puerto 53. Hay que detenerlo y deshabilitarlo:
```bash
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved
```
> ⚠️ Nota: si tu sistema usa otro servicio DNS, como dnsmasq, deberás detenerlo de forma similar. Consultá la documentación de tu distribución si no estás seguro.
#### 🛠️ Configurar el DNS del host de forma permanente
Para que el propio servidor pueda resolver dominios y su configuración no se sobreescriba, reemplazamos el enlace simbólico /etc/resolv.conf por un archivo estático e inmutable:
```bash
sudo rm /etc/resolv.conf
echo "nameserver 127.0.0.1" | sudo tee /etc/resolv.conf
sudo chattr +i /etc/resolv.conf
```

### 📁 2. Preparación de archivos y directorios
Antes de desplegar el stack, es necesario crear las carpetas para los volúmenes y colocar los archivos de configuración ya corregidos.
#### 📂 Crear directorios de volúmenes
```bash
mkdir -p /compose/data/pihole/etc-pihole
mkdir -p /compose/data/pihole/etc-dnsmasqd
mkdir -p /compose/data/unbound
```
> Estas rutas y nombres de carpetas son solo ejemplos, lo importante es que estén las mínimas requeridas como volumen en el stack (archivo yaml), y que haya una coherencia entre las rutas señaladas en el propio stack y las que hayas creado.
#### ⚙️ Colocar archivos de configuración
1. unbound.conf: Debe estar en /compose/data/unbound/. 
2. docker-compose.yml: Define los servicios de Pi-hole y Unbound, sus redes, puertos y volúmenes. Debería estar listo para desplegar sin modificarlo desde la interfaz web.

### 🚀 3. Despliegue y configuración final
Con los archivos listos, se procede a desplegar el stack desde OpenMediaVault y configurar Pi-hole.

#### 🧩 Desplegar el stack desde OMV
1. Ir a Services > Compose > Files
2. Pegar el contenido de docker-compose.yml
3. Modificar las líneas que tengan comentarios pidiendo modificaciones (en español)
4. Crear y desplegar el stack
> ⚠️ Nota: si Pihole no levanta, editas el stack y cambias momentáneamente el valor de la línea DNS1 a
> ``` bash
> 1.1.1.1
> ```
> Luego de que arranque, lo restauras a como estaba antes.
> ⚠️ Nota: si por algún motivo necesitas corregir configuraciones o stacks, el procedimiento correcto para volver a desplegarlos es "bajarlos" y "subirlos" de nuevo desde la interfaz de OMV, **poner stop no va a surtir el efecto deseado**.
### 🛡️ Configurar Pi-hole
Una vez que el contenedor esté corriendo, 
1. Acceder a la interfaz web de Pi-hole en tu navegador colocando la IP de la máquina host, el signo ":" y el puerto asignado. Por ejemplo: 
``` bash 
192.168.0.20:85
```
2. Loguearte con la contraseña establecida en el stack.
> ⚠️ Nota: si no te funciona esa contraseña, tenés que ir a la consola del host (si querés, podes hacerlo por ssh root@IP-del-host con una máquina cliente) y ejecutar
> ``` bash
> docker exec -it pihole pihole setpassword
> ```
3. Ir a Settings > DNS > Custom DNS server
4. Escribir:
```bash
172.23.0.8#53
```
5. Tocas donde dice "Basic" en verde y lo cambias a "Expert" en rojo
6. Marcas "Permit all origins" 
7. Tocas "Save & Apply"

### 🧪 4. Configuración de los clientes
En cada dispositivo en el que quieras usar esta configuración (PC, teléfono, etc.), establecer como servidor DNS la IP local del servidor OMV. Por ejemplo:
```bash
192.168.0.20
```
> ⚠️ Nota: es muy recomendable tener configurada una IP estática. En OMV esto es muy sencillo, y cualquier asistente de IA te puede guiar a hacerlo.

✅ Con esto, todos los dispositivos usarán tu nuevo sistema DNS recursivo y bloqueador de publicidad.

