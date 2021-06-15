---
title: "Le guide complet du débutant avec FastAPI - Partie 2"
date: 2021-06-11T19:33:20+01:00
draft: true
tags:
  - python
  - framework
  - fastapi
  - web
toc: true
---

## Projet : une newsletter à la Substack

J'ai toujours aimé apprendre par l'exemple et ce guide ne dérogera pas à la règle. Nous allons prendre comme prétexte la création d'un projet pour apprendre à nous servir de FastAPI. Nous allons développer une application de [publication de contenu/newsletter à la Substack](https://substack.com/).

Les principales fonctionnalités que nous développerons :
- Création d'articles
- Envoi des articles par email
- Gestion des utilisateurs avec inscription et authentification

Plein de bonus possibles :
- Gestion du multilingue
- Traduction automatique des articles
- Commentaires sur les articles
- Conversion du contenu en audio

## Structure de notre projet FastAPI

À la différence de beaucoup de Framework, FastAPI n'impose __aucune structure de répertoires__ ou de fichiers pour pouvoir fonctionner. Quelques __conventions__ se dégagent cependant parmi tous les projets disponibles. Voici celle que nous allons adopter :

```
fastapi-beginners-guide/   <-- répertoire racine de notre projet
├── app/                   <-- répertoire contenant le code Python
│   ├── core/              <-- fichiers partagés (config, exceptions, …)
│   │   └── __init__.py
│   ├── __init__.py
│   ├── main.py            <-- point d'entrée de notre programme FastAPI
│   ├── templates/         <-- fichiers html/jinja
│   │   └── __init__.py
│   ├── tests/             <-- fichiers html/jinja
│   │   └── __init__.py
│   └── views/             <-- fonctions gérant les requêtes HTTP
│       └── __init__.py
├── public/                <-- fichiers CSS, Javascript et fichiers statiques
└── venv/                  <-- environnement virtuel créé à la partie 1
```

Créez une structure de répertoire identique à celle ci-dessus. Vous ne devriez pas avoir à créer le répertoire `venv` puisque il a du être créé automatiquement suite à la [partie 1](/articles/le-guide-complet-du-debutant-avec-fastapi-partie-1/). Les fichiers `__init__.py` sont des fichiers vides nécessaires pour que Python puisse considérer vos répertoires comme des packages.

Copie/collez le code de la partie 1 dans le fichier `app/main.py` :

```python
# app/main.py
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def root():
    return {"message": "Hello World"}
```

Déplacez vous à la racine du projet puis activez l'environnement virtuel :

```
$ source ./venv/bin/activate
```

Assurez-vous ensuite que vous pouvez lancer `uvicorn` avec la commande suivante :


```
(venv) $ uvicorn app.main:app --reload
```

Le type de résultat attendu :

```
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [10884] using watchgod
INFO:     Started server process [10886]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

Notez la différence avec la commande de la [partie 1](/articles/le-guide-complet-du-debutant-avec-fastapi-partie-1/) : nous avons rajouté un `app.` devant le `main`. Celui-ci correspond au répertoire `app/` que nous venons de créer dans lequel se situe notre fichier `main.py`. Je rappelle que le `app` situé après le `:` est le nom de l'objet FastAPI qui a été créé dans notre fichier `main.py`.

## Première page HTML

Nous allons maintenant créer la première page de notre « site web » avec de l'HTML et du CSS.

Copiez collez le code ci-dessous dans votre fichier `app/main.py`.

```python
# app/main.py
from fastapi import FastAPI, Request
from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates

app = FastAPI()

app.mount("/static", StaticFiles(directory="public"), name="public")

templates = Jinja2Templates(directory="app/templates")


@app.get("/")
async def root(request: Request):
    return templates.TemplateResponse("home.html", {"request": request})
