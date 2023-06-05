# Exercices de la formation Gitlab CI

Ce repo contient différent exercices conçus pour accompagner la formation Gitlab CI des 06/06/23 et 13/06/23.

## Scénario

Les différents exercices vont vous placer dans la peau d'un ingénieur DevOps chargé de mettre en place une chaine de CICD à l'aide de Gitlab CICD pour une petite application (désolé si ça vous rappelle des souvenirs du travail 😇).

## Application

L'application utilisée pour les exercices est une API toute simple faite en Python à l'aide de Flask.

Le code source de l'application est disponible dans le dosser `accounting/`.

Quelques tests sont présents et utilisent le framework `pytest`. Ils sont disponibles dans le dossier `tests/`.

### Préparation de l'environnement

Un fichier `requirements.txt` est disponible pour créer un environnement virtuel Python. Les dépendances sont valides pour Python >= 3.8.

Pour préparer l'environnement :

```shell
python3 -m venv .venv

# Ne pas oublier d'activer l'environnement virtuel (ie. utiliser le Python de cet environnement au lieu de celui de la machine)
source .venv/bin/activate

pip install --upgrade pip
pip install -r requirements.txt
```

### Lancement de l'application

Pour lancer l'application Flask :

```shell
source .venv/bin/activate
export FLASK_APP=accounting.app:setup_app
flask run --debug
```

## Exercice 1

Vous avez été missionné pour mettre en place de manière progressive une chaine de CICD pour une application.

La première tâche qu'on vous a confié est de faire en sorte que l'ensemble des tests unitaires soit lancés de manière automatique à chaque fois que quelqu'un push un changement sur le projet.

### Notes pour la mise en place de la CICD

Les tests utilisent le framework `pytest` pour fonctionner.

Pour les faire lancer, il faut dans un premier temps **s'assurer d'être dans un environnement virtuel Python activé** avec les **dépendances du projet d'installées**. Voir section `Préparation de l'environnement` ci-dessus pour les commandes nécessaires à l'installation.

Pour lancer les tests, **il suffit de lancer la commande `pytest` à la racine du projet** :

```shell
> pytest

=================== test session starts ===================
platform linux -- Python 3.11.3, pytest-7.3.1, pluggy-1.0.0
rootdir: ...
configfile: pytest.ini
plugins: cov-4.1.0
collected 4 items

tests/test_expenses.py ..                            [ 50%]
tests/test_incomes.py ..                             [100%]

=================== 4 passed in 0.17s =====================
```

Une **image Docker avec Python >= 3.8 est nécessaire** afin de faire tourner le job.

Vous pouvez prendre n'importe laquelle, les images officielles conviennent parfaitement :
- `python:3.8`
- `python:3.9`
- `python:3.10`
- `python:3.11`

[> Détail d'une solution possible](https://gitlab.com/bastien-antoine/orness/formation-gitlab/exercises/-/tree/ex1-sol)

[> Exercice suivant](https://gitlab.com/bastien-antoine/orness/formation-gitlab/exercises/-/tree/ex2)

### Solution proposée

Une solution possible à la tâche demandée est proposée dans le fichier `.gitlab-ci-solution.yml`

> **Note :** Le fichier est nommé ainsi afin qu'il ne soit pas executé automatiquement par Gitlab lors des différentes actions effectuée sur le projet.

## Exercice 2

La tâche suivante est de mettre en place des contrôles de qualité du code de l'application (aka du lint).

Vous avez décidé d'utiliser [`flake8`](https://flake8.pycqa.org/en/latest/) pour contrôler le code produit.

### Notes pour la mise en place de la CICD

Le job nécessite Python >= 3.8 pour fonctionner. Il faut dans un premier temps **s'assurer d'être dans un environnement virtuel Python activé** avec les **dépendances du projet d'installées**. Voir section `Préparation de l'environnement` ci-dessus pour les commandes nécessaires à l'installation.

`flake8` n'est pas présent dans les dépendances du projet telles qu'elles sont indiquées dans le `requirements.txt`. Il faut donc l'installer au préalable dans le job avant de pouvoir l'utiliser :
```shell
pip install flake8
```

