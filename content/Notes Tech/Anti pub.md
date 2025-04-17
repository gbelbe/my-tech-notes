**  

1. ## Filtrage des pubs niveau réseau par DNS
    

  

Applications Adblock, adblock-lean ou Adguard home installables sur openwrt

  

[https://forum.openwrt.org/t/how-to-install-adblock/195784/3](https://forum.openwrt.org/t/how-to-install-adblock/195784/3)

  

On peut installer un logiciel de filtrage de Pub basés sur les DNS (il redirige vers un dns local tous les noms de domaines connus comme étant des serveurs de pubs)  c’est le concept de pihole 

  

L’avantage de ce système c’est que comme il est installé au niveau du routeur sur Openwrt, toutes les machines du réseau connectées au Wifi en profitent sans aucune conf ni aucun plugin à installer.

  

Des logiciels qui fonctionnent et prennent peu de mémoire sur openwrt sont adblock ou adblock-lean dispos sur software dans openwrt

  

pour tester: [https://canyoublockit.com/](https://canyoublockit.com/)

  

Si le VPN est activé (avec l’option splitDNS de Tailscale) il peut fonctionner sur le téléphone hors du réseau wifi. Pour celà il faut forcer à utiliser l’IP d’openwrt (ou est installé l’anti-pub) sur toutes les machines tailscale.

  

configuration sur le site de tailscale dans le panel DNS: 

[https://tailscale.com/kb/1114/pi-hole](https://tailscale.com/kb/1114/pi-hole)

  

note: on doit mettre l’option “--accept-dns=false” dans la commande de démarrage de tailscale sur oopenwrt pour que l’on n’ait pas un DNS récurrent.

L’anti pub d’openwrt qui appelle le DNS écrit sur tailscale qui le rappelle une fois de plus car il est configuré comme DNS principal sur le site de tailscale

  
  

Bug avec adguardhome qui perd la connection internet au bout d’un moment.

(aller voir le log du routeur rempli de requetes non abouties)

  

→ Mieux vaut désactiver les options de filtrages dns parentaux et sécu d’adguardhome qui gèrent de la latence et les remplacer par les DNS similaires de cloudflare dans la case DNS upstream de adguardhome

  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcWlOSaBiIc1VMclHKSQofEuOi5Nzuauh3F3dRx4EpnfHSrr0uHsviQG2OJ6KS3kHazG6hjFp79NqWwDpeP49_V8TOZmdRLxuPCjR6LXEIe-PeVFnXfZhtFswM8j8j5bS_REPme?key=0ptkJBDo_711SQ8YF5_g8C5D)

  
  

[https://www.reddit.com/r/Adguard/comments/1f15rc4/installing_adguard_home_in_openwrt_fix_issue_with/](https://www.reddit.com/r/Adguard/comments/1f15rc4/installing_adguard_home_in_openwrt_fix_issue_with/)

  

After installing AdGuard Home in OpenWrt 23.05.4 following the [official guide](https://openwrt.org/docs/guide-user/services/dns/adguard-home) in the OpenWrt documentation, there will be a problem at the end that the OpenWrt router will lose internet connectivity, and a [fix has to be applied](https://forum.openwrt.org/t/how-to-updated-2021-installing-adguardhome-on-openwrt-manual-and-opkg-method/113904/686) to the /etc/adguardhome.yaml file.

Initially the adguardhome.yaml file will only have the IPv4 address of the router in the dns: --> bind_hosts: section:

dns:

  bind_hosts:

    - 192.168.1.1

  port: 53

And you have to add the localhost information for IPv4 and IPv6, also the IPv6 address of the router, something like this:

dns:

  bind_hosts:

    - 127.0.0.1

    - ::1

    - 192.168.1.1

  port: 53

  
  
  

# 2) Youtube sans Pub

  

→ Il est possible de profiter des vidéos de youtube sans pub, même sur un Iphone avec une application spéciale (yattee)

  

L’application intègre sponsorblock qui détecte les pubs au moment ou elles arrivent et les passent immédiatement. Il est impossible de supprimer ces pubs côté serveur sans logiciel particulier car on ne peut les détecter par des DNS, ce sont des clips qui sont dans la Vidéo.

  
  

Note: le remplaçant de Youtube Vanced: projet “revanced”:  [https://revanced.app/](https://revanced.app/)  pour android

  

Pour ios, avec quelques manips on y arrive

 [https://www.reddit.com/r/ios/comments/1cf6vqg/does_youtube_revanced_work_on_ios/](https://www.reddit.com/r/ios/comments/1cf6vqg/does_youtube_revanced_work_on_ios/)

  
  

Pour ce qui concerne les PC, il faut un navigateur comme firefox ou brave avec plugin si nécessaire.

**



Side loading apps on IPHONE

**

L’union européenne a forcé Apple à autoriser le sideloading d’applications sur IOS,

  

pour ça vous devez aller sur le site altstore.io puis télécharger l’application qui vous permettra d’installer des apps qui ne sont pas sur l’appstore.

**
