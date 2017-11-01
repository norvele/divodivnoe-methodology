# Методология dd

В этом документе описана методология, ее понятия, синтаксис, принципы, рекомендации, примеры. Методология позволяет полностью исключить пересечения\протечки стилей, построить удобную структуру проекта, в которой легко находить нужное, легко и безопасно изменять код.

Методология не вводит искусственных ограничений, я считаю, что разработчики вправе менять правила если того требует ситуация, если это обосновано и не грозит потерей контроля над кодом.

Принципы и правила вырабатывались около 2 лет, обкатывались на боевых проектах, как мелких, так и в кровавом интерпрайзе.

## Основные понятия

### Контекст

Несмотря на то, что понятие контекста достаточно очевидно, нужно удостовериться что все правильно понимают в каких случаях элемент находится в контексте, а в каких нет, т.к. на этом в методологии строится очень многое. Я приведу несколько примеров:

Элемент `header-logo` в контексте своего компонента:
```html
<div class="header">
    <div class="header-logo"></div>
</div>
```

Элемент `header-logo` попрежнему в контексте своего компонента:
```html
<div class="header">
    <div class="header-wrapper">
        <div class="header-logo"></div>
    </div>
</div>
```

Элемент `menu-link` в контексте компонента `header`:
```html
<div class="header">
    <div class="menu-link"></div>
</div>
```

Как можно было догадаться - у элемента всегда есть контекст, но не всегда это контекст своего компонента. Далее я расскажу, что это за непонятные элементы и компоненты.

### Элемент

**Элемент** - единица разметки. Например: иконка бургера, заголовок, кнопка, инпут, шапка, меню шапки, пункт меню шапки. Вобщем все, что мы видим в разметке - это элементы.

Из примеров можно заметить, что элементы могут быть вложенными, а также могут находиться в контексте некоего root элемента (например, для элементов 'меню шапки', 'пункт меню шапки' корневым элементом будет являться шапка). Такие root элементы именуются компонентами и о них речь пойдет ниже.

Важно понимать, что элементы могут быть расположены в любом месте страницы. Если нам сильно захочется - мы чисто технически в футере сможем использовать элемент `header-logo` без каких либо последствий, но в таких ситуациях нас обязан останавливать здавый смысл.

Всвязи с тем, что мы не знаем в каком контексте будет использоваться элемент - мы не должны использовать "внешние" свойства для его описания в стилях. Ко внешним свойствам относятся те, которые могут повлиять на расположение других элементов:

* margin
* position
* top, left, right, bottom
* display
* float
* z-index
* clear
* width, min-width, max-width
* height, min-height, max-height

Эти свойства могут быть применены к элементу только его контекстом. Если просто - только родитель может влиять на расположение элемента внутри себя (есть, конечно, исключения). Также родитель вправе устанавливать окончательный вид элемента.

### Компонент

**Компонент** - элемент-библиотека (упомянутый ранее как root элемент), т.е. нечто представляющее собой обертку над своими элементами. Эта обертка может являться как вполне себе осязаемым DOM элементом, так и просто неймспейсом в стилях, под которым лежат какие-то элементы. Каждый компонент всегда представлен отдельным файлом в проекте.

Например могут быть такие компоненты:
* компонент-элемент `header`, его элементы: `header-logo`, `header-menu`, `header-link`
* компонент-библиотека `typography`, его элементы: `typography-link`, `typography-header`, `typography-block`

Все отличие между компонентом-элементом и компонентом-библиотекой в том, что у компонента-элемента могут быть свои стили и он задает стили своим элементам если они находятся в его контексте.

Вот пример стилей компонента `menu`:

```scss
.menu {
    $self: &; // Сюда поместится селектор .menu
    padding: 10px; // Задаем свои стили компонента
    
    &-item { // Стили для элемента
        padding: 10px;
        background-color: black;
        color: white;
    }
    
    /*
        Стили для элемента
        если тот находится в контексте своего компонента.
        На выходе будет селектор ".menu .menu-item"
     */
    #{$self}-item {
        display: inline-block;
        margin-bottom: 10px;
    }
}
```

