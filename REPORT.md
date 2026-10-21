# REPORT - TP1 DataOps : du script fragile au pipeline testé

## Étape 0
**Q1.** On écrit les bibliothèques dans `requirements.txt` pour rendre le projet
**reproductible** : n'importe qui peut recréer le même environnement avec
`pip install -r requirements.txt`, au lieu de deviner quoi installer.

## Étape 1
**Q2.** Le rapport affiche **8 lignes** au lieu de 5 : ` Tunisia`, `Tunisia`,
`tunisia`, `Morocco`, `MOROCCO`, `France`, `Germany`, `Italy`. Pandas traite
chaque variante de casse ou d'espaces comme un pays distinct.

**Q3.**

| order_id | Problème | Dimension |
|---|---|---|
| 1004 | Quantité négative (-2) | Validité |
| 1008 | customer_id vide | Complétude |
| 1013 | Date `2026-13-45` impossible | Validité |
| 1006, 1016, 1023 | `tunisia`, `MOROCCO`, ` Tunisia` | Consistance |
| 1010 | Commande présente deux fois | Unicité |
| 1019 | Clavier à 4500 au lieu de 45 | Exactitude |

**Q4.** Le total est trop **haut** (12771,10 contre 7655,90 réel). On ne peut pas
le deviner sans recalculer : le doublon et le prix aberrant le gonflent, la
quantité négative le réduit.

## Étape 2
**Q5.** (1) Chaque étape peut être **testée séparément** avec des tests unitaires.
(2) Le code est **plus lisible et plus facile à modifier** : on change une règle
sans toucher à la lecture ni à l'écriture.

## Étape 3
**Q6.** Total correct : **7655,90**. L'ancien script donnait 12771,10, soit une
erreur de (12771,10 - 7655,90) / 7655,90 = **+66,8 %**.

**Q7.** La commande **1019** (clavier à 4500 au lieu de 45) cause l'essentiel de
l'erreur (4455 en trop). Elle est attrapée par la règle 4, `invalid_unit_price`.

**Q8.** Le fichier de quarantaine permet d'**auditer** les rejets, de **corriger
à la source** et de **réinjecter** les lignes. Les supprimer ferait perdre la
trace de ce qui a été écarté et pourquoi.

## Étape 4
**Q9.** Avec `> -5`, la quantité -2 n'est plus rejetée : le test
`test_negative_quantity_is_rejected` échoue, car la commande 1 reste dans les
données propres.

**Q10.** Un **test unitaire** vérifie que **ma logique de code** est correcte, sur
une petite table écrite à la main. Un **test de données** vérifie que **les
données du jour** sont acceptables, sur les vraies données à chaque exécution.

## Étape 6
**Q11.**

| | Ancien script | Ton pipeline |
|---|---|---|
| Jour 2 : message d'erreur ? | Aucun | Oui : taux de rejet 100 %, pipeline arrêté |
| Jour 2 : total affiché | 270200 (faux) | Aucun, rien publié |
| Jour 3 : message d'erreur ? | `KeyError: 'unit_price'` (plantage) | Oui : `Missing columns: ['unit_price']`, pipeline arrêté |

**Q12.** **Jour 2** : les prix sont en **centimes** (44900 au lieu de 449,00),
donc tous supérieurs à 1000 et rejetés. **Jour 3** : la colonne `unit_price` a
été **renommée en `price`**.

**Q13.** Le **jour 2** est le plus dangereux : l'ancien script ne plante pas et
publie un total environ 100 fois trop grand (erreur silencieuse). Le jour 3
plante bruyamment, donc le problème se voit.

**Q14.** Le code de sortie (`1` = échec) peut être lu par un **orchestrateur**
(Airflow, cron), un **pipeline CI/CD** ou un **script d'alerte** pour déclencher
une alerte, arrêter les étapes suivantes ou relancer.