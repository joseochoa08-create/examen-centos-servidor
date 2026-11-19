# EXAMEN INTERCICLO PRÁCTICO
## CONFIGURACIÓN DE SERVIDOR CENTOS STREAM 10
**Herramientas:** `CENTOS` | `POWERSHELL` | `FILEZILLA`

* **Sistema operativo:** CentOS Stream 10
* **Virtualización:** VirtualBox
* **Estudiante:** JOSE JULIAN OCHOA (`jose_ochoa`)
* **Servidor:** `servidor-centos`

### ¿Cómo identificar dónde ejecutar cada instrucción?
* **CENTOS:** Ejecutar el comando dentro de la máquina virtual CentOS.
* **POWERSHELL:** Ejecutar el comando en PowerShell de Windows.
* **FILEZILLA:** Realizar la configuración en la interfaz gráfica de FileZilla.

---

## 1. PREPARACIÓN DE VIRTUALBOX (Configuración Principal)
Apagar la máquina virtual y configurar la red:

* **Adaptador 1:**
  * Habilitar adaptador de red.
  * Conectado a: **Adaptador puente (Bridged Adapter)**.
  * Nombre: Seleccionar la tarjeta de red activa de la PC (Wi-Fi o Ethernet).
* **Adaptador 2:** Deshabilitado.

Iniciar CentOS.

*(Nota: Si en la universidad el Wi-Fi bloquea la comunicación entre equipos locales, ir al **Plan B** al final de esta guía para usar NAT + Solo-anfitrión).*

---

## 2. IDENTIFICACIÓN DEL SERVIDOR
**CENTOS**
```bash
cat /etc/centos-release
whoami
hostnamectl
sudo hostnamectl set-hostname servidor-centos
ip -br addr
ip route
```
Identificar la dirección IP asignada a la interfaz principal (ej. `enp0s3`). Esa única dirección (`<IP_DEL_SERVIDOR>`) servirá para salir a Internet y para conectarse desde Windows (SSH, SCP, Apache y FileZilla).

> **Si la interfaz aparece `DOWN` o sin IP:**
> ```bash
> sudo nmcli connection up enp0s3
> ip -br addr
> ```

---

## 3. INSTALACIÓN DE HERRAMIENTAS
**CENTOS**
```bash
sudo dnf update -y
sudo dnf install -y openssh-server openssh-clients httpd vsftpd tree firewalld policycoreutils-python-utils curl
sudo systemctl enable --now firewalld
sudo systemctl status firewalld
```
*(Presionar `q` para salir de la pantalla de estado).*

---

## 4. CONECTIVIDAD A INTERNET Y PING
**CENTOS**
```bash
ping -c 10 8.8.8.8
ping -c 10 google.com
```
La opción `-c 10` envía diez paquetes para comprobar la conectividad hacia Internet.

---

## 5. CONFIGURACIÓN DEL SERVIDOR SSH

### 5.1. Activar SSH
**CENTOS**
```bash
sudo systemctl enable --now sshd
sudo systemctl status sshd
```

### 5.2. Crear copia de seguridad
**CENTOS**
```bash
sudo cp -a /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

### 5.3. Cambiar el puerto SSH
**CENTOS**
```bash
sudo nano /etc/ssh/sshd_config
```
Configurar el puerto personalizado (quitar `#` o ajustar la línea `Port 22` para evitar conflictos):
```text
Port <PUERTO>
```
*(Ejemplo: `Port 2222`).* Guardar con `Ctrl + O`, `Enter` y salir con `Ctrl + X`.

### 5.4. Configurar SELinux
**CENTOS**
```bash
sudo semanage port -l | grep ssh_port_t
```
Si el puerto `<PUERTO>` no aparece asociado a SSH:
```bash
sudo semanage port -a -t ssh_port_t -p tcp <PUERTO>
```
*(Si ya aparece asociado a `ssh_port_t`, no repetir el comando).*

### 5.5. Abrir el puerto en el firewall
Consultar las zonas activas:
**CENTOS**
```bash
sudo firewall-cmd --get-active-zones
```
Sustituir `<ZONA>` por la zona activa (normalmente `public`):
```bash
sudo firewall-cmd --permanent --zone=<ZONA> --add-port=<PUERTO>/tcp
sudo firewall-cmd --reload
```

### 5.6. Validar y reiniciar SSH
**CENTOS**
```bash
sudo sshd -t
sudo systemctl restart sshd
sudo ss -tlnp | grep <PUERTO>
```

