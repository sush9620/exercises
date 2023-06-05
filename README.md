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
