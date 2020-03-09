# TP 3 - Gestion des paquets

## Exercice 1. Commandes de base

Commencez par mettre à jour votre système avec les commandes vues dans le cours.

```
sudo apt update
sudo apt upgrade
```

**Donnez les commandes répondant aux questions suivantes :**

**1.Quels sont les 5 derniers paquets installés sur votre machine?**
```
ls -ltr /var/cache/apt/archives
```

**2.Utiliser dpkg et apt pour compter le nombre de paquets installés (ne pas hésiter à consulter le manuel!).Comment explique-t-on la (petite) différence de comptage?**
```
dpkg -l | wc -l
```
donne 559
```
sudo apt list --installed | wc -l
```
donne 555

La différence vient des fichiers de configuration de paquets désinstallés qui restent dans les dossiers et qui sont pris en compte par dpkg et pas par apt. Les paquets laissent des fichiers de configuration s'ils sont simplement remove et non purge.

**3.Combien de paquets sont disponibles en téléchargement?**
```
apt list --all-versions | wc -l
```
donne 124 907

**4.Créer un alias “maj” qui met à jour le système**

Dans ~/.bashrc rajouter
```
alias maj ='sudo apt update && sudo apt upgrade && sudo apt full-upgrade'
```
puis mettre à jour le fichier bashrc avec source .bashrc

**5.A quoi sert le paquet fortunes? Installez-le.**
 fortunes est un paquet qui permet d'afficher des citations sur une console.
 On l'installe avec : ```sudo apt install fortunes```

**6.Quels paquets proposent de jouer au sudoku?**

On installe le paquet gnome-sudoku

**7.Lister les derniers paquets installés explicitement avec la commande apt install**
```
grep "install" /var/log/dpkg.log | tail -5
```
## Exercice 2.

**A partir de quel paquet est installée la commande ls? Comment obtenir cette information en une seule commande, pour n’importe quel programme (indice : la réponse est dans le poly de cours 2, dans la liste des commandes utiles)? Utilisez la réponse à pour écrire un script appelé origine-commande(sans l’extension.sh) prenant en argument le nom d’une commande, et indiquant quel paquet l’a installée.**

ls est installé à partir du package coreutils. On obtient ce résulat grâce à : ```dpkg -S $(which -a ls)```

On crée un script origine-commande qui renvoit le package dont provient la commande rentrée par l'utilisateur :

```
#!/bin/bash

read -p 'Saisissez commande : ' cmd
package=sudo dpkg -S $(which -a $cmd)
echo $package
```


## Exercice 3.

**Ecrire une commande qui affiche “INSTALLÉ” ou “NON INSTALLÉ” selon le nom et le statut du package spécifié dans cette commande.**

```
#!/bin/bash

read -p 'Saisissez package: ' pck

if [ -z "$(apt list --installed | cut -d/ -f1 | grep ^$pck$)" ];then
	echo  "non installé"
else
	echo "installé"
fi
```

## Exercice 4.

