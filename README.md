## Proxmox:
Proxmox Virtual Environment (Proxmox VE) is a free and open-source virtualization platform that allows you to manage virtual machines (VMs), containers, storage, and networking all in a single web-based interface. It’s especially popular in home labs, data centers, and enterprise environments for virtualization and hyper-converged infrastructure.

**Proxmox VE is based on Debian**. This is why the install disk images (ISO files) provided by Proxmox include a complete Debian system as well as all necessary Proxmox VE packages.


### Key Features:
Proxmox VE is a **Type-1 hypervisor** (bare-metal) based on Debian Linux. Combines **KVM** (Kernel-based Virtual Machine) for full virtualization (VMs) and LXC (Linux Containers) for lightweight containerization.

- **Web Interface**:	Manage VMs, containers, storage, and backups through a browser.
- **Virtual Machines**:	Run OSs like Windows, Linux, BSD, etc., via KVM.
- **Containers (LXC)**:	Lightweight and fast alternative to full VMs.
- **High Availability**:	Clustered nodes can automatically restart VMs if one node fails.
- **Storage Flexibility**:	Support for ZFS, Ceph, local disks, and shared network storage.
- **Backup & Restore**:	Scheduled backups with integrated tools like ** vzdump**.
- **Live Migration**:	Move running VMs between nodes with no downtime.
- **Firewall**:	Built-in firewall for guests and host systems.
- **Snapshots**:	For both VMs and containers.
- **Clustering**:	Manage multiple nodes from a central GUI.


### Proxmox vs Other Platforms:

| Feature          | Proxmox VE          | VMware ESXi     | Hyper-V        | oVirt |
| ---------------- | ------------------- | --------------- | -------------- | ----- |
| **Open Source**  | ✅ Yes               | ❌ Proprietary   | ❌ Proprietary  | ✅ Yes |
| **Web GUI**      | ✅ Built-in          | ❌ (via vCenter) | ✅              | ✅     |
| **Containers**   | ✅ LXC support       | ❌               | ❌              | ❌     |
| **License Cost** | Free + paid support | \$\$\$          | Free (limited) | Free  |



### System Requirements:
- Intel 64 or AMD64 with Intel VT/AMD-V CPU flag.
- Memory, minimum 2 GB for OS and Proxmox VE services. Plus designated memory for guests. For Ceph or ZFS additional memory is required, approximately 1 GB memory for every TB used storage.
- Fast and redundant storage, best results with SSD disks.
- OS storage: Hardware RAID with batteries protected write cache (“BBU”) or non-RAID with ZFS and SSD cache.
- VM storage: For local storage use a hardware RAID with battery backed write cache (BBU) or non-RAID for ZFS. Neither ZFS nor Ceph are compatible with a hardware RAID controller. Shared and distributed storage is also possible.
- Redundant Gbit NICs, additional NICs depending on the preferred storage technology and cluster setup – 10 Gbit and higher is also supported.
- For PCI(e) passthrough a CPU with VT-d/AMD-d CPU flag is needed.



### For Evaluation:
- Minimum Hardware (for testing only)
- CPU: 64bit (Intel 64 or AMD64)
- Intel VT/AMD-V capable CPU/Mainboard (for KVM full virtualization support)
- Minimum 2 GB RAM
- Hard drive
- One NIC



### Using the Proxmox VE Installer: 
The installer ISO image includes the following:

- Complete operating system (Debian Linux, 64-bit)
- The Proxmox VE installer, which partitions the local disk(s) with `ext4`, `XFS`, `BTRFS` (technology preview), or ZFS and installs the operating system
- Proxmox VE Linux kernel with KVM and LXC support
- Complete toolset for administering virtual machines, containers, the host system, clusters and all necessary resources
- Web-based management interface



### Installation Steps:


#### Create Bootable USB:

Use a tool like **Ventoy** (Windows), **Rufus** (Windows) or Balena Etcher (cross-platform) to write the ISO to a USB drive. If you are working on **Linux**, the fastest way to create a bootable USB is to use the following syntax and Replace `/dev/sdX` with your USB device (e.g., /dev/sdd): 

