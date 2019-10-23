## CLASE 3

## Crear RAID1 en CentOS

Empezamos de cero y le añadimos dos discos de unos dos gigas:

1. Miramos nuestra configuración con `lsblk`.

2. Creamos un RAID por software con `mdadm`. Lo tendremos que instalar (iniciamos como root).

3. Como tendremos que entrar en red, vamos a manejar la tarjeta de red con `ip addr`.

   3.1. Levantamos la tarjeta de red con `ifup`.

4. Instalamos con `yum install -y "nombre"` (que va a ser mdadm).

5. Procedemos a hacer el RAID por software:
`mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc`

6. Ya tenemos el RAID, pero aún no tenemos creado un volumen físico, procedemos:

   6.1. `pvcreate /dev/md0`

   6.2. Ahora crearemos el grupo de volúmenes: `vgcreate pmraid1 (más argumentos)`

   6.3. Crear volúmenes lógico dentro de grupo pmraid1:
   `lvcreate -L 1G -n newvar pmraid1`

   6.4. Crear sistema de archivos:
   `mkfs -t ext4 /dev/mapper/pmraid1-newvar`

   6.5. Dejar de dar servicios a otros usuarios (aislar el sistema):
   `systemctl isolate runlevel1.target`

   6.6. Montar newvar:
   ```
   mkdir /mnt/newvar
   mount /dev/mapper/pmradi1-newvar /mnt/newvar
   cp -r --preserve=all /var/. /mnt/newvar
   ```

   6.7. Hacer copia de seguridad de fstab y luego modificarlo así:
   ```
   echo '/dev/mapper/pmraid1-newvar /var ext4 defaults 0 0' >> /etc/fstab
   umount /mnt/newvar
   ```

   6.8. Seguir todos los pasos anteriores de copia con contexto y finalmente montar todo: `mount -a`

Apagamos, creando snapshot y probamos a "pegarle un tirón al disco duro".

## Encriptación de volúmenes lógicos

Ahora vamos a encriptar los volúmenes lógicos con LUKS (Linux Unified Key Setup).
Podemos hacer LUKS sobre LVM o LVM sobre LUKS, siendo este caso el primero.

   1. Levantamos interfaz de red con `ifup enp0s3`.

   2. `yum install cryptsetup`.

   3. Entramos en modo mantenimiento y realizamos:
   ```
   mkdir /varRAID
   cp -r --preserve=all /var/. /varRAID
   umount /var
   mount | grep var
   cryptsetup luksFormat /dev/mapper/pmraid1-newvar
   cryptosetup luksOpen /dev/mapper/pmraid1-newvar pmraid1-newvar_crypt
   mkfs -t ext4 /dev/mapper/pmraid1-newvar_crypt
   ```

   4. Hacemos una copia de seguridad:
   ```
   mkdir /mnt/varCifr
   mount /dev/mapper/pmraid1-newvar_crypt /mnt/varCifr/
   cp -r --preserve=all /varRAID/. /mnt/varCifr/
   ```

   5. Para obtener el UUID del volumen lógico al que voy a enlazar el volumen encriptado usamos:
   `blkid | grep var`.

   6. Dentro de /etc/crypttab poner:
   `pmraid1-newvar_crypt UUID=(ID del volumen lógico sin comillas) none`.

   ```
   cp /etc/fstab /etc/fstab.backup
   echo '/dev/mapper/pmraid1-newvar_crypt /var ext4 defaults 0 0' >> /etc/fstab
   ```

   7. Comprobar el fstab y quitar penúltima línea, la del volumen lógico sin encriptar:
   ```
   umount /mnt/varCifr
   mount -a
   ```
