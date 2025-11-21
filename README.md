# Global Development Dashboard Documentation

## 👥 Présentation de l'équipe
- Membres : Salma, Safiya, asmaa, fatimazahra
- Taches : voir Trello

![trello](https://github.com/user-attachments/assets/6f31a6e7-b59d-46ef-a1a8-78f97f968003)



## Importation des donnes via les APIs (WORLD BANK / Rest Coutries)
### PHASE 1 : IMPORT DES DONNÉES WORLD BANK
#### Étape 1.1 : Comprendre l'API World Bank :`

1.1.1- Structure de l'URL :
https://api.worldbank.org/v2/country/all/indicator/{code}?date=2015:2022&format=json&per_page=20000

{code} => on le remplace par le code de l'indicateur qu'on est besoin a importer
date=2015:2022 => pour importer juste les donnees correspend a la periode entre 2015-2022
per_page=20000 => sans le parametre per_page, World Bank API applique une limite de donnees a importer  par défaut (50 résultats par page)

1.1.2- Les 4 indicateurs qu'on est besoin :
|Indicateur      |Code API            |Signification                |
|----------------|--------------------|-----------------------------|
|PIB             |NY.GDP.MKTP.CD      |PIB en dollars US courants   |
|Population      |SP.POP.TOTL         |Population totale            |
|CO₂             |EN.GHG.CO2.MT.CE.AR5|Émissions de CO₂ (kilotonnes)|
|Esperance de vie|SP.DYN.LE00.IN      |Émissions de CO₂ (kilotonnes)|


#### Étape 1.2 : Importer les donnees correspond au PIB :
pour importer les donnees qui correspond a cet indicateur on a suit les etapes suivantes:

a- ouvrir Power BI

b- transformer les donnees (on est dans power query)

c- Nouvelle source --> Web --> saisir le lien: https://api.worldbank.org/v2/country/all/indicator/NY.GDP.MKTP.CD?date=2015:2022&format=json&per_page=20000

e- une colonne s'affiche contient deux elements :
    - Record : des metadonnees a ignorer
    - List: c'est ce qui nous interesse, elle contient les donnees (cliquer sur List)

f- On voit une colonne (Liste des elements) --> On clique sur : Vers la table

g- une colonne appelee Column1 avec des enregistrements --> On clique sur : <-> a cote de "Column1" -->On selectionne juste : indicator, country, countryiso3code, date, value

<img width="920" height="500" alt="image" src="https://github.com/user-attachments/assets/b8aac8fd-b9c4-45f8-8efa-8667036725ef" />


h- developper encore (colonnes imbriquees) et pour la colonne Column1.indicator(id, value) et pour la colonne Column1.country(id, value)

<img width="920" height="500" alt="image-1" src="https://github.com/user-attachments/assets/7312423d-7daa-41da-9a4e-a508751da8db" />


i- supprimer colones inutiles et renommer les colonnes restantes :
   - supprimer les colonnes inutiles : Column1.indicator.id, Column1.indicator.value, Column1.country.id
   - renommer la colonne :
       - Column1.country.value -> Nom_Pays
       - Column1.countryiso3code -> cca3
       - Column1.date -> Annee
       - Column1.value -> PIB

j- Renommer la requête : 2022 -> WB_PIB

On suit les memes etapes pour les autres indicateurs :


#### Étape 1.3 : Importer les donnees correspond a la population :

url: https://api.worldbank.org/v2/country/all/indicator/SP.POP.TOTL?date=2015:2022&format=json&per_page=20000

i- renommer la Column1.value -> Population

#### Étape 1.4 : Importer les donnees correspond a l'emission de Co2 :

url: https://api.worldbank.org/v2/country/all/indicator/EN.GHG.CO2.MT.CE.AR5?date=2015:2022&format=json&per_page=20000

i- renommer la Column1.value -> Mt CO2e

#### Étape 1.5 : Importer les donnees correspond a l'espirance de vie :

url: https://api.worldbank.org/v2/country/all/indicator/SP.DYN.LE00.IN?date=2015:2022&format=json&per_page=20000

i- renommer Column1.value -> Esperence_vie

#### Étape 1.6 : Fusionnement des tables :

Pour avoir une seule table(**world bank data**) qui contient tous les indicateurs, on doit fusionner les tables **Population**, **Esperance de vie** et **CO2** avec la table PIB selon la colonne **cca3**

Voici les etapes de fusionnement qu'on a suit:
a- selectionner PIB
b- Accueil → Combiner → Fusionner des requêtes
c- configuration de fuionnement:
   └──Table 1 : PIB (déjà sélectionnée)
      └── Colonnes de correspondance : Sélectionner cca3 puis Annee
          └── Table 2 : Population
                  └── Colonnes de correspondance : Sélectionner cca3 puis Annee
                      └── Type de jointure : Externe gauche

d- maintenant on a une nouvelle colonne "Population" a droite.
---en cliquant sur <-> a cote de "Population"
---on decocher toutes les colonnes SAUF : Population

e- decocher "Utiliser le nom de colonne d'origine..." pour que le nom de la colonne reste le meme dans la table initiale

=> les memes etapes se repete avec les autres tables jusqu'a avoir tous les indicateurs dans une seule table


# PHASE 2 : IMPORT DES DONNÉES REST COUNTRIES
Voici ce qu'on est besoin d'apres cet API:

| Besoin KPI                                              | Champ REST Countries |
| ------------------------------------------------------- | -------------------- |
| Population                                              | `population`         |
| Superficie                                              | `area`               |
| Région                                                  | `region`             |
| Sous-région                                             | `subregion`          |
| Capitale                                                | `capital`            |
| Code pays ISO3 (clé pour World Bank)                    | `cca3`               |
| Nom du pays                                             | `name.common`        |
| Independence                                            | `independent`        |

#### Étape 2 : Se connecter à l'API :
a- Nouvelle source → Web

b- url: https://restcountries.com/v3.1/all?fields=name.common,cca3,region,subregion,population,area,languages,independent

c- Gerer la colonne des languages:
la colonne languages contient [Record] (non développée), pour extraire la premiere langue on a suit les etapes suivantes:
└── Ajouter une colonne
    └── Colonne personnalise
        └── Nouveau nom de colonne : language
        └── Formule de colonne personnalise : try Record.FieldValues([languages]){0} otherwise null
            └── supprimer languages (record)


#### Étape 3 : Filtrage des Agrégats World Bank :

**Problème identifié** : 
L'API World Bank retourne non seulement des pays individuels, mais aussi des agrégats régionaux (unions, groupes économiques) qui fausseraient nos analyses.

**Solution appliquée** :
Fusion avec jointure INTERNE entre `WB_Indicateurs` et `countries` (référentiel REST Countries) sur la colonne `Code_ISO3`.

**Impact** :
- **Avant** : 7088 lignes (+265 entités incluant agrégats)
- **Après** : 1720 lignes (250 pays réels uniquement)
- **Supprimé** : 70 agrégats (WLD, EUU, ARB, HIC, SSF, etc.)

**Justification** :
- Garantit la cohérence entre FactIndicateurs et DimPays
- Basé sur un référentiel international officiel (REST Countries)



## Preparation et Nettoyage des donnes
Maintenant on a deux tables essetiels:

***********************************
Table **world bank data** (1720 lignes)
└── Nom_Pays
└── Code_ISO3
└── Annee
└── CO2
└── Esperence de vie
└── PIB
└── Population
************************************
Table **Countries** (250 lignes)
└── common
└── Code_ISO3
└── region
└── subregion
└── area
└── language
************************************

# Nettoyage de world bank data:

#### Étape 1 : Verifier et Modifier types de donnees :
└── Nom_Pays --> text
└── Code_ISO3 --> text
└── Annee --> entier
└── CO2 --> decimal
└── Esperence de vie --> decimal
└── PIB --> decimal
└── Population --> entier

#### Étape 2 : Gere les valeurs nulls :
dans ce tableau on deux colonnes ayant des valeurs nulls:
└── CO2: 6% vide 
└── PIB: 3% vide

##### Étape 2.1 : Gere les valeurs nulls pour CO2:

🎯 Objectif : Certaines lignes de notre dataset contiennent une valeur CO₂ = null.
L’objectif est de choisir une méthode logique et cohérente pour remplacer ces valeurs sans fausser les analyses.

Notre décision repose sur :
--> la taille du pays (population)
--> son niveau d’industrialisation
--> sa situation géopolitique (guerre, sanctions, isolement)
--> sa disponibilité réelle de données internationales
--> la cohérence avec les organismes officiels (Banque Mondiale, UNData, Our World in Data)

🧩 Pays concernés: 
Les pays ayant CO₂ = null et durant toule la periode [2015-2022] dans le dataset sont :

Andorra, Curacao, Isle of Man, Liechtenstein, Monaco, Montenegro, San Marino, Serbia, Sint Maarten, South Sudan, St. Martin, West Bank and Gaza

🧠 Analyse par catégorie de pays :
Micro-États (<100k habitants, peu d’industrie) → CO₂ ≈ 0 :
→ Pour: Andorra, Monaco, Liechtenstein, San Marino, Isle of Man, St. Martin, Sint Maarten

<p align="center">
  <img src="https://github.com/user-attachments/assets/08a394a1-c377-41ee-b88d-968bb66a9975" width="350">
</p>

Pays avec industrie ou population significative → utiliser année précédente :
→ Pour: Montenegro, Serbia, Curacao

Explication :
Même des pays industrialisés ou avec population significative peuvent avoir des nulls si :
- Les données n’ont pas été collectées localement.
- Le reporting international est incomplet ou retardé.
- Les estimations ne sont pas fiables.
=> Donc, il est normal et logique de conserver null dans ces cas plutôt que de créer des données artificielles.

Pays en guerre ou contexte instable → garder null :
→ Pour: South Sudan, West Bank and Gaza

🛠️ Étapes de traitement dans Power Query : 
1️⃣ Ajouter une colonne -> colonne personnalise
2️⃣ Nouveau nom de colonne : co2 
3️⃣ Formule de colonne personnalise : 
    if [CO2] = null 
    and List.Contains({"AND","MCO","LIE","SMR","IMN","MAF","SXM"}, [Code_ISO3]) 
    then 0 
    else [CO2]

4️⃣ Supprimer la colonne de CO2 initiale :

On a maintenant la colonne CO2 avec 2% des valeurs vides

##### Étape 2.2 : Gere les valeurs nulls pour PIB:

voici le code iso3 des pays ayant PIB null: 
SSD, CUB, PRK, VEN, ERI, MAF, VGB, GIB

a- Pour SOUTH SUDAN (SSD): On va garder les vals null de SOUTH SUDAN de GDP tel que Les bases de données internationales comme World Bank manquent encore d’historique complet car Les guerres civiles ont empêché la collecte fiable de données 

b- Pour CUB: remplissage du pays CUB qui a val null juste dans 2021 et 2022 par moyenne des GDP dans [2015-2020].

<p align="center">
  <img src="https://github.com/user-attachments/assets/737d47c2-e980-4fd0-a1ba-0b04fe0842fb" width="350">
</p>


c- Pour YEM: On va utiliser la dernière valeur connue (2018) pour remplir 2019–2022 car la moyenne créerait une valeur plus haute que les dernières années, ce qui serait irréaliste, car le pays était en pleine guerre et son économie continuait de s’effondrer.


<p align="center">
  <img src="https://github.com/user-attachments/assets/0522fc1b-5362-42bb-b0d0-b523d5012366" width="350">
</p>

d- PRK – Corée du Nord (North Korea): la Corée du Nord (PRK) apparaît dans la base de la Banque mondiale avec certaines données (population, espérance de vie, CO₂), mais pas le PIB car ces données ne proviennent PAS officiellement du gouvernement nord-coréen mais la Banque mondiale peut utilise des estimations externes .Mais le PIB est impossible à estimer sans comptes nationaux donc il va rester null.

e- VEN – Venezuela: Les données économiques de Venezuela existent mais ne sont plus publiées depuis la crise économique. Après 2014, le Venezuela a cessé de transmettre ses séries économiques et nous avons travailler avec les infos juste depuis 2015 donc il va rester null.

f- ERI – Érythrée: Le gouvernement érythréen ne transmet aucune donnée économique officielle donc on a aucune information sur PIB alor il va rester null.

g- meme pour VGB et GIB

h- MAF : on a que deux annees ayant valeurs non nulls pour PIB, et on peux pas estimer PIB de 6 ans juste a partir de PIB de deux annees.

**Conclusion**:
On remplace par moyenne pour les pays ayant au min PIB de 4ans non nulls(cas de: CUB, ).
sinon reste null 

Apres Nettoyage :
└── CO2: 2% vide 
└── PIB: 3% vide (meme si le % des valeurs nulls reste le meme mais on moins on est capale d'analyser le developpement economique de 2 pays parmis ces 8)


# Nettoyage de countries:

#### Étape 1 : Verifier et Modifier types de donnees :
└── Nom_Pays --> text
└── Code_ISO3 --> text
└── region --> text
└── subregion --> text
└── language --> text
└── area --> decimal
└── Population --> entier
└── Independent --> entier

#### Étape 2 : Gere les valeurs nulls :
dans ce tableau on deux colonnes ayant des valeurs nulls:
└── language: 1% vide 

Explication: elle concerne le pays L'Antarctique. C'est un pays qui n'a pas de langue officielle car ce n'est pas un pays avec une population permanente.

=> decision: null -> Unknown

└── subregion: 2% vide

=> decision: null -> Unclasified


Afin d'avoir une modelisation...

voici les tables abordes pour notre modelisation:

![modilisation](https://github.com/user-attachments/assets/f08f1a7b-91aa-42ec-b5a4-8397bb1ade16)



#### 📝 Étape 4 – Création des mesures (DAX)

## Objectif
Traduire les formules mathématiques des KPI en mesures DAX dans Power BI, en vérifiant la cohérence des unités et des formats.

---

##  Tableau des KPI

| Thème | KPI | Description |
|-------|-----|-------------|
| Économie | PIB total (USD) | Valeur totale du PIB par année et par pays. |
|  | Croissance du PIB (%) | Variation du PIB d’une année sur l’autre. |
|  | PIB par habitant (USD) | PIB total / population. |
|  | Part du PIB régional (%) | Part du PIB d’une région dans le total mondial. |
|  | Évolution du PIB depuis 2015 (%) | Croissance cumulée du PIB depuis 2015. |
| Population & Démographie | Population totale | Somme des habitants. |
|  | Croissance démographique (%) | Variation annuelle de la population. |
|  | Densité de population | Population / superficie du pays. |
| Environnement | Émissions totales de CO₂ (kt) | Quantité totale annuelle de CO₂ émise. |
|  | Émissions par habitant (t/hab) | CO₂ total / population. |
|  | Intensité carbone (CO₂/PIB) | Émissions de CO₂ / PIB total. |
|  | Évolution des émissions CO₂ depuis 2015 (%) | Taux de variation cumulée depuis 2015. |
| Développement durable | PIB / CO₂ ratio | Efficience économique (production par tonne de CO₂). |
|  | PIB / Population / Superficie | Indicateur composite de productivité par km². |
|  | Espérance de vie moyenne* | (si ajoutée via World Bank) Indicateur social complémentaire. |
| Comparatifs régionaux | Classement mondial par PIB | Rang du pays en PIB global. |
|  | Classement par intensité carbone | Rang du pays selon pollution par unité économique. |




#### 📝 Étape 5 – Construction du tableau de bord Power BI

## Objectif
Créer un tableau de bord interactif de 4 pages, permettant de visualiser les KPI et d’explorer les données selon différents segments.

---

##  Pages de visualisation

| Page | Objectif | Visualisations clés |
|------|----------|-------------------|
| Monde | Vue globale des indicateurs | Carte du monde, graphiques d’évolution du PIB, population et CO₂, indicateurs clés |
| Région | Comparaison par continents et sous-régions | Histogrammes, cartes choroplèthes, KPI résumés par région |
| Pays | Fiche pays détaillée | Tableaux et cartes par pays, évolution du PIB, CO₂, population |
| Corrélation & Durabilité | Analyse de l’impact économique vs environnemental | Graphiques de dispersion PIB vs CO₂, ratio CO₂/PIB, tendances par région |


#### Vue mondiale:

<img width="1018" height="575" alt="image" src="https://github.com/user-attachments/assets/31599deb-9a8b-4c77-a42f-e910f03227c8" />

#### Vue Régional:
##### page 1:

<img width="1018" height="575" alt="image" src="https://github.com/user-attachments/assets/94c1e31b-85d9-48ff-a4db-d8a46b13005c" />

##### page 2:

<img width="1016" height="577" alt="image" src="https://github.com/user-attachments/assets/c86e9481-9529-4b69-a763-52b1d6ccc4da" />


#### Vue de pays:

<img width="1016" height="572" alt="image" src="https://github.com/user-attachments/assets/9b395c92-f442-4889-b048-cf8a40be4322" />

#### Vue de correlation:

<img width="1023" height="581" alt="image" src="https://github.com/user-attachments/assets/3411d0ba-acd3-4a5b-b13d-60b1962ddc2b" />


# Analyse et interprétation:

## les 3 pays choisies (ex : États-Unis, Inde, Nigeria).

### Quel pays a connu la plus forte croissance depuis 2015 ?
Parmi les trois pays analysés, l’Inde est celui qui présente la plus forte croissance économique depuis 2015, portée par une expansion démographique soutenue et une transition industrielle rapide.

### Quelle région a amélioré le plus son ratio PIB/CO₂ ?
La région qui améliore le plus son ratio PIB/CO₂ est l’Europe, grâce à une croissance économique modérée combinée à une réduction progressive des émissions.

### Quels pays combinent forte croissance et faible pollution ?
Parmi les pays étudiés, le Nigeria présente la meilleure combinaison de croissance économique et de faibles émissions absolues de CO₂, bien que son intensité carbone reste à surveiller. L’Inde et les États-Unis sont des pays à forte croissance mais fortement émetteurs.

### Quelles corrélations apparaissent entre PIB, population et CO₂ ?
On observe une corrélation positive entre PIB et émissions de CO₂ : plus un pays produit de richesse, plus il génère d’émissions. La relation entre population et CO₂ est moins systématique, car certains pays très peuplés restent faibles émetteurs. La population seule n’explique donc pas la pollution : c’est surtout l’intensité industrielle qui joue un rôle clé.

### Quels enseignements stratégiques pour GDW ?
Global Development Watch devrait prioriser les investissements dans les pays combinant potentiel de croissance et faible pollution, tout en accompagnant les grandes économies vers des solutions énergétiques propres. Une stratégie différenciée selon les régions est essentielle pour maximiser l’impact du programme World Progress 2030.