```
dd bs=1M conv=fdatasync if=./proxmox-ve_*.iso of=/device/name

dd bs=1M conv=fdatasync if=/path/to/proxmox-ve_*.iso of=/dev/sdX status=progress
```


#### Installing Proxmox in VMware Workstation:

One the guest operating system screen, select Linux and then under the version, select **Debian 11.x 64-bit** or **Debian 12.x 64-bit**.
- proxmox-ve_7.2-1.iso: Debian 11 (bullseye)
- proxmox-ve_8.2-1.iso: Debian 12 (bookworm)
- Other: FreeBSD 12 x64bit



#### Boot from USB:

1. On the Proxmox VE boot menu, select **Install Proxmox VE (Graphical)** for the standard installer. For older or newer hardware, choose **Install Proxmox VE (Terminal UI)** for better compatibility.
2. Press **Enter** to begin.
3. Accept the EULA: Click “**I agree**” to proceed.
4. Select **Target harddisk**: Choose the disk for installation (`note`: **this will erase all data on the selected disk**). 
    - Click **Options**:  to choose a file system (`ext4`, `XFS`, or `ZFS`). By default, it is set to `ext4`. **ZFS is recommended for advanced features like RAID but requires more resources**.
    - Advanced LVM Configuration Options for `ext4`:
        - `hdsize`: Defines the total hard disk size to be used. This way you can reserve free space on the hard disk for further partitioning (for example for an additional `PV` and `VG` on the same hard disk that can be used for LVM storage).
        - `swapsize`: Defines the size of the swap volume. The default is the size of the installed memory, **minimum 4 GB and maximum 8 GB**. The resulting value cannot be greater than hdsize/8. `Note` If set to 0, no swap volume will be created.
        - `maxroot`: Defines the maximum size of the **root volume**, which stores the operation system. The maximum limit of the root volume size is **hdsize/4**.
        - `maxvz`: Defines the maximum size of the data volume. The actual size of the data volume is: `datasize = hdsize - rootsize - swapsize - minfree` 
            - Where `datasize` cannot be bigger than `maxvz`. 
            - `Note`: In case of LVM thin, the data pool will only be created if `datasize` is bigger than `4GB`.
            - `Note`: If set to 0, no data volume will be created and the storage configuration will be adapted accordingly.
        - `minfree`: Defines the amount of free space that should be left in the LVM volume group `pve`. With more than `128GB` storage available, the default is `16GB`, otherwise hdsize/8 will be used. `Note`: LVM requires free space in the VG for snapshot creation (not required for lvmthin snapshots).

5. Set **Regional Settings**: Choose your country, time zone, and keyboard layout.
6. Set **Root Password and Email**: Enter a strong password for the `root` user and an email for notifications.
7. Configure Network: Set a static IP address (recommended) or use DHCP.
    - Enter a hostname (e.g., `pve1.local` or `pve1.technbd.net`).
    - IP: 192.168.10.191
    - Netmask: 255.255.255.0
    - Gateway: 192.168.10.1
    - DNS: 8.8.8.8
8. Summary or Review and Install: Verify settings on the summary page and click `Install`. The process takes a few minutes.
9. Reboot




### Basic Proxmox VE commands:

```
cat /etc/issue

------------------------------------------------------------------------------

Welcome to the Proxmox Virtual Environment. Please use your web browser to
configure this server - connect to:

  https://192.168.10.191:8006/

------------------------------------------------------------------------------
```


_Show Proxmox version:_

```
pveversion

Or,

pveversion -v 

pve-manager/7.2-3/c743d6c1 (running kernel: 5.15.30-2-pve)
```


_Check status of web interface service:_

```
systemctl status pveproxy
```



_Check host performance:_

```
pveperf

CPU BOGOMIPS:      11616.00
REGEX/SECOND:      3563004
HD SIZE:           24.19 GB (/dev/mapper/pve-root)
BUFFERED READS:    83.86 MB/sec
AVERAGE SEEK TIME: 0.34 ms
FSYNCS/SECOND:     3065.57
DNS EXT:           81.50 ms
DNS INT:           344.79 ms (technbd.net)
```



_Verify the subscription status:_

```
pvesubscription get
```


_Check Hostname:_

```
cat /etc/hostname

pve1
```


```
hostname -f

pve1.technbd.net
```


