**VPN

  

1. Wireguard: pour créer un vpn entre le réseau local et un téléphone ou autre: celà n’est possible que si le routeur a une adresse IP publique (visible de l’extérieur). Si l’adresse publique est dynamique: elle change tous les jours il faut l’actualiser avec un site de ddns (dns dynamique)
    

  

résultat beaucoup d’installs a faire + le service web de ddns, en plus finalement je n’avait pas d’IP publique sur sfr…

  

Attention: pas d'ip publique sur un réseau 4G sfr: on est derrière un double NAT ( sfr reroute l'ip d'accès vers une ip publique pas directement accessible)

donc pas possible d'utiliser un vpn sur le routeur directement. (l'IP publique accessible depuis whatismyip n'est pas la même que celle que l'on voit affichée sur l'interface WAN de openwrt, qui provient d'une plage d'ip interne à sfr)

[https://la-communaute.sfr.fr/t5/forfaits-sfr/adressse-ip-publique-et-priv%C3%A9e-sur-sim-4g/td-p/2196856](https://la-communaute.sfr.fr/t5/forfaits-sfr/adressse-ip-publique-et-priv%C3%A9e-sur-sim-4g/td-p/2196856)

  

2. Tailscale
    

  

Tailscale est un service cloud de VPN qui permet d’établir facilement un VPN même sans adresse publique. Il faut installer tailscale sur OpenWRT puis sur les telephones qui doivent avoir accès.

On crée un compte sur tailscale.com et on ajoute les machines sur lesquelles l’appli a été installée.

  

Installation tailscale sur openwrt:

[https://openwrt.org/docs/guide-user/services/vpn/tailscale/start](https://openwrt.org/docs/guide-user/services/vpn/tailscale/start)

###   

  

1. opkg update
    
2. opkg install tailscale
    
3. Correct a bug in config 
    

add the word “GOGC=10”   to the file  /etc/init.d/tailscale, like below  and restart the router

procd_set_param env TS_DEBUG_FIREWALL_MODE="$fw_mode" GOGC=10

  

  

After installing Tailscale, run the command below and finish device registration by pasting the given link into a web browser and authenticating via a supported method:

tailscale up

Once registered, device connectivity can be seen by using the “status” command:

tailscale status

  

Create a new unmanaged interface via LuCI: Network → Interfaces → Add new interface

- Name: tailscale
    
- Protocol: Unmanaged
    
- Device: tailscale0
    

Verify that the interface has had your Tailscale address assigned:

ip address show tailscale0

  

On doit ensuite ajouert des règles au Firewall:

  

Create a new firewall zone via LuCI: Network → Firewall → Zones → Add

- Name: tailscale
    
- Input: ACCEPT (default)
    
- Output: ACCEPT (default)
    
- Forward: ACCEPT
    
- Masquerading: on
    
- MSS Clamping: on
    
- Covered networks: tailscale
    
- Allow forward to destination zones: Select your LAN (and/or other internal zones or WAN if you plan on using this device as an exit node)
    
- Allow forward from source zones: Select your LAN (and/or other internal zones or leave it blank if you do not want to route LAN traffic to other tailscale hosts)
    

Click Save & Apply, puis rebooter le routeur

  
  

on active ensuite Tailscale pour écouter le réseau local avec:

  

tailscale up --advertise-routes=192.168.1.0/24 --accept-routes --accept-dns=false

une fois fait, on va voir sur le site tailscale.com, - Open the [Machines](https://login.tailscale.com/admin/machines) page in the Tailscale admin interface. Once you've found the machine from the ellipsis icon menu, open the Edit route settings.. panel, and approve exported routes

  
  
  

On peut suivre l’explication pour créer un subnet : on ouvre un accès à tout le réseau local a partir de la machine ou est installée Tailscale (le routeur wifi openwrt)

Pas la peine d’ouvrir un exit node. (pour moi je n’ai pas l’intention de sortir du réseau local verrs le web, mais seulement d’avoir accès aux services du reseau local à partir d’une machine extérieure).

  

tailscale up --advertise-routes=192.168.1.0/24 --accept-routes

  
  

On doit ensuite paramétrer un DNS global sur le site tailscale.com.

Si l’on veut profiter du filtrage du pub une fois le dns activé, on crée une adresse DNS locale, celà forcera toutes les requêtes de DNS à passer par Adguardhome. 

  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXebkMiPavazzysn38UzZ2hR18mmyUrA3QDy55jwV2_koKFSWRu_G6Q00DUfQJZcAmRCgtEs0rY4TuiVqKAbbTdF9WtnWnQHzAIyv19smfmkkHYBda9n0azJuArhCxYfNSyjhGeO8Q?key=0ptkJBDo_711SQ8YF5_g8C5D)**