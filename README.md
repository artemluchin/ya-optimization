## Оптимизация скорости загрузки страницы сайты lifehacker.ru

Для ускорения загрузки сайта требуется минимизировать:
* число первоочередных ресурсов
* число байтов
* продолжительность обработки

Но, для начала, выясним какие проблемы встречаются при загрузке страницы сайта.

### Основные проблемы при загрузке страницы

Проверив скорость загрузки сайта с помощью нескольких сервисов, получил следующие
данные.

#### HTML файл

Основные проблемы:
* большой html файл (много DOM элементов)
* блоки с основным контентом находятся ниже блоков с боковым меню и рекламой. Это
  влияет на скорость загрузки основного контента.
* html файл не минифицирован

#### CSS файлы

Количество блокирующих css файлов: **21**
```
  https://fonts.googleapis.com/…talic&subset=latin,cyrillic,cyrillic-ext
  https://fonts.googleapis.com/…oboto+Slab:400,700&subset=latin,cyrillic
  https://lifehacker.ru/…ate/static/public/css/main.css?ver=4.5.3
  https://lifehacker.ru/…-source/static/public/main.min.css?ver=2
  https://lifehacker.ru/…gins/lh-send-typo/css/styles.css?ver=1.5
  https://lifehacker.ru/…ider/static/public/all.min.css?ver=1.0.0
  https://lifehacker.ru/…ial-slider/assets/css/style.css?ver=1.10
  https://lifehacker.ru/…ns/lh-social/static/main.min.css?ver=2.9
  https://lifehacker.ru/…/assets/nivo/nivo-lightbox.css?ver=1.6.8
  https://lifehacker.ru/…ivo/themes/default/default.css?ver=1.6.8
  https://lifehacker.ru/…rettyPhoto/css/prettyPhoto.css?ver=4.5.3
  https://lifehacker.ru/…ajax-query-shortcode/style.css?ver=4.5.3
  https://lifehacker.ru/…lifehacker/static/fonts/style.css?ver=13
  https://lifehacker.ru/…ker/static/styles/vendors.min.css?ver=13
  https://lifehacker.ru/…ehacker/static/styles/all.min.css?ver=13
  https://lifehacker.ru/…ugins/lh-widgets/css/widgets.css?ver=1.4
  https://lifehacker.ru/…gets/image-widget/style.css?ver=20140808
  https://lifehacker.ru/…lugins/jetpack/css/jetpack.css?ver=4.1.1
  https://static.hypercomments.com/…widget/hc/2/20160719133038/css/index.css
  https://cdn.teads.tv/media/format/v3/teads-format.css
  https://cdn.teads.tv/media/format/v3/teads-format.css
```

Основные проблемы:
* слишком много css файлов;
* два раза загружается один и тот же файл `teads-format.css`;
* многие файлы имеют всего пару строк (например, `main.min.css`). Из-за этой мелочи
  делается отдельный запрос;
* несколько файлов не минифицировано (даже файл с именем `all.min.css` O_o);
* общий объем css в `307 KB` довольно большой для такого рода страницы;
* не используется 87% css на странице;
* очень много DNS lookups из-за большого кол-ва файлов, находящихся по разным хостам.

#### JS файлы

Количество блокирующих js файлов: **21**
```
https://lifehacker.ru/…-includes/js/jquery/jquery.js?ver=1.12.4
https://lifehacker.ru/…s/jquery/jquery-migrate.min.js?ver=1.4.1
https://lifehacker.ru/…t/plugins/lh-send-typo/js/app.js?ver=1.5
https://lifehacker.ru/…ocial-slider/assets/js/common.js?ver=1.8
https://lifehacker.ru/…tatic/js/adfox.asyn.code.ver3.js?ver=3.0
https://lifehacker.ru/…/plugins/lh-widgets/js/viewed.js?ver=1.6
https://cdn.onthe.io/io.js?V6Mze3eoxmNt
https://lifehacker.ru/…lider/static/public/all.min.js?ver=1.0.0
https://lifehacker.ru/…ins/lh-social/static/main.min.js?ver=2.9
https://vk.com/js/api/share.js
https://lifehacker.ru/…/plugins/lh-views/js/getviews.js?7&ver=1
https://lifehacker.ru/…sets/nivo/nivo-lightbox.min.js?ver=1.6.8
https://lifehacker.ru/…esponsive-lightbox/js/front.js?ver=1.6.8
https://lifehacker.ru/…ry-shortcode/js/masonry.min.js?ver=2.2.2
https://lifehacker.ru/…jax-query-shortcode/js/main.js?ver=2.2.2
https://s0.wp.com/…ontent/js/devicepx-jetpack.js?ver=201631
https://cdnjs.cloudflare.com/…ery.lazy/1.7.0/jquery.lazy.min.js?ver=13
https://cdn.pushwoosh.com/…sh/pushwoosh-web-notifications.js?ver=13
https://lifehacker.ru/…fehacker/static/js/vendors.min.js?ver=13
https://lifehacker.ru/…s/lifehacker/static/js/all.min.js?ver=13
https://lifehacker.ru/…wp-includes/js/wp-embed.min.js?ver=4.5.3
```
Основные проблемы:
* слишком много js файлов;
* несколько файлов не минифицировано;
* большой общий объем js - `341 KB`. Не критично, но думаю можно и меньше;
* некоторые файлы не сжаты;
* очень много DNS lookups из-за большого кол-ва файлов, находящихся по разным хостам.