### 5.7. Probar conexión desde Windows
**POWERSHELL**
```powershell
ssh -p <PUERTO> <USUARIO>@<IP_DEL_SERVIDOR>
```
*(Ejemplo: `ssh -p 2222 jochoa@192.168.0.9`).* Sustituir el usuario, puerto e IP por los datos reales. Tomar captura de pantalla y escribir `exit` para volver a Windows.

---

## 6. TRANSFERENCIA DE ARCHIVOS CON SCP

### 6.1. Comprobar SCP en Windows
**POWERSHELL**
```powershell
Get-Command scp
```
*(Si no está instalado, abrir PowerShell como administrador y ejecutar: `Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0`).*

### 6.2. Crear archivo en Windows
**POWERSHELL**
```powershell
New-Item -ItemType Directory -Force C:\prueba-scp
Set-Location C:\prueba-scp
"Archivo enviado desde Windows hacia CentOS" | Out-File -Encoding utf8 prueba.txt
Get-ChildItem
```

### 6.3. Enviar archivo a CentOS
**POWERSHELL** *(Nota: `-P` en mayúscula)*
```powershell
scp -P <PUERTO> C:\prueba-scp\prueba.txt <USUARIO>@<IP_DEL_SERVIDOR>:/home/<USUARIO>/
```

### 6.4. Comprobar archivo recibido
**CENTOS**
```bash
ls -l /home/<USUARIO>/
cat /home/<USUARIO>/prueba.txt
```

### 6.5. Descargar archivo a Windows
**POWERSHELL**
```powershell
scp -P <PUERTO> <USUARIO>@<IP_DEL_SERVIDOR>:/home/<USUARIO>/prueba.txt C:\prueba-scp\prueba_descargada.txt
Get-ChildItem C:\prueba-scp
```

---

## 7. ESTRUCTURA DE DIRECTORIOS DE LINUX

### 7.1. Mostrar estructura de directorios
**CENTOS**
```bash
sudo tree -L 2 -d /
```

### 7.2. Función de los directorios principales

| Directorio | Función |
| :--- | :--- |
| `/` | Directorio raíz del sistema. |
| `/bin` | Comandos y binarios esenciales. |
| `/boot` | Archivos de arranque y kernel. |
| `/dev` | Archivos de dispositivos de hardware. |
| `/etc` | Archivos de configuración del sistema. |
| `/home` | Carpetas personales de los usuarios. |
| `/root` | Carpeta personal del administrador (`root`). |
| `/lib` | Bibliotecas esenciales del sistema. |
| `/media` | Montaje automático de medios extraíbles. |
| `/mnt` | Punto de montaje temporal. |
| `/opt` | Software adicional de terceros. |
| `/proc` | Información virtual de procesos y kernel. |
| `/run` | Datos temporales de ejecución del sistema. |
| `/srv` | Datos de servicios alojados en el servidor. |
| `/tmp` | Archivos temporales. |
| `/usr` | Programas, librerías y recursos de usuario. |
| `/var` | Registros (`logs`) y datos variables. |

*(Nota: En CentOS moderno, directorios como `/bin` y `/lib` son enlaces simbólicos hacia `/usr`).*

---

## 8. INSTALACIÓN Y CONFIGURACIÓN DE APACHE

### 8.1. Instalar Apache y crear copia de seguridad
**CENTOS**
```bash
sudo dnf install -y httpd
sudo cp -a /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.backup
sudo systemctl enable --now httpd
```

### 8.2. Permitir HTTP en el firewall
**CENTOS**
```bash
sudo firewall-cmd --permanent --zone=<ZONA> --add-service=http
sudo firewall-cmd --reload
```

### 8.3. Crear la página web personalizada
**CENTOS**
```bash
sudo nano /var/www/html/index.html
```
Contenido de la página (incluye `nombre_apellido` según rúbrica):
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Servidor Apache</title>
</head>
<body>
    <h1>Servidor Apache funcionando correctamente</h1>
    <h2>Estudiante: JOSE JULIAN OCHOA (jose_ochoa)</h2>
    <p>Sistema operativo: CentOS Stream 10</p>
    <p>Práctica de configuración de servidor web.</p>
