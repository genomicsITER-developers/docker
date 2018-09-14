# Docker (Ubuntu Trusty Tahr 14.04 LTS)

This README explains how to install Docker in a Ubuntu 14.04 LTS system. It additionaly explains how to move the /var/lib/docker directory (images and containers) to an external hard drive via 'daemon.json' file at /etc/docker. It also explains how to modify fstab file to use the external drive mounted in a fixed path in the system.

# Uninstall Docker
See: https://askubuntu.com/questions/935569/how-to-completely-uninstall-docker
```[Bash]
sudo service docker stop
dpkg -l | grep -i docker
sudo apt-get purge -y docker-engine docker docker.io docker-ce 
sudo apt-get autoremove -y --purge docker-engine docker docker.io docker-ce
sudo rm -rf /var/lib/docker
sudo rm /etc/apparmor.d/docker
sudo groupdel docker
sudo rm -rf /var/run/docker.sock
```

# Install Docker
See: https://docs.docker.com/install/linux/docker-ce/ubuntu/
```[Bash]
#Extra steps for AUFS in Ubunutu 14.04
sudo apt-get update
sudo apt-get install \
    linux-image-extra-$(uname -r) \
    linux-image-extra-virtual

#Install Docker CE
sudo apt-get update

#Install support for https
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

#Add Docker’s official GPG key:    
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

#Verify the fingerprint
sudo apt-key fingerprint 0EBFCD88

#Set up the official repository
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
sudo apt-get install docker-ce

#Check installation
sudo docker run hello-world
  Unable to find image 'hello-world:latest' locally
  latest: Pulling from library/hello-world
  d1725b59e92d: Pull complete 
  Digest: sha256:0add3ace90ecb4adbf7777e9aacf18357296e799f81cabc9fde470971e499788
  Status: Downloaded newer image for hello-world:latest
  
  Hello from Docker!
  This message shows that your installation appears to be working correctly.
  
  To generate this message, Docker took the following steps:
   1. The Docker client contacted the Docker daemon.
   2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
      (amd64)
   3. The Docker daemon created a new container from that image which runs the
       executable that produces the output you are currently reading.
   4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.
  
  To try something more ambitious, you can run an Ubuntu container with:
  $ docker run -it ubuntu bash
  
  Share images, automate workflows, and more with a free Docker ID:
  https://hub.docker.com/
  
  For more examples and ideas, visit:
  https://docs.docker.com/get-started/
 
#Check that the daemon is working
ps aux | grep -i docker | grep -v grep
  root     24474  0.6  0.5 902716 92220 ?        Ssl  09:23   0:00 /usr/bin/dockerd --raw-logs
  root     24490  0.2  0.2 663664 39408 ?        Ssl  09:23   0:00 docker-containerd --config /var/run/docker/containerd/containerd.toml

#Post installation steps.
# See: https://docs.docker.com/install/linux/linux-postinstall/
#CreateDocker group and add the $USER to docker group
sudo groupadd docker
sudo usermod -aG docker $USER

#Verify that you can run docker commands without sudo.
docker run hello-world
  Hello from Docker!
  This message shows that your installation appears to be working correctly...

# See Docker installation info
docker info
  Containers: 2
   Running: 0
   Paused: 0
   Stopped: 2
  Images: 1
  Server Version: 18.06.1-ce
  Storage Driver: overlay2
   Backing Filesystem: extfs
   Supports d_type: true
   Native Overlay Diff: true
  Logging Driver: json-file
  Cgroup Driver: cgroupfs
  Plugins:
   Volume: local
   Network: bridge host macvlan null overlay
   Log: awslogs fluentd gcplogs gelf journald json-file logentries splunk syslog
  Swarm: inactive
  Runtimes: runc
  Default Runtime: runc
  Init Binary: docker-init
  containerd version: 468a545b9edcd5932818eb9de8e72413e616e86e
  runc version: 69663f0bd4b60df09991c08812a60108003fa340
  init version: fec3683
  Security Options:
   apparmor
  Kernel Version: 4.4.0-134-generic
  Operating System: Ubuntu 14.04.5 LTS
  OSType: linux
  Architecture: x86_64
  CPUs: 8 
  Total Memory: 15.59GiB
  Name: MSI-GE72
  ID: TT5Q:QK7M:GTXH:KBNX:BCUL:J473:B7KD:SNHN:AOL3:CVGB:GMPW:J5VR
  Docker Root Dir: /var/lib/docker
  Debug Mode (client): false
  Debug Mode (server): false
  Registry: https://index.docker.io/v1/
  Labels:
  Experimental: false
  Insecure Registries:
   127.0.0.0/8
  Live Restore Enabled: false
  WARNING: No swap limit support

#Check whether Docker is running
#See: https://docs.docker.com/config/daemon/#check-whether-docker-is-running
sudo status docker
#docker start/running, process 24474

sudo service docker status
#docker start/running, process 24474

ps -A | grep dockerd
#  24474 ?        00:00:01 dockerd
```