#### Images

* загружается очень большое количество изображений;
* основное время загрузки страницы занимают изображения!!!;
* некоторые изображения не оптимизированы;
* размер изображений иногда во много раз превышает размеры блока, в который они вписаны;
  Например, `2500х1250` вписывают в `330х165` :clap:.

На странице присутствует три возможных размера изображения (большой `1600х800` или `2500x1250`, 
средний `630х315` и маленький `310х155`). Допустим картинка должна быть маленького
размера. Тогда, по идее, нужно загружать только `pic310x155.jpeg`. Но не тут то было!
Загружаются сразу все размеры для данной картинки. И не понятно почему, поскольку
никаких манипуляций с картинкой не происходит...

Решил разобраться почему грузятся сразу все. Взял элемент, в котором должна быть
маленькая картинка(заменил длинные пути к файлам на короткие для удобства):

```
<div 
    data-src="https://lifehacker.ru/pic-1600x800.jpg"
    data-src-mobile="https://lifehacker.ru/pic-310x155.jpg"
    data-src-medium="https://lifehacker.ru/pic-630x315.jpg"
    data-src-ringo_800=""
    data-src-ringo_1600="https://lifehacker.ru/pic-1600x800.jpg"
    style="opacity: 1; background-image: url("https://lifehacker.ru/pic-310x155.jpg";);"
    class="stripe-item__img-background lazy-load">
</div>
```
Тут видно, что пути ко всем картинкам прописаны в `dataset`.Изначально я думал, что
используется `srcset`, и он почему-то грузит сразу все варианты. Но нет.
Получается, что вызвать загрузку этих файлов можно только используя скрипт. А тут уже
не ясно чем руководствовались разработчики.
Также осталось загадкой назначение атрибута `data-src-ringo_1600`, который содержит в
себе URL к той же самой картинке, как и в `data-src`.

Еще непонятен момент с размером изображения `1600х800`. Специально посмотрел на отображение
сайта с разных устройств. Размер самого большого блока с картинкой не превышает `800px`.
Можно спокойно уменьшать ширину в 2 раза. А если это сделано для DPR 2.0, то как бы `srcset`.

Общий объем всех изображений порядка 15 Mb, в зависимости от текущих статей на сайте. Иногда было и 23 Mb.
Такие объемы 

#### Кеширование

У некоторых файлов указано очень маленькое время жизни:
```
https://www.googletagmanager.com/gtm.js?id=GTM-5NFNJ2 (15 минут)
https://connect.facebook.net/ru_RU/sdk.js (20 минут)
https://apis.google.com/js/platform.js (30 минут)
https://an.yandex.ru/system/context.js (60 минут)
https://cdn.pushwoosh.com/…sh/pushwoosh-web-notifications.js?ver=13 (60 минут)
https://mc.yandex.ru/metrika/watch.js (60 минут)
https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js (60 минут)
https://www.google-analytics.com/analytics.js (2 часа)
https://lifehacker.ru/…lugins/jetpack/css/jetpack.css?ver=4.1.1 (24 часа)
https://lifehacker.ru/…gets/image-widget/style.css?ver=20140808 (24 часа)
https://lifehacker.ru/…ate/static/public/css/main.css?ver=4.5.3 (24 часа)
https://lifehacker.ru/…-source/static/public/main.min.css?ver=2 (24 часа)
https://lifehacker.ru/…gins/lh-send-typo/css/styles.css?ver=1.5 (24 часа)
https://lifehacker.ru/…ider/static/public/all.min.css?ver=1.0.0 (24 часа)
https://lifehacker.ru/…ial-slider/assets/css/style.css?ver=1.10 (24 часа)
https://lifehacker.ru/…ns/lh-social/static/main.min.css?ver=2.9 (24 часа)
https://lifehacker.ru/…ugins/lh-widgets/css/widgets.css?ver=1.4 (24 часа)
https://lifehacker.ru/…/assets/nivo/nivo-lightbox.css?ver=1.6.8 (24 часа)
https://lifehacker.ru/…ivo/themes/default/default.css?ver=1.6.8 (24 часа)
https://lifehacker.ru/…rettyPhoto/css/prettyPhoto.css?ver=4.5.3 (24 часа)
https://lifehacker.ru/…ajax-query-shortcode/style.css?ver=4.5.3 (24 часа)
https://lifehacker.ru/…lifehacker/static/fonts/style.css?ver=13 (24 часа)
https://lifehacker.ru/…ehacker/static/styles/all.min.css?ver=13 (24 часа)
https://lifehacker.ru/…ker/static/styles/vendors.min.css?ver=13 (24 часа)
```