### Модификатор

**Модификатор** - класс, позволяющий изменить внешний вид элемента (или компонента-элемента). Бывают 2 типа модификаторов: разновидности и состояния.

**Разновидности** описывают тип элемента, т.е. его какие-то постоянные характеристики. Например: эта кнопка большая `.button._large`, это карточка товара `.card._product`. Записываются такие модификаторы через нижнее подчеркивание, чтобы не вызывать коллизий с классами элементов.

**Состояния** описывают динамические характеристики, т.е. те, которые могут изменяться с течением времени. Например кнопка нажата `.button.-pressed`, основная вкладка активна `tab._main.-active`. Записываются через дефис.

Модификаторы не могут быть использованы без указания элемента, к которому они принадлежат. Т.е. нельзя написать `<div class="_large -active"></div>`, нужно писать `<div class="button _large -active"></div>`. Естественно, что в стилях они тоже должны распространяться только на свой элемент:

```scss
.button {
    
    &._large {};
    
    &.-active {};
}
```

Иногда допускается использовать модификаторы без элемента. Такие модификаторы записываются через два подчеркивания для разновидностей `__hidden` и через два дефиса для состояний `--loading`. Это достаточно удобно, если какие-то стили могут быть применены к абсолютно любому элементу.

Важно понимать, что модификаторы должны влиять только на внешний вид, не затрагивая разметку. Если установка элементу модификатора подразумевает еще и изменение html кода этого элемента, то это значит, что тут должны использоваться абстрактные и уточняющие компоненты.

### Абстрактные и уточняющие компоненты

Ввод понятий абстрактного и уточняющего позволяет очень прозрачно делать большие и сложные компоненты. Эта техника не привносит абсолютно ничего нового, более того - она основопологающая для css без препроцессоров.

Пример: у нас есть модальное окно с каким-то текстом. Также у нас есть модальное окно с видео и модальное окно со слайдером. Мы понимаем, что у этих модальных окон разный набор элементов (чуть разная html разметка) и чуть могут отличаться стили, но есть что-то общее (само окно, затенение страницы, заголовок), общее поведение. Есть несколько вариантов решения:

0. сделать разные компоненты `modal`, `modalVideo`, `modalSLider`;
0. сделать все в одном компоненте `modal` с модификаторами `_video`, `_slider`;
0. сделать абстрактный компонент `modal` и уточняющие `modalVideo`, `modalSLider` (о да, похоже на 1 вариант)

Очевидно, что делая разные компоненты для каждого модального окна, мы в любом случае продублируем общие стили, либо используя миксины, либо плейсхолдеры, поэтому этот вариант нам не подходит (т.к. общих стилей может быть достаточно много). Вариант с модификаторами тоже плохой, т.к. у нас изменяется html разметка. Также модификаторы для такой задачи в разы ухудшат читаемость кода и необосновано повысят специфичность селекторов. Поэтому нам нужны абстрактные и уточняющие компоненты.

Пример:
```html
<div class="modal">
    <div class="modal-header"></div>
    <div class="modal-content"></div>
</div>
    
<div class="modal modalSlider">
    <div class="modal-header"></div>
    <div class="modal-content">
        <div class="modalSlider-slider"></div>
    </div>
</div>
``` 
```scss
$abstract: '.modal';
    
// Абстрактный компонент модального окна
#{$abstract} {
    position: fixed;
    
    &-header {
        padding: 20px;
    };
    
    &-content {
        padding: 20px;
    },
}
    
// Уточняющий компоненты модального окна со слайдером
#{$abstract}Slider {

    &-slider {
        // Стили для слайдера
    }
    
    .modal-content {
        padding: 0; // Если нужно - переопределяем стили
    }
}
```

Для таких компонентов создается папка по названию абстрактного компонента `modal`, внутри папки создается 2 файла `modal.scss` и `modalSLider.scss`.