# Move docker's default /var/lib/docker to another directory on Ubuntu 14.04
See: https://adriel.co.nz/blog/2018/01/25/change-docker-data-directory-in-debian-jessie/
Edit file /etc/docker/daemon.json with your favorite text editor and place this lines on it.

Firstable, make sure that the external drive will mount in the same path any time you start your Linux:
1) Plug in your drive
2) Run: 
```[Bash]
sudo blkid -c /dev/null.
```

In my system, the target external drive will be the last one:<br>
/dev/sda1: LABEL="Windows Data" UUID="1EB835F0B835C755" TYPE="ntfs" <br>
/dev/sda2: UUID="02A5-3D12" TYPE="vfat" <br>
/dev/sda3: LABEL="drago" UUID="C4CAA676CAA66480" TYPE="ntfs" <br>
/dev/sda4: LABEL="BIOS_RVY" UUID="082CE6022CE5EAA0" TYPE="ntfs" <br>
/dev/sdb1: LABEL="WinRE tools" UUID="CE8ADC618ADC479B" TYPE="ntfs" <br>
/dev/sdb2: LABEL="SYSTEM" UUID="3CDE-45D9" TYPE="vfat" <br>
/dev/sdb4: LABEL="OS_Install" UUID="5268E08468E06865" TYPE="ntfs" <br>
/dev/sdb5: UUID="239b63d0-a4e2-42a6-a8a8-0782bfc46913" TYPE="swap" <br>
/dev/sdb6: UUID="78d731ab-9290-4316-a541-b30de8e98d8b" TYPE="ext4" <br>
/dev/sdb7: UUID="2cc39344-60c9-4a66-a25e-72edf2a524a7" TYPE="ext4" <br>
/dev/sdc1: LABEL="WDElements_5TB_disk2" UUID="C4BA57BBBA57A926" TYPE="ntfs" <br>
/dev/sdd1: LABEL="hiseq4000hd1" UUID="c825e418-d764-465e-b0a0-16db87f20dd8" TYPE="ext4"<br>

3) Grab the last line.

4) Make a copy of the partitions table:
```[Bash]
sudo cp /etc/fstab /etc/fstab.orig
```

5) Edit fstab and add the new external drive
```[Bash]
sudo gedit /etc/fstab
```

Add this information (see the last line):
```[Bash]
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sdb6 during installation
UUID=78d731ab-9290-4316-a541-b30de8e98d8b /               ext4    errors=remount-ro 0       1
# /boot/efi was on /dev/sda2 during installation
UUID=02A5-3D12  /boot/efi       vfat    defaults        0       1
# swap was on /dev/sdb5 during installation
UUID=239b63d0-a4e2-42a6-a8a8-0782bfc46913 none            swap    sw              0       0
# /home he cambiado su ubicación a /dev/sdb7
UUID=2cc39344-60c9-4a66-a25e-72edf2a524a7 /home           ext4    defaults          0       2
# This is the external khard drive mounted on a fixed folder
UUID=c825e418-d764-465e-b0a0-16db87f20dd8 /media/jlorsal/hiseq4000hd1 ext4 defaults 0 2
```

6) Save and make sure that the changes are ready:
```[Bash]
cat /etc/fstab
```