Для файлов, время существования которых истекает менее чем через 24 часа ситуация более
менее ясна. Это внешние файлы. И если знать время его реального обновления, то допускается использовать данное время.

А вот для файлов, у которых время существования 24 часа, следует указывать большее время (минимум неделя, 
согласно рекомендации). Если не ошибаюсь, то тут используются цифровые отпечатки URL.
Но тогда не ясно почему стоит такой маленький срок. Можно смело ставить год.

Для следующих файлов время вообще не указано:
```
https://cdn.teads.tv/media/format.js
https://cdn.teads.tv/media/format/v3/teads-format.css
```
#### Сжатие файлов

Следующие файлы не были сжаты перед отправкой:
```
https://lifehacker.ru/wp-content/themes/lifehacker/static/img/logo.svg
https://lifehacker.ru/wp-content/themes/lifehacker/static/img/category-icons/food.svg
https://lifehacker.ru/wp-content/themes/lifehacker/static/img/category-icons/sport.svg
https://lifehacker.ru/wp-content/themes/lifehacker/static/img/category-icons/teh.svg
https://lifehacker.ru/wp-content/themes/lifehacker/static/img/category-icons/life.svg
https://lifehacker.ru/wp-content/themes/lifehacker/static/img/category-icons/health.svg
https://lifehacker.ru/wp-content/themes/lifehacker/static/img/category-icons/creativity.svg
https://lifehacker.ru/wp-content/themes/lifehacker/static/img/category-icons/work.svg
```


### Рекомендации по уменьшению времени загрузки сайта lifehacker.ru

#### HTML

1. Пересмотреть структуру файла. Сделать его более компактным по возможности.
2. На первое место поставить блоки, содержащие основной контент. Боковое меню, блоки рекламы и т.д.
   следует ставить после, чтобы не мешали рендерингу основного контента.
3. Минифицировать файл, путем удаления пробелов.
4. Сжать gzip'ом перед отправкой. Возможно использовать несколько алгоритмов сжатия.

#### CSS

1. Объединить все стили в один файл.
2. Убрать из файла стили, которые не используются на странице.
3. Минифицировать файл.
4. Для более быстрой загрузки основного контента можно включить критичные стили в html
   в тэг `<style>`. Оставшуюся часть стилей загружать асинхронно (media = 'none').
5. Убрать inline-styles из тегов. 
6. Сжимать загружаемый файл стилей gzip'ом.
7. Кэшировать файл стилей в браузере. Время жизни кэша ставить от недели до года.
8. Использовать CDN для загрузки файла стилей. Ускорит загрузку для удаленных пользователей.

#### JS

1. Объединить все скрипты в один файл.
2. Проанализировать файл на наличие неиспользуемой функциональности и убрать все лишнее, что
   не используется.
3. Минифицировать файл скрипта.
4. Если какие то функции участвуют в отображении основного контента, то можно включить эти функции
   в тег `<script>` в html. Это позволит сразу загружать требуемую функциональность.
5. Поместить все скрипты в конец `<body>`.
6. Добавить тегам `<script>` свойство `async`/`defer` для асинхронной загрузки.
7. Сжимать загружаемый файл скрипта gzip'ом.
8. Кэшировать файл скрипта в браузере. Время жизни кэша ставить от недели до года.
9. Использовать CDN для загрузки файла скрипта. Ускорит загрузку для удаленных пользователей.

#### Images

1. Чтобы избежать скачивания всех размеров изображения, можно использовать
   тег `<picture>` с указанием источников и правил для них, а также можно просто
   указать все виды размеров изображений в атрибуте `scrset`.
