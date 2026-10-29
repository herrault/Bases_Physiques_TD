# Analyse de la Résilience de la Canopée Amazonienne face à l'Épisode El Niño (2015-2016)

Guide méthodologique et protocole de travaux pratiques (TP) pour l'évaluation de la réponse spatio-temporelle du couvert forestier amazonien à partir de séries temporelles satellitaires MODIS LAI (*Leaf Area Index*).

---

## 📌 Contexte Scientifique et Pédagogique

L'épisode climatique **El Niño de 2015-2016** compte parmi les anomalies thermiques et hydriques les plus extrêmes enregistrées en Amazonie au XXIᵉ siècle. L'arrêt prolongé des précipitations et l'élévation des températures ont induit un stress hydrique aigu au sein du massif forestier. L'article qui vous a été fourni décrit ces perturbations et s'intéresse à divers indicateurs pour décrire comment la végétation a répondu à ces évènements. Nous allons donc en partie reproduire cette étude en établissant différents points de référence permettant de traiter plusieurs questions telles que : Quel est l'impact immédiat de la sécheresse sur la défoliation, spatialement et en terme de magnitude ?  Quelle est la réponse de la végétation un an plus tard, c'est à dire après une saison des pluies complète ? Existe-t'il une mortalité différée de certains arbres conduisant à observer des baisses de LAI plus tardivement ? Pour rappel, voici ces quatre points de référence :

1. **État pré-choc (2013-2014) :** Référence de la forêten conditions initiales
2. **Impact immédiat (2015-2016) :** Pic de sécheresse et défoliation en raison du manque d'eau
3. **Reprise intermédiaire (2017) :** Réponse phénologique post-pluies 
4. **Bilan post-choc (2018) :** Détection de la mortalité différée (*legacy effect*).
---

## 🗂️ Données et Environnement de Travail

* **Produit satellitaire :** MODIS MCD15A2H.v061 (Indice Foliaire LAI à 500 m, composite 8 jours).
* **Zone d'étude :** Tuile MODIS `h12v09` (Bassin amazonien / Mato Grosso).
* **Système de Coordonnées de Référence (SCR) :** Reprojection en WGS 84 (`EPSG:4326`)
* **Logiciels requis :** QGIS (>= 3.28) avec modules GDAL/GRASS activés, R (>= 4.2) avec packages `terra`, `sf`, `ggplot2`, `ggpubr`, `dplyr`.
---

## 🛠️ Déroulement du T

### Séance 1 : Prétraitement, Composites et Moyennes annuelles

L'objectif de cette première séance est de préparer les données dans le but de construire des indicateurs d'anomalies de défoliation. Premièrement, il vous faudra éliminer les artefacts radiométriques (nuages, ombres, plans d'eau) et deuxièmement, construire des moyennes annuelles à partir des différentes dates. Nous souhaitons obtenir une moyenne des valeurs valides pour chaque date. 

1. **Intégration des données brutes :**
   * Importation des rasters LAI MODIS dans QGIS.
   * Vérification de l'emprise et de la plage de valeurs physiques ($0$ à $7\text{ m}^2/\text{m}^2$). Les valeurs supérieures à $7$ (ex. $25{,}4$) correspondent aux masques de nuages ou d'eau.

2. **Constitution des stacks annuels :**
   * Regroupement des scènes 8-jours en stacks annuels (`stack_2013`, `stack_2014`, `stack_2015`, `stack_2016`, `stack_2017`, `stack_2018`) via le menu `Raster > Construire un raster virtuel...` ou `Fusionner`. 

3. **Filtrage des artefacts (Reclassification par lot) :**
   * Outil : **Reclassification par table** (*Reclassify by table*) en mode **Processus par lot**.
   * Règle de reclassification :
     * Plage `[7.001, 255]` $\to$ Nouvelle valeur : `-9999` (NoData).
     * Désactiver l'option *Remplacer les valeurs non trouvées par NoData* pour conserver intactes les valeurs physiques de forêt ($0$ à $7$).
     * Valeur NoData de sortie : `-9999`.

4. **Agrégation temporelle annuelle :**
   * Outil : **GRASS `r.series`**
   * Paramètres :
     * Couches en entrée : rasters reclassifiés de l'année.
     * Opération agrégée : `average` (moyenne arithmétique ignorant les cellules NoData).
   * Sorties générées : `LAI_2013_mean.tif`, `LAI_2014_mean.tif`, `LAI_2016_mean.tif`, `LAI_2017_mean.tif`, `LAI_2018_mean.tif`.

5. **Représentation cartographique des moyennes annuelles :**
  * Appliquez un gradient continu à l'aide d'une palette de couleur sur chaque moyenne annuelle. Attention à conserver les mêmes classes par date dans le but d'établir ces comparaisons. 
  * Notez vos observations, vous en aurez besoin p)ar ailleurs