#Now edit /etc/docker/daemon.json (or creat it) and add these lines:
```[Bash]
#If the file does not exit, creat a new one
{
  "data-root": "/new/path/to/docker-data"
}

#Example:
{
  "data-root": "/media/jlorsal/hiseq4000hd1/docker"
}
```

#Save the file 'daemon.json'

#When ready stop docker service:
```[Bash]
sudo service docker stop
```

#Re-check that the docker daemon is stopped:
```[Bash]
ps aux | grep -i docker | grep -v grep
#No processes related to docker are shown
```

#Now copy /var/lib/docker to the external hard-drive path
```[Bash]
mkdir /new/path/to/docker
sudo rsync -axPS /var/lib/docker/ /new/path/to/docker
```

#Once this is done create a new directory you specified above and optionally rsync current docker 
#data to a new directory:
#Example: mkdir /media/jlorsal/hiseq4000hd1/GATK4Sevilla/docker
```[Bash]
rsync -axPS /var/lib/docker/ /media/jlorsal/hiseq4000hd1/GATK4Sevilla/docker
```

#Start docker service:
```[Bash]
sudo service docker start
```
#NOTE: if this does not work (for Ubuntu 14.04), just reboot your Linux. The daemon will be automatically
#loaded on reboot.


#And check that the paths have changed
```[Bash]
ps aux | grep -i docker | grep -v grep
root      7104  0.1  0.4 898036 78004 ?        Ssl  09:17   0:00 /usr/bin/dockerd --raw-logs
root      7122  0.2  0.2 679544 39536 ?        Ssl  09:17   0:00 docker-containerd --config /var/run/docker/containerd/containerd.toml
```

#Check that the containers and images will go to the new external path
```[Bash]
docker info | grep 'Docker Root Dir'
#WARNING: No swap limit support
#Docker Root Dir: /var/lib/docker
```

#Check it downloading a very light image as 'ningx'
```[Bash]
docker run nginx
  Unable to find image 'nginx:latest' locally
  latest: Pulling from library/nginx
  802b00ed6f79: Pull complete 
  e9d0e0ea682b: Pull complete 
  d8b7092b9221: Pull complete 
  Digest: sha256:24a0c4b4a4c0eb97a1aabb8e29f18e917d05abfe1b7a7c07857230879ce7d3d3
  Status: Downloaded newer image for nginx:latest
```

#Or give a try with 'alpine', another light docker image.
#Check images already installed in docker
```[Bash]
sudo docker images
```

If the target external drive is nor already mounted (i.e. you forgot to plug in the drive before booting your system),
you can do it once the system is running... but you need to stop and start (or restart) the docker service.

# Install GATK4 via docker
```[Bash]
docker pull broadinstitute/gatk:4.0.8.1
  4.0.8.1: Pulling from broadinstitute/gatk
  3620e2d282dc: Pull complete 
  ef22f5e4b3b2: Pull complete 
  99f229f854da: Pull complete 
  4fe433abe16a: Pull complete 
  c9b72a16d85e: Pull complete 
  c50d069245a0: Pull complete 
  c8e7ab072821: Pull complete 
  Digest: sha256:8455ffd27a9311693308c2e1db25638f3b1115c2d891881f264d18a87cadf3e5
  Status: Downloaded newer image for broadinstitute/gatk:4.0.8.1
```

#After GATK4 installation
```[Bash]
docker images
  REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
  alpine                latest              196d12cf6ab1        35 hours ago        4.41MB
  hello-world           latest              4ab4c602aa5e        5 days ago          1.84kB
  nginx                 latest              06144b287844        8 days ago          109MB
  broadinstitute/gatk   4.0.8.1             123712a62f94        3 weeks ago         3.46GB
```

#Start an instance of the GATK4 container by specifying the following command with your particular
#container ID and the filesystem location you want to mount.
```[Bash]
docker run -v /media/jlorsal/hiseq4000hd11/GATK4Sevilla/gatk_bundle_1809:/gatk/my_data -it broadinstitute/gatk:4.0.8.1
#Or
docker run -v /media/jlorsal/hiseq4000hd11/GATK4Sevilla/gatk_bundle_1809:/gatk/my_data -it 123712a62f94
```
#Great work!!!