2. Использовать сжатие для изображений.
3. Возможно использовать новые форматы для изображений, такие как `JPEG XR` и `WebP`.
4. Показывать изображение в его реальном размере. Если загружать изображение размером `100х100`
   и показывать его в блоке размерами `80х80`, то это приведет к скачиванию лишних байтов.
5. Кэшировать изображения в браузере.
6. По возможности отказаться от .gif, поскольку размер одного такого файла равен десяткам
   изображений в формате .jpeg



## Бонусное задание

В качестве дополнительного сайта я выбрал mts.ru. Поскольку большинство проблем
у лайфхакера и мтс схожи, то я опишу какие проблемы встретил на сайте mts.ru, а
в рекомендациях по оптимизации укажу некоторые отличия от рекомендаций лайфхакеру.

### Основные проблемы при загрузке страницы

Проверка проводилась аналогичными сервисами, что и в основном задании.
Получил следующие результаты.

#### HTML файл

Основные проблемы:
* файл html не минифицирован
* основной контент находится ниже баннеров и меню. Но тут сложно сказать приходят
  люди именно за контентом, или им нужно выбрать в меню какой то раздел. Тогда
  меню лучше грузить быстрее. Плюс есть ощущение, что основной контент все же грузится
  сразу, а баннеры добавляют потом динамически.

#### CSS файлы

Количество блокирующих css файлов: **4**
```
http://www.mts.ru/f/css/basic_single.v36366.css
http://www.mts.ru/…ition_lk/f/css/for_mts_header.v36366.css
http://www.mts.ru/f/css/new_page.v36366.css
http://login.mts.ru/profile/css/2014/basic.css
```
Основные проблемы:
* очень много файлов css. Как следствие много запросов.
* некоторые файлы не минифицированы. К файлам выше это не относится. Динамически
  подгружаются несколько файлов стилей после загрузки DOM. Вот они как раз не
  минифицированы.
* на главной странице не используется 95% правил css
* Сhrome devTools -> Audits ругается на то, что объявление файла css расположено
  не в `<head>`, а в `<body>`. Но я такого объявления не нашел. Возможно добавляют
  динамически после загрузки.
* файл `basis.css` расположен в `<head>` после внешних скриптов. Приводит к тому, что
  он загружается намного позже ранее объявленных css файлов.
* большой объем всех css файлов - 640 KB.

#### JS файлы

Количество блокирующих js файлов: **4**
```
http://www.mts.ru/f/js/main_header.v36366.js
http://www.mts.ru/f/js/frontpage.v36366.js
http://www.media.mts.ru/export/mtsrumain/
http://www.mts.ru/f/js/main_body_finish.v36366.js
```
Основные проблемы:
* если не ошибаюсь, то тут файлы js разделены по функциональности. Тогда имеет смысл
  оставить один файл, который относится к критичной функциональности, а остальным
  дописать атрибут `async`. Тогда будет меньше запросов, а оставшаяся часть функциональности
  догрузится после.
* очень много файлов вызывается динамически. Они в свою очередь блокируют рендеринг.
* некоторые файлы не минифицированы. В основном это относится к тем файлам, загрузка которых
  осуществляется динамически.
* некоторые файлы не сжаты.

#### Images

Следующие изображения можно сжать лучше:

```
(80.9 KB, compressed = 25.7 KB - savings of 55.1 KB) - http://www.media.mts.ru/upload/images/materials/Sky_Clouds_452.jpg
(52.6 KB, compressed = 9.0 KB - savings of 43.6 KB) - http://www.media.mts.ru/upload/images/materials/on_takoi_odin_452.jpg
(50.9 KB, compressed = 9.9 KB - savings of 41.0 KB) - http://www.media.mts.ru/upload/images/materials/Mechta_introverta_452.jpg
(64.5 KB, compressed = 24.0 KB - savings of 40.5 KB) - http://www.media.mts.ru/upload/images/materials/putevoditeli_452.jpg
(46.3 KB, compressed = 9.8 KB - savings of 36.5 KB) - http://banners.adfox.ru/160729/mts/235838/1747824.jpg
(50.1 KB, compressed = 16.6 KB - savings of 33.5 KB) - http://www.media.mts.ru/upload/images/materials/oblaka_452.jpg
(38.5 KB, compressed = 8.7 KB - savings of 29.7 KB) - http://www.media.mts.ru/upload/images/materials/social_f_452.jpg
(46.8 KB, compressed = 19.3 KB - savings of 27.5 KB) - http://www.media.mts.ru/upload/images/materials/bili_bili_452.jpg
(27.9 KB, compressed = 9.5 KB - savings of 18.5 KB) - http://static.mts.ru/uploadmsk/contents/3164/banner_main-bonus-mus.jpg
(25.4 KB, compressed = 7.4 KB - savings of 18.0 KB) - http://www.media.mts.ru/upload/images/materials/Pamyati_smartfona_posvyaschaetsya_452.jpg
(16.6 KB, compressed = 8.8 KB - savings of 7.8 KB) - http://www.media.mts.ru/upload/images/materials/semero_koz_452.jpg
(9.8 KB, compressed = 5.1 KB - savings of 4.7 KB) - http://www.media.mts.ru/upload/images/materials/gde_dengi_zin_452.jpg
```