---

### Séance 2 : Calcul des Anomalies de défoliation

Pour cette séance 2, nous allons calculer plus concrètement les indicateurs. Un moyen d'observer comment une situation s'écarte d'une autre dans les sciences du climat est de s'appuer sur des anomalies. Ces dernières décrivent les écarts par rapport à une référence tandis que le sens des valeurs résultant de ces écarts informe sur la positivité ou au contraire la négativité d'un processus en cours. Chaque point de référence sera décrit pas cette anomalie (voir indicateur ci-dessous). La calculatrice raster est à priivilégier pour l'ensemble des calculs.  

1. **Indicateur 1 : État de référence nominal ($\text{LAI}_{\text{ref}}$)**
   *Interprétation :* Appliquez un gradient continu pour représenter la biomasse foliaire avant perturbation. Que pouvez vous dire des valeurs compris au delà de 5 ? En dessous ?

2. **Indicateur 2 : Anomalie du choc de sécheresse ($\text{LAI}_{\text{sech}}$)**
   *Interprétation :* Appliquez un gradient continu asymétrique de la perte foliaire sous l'effet du pic El Niño. Que signifie les valeurs négatives ou positives ou bien autour de 0, dans un intervalle de +/- 0.5.

3. **Indicateur 3 : Anomalie résiduelle intermédiaire ($\text{LAI}_{\text{reprise}}$)**
   *Interprétation :*Appliquez un gradient continu asymétrique de la résilience à court terme ($+1$ an après perturbation). Que signifie les valeurs négatives ou positives ou bien autour de 0, dans un intervalle de +/- 0.5.

4. **Indicateur 4 : Bilan structurel à long terme ($\text{LAI}_{\text{reprise\_n1}}$)**
   *Interprétation :* De la même manière que précédemment, appliquez un gradient continu. Que signifie les valeurs obtenus et en quoi différent-t'elles des valeurs obtenus précédemment ?
---

### Séance 3 : Masquage Écologique, Typologie de Résilience et Analyse Statistique

#### 1. Masquage de la Forêt Tropicale Humide Fermée
Pour éviter les biais liés aux milieux ouverts, cultures et savanes anthropisées, la zone d'étude est restreinte aux massifs forestiers denses :
```text
("LAI_ref@1" >= 5) / ("LAI_ref@1" >= 5)
```
*Sortie :* `masque_foret_dense.tif` (1 pour forêt dense, NoData pour le reste).  
Les indicateurs d'anomalies sont ensuite multipliés par ce masque pour obtenir `LAI_sech_foret`, `LAI_reprise_foret` et `LAI_reprise_n1_foret`.

#### 2. Matrice de Typologie de Résilience (Calculatrice Raster)
Croisement de l'impact en 2016 et du bilan en 2018 avec un seuil de significativité écologique fixé à $|\Delta\text{LAI}| = 0{,}5\text{ m}^2/\text{m}^2$ :

| Choc 2016 (`LAI_sech`) | État 2018 (`LAI_reprise_n+1`) | Classe | Typologie | Définition Écologique |
| :---: | :---: | :---: | :--- | :--- |
| $\ge -0{,}5$ | $\ge -0{,}5$ | **1** | **Résistante** | Forêt tampon peu vulnérable, aucun décrochage majeur. |
| $< -0{,}5$ | $\ge -0{,}5$ | **2** | **Résiliente** | Forte perte en 2016 suivie d'une cicatrisation complète en 2018. |
| $< -0{,}5$ | $< -0{,}5$ | **3** | **Dégradée** | Dommages structurels immédiats et dépérissement durable. |
| $\ge -0{,}5$ | $< -0{,}5$ | **4** | **Legacy Effect** | Apparence saine en 2016, effondrement différé en 2018 (mortalité retardée). |

Formule appliquée :
```text
( ("LAI_sech_foret@1" >= -0.5) * ("LAI_reprise_n1_foret@1" >= -0.5) * 1 ) +
( ("LAI_sech_foret@1" < -0.5)  * ("LAI_reprise_n1_foret@1" >= -0.5) * 2 ) +
( ("LAI_sech_foret@1" < -0.5)  * ("LAI_reprise_n1_foret@1" < -0.5)  * 3 ) +
( ("LAI_sech_foret@1" >= -0.5) * ("LAI_reprise_n1_foret@1" < -0.5)  * 4 )
```

#### 3. Post-traitement Spatial (Génération des Patchs)
* Outil : **GRASS `r.neighbors`**
  * Paramètres : Voisinage circulaire ou carré de taille `3`, méthode `mode`.
  * Rôle : Lissage modal éliminant le bruit résiduel tout en conservant les frontières des massifs.
* Calcul des surfaces : Utilisation de l'outil *Rapport d'analyse de valeur unique de raster* pour extraire le décompte des pixels et les surfaces en $\text{km}^2$.

