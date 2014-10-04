ReaDiC
=======

Read Discursive Corpus : code to read and build feature structures for pdtb like corpora.


# Description

Le programme récupère les instances de relations voulues dans le corpus et produit en sortie des fichiers de traits.



# Code : *read_pdtb_like.sh*

Script shell partant du corpus format PDTB pour écrire des fichiers texte contenant tous les exemples et d'autres informations.

**Pour l'instant, problème avec la récuperation des arguments dans le script shell quand ils sont ensuite passés en option si on a défini une valeur par défaut. Donc appel à readCorpus/py/readCorpus.py commenté dans le script shell : utiliser le script shell pour générer les fichiers en sortie des scripts Java puis lancer le script python.**

Les chemins vers les programmes utilisés sont en dur dans le script shell, **il faut modifier ces chemins**. (TODO : à modifier)

Le script est composé de 3 étapes, il lance les 3 scripts décrits ci-après. Les données en sortie sont celles à utiliser en entrée du script python *ReaDiC/py/readCorpus.py*. Pour l'utiliser, on donne toutes les options décrites pour les 3 codes suivants.

### ReaDiC/java/pdtbRead.jar : 

Lit le corpus PDTB et écrit en sortie des fichiers contenant tous les exemples (implicit, explicit, altlex, entrel et norel) associés à des informations issus de l'annotation (relations, arguments ...) et des informations utilisées pour calculer des traits par la suite (règles de production pour chaque argument, tête des arguments ...). L'id de l'exemple informe sur son type, cf plus bas. Se base sur l'API Java de Penn.

Require : pdtb.jar, version java ? 

