project_path: /web/_project.yaml book_path: /web/resources/_book.yaml description: il s'agit de la description de page placée dans la tête.

{# wf_updated_on: 2017-12-06 #} {# wf_published_on: 2016-09-13 #} {# wf_blink_components: N / A #}

# Écrire un article {: .page-title}

{% include "web / _shared / contributors / petelepage.html"%}

Ceci est le paragraphe d'introduction. C'est l'équivalent de l'ancien attribut d' `introduction` yaml, mais au lieu de vivre à l'avant-plan YAML, il est désormais intégré au document.

Pour écrire un nouvel article, procédez comme suit. Ou tout simplement, faites une copie de ce fichier de démarquage, placez-le dans le répertoire de votre choix et modifiez-le.

## YAML Front Matter

Tous les documents doivent avoir au minimum un ensemble de `_project.yaml` YAML, y compris un lien vers `_project.yaml` et un autre vers `_book.yaml` .

```
project_path: /web/_project.yaml
book_path: /web/section/_book.yaml
```

Remarque: Si le chemin d'accès au projet ou au livre est introuvable, la page n'inclut pas la navigation de gauche et les onglets supérieurs ne sont pas correctement mis en évidence.

### description (facultatif)

Vous pouvez fournir une description de la page dans le préambule YAML utilisée comme `meta` description pour la page. La description doit être courte (<450 caractères) et ne fournir qu'un bref résumé de la page.

```
description: Lorem ipsum
```

Attention: N'incluez pas de blocs <code> (ou `) dans le champ de description.

### Autres attributs YAML

Reportez-vous à la section [Référence de l'attribut YAML et de son attribut](/web/resources/yaml-and-attr-reference) pour connaître l'ensemble de l' [attribut](/web/resources/yaml-and-attr-reference) YAML et les autres attributs que vous pouvez ou devez utiliser.

## Titre de la page (obligatoire)

Le titre de la page est défini par la première balise de type H1 avec la classe `.page-title` . Par exemple:

<pre class="prettyprint">& num; Écrire un article {: .page-title}</pre>

Attention: Les titres de page ne doivent inclure aucune balise HTML ou markdown.

## Attribution des auteurs et des traducteurs

Pour inclure une attribution d'auteur ou de traducteur, utilisez:

<pre class="prettyprint">{% include "web / _shared / contributors / petelepage.html"%}</pre>

## Ecrivez votre contenu

Ensuite, il est temps d'ajouter votre contenu. Reportez-vous au [guide de rédaction](/style/) et au [guide de](/style/) [syntaxe Markdown](markdown-syntax) pour obtenir des détails complets sur les styles que vous pouvez utiliser et sur la manière de rendre les choses plus jolies.

## Ajouter un article au livre

Pour que votre article apparaisse dans la navigation appropriée, vous devez mettre à jour le fichier `_book.yaml` ou `_toc.yaml` . Chaque section (mises à jour, spectacles, principes fondamentaux) a son propre `_book.yaml` et des liens vers des fichiers individuels `_toc.yaml` . Vous voudrez probablement ajouter votre article à l’un des fichiers `_toc.yaml` .

## Testez et soumettez votre PR

Lorsque vous êtes prêt, exécutez `gulp test` pour vous assurer que votre contenu ne pose aucun problème, puis envoyez votre demande d'extraction.
