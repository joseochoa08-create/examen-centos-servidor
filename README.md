# EXAMEN INTERCICLO PRÁCTICO — CENTOS STREAM 10

![CENTOS](https://shields.io)
![POWERSHELL](https://shields.io)
![FILEZILLA](https://shields.io)

* **Estudiante:** JOSE JULIAN OCHOA (`jose_ochoa`)
* **Morado (`CENTOS`):** Terminal de CentOS.
* **Azul (`POWERSHELL`):** PowerShell en Windows.
* **Verde (`FILEZILLA / NAVEGADOR`):** En Windows.
* **Regla:** Reemplaza `TU_USUARIO`, `TU_IP` y `TU_PUERTO` directamente por tus datos reales (sin signos extra).

---

## PREPARACIÓN INICIAL (Antes de empezar los 6 puntos)

### A. Red en VirtualBox
Apagar CentOS > **Configuración > Red > Adaptador 1** > Cambiar a **Adaptador puente** (*Bridged Adapter*) > Encender CentOS.

### B. Ver tu IP y tu Usuario (En terminal de CentOS)
> [!IMPORTANT]
> **CENTOS** — Anota el nombre que sale en `whoami` (`TU_USUARIO`) y la IP que sale al lado de `enp0s3` (`TU_IP`).

```bash
whoami
ip -br addr
```
*(Si `enp0s3` sale `DOWN` o sin IP, ejecuta: `sudo nmcli connection up enp0s3` y vuelve a poner `ip -br addr`).*

### C. Instalar todos los programas de una vez
> [!IMPORTANT]
> **CENTOS**

```bash
sudo dnf install -y openssh-server openssh-clients httpd vsftpd tree firewalld policycoreutils-python-utils curl
sudo systemctl enable --now firewalld
```

---

## PUNTO 1: CONECTIVIDAD (Salida a Internet)
> [!IMPORTANT]
> **CENTOS** — Tomar captura de pantalla mostrando los 10 paquetes recibidos (`0% packet loss`).

```bash
ping -c 10 8.8.8.8
```

---

## PUNTO 2: SERVIDOR SSH (IP y Puerto Personalizado)

### 2.1. Configurar puerto en CentOS
> [!IMPORTANT]
> **CENTOS** — Abrir el archivo, cambiar `Port 22` por `Port TU_PUERTO` (ej. `Port 2222`), guardar con `Ctrl + O`, `Enter` y salir con `Ctrl + X`.

```bash
sudo nano /etc/ssh/sshd_config
```

### 2.2. Abrir puerto en SELinux, Firewall y validar SSH
> [!IMPORTANT]
> **CENTOS** — Reemplaza `TU_PUERTO` por el número limpio (ejemplo: `2222`). Fíjate que en `semanage` va solo el número y en `firewall-cmd` lleva `/tcp`.

```bash
sudo semanage port -a -t ssh_port_t -p tcp TU_PUERTO
sudo firewall-cmd --permanent --zone=public --add-port=TU_PUERTO/tcp
sudo firewall-cmd --reload
sudo sshd -t
sudo systemctl restart sshd
sudo ss -tlnp | grep TU_PUERTO
```
*(Nota: `sudo sshd -t` comprueba que no haya errores de escritura; si no muestra nada, todo está bien).*

### 2.3. Conectarse desde Windows (Evidencia)
> [!NOTE]
> **POWERSHELL** — Ingresa tu contraseña, **toma captura dentro del servidor** y luego escribe `exit`.

```powershell
ssh -p TU_PUERTO TU_USUARIO@TU_IP
```

---

## PUNTO 3: CONEXIÓN POR SCP (Subida y Descarga)
> [!NOTE]
> **POWERSHELL** — Nota que en `scp` se usa **`-P` mayúscula**. Toma captura de ambas transferencias al `100%`.

### 3.1. Crear archivo de prueba en Windows
```powershell
New-Item -ItemType Directory -Force C:\prueba-scp
Set-Location C:\prueba-scp
"Archivo de prueba" | Out-File -Encoding utf8 prueba.txt
```

### 3.2. Subir archivo de Windows a CentOS (Ejecutar primero)
```powershell
scp -P TU_PUERTO C:\prueba-scp\prueba.txt TU_USUARIO@TU_IP:/home/TU_USUARIO/
```

### 3.3. Descargar archivo de CentOS a Windows (Ejecutar segundo)
```powershell
scp -P TU_PUERTO TU_USUARIO@TU_IP:/home/TU_USUARIO/prueba.txt C:\prueba-scp\descargado.txt
Get-ChildItem C:\prueba-scp
```


---

## PUNTO 4: ESTRUCTURA DE DIRECTORIOS DE LINUX
> [!IMPORTANT]
> **CENTOS** — Toma captura del árbol y explica los directorios principales.

```bash
sudo tree -L 2 -d /
```

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
| `/media` | Montaje de medios extraíbles (USB, CD). |
| `/mnt` | Punto de montaje temporal. |
| `/opt` | Software adicional de terceros. |
| `/proc` | Información virtual de procesos y kernel. |
| `/run` | Datos temporales de ejecución del sistema. |
| `/srv` | Datos de servicios alojados en el servidor. |
| `/tmp` | Archivos temporales. |
| `/usr` | Programas, librerías y recursos de usuario. |
| `/var` | Registros (`logs`) y datos variables. |

---

## PUNTO 5: SERVIDOR WEB APACHE (`httpd`)

### 5.1. Iniciar Apache, abrir Firewall y editar la página
> [!IMPORTANT]
> **CENTOS** — Guardar con `Ctrl + O`, `Enter` y salir con `Ctrl + X`.

```bash
sudo systemctl enable --now httpd
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --reload
sudo nano /var/www/html/index.html
```

Pegar dentro del archivo:
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>HTTP Server Test Page - jose_ochoa</title>
</head>
<body>
    <h1>HTTP Server Test Page - jose_ochoa (JOSE JULIAN OCHOA)</h1>
    <p>Servidor Apache funcionando correctamente en CentOS Stream 10.</p>
</body>
</html>
```

Aplicar permisos SELinux y reiniciar Apache:
```bash
sudo restorecon -Rv /var/www/html

sudo systemctl restart httpd
```

### 5.2. Comprobar en el navegador de Windows (Evidencia)
> [!TIP]
> **NAVEGADOR DE WINDOWS** — Entra a la dirección de tu servidor y toma captura donde se vea **`jose_ochoa`**:

```text
http://TU_IP
```

```

---

## PUNTO 6: SERVIDOR FTP (`vsftpd`) + FILEZILLA

### 6.1. Crear usuario y configurar `vsftpd`
> [!IMPORTANT]
> **CENTOS**

```bash
sudo cp -a /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.backup
sudo useradd -m ftpusuario
sudo passwd ftpusuario
sudo nano /etc/vsftpd/vsftpd.conf
```
Asegurar que estas líneas tengan estos valores (modificar las existentes **sin duplicarlas**, revisar `listen` y `listen_ipv6`):
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

### 6.2. Configurar SELinux, Firewall y activar FTP
> [!IMPORTANT]
> **CENTOS**

```bash
sudo setsebool -P ftpd_full_access on
sudo firewall-cmd --permanent --zone=public --add-service=ftp
sudo firewall-cmd --permanent --zone=public --add-port=30000-30100/tcp
sudo firewall-cmd --reload
sudo systemctl enable --now vsftpd
sudo systemctl restart vsftpd
```

### 6.3. Subir y descargar en FileZilla (Evidencia)
> [!TIP]
> **FILEZILLA (EN WINDOWS)** — Llena la barra superior y haz clic en **Conexión rápida** *(si falla al listar carpetas, activar modo **Pasivo** en Edición > Opciones > FTP)*:

* **Servidor:** `TU_IP`
* **Nombre de usuario:** `ftpusuario`
* **Contraseña:** *(la que creaste en el paso 6.1)*
* **Puerto:** `21`

Arrastra un archivo de izquierda a derecha (**subir**) y otro de derecha a izquierda (**descargar**), y toma captura.

---

## RESPALDO: SI EL WI-FI DE LA UNIVERSIDAD BLOQUEA EL MODO PUENTE
> [!CAUTION]
> Usar **solo** si tienes Internet en CentOS (`ping 8.8.8.8`), pero Windows no conecta a la IP puente:
> 1. Apagar CentOS > VirtualBox > **Adaptador 1:** `NAT` | **Adaptador 2:** `Adaptador solo-anfitrión (Host-only)`.
> 2. Encender CentOS y ejecutar:
> ```bash
> sudo nmcli connection modify enp0s8 ipv4.method manual ipv4.addresses 192.168.56.101/24
> sudo nmcli connection up enp0s8
> ```
> 3. Usar `192.168.56.101` como `TU_IP` desde Windows.
