# dotNet Core on Kubernetes

Application ASP.NET Core déployé en local dans un cluster Kubernetes à node unique


# 1. Installation des outils

## A. Debian 10 on Hyper-V (windows 10)

### I.Prérequis

L’installation ou l’activation de Hyper-V sur Windows 10 nécessite les prérequis suivants :
- Windows 10 Entreprise, Professionnel ou Éducation
- Processeur 64 bits avec traduction d’adresse de second niveau (SLAT).
- Prise en charge du processeur pour l’extension du mode moniteur (VT-c sur les processeurs Intel)
- Minimum de 4 Go de mémoire
- De l’espace disque

### II.Activation Hyper-V sous windows 10

Pour activer Hyper-V sous Windows 10, exécutez la commande PowerShell suivante en tant qu’administrateur:
```Powershell
PS > Enable-WindowsOptionalFeature -Online -FeatureName:Microsoft-Hyper-V -All
````

### III.Mise en réseau de la machine virtuelle

Mise en réseau NAT (NAT = Network Address Translation) permet à la machine virtuelle d’accéder au réseau de l’ordinateur « hôte » en combinant l’adresse IP de l’hôte avec un port par le biais d’un commutateur virtuel Hyper-V interne.

On crée le commutateur interne:
```Powershell
PS > New-VMSwitch -SwitchName "{{switch name}}" -SwitchType Internal
````

Ensuite, on récupère l’index de l’interface du commutateur virtuel.
Pour trouver l’index d’interface, on utilise Get-NetAdapter.

```Powershell
PS > Get-NetAdapter
````

Le commutateur interne a un nom semblable à vEthernet (VNATSwitch) et la description d’interface Hyper-V Virtual Ethernet Adapter.

Il faut noter son ifIndex pour pouvoir configurer l'addresse IP la passerelle NAT à l’aide de New-NetIPAddress :

```Powershell
PS > New-NetIPAddress -IPAddress 192.168.100.1 -PrefixLength 24 -InterfaceIndex {{ifIndex}}
````

Pour configurer la passerelle, il faut des informations réseau basique.
La VM sera dans un réseau privé, l’adresse IP de la passerelle est `192.168.100.1` par exemple.
Ensuite on lance une commande comme celle-ci pour créer la passerelle NAT :

```Powershell
PS > New-NetNat -Name KDF -InternalIPInterfaceAddressPrefix 192.168.100.0/24
````

### IV.Creation de la machine virtuelle

On ouvrant une cmd avec élévation de privilège et vous saisissez : 
```cmd
cmd > virtmgmt.msc
````
Et cela ouvre la console du Gestionnaire Hyper-V, 
- Cliquez sur Paramètres Hyper-V
- Définissez le chemin des répertoire utilisés pour votre VM,car par défaut, tout est sur C:
- Faites votre choix dans Disques durs virtuels, Ordinateurs virtuels. On termine en cliquant OK
- Puis on clique sur « Nouveau » puis « Ordinateur Virtuel »
- Cliquez sur suivant à l'arrivé sur l'écran Assistant Nouvel ordinateur virtuel pour commencer
- On spécifie le nom et l’emplacement de la VM
- On spécifie de quelle génération sera la VM. Sur un PC récent avec  Windows 10 64bit en host et une Debian 64. 2eme génération est le bon choix.
- On choisi la taille mémoire
- On configure le réseau et c’est la que l’on choisit le NAT que l’on a configuré précédemment.
- On crée le disque virtuel et son emplacement
- On choisi le média avec lequel on installera la Debian
- A la fin, il y a un résumé de ce que l’on souhaite, avant de faire un click définitif
- Vous cliquez sur Terminer. Vous connectez la VM et vous procédez à l’installation.


## B. Installation de docker sur Debian

Installation des paquets nécéssaires pour docker

```bash
$ sudo apt update
$ sudo apt install apt-transport-https ca-certificates curl software-properties-common gnupg2
````

Import des repositories docker en utilisant curl

```bash
$ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
````

Ajout de paquets APT docker dans la liste de repository de debian

```bash
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
````

Mise à jour de la liste des paquets APT et installation de la derniere version de docker CE

```bash
$ sudo apt update
$ sudo apt install docker-ce
````

Verify the newly installed docker
```bash
$ sudo systemctl status docker
$ docker -v
````

## C. Installation de snap sur debian pour installer miniK8s

Installer snapd depuis l'outil apt

```bash
$ sudo apt update
$ sudo apt install snapd
````

Après cela, il d'agit d'installer le core snap via la commande:

```bash
$ snap install core
core 16-2.45.2 from Canonical✓ installed
````

## D. Installation de miniK8s

### I. Installation et activation des services
Installer minik8s depuis snap
```bash
$ sudo snap install microk8s --classic
````

Verifier le status de l'installation
```bash
$ microk8s status --wait-ready
````

Activer les services kubernetes utiles
```bash
$ microk8s enable dashboard dns registry istio
````

Essayer `microk8s enable --help` pour la liste des services disponibles
Pour désactiver un service, entrer `microk8s disable <name>`

### II.Utilisation et demarrage

Pour utiliser minik8s avec kubectl, executer les commandes:
```bash
$ microk8s kubectl get all --all-namespaces
````

Pour demarrer ou arreter le service kubernetes de minik8s
```bash
$ microk8s start
$ microk8s stop
````

### III.Accès à la portail d'administration web

```bash
$ microk8s dashboard-proxy
````

## E.Installation de Helm
Installer Helm depuis snap
```bash
$ sudo snap install helm --classic
````

ou depuis apt
```bash
$ curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
$ sudo apt-get install apt-transport-https --yes
$ echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
$ sudo apt-get update
$ sudo apt-get install helm
````