---

## 📊 Échantillonnage et Tests Statistiques sous R

Pour tester si la densité foliaire initiale conditionne la vulnérabilité et analyser le comportement transitoire de 2017 :

1. Vectorisation du raster lissé (`Polygonaliser`).
2. Regroupement par classe (`Regrouper` / *Dissolve*).
3. Tirage aléatoire équilibré de 100 à 300 points par entité (`Points aléatoires dans les polygones`).
4. Export au format GeoPackage (`points_resilience_sampling.gpkg`).

### Script R d'Analyse Comparative (Violin Plots & Tests de Wilcoxon)

```r
library(terra)
library(sf)
library(dplyr)
library(tidyr)
library(ggplot2)
library(ggpubr)

# 1. Chargement du vecteur d'échantillonnage et des rasters
pts <- st_read("points_resilience_sampling.gpkg")
r_ref <- rast("LAI_ref.tif")
r_rep <- rast("LAI_reprise.tif")

# 2. Extraction des valeurs
nom_classe <- if("classe" %in% names(pts)) "classe" else "DN"
pts_terra <- vect(pts)

df <- data.frame(
  classe      = as.factor(pts[[nom_classe]]),
  LAI_ref     = terra::extract(r_ref, pts_terra)[[2]],
  LAI_reprise = terra::extract(r_rep, pts_terra)[[2]]
) %>%
  drop_na()

levels(df$classe) <- c("Résistante", "Résiliente", "Dégradée", "Legacy")

# 3. Paires de comparaison statistique
comparaisons <- list(
  c("Résistante", "Résiliente"),
  c("Résiliente", "Dégradée"),
  c("Dégradée", "Legacy"),
  c("Résistante", "Legacy")
)

palette_classes <- c(
  "Résistante" = "#2a9d8f",
  "Résiliente"  = "#8ab17d",
  "Dégradée"    = "#e76f51",
  "Legacy"      = "#9b5de5"
)

# 4. Figure A : LAI initial (2014)
p1 <- ggplot(df, aes(x = classe, y = LAI_ref, fill = classe)) +
  geom_violin(trim = FALSE, alpha = 0.5, color = NA) +
  geom_boxplot(width = 0.18, color = "black", outlier.size = 0.8, alpha = 0.8) +
  scale_fill_manual(values = palette_classes) +
  scale_y_continuous(expand = expansion(mult = c(0.05, 0.15))) +
  stat_compare_means(comparisons = comparaisons, method = "wilcox.test", label = "p.signif") +
  labs(
    title = "Densité foliaire pré-sécheresse (2014)",
    x = "Typologie de résilience",
    y = expression("LAI de référence ("*m^2*"/"*m^2*")")
  ) +
  theme_minimal(base_size = 12) +
  theme(legend.position = "none", panel.grid.minor = element_blank())

# 5. Figure B : Anomalie intermédiaire (2017)
p2 <- ggplot(df, aes(x = classe, y = LAI_reprise, fill = classe)) +
  geom_hline(yintercept = 0, linetype = "dashed", color = "grey50") +
  geom_violin(trim = FALSE, alpha = 0.5, color = NA) +
  geom_boxplot(width = 0.18, color = "black", outlier.size = 0.8, alpha = 0.8) +
  scale_fill_manual(values = palette_classes) +
  scale_y_continuous(expand = expansion(mult = c(0.05, 0.15))) +
  stat_compare_means(comparisons = comparaisons, method = "wilcox.test", label = "p.signif") +
  labs(
    title = "Anomalie pendant la reprise intermédiaire (2017)",
    x = "Typologie de résilience",
    y = expression(Delta*"LAI 2017 ("*m^2*"/"*m^2*")")
  ) +
  theme_minimal(base_size = 12) +
  theme(legend.position = "none", panel.grid.minor = element_blank())

# 6. Assemblage
fig <- ggarrange(p1, p2, ncol = 2)
ggsave("resultats_resilience_amazonie.png", fig, width = 14, height = 7, dpi = 300)
```

---

## 🔬 Principaux Résultats Écologiques Attendus

* **Vulnérabilité des fortes biomasses foliaires :** Les peuplements de la classe *Dégradée* présentent des valeurs initiales de LAI significativement plus élevées en 2014 ($p < 0{,}05$). En contexte de déficit hydrique sévère, leur demande évapotranspiratoire accrue précipite le phénomène de rupture hydraulique par cavitation.
* **Le piège phénologique de 2017 (*Flush* foliaire) :** La classe *Legacy* conserve une anomalie neutre ou positive en 2017, indiscernable statistiquement de la classe *Résistante* (`ns`). Cette reprise temporaire de surface foliaire masque l'épuisement des réserves carbonées internes qui conduit à une mortalité différée mesurable uniquement à $+2$ ans (2018).
