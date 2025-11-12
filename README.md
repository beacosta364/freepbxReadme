# 🧠 Servidor de Telefonía IP con FreePBX y Zoiper

## 📌 Descripción

Este proyecto implementa un **servidor de telefonía IP** usando **FreePBX (Asterisk 16)** como PBX y **Zoiper 5** como cliente SIP.  
Incluye instalación, configuración de extensiones, medidas de seguridad y pruebas funcionales entre usuarios internos.

---

## ⚙️ Requerimientos del sistema

| Recurso | Mínimo | Recomendado |
|----------|--------|-------------|
| CPU | 1 vCPU | 2 vCPU o más |
| RAM | 2 GB | 4 GB o más |
| Disco duro | 20 GB | 40 GB |
| SO base | SNG7-PBX-64bit-2203-2 (basado en CentOS 7) |  |
| Red | IP estática | Red interna o VLAN controlada |

---

## 💾 Instalación de FreePBX

1. Montar la ISO `SNG7-PBX-64bit-2203-2.iso` en una máquina virtual.
2. Asignar recursos según la tabla anterior.
3. Completar la instalación gráfica.
4. Acceder al sistema por SSH:
   ```bash
   ssh root@<IP_del_servidor>

   
Verificar servicios activos:
systemctl status asterisk
systemctl status fail2ban
fwconsole list

Acceso a la interfaz web (GUI)

Acceder desde el navegador con:

http://<IP_del_servidor>/


Ejemplo: http://192.168.50.189

Desde la GUI se gestionan:

Extensiones SIP/PJSIP

Troncales (Trunks)

Grabaciones y buzones de voz

Firewall y Fail2Ban

📞 Configuración de extensiones SIP

Ir a: Applications → Extensions → Add New Extension (PJSIP)

Crear extensiones de prueba (por ejemplo 2001 y 2002)

Guardar y aplicar cambios (Apply Config)

Verificar desde la CLI:

asterisk -rvv
sip show peers
core show channels

📱 Configuración del cliente Zoiper 5

Instalar Zoiper 5 en el equipo cliente.

Crear una nueva cuenta SIP con los siguientes datos:

Usuario: 2001
Contraseña: (la generada en FreePBX)
Dominio/Host: 192.168.50.189
Puerto: 5060


Registrar y realizar llamadas entre extensiones (2001 ↔ 2002).

🔐 Seguridad del servidor
🔸 Firewall de FreePBX

Comandos básicos:

fwconsole firewall start        # Iniciar firewall
fwconsole firewall restart      # Reiniciar firewall
fwconsole firewall list rules   # Ver reglas activas


Ejemplo de restricción SIP (solo red interna):

fwconsole firewall trust 192.168.50.0/24
fwconsole firewall zone list


Bloquear acceso SSH público:

firewall-cmd --zone=public --remove-service=ssh --permanent
firewall-cmd --reload

🔸 Fail2Ban

Ver estado:

systemctl status fail2ban


Listar jails activos:

fail2ban-client status


Ejemplo de salida:

Jail list: apache-tcpwrapper, ssh-iptables, pbx-gui, asterisk-iptables


Protege contra:

Intentos de acceso SSH fallidos

Ataques SIP por fuerza bruta

Escaneos web o de puertos

🔸 Validación y mantenimiento
fwconsole validate          # Revisión de integridad
fwconsole validate --clean  # Limpieza de archivos sospechosos
fwconsole chown             # Corrige permisos del sistema

🧱 Puertos utilizados
Servicio	Puerto	Protocolo	Descripción
SIP (PJSIP)	5060 / 5160	UDP	Señalización de llamadas
RTP	10000–20000	UDP	Transmisión de voz
HTTP / HTTPS	80 / 443	TCP	Interfaz Web
SSH	22	TCP	Administración remota
Fail2Ban	—	—	Bloqueo dinámico de IPs
🧰 Comandos útiles
fwconsole restart           # Reinicia FreePBX y Asterisk
fwconsole firewall start    # Inicia el cortafuegos
fwconsole sysadmin ports    # Lista puertos activos
asterisk -rvv               # CLI de Asterisk
sip show peers              # Ver registros SIP
core show channels           # Llamadas activas
netstat -tulnp              # Ver puertos en escucha

💡 Buenas prácticas de seguridad
Medida	Acción
Cambiar puerto SSH	Editar /etc/ssh/sshd_config y modificar Port 2222
Permitir SSH solo a IPs internas	AllowUsers root@192.168.50.*
Usar HTTPS en interfaz	Configurar certificado en Admin → Certificate Management
Deshabilitar módulos no usados	Desde la GUI o fwconsole ma delete <modulo>
Cerrar puertos no utilizados	firewall-cmd --remove-port=<puerto>
🧾 Créditos

Autor: Brayan Acosta
Materia: Seguridad y Redes / Telefonía IP
Software usado: FreePBX 15 + Asterisk 16 + Zoiper 5
Objetivo: Configuración segura de un entorno de telefonía VoIP interno.


## 📊 Monitoreo del sistema y del servicio Asterisk

### 🔹 Monitoreo general del sistema

| Propósito | Comando | Descripción |
|------------|----------|--------------|
| Ver uso de CPU y memoria | `top` o `htop` | Muestra procesos activos y consumo de recursos. |
| Espacio en disco | `df -h` | Muestra espacio disponible en todas las particiones. |
| Uso de red | `iftop` o `ip -s link` | Monitorea tráfico en tiempo real por interfaz. |
| Ver servicios activos | `systemctl list-units --type=service` | Lista todos los servicios en ejecución. |
| Consultar IP asignada | `ip addr show` | Verifica configuración de red actual. |

---

### 🔹 Monitoreo del servicio FreePBX / Asterisk

| Propósito | Comando | Descripción |
|------------|----------|--------------|
| Entrar a la consola de Asterisk | `asterisk -rvv` | Acceso interactivo con logs en tiempo real. |
| Ver llamadas activas | `core show channels` | Muestra las llamadas y canales abiertos. |
| Ver peers SIP registrados | `sip show peers` | Lista las extensiones registradas con su IP y estado. |
| Ver registros PJSIP | `pjsip show endpoints` | Similar al anterior, pero para endpoints PJSIP. |
| Ver cola de mensajes SIP | `sip show registry` | Verifica registro de troncales externas. |
| Monitorear flujo de llamadas | `core show calls` | Muestra número de llamadas procesadas. |
| Ver consumo de CPU de Asterisk | `ps -aux | grep asterisk` | Identifica el uso de recursos del proceso. |

---

### 🔹 Monitoreo de seguridad

| Propósito | Comando | Descripción |
|------------|----------|--------------|
| Revisar intentos de acceso SSH | `grep "Failed password" /var/log/secure` | Detecta intentos de intrusión. |
| Ver estado de Fail2Ban | `fail2ban-client status` | Muestra jails activos y estado general. |
| Ver IPs bloqueadas | `fail2ban-client status asterisk-iptables` | Muestra IPs bloqueadas por ataques SIP. |
| Logs de Asterisk | `tail -f /var/log/asterisk/full` | Monitorea actividad y errores del sistema. |
| Logs del sistema | `journalctl -xe` | Verifica mensajes críticos del sistema. |

---

### 🔹 Monitoreo en la interfaz Web (GUI)

Desde la interfaz gráfica de FreePBX:

- **Reports → Asterisk Info:** muestra estadísticas en tiempo real (canales, troncales, codecs).  
- **Reports → Call Event Logging (CEL):** historial detallado de llamadas.  
- **Dashboard:** estado general del sistema (CPU, RAM, Asterisk, servicios y actualizaciones).  
- **Admin → System Admin → Updates:** control de parches y versiones.

---

### 🔹 Comando resumen de estado general

fwconsole show
fwconsole status
asterisk -rx "core show uptime"
asterisk -rx "core show channels"
asterisk -rx "pjsip show endpoints"
asterisk -rx "core show version"
---

Ejemplo de monitoreo continuo
watch -n 5 "asterisk -rx 'core show channels concise'"



Puedes crear un script de monitoreo rápido con:

nano monitor-voip.sh

#!/bin/bash
echo "===== ESTADO GENERAL DE FREEPBX ====="
fwconsole status
echo
echo "===== LLAMADAS ACTIVAS ====="
asterisk -rx "core show channels"
echo
echo "===== ENDPOINTS PJSIP ====="
asterisk -rx "pjsip show endpoints"
echo
echo "===== ESTADO FAIL2BAN ====="
fail2ban-client status asterisk-iptables





Dale permisos y ejecútalo:

chmod +x monitor-voip.sh
./monitor-voip.sh




 1. Comandos básicos del sistema
Comando	Descripción
uname -a	Muestra información del sistema operativo.
uptime	Indica cuánto tiempo lleva encendido el sistema.
df -h	Muestra el uso del disco.
free -h	Muestra la memoria RAM usada y disponible.
top	Muestra los procesos activos en tiempo real.
	
journalctl -xe	Muestra los últimos eventos o errores del sistema.

2. Comandos FreePBX / Asterisk
Comando	Descripción
fwconsole status	Verifica el estado general de FreePBX y Asterisk.
fwconsole start	Inicia Asterisk y todos los módulos FreePBX.
fwconsole restart	Reinicia Asterisk y FreePBX.
fwconsole stop	Detiene Asterisk.
fwconsole reload	Recarga la configuración sin reiniciar servicios.
asterisk -rvvv	Entra a la consola de Asterisk en modo interactivo.
core show channels	(Dentro de Asterisk) Muestra las llamadas activas.
sip show peers	(Dentro de Asterisk) Lista las extensiones SIP registradas.
pjsip show endpoints	(Dentro de Asterisk) Lista los endpoints PJSIP activos.
core show version	Muestra la versión exacta de Asterisk.

4.	Comandos de Firewall FreePBX
Comando	Descripción
fwconsole firewall start	Inicia el módulo de firewall de FreePBX.
fwconsole firewall stop	Detiene el firewall.
fwconsole firewall status	Muestra si el firewall está activo o no.
fwconsole firewall list	Lista todas las zonas y reglas activas.
fwconsole firewall list trusted	Muestra lo que está en la zona “trusted”.
fwconsole firewall trust 192.168.0.0/16	Marca toda la red local como de confianza.
fwconsole firewall add trusted 5060/udp	Permite el puerto SIP (voz).
fwconsole firewall add trusted 22/tcp	Permite conexión SSH.
fwconsole firewall block 23/tcp	Bloquea un puerto (por ejemplo, Telnet).
fwconsole firewall restart	Reinicia el firewall aplicando los cambios.

5.	Comandos de red y conectividad
Comando	Descripción
ip addr	Muestra todas las interfaces de red y sus IP.
ping 8.8.8.8	Comprueba la conexión a internet.
`netstat -tulnp	grep asterisk`
ss -tulnp	Similar a netstat, muestra conexiones activas.
hostname -I	Muestra solo las IP del sistema.
6.	Monitoreo y seguridad
Comando	Descripción
fail2ban-client status	Muestra el estado de Fail2Ban y los jails activos.
fail2ban-client status asterisk-iptables	Muestra IPs bloqueadas por ataques SIP.
fwconsole firewall f2bstatus	Muestra IPs bloqueadas desde FreePBX.
tail -f /var/log/asterisk/full	Muestra en tiempo real los eventos de Asterisk.
asterisk -rx "core show calls"	Muestra cuántas llamadas hay activas.
asterisk -rx "sip show registry"	Muestra el estado de los registros SIP externos.
7.	Utilidades adicionales
Comando	Descripción
fwconsole reload	Aplica cambios sin reiniciar.
fwconsole ma list	Lista los módulos instalados de FreePBX.
fwconsole ma upgradeall	Actualiza todos los módulos disponibles.
fwconsole chown	Corrige permisos de archivos FreePBX.
fwconsole notifications	Muestra notificaciones pendientes.

Monitoreo en consola:
Propósito	Comando
Ver llamadas activas	asterisk -rx "core show channels"
Ver extensiones activas	asterisk -rx "pjsip show endpoints"
Logs en tiempo real	tail -f /var/log/asterisk/full
Recursos del sistema	top o htop
Puertos abiertos	netstat -tulnp
Validar integridad FreePBX	fwconsole validate


d) Activar Fail2Ban
“Fail2Ban detecta intentos de acceso sospechosos y bloquea automáticamente las IPs que fallan muchas veces.

Lo activamos con:
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
sudo fail2ban-client status
Para ver las IPs bloqueadas:
sudo fail2ban-client status asterisk-iptables
Y si queremos desbloquear una:
sudo fail2ban-client set asterisk-iptables unbanip 192.168.1.20
Con esto, el sistema se defiende automáticamente contra ataques de fuerza bruta.”
