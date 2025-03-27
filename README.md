# Modulo-VII
Samba
# Instalar Samba y sus dependencias
sudo dnf install samba samba-client samba-common -y

# Habilitar y arrancar los servicios de Samba
sudo systemctl enable --now smb
sudo systemctl enable --now nmb

# Crear el directorio compartido y establecer permisos
sudo mkdir -p /srv/samba/adrianes
sudo chmod -R 777 /srv/samba/adrianes
sudo chown nobody:nobody /srv/samba/adrianes
sudo chcon -t samba_share_t /srv/samba/adrianes

# Crear archivos de prueba en el directorio compartido
cd /srv/samba/adrianes/ || exit
sudo touch adrian{1..100}.txt

# Configurar firewall para permitir tráfico Samba
sudo firewall-cmd --add-service=samba --permanent
sudo firewall-cmd --reload

# Configurar el archivo smb.conf
sudo bash -c 'cat <<EOF >> /etc/samba/smb.conf

[public]
    path = /srv/samba/adrianes
    browsable = yes
    writable = yes
    guest ok = yes
    read only = no
EOF'

# Crear grupo y usuario Samba con contraseña
sudo groupadd -f sambausers
sudo useradd -m -G sambausers papolo
echo "papolo:12345" | sudo chpasswd
echo -e "12345\n12345" | sudo smbpasswd -a papolo

# Verificar grupos del usuario
groups papolo

# Reiniciar servicios para aplicar cambios
sudo systemctl restart smb nmb

echo "Configuración de Samba completada exitosamente."

Practica NSF
Configuración en el Servidor (Nodo 1 - 192.168.198.130)
1. Instalar y configurar NFS

sudo dnf install -y nfs4-acl-tools nfs-utils
sudo systemctl restart nfs-server --now

2. Crear el directorio compartido y asignar permisos

sudo mkdir -p /var/nfs/OS3
sudo chown nobody:nobody /var/nfs/OS3
cd /var/nfs/OS3/
sudo touch adrian{1..100}.txt
ls
sudo chown rae:rae *
sudo chmod 666 *
ls -l
3. Configurar los directorios a compartir en NFS

sudo no /etc/exports
Agregar las siguientes líneas:


/var/nfs/OS3    *(rw,sync,no_subtree_check)
/home           *(rw,sync,no_root_squash,no_subtree_check)
4. Exportar los directorios y configurar firewall

sudo exportfs -avr
sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --permanent --add-service=rpc-bind
sudo firewall-cmd --permanent --add-service=mountd
sudo firewall-cmd --reload

Configuración en el Cliente (Nodo 2 - 192.168.198.131)

5. Verificar el recurso compartido desde el cliente

showmount -e 192.168.198.130

6. Crear directorios locales y montar NFS

sudo mkdir -p /nfs/OS3
sudo mkdir -p /home/OS3
sudo mount 192.168.198.130:/var/nfs/OS3 /nfs/OS3/
sudo mount 192.168.198.130:/home /home/OS3
df -h
sudo ls /nfs/OS3

7. Crear y modificar archivos para prueba

sudo nano /nfs/OS3/adrian1.txt
Escribir algún texto y guardar con

8. Verificar los cambios en el servidor

sudo cat /var/nfs/OS3/adrian1.txt
9. Configurar montaje persistente en /etc/fstab

sudo nano /etc/fstab
Agregar las siguientes líneas:


192.168.198.130:/var/nfs/OS3  /nfs/OS3  nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
192.168.198.130:/home /home/OS3  nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
10. Reiniciar y verificar

sudo reboot
df -h
sudo ls /nfs/OS3