Такой подход дает много плюсов:
* нет дублирования стилей;
* файлы стилей не перегружены;
* мы можем менять разметку и добавлять элементы, которые относятся только к нужному нам уточняющему компоненту;
* адекватная специфичность селекторов.

Важно понимать, что мы не можем использовать уточняющий класс `.modalSLider` без использования `.modal` на том же элементе. А вот ограничиться только абстрактным `.modal` можем. Наименование уточняющих классов должно начинаться с имени абстрактного.

## Синтаксис, кодстайл и структура папок

Методология не запрещает использовать другие стили именования, например БЭМ, но придерживается своего стиля именования классов, который дает более высокую читаемость.

Пример именования сложного селектора: `.searchForm-userActions-button._size-extraLarge.-active`.

Тут мы сказали, что у нас есть компонент `searchForm`, у которого есть элемент `userActions`, в свою очередь у которого есть еще элемент `button`. У кнопки есть разновидность `size-extraLarge`, а также состояние `active`.

Все имена селекторов разбиты на логические части, отделяемые дефисом. В случае с селектором элемента они разделяют между собой элементы\компоненты. В случае с модификатором они образуют неймспейсы, которые позволяют проще понимать на что они влияют. Например: помимо изменения размера самой кнопки нам может понадобиться изменять размер внутренних отступов кнопки, тогда запись `_extraLarge` будет уже не такой прозрачной, а вот записи `_size-extraLarge` и `_gutters-extraLarge` вполне конкретно сообщают нам что мы будем модифицировать.

Рекомендованная структура проекта для верстки:

* `index.html` - точка входа
* `index.scss` - подключение стилей папок common и components
* `common`
  * `setting.scss` - настройки, переменные, миксины
  * `global.scss` - глобальные стили (ресеты\нормалайзы, стили для html, body и т.д.)
* `components`
  * `modal` - папка абстрактного компонента
    * `modal.scss` - абстрактный компонент
    * `modalSlider.scss` - уточняющий компонент
  * `header.scss` - компонент
  * `menu.scss` - компонент
  * ...

Здесь показана идея того, как могла бы выглядеть структура простого проекта для верстки. На проектах с бэкэндом или фронтэндом она будет претерпевать изменения, но организация файлов стилей должна оставаться такой.

Преимущество плоской организации компонентов в том, что это гарантирует уникальность имен компонентов, а также облегчает поиск.

Описание каждого компонента так же подчиняется определнным правилам:

```scss
.component {

    // #> Описание библиотеки компонента #
    &-element {};  // Элементы
    &::before {}; // и псевдоэлементы
    // <# Описание библиотеки компонента #
    
    // #> Описание в контексте #
    .component-element {}; // Элементы в контексте (свои)
    .menu-item {} // Элементы в контексте (чужие)
    // <# Описание в контексте #
    
    // #> Состояния и разновидности #
    &:hover {}; // Стандартные состояния
    &._size-large {}; // Модификаторы-разновидности
    &.-active {}; // Модификаторы-состояния
    // <# Состояния и разновидности #
    
    // #> Адаптивные свойства #
    @media all and (min-width: 640px) {}
    // <# Адаптивные свойства #
}
```

Описание состоит из 4 блоков, которые должны идти в представленном выше порядке.

// TODO

## Специфичность селекторов

Методология позволяет работать с разными уровнями специфичности селекторов, в отличие от того же БЭМ, где специфичность любого селектора равна 0010. Почему бы не использовать все возможности css? Мы имеем право в селекторах использовать теги, писать стили в style атрибутах и даже использовать !important, но обо всем по порядку.

Важно понимать, что увеличение специфичности всегда должно быть оправдано, в любом случае нельзя писать селекторы вида `.header #form .menu p span > h2`.