Итого можно выиграть ~350 kB.

#### Кеширование

Для многих файлов не указан срок действия:
```
http://login.mts.ru/profile/css/2014/basic.css (не указан срок действия)
http://login.mts.ru/profile/css/2015v3/basic-1.css (не указан срок действия)
http://login.mts.ru/profile/css/2015v3/basic.css (не указан срок действия)
http://login.mts.ru/profile/css/2015v3/header.css (не указан срок действия)
http://login.mts.ru/profile/css/2015v3/init.css (не указан срок действия)
http://login.mts.ru/profile/css/2015v3/new-page.css (не указан срок действия)
http://login.mts.ru/…profile/css/fnt/CorpidE1SCd_Regular.woff (не указан срок действия)
http://login.mts.ru/…rofile/img/2015/cabinet-subnav/img-1.png (не указан срок действия)
http://login.mts.ru/…rofile/img/2015/cabinet-subnav/img-2.png (не указан срок действия)
http://login.mts.ru/…rofile/img/2015/cabinet-subnav/img-3.png (не указан срок действия)
http://login.mts.ru/profile/img/2015/spr_v3.png (не указан срок действия)
http://login.mts.ru/profile/js/2015v3/jquery-1.9.1.min.js (не указан срок действия)
http://login.mts.ru/…file/js/2015v3/modernizr.custom.84103.js (не указан срок действия)
http://www.media.mts.ru/…ages/materials/Mechta_introverta_452.jpg (не указан срок действия)
http://www.media.mts.ru/…amyati_smartfona_posvyaschaetsya_452.jpg (не указан срок действия)
http://www.media.mts.ru/…load/images/materials/Sky_Clouds_452.jpg (не указан срок действия)
http://www.media.mts.ru/…pload/images/materials/bili_bili_452.jpg (не указан срок действия)
http://www.media.mts.ru/…d/images/materials/gde_dengi_zin_452.jpg (не указан срок действия)
http://www.media.mts.ru/upload/images/materials/oblaka_452.jpg (не указан срок действия)
http://www.media.mts.ru/…d/images/materials/on_takoi_odin_452.jpg (не указан срок действия)
http://www.media.mts.ru/…ad/images/materials/putevoditeli_452.jpg (не указан срок действия)
http://www.media.mts.ru/…load/images/materials/semero_koz_452.jpg (не указан срок действия)
http://www.media.mts.ru/…upload/images/materials/social_f_452.jpg (не указан срок действия)
http://www.googletagmanager.com/gtm.js?id=GTM-P2TC9V (15 минут)
http://www.googletagmanager.com/gtm.js?id=GTM-TLXGKS (15 минут)
https://mc.yandex.ru/metrika/watch.js (60 минут)
http://www.google-analytics.com/analytics.js (2 часа)
http://static.mts.ru/…/contents/3164/banner_main-bonus-mus.jpg (4 часа)
```

Некоторые картинки, которые относятся к новостям или периодическим акциям, имеют
эпизодический характер, и, возможно, их не стоит заносить в кэш надолго. Но, хоть
какой-то срок им поставить нужно. Ну, а статику, иконки и постоянные картинки нужно
заносить в кэш на максимальный срок.

#### Сжатие файлов

Тут все ок, никаких нареканий.

### Рекомендации по уменьшению времени загрузки сайта mts.ru

Тут все практически идентично рекомендациям для лайфхакера. За исключением мелких
моментов, которые я сразу описывал в проблемах выше.

### Заключение

Хочу выразить благодарность создателям следующих сервисов, которые помогают
ускорить обнаружение проблем, связанных с медленной загрузкой сайта:

- [WebPage test](http://www.webpagetest.org/)
- [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/)
- [GT Metrix](https://gtmetrix.com)
- [Varvy Pagespeed](https://varvy.com/pagespeed/)