**Lister les programmes livrés avec coreutils. A quoi sert la commande ’[’ et comment aﬀicher ce qu’elle retourne?**
```
apt show coreutils | tail -9
```

## Exercice 5. aptitude

**Installez le paquet emacs à l’aide de la version graphique d’aptitude.**
```
sudo aptitude
/emacs (3e occurence)
g
gg
```

## Exercice 6. Installation d’un paquet par PPA

Certains logiciels ne figurent pas dans les dépôts officiels. C’est le cas par exemple de la version ”officielle” de Java depuis qu’elle est développée par Oracle. Dans ces cas, on peut parfois se tourner vers un ”dépôt personnel” ou PPA.

**1.Installer la version Oracle de Java (avec l’ajout des PPA)**
```
sudo add-apt-repository ppa:linuxuprising/java
sudo apt update
sudo apt install oracle-java13-installer-local
```

**2.Vérifiez qu’un nouveau fichier a été créé dans /etc/apt/sources.list.d. Que contient-il?**
Le fichier contient des lignes APT qui contiennent des informations sur les dépots de paquets de notre linux.

## Exercice 7. Création de dépôt personnalisé

Dans cet exercice, vous allez créer vos propres paquets et dépôts, ce qui vous permettra de gérer les
programmes que vous écrivez comme s’ils provenaient de dépôts officiels.

### Création d’un paquet Debian avec dpkg-deb

**1. Dans le dossier scripts créé lors du TP 2, créez un sous-dossier origine-commande où vous créerez un
sous-dossier DEBIAN, ainsi que l’arborescence usr/local/bin où vous placerez le script écrit à l’exercice
2**


**2. Dans le dossier DEBIAN, créez un fichier control avec les champs suivants :**  
Package: origine-commande #nom du paquet  
Version: 0.1 #numéro de version  
Maintainer: Foo Bar #votre nom  
Architecture: all #les architectures cibles de notre paquet (i386, amd64...)  
Description: Cherche l'origine d'une commande  
Section: utils #notre programme est un utilitaire  
Priority: optional #ce n'est pas un paquet indispendable    

**3. Revenez dans le dossier parent de origine-commande (normalement, c’est votre $HOME) et tapez la
commande suivante pour construire le paquet :**  
`dpkg-deb --build origine-commande`  
Félicitations ! Vous avez créé votre propre paquet !  

### Création du dépôt personnel avec reprepro  
**1. Dans votre dossier personnel, commencez par créer un dossier repo-cpe. Ce sera la racine de votre
dépôt**  

**2. Ajoutez-y deux sous-dossiers : conf (qui contiendra la configuration du dépôt) et packages (qui contiendra
nos paquets)**  

**3. Dans conf, créez le fichier distributions suivant :**  
Origin: Un nom, une URL, ou tout texte expliquant la provenance du dépôt  
Label: Nom du dépôt  
// Suite: stable  
Codename: cosmic #le nom de la distribution cible  
Architectures: i386 amd64 #(architectures cibles)   
Components: universe #(correspond à notre cas)  
Description: Une description du dépôt  
**4. Dans le dossier repo-cpe, générez l’arborescence du dépôt avec la commande**  
`reprepro -b . export`  

**5. Copiez le paquet origine-commande.deb créé précédemment dans le dossier packages du dépôt, puis,
à la racine du dépôt, exécutez la commande**  
`reprepro -b . includedeb eoan origine-commande.deb`  
afin que votre paquet soit inscrit dans le dépôt.  

**6. Il faut à présent indiquer à apt qu’il existe un nouveau dépôt dans lequel il peut trouver des logiciels.
Pour cela, créez (avec sudo) dans le dossier /etc/apt/sources.list.d le fichier repo-cpe.list
contenant :**  

`deb file:/home/marialice/repo-cpe eoan multiverse  `
(cette ligne reprend la configuration du dépôt, elle est à adapter au besoin)  

**7. Lancez la commande sudo apt update.**  

Féliciations ! Votre dépôt est désormais pris en compte ! ... Enfin, pas tout à fait... Si vous regardez
la sortie d’apt update, il est précidé que le dépôt ne peut être pris en compte car il n’est pas signé.
La signature permet de vérifier qu’un paquet provient bien du bon dépôt. On doit donc signer notre
dépôt.  

### Signature du dépôt avec GPG  
GPG est la version GNU du protocole PGP (Pretty Good Privacy), qui permet d’échanger des données de
manière sécurisée. Ce système repose sur la notion de clés de chiffrement asymétriques (une clé publique et
une clé privée)  
**1. Commencez par créer une nouvelle paire de clés avec la commande**  
`gpg --gen-key`  
Attention ! N’oubliez pas votre passphrase !!! : tp 

**2. Ajoutez à la configuration du dépôt (fichier distributions la ligne suivante :**  
`SignWith: yes`  

**3. Ajoutez la clé à votre dépôt :**  
`reprepro --ask-passphrase -b . export`  

Attention ! Cette méthode n’est pas sécurisée et obsolète ; dans un contexte professionnel, on utiliserait
plutot un gpg-agent.  

**4. Ajoutez votre clé publique à votre dépôt avec la commande**  
`gpg --export -a "marialice" > public.key`  

**5. Enfin, ajoutez cette clé à la liste des clés fiables connues de apt :**  
`sudo apt-key add public.key`  

Félicitations ! La configuration est (enfin) terminée ! Vérifiez que vous pouvez installer votre paquet comme
n’importe quel autre paquet. 



## Exercice 8. Installation d’un logiciel à partir du code source

Lorsqu’un logiciel n’est disponible ni dans les dépôts oﬀiciels, ni dans un PPA, ou encore parce qu’onsouhaite n’installer qu’une partie de ses fonctionnalités, on peut se tourner vers la compilation du codesource.

Malheureusement, cette installation ”à la main” fait qu’on ne propose pas des bénéfices de la gestion de paquets apportée par dpkgouapt. Heureusement, il est possible de transformer un logiciel installé ”à la main” en un paquet, et de le gérer ensuite avec apt; c’est ce que permet par exemple checkinstall.

**1.Commencez par cloner le dépôt git suivant :**
`git clone https://github.com/jubalh/nudoku`  

Ceci permet de récupérer en local le code source du logiciel nudoku.

**2.Rendez vous dans le dossier nudoku qui vient d’être créé et lancez la commande `autoreconf -i`  (ainsi que spécifié dans le fichier README.md).
A vous d’installer les éventuels paquets manquants (un peu d’aide : pour résoudre le problème de la macroAM_GNU_GETTEXT manquante, installez le paquet gettext).Relancez la commande `autoreconf -i` après chaque paquet installé jusqu’à ce qu’elle se termine sans erreur.**

On a installé gettext et autopoint. Néanmoins, la version de gettext est trop anciennne pour fonctionner avec autopoint. Il nous faudrait au moins la version 0.20.0, et nous avons une version 0.19. Les mises à jour de gettext ne donnent rien.
