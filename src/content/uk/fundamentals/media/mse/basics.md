project_path: /web/fundamentals/_project.yaml book_path: /web/fundamentals/_book.yaml description: Розширення медіа джерел (MSE) - це JavaScript API, який дозволяє створювати потоки для відтворення з сегментів аудіо чи відео.

{# wf_published_on: 2017-02-08 #} {# wf_updated_on: 2018-09-20 #} {# wf_blink_components: внутрішні> медіа #}

# Розширення медіа-джерела {: .page-title}

{% include "web / _shared / contributors / josephmedley.html"%} {% include "

[Розширення медіа джерел (MSE)](https://www.w3.org/TR/media-source/) - це JavaScript API, який дозволяє створювати потоки для відтворення з сегментів аудіо чи відео. Незважаючи на те, що ця стаття не розглядається, потрібно розуміти MSE, якщо ви хочете вставляти відео на свій сайт, виконуючи такі дії:

- Адаптивне потокове передавання, що є іншим способом пристосування до можливостей пристрою та умов мережі
- Адаптивне сплайсинг, наприклад вставлення оголошення
- Час перемикання
- Контроль продуктивності та розміру завантаження

<figure id="basic-mse" class="attempt-right">
  <img src="../../../../en/fundamentals/media/mse/images/basic-mse-flow.png" alt="Basic MSE data flow">
  <figcaption><b>Рисунок 1</b> : Основний потік даних MSE</figcaption>
</figure>

Можна майже уявити MSE як ланцюжок. Як проілюстровано на малюнку, між завантаженим файлом і елементами медіа є кілька шарів.

- Елемент `<audio>` або `<video>` для відтворення носія.
- Екземпляр `MediaSource` з `SourceBuffer` для подачі медіа-елемента.
- Виклик `fetch()` або XHR для отримання мультимедійних даних у об'єкті `Response` .
- Виклик `Response.arrayBuffer()` для подачі `MediaSource.SourceBuffer` .

<div class="clearfix"></div>

На практиці ланцюг виглядає так:

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

Якщо ви все-таки можете розібратися з поясненнями, не соромтеся зараз читати. Якщо ви хочете отримати більш детальне пояснення, будь ласка, продовжуйте читати. Я збираюся пройти цей ланцюжок, створивши базовий приклад MSE. Кожен з кроків збірки додасть код до попереднього кроку.

### Примітка про ясність

Чи буде ця стаття розповісти вам все, що вам потрібно знати про відтворення медіа на веб-сторінці? Ні, він призначений лише для того, щоб допомогти вам зрозуміти більш складний код, який можна знайти в іншому місці. Заради ясності цей документ спрощує та виключає багато речей. Ми вважаємо, що ми можемо піти з цим, оскільки ми також рекомендуємо використовувати бібліотеку, таку як [Shaka Player від Google](https://shaka-player-demo.appspot.com/demo/) . Відзначу, де я навмисно спрощую.

### Кілька речей не охоплені

Тут, не в особливому порядку, є кілька речей, які я не охоплюю.

- Керування відтворенням. Ми отримуємо їх безкоштовно, використовуючи елементи HTML5 `<audio>` і `<video>` .
- Обробка помилок.

### Для використання у виробничих умовах

Ось деякі речі, які я рекомендував би використовувати при виробництві API, пов'язаних з MSE:

- Перш ніж здійснювати виклики на цих API, `HTMLMediaElement.readyState` будь-які події помилок або виключення API, а також перевірте `HTMLMediaElement.readyState` і `MediaSource.readyState` . Ці значення можуть змінюватись до того, як будуть доставлені асоційовані події.
- Переконайтеся , що попередній `appendBuffer()` і `remove()` дзвінки, крім все ще триває, перевіряючи `SourceBuffer.updating` логічного значення перед оновленням `SourceBuffer` «сек `mode` , `timestampOffset` , `appendWindowStart` , `appendWindowEnd` , або зателефонувавши за телефоном `appendBuffer()` або `remove()` на `SourceBuffer` .
- Для всіх екземплярів `SourceBuffer` доданих до вашого `MediaSource` , переконайтеся, що жодне з їхніх значень `updating` є істинним до виклику `MediaSource.endOfStream()` або оновлення `MediaSource.duration` .
- Якщо значення `MediaSource.readyState` `ended` , виклики, такі як `appendBuffer()` і `remove()` , або налаштування `SourceBuffer.mode` або `SourceBuffer.timestampOffset` призведуть до `open` цього значення переходу. Це означає, що ви повинні бути готові працювати з кількома подіями, що `sourceopen` .
- При обробці `HTMLMediaElement error` , вміст [`MediaError.message`](https://googlechrome.github.io/samples/media/error-message.html) може бути корисним для визначення першопричини збою, особливо для помилок, які важко відтворити в тестових середовищах.

## Приєднайте примірник MediaSource до медіа-елементу

Як і багато інших речей у веб-розробці, ви починаєте з виявлення функцій. Далі отримаємо медіа-елемент, або `<audio>` або `<video>` елемент. Нарешті, створіть екземпляр `MediaSource` . Він перетворюється на URL і передається атрибуту джерела медіа-елементу.

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

Примітка: кожен неповний приклад коду містить коментар, який дає підказку про те, що я додаду на наступному кроці. У наведеному вище прикладі цей коментар говорить: "Чи готовий екземпляр MediaSource?", Який відповідає назві наступного розділу.

<figure id="src-as-blob">
  <img src="../../../../en/fundamentals/media/mse/images/media-url.png" alt="A source attribute as a blob">
  <figcaption><b>Рисунок 1</b> : Атрибут джерела як крапка</figcaption>
</figure>

<div class="clearfix"></div>

Те, що об'єкт `MediaSource` може бути переданий атрибуту `src` може здатися дещо дивним. Вони зазвичай є рядками, але [вони також можуть бути краплями](https://www.w3.org/TR/FileAPI/#url) . Якщо ви перевіряєте сторінку з вбудованими носіями та вивчаєте її медіа-елемент, ви побачите, що я маю на увазі.

### Чи готовий екземпляр MediaSource?

`URL.createObjectURL()` сам по собі є синхронним; однак, він обробляє вкладення асинхронно. Це спричиняє невелику затримку перед тим, як можна зробити що-небудь з екземпляром `MediaSource` . На щастя, є способи перевірити це. Найпростішим способом є властивість `MediaSource` названа `readyState` . `readyState` описує зв'язок між екземпляром `MediaSource` і медіа-елементом. Він може мати одне з таких значень:

- `closed` - Екземпляр `MediaSource` не приєднаний до медіа-елементу.
- `open` - Екземпляр `MediaSource` приєднаний до медіа-елементу і готовий до прийому даних або отримання даних.
- `ended` - примірник `MediaSource` приєднано до медіа-елементу, і всі його дані передано до цього елемента.

Запити цих параметрів можуть негативно вплинути на продуктивність. До щастя, `MediaSource` також генерує події , коли `readyState` зміни, в зокрема `sourceopen` , `sourceclosed` , `sourceended` . Для прикладу, який я `sourceopen` , я збираюся використовувати подію `sourceopen` щоб сказати мені, коли потрібно витягувати і буферизувати відео.


<pre class="prettyprint">var vidElement = document.querySelector ('відео'); if (window.MediaSource) {var mediaSource = новий MediaSource (); vidElement.src = URL.createObjectURL (mediaSource); <strong>mediaSource.addEventListener ('sourceopen', sourceOpen);</strong> } else {console.log ("API розширення джерела медіа не підтримується.")} <strong>функція sourceOpen (e) {URL.revokeObjectURL (vidElement.src); // Створити SourceBuffer і отримати медіа-файл. }</strong></pre>  


Notice that I've also called `revokeObjectURL()`. I know this seems premature,
but I can do this any time after the media element's `src` attribute is
connected to a `MediaSource` instance. Calling this method doesn't destroy any
objects. It *does* allow the platform to handle garbage collection at an
appropriate time, which is why I'm calling it immediately.

## Створити SourceBuffer

Тепер настав час створити `SourceBuffer` , який є об'єктом, який фактично виконує роботу з передачі даних між медіа-джерелами та медіа-елементами. `SourceBuffer` має бути специфічним для типу мультимедійного файлу, який ви завантажуєте.

На практиці це можна зробити, викликавши `addSourceBuffer()` з відповідним значенням. Зверніть увагу, що в прикладі нижче рядок типу mime містить тип MIME і *два* кодеки. Це рядок MIME для відеофайлу, але він використовує окремі кодеки для відео та аудіо-частин файлу.

Версія 1 специфікації MSE дозволяє користувальницьким агентам відрізнятися щодо того, чи потрібно вимагати як тип MIME, так і кодек. Деякі агенти користувача не вимагають, але дозволяють лише тип MIME. Деякі агенти користувача, наприклад Chrome, вимагають кодек для типів MIME, які не описують їх кодеки. Замість того, щоб намагатися сортувати все це, краще просто включити обидва.

Примітка: Для простоти приклад показує лише один сегмент носія, хоча на практиці MSE має сенс лише для сценаріїв з декількома сегментами.

<pre class="prettyprint">var vidElement = document.querySelector ('відео'); if (window.MediaSource) {var mediaSource = новий MediaSource (); vidElement.src = URL.createObjectURL (mediaSource); mediaSource.addEventListener ('sourceopen', sourceOpen); } else {console.log ("API розширення джерела медіа не підтримується.")} функція sourceOpen (e) {URL.revokeObjectURL (vidElement.src); <strong>var mime = 'відео / webm; codecs = "opus, vp09.00.10.08" '; // e.target відноситься до екземпляра mediaSource. // Зберігаємо його у змінну, щоб її можна було використовувати у закритті. var mediaSource = e.target; var sourceBuffer = mediaSource.addSourceBuffer (mime); // Отримати та обробити відео.</strong> }</pre>

## Отримати медіа-файл

Якщо ви виконуєте пошук в Інтернеті для прикладів MSE, ви знайдете багато, що отримують мультимедійні файли за допомогою XHR. Щоб бути більш передовим, я збираюся використовувати API [Fetch](https://developer.mozilla.org/en-US/docs/Web/API/GlobalFetch) і [Promise, яку](/web/fundamentals/getting-started/primers/promises) він повертає. Якщо ви намагаєтеся це зробити в Safari, він не буде працювати без полівзаповнення `fetch()` .

Примітка: Просто щоб допомогти речі потрапити на екран, звідси до кінця я тільки збираюся показати частину прикладу ми будуємо. Якщо ви хочете побачити його в контексті, [перейдіть до кінця](#the_final_version) .

<pre class="prettyprint">функція sourceOpen (e) {URL.revokeObjectURL (vidElement.src); var mime = 'відео / webm; codecs = "opus, vp09.00.10.08" '; var mediaSource = e.target; var sourceBuffer = mediaSource.addSourceBuffer (mime); var videoUrl = 'droid.webm'; <strong>fetch (videoUrl). then (function (response) {// обробляти об'єкт відповіді.});</strong> }</pre>

Програвач якості продукції має один і той же файл у декількох версіях для підтримки різних браузерів. Він може використовувати окремі файли для аудіо та відео, щоб дозволити вибір аудіо на основі мовних налаштувань.

Реальний світовий код також матиме кілька копій мультимедійних файлів з різною роздільною здатністю, щоб він міг адаптуватися до різних можливостей пристроїв і мережевих умов. Таке додаток здатне завантажувати та відтворювати відео в шматках, використовуючи запити діапазонів або сегменти. Це дозволяє пристосовуватися до умов мережі *під час відтворення медіа* . Можливо, ви чули терміни DASH або HLS, які є двома методами досягнення цього. Повне обговорення цієї теми виходить за рамки цього вступу.

## Обробляти об'єкт відповіді

Код виглядає майже готовим, але ЗМІ не грають. Нам потрібно отримати медіа-дані від об'єкта `Response` до `SourceBuffer` .

Типовим способом передачі даних від об'єкта відповіді до екземпляра `MediaSource` є отримання `ArrayBuffer` з об'єкта відповіді і передачі його в `SourceBuffer` . Почніть з виклику `response.arrayBuffer()` , який повертає обіцянку в буфер. У моєму коді я передав цю обіцянку другому `then()` `SourceBuffer` де я додаю його до `SourceBuffer` .

<pre class="prettyprint">функція sourceOpen (e) {URL.revokeObjectURL (vidElement.src); var mime = 'відео / webm; codecs = "opus, vp09.00.10.08" '; var mediaSource = e.target; var sourceBuffer = mediaSource.addSourceBuffer (mime); var videoUrl = 'droid.webm'; fetch (videoUrl). then (функція (відповідь) { <strong>return response.arrayBuffer ();</strong> }) <strong>.</strong> }</pre>

#### Виклик endOfStream ()

Після того, як всі `ArrayBuffers` додаються, і не очікується подальших даних, викличте `MediaSource.endOfStream()` . Це змінить `MediaSource.readyState` на `ended` і звільнить `sourceended` подію.

<pre class="prettyprint">функція sourceOpen (e) {URL.revokeObjectURL (vidElement.src); var mime = 'відео / webm; codecs = "opus, vp09.00.10.08" '; var mediaSource = e.target; var sourceBuffer = mediaSource.addSourceBuffer (mime); var videoUrl = 'droid.webm'; fetch (videoUrl). then (функція (відповідь) {return response.arrayBuffer ();}). then (function (arrayBuffer) { <strong>sourceBuffer.addEventListener ('updateend', функція (e) {if (! sourceBuffer.updating && mediaSource .readyState === 'open') {mediaSource.endOfStream ();}</strong> }); }</pre>

## Остаточний варіант

Ось повний приклад коду. Сподіваюся, ви дізналися щось про розширення Media Source.

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

## Зворотній зв'язок {: #feedback}

{% include "web / _shared / helpful.html"%}
