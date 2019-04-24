project_path: /web/fundamentals/_project.yaml book_path: /web/fundamentals/_book.yaml description: Media Source Extensions (MSE) est une API JavaScript qui vous permet de créer des flux à lire à partir de segments audio ou vidéo.

{# wf_published_on: 2017-02-08 #} {# wf_updated_on: 2018-09-20 #} {# wf_blink_components: Internals> Médias #}

# Extensions de la source multimédia {: .page-title}

{% include "web / _shared / contributors / josephmedley.html"%} {% include "web / _shared / contributors / beaufortfrancois.html"%}

[Media Source Extensions (MSE)](https://www.w3.org/TR/media-source/) est une API JavaScript qui vous permet de créer des flux pour la lecture à partir de segments audio ou vidéo. Bien que cela ne soit pas abordé dans cet article, la compréhension de MSE est nécessaire si vous souhaitez intégrer des vidéos sur votre site présentant des actions telles que:

- Diffusion en continu adaptative, une autre façon de s’adapter aux capacités du périphérique et aux conditions du réseau
- Épissage adaptatif, tel que l'insertion d'annonce
- Décalage dans le temps
- Contrôle des performances et de la taille du téléchargement

<figure id="basic-mse" class="attempt-right">
  <img src="../../../../en/fundamentals/media/mse/images/basic-mse-flow.png" alt="Basic MSE data flow">
  <figcaption><b>Figure 1</b> : Flux de données MSE de base</figcaption>
</figure>

Vous pouvez presque penser à MSE comme une chaîne. Comme illustré sur la figure, plusieurs couches se situent entre le fichier téléchargé et les éléments multimédias.

- Un élément `<audio>` ou `<video>` pour lire le média.
- Une instance `MediaSource` avec un `SourceBuffer` pour alimenter l'élément multimédia.
- Un appel `fetch()` ou XHR pour récupérer des données multimédia dans un objet `Response` .
- Un appel à `Response.arrayBuffer()` pour alimenter `MediaSource.SourceBuffer` .

<div class="clearfix"></div>

En pratique, la chaîne ressemble à ceci:

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

Si vous pouvez vous débrouiller avec les explications, n'hésitez pas à arrêter de lire maintenant. Si vous souhaitez une explication plus détaillée, continuez à lire. Je vais parcourir cette chaîne en construisant un exemple de base de MSE. Chacune des étapes de construction ajoutera du code à l'étape précédente.

### Une note sur la clarté

Cet article vous dira-t-il tout ce que vous devez savoir sur la lecture de contenu multimédia sur une page Web? Non, il est uniquement destiné à vous aider à comprendre le code plus compliqué que vous pourriez trouver ailleurs. Par souci de clarté, ce document simplifie et exclut beaucoup de choses. Nous pensons pouvoir nous en sortir, car nous vous recommandons également d'utiliser une bibliothèque telle que [Shaka Player de Google](https://shaka-player-demo.appspot.com/demo/) . Je noterai tout au long où je simplifie délibérément.

### Quelques choses non couvertes

Voici, sans ordre particulier, quelques points que je ne traiterai pas.

- Commandes de lecture. Nous les obtenons gratuitement en utilisant les éléments HTML5 `<audio>` et `<video>` .
- La gestion des erreurs.

### Pour une utilisation dans des environnements de production

Voici quelques recommandations relatives à l'utilisation d'API associées à MSE en production:

- Avant de passer des appels sur ces API, gérez les événements d'erreur ou les exceptions d'API et vérifiez `HTMLMediaElement.readyState` et `MediaSource.readyState` . Ces valeurs peuvent changer avant que les événements associés ne soient livrés.
- Assurez - vous que précédente `appendBuffer()` et `remove()` les appels ne sont pas encore en cours en cochant la `SourceBuffer.updating` valeur booléenne avant la mise à jour du `SourceBuffer` de » `mode` , `timestampOffset` , `appendWindowStart` , `appendWindowEnd` , ou en appelant `appendBuffer()` ou `remove()` sur le `SourceBuffer` .
- Pour toutes les instances `SourceBuffer` ajoutées à votre `MediaSource` , assurez-vous qu'aucune de leurs valeurs de `updating` n'est vraie avant d'appeler `MediaSource.endOfStream()` ou de mettre à jour le `MediaSource.duration` .
- Si `MediaSource.readyState` valeur est `ended` , les appels comme `appendBuffer()` et `remove()` , ou la mise en `SourceBuffer.mode` ou `SourceBuffer.timestampOffset` entraînera cette valeur à la transition vers `open` . Cela signifie que vous devez être prêt à gérer plusieurs événements `sourceopen` .
- Lors du traitement `HTMLMediaElement error` événements d' `HTMLMediaElement error` , le contenu de [`MediaError.message`](https://googlechrome.github.io/samples/media/error-message.html) peut être utile pour déterminer la cause première de l'échec, notamment pour les erreurs difficiles à reproduire dans les environnements de test.

## Attacher une instance MediaSource à un élément multimédia

Comme avec beaucoup de choses dans le développement web ces jours-ci, vous commencez avec la détection de fonctionnalités. Ensuite, obtenez un élément multimédia, élément `<audio>` ou `<video>` . Enfin, créez une instance de `MediaSource` . Il est transformé en une URL et transmis à l'attribut source de l'élément multimédia.

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

Remarque: chaque exemple de code incomplet contient un commentaire qui vous donne une idée de ce que je vais ajouter à l'étape suivante. Dans l'exemple ci-dessus, ce commentaire indique: "L'instance MediaSource est-elle prête?", Ce qui correspond au titre de la section suivante.

<figure id="src-as-blob">
  <img src="../../../../en/fundamentals/media/mse/images/media-url.png" alt="A source attribute as a blob">
  <figcaption><b>Figure 1</b> : un attribut source sous forme de blob</figcaption>
</figure>

<div class="clearfix"></div>

Qu'un objet `MediaSource` puisse être transmis à un attribut `src` peut sembler un peu étrange. Ce sont généralement des chaînes, mais [elles peuvent aussi être des blobs](https://www.w3.org/TR/FileAPI/#url) . Si vous inspectez une page avec un média intégré et examinez son élément multimédia, vous verrez ce que je veux dire.

### L'instance MediaSource est-elle prête?

`URL.createObjectURL()` est lui-même synchrone; Cependant, il traite la pièce jointe de manière asynchrone. Cela provoque un léger délai avant que vous puissiez faire quoi que ce soit avec l'instance `MediaSource` . Heureusement, il existe des moyens de tester cela. Le moyen le plus simple consiste à `MediaSource` une propriété `readyState` appelée `readyState` . La propriété `readyState` décrit la relation entre une instance de `MediaSource` et un élément multimédia. Il peut avoir l'une des valeurs suivantes:

- `closed` - L'instance `MediaSource` n'est pas attachée à un élément multimédia.
- `open` - L'instance `MediaSource` est attachée à un élément de média et est prête à recevoir des données ou reçoit des données.
- `ended` - L'instance `MediaSource` est attachée à un élément de média et toutes ses données ont été transmises à cet élément.

L'interrogation directe de ces options peut avoir un impact négatif sur les performances. Heureusement, `MediaSource` déclenche également des événements lorsque `readyState` modifié, en particulier `sourceopen` , `sourceclosed` , `sourceended` . Pour l'exemple que je construis, je vais utiliser l'événement `sourceopen` pour me dire quand `sourceopen` et mettre en mémoire tampon la vidéo.


<pre class="prettyprint">var vidElement = document.querySelector ('video'); if (window.MediaSource) {var mediaSource = new MediaSource (); vidElement.src = URL.createObjectURL (mediaSource); <strong>mediaSource.addEventListener ('sourceopen', sourceOpen);</strong> } else {console.log ("L'API Media Source Extensions n'est pas prise en charge.")} <strong>function sourceOpen (e) {URL.revokeObjectURL (vidElement.src); // Crée un SourceBuffer et récupère le fichier multimédia. }</strong></pre>  


Notez que j'ai aussi appelé `revokeObjectURL()` . Je sais que cela semble prématuré, mais je peux le faire à tout moment une fois l'attribut `src` l'élément multimédia connecté à une instance de `MediaSource` . L'appel de cette méthode ne détruit aucun objet. Cela *permet* à la plate-forme de gérer le ramassage des ordures à un moment opportun, c'est pourquoi je l'appelle immédiatement.

## Créer un SourceBuffer

Il est maintenant temps de créer le `SourceBuffer` , l’objet qui effectue réellement le travail de transfert de données entre des sources de média et des éléments de média. Un `SourceBuffer` doit être spécifique au type de fichier multimédia que vous chargez.

En pratique, vous pouvez le faire en appelant `addSourceBuffer()` avec la valeur appropriée. Notez que dans l'exemple ci-dessous, la chaîne de type mime contient un type mime et *deux* codecs. Il s'agit d'une chaîne mime pour un fichier vidéo, mais elle utilise des codecs distincts pour les parties vidéo et audio du fichier.

La version 1 de la spécification MSE permet aux agents d'utilisateurs de choisir entre un type mime et un codec. Certains agents utilisateurs ne nécessitent pas, mais n'autorisent que le type mime. Certains agents d'utilisateur, Chrome par exemple, nécessitent un codec pour les types mime qui ne décrivent pas eux-mêmes leurs codecs. Plutôt que d'essayer de régler tout cela, il vaut mieux inclure les deux.

Remarque: par souci de simplicité, l'exemple ne montre qu'un seul segment de média, mais dans la pratique, MSE n'a de sens que pour les scénarios comportant plusieurs segments.

<pre class="prettyprint">var vidElement = document.querySelector ('video'); if (window.MediaSource) {var mediaSource = new MediaSource (); vidElement.src = URL.createObjectURL (mediaSource); mediaSource.addEventListener ('sourceopen', sourceOpen); } else {console.log ("L'API Media Source Extensions n'est pas prise en charge.")} function sourceOpen (e) {URL.revokeObjectURL (vidElement.src); <strong>var mime = 'video / webm; codecs = "opus, vp09.00.10.08" '; // e.target fait référence à l'instance mediaSource. // Stocke-le dans une variable pour pouvoir l'utiliser dans une fermeture. var mediaSource = e.target; var sourceBuffer = mediaSource.addSourceBuffer (mime); // Récupère et traite la vidéo.</strong> }</pre>

## Obtenir le fichier multimédia

Si vous recherchez des exemples MSE sur Internet, vous en trouverez de nombreuses qui permettent de récupérer des fichiers multimédias à l'aide de XHR. Pour être plus en pointe, je vais utiliser l’API [Fetch](https://developer.mozilla.org/en-US/docs/Web/API/GlobalFetch) et la [promesse](/web/fundamentals/getting-started/primers/promises) qu’elle renvoie. Si vous essayez de faire cela dans Safari, cela ne fonctionnera pas sans un polyfill `fetch()` .

Remarque: pour que les choses s’ajustent à l’écran, d’ici la fin, je ne montrerai qu’une partie de l’exemple que nous développons. Si vous voulez le voir en contexte, [sautez à la fin](#the_final_version) .

<pre class="prettyprint">fonction sourceOpen (e) {URL.revokeObjectURL (vidElement.src); var mime = 'video / webm; codecs = "opus, vp09.00.10.08" '; var mediaSource = e.target; var sourceBuffer = mediaSource.addSourceBuffer (mime); var videoUrl = 'droid.webm'; <strong>fetch (videoUrl) .then (function (response) {// Traite l'objet de réponse.});</strong> }</pre>

Un lecteur de qualité de production aurait le même fichier dans plusieurs versions pour prendre en charge différents navigateurs. Il pourrait utiliser des fichiers audio et vidéo séparés pour permettre la sélection de l'audio en fonction des paramètres de langue.

Le code du monde réel comporterait également plusieurs copies de fichiers multimédias à différentes résolutions, ce qui lui permettrait de s’adapter à différentes capacités de périphérique et conditions de réseau. Une telle application est capable de charger et de lire des vidéos en morceaux, en utilisant des requêtes de plage ou des segments. Cela permet une adaptation aux conditions du réseau *pendant la lecture du média* . Vous avez peut-être entendu les termes DASH ou HLS, qui sont deux méthodes pour y parvenir. Une discussion complète sur ce sujet dépasse le cadre de cette introduction.

## Traiter l'objet de réponse

Le code semble presque terminé, mais le média ne joue pas. Nous devons extraire les données multimédia de l'objet `Response` vers le `SourceBuffer` .

Le moyen typique de transmettre des données de l'objet réponse à l'instance `MediaSource` consiste à obtenir un `ArrayBuffer` partir de l'objet réponse et à le transmettre à `SourceBuffer` . Commencez par appeler `response.arrayBuffer()` , qui renvoie une promesse au tampon. Dans mon code, j'ai passé cette promesse à une seconde clause `then()` où je l' `SourceBuffer` au `SourceBuffer` .

<pre class="prettyprint">fonction sourceOpen (e) {URL.revokeObjectURL (vidElement.src); var mime = 'video / webm; codecs = "opus, vp09.00.10.08" '; var mediaSource = e.target; var sourceBuffer = mediaSource.addSourceBuffer (mime); var videoUrl = 'droid.webm'; chercher (videoUrl) .then (function (response) { <strong>return response.arrayBuffer ();</strong> }) <strong>.then (function (arrayBuffer) {sourceBuffer.appendBuffer (arrayBuffer);});</strong> }</pre>

#### Appelez endOfStream ()

Une fois tous les `ArrayBuffers` ajoutés et qu'aucune donnée multimédia supplémentaire n'est attendue, appelez `MediaSource.endOfStream()` . Cela modifiera `MediaSource.readyState` en `ended` et déclenchera l'événement `sourceended` .

<pre class="prettyprint">fonction sourceOpen (e) {URL.revokeObjectURL (vidElement.src); var mime = 'video / webm; codecs = "opus, vp09.00.10.08" '; var mediaSource = e.target; var sourceBuffer = mediaSource.addSourceBuffer (mime); var videoUrl = 'droid.webm'; fetch (videoUrl) .then (function (response) {return response.arrayBuffer ();}) .then (function (arrayBuffer) { <strong>sourceBuffer.addEventListener ('updateend', fonction (e) {if (! sourceBuffer.updating && mediaSource .readyState === 'open') {mediaSource.endOfStream ();}});</strong> sourceBuffer.appendBuffer (arrayBuffer);}); }</pre>

## La version finale

Voici l'exemple de code complet. J'espère que vous avez appris quelque chose sur les extensions de source média.

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

## Commentaires {: #feedback}

{% include "web / _shared / utile.html"%}