```
cat /etc/hosts

127.0.0.1 localhost.localdomain localhost
192.168.10.191 pve1.technbd.net pve1
```



```
lsblk

NAME               MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda                  8:0    0   100G  0 disk
├─sda1               8:1    0  1007K  0 part
├─sda2               8:2    0   512M  0 part
└─sda3               8:3    0  99.5G  0 part
  ├─pve-swap       253:0    0     4G  0 lvm  [SWAP]
  ├─pve-root       253:1    0  24.8G  0 lvm  /
  ├─pve-data_tmeta 253:2    0     1G  0 lvm
  │ └─pve-data     253:4    0  58.7G  0 lvm
  └─pve-data_tdata 253:3    0  58.7G  0 lvm
    └─pve-data     253:4    0  58.7G  0 lvm
sr0                 11:0    1 994.2M  0 rom
```


```
df -hT

Filesystem           Type      Size  Used Avail Use% Mounted on
udev                 devtmpfs  1.9G     0  1.9G   0% /dev
tmpfs                tmpfs     390M  972K  389M   1% /run
/dev/mapper/pve-root ext4       25G  2.5G   21G  11% /
tmpfs                tmpfs     2.0G   28M  1.9G   2% /dev/shm
tmpfs                tmpfs     5.0M     0  5.0M   0% /run/lock
/dev/fuse            fuse      128M   16K  128M   1% /etc/pve
tmpfs                tmpfs     390M     0  390M   0% /run/user/0
```



```
free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       1.3Gi       2.2Gi        28Mi       257Mi       2.2Gi
Swap:          4.0Gi          0B       4.0Gi
```


```
pvs

  PV         VG  Fmt  Attr PSize   PFree
  /dev/sda3  pve lvm2 a--  <99.50g 10.00g
```


```
vgs

  VG  #PV #LV #SN Attr   VSize   VFree
  pve   1   3   0 wz--n- <99.50g 10.00g
```


```
lvs

  LV   VG  Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  data pve twi-a-tz-- <58.75g             0.00   1.59
  root pve -wi-ao----  24.75g
  swap pve -wi-ao----   4.00g
```



### Access the Web Interface:

Open a browser on another computer and navigate to `https://your-server-ip:8006` like: (`https://192.168.10.191:8006`).



### Create a VM:

1. Make sure you have upload ISO images for installation mediums:
  - Select your host `pve1` - click `local (pve1)`:
  - Click `ISO Images` - click `Upload` from the menu and choose between uploading an image or downloading it from a URL.

```
ls -l /var/lib/vz/template/iso/

-rw-r--r-- 1 root root  256901120 May 26 13:22 alpine-standard-3.21.3-x86_64.iso
-rw-r--r-- 1 root root 1331691520 May 26 16:45 Ubuntu_20.04_Live_Server-minimal.iso
```


2. Once you have added an ISO image, spin up a virtual machine. Click **Create VM** button.
  - General: 
    - Node: `pve1`
    - VM ID: 100
    - Name: web-server
  - OS:
    - [✔] Use CD/DVD disc image file (iso)
    - Storage: local
    - ISO image: alpine-standard-3.21.3-x86_64.iso
    - Guest OS:
      - Type: Linux
      - Version: 5.x - 2.6 Kernel
  - System:
    - Graphic card: Default
    - Machine:
    - BIOS: 
    - SCSI Controller: VirtIO SCSI
  - Hard Disk: (`Note`: You can leave all the default settings. However, if the physical server uses an **SSD**, enable the **Discard** option.)
    - Disk:
      - Bus/Device: 
      -  SCSI Controller: VirtIO SCSI
      - Storage: local-lvm
      - Disk Size (GiB): 10
      - Cache: Default (No cache)
      - Discard: [ ]
  - CPU:
    - Sockets:
    - Cores:
    - Type: Default (kvm64)
    - Total cores: 
  - Memory: 
    - Memory (MiB): 512
  - Network:
    - Bridge: `vmbr0`
    - VLAN Tag:
    - Model: VirtIO (paravirtualized)
    - MAC: auto
    - Firewall: [✔]
  - Confirm:
    - [✔] Start after created
    - Click Finish to create the VM. 


