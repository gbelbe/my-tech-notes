
- Vérifier que votre routeur est compatible et mettez à jour le firmware.

fonctionnement au démarrage: 

une fois le logiciel flashé, le routeur n’active pas le wifi pour des raisons de sécu, il faut donc se connecter avec un cable Ethernet et taper l’adresse 192.168.1.1 pour entrer dans l’admin puis activer le wifi.

Openwrt est une distribution linux préparée pour des routeurs avec ressources limitées
Le paramétrage se fait par SSH ou avec l'interface graphique "Luci"

installations de softs:

`opkg update
`opkg install xxxx`

La mémoire des routeurs est limitée et il peut être utile d’en rajouter pour pouvoir installer d’autres logiciels (impossible d’installer tailscale ou adguard home sur le mien sans ajouter de la mémoire)

- add memory and disk management to the router: 
`
`opkg update`

`opkg install block-mount kmod-fs-ext4 e2fsprogs parted kmod-usb-storage`
`
- Create a partition table

`DISK="/dev/sda"`

- Create a GPT partition table

`parted -s ${DISK} -- mklabel gpt`

- Create a 7,5 GB partition for extroot

`parted -s ${DISK} -- mkpart primary ext4 1MiB 7500MiB`

- Format the partition as ext4 for extroot

`mkfs.ext4 -L extroot ${DISK}1`

- Ensuite on prépare la partition pour qu’elle soit montée au démarrage sur le routeur:

`# Variables`
`EXTROOT_DEVICE="/dev/sda1"`
`MOUNT_POINT="/mnt/extroot"`

`# Step 1: Unmount the extroot partition if mounted`
`umount ${EXTROOT_DEVICE} 2>/dev/null`

`# Step 2: Create the mount point`
`mkdir -p ${MOUNT_POINT}`

`# Step 3: Mount the extroot partition`
`mount ${EXTROOT_DEVICE} ${MOUNT_POINT}`
`if [ $? -ne 0 ]; then`
    `echo "Error: Failed to mount ${EXTROOT_DEVICE}. Exiting..."`    
 `exit 1`
`fi`

`# Step 4: Copy current root filesystem to extroot partition`
`echo "Copying current root filesystem to extroot partition..."`    
`tar -C /overlay -cvf - . | tar -C ${MOUNT_POINT} -xf -`
`if [ $? -ne 0 ]; then`    
    `echo "Error: Failed to copy root filesystem. Exiting..."`
    `umount ${MOUNT_POINT}`
`exit 1`
`fi`

`# Step 5: Configure extroot in /etc/config/fstab`
`echo "Configuring extroot in /etc/config/fstab..."`
`uci -q delete fstab.overlay`
`uci set fstab.overlay="mount"`
`uci set fstab.overlay.device="${EXTROOT_DEVICE}"`
`uci set fstab.overlay.target="/overlay"`
`uci commit fstab`

`# Step 6: Update fstab for automatic mounting`
`cat << EOF >> /etc/config/fstab`
`config 'mount'`
`option target '/overlay'`
`option device '${EXTROOT_DEVICE}'`
`option fstype 'ext4'`
`option options 'rw,sync'`
`option enabled '1'`
`option enabled_fsck '1'`
`EOF`

`# Step 7: Enable and start the fstab service`
`/etc/init.d/fstab enable`
`/etc/init.d/fstab start`

`# Step 8: Unmount the extroot partition and reboot`
`echo "Unmounting extroot partition and rebooting..."`
`umount ${MOUNT_POINT}`
`echo "Rebooting the system in 5 seconds..."`
`sleep 5`
`reboot`
    

  
## peut on également augmenter la ram?
    


On peut installer le package qui optimise l’usage de la RAM zram-swap

`opkg update`
`opkg install zram-swap`

  

- [Parer aux pertes de connection](https://docs.google.com/document/d/1FOOG4MD-A9jeWVDq9sq60PTSC1il8mkU_1wvndGSgks/edit?tab=t.ckrv6u7mawp5) lors de changement d’adresse de l’ISP (reseau mobile ou autre par ex)
    

  
  
### note : déconnections:

[OpenWRT, ISP modem and dynamic IP addresses: how to fix connectivity issues without rebooting your router every time](https://ounapuu.ee/posts/2024/05/20/openwrt-connectivity-fix/)

2024-05-20

#[free tech tip](https://ounapuu.ee/tags/free-tech-tip/)  #[openwrt](https://ounapuu.ee/tags/openwrt/) 

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXeXlO7sJ-cWpw6lFitT1Pk_vBfnwP7CPKyDedOjdkTh_LGnhzP7CeUoPAUoZbSShtMUmKDCAR-3ueGiGX3biwJ7bD50M1Btoeifsoal1ZLE4rnUKHM-hyJpLa-XPYXmGKJ9Sd56QA?key=0ptkJBDo_711SQ8YF5_g8C5D)

[My current ISP](https://elisa.ee/) provides an internet connection over a copper wire. To use it, I have a crappy modem (Technicolor CGA2121, DOCSIS 3.0). It’s running in bridge mode, meaning that all it does is convert the signal running over the coax cable into plain old Ethernet.

My main networking device is a TP-Link Archer C7 v5. It runs OpenWRT. This router/Wi-Fi AP box connects to the modem and handles everything, including getting a public IPv4 address from the ISP.[1](https://ounapuu.ee/posts/2024/05/20/openwrt-connectivity-fix/#fn:1)

After a power outage or my ISP doing maintenance, the public IP address has usually changed. This wouldn’t be a problem if I just stuck to the ISP-approved modem.[2](https://ounapuu.ee/posts/2024/05/20/openwrt-connectivity-fix/#fn:2)

With my setup, there was a problem. The OpenWRT box would try to operate with the IPv4 address that it was given since the DHCP lease had not yet expired. However, this meant that there was no internet connectivity. A reboot of the OpenWRT box would resolve the issue.

This manual workaround wasn’t good enough for me. It would be quite problematic if this issue happened while I was away from home because I’d still like to access my home server.

After traversing OpenWRT forums and consulting the Slack workspace of my local hackerspace, I found that bringing up the WAN interface again would result in the OpenWRT box getting a new public IPv4 address. Problem solved!

To automate this workaround, I created a single crontab entry in the OpenWRT box. This is also configurable in a graphical user interface as long as you have [LuCI installed.](https://openwrt.org/docs/guide-user/luci/start)

The crontab entry looks like this:

`*/5 * * * * /bin/ash -c '/bin/ping -c 3 8.8.8.8 > /dev/null || /sbin/ifup wan'`

Every 5 minutes, the router pings Google’s DNS server. If that command succeeds, then the internet connection works and that’s it. If the ping fails, then the other half of the shell command is executed, which brings up the wan interface on my router.

Feel free to use a different IP address to test with. Your WAN network interface might also have a different name.

The downside of this solution is that if the server you’re using to verify your internet connection is down or refuses pings, then you’ll be causing interruptions in your home network every 5 minutes.


---