</body>
</html>
```
Guardar con `Ctrl + O`, `Enter` y salir con `Ctrl + X`.

### 8.4. Aplicar seguridad y reiniciar Apache
**CENTOS**
```bash
sudo restorecon -Rv /var/www/html
sudo systemctl restart httpd
curl http://localhost
```

### 8.5. Probar página desde Windows
Abrir el navegador en Windows y escribir:
```text
http://<IP_DEL_SERVIDOR>
```

---

## 9. INSTALACIÓN Y CONFIGURACIÓN DE FTP

### 9.1. Instalar FTP y guardar la configuración original
**CENTOS**
```bash
sudo dnf install -y vsftpd
sudo cp -a /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.backup
```

### 9.2. Crear usuario FTP
**CENTOS**
```bash
sudo useradd -m ftpusuario
sudo passwd ftpusuario
```
*(Comprobar usuario con `id ftpusuario`).*

### 9.3. Configurar `vsftpd`
**CENTOS**
```bash
sudo nano /etc/vsftpd/vsftpd.conf
```
Comprobar que estas directivas tengan estos valores (modificar las existentes, **sin duplicarlas**, especialmente `listen` y `listen_ipv6`):
```text
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
chroot_local_user=YES
allow_writeable_chroot=YES
listen=YES
listen_ipv6=NO
pasv_enable=YES
pasv_min_port=30000
pasv_max_port=30100
```
Guardar con `Ctrl + O`, `Enter` y salir con `Ctrl + X`.

### 9.4. Configurar SELinux (CentOS Stream 10)
**CENTOS**
```bash
sudo setsebool -P ftpd_full_access on
getsebool ftpd_full_access
```
Debe aparecer: `ftpd_full_access --> on`.

### 9.5. Abrir puertos en el firewall
**CENTOS**
```bash
sudo firewall-cmd --permanent --zone=<ZONA> --add-service=ftp
sudo firewall-cmd --permanent --zone=<ZONA> --add-port=30000-30100/tcp
sudo firewall-cmd --reload
```

### 9.6. Activar FTP
**CENTOS**
```bash
sudo systemctl enable --now vsftpd
sudo systemctl restart vsftpd
sudo systemctl status vsftpd
sudo ss -tlnp | grep ':21'
```
(Presionar q para salir).

10. CONEXIÓN Y TRANSFERENCIA CON FILEZILLA

FILEZILLA
Abrir FileZilla en Windows y configurar la barra de conexión rápida:

Campo	Valor
Protocolo	FTP (en Gestor de Sitios: Usar FTP plano / sin cifrado)
Servidor	<IP_DEL_SERVIDOR>
Puerto	21
Modo de acceso	Normal
Usuario	ftpusuario
Contraseña	Contraseña creada para ftpusuario

(Si da error al listar carpetas, ir a Edición > Opciones > FTP y confirmar que esté marcado Pasivo).

10.1. Subir un archivo

1. Buscar un archivo en el panel izquierdo (Windows).
2. En el panel derecho localizar la carpeta del usuario FTP (/home/ftpusuario).
3. Arrastrar el archivo desde Windows hacia CentOS y verificar la transferencia.

10.2. Descargar un archivo

1. Seleccionar un archivo en el panel derecho (CentOS).
2. Elegir una carpeta de destino en el panel izquierdo (Windows).
3. Arrastrar el archivo hacia Windows y comprobar que se descargó.

11. COMPROBAR COPIAS DE SEGURIDAD Y ESTADO FINAL

CENTOS
bash sudo ls -l /etc/ssh/sshd_config.backup /etc/httpd/conf/httpd.conf.backup /etc/vsftpd/vsftpd.conf.backup sudo systemctl status sshd httpd vsftpd firewalld sudo ss -tlnp 

Lista de verificación de evidencias:

• Conectividad: Ping de 10 paquetes (ping -c 10 8.8.8.8).
• SSH: Acceso remoto desde Windows usando IP y puerto personalizado (<PUERTO>).
• SCP: Envío de archivo de Windows a CentOS y descarga de CentOS a Windows.
• Estructura: Comando tree desde / y explicación de directorios.
• Apache: Página web visible desde navegador de Windows con nombre_apellido.
• FTP: Conexión en FileZilla con usuario/contraseña, subida y descarga de archivos.

PLAN B: SI EL WI-FI DE LA UNIVERSIDAD BLOQUEA EL MODO PUENTE

Si al usar Adaptador Puente tienes internet (ping 8.8.8.8), pero Windows no logra conectarse por SSH/Apache/FileZilla a la IP de CentOS:
1. Apagar CentOS y en VirtualBox configurar:
	• Adaptador 1: Habilitado > NAT (para internet).
	• Adaptador 2: Habilitado > Adaptador solo-anfitrión (Host-only) (para Windows).
2. Encender CentOS y revisar la IP de la segunda interfaz (enp0s8) con ip -br addr.
3. Si enp0s8 aparece sin IPv4, asignarle una fija:
   bash sudo nmcli connection modify enp0s8 ipv4.method manual ipv4.addresses 192.168.56.101/24 sudo nmcli connection up enp0s8    
4. Usar 192.168.56.101 como <IP_DEL_SERVIDOR> desde Windows (SSH, SCP, Navegador y FileZilla).
