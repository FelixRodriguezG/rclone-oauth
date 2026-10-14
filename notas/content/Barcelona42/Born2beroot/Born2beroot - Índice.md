---
tags: [proyecto, born2beroot, indice, rank-2, sistemas, linux]
aliases: ["Born2beroot", "B2BR"]
---
# Born2beroot - Índice

[[Inicio|← Inicio]] · Proyectos · Rank 2

> **Born2beroot** es el primer proyecto de administración de sistemas: instalas un servidor Linux mínimo en una máquina virtual y lo configuras con reglas de seguridad estrictas (particiones cifradas, `sudo`, contraseñas, cortafuegos, SSH) y un script de monitorización. No se programa en C; se evalúa sobre todo lo que **sabes explicar** en la defensa.

> [!warning] Comprueba el subject
> Resumen basado en versiones públicas del subject; los detalles cambian entre versiones. Lo que manda es el PDF de tu intra. Los valores concretos (días de caducidad, longitud mínima de contraseña, número de intentos…) que aparecen aquí son los que suelen pedirse; compruébalos uno a uno.

## Qué se pide

Lo comúnmente conocido del subject (parte obligatoria):

| Área | Lo habitual |
|---|---|
| Sistema | Última versión estable de **Debian** o de **Rocky**, sin interfaz gráfica |
| Discos | Al menos 2 particiones **cifradas** con **LVM** |
| Seguridad | AppArmor activo en Debian (SELinux en Rocky) |
| SSH | Solo en el puerto **4242**; prohibido entrar como `root` por SSH |
| Cortafuegos | UFW (firewalld en Rocky) con solo el puerto 4242 abierto |
| Hostname | Tu login seguido de `42` (p. ej. `felixrod42`) |
| Usuarios | Un usuario con tu login, en los grupos `user42` y `sudo` |
| Contraseñas | Caducidad, longitud y complejidad mínimas (ver más abajo) |
| sudo | Intentos limitados, mensaje propio, registro de todo, TTY obligatoria, rutas restringidas |
| Script | `monitoring.sh` que muestra información del sistema cada 10 minutos en todas las terminales |
| Entrega | Solo un fichero `signature.txt` con la firma del disco virtual |

**Bonus** habitual: particionado más detallado; un WordPress con lighttpd, MariaDB y PHP; y un servicio extra a tu elección (justificado).

## Vocabulario

| Término | Significado |
|---|---|
| **VM** (máquina virtual) | Un ordenador simulado por software dentro de tu ordenador real |
| **Hipervisor** | El programa que crea y ejecuta las VM (VirtualBox, UTM) |
| **Host / guest** | Tu máquina real / la máquina virtual |
| **Partición** | Trozo de disco que el sistema trata como unidad independiente |
| **LVM** | *Logical Volume Manager*: capa que permite crear «discos lógicos» redimensionables |
| **LUKS** | Formato estándar de cifrado de disco en Linux (lo maneja `cryptsetup`) |
| **Servicio / daemon** | Programa que corre en segundo plano (`sshd`, `cron`, `ufw`) |
| **TTY** | Terminal; `requiretty` obliga a usar `sudo` desde una terminal real |

## Qué es una máquina virtual

Una VM usa una parte de la CPU, la RAM y el disco del host para ejecutar **otro sistema operativo aislado**. Su disco es un fichero (`.vdi` en VirtualBox). Ventajas: puedes romperla sin miedo, hacer instantáneas (*snapshots*) y moverla de ordenador.

```text
┌──────────── Host (Linux de 42) ────────────┐
│  VirtualBox (hipervisor)                    │
│  ┌──────── Guest: Debian ────────┐          │
│  │ sshd :4242   ufw   cron        │          │
│  └───────────────────────────────┘          │
│        ▲ reenvío de puertos NAT             │
│  ssh -p 4242 usuario@localhost              │
└─────────────────────────────────────────────┘
```

## Debian o Rocky

| | Debian | Rocky Linux |
|---|---|---|
| Familia | Debian (base de Ubuntu) | Red Hat (compatible con RHEL, sucesor comunitario de CentOS) |
| Paquetes | `apt` / `aptitude`, `.deb` | `dnf`, `.rpm` |
| Seguridad obligatoria | AppArmor | SELinux |
| Cortafuegos del proyecto | UFW | firewalld |
| Dificultad en 42 | Recomendado para empezar | Más complejo de configurar |

> [!question] ¿Diferencia entre `apt` y `aptitude`?
> Los dos instalan paquetes `.deb`. `apt` es la herramienta de línea de comandos estándar. `aptitude` es de más alto nivel: tiene interfaz de texto interactiva y un resolvedor de dependencias que propone soluciones cuando hay conflictos.