#### Start VM at Boot:

If the **Start at boot** option is set to `No`, the VM does not automatically start after rebooting the server. This means you need to log in to the Proxmox interface and start the VM manually.
- Select your VM: `100 (web-server)`
- Click Options:
  - Select `Start at boot` - click `Edit`: Start at boot [✔] - OK



### Post-Installation:

Update Proxmox: If you don’t have a subscription, Comment out the enterprise repository by adding hash (`#`) and configure the community repository for updates:

```
ls -l /etc/apt/sources.list.d/

-rw-r--r-- 1 root root 72 May 26 15:25 pve-enterprise.list
```


```
nano /etc/apt/sources.list.d/pve-enterprise.list

#deb https://enterprise.proxmox.com/debian/pve bullseye pve-enterprise
```


_Add the community repository:_
```
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list
```


_Update and upgrade:_
```
apt update && apt full-upgrade
```




### VM/Container Management:

_List all VMs:_
```
qm list

      VMID NAME                 STATUS     MEM(MB)    BOOTDISK(GB) PID
       100 web-server           running    512               10.00 57965
```


_Stop a VM:_
```
qm stop <VMID>
```


_Start a VM:_
```
qm start <VMID>
```


```
qm status <VMID>
```


```
qm reboot <VMID>
```


_Forcefully resets (reboots) VM:_
```
qm reset <VMID>
```



_Gracefully shut down a VM:_
```
qm shutdown <VMID>
```


_Displays the configuration of a specific VM:_
```
qm config <VMID>
```


_A VM or container can become locked during certain operations such as: Backup or restore, Snapshot, Migration, Clone:_
```
qm lock <VMID>
qm unlock <VMID>
```



_Delete a VM:_
```
qm destroy <VMID>	
```



_For Container:_
```
pct list
pct stop <CTID>
pct start <CTID>
pct destroy <CTID>
```



#### Create a VM via CLI:

_Find next free VM ID:_
```
pvesh get /cluster/nextid 
```


#### Explanation:
- `101`: VMID (unique identifier; check existing IDs with qm list).
- `--name`: Name of the VM (e.g., ubuntu-vm).
- `--memory`: RAM in MB (e.g., 2048 MB = 2 GB).
- `--cores`: Number of CPU cores.
- `--sockets`: Number of CPU sockets (usually 1).
- `--net0`: Network interface (uses virtio driver, bridged to `vmbr0`).
- `--boot`: Boot order (e.g., boot from `scsi0` disk).
- `--scsi0`: Disk storage and size (e.g., 32 GB on `local-lvm`).
- `--cdrom`: Path to the ISO image for installation.
- `--ostype l26`: OS type (Linux 2.6+ kernel, common for modern Linux distros).
- `--agent 1`: Enables QEMU Guest Agent for better integration.



```
qm create 101 \
  --name ubuntu-vm \
  --memory 256 \
  --cores 2 \
  --sockets 1 \
  --net0 virtio,bridge=vmbr0 \
  --boot order=scsi0 \
  --scsi0 local-lvm:10 \
  --cdrom local:iso/Ubuntu_20.04_Live_Server-minimal.iso \
  --ostype l26 \
  --agent 1
```


_Check the VM configuration:_
```
qm list
qm config 101
```



_Start the VM to begin the OS installation:_
```
qm start 101
```




_View sum of memory allocated to VMs and CTs:_
```
grep -R memory /etc/pve/local | awk '{sum += $NF } END {print sum;}'
```


_View sorted list of VMs like vmid proxmox_host type:_
```
cat /etc/pve/.vmlist | grep node | tr -d '":,'| awk '{print $1" "$4" "$6 }' | sort -n | column -t
```


_View sorted list of vmid:_
```
cat /etc/pve/.vmlist | grep node | cut -d '"' -f2 | sort -n
```



#### VM Memory Resize:

```
qm list

      VMID NAME                 STATUS     MEM(MB)    BOOTDISK(GB) PID
       100 web-srv              running    512               10.00 975
       101 ubuntu-vm            running    2048              20.00 5690
```


```
qm stop 101
qm config 101
qm config 101 | grep ^memory
```


```
qm set 101 --memory 512
```


```
qm start 101
```


