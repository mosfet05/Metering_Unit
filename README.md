# Metering_Unit
Centrale de mesure

Voici une centrale de mesure avec :
* 18 mesures d'énergie électrique
* 1 interface TIC pour le compteur électrique
* 3 entrées de comptage tout ou rien
* 2 voies multifonctions (puisqu'il me restait 2 GPIO de libre)
* Un afficheur avec son codeur pour l'affichage des données
* Une interface Ethernet

Ce projet est terminé et fonctionnel.

**Alimentation**

Le montage a besoin de 3 tensions pour fonctionner :
* 3.3v : cette tension est produite par un régulateur à découpage. C'est la tension principale, utilisée par presque tous les composants du montage.
* 5v : cette tension est aussi produite par un régulateur à découpage. Elle est utilisée pour alimenter la carte contenant le microcontrôleur ESP32 et l'afficheur.
* 12v : cette tension est mise à disposition pour l'alimentation des capteurs de comptage (type capteur inductif).

**Mesure d'énergies électrique**

Cette partie repose sur des circuits Atmel ATM90E32. Ces circuits sont prévus pour mesurer l'énergie sur 3 phases. Les mesures de courant s'effectuent avec des transformateurs de courant. Le montage prévoit la possibilité d'utiliser des TC avec sortie courant (les plus simples) ou avec sortie tension. L'acquisition des 3 tensions se fait au travers de transformateur(s), pour garantir l'isolation galvanique par rapport au secteur. Si l'appareil est utilisé sur une installation monophasée, les 3 entrées tension sont à ponter entre elles. Si l'installation est en triphasé, il faut prévoir 3 transformateurs. L'accès aux données des circuits de mesure se fait via un bus SPI.

Cet partie du schéma est issue des schémas de CircuitSetup, je n'ai rien inventé.

**Interface TIC**

Cette interface permet de récupérer les données du compteur électrique via l'interface TIC. Là encore, je n'ai rien inventé. Tous les détails sont disponibles ici : https://miniprojets.net/index.php/2019/06/28/recuperer-les-donnees-de-son-compteur-linky/
A noter : c'est la deuxième fois que j'utilise ce schéma et c'est la deuxième fois que j'ai du remplacer la résistance de 10k placée après l'optocoupleur par une 100k. Sans cette modification le transistor ne commute pas car le signal n'a pas assez d'amplitude.

**Interface de comptage tout ou rien**

Il y a 3 entrées de comptage tout ou rien. Ce montage permet d'utiliser aussi bien un capteur à contact sec qu'une sortie à collecteur ouvert. Il est possible d'utiliser une tension comprise entre 3.3V et 12V.
J'ai utilisé un simple transistor, ce qui fonctionne très bien, mais il aurait aussi été possible d'utiliser une bascule de Schmitt.
Pour chaque voie, il est possible d'activer une résistance de pull-up ou de pull-down avec un cavalier. La valeur est volontairement faible pour permettre l'usage d'un capteur inductif en mode deux fils (ce qui est mon cas pour le compteur d'eau). Elle est même trop faible pour la sortie impulsion du Gazpar. Dans ce cas, il faut utiliser une résistance de tirage externe (10k pour le Gazpar).

**Entrées multifonction**

Il s'agit de deux entrées/sorties du microcontrôleur mises directement à disposition (avec une résistance de pull-up et une protection anti-esd). Ces voies ne supportent que du 3.3v.

Je me sers de la première voie pour un capteur de température et d'humidité DTH-22 et de la seconde pour quelques sondes de température DS18B20.

**Interface Ethernet**

L'interface Ethernet est assurée par un circuit Wiznet W5500. J'ai préféré une solution sur bus SPI
plutôt qu'avec une interface RMII en raison de la faible consommation de GPIO.
Cette solution nécessite tout de même un bus SPI dédié.
A noter que le serveur Web ne semble pas fonctionner correctement. Je pense que c'est dû à la "lenteur" du bus SPI vu le volume important d'informations à échanger. La communication avec Home Assistant ne pose aucun problème, de même pour les mises à jour en OTA.

**Afficheur**

La partie afficheur se base sur un ST7920 accompagné d'un codeur.
L’afficheur fonctionne en 5V ce qui nécessite une adaptation de niveau pour le faire dialoguer avec le microcontrôleur (toujours en SPI).
Le codeur comprend un bouton poussoir. Les deux voies du codeurs et le bouton poussoir sont filtrées avant d'être transmises au microcontrôleur.
A noter, j'ai de léger artefacts sur l'afficheur. Je pense que c'est causé par la bibliothèque, car, dans la même configuration, je n'ai aucun problème avec un ATMEGA328P et la bibliothèque U8g2.

**Microcontrôleur**

Rien de spécial à dire sur cette partie : j'utilise un ESP32-S2FH4 de chez Waweshare.

A noter, il est possible d'utiliser un ESP32 ou un RP2040 sans modification du circuit imprimé.

J'ai été confronté à une subtilité : l'ESP32 ne peut pas gérer plus de 6 périphériques SPI par bus SPI. Si on fait les comptes, j'ai 7 périphériques SPI (l'interface Ethernet ne compte pas car elle dispose de son propre bus SPI). J'ai du utiliser "l'émulateur" (software) pour contourner cette limite. Cela peut poser problème, notamment au niveau de l'afficheur (si l'afficheur utilisé est plus gourmand en ressource, cela peut causer des ralentissements). Dans mon cas, je ne vois pas de problème.
