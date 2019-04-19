project_path: /web/fundamentals/_project.yaml book_path: /web/fundamentals/_book.yaml description: Расширения Media Source Extensions (MSE) - это API-интерфейс JavaScript, который позволяет создавать потоки для воспроизведения из сегментов аудио или видео.

{# wf_published_on: 2017-02-08 #} {# wf_updated_on: 2018-09-20 #} {# wf_blink_components: Internals> Media #}

# Расширения медиаресурсов {: .page-title}

{% include "web / _shared / contributors / josephmedley.html"%} {% include "web / _shared / contributors / beaufortfrancois.html"%}

[Расширения Media Source Extensions (MSE)](https://www.w3.org/TR/media-source/) - это API-интерфейс JavaScript, который позволяет создавать потоки для воспроизведения из сегментов аудио или видео. Хотя это и не рассматривается в этой статье, понимание MSE необходимо, если вы хотите встроить видео на свой сайт, которые выполняют такие вещи, как:

- Адаптивная потоковая передача - это еще один способ адаптации к возможностям устройства и условиям сети.
- Адаптивный сплайсинг, такой как вставка рекламы
- Временной сдвиг
- Контроль производительности и размера загрузки

<figure id="basic-mse" class="attempt-right">
  <img src="../../../../en/fundamentals/media/mse/images/basic-mse-flow.png" alt="Basic MSE data flow">
  <figcaption><b>Рисунок 1</b> : Основной поток данных MSE</figcaption>
</figure>

Вы можете почти думать о MSE как о цепи. Как показано на рисунке, между загруженным файлом и медиа-элементами находятся несколько слоев.

- Элемент `<audio>` или `<video>` для воспроизведения мультимедиа.
- Экземпляр `MediaSource` с `SourceBuffer` для подачи медиа-элемента.
- Вызов `fetch()` или XHR для извлечения мультимедийных данных в объекте `Response` .
- Вызов `Response.arrayBuffer()` для подачи `MediaSource.SourceBuffer` .

<div class="clearfix"></div>

На практике цепочка выглядит так:

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

Если вы можете разобраться с объяснениями, не стесняйтесь прекратить чтение сейчас Если вы хотите более подробное объяснение, пожалуйста, продолжайте читать. Я собираюсь пройти через эту цепочку, создав основной пример MSE. Каждый из шагов сборки будет добавлять код к предыдущему шагу.

### Записка о ясности

Расскажет ли вам эта статья все, что вам нужно знать о воспроизведении мультимедиа на веб-странице? Нет, он предназначен только для того, чтобы помочь вам понять более сложный код, который вы можете найти в другом месте. Для ясности этот документ упрощает и исключает многие вещи. Мы думаем, что можем справиться с этим, потому что мы также рекомендуем использовать такую библиотеку, как [Google Shaka Player](https://shaka-player-demo.appspot.com/demo/) . Я отмечу везде, где я намеренно упрощаю.

### Несколько вещей, не охваченных

Здесь, в произвольном порядке, есть несколько вещей, которые я не буду освещать.

- Элементы управления воспроизведением. Мы получаем их бесплатно благодаря использованию элементов HTML5 `<audio>` и `<video>` .
- Обработка ошибок.

### Для использования в производственных условиях

Вот некоторые вещи, которые я бы порекомендовал при использовании API-интерфейсов, связанных с MSE:

- Перед выполнением вызовов этих API обработайте все события ошибок или исключения API и проверьте `HTMLMediaElement.readyState` и `MediaSource.readyState` . Эти значения могут изменяться до доставки связанных событий.
- Убедитесь , что предыдущий `appendBuffer()` и `remove()` вызовы не все еще продолжается, проверяя `SourceBuffer.updating` логического значения перед обновлением `SourceBuffer` «сек `mode` , `timestampOffset` , `appendWindowStart` , `appendWindowEnd` , или позвонив по телефону `appendBuffer()` или `remove()` на `SourceBuffer` ,
- Для всех экземпляров `SourceBuffer` добавленных к вашему `MediaSource` , убедитесь, что ни одно из их значений `updating` является истинным, перед вызовом `MediaSource.endOfStream()` или обновлением `MediaSource.duration` .
- Если значение `MediaSource.readyState` `ended` , вызовы, такие как `appendBuffer()` и `remove()` , или установка `SourceBuffer.mode` или `SourceBuffer.timestampOffset` вызовут переход этого значения в `open` . Это означает, что вы должны быть готовы к обработке нескольких `sourceopen` событий.
- При обработке `HTMLMediaElement error` содержимое [`MediaError.message`](https://googlechrome.github.io/samples/media/error-message.html) может быть полезно для определения основной причины сбоя, особенно для ошибок, которые трудно воспроизвести в тестовых средах.

## Присоедините экземпляр MediaSource к элементу мультимедиа

Как и многие вещи в веб-разработке в наши дни, вы начинаете с обнаружения функций. Затем получите элемент мультимедиа, элемент `<audio>` или `<video>` . Наконец, создайте экземпляр `MediaSource` . Он превращается в URL и передается в атрибут источника медиа-элемента.

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

Примечание. Каждый пример неполного кода содержит комментарий, который дает подсказку о том, что я добавлю на следующем шаге. В приведенном выше примере этот комментарий говорит: «Готов ли экземпляр MediaSource?», Что соответствует заголовку следующего раздела.

<figure id="src-as-blob">
  <img src="../../../../en/fundamentals/media/mse/images/media-url.png" alt="A source attribute as a blob">
  <figcaption><b>Рисунок 1</b> : атрибут источника в виде большого двоичного объекта</figcaption>
</figure>

<div class="clearfix"></div>

То, что объект `MediaSource` может быть передан атрибуту `src` может показаться странным. Обычно это строки, но [они также могут быть каплями](https://www.w3.org/TR/FileAPI/#url) . Если вы осмотрите страницу со встроенным медиа и изучите ее медиа-элемент, вы поймете, что я имею в виду.

### Готов ли экземпляр MediaSource?

`URL.createObjectURL()` сам по себе является синхронным; однако он обрабатывает вложение асинхронно. Это вызывает небольшую задержку, прежде чем вы сможете что-либо сделать с экземпляром `MediaSource` . К счастью, есть способы проверить это. Самый простой способ - использовать свойство `MediaSource` именем `readyState` . Свойство `readyState` описывает отношение между экземпляром `MediaSource` и медиа-элементом. Может иметь одно из следующих значений:

- `closed` - экземпляр `MediaSource` не присоединен к медиа-элементу.
- `open` - экземпляр `MediaSource` прикреплен к медиа-элементу и готов к приему данных или принимает данные
- `ended` - экземпляр `MediaSource` присоединен к медиа-элементу, и все его данные переданы этому элементу.

Запрос этих параметров напрямую может негативно повлиять на производительность. К счастью, `MediaSource` также `readyState` события при изменении `readyState` , в частности, `sourceopen` , `sourceclosed` , `sourceended` . Для примера, который я создаю, я собираюсь использовать событие `sourceopen` чтобы сообщить мне, когда нужно извлекать и буферизовать видео.


<pre class="prettyprint">var vidElement = document.querySelector ('video'); if (window.MediaSource) {var mediaSource = new MediaSource (); vidElement.src = URL.createObjectURL (mediaSource); <strong>mediaSource.addEventListener ('sourceopen', sourceOpen);</strong> } else {console.log («API расширений мультимедиа не поддерживается.»)} <strong>function sourceOpen (e) {URL.revokeObjectURL (vidElement.src); // Создать SourceBuffer и получить медиа-файл. }</strong></pre>  


Notice that I've also called `revokeObjectURL()`. I know this seems premature,
but I can do this any time after the media element's `src` attribute is
connected to a `MediaSource` instance. Calling this method doesn't destroy any
objects. It *does* allow the platform to handle garbage collection at an
appropriate time, which is why I'm calling it immediately.

## Создать SourceBuffer

Теперь пришло время создать `SourceBuffer` , который является объектом, который фактически выполняет передачу данных между источниками мультимедиа и элементами мультимедиа. `SourceBuffer` должен соответствовать типу медиа-файла, который вы загружаете.

На практике вы можете сделать это, вызвав `addSourceBuffer()` с соответствующим значением. Обратите внимание, что в приведенном ниже примере строка типа MIME содержит тип MIME и *два* кодека. Это строка MIME для видеофайла, но она использует отдельные кодеки для видео и аудио частей файла.

Версия 1 спецификации MSE позволяет агентам пользователей различать, требуется ли тип mime и кодек. Некоторые пользовательские агенты не требуют, но допускают только тип MIME. Некоторым пользовательским агентам, например Chrome, требуется кодек для типов MIME, которые не описывают свои кодеки самостоятельно. Вместо того, чтобы пытаться разобраться во всем этом, лучше просто включить оба.

Примечание. Для простоты в примере показан только один сегмент мультимедиа, хотя на практике MSE имеет смысл только для сценариев с несколькими сегментами.

<pre class="prettyprint">var vidElement = document.querySelector ('video'); if (window.MediaSource) {var mediaSource = new MediaSource (); vidElement.src = URL.createObjectURL (mediaSource); mediaSource.addEventListener ('sourceopen', sourceOpen); } else {console.log («API расширений мультимедиа не поддерживается.»)} function sourceOpen (e) {URL.revokeObjectURL (vidElement.src); <strong>var mime = 'video / webm; codecs = "opus, vp09.00.10.08" '; // e.target ссылается на экземпляр mediaSource. // Сохраняем его в переменной, чтобы его можно было использовать в замыкании. var mediaSource = e.target; var sourceBuffer = mediaSource.addSourceBuffer (mime); // Выбрать и обработать видео.</strong> }</pre>

## Получить медиа файл

Если вы выполните поиск в Интернете примеров MSE, вы найдете множество файлов, которые извлекают мультимедийные файлы с использованием XHR. Чтобы быть более передовыми, я собираюсь использовать [Fetch](https://developer.mozilla.org/en-US/docs/Web/API/GlobalFetch) API и [Promise, который](/web/fundamentals/getting-started/primers/promises) он возвращает. Если вы пытаетесь сделать это в Safari, он не будет работать без полизаполнения `fetch()` .

Примечание: просто чтобы помочь вещам поместиться на экране, отсюда до конца я собираюсь показать только часть примера, который мы строим Если вы хотите увидеть это в контексте, [переходите к концу](#the_final_version) .

<pre class="prettyprint">function sourceOpen (e) {URL.revokeObjectURL (vidElement.src); var mime = 'video / webm; codecs = "opus, vp09.00.10.08" '; var mediaSource = e.target; var sourceBuffer = mediaSource.addSourceBuffer (mime); var videoUrl = 'droid.webm'; <strong>fetch (videoUrl) .then (function (response) {// Обработать объект ответа.});</strong> }</pre>

Плеер производственного качества будет иметь один и тот же файл в нескольких версиях для поддержки разных браузеров. Он может использовать отдельные файлы для аудио и видео, чтобы аудио можно было выбирать в зависимости от языковых настроек.

Реальный код также имеет несколько копий мультимедийных файлов с разным разрешением, чтобы он мог адаптироваться к различным возможностям устройства и условиям сети. Такое приложение может загружать и воспроизводить видео частями, используя запросы диапазона или сегменты. Это позволяет адаптироваться к условиям сети *во время воспроизведения мультимедиа* . Возможно, вы слышали термины DASH или HLS, которые являются двумя способами достижения этой цели. Полное обсуждение этой темы выходит за рамки данного введения.

## Обработать объект ответа

Код выглядит почти готовым, но медиа не воспроизводится. Нам нужно получить медиа-данные из объекта `Response` в `SourceBuffer` .

Типичный способ передачи данных из объекта ответа в экземпляр `MediaSource` - получить `ArrayBuffer` из объекта ответа и передать его в `SourceBuffer` . Начните с вызова `response.arrayBuffer()` , который возвращает обещание в буфер. В моем коде я передал это обещание второму предложению `then()` , где я добавляю его в `SourceBuffer` .

<pre class="prettyprint">function sourceOpen (e) {URL.revokeObjectURL (vidElement.src); var mime = 'video / webm; codecs = "opus, vp09.00.10.08" '; var mediaSource = e.target; var sourceBuffer = mediaSource.addSourceBuffer (mime); var videoUrl = 'droid.webm'; fetch (videoUrl) .then (function (response) { <strong>return response.arrayBuffer ();</strong> }) <strong>.then (function (arrayBuffer) {sourceBuffer.appendBuffer (arrayBuffer);});</strong> }</pre>

#### Вызовите endOfStream ()

После того, как все `ArrayBuffers` добавлены, и больше никаких мультимедийных данных не ожидается, вызовите `MediaSource.endOfStream()` . Это изменит `MediaSource.readyState` на `ended` и `sourceended` событие `sourceended` .

<pre class="prettyprint">function sourceOpen (e) {URL.revokeObjectURL (vidElement.src); var mime = 'video / webm; codecs = "opus, vp09.00.10.08" '; var mediaSource = e.target; var sourceBuffer = mediaSource.addSourceBuffer (mime); var videoUrl = 'droid.webm'; fetch (videoUrl) .then (function (response) {return response.arrayBuffer ();}) .then (function (arrayBuffer) { <strong>sourceBuffer.addEventListener ('updateend', function (e) {if (! sourceBuffer.updating && mediaSource .readyState === 'open') {mediaSource.endOfStream ();}});</strong> sourceBuffer.appendBuffer (arrayBuffer);}); }</pre>

## Финальная версия

Вот полный пример кода. Я надеюсь, что вы узнали что-то о расширениях Media Source.

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

## Обратная связь {: #feedback}

{% include "web / _shared / полезно.html"%}
