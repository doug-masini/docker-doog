# Static IP Setup
    /etc/network/interfaces.d/eth0
# Modify /etc/netplan/01-netcfg.yaml     
    network:
     version: 2
     renderer: NetworkManager
     ethernets:
       eth0:
         dhcp4: no
         addresses: [192.168.*.*/24]
         gateway4: 192.168.1.1
         nameservers:
             addresses: [8.8.8.8,8.8.8.4]
  ## Then Try
    sudo netplan try
    sudo netplan apply 
    Reboot the system if needed to apply changes
  # SSH configuration
  ## You can set several options in /etc/ssh/sshd_config. One is the listen address. If You set a listen address on your subnet. A private IP address is not routable over the internet.
    ListenAddress 192.168.0.10
  ## You can use the AllowUsers
    AllowUsers you@192.168.0.0/16
  ## You should change the port from the default
    Port 1234
  ## Match blocks can be used for specific users and addresses
    PasswordAuthentication no   #
    PubkeyAuthentication no     # to disable access for all other users
    Match Address 192.168.1.0/24
            PermitRootLogin yes
            PasswordAuthentication yes
            PubkeyAuthentication yes
    
    Match User vpnuser Address 10.10.10.0/24
            PasswordAuthentication yes
            PubkeyAuthentication yes
            
    Match User outsideUser Address *
            PasswordAuthentication no
            PubkeyAuthentication yes

# Installing Docker
  ## Add Docker's official GPG key:
    sudo apt-get update
    sudo apt-get install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc
  
  ## Add the repository to Apt sources:
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
      $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
  # Appdata Folder Setup
    mkdir /home/{userName}/docker/appdata
  # Secrets Folder
    sudo chown root:root /home/{userName}/docker/secrets
    sudo chmod 600 /home/{userName}/docker/secrets
  # Env Variables Setup
    touch /home/{userName}/docker/.env
    sudo chown root:root /home/{userName}/docker/.env
    sudo chmod 600 /home/{userName}/docker/.env
    sudo nano /home/{userName}/docker/.env
  ## Get PUID and PGID with 
    id
  ## Enter this into .env file
    PUID=1000
    PGID=1000
    TZ="America/New_York"
    USERDIR="/home/dumds"
    DOCKERDIR="/home/{userName}/docker"
    DATADIR="/media/storage"
    HOSTNAME="dumds"
# Setup Folder Permissions
    sudo chmod 775 /home/{userName}/docker
## May need to set permissions with acl
    sudo apt install acl
    sudo setfacl -Rdm u:{userName}:rwx /home/dumds/docker
    sudo setfacl -Rm u:{userName}:rwx /home/dumds/docker
    sudo setfacl -Rdm g:docker:rwx /home/dumds/docker
    sudo setfacl -Rm g:docker:rwx /home/dumds/docker
  
# Creating a Raid5 disk
    lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT
    sudo mdadm --create --verbose /dev/md0 --level=5 --raid-devices=3 /dev/sda /dev/sdb /dev/sdc
    cat /proc/mdstat

## Next, create a filesystem on the array:
    sudo mkfs.ext4 -F /dev/md0

## Create a mount point to attach the new filesystem:
    sudo mkdir -p /mnt/md0

## You can mount the filesystem with the following:
    sudo mount /dev/md0 /mnt/md0

## Check whether the new space is available:
    df -h -x devtmpfs -x tmpfs

# Utility Functions
## Check status for docker containers
    sudo docker ps -a
## Run Docker Compose to build the containers
    sudo docker compose -f /home/bauldur/docker/docker-compose-dumds.yml up -d

## To Update Images Run
    sudo docker compose -f /home/bauldur/docker/docker-compose-dumds.yml pull
    sudo docker compose -f /home/bauldur/docker/docker-compose-dumds.yml down
    sudo docker compose -f /home/bauldur/docker/docker-compose-dumds.yml up -d
## Check Timezone
    ls -l /etc/localtime