Pour lancer le lint, **il suffit de lancer la commande `flake8` en indiquant le dossier à analyser** :

```shell
> flake8 accounting
accounting/model/income.py:11:80: E501 line too long (81 > 79 characters)
```

Une **image Docker avec Python >= 3.8 est nécessaire** afin de faire tourner le job.

Vous pouvez prendre n'importe laquelle, les images officielles conviennent parfaitement :
- `python:3.8`
- `python:3.9`
- `python:3.10`
- `python:3.11`

> **Note :** Dans l'état actuel du code, le job lançant `flake8` sera probablement en échec, dû à une erreur de mise en forme dans le code. **C'est normal.**

[> Détail d'une solution possible](https://gitlab.com/bastien-antoine/orness/formation-gitlab/exercises/-/tree/ex2-sol)

[> Exercice suivant](https://gitlab.com/bastien-antoine/orness/formation-gitlab/exercises/-/tree/ex3)

### Solution proposée

Une solution possible à la tâche demandée est proposée dans le fichier `.gitlab-ci-solution.yml`

> **Note :** Le fichier est nommé ainsi afin qu'il ne soit pas executé automatiquement par Gitlab lors des différentes actions effectuée sur le projet.

> **Note :** Ce fichier contient aussi une solution possible à l'exercice précédent. Ne pas hésiter à adapter la solution à ce que vous avez produit à l'exercice précédent.

> **Note :** flake8 doit normalement remonter une erreur au sein du fichier `accounting/model/income.py`. Il peut être nécessaire de corriger cette erreur afin que ce job ne bloque plus les pipelines futures.
>
> ```diff
>   # accounting/model/income.py
>   class Income(Transaction):
>       def __init__(self, description: str, amount: str):
> -         super(Income, self).__init__(description, amount, TransactionType.INCOME)
> +         super(Income, self).__init__(
>               description,
>               amount,
>               TransactionType.INCOME,
>           )
> ```

## Exercice 3

Après quelques temps d'utilisation de la pipeline que vous avez mis en place, vous avez eu quelques retours qui nécessitent que vous l'adaptiez un peu :
1. Le job de lint du code qui est bloquant est un peu contraignant
2. C'est assez lourd de lancer tous les tests à chaque action effectuées sur le projet.

Voici ce qui a été décidé :
1. Ces deux jobs ne doivent être **lancés que quand la pipeline est lancée pour une branche différente de la branche principale et qui a au moins une merge request d'ouverte associée**. Il a été estimé que le process de vérification du code et de protection des différentes branches autorisait de ne lancer les tests que sur les branches avant de merger sur la branche principale.
2. Le job de lint du code **ne doit plus être bloquant**

### Notes pour la mise en place de la CICD

Pour faire en sorte qu'un job ne soit ajouté que lorsque sa pipeline est associée à une branche donnée, il faut **ajouter une règle sur la valeur de la variable `CI_COMMIT_BRANCH`**. On peut récupérer le **nom de la branche par défaut dans la variable `CI_DEFAULT_BRANCH`**.

Il est également possible de récupérer une **liste des merge requests associées à la branche en question via la variable `CI_OPEN_MERGE_REQUESTS`**.

Un job non bloquant est un job dont le résultat (succès ou échec) n'influe pas sur le résultat final de la pipeline. Autrement dit c'est un **job qui est autorisé à terminer en échec**.

Tous ces paramètres se **contrôlent via les [`rules`](https://docs.gitlab.com/ee/ci/yaml/#rules)** du job.

[> Détail d'une solution possible](https://gitlab.com/bastien-antoine/orness/formation-gitlab/exercises/-/tree/ex3-sol)

[> Exercice suivant](https://gitlab.com/bastien-antoine/orness/formation-gitlab/exercises/-/tree/ex4)

### Solution proposée

Une solution possible à la tâche demandée est proposée dans le fichier `.gitlab-ci-solution.yml`

> **Note :** Le fichier est nommé ainsi afin qu'il ne soit pas executé automatiquement par Gitlab lors des différentes actions effectuée sur le projet.

> **Note :** Ce fichier contient aussi une solution possible aux exercices précédents. Ne pas hésiter à adapter la solution à ce que vous avez produit aux exercices précédents.

**Quelques explications :**

La règle, identique au deux jobs, se découpe en 3 parties :
- `$CI_COMMIT_BRANCH`: ici on ne souhaite ajouter le job que dans le cas d'une pipeline lancée pour une branche. La documentation nous indique : *Available in branch pipelines, including pipelines for the default branch. Not available in merge request pipelines or tag pipelines*.
- `$CI_COMMIT_BRANCH != $CI_DEFAULT_BRANCH`: la documentation de la variable précédente nous indique que celle-ci est définie aussi pour les pipelines lancées pour la branche par défaut. Comme on ne souhaite lancer ces jobs que pour les branches autres que celle par défaut, il faut filtrer sur le nom de la branche.
- `$CI_OPEN_MERGE_REQUESTS`: on souhaite de plus ne lancer que les jobs pour les branches qui ont au moins une merge request associée. la documentation de cette variable nous indique: *A comma-separated list of up to four merge requests that use the current branch and project as the merge request source. For example, gitlab-org/gitlab!333,gitlab-org/gitlab-foss!11.*. Ici on est pas intéressé par le(s) merge request(s) associée(s), seulement savoir s'il y en a.

## Exercice 4

Place au déploiement ! Votre application se modernise et il vous faut maintenant la conteneuriser dans une image Docker. On vous a fourni un Dockerfile qui encapsule votre application et ses dépendances, il faut maintenant mettre en place un job qui va builder votre image et la pousser sur une registry.

Les règles pour la mise en place du job de build sont les suivantes:
- Si la **pipeline est lancée sur une branche différente de la branche par défaut, et à laquelle est associée au moins une merge request** d'ouverte, alors il faut **seulement builder l'image, et ne pas la pousser sur la registry**.
- Si la **pipeline est lancée sur la branche par défaut**, il faut **builder l'image et la pousser sur la registry**.

### Notes pour la mise en place de la CICD

Pour effectuer le build de notre image, il nous faut un outil capable de construire des images Docker. Comme notre job tourne déjà au sein d'un conteneur Docker, il nous faut un outil qui soit capable de le faire dans ce contexte.

Plusieurs possibilités :
- [*"Docker-in-docker" (`dind`)*](https://docs.gitlab.com/ee/ci/docker/using_docker_build.html#use-docker-in-docker)
- [kaniko](https://docs.gitlab.com/ee/ci/docker/using_kaniko.html)

Nous allons utiliser la deuxième méthode.

Pour utiliser kaniko au sein d'un job Gitlab CI, il faut **utiliser une image dédiée** :

```yaml
build:
  stage: build
  image:
    name: gcr.io/kaniko-project/executor:v1.9.0-debug
    entrypoint: [""]
  script: ...
```

**La commande à lancer est la suivante** :
```bash
/kaniko/executor
  --context "${CI_PROJECT_DIR}"
  --dockerfile "${CI_PROJECT_DIR}/Dockerfile"
  --destination "${CI_REGISTRY_IMAGE}:${CI_COMMIT_TAG}"
```

> **Note :** cette exercice utilise la fonctionalité de [registry de conteneur intégrée à Gitlab](https://docs.gitlab.com/ee/user/packages/container_registry/index.html). L'offre SaaS communautaire (ie. [gitlab.com](https://gitlab.com) gratuit) a une limite de 5 Go de stockage total par utilisateur. Les images stockées sur la registry de conteneurs comptent dans le calcul de cette limite. **Pensez à supprimer les images si vous atteignez les limites de l'offre**.

Par défaut, kaniko va construire l'image et la pousser vers la registry indiquée. **Si on ne veut pas pousser l'image, il faut rajouter l'option `--no-push`**.

[> Détail d'une solution possible](https://gitlab.com/bastien-antoine/orness/formation-gitlab/exercises/-/tree/ex4-sol)

[> Exercice suivant](https://gitlab.com/bastien-antoine/orness/formation-gitlab/exercises/-/tree/ex5)