> [!question] ¿Qué es AppArmor?
> Un módulo de seguridad del kernel que limita qué ficheros y recursos puede usar cada programa según un **perfil**. Si un servicio es atacado, el daño queda restringido a lo que su perfil permite. Se comprueba con `sudo aa-status`.

## Particiones cifradas con LVM

En el instalador de Debian elige el particionado guiado **«usar todo el disco y configurar LVM cifrado»** (o hazlo a mano para el bonus). La estructura queda así:

```text
sda (disco virtual)
├─ sda1   /boot           sin cifrar (el arranque necesita leerla)
└─ sda5   LUKS (cifrada)  ← pide la frase de paso al arrancar
   └─ sda5_crypt
      └─ Grupo de volúmenes LVM
         ├─ root   /
         ├─ swap   [SWAP]
         └─ home   /home
            (bonus: var, srv, tmp, var-log…)
```

```bash
lsblk                       # árbol de discos, particiones y volúmenes lógicos
sudo vgs ; sudo lvs         # grupos de volúmenes y volúmenes lógicos de LVM
```

> [!question] ¿Para qué sirve LVM?
> Agrupa uno o varios discos físicos en un «grupo de volúmenes» y lo reparte en volúmenes lógicos que puedes **ampliar, reducir o mover** sin reparticionar. Sin LVM, cambiar el tamaño de una partición es mucho más delicado.

## sudo y su configuración

`sudo` permite a un usuario autorizado ejecutar comandos como `root`, dejando registro. Se configura **siempre con `visudo`**, que comprueba la sintaxis antes de guardar (un error en `sudoers` puede dejarte sin `sudo`).

```bash
su -                                  # hazte root (solo durante la instalación)
apt install sudo
usermod -aG sudo felixrod             # añade tu usuario al grupo sudo
visudo -f /etc/sudoers.d/sudo_config  # fichero propio, más limpio que editar /etc/sudoers
mkdir -p /var/log/sudo                # carpeta para los registros
```

Contenido habitual de `/etc/sudoers.d/sudo_config` (cada opción está en `man 5 sudoers`):

```text
Defaults	passwd_tries=3
Defaults	badpass_message="Contraseña incorrecta"
Defaults	logfile="/var/log/sudo/sudo.log"
Defaults	log_input, log_output
Defaults	iolog_dir="/var/log/sudo"
Defaults	requiretty
Defaults	secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
```

| Opción | Qué hace |
|---|---|
| `passwd_tries` | Intentos de contraseña antes de abortar (por defecto 3) |
| `badpass_message` | Mensaje al fallar la contraseña |
| `logfile` | Fichero donde se registra cada uso de `sudo` |
| `log_input`, `log_output` | Graba la entrada y la salida de los comandos |
| `iolog_dir` | Carpeta donde se guardan esas grabaciones |
| `requiretty` | Solo se puede usar `sudo` desde una terminal |
| `secure_path` | El `PATH` que usa `sudo`, para que no se ejecuten programas de rutas manipuladas |

## Política de contraseñas

Dos sitios distintos, y conviene saber cuál controla qué:

### Caducidad: `/etc/login.defs` y `chage`

```text
PASS_MAX_DAYS	30
PASS_MIN_DAYS	2
PASS_WARN_AGE	7
```

> [!warning] Solo afecta a usuarios nuevos
> Según `man 5 login.defs`, estos valores **solo se aplican al crear la cuenta**. Para los usuarios que ya existen (tu usuario y `root`) usa `chage`:
> ```bash
> sudo chage -M 30 -m 2 -W 7 felixrod
> sudo chage -M 30 -m 2 -W 7 root
> sudo chage -l felixrod          # comprueba los valores
> ```

### Complejidad: `libpam-pwquality`

```bash
sudo apt install libpam-pwquality
sudo nano /etc/pam.d/common-password
```

Línea habitual (cada opción en `man 8 pam_pwquality`):

```text
password  requisite  pam_pwquality.so retry=3 minlen=10 ucredit=-1 lcredit=-1 dcredit=-1 maxrepeat=3 reject_username difok=7 enforce_for_root
```

| Opción | Significado |
|---|---|
| `retry=3` | 3 intentos para escribir una contraseña válida |
| `minlen=10` | Mínimo 10 caracteres |
| `ucredit=-1` / `lcredit=-1` / `dcredit=-1` | Al menos 1 mayúscula / minúscula / dígito (el valor negativo es «mínimo obligatorio») |
| `maxrepeat=3` | Como mucho 3 caracteres idénticos seguidos |
| `reject_username` | No puede contener el nombre de usuario |
| `difok=7` | Al menos 7 caracteres distintos respecto a la anterior |
| `enforce_for_root` | Las reglas también valen para `root` |