##### Options (ou arguments dans l'ordre si fichier jar utilisé seul) : 
* **-l** LABELS : jeu de relations, fichier avec correspondance relation et label numérique (TODO : en fait inutile ici, à supprimer) ;
* **-e** HEADRULES : liste de règles de percolation des têtes, fichier head_rules.txt dans readCorpus/data/ressources/ ;
* **-r** RAW : répertoire des fichiers texte brut au format PDTB, ie : des sous-répertoires de type 00, 01, ... (TODO : check, mais il me semble qu'on ne peut pas avoir plus de 25 sous-répertoires), contenant des fichiers dont le nom a le format wsj_XXYY, où XX correspond au sous-répertoire et YY au fichier ;
* **-p** PTB : répertoire des fichiers parse au format PTB, idem RAW avec format fichier wsj_XXYY.mrg (correspondant au RAW wsj_XXYY) ;
* **-d** PDTB : répertoire des fichiers annotations niveau discursif, idem RAW avec fichier wsj_XXYY.pdtb (corr. au RAW wsj_XXYY et au PTB wsj_XXYY.mrg) ;
* **-i** JAVAOUTR : répertoire de sortie contenant les exemples extraits, idem organisé en sous-répertoires de type 00, 01 ... contenant des fichiers dont le nom a le format wsj_XXYY.read (corr. au RAW wsj_XXYY, PTB wsj_XXYY.mrg et PDTB wsj_XXYY.pdtb).

##### Format fichiers en sortie : TODO (très moche)


### ReaDiC/java/pdtbDiscourseReading.jar : 

Lit (à nouveau) le corpus PDTB et écrit en sortie des fichiers contenant les exemples explicites (en emploi discursif, exemple positif) et les exemples correspondant à des formes de connecteur en emploi non discursif (emploi non discursif, exemple négatif). L'id de l'exemple informe sur son type : si l'id contient _p_, exemple positif, si l'id contient _f_, exemple négatif. Se base sur l'API Java de Penn.
  
Require : pdtb.jar, version java ?

##### Options (ou arguments dans l'ordre si fichier jar utilisé seul) :
  * **-m** CONNECTIVES : liste de tous les connecteurs du PDTB, fichier connective_pdtb.txt dans readCorpus/data/ressources/ ;
  * **-r** RAW : idem pdtbRead.jar ;
  * **-p** PTB : idem pdtbRead.jar ;
  * **-d** PDTB : idem pdtbRead.jar ;
  * **-g** JAVAOUTD : idem pdtbRead.jar, avec des informations différentes et uniquement les exemples explicites et en emploi non discursif.

##### Format fichiers en sortie : TODO (très moche)

### ReaDiC/py/union_javaFiles.py : 

Lit les deux corpus précédemment construit pour aboutir à un seul corpus contenant toutes les informations (ie ajoute aux exemples explicites du premier corpus les informations présentes seulement dans le second et ajoute les exemples d'emploi non discursif).

Require : none

##### Options :
* **-i** JAVAOUTR : répertoire en sortie de la première étape ;
* **-g** JAVAOUTD : répertoire en sortie de la seconde étape ;
* **-o** JAVAOUT : répertoire de sortie contenant les fichiers toujours organisés en sous-répertoires et prtant toujours le même nom, mais avec les informations provenant des deux répertoires en entrée.

##### Format fichiers en sortie : TODO (très moche)


# Code : *ReaDiC/py/readCorpus.py* 

Lit les fichiers en sortie de l'étape précédente et écrit en sortie des fichiers de traits. Certaines options sont obligatoire (req) et d'autres ont une valeur par défaut (not req:default).

Environ 40s.

Require : nltk version ? python version ?

### Options :

* **-b** JAVAOUT (req) : le répertoire contenant les fichiers en sortie des 3 étapes précédentes (**sauf pour l'instant pour les instances de type discourse reading**, nécessite le répertoire en sortie de la seconde étape) ;
* **-o** OUT  (not req:~/readCorpus_out/) : répertoires où seront créés les répertoires et fichiers en sortie (décrits ci-dessous) ;
* **-l** LABELS (req) : le même fichier de relations que précédemment ;
* **-n** LEVEL (not req:-1) : le niveau d'annotation voulu au niveau de la hiérarchie de relations, soit on conserve tous les exemples avec les relations les plus profondes annotées (-1), soit les exemples annotés au niveau 1 (1), au niveau 2 (2) ou au niveau (3) ;
* **-d** CORPUS (not req:pdtb) : le type de corpus lu, normalement entre pdtb, bllip, annodis mais seul pdtb est implémenté (option inutile pour l'instant) ;
* **-t** TYPE (not req:implicit) : le type d'instance conservé, entre implicit, explicit, altlex, entrel, norel, discoursereading (en cours : position). On peut combiner ces types, par ex. implicit+explicit ;
* **-f** FEATS (not req:all) : le type de traits que l'on veut utiliser, par exemple bigram, prodrules, wang, park, pitler ou all (tous les traits, dépend du type d'instance conservé). La liste complète des traits disponibles est donné infra. Ces traits peuvent être combinés, par ex. bigram+prodrules ;
* **-s** SPLIT (not req:2-21:00#01#22#24:23) : le split train/dev/test à utiliser en termes de section, de la forme train:dev:test, avec train = XX-ZZ (de la section XX incluse à la section ZZ incluse), dev ou test = XX(#ZZ#TT) (la section XX et la section ZZ et la section TT). La valeur par défaut correspond au split de Lin09 (recommendation du manuel du PDTB), le split de Pitler ou Wang est obtenu avec 2-20:00#01#23#24:21#22 ;
* **-m** ANNOTATION (not req:1A) : certains exemples peuvent être annotés avec plusieurs relations, les implicites peuvent recevoir jusqu'à 4 relations (1A, 1B, 2A et 2B), les explicites jusqu'à 2 (1A et 1B). On peut donc choisir de ne conserver que la première relation annotée (1A) ou dupliquer les exemples (avec des relations différentes) pour chaque relation annotée. On conserve toujours les annotations supérieures (ie : si on choisit 1B, on conserve aussi 1A) ;
* **-a** FEATALPHA (not req:OUT/correspondance_featsnum2featname.txt) : fichier contenant l'alphabet de traits, ie tous les traits utilisés et leur correspondance numérique (pour format svmlight). Si aucun fichier n'est donné, le fichier est créé. Si un fichier est donné, il est utilisé pour trouver la correspondance numérique pour les traits calculés, et les nouveaux traits sont ajoutés à la fin du fichier ;
* **-c** CONNECTIVES (not req:None) : la liste/le lexique de connecteurs (non utilisé pour l'instant) ;
* **-r** RESSOURCES (not req:readCorpus/data/ressources/) : répertoire contenant des lexiques utilisés pour calculer certains traits (classes de Levin, lexique de subjectivité, Inquirer) ;
* **-x** MODE (not req:all) : en mode read, des fichiers au format colonne (décrits ci-dessous) sont produits en plus des fichiers de traits.

### Sortie :
* Les fichiers de traits comportent une instance par ligne, les instances ont le format : (ID) label trait1:val1 trait2:val2. Le format svmlight correspond à des noms de traits au format numérique (ie traitX -> 42), avec les traits ordonnés par ordre croissant pour chaque instance. L'ID d'un exemples est de la forme $$$CORPUS_type_section_file$numéro$$$ (par ex $$$PDTB_e_00_03$0$$$), avec type représentant le type d'instance :
  * **i** : implicit,
  * **e** : explicit,
  * **n** : norel (annoté norel dans le PDTB, différent de no discourse reading),
  * **t** : entrel,
  * **a** : altlex,
  * **f** : no discourse reading.
* un **répertoire feat_FFF/** (FFF = la valeur de l'option FEATS) contenant des fichiers au format wsj_XXYY.feat et wsj_XXYY.feat.svmlight (wsj_XXYY étant le fichier raw d'origine). Les fichiers wsj_XXYY.feat contiennent les traits avec leur valeur *nominal* (par ex. N#bigram=the+cat) tandis que les fichiers wsj_XXYY.feat.svmlight contiennent les traits avec leur correspondance numérique (selon $FEATALPHA). Dans ces fichiers, l'ID de l'exemple apparaît en début de ligne ;
* un **répertoire train_dev_test_FFF/** (FFF = la valeur de l'option FEATS) contenant les fichiers de train, dev et test selon le split $SPLIT choisi. Les fichiers ont la forme trainSPLIT_DU_TRAIN.(no)id.txt (par ex. train2-21.id.txt et train2-21.noid.txt). Le fichier en .id contient les ID des exemples en début de ligne. Les fichiers en .noid ne contiennent plus ces id. Les exemples ont été mélangés aléatoirement pour chaque corpus. Ces fichiers sont au format svmlight ;
* le fichier **alphabet de traits** FEATALPHA décrits plus haut est soit créé soit modifié ;
* éventuellement, en mode -x read, on obtient aussi en sortie un **répertoire read/** contenant des fichiers au format wsj_XXYY.read et  wsj_XXYY.read.simpl contenant tous les exemples dans un format colonne, la première ligne correspond aux catégories. Les fichiers wsj_XXYY.read.simpl contiennent moins d'informations pour rester lisibles.

### Traits utilisables

**Le connecteur des exemples explicites n'apparaît pas dans les arguments**, donc si on ne prend pas de traits concernant le connecteur pour des explicites, on obtient des instances de type artificiel.

La plupart des traits sont utilisables indifféremment pour des exemples de type explicite ou implicite. Cependant, certains traits ne peuvent être utilisés que pour les explicites, et certains traits ne peuvent être utilisés pour les instances de type *discoursereading* (désambiguïsation en emploi) et *position* puisqu'on n'a pas d'argument pour ces types.

* explicit ou implicit :
  * *prodrules* : règles de production de chaque argument,
  * *unigram* : unigrammes de chaque argument,
  * *bigram* : bigrammes de chaque argument,
  * *pcstem* : produit cartésien en stems sur les arguments,
  * *pclem* : produit cartésien en lemmes sur les arguments,
  * *firstlem* : premier lemme de chaque argument,
  * *lastlem* : dernier lemme de chaque argument,
  * *first3lem* : trois premiers lemmes de chaque argument,
  * *firstlastfirst3* : *firstlem*+*lastlem*+*first3lem*
  * *poshead* : POS de la tête de chaque argument,
  * *posheadsimple* : POS simplifié de la tête de chaque argument,
  * *pcheadpos* : produit cartésien des POS des têtes de chaque argument,
  * *meanlenghtvp* : longueur moyenne des VP de chaque argument,
  * *pcheadlemma* : produit cartésien des lemmes des têtes de chaque argument,
  * *overlapstem* : chevauchement en stem entre les arguments,
  * *overlapstemouv* : chevauchement en stem de catégorie ouverte entre les arguments,
  * *overlaplem* : chevauchement en lemmes entre les arguments,
  * *length* : longueur des arguments,
  * *pcverbsamelevin* : somme du nombre de verbes de mêmes classes de Levin,
  * *polarity* : #positive, negate positive, negative, neutral dans chaque argument,
  * *pcpolarity* : somme des *polarity*,
  * *inquirer* : catégorie Inquirer des têtes des arguments,
  * *pcinquirerlemmas* : produit cartésien des *inquirer*,
  * *pcverbstense* : paires des temps, entre passé et présent, des verbes,
  * *number* : dollars, percent, number, dates,
  * *modality* : #modaux dans chaque argument, lemmes modaux dans chaque argument, pc lemmes modaux sur les arguments,
  * *pronouns* : nombre de pronoms de chaque genre et personnes dans chaque argument,
  * *possequence* : séquence des POS de chaque argument,
  * *lemmaopencat* : lemme de catégorie ouverte dans chaque argument,
  * *multisentence* : si un argument est multi-phrastique,
  * *intrawordpairs* : produit cartésien en lemmes à l'intérieur de chaque argument,
  * *position* : si l'exemple est inter ou intraphrastique,
  * *wang* : groupe
    * *polarity*
    * *pcpolarity*
    * *pcinquirerlemmas*
    * *modality*
    * *overlapstemouv*
    * *firstlastfirst3*
    * *pcstem*
    * *intrawordpairs*
  * *wangsimpl* : groupe
    * *polarity*
    * *pcpolarity*
    * *pcinquirerlemmas*
    * *modality*
    * *overlapstemouv*
  * *park* : groupe
    * *firstlastfirst3*
    * *pcverbsamelevin*
    * *meanlenghtvp*
    * *poshead*
    * *polarity*
    * *inquirer*
    * *pcinquirerlemmas*
    * *prodrules*
  * *perso* : groupe
    * *poshead*
    * *pcverbsamelevin*
    * *polarity*
    * *pcpolarity*
    * *inquirer*
    * *pcinquirerlemmas*
    * *prodrules*
    * *modality*
    * *overlapstemouv*
    * *overlapstem*
    * *pcverbstense*
    * *pronouns*
    * *number*
    * *length*
* explicit ou discoursereading ou position :
  * *connective* : forme de connecteur,
  * *pitler* : groupe
  * *wordcontext* : groupe
  * *path* : chemin et chemin compressé du connecteur vers le root,
  * *synsyninteraction* : groupe
  * *connsyninteraction* : groupe
  * *contextinteraction* : groupe
  * *poswdcontext* : groupe
* *artificial* : pour les explicit, prend en compte le groupe *all* pour les implicit ;
* *all* : selon le type,
  * implicit : prend tous les traits de la première liste (sauf les groupes),
  * explicit : prend tous les traits de la seconde liste (sauf les groupes).

# Exemples

### Script shell

**Sans lancer la partie python**

```
./ReaDiC/read_pdtb_like.sh -l ReaDiC/data/ressources/pdtb_hierarchie_lin.txt -e ReaDiC/data/ressources/head_rules.txt -r ReaDiC/data/data2test/PDTB/raw/ -p ReaDiC/data/data2test/PDTB/ptb/ -d ReaDiC/data/data2test/PDTB/pdtb/ -i ReaDiC/data/data2test/out/java_outR/ -g ReaDiC/data/data2test/out/java_outD/ -j ReaDiC/data/data2test/out/java_out/ -m ReaDiC/data/ressources/connective_pdtb.tx`
```

### Script python

Pour créer des fichiers de traits pour les exemples explicites, annotés avec des relations de niveau 1, en ne conservant que la première annotation, avec des traits de type *connective* (uniquement la forme de connecteur comme trait), avec un split train:sections 00 et 01, dev:section 02, test:section 03 :

```
python ReaDiC/py/readCorpus.py -b ReaDiC/data/data2test/out/java_out/ -o ReaDiC/data/data2test/out/read/explicit_connective_lvl1/ -l ReaDiC/data/ressources/pdtb_hierarchie.txt -a ReaDiC/data/data2test/out/read/lexfeats.txt -s 0-1:02:03  -t explicit -n 1 -m 1A
```

Pour créer des fichiers de traits pour les exemples implicites, annotés avec des relations de niveau 2 (mais en ne conservant que les relations conservées par Lin, car utilisation du fichier *pdtb_hierarchie_lin.txt*), en ne conservant que la première annotation, avec des traits de type *bigram* (bigrammes sur chaque argument), avec un split train:sections 00 et 01, dev:section 02, test:section 03 :

```
python ReaDiC/py/readCorpus.py -b ReaDiC/data/data2test/out/java_out/ -o ReaDiC/data/data2test/out/read/implicit_bigram_lvl2/ -l ReaDiC/data/ressources/pdtb_hierarchie_lin.txt -a ReaDiC/data/data2test/out/read/lexfeats.txt -s 0-1:02:03 -t implicit -f bigram -n 2 -m 1A 
```


# TODO

* Supprimer l'argument "labels" dans le pdtb.jar.
* Modifier les affichages des parties Java (un peu le bazar ..).
* modifier le nom du lexique de connecteurs (le même que dans MarTa).
