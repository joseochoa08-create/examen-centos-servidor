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

### 5.1. Iniciar Apache, abrir Firewall y editar la página oficial de CentOS
> [!IMPORTANT]
> **CENTOS** — Inserta automáticamente tu **`jose_ochoa`** dentro de la página oficial original de prueba de CentOS sin perder sus colores ni diseño.

```bash
sudo systemctl enable --now httpd
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --reload
sudo rm -f /var/www/html/index.html
sudo sed -i 's/CentOS/CentOS - jose_ochoa (JOSE JULIAN OCHOA)/g' /usr/share/httpd/noindex/index.html
sudo restorecon -Rv /usr/share/httpd/noindex
sudo systemctl restart httpd
```

> **Si prefieres crear un archivo `index.html` nuevo desde cero en `/var/www/html/index.html`:**
> ```bash
> sudo nano /var/www/html/index.html
> ```
> Pegar este código, guardar con `Ctrl + O`, `Enter` y salir con `Ctrl + X`:
> ```html
> <!DOCTYPE html>
> <html lang="es">
> <head>
>     <meta charset="UTF-8">
>     <title>HTTP Server Test Page - jose_ochoa</title>
> </head>
> <body>
>     <h1>HTTP Server Test Page - jose_ochoa (JOSE JULIAN OCHOA)</h1>
>     <p>Servidor Apache funcionando correctamente en CentOS Stream 10.</p>
> </body>
> </html>
> ```
> Aplicar permisos y reiniciar:
> ```bash
> sudo restorecon -Rv /var/www/html
> sudo systemctl restart httpd
> ```

### 5.2. Comprobar en el navegador de Windows (Evidencia)
> [!TIP]
> **NAVEGADOR DE WINDOWS** — Entra a la dirección de tu servidor, presiona `Ctrl + F5` y toma captura donde se vea **`jose_ochoa`**:

```text
http://TU_IP
```



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
> 4. # Implementación y Administración de Servicios de Red en Servidor Linux CentOS

**Autor:** [Tu Nombre y Apellido]  
**Institución:** [Nombre de tu Universidad / Carrera / Asignatura]  
**Contacto:** [tu.correo@ejemplo.com]  

---

## Resumen
En este informe se detalla la configuración paso a paso de un servidor basado en la distribución Linux CentOS dentro de un entorno virtualizado. El objetivo principal fue habilitar y comprobar servicios esenciales de red y administración remota. Se configuró la salida a Internet, el acceso remoto seguro mediante SSH en un puerto personalizado, la transferencia de archivos cifrada con SCP, el despliegue de un servidor web Apache personalizado y un servidor de archivos FTP verificado desde un cliente externo. Todas las pruebas demostraron una comunicación estable y segura entre la máquina anfitriona y el servidor virtual.

**Palabras clave:** CentOS, Administración de Servidores, SSH, SCP, Apache, FTP.

---

## 1. Introducción
La administración de servidores Linux es fundamental para gestionar servicios en red de forma segura y eficiente. En esta práctica de examen interciclo se despliega una máquina virtual con CentOS configurada como servidor, asegurando que pueda comunicarse con redes externas y ofrecer servicios básicos tanto de administración remota como de publicación web y transferencia de archivos.

## 2. Entorno y Requisitos Previos
Para realizar esta práctica se utilizó el siguiente entorno de trabajo:
* **Hipervisor:** [VirtualBox / VMware] con adaptador de red en modo **Adaptador Puente (Bridged)** para que la máquina virtual obtenga una IP propia dentro de la red local.
* **Sistema Operativo Servidor:** CentOS [7 / Stream 8 / Stream 9].
* **Sistema Operativo Cliente (Anfitrión):** [Windows 10/11 o Linux] con terminal y cliente **FileZilla**.

---

## 3. Desarrollo y Configuración de Servicios

### 3.1. Verificación de Conectividad a Internet
Para confirmar que el servidor tiene salida hacia redes externas, se verificó la configuración de red y se envió una prueba de 10 paquetes ICMP (`ping`) hacia un servidor público en Internet (DNS de Google).

