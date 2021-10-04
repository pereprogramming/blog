---
title: "Le guide complet du débutant avec FastAPI - Partie 4 : création, récupération et suppression des articles"
date: 2021-09-29T09:33:20+01:00
tags:
  - python
  - framework
  - fastapi
  - web
  - fastapi-tutorial
toc: true
---

## Manipulation des articles

Dans cette partie, nous allons mettre en place l'API pour nous permettre de __récupérer__ les articles, les __créer__ de manière dynamique, les __modifier__ et les __supprimer__.Vous verrez souvent cela résumé par l'acronyme __CRUD__ : **C**reate, **R**retrieve, **U**pdate et **D**elete.

### Utilisation de Pydantic

Comme je vous l'avais mentionné dans l'instroduction, FastAPI a la particularité d'utiliser au maximum les [types de python](https://fastapi.tiangolo.com/python-types/) et plus particulièrement une librairie appelée [Pydantic](https://pydantic-docs.helpmanual.io/).

Pydantic se définit comme ceci :

> pydantic enforces type hints at runtime, and provides user friendly errors when data is invalid.

Pydantic va se servir des types que nous allons définir sur nos objets pour automatiquement faire plein de choses : valider les données que les utilisateurs nous envoient (s'il sait qu'on attend un entier et qu'on nous donne un string, il enverra une erreur lisible par un humain) et, entres autres, permettre de générer automatiquement la documentation de notre API.

Nous allons donc commencer par définir ce que l'on appelle un _schéma_ pydantic pour notre classe `Article`. Commencez par créer le répertoire `app/schemas` :

```bash
mkdir app/schemas
```

Puis ajoutez-y un fichier nommé `article.py` avec le code suivant :

```python
# app/schemas/article.py

from pydantic import BaseModel
from datetime import datetime


class Article(BaseModel):
    id: int
    title: str
    content: str
    updated_at: datetime
    created_at: datetime
```

Vous voyez ici qu'on reproduit la structure de notre modèle article qui se trouve dans `app/models/article.py` presque à l'identique, en spécifiant le type de chaque champ.

> Il existe une [fonctionnalité de Tortoise](https://tortoise-orm.readthedocs.io/en/latest/contrib/pydantic.html) qui vous permet de générer le schéma Pydantic directement à partir du modèle. Nous ne l'utiliserons pas dans ce tutorial pour que vous puissiez voir comment cela fonctionne sans _magie_.

Nous pouvons déjà tirer bénéfice de ce nouvel ajout en spécifiant le type de retour que l'on attend dans nos vues. Cela va nous permettre d'avoir une première version de notre documentation. Dans notre fichier `app/views/article.py`, nous allons spécifier le type de retour de la fonction `api_articles_list`.

Commencez par y ajouter les deux imports suivants :

```python
# app/views/article.py
# … début des imports
from app.schemas.article import Article as ArticleSchema
from typing import List

# … reste du fichier
```

Puis modifiez le décorateur de la fonction comme ceci :

```python
# app/views/article.py
# … début du fichier

@articles_views.get("/api/articles", response_model=List[ArticleSchema])
async def api_articles_list():

    articles = await Article.all().order_by('created_at')

    return articles
```

L'appel à votre API devrait toujours vous retourner la même chose (`http http://localhost:8000/api/articles`), c'est à dire la liste des articles. En revanche, vous avez gagné un bout de documentation gratuite. Rendez-vous sur [http://localhost:8000/docs](http://localhost:8000/docs) et vous devriez avoir quelque chose qui ressemble à cela :

[![Documentation des articles](images/docs_articles_list.png)](images/docs_articles_list.png)

Vous pouvez maintenant voir que l'utilisateur de votre API est au courant du format de votre objet article et des types qu'il contient. Pratique !