### Serial terminal/console:

To access a Proxmox VM’s console directly from the host CLI, you can use `qm monitor`, `qm terminal`, or `screen` (for serial consoles). 

_Add a virtual serial port to the VM:_
```
nano /etc/pve/qemu-server/101.conf

serial0: socket
```

Or,

```
qm set 101 --serial0 socket
```



_Access the VM console from the Proxmox host CLI (press `Ctrl+O` to exit):_
```
qm terminal 101
```


#### Configure the Guest OS or VM (Optional but Recommended):

_Edit the GRUB config file and modify this line or append it to existing options:_
```
nano /etc/default/grub

GRUB_CMDLINE_LINUX_DEFAULT="console=ttyS0"
```


_Update GRUB:_
```
update-grub
```


_Enable a serial login console (on Debian/Ubuntu):_
```
systemctl enable serial-getty@ttyS0.service
```


`Note`: After that reboot the VM.



```
qm terminal 101
```



### Network:

The configuration is a Proxmox VE network configuration file, typically found in `/etc/network/interfaces` on the Proxmox host. It sets up a network bridge (`vmbr0` bridge to the physical NIC `ens33`) for VMs and enables NAT (Network Address Translation) to allow VMs on a private subnet (`10.10.10.0/24`) to access external networks via the Proxmox host’s IP.

The file shows that `vmbr0` is the **default bridge public network for Proxmox host**, as in the example below:

```
cat /etc/network/interfaces


auto lo
iface lo inet loopback

iface ens33 inet manual

auto vmbr0
iface vmbr0 inet static
        address 192.168.10.191/24
        gateway 192.168.10.1
        bridge-ports ens33
        bridge-stp off
        bridge-fd 0
```


#### Enable NAT Networking Mode:

To make it permanent, edit `/etc/sysctl.conf` or create a file in `/etc/sysctl.d/`:

```
echo "net.ipv4.ip_forward=1" > /etc/sysctl.d/99-ipforward.conf
sysctl -p /etc/sysctl.d/99-ipforward.conf
```


**Enables IP forwarding on the Proxmox host**, allowing it to route traffic between the VM’s private network and the external network: 

```
nano /etc/network/interfaces

auto lo
iface lo inet loopback

iface ens33 inet manual

auto vmbr0
iface vmbr0 inet static
        address 192.168.10.191/24
        gateway 192.168.10.1
        bridge-ports ens33
        bridge-stp off
        bridge-fd 0
post-up echo 1 > /proc/sys/net/ipv4/ip_forward
        post-up   iptables -t nat -A POSTROUTING -s '10.10.10.0/24' -o vmbr0 -j MASQUERADE
        post-down iptables -t nat -D POSTROUTING -s '10.10.10.0/24' -o vmbr0 -j MASQUERADE
```




If you want a VM to use `10.10.10.2` and get NAT access, **create a new bridge** in `/etc/network/interfaces` on the proxmox host has `vmbr1` configured like this:

```
nano /etc/network/interfaces

auto vmbr1
iface vmbr1 inet static
    address 10.10.10.1
    netmask 255.255.255.0
    bridge_ports none
    bridge_stp off
    bridge_fd 0
```



```
systemctl status networking
systemctl restart networking
```



_Then a VM connected to `vmbr1`:_
```
qm set 101 --ipconfig0 ip=10.10.10.3/24,gw=10.10.10.1
qm set 101 --nameserver 8.8.8.8
qm set 101 --net0 virtio,bridge=vmbr1
```



#### Set IP on Guest VM (Set as Manual): 

_Assigns the VM a private IP and sets the Proxmox host’s IP as the gateway, as it handles NAT._

```
vim /etc/netplan/00-installer-config.yaml

# This is the network config written by 'subiquity'
network:
  ethernets:
    ens18:
      dhcp4: no
      addresses: [10.10.10.2/24]
      gateway4: 192.168.10.191
      nameservers:
        addresses: [8.8.8.8]
  version: 2
```


```
netplan apply
```


_Check from inside the VM:_
```
ping 8.8.8.8
```




### Storage Backup and Restore:

Backup modes:
- Stop mode
- Suspend mode
- Snapshot mode


