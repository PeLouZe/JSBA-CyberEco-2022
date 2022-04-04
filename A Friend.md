# JSBA-CyberEco-2022
Un write-up pour le Challenge 'A Friend' du CTF d'Avril 2022 (CTF débutant).

## A friend - Description générale :
Défi catégorie d'investigation
Outils utilisés : Hex Editor, BinWalk, CyberChef.

### Pourquoi c'est intéressant ?
Ce qui est intéressant tout d'abord, c'est que le fichier PNG est dans le code du fichier .zip, mais n'est pas actuellement compressé par celui-ci.

Aussi, plusieurs malwares peuvent utiliser ce type de 'cachette' pour passer leurs fichiers, ça se fait aussi dans des fichiers images (voir stéganographie).
Dans ce cas-ci, l'image est cachée dans un .zip. Mais un fichier malicieux pourrait se cacher dans n'importe quel autre type de fichier de la même façon.

Voir : https://www.sentinelone.com/blog/hiding-code-inside-images-malware-steganography/

MITRE Technique : T1027.003, Defense Evasion : https://attack.mitre.org/techniques/T1027/003/

Oui, oui, ça se peut dans la vraie vie!

## Contexte
Le défi présenté est le suivant (je n'ai plus accès au texte exacte) : "Notre ami a commencé a se comporter de façon étrange récemment, pouvez-vous l'aider ?"
Ils nous fournissent aussi un fichier Zip "A friend.zip".

## Let's go

Démystifions ce défi ensemble et tentons d'aider notre ami.
Je tente de d'extraire le fichier et la première chose que je note, c'est que l'extracteur d'archive natif de Windows n'est pas capable d'extraire l'archive (je me pose déja des questions).
C'est pas grave, on a qu'à l'ouvrir avec 7-zip :
![a frienddocx](https://user-images.githubusercontent.com/16509773/161454808-c3b95a19-e2d8-4f97-b61e-3912daff99af.jpg)

On peut ensuite ouvrir le DOCX extrait : 
![word](https://user-images.githubusercontent.com/16509773/161454943-5c069d2c-64d2-4333-8e4e-10be53d3c7fc.jpg)

Ce fichier n'a rien de plus banal. 

Cependant, pourquoi ils auraient placé un fichier Word, sans flag ? Investiguons!


Le fichier semble plutôt court. Dans ce type de scénario je tente toujours de tout sélectionner pour voir si il n'y aurait pas quelque chose de caché.
CTRL+A :
![selectall](https://user-images.githubusercontent.com/16509773/161455009-b9e0c46b-c4b4-4f69-8ac0-481391505ab4.jpg)

## Que vois-je, Que vois-je ? C'est un bâteau ? c'est un avion ? 

## Mon flag est-il sur une le déserte ?🚩

Comme vous l'aurez vue, un petit texte est caché sur la 3ième ligne, qui a pu être retrouvé en sélectionnant tout.


Allons-voir si c'est notre Flag, en changeant la couleur du texte...


![noflag](https://user-images.githubusercontent.com/16509773/161455109-47385f62-08fc-4b36-8b75-1b4a2d37b30a.jpg)

Hmmm... il semblerait que ce ne soit pas notre FLAG (pas de 100 points ici), mais on obtient un indice 'I hid my location', on sait donc que sa localisation devrait être dans le fichier quelquepart.

## Si la vie était comme ça, je n'aurais pas besoin d'une carte Visa
Évidemment, ça aurait été trop facile.

À ce moment précis, j'ai bloqué un peu, j'ai tenté plusieurs tentatives sur le fichier Word, voici ce que j'ai fait précisément, sans succès :

-Le passer dans Binwalk pour extraire son contenu (son contenu est extrait, des fichiers .xml etc..., mais pas de Flag en vue).

-Changer toutes les couleurs, l'ouvrir dans CyberChef pour y vérifier le code... rien d'anormal.



Le temps file, je dois trouver la localisation de mon ami!!!

## Et Windows qui n'ouvre pas le Zip lui ?
Après pas mal de recherche, j'en suis arrivé à mettre le .zip original dans CyberChef. À ce moment là, je me suis rendu compte que ce fameux fichier .zip était très volumineux pour ne contenir qu'un fichier Word.

Observations :

-Le fichier .zip fait une taille de 18Mo.

-Le fichier .docx fait une taille de 24Ko.


Pourtant, la fonction compression devrait réduire la taille d'un fichier. 
(oui oui, les puristes diront qu'un word c'est déjà compressé et qu'on ne peut le compresser d'avantage, mais à ce moment on aurait une petite différence de taille, pas plusieurs Mo.)

## Dig'in'Deeper'in'the'Zip
À ce moment, comme CyberChef ne fonctionnait pas (fichier trop volumineux), j'ai tenté de tout extraire avec Binwalk.

Encore une fois, l'extraction fonctionne, mais rien d'intéressant, pas de flag.

J'ai ensuite envoyé le tout dans un éditeur Hex en ligne.

J'en arrive à la conclusion que le fichier est effectivement volumineux.

![hexonline](https://user-images.githubusercontent.com/16509773/161455549-503aac24-0e2a-4d9f-addf-dfe533aeb6eb.jpg)

Lorsque je tombe sur ce genre de défi, j'aime toujours vérifier les signatures de fichiers (début et fin), les fameux Magic Bytes.


Aussi, je regarde toujours si l'entropie change drastiquement quelque part dans le fichier (un fichier compressé doit toujours s'approcher de code au hasard, mais si 
au milieu de celui-ci ce n'est plus du hasard, on peut se poser des questions).

Ici, à première vue, tout semble être compressé.

## Magic Bytes to the rescue of our Friend
Au début du fichier, on remarque que la signature est bonne. Mais il faut savoir qu'une archive ZIP commence et doit se terminer par les caractères ''PK''.

À la fin du fichier je remarque que ce n'est pas le cas!!!

![iendb](https://user-images.githubusercontent.com/16509773/161455727-cfb77fe5-5cc5-4505-959c-04cf56ffcc1f.jpg)

À quoi correspond cette fameuse signature de fin IEND ? ....... Un fichier .PNG, évidemment!!!

Tentons de trouver les bytes magiques, pour le début d'un fichier .PNG :

https://en.wikipedia.org/wiki/List_of_file_signatures

On cherche donc les bits suivants : '89 50 4E 47 0D 0A 1A 0A'

Bingo!

![magicbytes](https://user-images.githubusercontent.com/16509773/161455937-e4711519-0c93-4cbe-8524-83fc62b22eae.jpg)

Qu'est-ce qu'on peut bien faire avec le début et la fin d'un fichier ?

Générer le fichier!!!

## On enlève la junk et on trouve le FLAG
Il ne nous reste plus qu'à utiliser l'éditeur Hex, pour enlever toute la partie du .zip et garder seulement la partie de l'Image .PNG :

![selected](https://user-images.githubusercontent.com/16509773/161456160-21ee4cdb-95eb-47fd-b783-4be10a72cbaf.jpg)

Si votre sélection est bonne, vous devriez avoir enlevé environ 12 000 bytes au fichier.

Ceci vous permettra de garder la signature de début du fichier .PNG et la signature de fin, pour générer ce fichier.

Il ne reste plus qu'à l'enregistrer au format .PNG.

![friend](https://user-images.githubusercontent.com/16509773/161456444-3931228d-8089-4d11-9661-19d112e8ba6f.png)





Pauvres petits ils sont pris au Pet shop...







### Ce que vous ne voulez vraiment pas savoir.

Voici quelques unes des chansons qui m'ont motivés pendant ce défi:

https://www.youtube.com/watch?v=mP0TVptLDQE

https://www.youtube.com/watch?v=XhepwtKR5lg







Merci pour le CTF :)
