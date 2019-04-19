Projektpfad: /web/fundamentals/_project.yaml book_path: /web/fundamentals/_book.yaml Beschreibung: Media Source Extensions (MSE) ist eine JavaScript-API, mit der Sie Streams zur Wiedergabe aus Audio- oder Videosegmenten erstellen können.

{# wf_published_on: 2017-02-08 #} {# wf_updated_on: 2018-09-20 #} {# wf_blink_components: Internes> Media #}

# Medienquellenerweiterungen {: .page-title}

{% include "web / _shared / contributors / josephmedley.html"%} {% include "web / _shared / contributors / beaufortfrancois.html"%}

[Media Source Extensions (MSE)](https://www.w3.org/TR/media-source/) ist eine JavaScript-API, mit der Sie Streams für die Wiedergabe aus Audio- oder Videosegmenten erstellen können. Obwohl in diesem Artikel nicht behandelt, ist ein Verständnis von MSE erforderlich, wenn Sie Videos in Ihre Website einbetten möchten, die beispielsweise Folgendes ausführen:

- Adaptives Streaming ist eine weitere Möglichkeit, sich an die Gerätefunktionen und Netzwerkbedingungen anzupassen
- Adaptives Spleißen, z. B. Einfügen von Anzeigen
- Zeitverschiebung
- Kontrolle der Leistung und Downloadgröße

<figure id="basic-mse" class="attempt-right">
  <img src="../../../../en/fundamentals/media/mse/images/basic-mse-flow.png" alt="Basic MSE data flow">
  <figcaption><b>Abbildung 1</b> : Grundlegender MSE-Datenfluss</figcaption>
</figure>

Man kann sich MSE fast als Kette vorstellen. Wie in der Abbildung dargestellt, befinden sich zwischen der heruntergeladenen Datei und den Medienelementen mehrere Ebenen.

- Ein `<audio>` oder `<video>` -Element zum Abspielen der Medien.
- Eine `MediaSource` Instanz mit einem `SourceBuffer` zum `SourceBuffer` des `SourceBuffer` .
- Ein Aufruf von `fetch()` oder XHR zum Abrufen von Mediendaten in einem `Response` Objekt.
- Ein Aufruf von `Response.arrayBuffer()` zum `MediaSource.SourceBuffer` von `MediaSource.SourceBuffer` .

<div class="clearfix"></div>

In der Praxis sieht die Kette so aus:

```
var vidElement = document.querySelector('video');

if (window.MediaSource) {
  var mediaSource = new MediaSource();
  vidElement.src = URL.createObjectURL(mediaSource);
  mediaSource.addEventListener('sourceopen', sourceOpen);
} else {
  console.log("The Media Source Extensions API is not supported.")
}

function sourceOpen(e) {
  URL.revokeObjectURL(vidElement.src);
  var mime = 'video/webm; codecs="opus, vp09.00.10.08"';
  var mediaSource = e.target;
  var sourceBuffer = mediaSource.addSourceBuffer(mime);
  var videoUrl = 'droid.webm';
  fetch(videoUrl)
    .then(function(response) {
      return response.arrayBuffer();
    })
    .then(function(arrayBuffer) {
      sourceBuffer.addEventListener('updateend', function(e) {
        if (!sourceBuffer.updating && mediaSource.readyState === 'open') {
          mediaSource.endOfStream();
        }
      });
      sourceBuffer.appendBuffer(arrayBuffer);
    });
}
```

Wenn Sie aus den bisherigen Erklärungen herausfinden können, können Sie jetzt aufhören zu lesen. Wenn Sie eine ausführlichere Erklärung wünschen, lesen Sie bitte weiter. Ich werde durch diese Kette gehen, indem ich ein einfaches MSE-Beispiel aufbaue. Jeder der Erstellungsschritte fügt dem vorherigen Schritt Code hinzu.

### Ein Hinweis zur Klarheit

Kann dieser Artikel alles über das Abspielen von Medien auf einer Webseite erfahren? Nein, es soll Ihnen nur helfen, komplizierteren Code zu verstehen, den Sie möglicherweise an anderer Stelle finden. Aus Gründen der Klarheit vereinfacht und schließt dieses Dokument viele Dinge aus. Wir glauben, dass wir damit fertig werden können, da wir auch die Verwendung einer Bibliothek wie [dem Shaka Player von Google](https://shaka-player-demo.appspot.com/demo/) empfehlen. Ich werde überall auffallen, wo ich bewusst vereinfache.

### Ein paar Dinge nicht abgedeckt

Hier, in keiner bestimmten Reihenfolge, gibt es einige Dinge, die ich nicht behandeln werde.

- Wiedergabesteuerung. Diese erhalten wir kostenlos, indem wir die HTML5-Elemente `<audio>` und `<video>` verwenden.
- Fehlerbehandlung.

### Für den Einsatz in Produktionsumgebungen

Hier sind einige Dinge, die ich bei einer produktiven Verwendung von MSE-bezogenen APIs empfehlen würde:

- Bevor Sie Aufrufe für diese APIs durchführen, müssen Sie alle Fehlerereignisse oder API-Ausnahmen behandeln und `HTMLMediaElement.readyState` und `MediaSource.readyState` überprüfen. Diese Werte können sich ändern, bevor zugehörige Ereignisse ausgeliefert werden.
- `appendBuffer()` Sie sicher, dass die vorherigen `appendBuffer()` und `remove()` Aufrufe nicht noch ausgeführt werden, indem Sie den booleschen Wert `SourceBuffer.updating` überprüfen, bevor Sie den `SourceBuffer` - `mode` , `timestampOffset` , `appendWindowStart` , `appendWindowEnd` oder den Aufruf von `appendBuffer()` oder `remove()` des `SourceBuffer` .
- `SourceBuffer` für alle `SourceBuffer` Instanzen, die zu Ihrer `MediaSource` hinzugefügt werden, sicher, dass keiner ihrer `updating` wahr ist, bevor Sie `MediaSource.endOfStream()` aufrufen oder die `MediaSource.duration` aktualisieren.
- Wenn der `MediaSource.readyState` Wert `ended` , führen Aufrufe wie `appendBuffer()` und `remove()` oder das Setzen von `SourceBuffer.mode` oder `SourceBuffer.timestampOffset` dazu, dass dieser Wert `open` . Das bedeutet, dass Sie darauf vorbereitet sein sollten, mehrere `sourceopen` Ereignisse zu verarbeiten.
- Bei der Verarbeitung von `HTMLMediaElement error` kann der Inhalt von [`MediaError.message`](https://googlechrome.github.io/samples/media/error-message.html) hilfreich sein, um die Hauptursache des Fehlers zu ermitteln, insbesondere für Fehler, die in Testumgebungen schwer reproduzierbar sind.

## Hängen Sie eine MediaSource-Instanz an ein Medienelement an

Wie bei vielen anderen Dingen in der Webentwicklung fängt man mit der Featureerkennung an. Als nächstes erhalten Sie ein Medienelement, entweder ein `<audio>` oder ein `<video>` -Element. Erstellen Sie schließlich eine Instanz von `MediaSource` . Es wird in eine URL umgewandelt und an das Quellattribut des Medienelements übergeben.

```
var vidElement = document.querySelector('video');

if (window.MediaSource) {
  var mediaSource = new MediaSource();
  vidElement.src = URL.createObjectURL(mediaSource);
  // Is the MediaSource instance ready?
} else {
  console.log("The Media Source Extensions API is not supported.")
}
```

Hinweis: Jedes unvollständige Codebeispiel enthält einen Kommentar, der Sie darauf hinweist, was ich im nächsten Schritt hinzufügen werde. Im obigen Beispiel lautet dieser Kommentar: 'Ist die MediaSource-Instanz bereit?', Die dem Titel des nächsten Abschnitts entspricht.

<figure id="src-as-blob">
  <img src="../../../../en/fundamentals/media/mse/images/media-url.png" alt="A source attribute as a blob">
  <figcaption><b>Abbildung 1</b> : Ein Quellattribut als Blob</figcaption>
</figure>

<div class="clearfix"></div>

Dass ein `MediaSource` Objekt an ein `src` Attribut übergeben werden kann, erscheint möglicherweise etwas seltsam. Sie sind normalerweise Streicher, aber [sie können auch Blobs sein](https://www.w3.org/TR/FileAPI/#url) . Wenn Sie eine Seite mit eingebetteten Medien untersuchen und deren Medienelement untersuchen, werden Sie sehen, was ich meine.

### Ist die MediaSource-Instanz bereit?

`URL.createObjectURL()` ist selbst synchron; Der Anhang wird jedoch asynchron verarbeitet. Dies führt zu einer kurzen Verzögerung, bevor Sie etwas mit der `MediaSource` Instanz tun können. Glücklicherweise gibt es Möglichkeiten, dies zu testen. Am einfachsten ist dies mit einer `MediaSource` Eigenschaft namens `readyState` . Die `readyState` Eigenschaft beschreibt die Beziehung zwischen einer `MediaSource` Instanz und einem `readyState` . Es kann einen der folgenden Werte annehmen:

- `closed` - Die `MediaSource` Instanz ist keinem `MediaSource` .
- `open` - Die `MediaSource` Instanz ist an ein `MediaSource` angehängt und kann Daten empfangen oder empfängt Daten.
- `ended` - Die `MediaSource` Instanz ist an ein `MediaSource` angehängt, und alle Daten wurden an dieses Element übergeben.

Das direkte Abfragen dieser Optionen kann sich negativ auf die Leistung auswirken. Glücklicherweise `readyState` `MediaSource` auch Ereignisse aus, wenn sich `readyState` ändert, insbesondere `sourceopen` , `sourceclosed` , `sourceended` . Für das Beispiel, das ich `sourceopen` , verwende ich das `sourceopen` , um mir zu sagen, wann das Video `sourceopen` und zwischengespeichert werden muss.


<pre class="prettyprint">var vidElement = document.querySelector ('video'); if (window.MediaSource) {var mediaSource = neue MediaSource (); vidElement.src = URL.createObjectURL (mediaSource); <strong>mediaSource.addEventListener ('sourceopen', sourceOpen);</strong> } else {console.log ("Die Media Source Extensions-API wird nicht unterstützt.")} <strong>Funktion sourceOpen (e) {URL.revokeObjectURL (vidElement.src); // Erstellen Sie einen SourceBuffer und rufen Sie die Mediendatei ab. }</strong></pre>  


Notice that I've also called `revokeObjectURL()`. I know this seems premature,
but I can do this any time after the media element's `src` attribute is
connected to a `MediaSource` instance. Calling this method doesn't destroy any
objects. It *does* allow the platform to handle garbage collection at an
appropriate time, which is why I'm calling it immediately.

## Erstellen Sie einen SourceBuffer

Jetzt ist es an der Zeit, den `SourceBuffer` zu erstellen. `SourceBuffer` handelt es sich um das Objekt, das die Daten zwischen Medienquellen und `SourceBuffer` . Ein `SourceBuffer` muss spezifisch für den Typ der Mediendatei sein, die Sie laden.

In der Praxis können Sie dies tun, indem Sie `addSourceBuffer()` mit dem entsprechenden Wert aufrufen. Beachten Sie, dass der Mime-Typ-String im folgenden Beispiel einen Mime-Typ und *zwei* Codecs enthält. Dies ist eine Mime-Zeichenfolge für eine Videodatei, es werden jedoch separate Codecs für die Video- und Audioteile der Datei verwendet.

Version 1 der MSE-Spezifikation ermöglicht es Benutzeragenten, zu unterscheiden, ob ein Mime-Typ und ein Codec erforderlich sind. Einige Benutzeragenten erfordern nicht den Mime-Typ. Einige Benutzerprogramme, z. B. Chrome, benötigen einen Codec für MIME-Typen, die ihre Codecs nicht selbst beschreiben. Anstatt zu versuchen, das alles zu klären, ist es besser, beides einzubeziehen.

Hinweis: Der Einfachheit halber zeigt das Beispiel nur ein einzelnes Mediensegment. In der Praxis ist MSE jedoch nur für Szenarien mit mehreren Segmenten sinnvoll.

<pre class="prettyprint">var vidElement = document.querySelector ('video'); if (window.MediaSource) {var mediaSource = neue MediaSource (); vidElement.src = URL.createObjectURL (mediaSource); mediaSource.addEventListener ('sourceopen', sourceOpen); } else {console.log ("Die Media Source Extensions-API wird nicht unterstützt.")} Funktion sourceOpen (e) {URL.revokeObjectURL (vidElement.src); <strong>var mime = 'video / webm; codecs = "opus, vp09.00.10.08" '; // e.target verweist auf die mediaSource-Instanz. // Speichern Sie es in einer Variablen, damit es in einem Abschluss verwendet werden kann. var mediaSource = e.target; var sourceBuffer = mediaSource.addSourceBuffer (mime); // Video abrufen und verarbeiten.</strong> }</pre>

## Holen Sie sich die Mediendatei

Wenn Sie eine Internetsuche nach MSE-Beispielen durchführen, finden Sie viele, die Mediendateien mit XHR abrufen. Um mehr auf dem [neuesten Stand](https://developer.mozilla.org/en-US/docs/Web/API/GlobalFetch) zu sein, werde ich die [Fetch-](https://developer.mozilla.org/en-US/docs/Web/API/GlobalFetch) API und das [Promise verwenden, das](/web/fundamentals/getting-started/primers/promises) sie zurückgibt. Wenn Sie versuchen, dies in Safari zu tun, funktioniert es nicht ohne `fetch()` Polyfill.

Hinweis: Um die Dinge auf den Bildschirm zu bringen, werde ich von hier bis zum Ende nur einen Teil des Beispiels zeigen, das wir erstellen. Wenn Sie es im Kontext sehen möchten, [springen Sie zum Ende](#the_final_version) .

<pre class="prettyprint">Funktion sourceOpen (e) {URL.revokeObjectURL (vidElement.src); var mime = 'video / webm; codecs = "opus, vp09.00.10.08" '; var mediaSource = e.target; var sourceBuffer = mediaSource.addSourceBuffer (mime); var videoUrl = 'droid.webm'; <strong>fetch (videoUrl) .then (Funktion (Antwort) {// Antwortobjekt bearbeiten.});</strong> }</pre>

Ein Player mit Produktionsqualität verfügt über dieselbe Datei in mehreren Versionen, um verschiedene Browser zu unterstützen. Es könnte separate Dateien für Audio und Video verwenden, um die Auswahl von Audio anhand der Spracheinstellungen zu ermöglichen.

Real-World-Code hätte auch mehrere Kopien von Mediendateien mit unterschiedlichen Auflösungen, so dass er sich an verschiedene Gerätefunktionen und Netzwerkbedingungen anpassen könnte. Eine solche Anwendung kann Videos in Abschnitten entweder über Bereichsanfragen oder Segmente laden und wiedergeben. Dies ermöglicht die Anpassung an die Netzwerkbedingungen *während der Wiedergabe von Medien* . Möglicherweise haben Sie die Begriffe DASH oder HLS gehört. Dies sind zwei Methoden, um dies zu erreichen. Eine vollständige Diskussion dieses Themas geht über den Rahmen dieser Einführung hinaus.

## Verarbeiten Sie das Antwortobjekt

Der Code sieht fast fertig aus, aber die Medien werden nicht abgespielt. Wir müssen Mediendaten vom `Response` Objekt in den `SourceBuffer` .

`ArrayBuffer` Daten vom `ArrayBuffer` an die `MediaSource` Instanz übergeben, `ArrayBuffer` ein `ArrayBuffer` vom `ArrayBuffer` und an den `SourceBuffer` . Beginnen Sie mit dem Aufruf von `response.arrayBuffer()` , der ein Versprechen an den Puffer zurückgibt. In meinem Code habe ich dieses Versprechen an eine zweite `then()` Klausel übergeben, wo ich es an den `SourceBuffer` .

<pre class="prettyprint">Funktion sourceOpen (e) {URL.revokeObjectURL (vidElement.src); var mime = 'video / webm; codecs = "opus, vp09.00.10.08" '; var mediaSource = e.target; var sourceBuffer = mediaSource.addSourceBuffer (mime); var videoUrl = 'droid.webm'; fetch (videoUrl) .then (Funktion (Antwort) { <strong>return response.arrayBuffer ();</strong> }) <strong>.then (Funktion (arrayBuffer) {sourceBuffer.appendBuffer (arrayBuffer);});</strong> }</pre>

#### EndOfStream () aufrufen

Rufen Sie `MediaSource.endOfStream()` nachdem alle `ArrayBuffers` angehängt wurden und keine weiteren Mediendaten erwartet werden. Dies wird sich ändern `MediaSource.readyState` zu `ended` und das Feuer `sourceended` Ereignis.

<pre class="prettyprint">Funktion sourceOpen (e) {URL.revokeObjectURL (vidElement.src); var mime = 'video / webm; codecs = "opus, vp09.00.10.08" '; var mediaSource = e.target; var sourceBuffer = mediaSource.addSourceBuffer (mime); var videoUrl = 'droid.webm'; fetch (videoUrl) .then (Funktion (Antwort) {return response.arrayBuffer ();}) .then (Funktion (arrayBuffer) { <strong>sourceBuffer.addEventListener ('updateend', Funktion (e) {if (! sourceBuffer.updating && mediaSource .readyState === 'open') {mediaSource.endOfStream ();}};</strong> sourceBuffer.appendBuffer (arrayBuffer);}); }</pre>

## Die endgültige Version

Hier ist das vollständige Codebeispiel. Ich hoffe, Sie haben etwas über Media Source Extensions gelernt.

```
var vidElement = document.querySelector('video');

if (window.MediaSource) {
  var mediaSource = new MediaSource();
  vidElement.src = URL.createObjectURL(mediaSource);
  mediaSource.addEventListener('sourceopen', sourceOpen);
} else {
  console.log("The Media Source Extensions API is not supported.")
}

function sourceOpen(e) {
  URL.revokeObjectURL(vidElement.src);
  var mime = 'video/webm; codecs="opus, vp09.00.10.08"';
  var mediaSource = e.target;
  var sourceBuffer = mediaSource.addSourceBuffer(mime);
  var videoUrl = 'droid.webm';
  fetch(videoUrl)
    .then(function(response) {
      return response.arrayBuffer();
    })
    .then(function(arrayBuffer) {
      sourceBuffer.addEventListener('updateend', function(e) {
        if (!sourceBuffer.updating && mediaSource.readyState === 'open') {
          mediaSource.endOfStream();
        }
      });
      sourceBuffer.appendBuffer(arrayBuffer);
    });
}
```

## Feedback Feedback }

{% include "web / _shared / hilfreich.html"%}
