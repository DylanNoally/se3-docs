
# Changer l'adresse du réseau

* [Position du problème](#position-du-problème)
* [Solution préconisée](#solution-préconisée)
* [Étapes du changement](#Étapes-du-changement)
* [Vérifications](#vérifications)
* [Cas des `clients-linux`](#cas-des-clients-linux)
* [Problèmes éventuels](#problèmes-éventuels)


## Position du problème

Votre réseau a un adressage ne pouvant gérer qu'un nombre très limité de machines. C'est le cas d'un réseau ayant pour adresse `192.168.1.0/24` (le masque s'occupant des 3 premiers octets, 3×8 = 24, est donc `255.255.255.0`) qui ne peut gérer que 254 machines.

Pour augmenter nettement le nombre d'IP disponible, on peut envisager de passer à une adresse telle que `172.16.0.0/16` ou bien `192.168.0.0/16` (le masque s'occupant des 2 premiers octets, 2×8 = 16, est donc `255.255.0.0`). Autres solutions possibles : `172.17.0.0/16`, `172.18.0.0/16`, `172.19.0.0/16`, `172.20.0.0/16`.

Nous vous proposons ici les principales étapes à suivre pour mener à bien cette opération de changement de l'adresse du réseau.


## Solution préconisée

À noter qu'il faudra coordonner les changements concernant le réseau pédagogique, les changements au niveau des différents switchs et celui concernant la passerelle de votre réseau : cela doit être fait par l'administrateur de cette passerelle (souvent c'est un `Amon`) et de ces switchs (souvent ce sont des `VLAN`), administrateur qui est sans doute la `DSI` de votre académie.

Cependant, pour faciliter ce changement, on peut ne changer que le masque du réseau : ainsi, les différentes opérations pourront se réaliser tranquillement, sans se retrouver dans l'urgence car les machines actuelles seront encore dans le réseau.

Par exemple, vous avez un réseau dont l'adresse est `192.168.1.0/24` : le masque est donc `255.255.255.0`. Nous vous conseillons de passer au réseau d'adresse `192.168.0.0/16` : le masque sera alors `255.255.0.0`, et vous n'aurez ainsi, que ce changement de masque à réaliser avant de prendre le temps de réorganiser le plan d'adressage.

Les avantages sont, d'une part, que les différents changements ne se feront pas dans l'urgence, et d'autre part, que vous aurez plus de souplesse pour les opérations à réaliser. Ce qui ne signifie pas que les choses sont plus faciles pour autant.

Ainsi,vous pourrez commencer par le se3 et le dhcp puis attendre le changement du masque de la passerelle (par exemple un `Amon`) avant de réorganiser le plan d'adressage pour profiter des nouvelles possibilités. Toutefois, le changement de l'ip de la passerelle peut intervenir en premier sans tout bloquer.

Ensuite, vous pourrez modifier petit à petit les `ip` réservées, et les masques des machines ayant des `ip` fixes (sinon elles ne verront qu'une partie des machines du réseau), pour les réorganiser dans le nouveau plan d'adressage.


## Étapes du changement

- utiliser le script `chg_ip_se3.sh` en indiquant la nouvelle IP, masque, passerelle, DNS.

- contacter la `DSI` pour faire changer les adresses (en pratique, le masque) au niveau des switchs/VLAN et l'IP de la passerelle du réseau pédagogique ; passerelle qui, dans le cas de l'Amon, dépend de la `DSI`. Dans le cas d'un `Slis`, normalement on peut le changer sans passer par la `DSI`.

- modifier le service `DHCP` du SE3 avec les nouveaux paramètres (uniquement le masque si vous suivez notre préconisation).

- modifier, si besoin, le proxy Firefox et la page de démarrage Firefox personnalisée.

- modifier le fichier `/etc/wgetrc` pour indiquer éventuellement l'adresse du nouveau proxy pour `http`, `https` et `ftp`.

- supprimer, si nécessaire, les anciennes réservations d'adresses IP et renseigner les nouvelles. Pour cela, on pourra s'aider des possibilités de l'interface web du se3 : http://wwdeb.crdp.ac-caen.fr/mediase3/index.php/Le_module_DHCP#Modifier_le_plan_d.27adressage_des_machines_clientes  
Le principe consiste à exporter les données de l'annuaire dans un fichier au format csv avant de les envoyer dans cette page qui se charge de faire un remplacement automatique des adresses.

- modifier les masques (et les `IP` si nécessaire, selon le nouveau plan d'adressage) des machines en `IP fixes` : serveur antivirus, NAS pédagogique, imprimantes, bornes Wi-Fi…

- modifier le fichier `/etc/apt/apt.conf` pour remplacer l'`IP` de l'`Amon` si nécessaire.

- ré-installer le paquet `sambaedu-client-windows` à l'aide des commandes suivantes :
```sh
apt-get update
apt-get install --reinstall sambaedu-client-windows
```


## Vérifications

La vérification est simple : dans chaque parc de machines, ouvrir une session sur une des machines de ce parc. Tout doit fonctionner.


## Cas des `clients-linux`

Pour retrouver la possibilité d'installer des `clients-linux`, il suffit, dans le module `Serveur FTP` de l'interface web du `se3`, de [supprimer le mécanisme et de valider](../pxe-clients-linux/misenplace.md#mise-à-jour) : les nouveaux paramètres seront pris en compte pour les nouveaux `clients-linux`.

Pour les `clients-linux` déjà installés, il y a la possibilité de les ré-installer, ce qui va assez vite. Cependant, il est possible de modifier quelques fichiers de chaque clients ayant l'ancienne `IP` du `se3`. [Un script](ch_ip_linux.sh) a été mis au point par Louis-Maurice De Souza. Il faudra l'adapter à votre situation.


## Problèmes éventuels

**Problème 1 :** Aucun compte n'arrive à se connecter au serveur, j'obtiens le fameux : "aucun serveur d'accès disponible".

**Solution :** Vérifier s'il n'y a pas une *coquille* dans le masque de sous-réseau (ou un autre paramètre) lors de la procédure de changement d'adresse ip…


**Problème 2 :** Depuis le changement de plan d'adressage IP, les profils Windows ne sont plus complets : seul le partage K est disponible dans les lecteurs réseaux. Plus de fond d’écran, de documents sur le bureau, les pages d’accueil et proxy ne descendent plus sous Firefox (mais bien sous IE ou Chrome) alors qu’ils sont bien renseignés sur l’interface web du se3.

**Solution :** Vérifier que le pare-feu windows est bien désactivé sur les machines...
