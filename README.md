# ChallengeData-ENEDIS


# Contexte

Le compteur intelligent Linky a été déployé à la majeure partie des 38 millions de clients basse tension d’Enedis. Selon le choix du client, il transmet à Enedis l’index de consommation journalier ou les courbes de consommation infra-journalières.

Enedis utilise certaines de ces courbes pour estimer des agrégats de consommation qui sont utilisés dans le cadre du mécanisme d’ajustement et de réglement des écarts. Ce mécanisme permet d’assurer le respect de l’équilibre offre/demande sur le marché électrique français.

Comme tout processus informatique de masse, la collection des données peut subir des problèmes techniques à toutes les étapes de la mesure, de la transmission ou du stockage des données, causant des trous de données. Pour remplir ses missions Enedis doit développer des algorithmes de complétion des données, mais sans utiliser les courbes des consommateurs réels, pour des raisons de respect du réglement général de protection des données (RGPD).

# But 

Le challenge que nous proposons consiste à reconstruire les données manquantes de certaines courbes de consommation électrique en utilisant uniquement d’autres courbes de consommation électrique.

Nous avons utilisé un outil de génération de courbes synthétiques, DeepCourbogen, pour générer environ 69 000 courbes. 1 000 de ces courbes ont subi des suppressions aléatoires de données, pour simuler les données manquantes de courbes réelles.

Le but des challengers est de proposer des remplacements pour les données manquantes (« remplir les trous ») dans les 1 000 courbes.

# Description des données 

Pour l’échantillon d’entrainement :

- la donnée d’entrée est une dataframe de 21 000 colonnes. Chacune d’entre elles est une courbe synthétique générée par DeepCourbogen. Chaque nom de colonne est un identifiant () généré aléatoirement. L’index de la dataframe est le timestamp de chaque point, les valeurs de consommation sont en watts. Les 1 000 dernières colonnes contiennent les courbes auquelles il manque des données. Les noms de ces colonnes sont de la forme holed_
- La donnée de sortie est une dataframe de 1 000 colonnes contenant les données complétées des 1 000 dernières colonnes des données d’entrée.
- Le fichier d’entrée fait 140 Mo, le fichier de sortie 6 Mo

Pour l’échantillon de test, la donnée d’entrée est une dataframe de 38140 colonnes (260 Mo), la donnée de sortie fait toujours 1000 colonnes (6 Mo)

# Description du benchmark 

Pour ce challenge, nous avons utilisé un benchmark basique consistant à faire de l’interpolation linéaire sur les données manquantes.
Globalement, cette méthode remplit les valeurs manquantes par les valeurs prises par la fonction affine passant par le précédent point connu et le point connu suivant.
Pour plus de détails, se référer à [cet article](https://fr.wikipedia.org/wiki/Interpolation_lin%C3%A9aire).

```python
def fill_nan_with_interpolation(column):
    col = column.copy()
    col = col.interpolate(method='linear', limit_direction='both')
    return col
y_pred = X_test_holed.apply(fill_nan_with_interpolation, axis=0)
```

# Métrique

La métrique utilisée pour l’évaluation est une simple **MAE** (Erreur Absolue Moyenne) qui est la moyenne des erreurs absolues ramenée au nombre réel de valeurs manquantes.