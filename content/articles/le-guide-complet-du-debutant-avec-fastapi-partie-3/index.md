---
title: "Le guide complet du débutant avec FastAPI - Partie 3 : réorganisation du code, tests automatisés"
date: 2021-06-25T19:33:20+01:00
tags:
  - python
  - framework
  - fastapi
  - web
toc: true
---

## Restructuration du code

Jusqu'ici nous avons placé tout notre code dans le même fichier `main.py`. Même si nous pourrions continuer comme cela, il est souvent préférable de séparer son code dans des fichiers et des modules différents. Cela va nous aider à nous y retrouver et va encourager le fait de séparer les responsabilités/préoccupations ([Separation of concerns](https://en.wikipedia.org/wiki/Separation_of_concerns) en anglais). Lors de la [partie 2](http://localhost:1313/articles/le-guide-complet-du-debutant-avec-fastapi-partie-2/) nous avions déjà posé quelques bases en créant des répertoires pour les modèles, les templates, le core, etc. Il est maintenant temps d'aller plus loin et de les utiliser à bon escient.

### Les modèles

Créez un fichier `article.py` dans le répertoire précédemment créé à l'emplacement `app/models`.

Mettez-y le contenu de votre modèle qui se trouvait précédemment dans `main.py`


```python
# app/models/article.py

from tortoise import fields
from tortoise.models import Model


class Article(Model):

    id = fields.IntField(pk=True)

    title = fields.TextField()
    content = fields.TextField()

    created_at = fields.DatetimeField(auto_now_add=True)
    updated_at = fields.DatetimeField(auto_now=True)

    def __str__(self):
        return self.title

```

N'oubliez pas de supprimer les lignes correspondantes dans `main.py`. Il naut faut maintenant importer ce nouveau fichier dans `main.py`.

```python
from app.models.article import Article
```

L'entête de votre fichier `main.py` devrait ressembler à cela maintenant :


```python
# app/main.py

from fastapi import FastAPI, Request
from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates
from tortoise.contrib.fastapi import register_tortoise
from app.models.article import Article

# … reste du fichier
```

Pour être sûr que vous n'avez rien cassé, vous pouvez aller vérifier que [la page listant vos articles](http://localhost:8000/articles) fonctionne toujours correctement.

### Les vues

Maintenant que nous avons rangé notre modèle à sa place, nous allons faire de même avec les vues, aka les fonctions qui construisent le résultat envoyé au navigateur, que ça soit du HTML ou du JSON.

Pour pouvoir mettre les vues autre part que dans le fichier `main.py`, FastAPI utilise le concept de __routeur__. Ce routeur va faire le lien entre votre app principale FastAPI et des vues placées dans des modules/fichiers Python.

Dans votre répertoire `app/views`, créez un fichier nommé `article.py` qui contiendra vos vues. Placez-y le code suivant :

```python
# app/views/article.py

from fastapi import APIRouter
from fastapi import Request
from app.models.article import Article

from app.main import app
from app.main import templates

articles_views = APIRouter()
```

Ici nous préparons les imports dont nous allons avoir besoin et créons un objet `articles_views` de type `APIRouter` qui va nous permettre d'y attacher nos vues. Copiez-collez vos routes du fichier `main.py` vers `app/views/article.py`, ainsi que la déclaration de la variable `templates`. La seule chose que vous devriez avoir à changer est le décorateur situé avant chaque fonction. Au lieu d'appeler `app.get` vous devrez maintenant appeler `articles_views.get` par exemple.

Voic le contenu complet du fichier `app/views/article.py` :

```python
from fastapi import APIRouter
from fastapi import Request
from fastapi.templating import Jinja2Templates
from app.models.article import Article

articles_views = APIRouter()

templates = Jinja2Templates(directory="app/templates")


@articles_views.get("/articles/create", include_in_schema=False)
async def articles_create(request: Request):

    article = await Article.create(
        title="Mon titre de test",
        content="Un peu de contenu<br />avec deux lignes"
    )

    return templates.TemplateResponse(
        "articles_create.html",
        {
            "request": request,
            "article": article
        })


@articles_views.get("/articles", include_in_schema=False)
async def articles_list(request: Request):

    articles = await Article.all().order_by('created_at')

    return templates.TemplateResponse(
        "articles_list.html",
        {
            "request": request,
            "articles": articles
        })


@articles_views.get("/api/articles")
async def api_articles_list():

    articles = await Article.all().order_by('created_at')

    return articles


@articles_views.get("/", include_in_schema=False)
async def root(request: Request):

    return templates.TemplateResponse(
        "home.html",
        {
            "request": request
        })
```

Nous allons maintenant devoir faire le ménage dans `main.py` et appeler la fonction `app.include_router` pour inclure le routeur que nous venons de déclarer. Voici à quoi devrait ressembler votre fichier `main.py` :

```python
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles
from tortoise.contrib.fastapi import register_tortoise
from app.views.article import articles_views

app = FastAPI()

app.mount("/public", StaticFiles(directory="public"), name="public")

register_tortoise(
    app,
    db_url="sqlite://db.sqlite3",
    modules={"models": ["app.models.article"]},
    generate_schemas=True,
    add_exception_handlers=True,
)

app.include_router(
    articles_views,
    tags=["Articles"])
```
