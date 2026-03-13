# README - Algorithmes de Recherche dans un Labyrinthe

Ce projet implémente et compare trois algorithmes de recherche de chemin sur un labyrinthe en Python avec Pygame : **BFS** (Recherche en Largeur), **DFS** (Recherche en Profondeur) et **UCS** (Recherche à Coût Uniforme). Les réponses ci-dessous s'appuient sur les observations faites en exécutant les trois visualisations.

---

## Question 1 - Quelle recherche trouve la solution le plus rapidement ?

En pratique sur ce labyrinthe, c'est **UCS avec le g-score de Manhattan (Version 3)** qui trouve la sortie le plus vite. On le voit clairement dans la visualisation : le pingouin se dirige presque en ligne droite vers l'arrivée, sans trop s'éparpiller sur les côtés.

Le **DFS** peut parfois surprendre en trouvant la sortie très rapidement lui aussi - mais c'est une question de chance. Si la première branche qu'il explore pointe dans la bonne direction, il fonce. Si ce n'est pas le cas, il peut errer longtemps dans une mauvaise partie du labyrinthe avant de revenir.

Le **BFS** est plus régulier mais plus lent visuellement : on le voit "gonfler" depuis le départ en cercles concentriques, visitant beaucoup de cases avant d'atteindre la sortie.

Quant à **UCS V1** (coûts aléatoires), c'est clairement le plus lent des quatre. Puisque les coûts ne sont pas liés à la position des cases, l'algorithme n'a aucune raison de préférer les cases proches de la sortie - il explore un peu partout.

**Classement du plus rapide au plus lent :**
1. UCS V3 - Manhattan *(très dirigé vers la sortie)*
2. DFS *(rapide si la topologie du labyrinthe est favorable, sinon dernier)*
3. BFS *(régulier mais exhaustif)*
4. UCS V1 - coûts aléatoires *(exploration quasi totale du labyrinthe)*

---

## Question 2 - Avantages et inconvénients de chaque recherche

### BFS - Recherche en Largeur

BFS explore le labyrinthe couche par couche à partir du départ. C'est l'algorithme le plus "juste" des trois pour trouver le chemin le plus court.

**Avantages :**
- Il garantit toujours le chemin le plus court en nombre de pas. C'est sa grande force.
- Il est complet : s'il existe un chemin, BFS le trouvera forcément.
- Son comportement est très prévisible et facile à analyser.

**Inconvénients :**
- Il utilise beaucoup de mémoire, car il stocke en même temps toute la "frontière" (toutes les cases à explorer).
- Sur un grand labyrinthe, il visite énormément de cases avant d'arriver à la sortie, ce qui le rend lent visuellement.
- Il ne tient pas compte des coûts de déplacement - toutes les cases sont traitées de façon égale.

---

### DFS - Recherche en Profondeur

DFS plonge dans une direction jusqu'au bout, puis revient en arrière si ça ne mène nulle part (backtracking). C'est l'algorithme qui ressemble le plus à la façon dont un humain explore un labyrinthe à l'aveugle.

**Avantages :**
- Il consomme très peu de mémoire : il ne stocke que le chemin actuel, pas toute la frontière.
- Dans le meilleur cas, il trouve la sortie extrêmement vite - si la première branche explorée est la bonne.
- Simple à implémenter, naturellement adapté aux structures de type arbre.

**Inconvénients :**
- Il ne garantit pas le chemin le plus court. Le chemin trouvé peut être bien plus long que nécessaire.
- Ses performances sont très imprévisibles : tout dépend de la forme du labyrinthe et de l'ordre d'exploration des voisins.
- Sur un labyrinthe défavorable, il peut passer un temps très long à explorer de mauvaises branches.

---

### UCS - Recherche à Coût Uniforme

UCS utilise une file de priorité pour toujours explorer en premier la case dont le coût cumulé depuis le départ est le plus faible. C'est fondamentalement l'algorithme de Dijkstra.

**Version 1 - coûts aléatoires**

**Avantages :**
- Garantit le chemin de coût minimal selon les coûts définis (utile si les cases ont vraiment des difficultés différentes, comme du terrain accidenté).
- Peut modéliser des situations réalistes où traverser certaines zones coûte plus cher.