> [!note] `difok` y `root`
> El man de `pam_pwquality` explica que a `root` no se le pide la contraseña antigua, así que las comprobaciones que comparan la nueva con la vieja (como `difok`) no se le aplican. Esto encaja con lo que suele pedir el subject.

Después de configurar, **cambia las contraseñas** de tu usuario y de `root` para que cumplan las reglas (`passwd`, `sudo passwd root`).

## UFW (cortafuegos)

UFW (*Uncomplicated Firewall*) es una interfaz sencilla sobre el filtrado de paquetes del kernel.

```bash
sudo apt install ufw
sudo ufw enable                 # activa el cortafuegos (y al arrancar)
sudo ufw allow 4242             # abre el puerto de SSH
sudo ufw status numbered        # lista las reglas con número
sudo ufw allow 8080             # en la defensa suelen pedir abrir otro puerto…
sudo ufw delete 3               # …y borrarlo por su número (revisa el número antes)
```

> [!warning] Reglas IPv6
> `ufw allow 4242` crea dos reglas: IPv4 y `(v6)`. Al borrar, revisa `status numbered` otra vez: los números cambian tras cada borrado.

## SSH en el puerto 4242

SSH permite abrir una terminal remota cifrada. En `/etc/ssh/sshd_config`:

```text
Port 4242
PermitRootLogin no
```

```bash
sudo apt install openssh-server
sudo systemctl restart ssh
sudo systemctl status ssh       # comprueba que escucha
ss -tlnp | grep 4242            # ¿hay algo escuchando en el 4242?
```

En VirtualBox, con red NAT, añade un **reenvío de puertos**: puerto anfitrión 4242 → puerto invitado 4242. Luego, desde el host:

```bash
ssh felixrod@localhost -p 4242
```

## Hostname, usuarios y grupos

```bash
hostnamectl                               # ver el hostname
sudo hostnamectl set-hostname nuevo42     # cambiarlo
sudo nano /etc/hosts                      # cambia también el nombre antiguo aquí
sudo reboot

sudo adduser evaluador                    # crea usuario (pide contraseña: se aplica la política)
sudo groupadd evaluating                  # crea grupo
sudo usermod -aG evaluating evaluador     # añade al grupo (-a: sin quitarlo de los demás)
groups evaluador                          # grupos de un usuario
getent group sudo user42                  # miembros de cada grupo
```

> [!danger] `usermod -G` sin `-a`
> Sin `-a`, `usermod -G` **sustituye** todos los grupos secundarios del usuario. Puedes quitarte del grupo `sudo` sin querer.

## cron y monitoring.sh

`cron` es el servicio que ejecuta tareas programadas. Cada usuario tiene una tabla (*crontab*) con líneas de cinco campos de tiempo + comando:

```text
┌ minuto (0-59)
│ ┌ hora (0-23)
│ │ ┌ día del mes
│ │ │ ┌ mes
│ │ │ │ ┌ día de la semana
*/10 * * * *  /usr/local/bin/monitoring.sh
```

```bash
sudo crontab -u root -e     # edita la crontab de root
sudo crontab -u root -l     # la muestra
```

`*/10` significa «en los minutos múltiplos de 10» (:00, :10, :20…), **no** «cada 10 minutos desde el arranque». Si el subject lo pidiera relativo al arranque, tendrías que calcular un `sleep` a partir del tiempo de encendido.

### Ejemplo de monitoring.sh

Script orientativo (Bash). Ajusta el formato exacto al que pida tu subject:

```bash
#!/bin/bash
arch=$(uname -a)
pcpu=$(grep "physical id" /proc/cpuinfo | sort -u | wc -l)
vcpu=$(grep -c "^processor" /proc/cpuinfo)
ram=$(free --mega | awk '$1 == "Mem:" {printf "%d/%dMB (%.2f%%)", $3, $2, $3 * 100 / $2}')
disk=$(df -BM --total -x tmpfs -x devtmpfs | awk '$1 == "total" {printf "%d/%dMB (%s)", $3, $2, $5}')
cpu=$(top -bn1 | awk '/^%Cpu/ {printf "%.1f%%", 100 - $8}')
boot=$(uptime -s)
lvm=$(if lsblk | grep -q "lvm"; then echo yes; else echo no; fi)
tcp=$(ss -ta | grep -c ESTAB)
ulog=$(users | wc -w)
ip=$(hostname -I | awk '{print $1}')
mac=$(ip link | awk '/link\/ether/ {print $2; exit}')
cmds=$(journalctl _COMM=sudo | grep -c COMMAND)
wall "#Architecture: $arch
#CPU physical: $pcpu
#vCPU: $vcpu
#Memory Usage: $ram
#Disk Usage: $disk
#CPU load: $cpu
#Last boot: $boot
#LVM use: $lvm
#Connections TCP: $tcp ESTABLISHED
#User log: $ulog
#Network: IP $ip ($mac)
#Sudo: $cmds cmd"
```

