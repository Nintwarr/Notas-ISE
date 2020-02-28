## CLASE 2

- **brm:** Sectores de tratamiento de discos

### Comandos CentOS7

**Casi todos sirven para Ubuntu-Server**

- `lsblk`: list  block devices
- `df`: disk free space
- `lvmdiskscan`
- `lvdisplay | vgdisplay | pvdisplay`
- `lvs | vgs | pvs` logical volume short list
- `mkfs` make file system

### Rutas de los volúmenes

```
/dev/VG(volume group)/LV
/dev/mapper/VG-LV
```

### Comandos de instalación de nuevo volumen físico 

```
pvcreate (path) "/dev/sdb"
vgextend (parámetros)
lvcreate -L (tamaño)4G -n (name)"newvar" (volume group) cl
```

- **Pasos:**
  - Asignar el sistema de ficheros
  - Crear un directorio /media/newvar y montar el LV(SA)
  - Copiar de /var a /media/newvar
  - Montar el volumen lógico LV en /var
  - Liberar espacio


### Ejecución

```shell
mkfs -t (tipo) path "/dev/cl/newvar"
mkdir /media/newvar
mount (qué montar) /dev/cl/newvar (dónde montar) /media/newvar
systemctl isolate runlevel1.target
cp -a /var/. /media/newvar # Guarda contextos
```

"snapshot"

 dentro de fstab incluir:    usar editor vi

```
      /dev/mapper/cl-newvar      /var            ext4      defaults    0 0
```

```shell
 umount /media/newvar
```

comando para monitorizar: `sytemctl status`
comando para quedarte tú con el control exclusivo: `systemctl isolate runlevel1.target`

//	La polichía: SELinux (Security Enhanced) --> se asegura que no te metes en sus asuntos. Si no se respeta el contexto en copia de datos, no te dejará
//

  Sintaxis vi:

 i - insert

 a- add

 ESC :q! -> sail sin guardar

 ESC :wq -> salir guardando



mount -a   para "actualizar"

umount /media/newvar   //desmontamos el volúmen temporal

Paso 5:

Hacer una copia que mantendremos durante un tiempo para poder evitar incidencias

umount /dev/mapper/cl-newvar
mv /var /varold
mkdir /var
ls -Z /var
restorecon (-Rv) /var
mount -a
ls -Z



Si aparece problema de disco ocupado:   instalar un programilla llamado lsof



Y para poder salir del modo runlevel1, usar:

sytemctl get-default
sytem isolate "resultado de get-default".target