* Для обычных элементов специфичность равна 0010, т.к. элементы представленны одним классом, например `.header`.
* Для элементов в контексте специфичность равна 0020, это логично и оправдано, т.к. мы можем переопределять стили такому элементу, например `.header .menu-item`.
* Элементы с модификаторами также имеют специфичность 0020, т.к. переопределяются стили по умолчанию, например `.button._primary`.
* Элементы в контексте модификатора имеют специфичность 0030 `.button._primary .icon`. Мы хотим изменить иконку внутри кнопки, причем только внутри основной кнопки.

// TODO

## Разделение ответственности

Ранее мы говорили, что только контекст может устанавливать элементам внешние свойства - это одно из правил разделения ответственности.

Другое правило состоит в том, что элементы должны выполнять только свои функции, сетки должны отвечать за позиционирование элементов, типографика за отображение текстовой информации, формы за взаимодействие с пользователем и т.д.

Допустим у нас есть тривиальная задача сделать список новостей по 3 карточки в колонку, с наскоку мы бы могли сделать что-то такое:

```html
<!-- Плохой пример -->
<div class="newsList">
    <div class="newsList-item">
        <div class="newsList-item-image"></div>
        <div class="newsList-item-title"></div>
        <div class="newsList-item-desc"></div>
    </div>
    <div class="newsList-item">...</div>
    ...
</div>
```

Т.е. у нас получается, что `.newsList-item` будет являться карточкой новости, а также отвечать за свое позиционирование. Что здесь плохого? Плохо то, что для выстраивания по 3 карточки в ряд мы будем дожны назначать карточкам ширину и маргины (для отступов между ними):

```scss
// Плохой пример
.newsList {
    $self: &;
    
    &-item {
        // Стили карточки
    }
    
    #{self}-item {
        $gutter: 20px;
        $columns: 3;
        margin-right: $gutter;
        width: calc(#{100% / 3} - #{$gutter * ($columns - 1)});
        
        &:nth-child(3n + 3) {
            margin-right: 0;
        }
    }
}
```

При том, что часть кода опущена - выглядит очень заморочено, помимо этого у нас есть еще куча проблем: мы всегда должны контролировать кол-во колонок в списке, мы должны будем постоянно переопределять `&:nth-child(3n + 3)` если делаем адаптив, мы всегда ограничены по кол-ву элементов в сетке и не сможем в одном месте сделать 3 элемента в строку, в другом 4. Если сделать сетку на базе отрицательных маргинов - то часть проблем уйдет, но появится другая проблема: паддинги для создания отступов между колонками станут паддингами карточек и нужно будет делать еще 1 обертку.

Что же делать? Очевидно, что за расположение элементов должна отвечать сетка, а за внешний вид карточки новости - собственно компонент карточки новости.

```html
<div class="newsList">
    <div class="newsList-grid">
        <div class="newsList-item">
            <div class="newsCard">
                <div class="newsCard-image"></div>
                <div class="newsCard-title"></div>
                <div class="newsCard-desc"></div>
            </div>
        </div>
        <div class="newsList-item">...</div>
        ...
    </div>
</div>
```

```scss
.newsList {
    $self: &;
    $gutter: 20px;
    
    #{$self}-grid {
        margin: 0 -$gutter/2;
    }
    
    #{$self}-item {
        padding: 0 $gutter/2;
        width: 100% / 12 * 4;
    }
}

.newsCard {
    // Стили карточки
}
```

Пример выше лишен всех недостатков: код стал чистым и менее хрупким, карточка полностью отвязана от сетки, мы можем легко сделать адаптив и менять кол-во колонок простым переопределением свойства `width` у ячеек сетки. И это всего лишь первый уровень абстракции, на крупных проектах используется более высокий уровень, так можно сетки вынести в компонент `grid` и использовать их для всего в проекте, естественно, что в результате появится множество инструментов, которые позволят подстраивать сетки под каждый конкретный случай.

В конечном итоге правильная работа с абстракциями и разделем ответственности приводит к консистентности кода, лучшей читаемости и увеличению скорости разработки.

## Рекомендации

* про & > span
* про & span

// TODO