| Línea | De dónde sale el dato |
|---|---|
| CPU físicas / vCPU | `/proc/cpuinfo`: ids físicos distintos / número de `processor` |
| Memoria | `free --mega`: columnas *used* y *total* |
| Disco | `df` con fila `total`, sin sistemas de ficheros en RAM |
| Carga CPU | `top` en modo lote: 100 − porcentaje *idle* (`id`) |
| Último arranque | `uptime -s` |
| LVM | ¿aparece `lvm` en `lsblk`? |
| Conexiones TCP | `ss -ta`, líneas en estado `ESTAB` |
| Usuarios | `users`: un nombre por sesión abierta |
| Red | `hostname -I` (IPv4) e `ip link` (MAC) |
| Comandos sudo | Registro del sistema (`journalctl`) filtrado por `sudo` |

```bash
sudo cp monitoring.sh /usr/local/bin/ && sudo chmod +x /usr/local/bin/monitoring.sh
sudo systemctl stop cron     # pregunta típica: pararlo sin tocar el script
sudo systemctl start cron
```

## La firma (signature.txt)

La entrega es la huella **SHA-1** del fichero de disco de la VM:

```bash
sha1sum Born2beroot.vdi > signature.txt     # Linux (en macOS: shasum)
```

> [!danger] La firma cambia si arrancas la VM
> Cualquier cambio en el disco (arrancar, crear un usuario) cambia la huella. Lo habitual es generar la firma y **no volver a arrancar esa VM**, o clonarla/hacer un snapshot para las evaluaciones. Si la firma no coincide en la defensa, es un 0.

## Plan de trabajo

1. Instala VirtualBox (o UTM) y descarga la ISO estable de Debian desde debian.org.
2. Crea la VM e instala con particionado LVM cifrado. Sin entorno gráfico.
3. Instala y configura `sudo`; añade tu usuario a `sudo` y crea `user42`.
4. Configura SSH (puerto 4242, sin `root`) y el reenvío de puertos.
5. Instala UFW y abre solo el 4242.
6. Política de contraseñas (`login.defs`, `chage`, `pam_pwquality`) y cambia las contraseñas.
7. Configura los `Defaults` de `sudo` con `visudo`.
8. Escribe `monitoring.sh` y prográmalo con `cron`.
9. Comprueba todo con la lista de la defensa (abajo). Haz un snapshot.
10. Genera `signature.txt` y súbelo al repo con [[Git para 42]].

## Errores frecuentes que suspenden

> [!danger] Lista de comprobación antes de entregar
> - Firma que no coincide (la VM se arrancó después de generarla).
> - Interfaz gráfica instalada (desmarca «entorno de escritorio» en el instalador).
> - `root` puede entrar por SSH, o SSH escucha también en el 22.
> - Reglas de `login.defs` puestas pero sin `chage` en los usuarios existentes.
> - Contraseñas antiguas que no cumplen la nueva política.
> - `sudo` sin registro, o carpeta de logs inexistente.
> - No saber explicar **por qué** de cada configuración (la defensa es oral).

## Preguntas de evaluación

> [!question] ¿Qué es una máquina virtual y para qué sirve?
> Un ordenador simulado por software que comparte el hardware del host. Sirve para aislar, probar, tener varios sistemas a la vez y recuperar estados con snapshots.

> [!question] ¿Por qué elegiste Debian (o Rocky)?
> Debian: estable, muy documentado, más sencillo para empezar. Rocky: compatible con RHEL, habitual en empresas, pero más complejo.

> [!question] Demuestra que no hay interfaz gráfica, que UFW y SSH están activos y que AppArmor funciona
> `systemctl get-default` (debe ser `multi-user.target`, no `graphical.target`), `sudo ufw status`, `sudo systemctl status ssh`, `sudo aa-status`.

> [!question] Crea un usuario, un grupo `evaluating` y mete al usuario en él
> `adduser`, `groupadd`, `usermod -aG`, `getent group`. Explica cómo se aplica la política de contraseñas al crearlo.