_Show Proxmox storage status:_
```
pvesm status

Name             Type     Status           Total            Used       Available        %
local             dir     active        25368820         4327068        19727756   17.06%
local-lvm     lvmthin     active        61599744         2833588        58766155    4.60%
```


_List all contents stored in the local storage in your Proxmox host:_
- `pvesm` = Proxmox VE Storage Manager
- `list` = List contents of storage
- `local` = Storage ID (usually maps to `/var/lib/vz`)

```
pvesm list local

Volid                                                    Format  Type            Size VMID
local:backup/vzdump-qemu-101-2025_05_26-17_54_02.vma.zst vma.zst backup     901146304 101
local:iso/alpine-standard-3.21.3-x86_64.iso              iso     iso        256901120
local:iso/Ubuntu_20.04_Live_Server-minimal.iso           iso     iso       1331691520
local:vztmpl/ubuntu-20.04-standard_20.04-1_amd64.tar.gz  tgz     vztmpl     214203058
local:vztmpl/centos-7-default_20190926_amd64.tar.xz      txz     vztmpl      68649456
local:vztmpl/ubuntu-22.04-standard_22.04-1_amd64.tar.zst tzst    vztmpl     129824858
```


_Storage Types in Output:_

- `iso` → ISO images for VMs
- `vztmpl` → LXC container templates
- `backup` → Backups made via vzdump
- `snippets` → Cloud-init files or custom configs




_Shows the LXC container templates:_

- `pveam` = Proxmox VE Appliance Manager
- `list` = list available templates
- `local` = refers to the storage name (usually `/var/lib/vz`)

```
pveam list local

NAME                                                         SIZE
local:vztmpl/centos-7-default_20190926_amd64.tar.xz          65.47MB
local:vztmpl/ubuntu-20.04-standard_20.04-1_amd64.tar.gz      204.28MB
local:vztmpl/ubuntu-22.04-standard_22.04-1_amd64.tar.zst     123.81MB
```




_Backup a VM/container:_
```
vzdump <VMID>	
vzdump 101
```


_Backup with compression:_
```
vzdump <VMID> --mode stop --compress zstd	
```



_VM/Container Backup (GUI):_

1. Go to Dashboard: 
  - Select host `pve1` - select your VM `101 (ubuntu-vm)`- click `Backup` - click `Backup now`:
  - Storage: local
  - Mode: Stop
  - Compression: ZSTD (fast and good)
  - click `Backup`



_VM/Container Restore:_
1. Go to Dashboard: 
  - Select host `pve1` - select storage - `local (pve1)` - click `Backups` - select desire backup file `vzdump-qemu-101-2025_05_26-17_54_02.vma.zst`  - click `Restore` 
    - Source: vzdump-qemu-101-2025_05_26-17_54_02.vma.zst
    - Storage: local-lvm
    - VM: 101 
    - Bandwidth limit:
    - Unique:
    - Override setting:
      - Name: ubuntu-vm
      - Memory: 1024
      - Cores: 1
      - Sockets: 1
    - click `Restore`




```
ls -l /var/lib/vz/dump

-rw-r--r-- 1 root root       1771 May 26 17:48 vzdump-qemu-101-2025_05_26-17_48_08.log
-rw-r--r-- 1 root root 2501670400 May 26 17:48 vzdump-qemu-101-2025_05_26-17_48_08.vma
```







### Links:
- [Downloads Proxmox](https://www.proxmox.com/en/downloads/proxmox-virtual-environment)
- [Comparing Proxmox VE to alternatives](https://www.proxmox.com/en/products/proxmox-virtual-environment/comparison)
- [Proxmox LVM options](https://pve.proxmox.com/wiki/Installation#advanced_lvm_options)
- [Proxmox ZFS options](https://pve.proxmox.com/wiki/Installation#advanced_zfs_options)
- [Install Proxmox in VMware](https://www.virtualizationhowto.com/2024/05/install-proxmox-in-vmware-workstation-pro/)
- [Install Proxmox VE](https://phoenixnap.com/kb/install-proxmox)
- [Serial Terminal](https://pve.proxmox.com/wiki/Serial_Terminal)
- [Proxmox VE Scripts](https://tteck.github.io/Proxmox/)