```

Décortiquons ce que nous venons de faire.

```python
app.mount("/static", StaticFiles(directory="public"), name="public")
```

Nous « _montons_ » (`app.mount`) une route qui va répondre à l'URL `/static` et qui servira, sous cette adresse, les fichiers que nous mettrons dans le répertoire `public/` précédemment créé (`directory="public"`). Nous nommons cette route `public` (`name="public"`), car nous aurons besoin de l'appeler par son nom pour nous en servir un peu plus loin. Toto aurait aussi fonctionné comme nom, mais c'est moins parlant 😉

__En résumé__ : Si nous plaçons un fichier nommé `styles.css` dans notre répertoire `public/`, cette route va nous permettre d'y accéder par l'adresse `http://localhost:8000/public/styles.css`.

```python
templates = Jinja2Templates(directory="app/templates")
```

Nous créons un objet (`templates`) qui va nous permettre de créer de l'HTML avec le moteur de templates [Jinja2](https://jinja2docs.readthedocs.io/en/stable/). Cet objet ira chercher ses templates dans le répertoire que nous avons créé, `app/templates/`.


```python
@app.get("/")
async def root(request: Request):
    return templates.TemplateResponse("home.html", {"request": request})
```

Nous avons ensuite modifié notre méthode `root` pour qu'elle récupère l'objet `request`. Cet objet est fourni par FastAPI (plus précisement par [starlette](https://www.starlette.io), Framework sur lequel FastAPI est basé) et permet d'obtenir des informations sur la requête : l'URL d'origine, les cookies, les headers, etc. La documentation complète est disponible ici : https://www.starlette.io/requests/. Nous y reviendrons.

Au lieu de retourner un simple dictionnaire Python comme précédemment, notre méthode renvoie maintenant un objet `TemplateResponse`. C'est un objet qui va être en charge de créer du HTML à partir d'un template, `home.html` dans notre cas. Il ira chercher ce template dans le répertoire que nous avons spécifié plus haut avec `directory="app/templates"`.

Notez le dictionnaire Python passé deuxième paramètre, `{"request": request}`. Ce dictionnaire va nous permettre de passer des données de notre vue à notre template. Dans ce cas précis, nous passons la requête en paramètre. La clé du dictionnaire `"request"` est le nom que nous voulons donner à notre valeur dans le template et la valeur du dictionnaire `request` est la valeur que nous souhaitons passer au template sous ce nom.

Il nous faut ensuite créer le contenu du template dans `app/templates/home.html`. Copiez-collez le code suivant :

```django
<html>
<head>
    <title>Home</title>
    <link href="{{ url_for('public', path='/styles.css') }}" rel="stylesheet">
</head>
<body>
    <h1>Home title</h1>
    <p>Current url: <strong>{{ request.url }}</strong></p>
</body>
</html>
```

Outre le code HTML classique, la première ligne intéressante est la suivante :
```django
<link href="{{ url_for('public', path='/styles.css') }}" rel="stylesheet">
```

Nous construisons un lien dynamique grâce à la fonction `url_for`. Cette fonction prend en paramètres le nom de la route, `public` dans notre cas (le nom que nous avions donné plus haut, lors du `app.mount`) et l'emplacement du fichier, `path='/styles.css'`, qu'il nous restera à créer. Avec Jinja, tout ce qui est entre `{{` et `}}` sera affiché dans le code HTML.

L'autre ligne intéressante est celle-ci :
```django
<p>Current url: <strong>{{ request.url }}</strong></p>
```
Ici nous nous servons de l'objet `request` de starlette que nous avions passé à notre template (`templates.TemplateResponse("home.html", {"request": request})`) pour afficher l'url courante.

Il nous reste à créer le fichier `styles.css` dans le répertoire `public/` et d'y mettre le contenu suivant par exemple :

```css
h1 {
  color: #fe5186;
}
```

Rechargez votre page d'accueil à l'adresse [http://localhost:8000/](http://localhost:8000/) et vous devriez obtenir le résultat suivant :

![Page d'accueil Jinja](images/home.png)
