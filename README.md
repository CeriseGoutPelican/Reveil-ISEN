# Showcase : réveil ISEN
## Introduction & concept
Imaginez, en un seul objet, un serveur musicale Spotify totalement indépendant, un réveil, une horloge, une station météo et un afficheur vous permettant d'afficher le contenu de n'importe quel contenu d'un packet [Constellation](https://developer.myconstellation.io/) !

C'est ce qu'est le réveil ISEN : une inteface simple et intuitive au design 8-bits où vous pouvez afficher toutes les informations nécessaires à votre quotidien et à votre réseau domotique, sans avoir à passer par un ordinateur ou à un smartphone. Le concept, pour un prix abordable, a été conçu pour être le plus modulable possible et, même avec des connaissances basiques en programmation et en électronique, vous devriez pouvoir facilement l'adapter parfaitement à vos besoins.

![Equipe du projet](http://www.shiningparadox.fr/wp-content/uploads/2017/07/IMG_4408.jpg)
![Premier prototype](http://www.shiningparadox.fr/wp-content/uploads/2017/07/1.jpg)
![Interface Twitter](http://www.shiningparadox.fr/wp-content/uploads/2017/07/4.jpg)

## Coût des composants 
L'ensemble des composants porte le projet a près de `105€` au total, une somme non négligeable principalement dû à la présence d'un Raspberry Pi 3 et de la matrice LED. 

| Composant             | Prix unitaire | Quantité | Référence |
|-----------------------|--------------:|---------:|-----------|
| Raspberry Pi 3 (Wifi) | 38€           | 1        | [Amazon](https://www.amazon.fr/Raspberry-Pi-Carte-Mère-Model/dp/B01CD5VC92/ ) |
| Mini haut-parleur     | 4.7€          | 2        | [Gotronic](https://www.gotronic.fr/art-haut-parleur-hp8r3w-25596.htm) |
| Matrice LED RGB 8x32  | 42.88€        | 1        | [Amazon](https://www.amazon.fr/gp/product/B01DC0IPVU/) |
| Boite                 | de 10€ à 50€  | 1        | Voir "Conception de la boite" |
| Electronic divers     | 5€            | -        | Câbles / boutons / soudures |

Deux optimisations sont possibles afin de faire baisser le prix global du projet :
 1. Utiliser un [Raspberry Pi Zero W](https://www.kubii.fr/fr/pi-zero-w/1851-raspberry-pi-zero-w-3272496006997.html) (`11€`) 
 2. Remplacer la matrice LED RGB 8x32 par une matrice LED RGB 8x8 couplée avec une matrice blanche (prix estimé aux alentours de la vingtaine d'euros)

Le projet complété pourrait alors tourner aux alentours de `60€` environ.

## Structure du projet 
Le premier jet du projet s'est structuré sur six paquets que l'on a voulu indépendant du réveil et d'un paquet jouant le rôle de "cerveau" sur le projet.

 - Réveil ISEN (CERVEAU) : [voir le Github](https://github.com/)
 - Serveur Spotify : [voir le Github](https://github.com/nicolasroi/Constellation-Spotify)
 - Package Twitter pour Constellation : [voir le Github](https://github.com/CeriseGoutPelican/Package-Twitter-pour-Constellation)
 - Package bouton GPIO pour Raspberry Pi pour Constellation : [voir le Github](https://github.com/CeriseGoutPelican/Package-boutons-GPIO-pour-Constellation)
 - Package affichage RGB LED (8x32) pour Constellation : [voir le Github](https://github.com/CeriseGoutPelican/Package-Affichage-LED-pour-Constellation)
 - Package Alarme pour Constellation : [voir le Github](https://github.com/)
 - Package Forecast IO pour Constellation : [voir le Github](https://github.com/myconstellation/constellation-packages/tree/master/ForecastIO)

Chacun des gitHub sont documentés et expliqués en détails ! N'hésitez pas à vous y rendre pour en savoir plus dans leur fonctionnement technique, en particulier pour le package REVEIL ISEN.

Le réveil ISEN se base sur trois packages principaux : 
 1. **Bouton RPi :** représente l'interface physque du réveil et "écoute" les cinq boutons physiques (voir la partie Fonctionnement)
 2. **REVEIL ISEN :** représente l'intelligenge du réveil. Il écoute les `StateObjects` des autres packages et pousses des `messages Callback` aux autres packets pour leur bon fonctionnement.
 3. **AFFICHAGE LED :** représente l'interface graphique du réveil. Il n'a pas d'intelligence et ne fait qu'afficher ce que le packet REVEIL ISEN lui envoie.

Voici l'organisation générale des septs packages étudié lors du démarage du réveil le matin par exemple :

![Organisation des packages](http://www.shiningparadox.fr/wp-content/uploads/2017/06/Organisation.png)

1. **ETAPE 1 :** Le package ALARM passe une variable `isRinging = True` via un `stateObject` écouté par le package REVEIL ISEN :
```json
{
  "clockName": "Test",
  "wakingHour": 15,
  "wakingMinute": 41,
  "wakingDays": null,
  "isRinging": true,
  "snooze": false,
  "snoozedTime": 0
}
```
2. **ETAPE 2 :** Le package REVEIL ISEN, après avoir reçut le `stateObject` démare une playlist Spotify en envoyant un `message Callback`au package SPOTIFY et met à jour l'affichage de la matrice LED en envoyant un autre `message Callback` au package AFFICHAGE LED. Ces deux messages sont écrits de la manière suivante :
```python
Constellation.SendMessage(MUSIC_PACKAGE, "MusicControl", {"instruction":"PLAY_PLAYLIST", "uri":MORNING_URI})
Constellation.SendMessage(DISPLAY_PACKAGE, "DisplayContent", {"icon":"musique", "text":alarmName,"time":None,"matrix":None})
```
3. **ETAPE 3 :** L'utilisateur appuie sur un bouton physique dont la position est écouté par le package BOUTONS RPi et envoie un `message Callback` au package REVEIL ISEN qui va alors envoyer un autre `message Callback` au package ALARME pour lui spécifier la fin de l'alarme. 
```python
Constellation.SendMessage(MUSIC_PACKAGE, "MusicControl", {"instruction":"PLAY_PAUSE"})
Constellation.SendMessage(ALARM_PACKAGE, "StopRinging", alarmName)
```

## Fonctionnement

Le réveil possède cinq boutons permettant d'utiliser le réveil sans aucune interface graphique autre que la matrice d'affichage. Le tableau suivant rassemble les différentes actions possibles :

|             | Bouton Gauche        |--|  Bouton Centre Gauche  | Bouton Centre | Bouton Centre Droit |--| Bouton Droite      |
|-------------|----------------------|--|------------------------|---------------|---------------------|--|--------------------| 
| Clic simple | Interface précédente |  | Volume moins           | Play / pause  | Volume plus         |  | Interface suivante |
| Double clic | Interface précédente |  | Volume moins           | Play / pause  | Volume plus         |  | Interface suivante |
| Clic long   | Snooze               |  | Musique précédente     | Accueil       | Musique suivante    |  | Snooze             |

Ces boutons ont été dans un premier temps placé directement sur carte de prototypage. L'idée serait de les placer, avec le même type d'espacement sur la boite, au dessus de la matrice LED.

![Carte de prototypage](http://www.shiningparadox.fr/wp-content/uploads/2017/07/2.jpg)

### Ajouter une interface
L'ajout d'une interface doit pour le moment passer par l'édition du package REVEIL ISEN. Habituellement l'ajout d'une nouvelle interface se fait en trois étapes :
1. Créer une nouvelle fonction pour l'affichage : `displayMonInterface()` contenant les informations requise :
```python
def display():
    Constellation.SendMessage(DISPLAY_PACKAGE, "DisplayContent", {"icon":"monIcone", "text":"monTexte","time":None,"matrix":None}) 
```
2. Ajouter la fonction à la liste des affichages à l'emplacement souhaité :
```python
displays = [displayHome, displayWeather, displayTwitter, displayMusic, displayMonInterface]
```
3. Enfin il faut mettre à jour votre affichage en écoutant par exemple un `stateObject`:
```python
@Constellation.StateObjectLink(package = "monPackage", name = "monSOName")
def monPackageUpdated(stateobject):
    # Mise à jour de la variable globale contenant votre contenu
    global monPackageSO
    monPackageSO = stateobject.Value
    
    # Update en live si l'interface est correcte :
    if(displays[currentDisplay] == displayMonInterface):
        toDisplay(currentDisplay)
```

### Ajouter un réveil
Pour ajouter un réveil, deux possibilités sont offertes : envoyer un `message Callback` depuis la console Constellation ou alors utiliser l'interface web dédiée :

...

## Conception de la boite
Lors de la création de la boite, deux possibilités se sont présentées : l'impression 3D et la  découpe laser. Si l'impression 3D permet plus de liberté et plus de précision dans les volumes et le squellette, celle-ci est terriblement chère pour des structures supérieures à une vingtaine de centimètres (la matrice LED fait une trentaine de centimètres). Le prix de la boite 3D sur sculpteo s'élevait à `145€` et pour une impression "maison" autour d'une cinquentaine d'euros.

![Impression 3D](http://www.shiningparadox.fr/wp-content/uploads/2017/06/Impression-3D.jpg)

Nous avons donc décidé de partir sur une découpe laser, peu chère car elle ne demande que les prix des matériaux (bois, PVC, etc.) et peut se faire pour moins de dix euros et en seulement quelques minutes.

![Découpe laser : machine](http://www.shiningparadox.fr/wp-content/uploads/2017/07/IMG_20170629_114939.jpg)
![Découpe laser : rendu final](http://www.shiningparadox.fr/wp-content/uploads/2017/07/IMG_4428.jpg)

## Les ouvertures possibles
Au dela de ce premier jet, un énorme travaille sera encore nécessaire pour rendre le réveil accessible et agréable à utiliser. Voici les futures fonctionnalités prévues pour les futures mises à jour :
- [ ] Ajouter deux haut-parleurs sur les côtés (les encoches ont déjà été prévues pour dans la boite)
- [ ] Pouvoir ajouter des interfaces directement par interface graphique depuis un dashboard web ou une application mobile
- [ ] Améliorer le design et la propreté du concept en général
- [ ] Optimisation logicielle pour plus de fluidité
- [ ] Augmenter le nombre d'applications compatibles avec l'affichage, et ce de manière native
