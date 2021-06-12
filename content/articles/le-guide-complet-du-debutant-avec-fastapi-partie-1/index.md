---
title: "Le guide complet du débutant avec FastAPI - Partie 1"
date: 2021-06-11T19:33:20+01:00
draft: true
toc: true
---

## Pourquoi FastAPI ?

Commençons par le commencement.

FastAPI est un framework de __développement Web__ écrit en Python. Il est très polyvalent et va permettre de développer :
- Des __sites internet « classiques »__ avec juste du contenu
- Des __sites dynamiques__ avec formulaires et gestion des utilisateurs
- La partie « cachée » des __applications mobiles iOS ou Android__, que l'on appelle une __API__. Vous entendrez aussi parler de __backend__, ce n'est pas tout à fait la même chose, mais on va faire comme-ci pour l'instant.
- Des applications en __temps réel__ comme peuvent l'être les applications de discussion, les cours de la bourse, etc. Vous entendrez souvent parler de __WebSocket__ pour ces applications, on y reviendra.
- Les interfaces d'accès des applications de __Machine Learning__ et d'__Intelligence Artificielle__ qui sont majoritairement développées en Python.

L'idée d'utiliser un framework va être de __ne pas réinventer la roue__ et de se baser sur ce que d'autres personnes talentueuses ont fait avant nous pour résoudre les problèmes classiques des applications web : gestion de la base de données, des URL, de la sécurité des formulaires, des sessions, etc.

FastAPI est un framework assez __récent__ puisque sa première version date de __décembre 2018__. C'est d'ailleurs pour cela que je le choisis maintenant pour la majorité de mes projets : il se base sur une __version récente de Python (minimum 3.6)__ et en tire tous les bénéfices que nous verrons un peu plus tard (simplicité et rapidité notamment).

Il est notamment utilisé dans de __grosses entreprises__ comme [__Microsoft__](https://github.com/tiangolo/fastapi/pull/26#issuecomment-463768795), [__Uber__](https://eng.uber.com/ludwig-v0-2/) ou encore [__Netflix__](https://netflixtechblog.com/introducing-dispatch-da4b8a2a8072).
