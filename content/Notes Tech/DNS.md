**

## Redirection des requêtes DNS vers Adguardhome plutot que vers Dnsmasq (openwrt default)

  

Le système de filtrage de pub Adguardhome fonctionne sur la base d’un filtrage de DNS, il doit donc être utilisé comme DNS principal.

  
  

>> pas nécessaire a priori << 

Pour s’assurer que toutes les requêtes dns qui passent vers l’extérieur utilisent bien le serveur local (adguardhome),

on modifie l’interface WAN pour y ajouter l’adresse IP du routeur comme adresse DNS. (192.168.1.1)

>> a tester <<

  
  

Si l’on veut éviter que les requetes DNS passent par dnsmasq (le DNS par défaut dans openwrt) on doit modifier le port DNS dans Openwrt (menu dhcp et dns port)

  

Le port par défaut des serveurs dns est le 53 on doit donc modifie le port paramétre par défaut dans Openwrt (dhcp et dns) vers un autre port (le port 54 par exemple).

  

On peut aussi activer l’option “Local service only” dans openwrt menu DHCP et DNS / Filter

(a priori laisser désactivé l’option protect from rebind)

  

Celà fait que Dnsmasq n’est plus utilisé pour résoudre les noms de domaines, qui passeront dorénavant par Adguardhome.

  
  
  
  

## Gestion des noms de domaines locaux

  

On peut souhaiter néanmoins que la résolution des domaines locaux soient gérés par Dnsmasq (car adguardhome ne s’en occupe pas).

  

Pour celà il faut ajouter une règle dans Adguardhome pour filtrer les requêtes qui vont vers les domaines locaux (.lan si c’est configuré comme ça dans Openwrt)

  

On ajoute la ligne 

  

[/lan/]127.0.0.1:54

pour indiquer que l’on doit rediriger toutes les requêtes sur le domaine .lan vers le port 54

  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdE2mY0I7fk3BjdPio1XcbPXE4_UOAdBGnZi7dVpopOKFqdZtKZ9QA-nfDmS_JDmIEYxFpsiKeS8bzpSjnWbkLiny3KIPrC9W_P--AtimaI3rfaazOyGh5Rnngfm1__787falOKtw?key=0ptkJBDo_711SQ8YF5_g8C5D)

  

Quand on est sur le réseau local (en wifi par exemple) on peut désormais accéder aux services par leurs noms de domaine (par ex: nas.lan ou openwrt.lan …. ) plutôt que par leur adresse IP.

  

--------

  

on peut modifier les dns upstream pour mettre ceux de Cloudflare avec option parental control et malware

[https://blog.cloudflare.com/introducing-1-1-1-1-for-families/](https://blog.cloudflare.com/introducing-1-1-1-1-for-families/)

  

Malware and Adult Content

Primary DNS: 1.1.1.3  Secondary DNS: 1.0.0.3

Primary DNS: 2606:4700:4700::1113  Secondary DNS: 2606:4700:4700::1003

  
  
  

-------

  
  

Quand on y accède par Tailscale en mode VPN on n’est pas sur le réseau local donc le DNS ne marche pas.

  

Pour le faire marcher on va sur le menu DNS sur le site de Tailscale et on ajoute un DNS avec l’option “restrict to domain / split dns”  par exemple j’ai mis l’adresse IP locale de mon routeur OpenWrt avec l’option de résoudre les domaines .lan (domaines locaux)

  

Donc quand j’accède à un site se terminant par .lan avec tailscale activé, il utilisera le dns local d’Openwrt.

  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXe6vY-uokfDVplB4cH8-wIbn_0auO74tvU4hLVmDA1gP9YeZ10MvZgNtoxRSMd9AhDR-shiUYeX3DXSz_3PvoBmZKdt4-SWVF1-kL6ofUjbKU2j_yRJMFf6VjXJjmR91RR20iT3bw?key=0ptkJBDo_711SQ8YF5_g8C5D)

  

Attention : conflit potentiel avec des noms de domaines automatiques donnés par mdns (par ex homeassistant.local  n’est pas géré par le dns mais par un autre système.

  

J’ai du modifier la configuration des DNS sur Openwrt pour permettre à tous les domaines locaux d’avoir une adresse en .lan

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfT8GSRqaN7Jr7KIxFrYDGzIzOibRrTvH1AqxBd0MBMfegYt-kN3WLbeDtZzAbEuLMnu_kMk7UV6s-Vi_44z0_KeUv9fsH3rOPxpIkODhhB89kUT3n8ME-H_aOZ231HloRqtL2T?key=0ptkJBDo_711SQ8YF5_g8C5D)

  

Choses à vérifier sur la résolution des noms de domaines locaux:

  

vérifier à ce que l’accès est bien en http si le serveur n’est pas configuré en https 

veiller à ce que le l’option “rebind protection” n’est pas cochée dans le menu “Filter” de DHCP & DNS d’Openwrt

Si un des services n'apparaît pas dans la liste, sur la page d’accueil d’Openwrt il faut parfois le redémarrer  pour qu’il communique son nom en DHCP au serveur dns. (ex home assistant) il ne suffit pas de rebrancher la prise réseau, il faut aussi le faire redémarrer.

**