**Inconvénients :**
- Avec des coûts aléatoires, l'algorithme n'a aucune information sur la direction de l'arrivée.
- Il explore potentiellement tout le labyrinthe avant de trouver la sortie - c'est le plus lent des quatre.

**Version 2 - coût = distance en colonnes à l'arrivée**

**Avantages :**
- Déjà beaucoup mieux que V1 : l'exploration se dirige horizontalement vers la sortie.
- Simple à calculer : `abs(colonne_case - colonne_arrivée) + 1`.

**Inconvénients :**
- Ignore complètement la dimension verticale. Si la sortie est sur une ligne très différente, l'algorithme peut se retrouver bloqué à chercher sur la bonne colonne sans descendre/monter suffisamment.

**Version 3 - coût = distance de Manhattan (recommandée)**

**Avantages :**
- Guide l'exploration dans les deux dimensions à la fois (horizontale + verticale).
- Très efficace : réduit drastiquement le nombre de cases visitées.
- Se comporte comme A* avec une heuristique de Manhattan.

**Inconvénients :**
- Utilise la position de l'arrivée pour calculer les coûts → c'est techniquement une recherche *informée* (contrairement à BFS et DFS).
- Sur un labyrinthe avec de longs détours obligatoires, peut être légèrement trompé (mais trouve quand même le chemin).

---

### Tableau comparatif

| Critère | BFS | DFS | UCS V1 | UCS V3 |
|---|---|---|---|---|
| Chemin optimal ? |  Oui (en pas) |  Non |  Oui (en coût) |  Oui |
| Vitesse | Moyenne | Variable | Très lente | Rapide |
| Mémoire | Élevée | Faible | Élevée | Moyenne |
| Recherche informée ? | Non | Non | Non | Oui |
| Garantie de trouver ? |  Oui |  Oui |  Oui |  Oui |

---

## Question 3 - Une recherche non informée plus optimale ?

Parmi les recherches non informées (sans connaissance de la position de l'arrivée), la plus efficace pour un labyrinthe serait le **BFS bidirectionnel**.

### Principe

L'idée est simple : au lieu de lancer un seul BFS depuis le départ, on en lance **deux en parallèle** - un depuis le départ S, et un autre depuis l'arrivée E. On alterne une étape de chaque côté à chaque itération. L'algorithme s'arrête dès que les deux frontières se rencontrent sur un nœud commun. Le chemin final est alors la concaténation du chemin S → nœud commun et du chemin E → nœud commun (inversé).

### Pourquoi est-ce plus efficace ?

Le BFS classique explore un "disque" autour du départ dont le rayon grandit jusqu'à atteindre la sortie. Si la solution est à une profondeur `d`, il explore environ `b^d` nœuds (avec `b` le nombre moyen de voisins par case).

Le BFS bidirectionnel explore deux disques de rayon `d/2` chacun. Cela donne `2 × b^(d/2)` nœuds - soit une **réduction exponentielle** par rapport au BFS classique.

**Exemple concret sur ce labyrinthe :** si la sortie est à 30 cases, BFS classique peut explorer jusqu'à `4^30` nœuds dans le pire cas. BFS bidirectionnel n'en explore que `2 × 4^15` - environ **32 000 fois moins**.

### Ce qu'il conserve

- Reste entièrement **non informé** : aucune heuristique spatiale, aucune connaissance de la géométrie.
- **Garantit le chemin le plus court** en nombre de pas (comme BFS classique).
- **Complet** : trouvera toujours la solution si elle existe.

### Petite contrainte

Il faut connaître le nœud d'arrivée à l'avance pour lancer le BFS depuis E - ce qui est toujours le cas dans notre labyrinthe puisque la case `E` est fixe et connue dès le départ.

---

> **En résumé :** pour rester dans le domaine des recherches non informées tout en étant plus efficace que BFS, DFS et UCS V1, le BFS bidirectionnel est la meilleure proposition. Il exploite simplement le fait qu'on connaît les deux extrémités du chemin pour diviser l'effort d'exploration par un facteur exponentiel.
