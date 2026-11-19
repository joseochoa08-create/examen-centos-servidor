# EXAMEN INTERCICLO PRÁCTICO
## CONFIGURACIÓN DE SERVIDOR CENTOS STREAM 10

![CENTOS](https://img.shields.io/badge/EJECUTAR_EN-CENTOS-blue?style=for-the-badge)
![POWERSHELL](https://img.shields.io/badge/EJECUTAR_EN-POWERSHELL-green?style=for-the-badge)
![FILEZILLA](https://img.shields.io/badge/EJECUTAR_EN-FILEZILLA-purple?style=for-the-badge)

**Sistema operativo:** CentOS Stream 10  
**Virtualización:** VirtualBox  
**Estudiante:** JOSE JULIAN OCHOA  
**Servidor:** servidor-centos

### ¿Cómo identificar dónde ejecutar cada instrucción?

- ![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square): ejecutar el comando dentro de la máquina virtual CentOS.
- ![POWERSHELL](https://img.shields.io/badge/POWERSHELL-Windows-green?style=flat-square): ejecutar el comando en PowerShell de Windows.
- ![FILEZILLA](https://img.shields.io/badge/FILEZILLA-Cliente_FTP-purple?style=flat-square): realizar la configuración en FileZilla.

---

# 1. PREPARACIÓN DE VIRTUALBOX

Apagar la máquina virtual y configurar los adaptadores de red:

**Adaptador 1:**
- Habilitar adaptador.
- Conectado a: NAT.

**Adaptador 2:**
- Habilitar adaptador.
- Conectado a: Adaptador solo-anfitrión (Host-only).

Iniciar CentOS.

# 2. IDENTIFICACIÓN DEL SERVIDOR

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
cat /etc/centos-release
whoami
hostnamectl
sudo hostnamectl set-hostname servidor-centos
ip -br addr
ip route
```

Identificar la dirección IP de la interfaz Host-only. Esa dirección se utilizará desde Windows para conectarse al servidor.

# 3. INSTALACIÓN DE HERRAMIENTAS

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
sudo dnf update -y
sudo dnf install -y openssh-server openssh-clients httpd vsftpd tree firewalld policycoreutils-python-utils curl
sudo systemctl enable --now firewalld
sudo systemctl status firewalld
```

Presionar `q` para salir de la pantalla de estado.

# 4. CONECTIVIDAD A INTERNET Y PING

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
ping -c 10 8.8.8.8
ping -c 10 google.com
```

La opción `-c 10` envía diez paquetes para comprobar la conectividad.

# 5. CONFIGURACIÓN DEL SERVIDOR SSH

## 5.1. Activar SSH

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
sudo systemctl enable --now sshd
sudo systemctl status sshd
```

## 5.2. Crear copia de seguridad

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
sudo cp -a /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

## 5.3. Cambiar el puerto SSH

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
sudo nano /etc/ssh/sshd_config
```

Configurar el puerto:

```text
Port 2222
```

Si existe una línea activa `Port 22`, comentarla o ajustarla para evitar conflictos.

Guardar con `Ctrl + O`, `Enter` y salir con `Ctrl + X`.

## 5.4. Configurar SELinux

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
sudo semanage port -l | grep ssh_port_t
```

Si el puerto 2222 no aparece asociado a SSH:

```bash
sudo semanage port -a -t ssh_port_t -p tcp 2222
```

Si ya aparece asociado a `ssh_port_t`, no repetir el comando.

## 5.5. Abrir el puerto en el firewall

Consultar las zonas activas:

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
sudo firewall-cmd --get-active-zones
```

Sustituir `<ZONA>` por la zona correspondiente a la interfaz Host-only:

```bash
sudo firewall-cmd --permanent --zone=<ZONA> --add-port=2222/tcp
sudo firewall-cmd --reload
```

## 5.6. Validar y reiniciar SSH

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
sudo sshd -t
sudo systemctl restart sshd
sudo ss -tlnp | grep 2222
```

## 5.7. Probar conexión desde Windows

![POWERSHELL](https://img.shields.io/badge/POWERSHELL-Windows-green?style=flat-square)

```powershell
ssh -p 2222 USUARIO@IP_DEL_SERVIDOR
```

Ejemplo:

```powershell
ssh -p 2222 jose@192.168.56.101
```

Sustituir el usuario y la IP por los datos reales.

# 6. TRANSFERENCIA DE ARCHIVOS CON SCP

## 6.1. Comprobar SCP en Windows

![POWERSHELL](https://img.shields.io/badge/POWERSHELL-Windows-green?style=flat-square)

```powershell
Get-Command scp
```

Si no está instalado, abrir PowerShell como administrador:

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
```

## 6.2. Crear archivo en Windows

![POWERSHELL](https://img.shields.io/badge/POWERSHELL-Windows-green?style=flat-square)

```powershell
New-Item -ItemType Directory -Force C:\prueba-scp
Set-Location C:\prueba-scp
"Archivo enviado desde Windows hacia CentOS" | Out-File -Encoding utf8 prueba.txt
Get-ChildItem
```

## 6.3. Enviar archivo a CentOS

![POWERSHELL](https://img.shields.io/badge/POWERSHELL-Windows-green?style=flat-square)

```powershell
scp -P 2222 C:\prueba-scp\prueba.txt USUARIO@IP_DEL_SERVIDOR:/home/USUARIO/
```

## 6.4. Comprobar archivo recibido

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
ls -l /home/USUARIO/
cat /home/USUARIO/prueba.txt
```

## 6.5. Descargar archivo a Windows

![POWERSHELL](https://img.shields.io/badge/POWERSHELL-Windows-green?style=flat-square)

```powershell
scp -P 2222 USUARIO@IP_DEL_SERVIDOR:/home/USUARIO/prueba.txt C:\prueba-scp\
Get-ChildItem C:\prueba-scp
```

# 7. ESTRUCTURA DE DIRECTORIOS DE LINUX

## 7.1. Instalar tree

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
sudo dnf install -y tree
```

## 7.2. Mostrar estructura de directorios

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
sudo tree -L 2 -d /
```

## 7.3. Función de los directorios

| Directorio | Función |
|---|---|
| `/` | Directorio raíz. |
| `/bin` | Comandos esenciales. |
| `/boot` | Archivos de arranque. |
| `/dev` | Archivos de dispositivos. |
| `/etc` | Configuraciones del sistema. |
| `/home` | Carpetas personales. |
| `/root` | Carpeta del administrador. |
| `/lib` | Bibliotecas del sistema. |
| `/media` | Montaje de medios extraíbles. |
| `/mnt` | Montaje temporal. |
| `/opt` | Software adicional. |
| `/proc` | Información de procesos y sistema. |
| `/run` | Datos temporales de ejecución. |
| `/srv` | Datos de servicios. |
| `/tmp` | Archivos temporales. |
| `/usr` | Programas y recursos. |
| `/var` | Registros y datos variables. |

Nota: en CentOS moderno, algunos directorios pueden ser enlaces simbólicos hacia directorios dentro de `/usr`.

# 8. INSTALACIÓN Y CONFIGURACIÓN DE APACHE

## 8.1. Instalar Apache y crear copia de seguridad

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
sudo dnf install -y httpd
sudo cp -a /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.backup
sudo systemctl enable --now httpd
```

## 8.2. Permitir HTTP en el firewall

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
sudo firewall-cmd --permanent --zone=<ZONA> --add-service=http
sudo firewall-cmd --reload
```

## 8.3. Crear la página web

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
sudo nano /var/www/html/index.html
```

Contenido de la página:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Servidor Apache</title>
</head>
<body>
    <h1>Servidor Apache funcionando correctamente</h1>
    <h2>Estudiante: JOSE JULIAN OCHOA</h2>
    <p>Sistema operativo: CentOS Stream 10</p>
    <p>Práctica de configuración de servidor web.</p>
</body>
</html>
```

Guardar con `Ctrl + O`, `Enter` y `Ctrl + X`.

## 8.4. Aplicar seguridad y reiniciar Apache

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
sudo restorecon -Rv /var/www/html
sudo systemctl restart httpd
curl http://localhost
```

## 8.5. Probar página desde Windows

![POWERSHELL](https://img.shields.io/badge/POWERSHELL-Windows-green?style=flat-square)

Abrir el navegador de Windows y escribir:

```text
http://IP_DEL_SERVIDOR
```

Ejemplo:

```text
http://192.168.56.101
```

Debe aparecer la página creada.

# 9. INSTALACIÓN Y CONFIGURACIÓN DE FTP

## 9.1. Instalar FTP y guardar configuración original

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
sudo dnf install -y vsftpd
sudo cp -a /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.backup
```

## 9.2. Crear usuario FTP

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
sudo useradd -m ftpusuario
sudo passwd ftpusuario
```

Si el usuario ya existe, comprobarlo:

```bash
id ftpusuario
```

## 9.3. Configurar vsftpd

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
sudo nano /etc/vsftpd/vsftpd.conf
```

Comprobar que estas directivas tengan estos valores. Modificar las existentes, sin duplicarlas:

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

Guardar con `Ctrl + O`, `Enter` y `Ctrl + X`.

## 9.4. Configurar SELinux

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
sudo setsebool -P ftp_home_dir on
getsebool ftp_home_dir
```

Debe aparecer:

```text
ftp_home_dir --> on
```

## 9.5. Abrir puertos en el firewall

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
sudo firewall-cmd --permanent --zone=<ZONA> --add-service=ftp
sudo firewall-cmd --permanent --zone=<ZONA> --add-port=30000-30100/tcp
sudo firewall-cmd --reload
```

## 9.6. Activar FTP

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
sudo systemctl enable --now vsftpd
sudo systemctl restart vsftpd
sudo systemctl status vsftpd
sudo ss -tlnp | grep ':21'
```

Presionar `q` para salir.

# 10. CONEXIÓN Y TRANSFERENCIA CON FILEZILLA

![FILEZILLA](https://img.shields.io/badge/FILEZILLA-Cliente_FTP-purple?style=flat-square)

Abrir FileZilla en Windows y configurar:

| Campo | Valor |
|---|---|
| Protocolo | FTP |
| Servidor | IP Host-only de CentOS |
| Puerto | 21 |
| Modo de acceso | Normal |
| Usuario | ftpusuario |
| Contraseña | Contraseña creada para ftpusuario |

Seleccionar FTP simple sin cifrado únicamente para esta práctica dentro de la red Host-only aislada. No utilizar FTP simple para información sensible en Internet.

Utilizar el modo de transferencia pasivo.

## 10.1. Subir un archivo

1. Buscar un archivo en el panel izquierdo (Windows).
2. En el panel derecho localizar la carpeta del usuario FTP.
3. Arrastrar el archivo desde Windows hacia CentOS.
4. Comprobar que la transferencia finalizó.

## 10.2. Descargar un archivo

1. Seleccionar un archivo en el panel derecho (CentOS).
2. Elegir una carpeta de destino en el panel izquierdo (Windows).
3. Arrastrar el archivo hacia Windows.
4. Comprobar que se descargó.

# 11. COMPROBAR COPIAS DE SEGURIDAD

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
sudo ls -l /etc/ssh/sshd_config.backup
sudo ls -l /etc/httpd/conf/httpd.conf.backup
sudo ls -l /etc/vsftpd/vsftpd.conf.backup
```

# 12. COMPROBACIÓN FINAL

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
sudo systemctl status sshd
sudo systemctl status httpd
sudo systemctl status vsftpd
sudo systemctl status firewalld
sudo ss -tlnp
```

Presionar `q` para salir de las pantallas de estado.

Resultados esperados:

- Internet funcionando.
- Ping de diez paquetes.
- SSH funcionando por el puerto 2222.
- SCP enviando y descargando archivos.
- Estructura de directorios visible.
- Apache mostrando la página personalizada.
- FTP conectado mediante FileZilla.
- Subida y descarga FTP funcionando.
- Copias de seguridad creadas.

# 13. SOLUCIÓN DE ERRORES COMUNES

## Error: CentOS no tiene Internet

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
ip -br addr
ip route
nmcli device status
```

Comprobar que el Adaptador 1 de VirtualBox esté en NAT.

## Error: SSH no funciona

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
sudo systemctl status sshd
sudo sshd -t
sudo ss -tlnp | grep 2222
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --list-all
```

## Error: SCP no funciona

![POWERSHELL](https://img.shields.io/badge/POWERSHELL-Windows-green?style=flat-square)

```powershell
Get-ChildItem C:\prueba-scp
```

Comprobar que SSH funciona, que el usuario es correcto y que la IP corresponde a la interfaz Host-only.

## Error: Apache no muestra la página

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
sudo systemctl status httpd
sudo apachectl configtest
sudo firewall-cmd --list-all
sudo systemctl restart httpd
```

## Error: FTP no funciona

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
sudo systemctl status vsftpd
sudo ss -tlnp | grep ':21'
getsebool ftp_home_dir
sudo firewall-cmd --list-all
id ftpusuario
```

## Error: Windows no se conecta con CentOS

![CENTOS](https://img.shields.io/badge/CENTOS-Terminal-blue?style=flat-square)

```bash
ip -br addr
```

Verificar que Windows y CentOS estén conectados a la misma red Host-only y utilizar la IP correcta.

# 14. CONCLUSIÓN

Durante esta práctica se instaló y configuró un servidor CentOS Stream 10 en VirtualBox. Se comprobó la conectividad a Internet, se configuró SSH, se realizaron transferencias mediante SCP, se examinó la estructura de directorios de Linux y se instaló Apache. Finalmente, se configuró FTP y se comprobó la transferencia de archivos mediante FileZilla.

La práctica permitió aplicar comandos de administración de Linux y comprobar el funcionamiento de diferentes servicios de red.

---

# RECOMENDACIONES PARA EL EXAMEN

1. Seguir todos los apartados en orden.
2. Ejecutar cada comando en el lugar indicado por su distintivo.
3. Sustituir `USUARIO` por el usuario real de CentOS.
4. Sustituir `IP_DEL_SERVIDOR` por la IP Host-only real.
5. Sustituir `<ZONA>` por la zona real de la interfaz Host-only.
6. No cerrar la sesión original antes de comprobar SSH.
7. Realizar las pruebas desde Windows.
8. Conservar las copias de seguridad.
9. Si un comando presenta un error, detenerse y revisar el mensaje.
10. Mantener CentOS encendido mientras se realizan las pruebas.

**GitHub es la guía de consulta. Los comandos se ejecutan en CentOS y Windows, y las transferencias FTP se comprueban en FileZilla.**
