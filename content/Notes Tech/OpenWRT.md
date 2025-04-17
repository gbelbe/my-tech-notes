
Vérifier que votre routeur est compatible et mettez à jour le firmware.

  
fonctionnement au démarrage: 

1. une fois le logiciel flashé, le routeur n’active pas le wifi pour des raisons de sécu, il faut donc se connecter avec un cable Ethernet et taper l’adresse 192.168.1.1 pour entrer dans l’admin puis activer le wifi.
    

2. paramétrage par SSH
    

  

Openwrt est une distribution linux préparée pour des routeurs avec ressources limitées

  

installations de softs:

`opkg update

`opkg install xxxx`
`
La mémoire des routeurs est limitée et il peut être utile d’en rajouter pour pouvoir installer d’autres logiciels (impossible d’installer tailscale ou adguard home sur le mien sans ajouter de la mémoire)


add memory to the router:

add disk management packages   
`
`opkg update`

`opkg install block-mount kmod-fs-ext4 e2fsprogs parted kmod-usb-storage`
`
Create a partition table

`DISK="/dev/sda"`

### Create a GPT partition table

`parted -s ${DISK} -- mklabel gpt`

### Create a 7,5 GB partition for extroot

`parted -s ${DISK} -- mkpart primary ext4 1MiB 7500MiB`

### Format the partition as ext4 for extroot

`mkfs.ext4 -L extroot ${DISK}1`

Ensuite on prépare la partition pour qu’elle soit montée au démarrage sur le routeur:

1. # Variables
    
2. EXTROOT_DEVICE="/dev/sda1"
    
3. MOUNT_POINT="/mnt/extroot"
    
4. # Step 1: Unmount the extroot partition if mounted
    
5. umount ${EXTROOT_DEVICE} 2>/dev/null
    
6. # Step 2: Create the mount point
    
7. mkdir -p ${MOUNT_POINT}
    
8. # Step 3: Mount the extroot partition
    
9. mount ${EXTROOT_DEVICE} ${MOUNT_POINT}
    
10. if [ $? -ne 0 ]; then
    
11.     echo "Error: Failed to mount ${EXTROOT_DEVICE}. Exiting..."
    
12.     exit 1
    
13. fi
    
14. # Step 4: Copy current root filesystem to extroot partition
    
15. echo "Copying current root filesystem to extroot partition..."
    
16. tar -C /overlay -cvf - . | tar -C ${MOUNT_POINT} -xf -
    
17. if [ $? -ne 0 ]; then
    
18.     echo "Error: Failed to copy root filesystem. Exiting..."
    
19.     umount ${MOUNT_POINT}
    
20.     exit 1
    
21. fi
    
22. # Step 5: Configure extroot in /etc/config/fstab
    
23. echo "Configuring extroot in /etc/config/fstab..."
    
24. uci -q delete fstab.overlay
    
25. uci set fstab.overlay="mount"
    
26. uci set fstab.overlay.device="${EXTROOT_DEVICE}"
    
27. uci set fstab.overlay.target="/overlay"
    
28. uci commit fstab
    
29. # Step 6: Update fstab for automatic mounting
    
30. cat << EOF >> /etc/config/fstab
    
31. config 'mount'
    
32.     option target '/overlay'
    
33.     option device '${EXTROOT_DEVICE}'
    
34.     option fstype 'ext4'
    
35.     option options 'rw,sync'
    
36.     option enabled '1'
    
37.     option enabled_fsck '1'
    
38. EOF
    
39. # Step 7: Enable and start the fstab service
    
40. /etc/init.d/fstab enable
    
41. /etc/init.d/fstab start
    
42. # Step 8: Unmount the extroot partition and reboot
    
43. echo "Unmounting extroot partition and rebooting..."
    
44. umount ${MOUNT_POINT}
    
45. echo "Rebooting the system in 5 seconds..."
    
46. sleep 5
    
47. reboot
    

  

  



## peut on augmenter la ram?
    

  

On peut installer le package qui optimise l’usage de la RAM zram-swap

opkg update

opkg install zram-swap

  
  

- [Parer aux pertes de connection](https://docs.google.com/document/d/1FOOG4MD-A9jeWVDq9sq60PTSC1il8mkU_1wvndGSgks/edit?tab=t.ckrv6u7mawp5) lors de changement d’adresse de l’ISP (reseau mobile ou autre par ex)
    

  
  
## Déconnections:

  **

# [OpenWRT, ISP modem and dynamic IP addresses: how to fix connectivity issues without rebooting your router every time](https://ounapuu.ee/posts/2024/05/20/openwrt-connectivity-fix/)

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

*/5 * * * * /bin/ash -c '/bin/ping -c 3 8.8.8.8 > /dev/null || /sbin/ifup wan'

Every 5 minutes, the router pings Google’s DNS server. If that command succeeds, then the internet connection works and that’s it. If the ping fails, then the other half of the shell command is executed, which brings up the wan interface on my router.

Feel free to use a different IP address to test with. Your WAN network interface might also have a different name.

The downside of this solution is that if the server you’re using to verify your internet connection is down or refuses pings, then you’ll be causing interruptions in your home network every 5 minutes.

Talking to the ISP about this issue was something I considered as well. Then I remembered that it took me 1.5 months of fighting chatbots and repeating the same information to different customer care agents to use my own modem that’s identical to the one the ISP uses. That’s a hell no from me.

---

  

  
**