**Comando ejecutado en CentOS:**
```bash
ping -c 10 8.8.8.8
```
* **Explicación:** El parámetro `-c 10` indica que se enviarán exactamente 10 paquetes y luego se detendrá mostrando el resumen de pérdida de paquetes (0% de pérdida confirma conectividad estable).

![Prueba de conectividad con 10 paquetes](img/fig1_ping.png)  
*Fig. 1. Resultado exitoso del envío de 10 paquetes hacia Internet desde el servidor CentOS.*

---

### 3.2. Configuración del Servidor SSH en Puerto Personalizado
Para administrar el servidor de forma remota sin usar el puerto por defecto (22) por motivos de seguridad, se modificó el archivo de configuración de SSH para escuchar en el puerto **2222** (puedes cambiarlo por el que elegiste).

**Pasos realizados en el servidor:**
1. Editar el archivo de configuración:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```
2. Descomentar y cambiar la línea del puerto:
   ```text
   Port 2222
   ```
3. Permitir el nuevo puerto en el firewall y SELinux (si está activo), y reiniciar el servicio:
   ```bash
   sudo firewall-cmd --permanent --add-port=2222/tcp
   sudo firewall-cmd --reload
   sudo semanage port -a -t ssh_port_t -p tcp 2222
   sudo systemctl restart sshd
   ```

**Prueba desde la máquina cliente:**
```bash
ssh -p 2222 [usuario_centos]@[IP_DEL_SERVIDOR]
```

![Conexión SSH por puerto personalizado](img/fig2_ssh.png)  
*Fig. 2. Acceso remoto mediante SSH utilizando dirección IP del servidor y puerto personalizado.*

---

### 3.3. Transferencia de Archivos mediante SCP
Utilizando el protocolo seguro SCP (que trabaja sobre SSH), se realizaron pruebas bidireccionales entre la máquina cliente y el servidor CentOS usando el puerto personalizado (`-P 2222`).

1. **Envío desde el cliente hacia el servidor:**
   ```bash
   scp -P 2222 archivo_cliente.txt [usuario_centos]@[IP_DEL_SERVIDOR]:/home/[usuario_centos]/
   ```
2. **Descarga desde el servidor hacia el cliente:**
   ```bash
   scp -P 2222 [usuario_centos]@[IP_DEL_SERVIDOR]:/home/[usuario_centos]/archivo_servidor.txt ./
   ```

![Transferencia de archivos con SCP](img/fig3_scp.png)  
*Fig. 3. Evidencia de subida y descarga de archivos entre cliente y servidor mediante SCP.*

---

### 3.4. Estructura de Archivos del Sistema Linux (`/`)
Para visualizar la jerarquía de directorios desde la raíz (`/`) hasta el primer nivel de profundidad, se utilizó el comando `tree`.

**Comandos ejecutados:**
```bash
sudo yum install tree -y
tree -L 1 /
```

| Directorio | Función principal dentro del sistema |
| :--- | :--- |
| **`/bin` y `/usr/bin`** | Contienen los comandos y programas básicos ejecutables para todos los usuarios (como `ls`, `cp`, `ping`). |
| **`/sbin` y `/usr/sbin`** | Almacenan comandos esenciales destinados a la administración del sistema por parte del usuario `root`. |
| **`/etc`** | Guarda todos los archivos de configuración del sistema y de los servicios instalados (como SSH, Apache o red). |
| **`/home`** | Contiene las carpetas personales donde cada usuario guarda sus documentos y configuraciones propias. |
| **`/root`** | Es el directorio personal exclusivo del superusuario o administrador del sistema (`root`). |
| **`/var`** | Almacena datos que cambian constantemente, como registros del sistema (*logs*), correos y páginas web (`/var/www`). |
| **`/tmp`** | Espacio destinado a archivos temporales creados por el sistema o los programas; suelen borrarse al reiniciar. |
| **`/dev`** | Contiene archivos especiales que representan los dispositivos de hardware conectados (discos, memoria, terminales). |
| **`/boot`** | Guarda los archivos necesarios para el arranque del sistema operativo, incluyendo el kernel de Linux. |
| **`/opt`** | Se utiliza para instalar software adicional o aplicaciones de terceros que no vienen en los repositorios estándar. |

![Estructura de directorios con tree](img/fig4_tree.png)  
*Fig. 4. Jerarquía de directorios raíz (`/`) visualizada mediante el comando `tree -L 1 /`.*

---

### 3.5. Despliegue y Personalización del Servidor Web Apache
Se instaló el servicio web Apache (`httpd`), se habilitó en el firewall y se creó una página de prueba personalizada con los datos del estudiante.

**Comandos ejecutados:**
1. Instalar e iniciar Apache:
   ```bash
   sudo yum install httpd -y
   sudo systemctl enable --now httpd
   ```
2. Permitir el tráfico HTTP en el firewall:
   ```bash
   sudo firewall-cmd --permanent --add-service=http
   sudo firewall-cmd --reload
   ```
3. Crear y personalizar el archivo `index.html`:
   ```bash
   echo "<h1>Servidor Apache Funcionando - [nombre_apellido]</h1>" | sudo tee /var/www/html/index.html
   ```
4. **Comprobación:** Se ingresó a `http://[IP_DEL_SERVIDOR]` desde el navegador web de la máquina anfitriona.