> [!question] Cambia el hostname y reinicia
> `hostnamectl set-hostname`, editar `/etc/hosts`, `reboot`, y comprobar.

> [!question] Enséñame las particiones
> `lsblk` y explica qué es LVM y qué partición está cifrada.

> [!question] ¿Dónde están los registros de sudo?
> En la ruta de `logfile` y en `iolog_dir` (por ejemplo `/var/log/sudo/`). Ejecuta un comando con `sudo` y enseña la nueva entrada.

> [!question] Abre el puerto 8080 en UFW y luego bórralo
> `ufw allow 8080`, `ufw status numbered`, `ufw delete N`.

> [!question] ¿Qué es cron? Haz que el script se ejecute cada minuto y luego páralo sin modificar el script
> Cambia `*/10` por `*` en la crontab; para pararlo, `systemctl stop cron` o comenta la línea.

## Relacionadas

[[Git para 42]] · [[Vim]] · [[ft_printf - Índice]] · [[get_next_line - Índice]] · [[La Norma]]

## Referencias

- [Debian Administrator's Handbook](https://debian-handbook.info/browse/stable/) — instalación, gestión de usuarios y grupos ([sect.user-group-databases](https://debian-handbook.info/browse/stable/sect.user-group-databases.html)), SSH ([sect.remote-login](https://debian-handbook.info/browse/stable/sect.remote-login.html)), cortafuegos ([sect.firewall-packet-filtering](https://debian-handbook.info/browse/stable/sect.firewall-packet-filtering.html)), cron ([sect.task-scheduling-cron-atd](https://debian-handbook.info/browse/stable/sect.task-scheduling-cron-atd.html)) y LVM ([advanced-administration](https://debian-handbook.info/browse/stable/advanced-administration.html)).
- [sudoers(5) — Debian bookworm](https://manpages.debian.org/bookworm/sudo/sudoers.5.en.html) — `passwd_tries` (por defecto 3), `badpass_message`, `logfile`, `log_input`, `log_output`, `iolog_dir`, `requiretty`, `secure_path`.
- [pam_pwquality(8)](https://manpages.debian.org/bookworm/libpam-pwquality/pam_pwquality.8.en.html) y [pwquality.conf(5)](https://manpages.debian.org/bookworm/libpwquality-common/pwquality.conf.5.en.html) — opciones de complejidad; `enforce_for_root` y la excepción de comparar con la contraseña antigua.
- [login.defs(5)](https://manpages.debian.org/bookworm/login/login.defs.5.en.html) — `PASS_MAX_DAYS`, `PASS_MIN_DAYS`, `PASS_WARN_AGE` solo al crear cuentas. [chage(1)](https://manpages.debian.org/bookworm/passwd/chage.1.en.html).
- [ufw(8)](https://manpages.debian.org/bookworm/ufw/ufw.8.en.html) — `enable`, `allow`, `status numbered`, `delete`.
- [sshd_config(5)](https://manpages.debian.org/bookworm/openssh-server/sshd_config.5.en.html) — `Port`, `PermitRootLogin`.
- [crontab(5)](https://manpages.debian.org/bookworm/cron/crontab.5.en.html) y [wall(1)](https://manpages.debian.org/bookworm/bsdutils/wall.1.en.html) — sintaxis de la crontab y mensajes a todas las terminales.
- [lvm(8)](https://manpages.debian.org/bookworm/lvm2/lvm.8.en.html), [cryptsetup(8)](https://manpages.debian.org/bookworm/cryptsetup-bin/cryptsetup.8.en.html) y [lsblk(8)](https://manpages.debian.org/bookworm/util-linux/lsblk.8.en.html) — volúmenes lógicos y cifrado LUKS.
- [usermod(8)](https://manpages.debian.org/bookworm/passwd/usermod.8.en.html), [groupadd(8)](https://manpages.debian.org/bookworm/passwd/groupadd.8.en.html), [hostnamectl(1)](https://manpages.debian.org/bookworm/systemd/hostnamectl.1.en.html).
- [AppArmor — Debian Wiki](https://wiki.debian.org/AppArmor) y [aa-status(8)](https://manpages.debian.org/bookworm/apparmor/aa-status.8.en.html).
- [Rocky Linux documentation](https://docs.rockylinux.org/) y [VirtualBox User Manual](https://www.virtualbox.org/manual/).
- Requisitos del proyecto (valores de la política, puerto 4242, formato de `monitoring.sh`, bonus): conocimiento general de versiones públicas del subject de Born2beroot, no verificado contra el PDF. Los comandos del script se han probado en Ubuntu 22.04, no en una Debian recién instalada.
