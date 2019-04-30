Projektpfad: /web/_project.yaml book_path: /web/resources/_book.yaml description: Dies ist die Seitenbeschreibung im Kopf.

{# wf_updated_on: 2017-12-06 #} {# wf_published_on: 2016-09-13 #} {# wf_blink_components: Nicht zutreffend #}

# Artikel schreiben {: .page-title}

{% include "web / _shared / contributors / petelepage.html"%}

Dies ist der Intro-Absatz. Es ist das Äquivalent des alten `introduction` yaml-Attributs, aber anstatt in der YAML-Frontsache zu leben, lebt es jetzt im Dokument.

Gehen Sie folgendermaßen vor, um einen neuen Artikel zu schreiben. Oder erstellen Sie einfach eine Kopie dieser Markdown-Datei, legen Sie sie im gewünschten Verzeichnis ab und bearbeiten Sie sie.

## YAML-Hauptsache

Alle Dokumente müssen über ein Minimum an YAML-Themen `_project.yaml` , einschließlich eines Links zu `_project.yaml` und eines zu `_book.yaml` .

```
project_path: /web/_project.yaml
book_path: /web/section/_book.yaml
```

Hinweis: Wenn der Pfad zum Projekt oder Buch nicht gefunden werden kann, enthält die Seite nicht die linke Navigation und die oberen Registerkarten werden nicht richtig hervorgehoben.

### Beschreibung (optional)

Sie können eine Seitenbeschreibung in der YAML-Frontsache angeben, die als `meta` Beschreibung für die Seite verwendet wird. Die Beschreibung sollte kurz sein (<450 Zeichen) und nur eine kurze Zusammenfassung der Seite enthalten.

```
description: Lorem ipsum
```

Achtung: Fügen Sie keine <code> -Blöcke (oder `) in das Beschreibungsfeld ein.

### Andere YAML-Attribute

Weitere [Informationen](/web/resources/yaml-and-attr-reference) zu allen YAML-Front-Matter-Attributen und anderen Attributen, die Sie verwenden können oder sollten, finden Sie in der YAML-Übersicht über die vordere Materie und Attribute.

## Seitentitel (erforderlich)

Der `.page-title` wird durch das erste H1-ähnliche Tag mit der `.page-title` Klasse definiert. Zum Beispiel:

<pre class="prettyprint">num Artikel schreiben {: .page-title}</pre>

Achtung: Seitentitel sollten keine Markdown- oder HTML-Tags enthalten.

## Autor und Übersetzer Namensnennung

Um eine Autor- oder Übersetzerzuordnung hinzuzufügen, verwenden Sie:

<pre class="prettyprint">{% include "web / _shared / contributors / petelepage.html"%}</pre>

## Schreiben Sie Ihren Inhalt

Als Nächstes müssen Sie Ihren Inhalt hinzufügen. Ausführliche Informationen zu den Stilen, die Sie verwenden können, und darüber, wie Sie das Aussehen hübsch gestalten können, finden Sie im Handbuch zum [Schreibstil](/style/) und zum [Markdown-](markdown-syntax) Verfahren.

## Artikel zum Buch hinzufügen

Damit Ihr Artikel in der entsprechenden Navigation `_book.yaml` `_toc.yaml` Datei `_book.yaml` oder `_toc.yaml` aktualisieren. Jeder Abschnitt (Updates, Shows, Grundlagen) hat sein eigenes `_book.yaml` und einen Link zu einzelnen `_toc.yaml` Dateien. Sie möchten Ihren Artikel höchstwahrscheinlich einer der `_toc.yaml` Dateien `_toc.yaml` .

## Testen und senden Sie Ihre PR

Wenn Sie fertig sind, führen Sie `gulp test` , um sicherzustellen, dass keine Probleme mit Ihrem Inhalt vorliegen, und senden Sie dann Ihre Pull-Anfrage.