![Página web Apache personalizada](img/fig5_apache.png)  
*Fig. 5. Visualización de la página `index.html` personalizada desde el navegador del cliente.*

---

### 3.6. Configuración y Prueba del Servidor de Archivos FTP
Para permitir la carga y descarga de archivos desde un entorno gráfico externo, se instaló y configuró el servicio **vsftpd**.

**Pasos realizados en el servidor:**
1. Instalar el paquete del servidor FTP:
   ```bash
   sudo yum install vsftpd -y
   ```
2. Configurar `/etc/vsftpd/vsftpd.conf` asegurando que los usuarios locales puedan iniciar sesión y subir archivos:
   ```text
   anonymous_enable=NO
   local_enable=YES
   write_enable=YES
   chroot_local_user=YES
   allow_writeable_chroot=YES
   ```
3. Habilitar el servicio, el firewall y el permiso de SELinux para directorios personales:
   ```bash
   sudo systemctl enable --now vsftpd
   sudo firewall-cmd --permanent --add-service=ftp
   sudo firewall-cmd --reload
   sudo setsebool -P ftpd_full_access on
   ```

**Comprobación con FileZilla:**
Desde la máquina anfitriona se abrió **FileZilla Client**, ingresando el host (`[IP_DEL_SERVIDOR]`), nombre de usuario de CentOS, contraseña y puerto `21`, logrando subir y descargar un archivo de prueba exitosamente.

![Prueba de conexión en FileZilla](img/fig6_ftp.png)  
*Fig. 6. Transferencia bidireccional de archivos verificada mediante el cliente FileZilla.*

---

## 4. Conclusiones
* La configuración del adaptador de red en modo puente permitió integrar el servidor CentOS dentro de la red local, facilitando tanto la salida a Internet como el acceso desde equipos externos.
* Cambiar el puerto por defecto en SSH y gestionar las reglas del firewall y SELinux garantiza que los servicios funcionen bajo buenas prácticas básicas de seguridad en Linux.
* La combinación de herramientas de terminal (`SSH`, `SCP`, `tree`) y servicios cliente-servidor (`Apache`, `vsftpd`) permite administrar de manera integral un servidor sin depender de una interfaz gráfica.

## Referencias
1. Documentación oficial de CentOS Linux. Disponible en: https://centos.org
2. Manual del servidor HTTP Apache. Disponible en: https://apache.org
3. Documentación oficial de OpenSSH y vsftpd para sistemas Enterprise Linux.

