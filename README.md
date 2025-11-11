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
