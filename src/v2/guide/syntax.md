---
title: Синтаксис шаблону
type: guide
order: 4
---

Vue.js використовує синтаксис на основі HTML-шаблону, що дозволяє декларативно зв'язувати промальовування об'єктної моделі документу (DOM) за даними, закріпленими з екземпляром Vue. Усі шаблони Vue.js є дійсним HTML-форматом, придатним для обробки браузерами відповідно до специфікацій, а також іншими обробниками HTML.

"Під капотом", Vue перетворює ці шаблони в так звані функції промальовування віртуальної об'єктної моделі документу (Virtual DOM). В комбінації із системою реактивності, Vue здатний розумно визначати мінімальне число компонентів для перемальовування та застосовувати мінімальну кількість маніпуляцій з DOM по мірі змін даних додатку.

При охоті, якщо ви знайомі з концепцією Virtual DOM і надаєте перевагу можливостям чистого JavaScript, ви зможете [відразу писати функції промальовування](render-function.html) замість шаблонів, з використанням JSX.

## Інтерполяція

### Текст

Найпростішою формою зв'язування даних — це текстова інтерполяція з використанням синтаксису "вуса" (подвійні фігурні дужки)

``` html
<span>Повідомлення: {{ msg }}</span>
```

Тут, "вуса" будуть замінені на дійсне значення властивості `msg` відповідно до переданого об'єкту даних. Воно буде автоматично змінюватися по мірі змін цієї властивості.

Ви також можете застосовувати одноразову інтерполяцію, яка не буде оновлюватися наступні рази, використовуючи [директиву v-once](../api/#v-once), але майте на увазі, що це вплине також на будь-які інші зв'язані дані того ж вузла:

``` html
<span v-once>Це ніколи не зміниться: {{ msg }}</span>
```

### Чистий HTML

Подвійні фігурні дужки розуміють дані як звичайний текст, а не HTML. Для того, щоб вказати реальний HTML, вам потрібно використовувати директиву [`v-html`](../api/#v-html):

``` html
<p>Використання "вусів": {{ rawHtml }}</p>
<p>Використання v-html: <span v-html="rawHtml"></span></p>
```

{% raw %}
<div id="example1" class="demo">
  <p>Використання mustache: {{ rawHtml }}</p>
  <p>Використання v-html: <span v-html="rawHtml"></span></p>
</div>
<script>
new Vue({
  el: '#example1',
  data: function () {
    return {
      rawHtml: '<span style="color: red">Цей текст червоний.</span>'
    };
  }
});
</script>
{% endraw %}

Вміст тегу `span` буде замінено значенням властивості `rawHtml`, інтерпретовану як звичайний HTML — зв'язування даних ігнорується. Зауважте, що ви не можете використовувати `v-html` для створення частин шаблонів, тому що Vue не є рядковим шаблонізатором. Замість цього краще використовувати компоненти як базову одиницю UI для повторного використання та композиції.

<p class="tip">Динамічне промальовування довільного HTML на вашому сайті може бути дуже небезпечним, оскільки може легко призвести до [XSS вразливостей](https://uk.wikipedia.org/wiki/%D0%9C%D1%96%D0%B6%D1%81%D0%B0%D0%B9%D1%82%D0%BE%D0%B2%D0%B8%D0%B9_%D1%81%D0%BA%D1%80%D0%B8%D0%BF%D1%82%D0%B8%D0%BD%D0%B3). Використовуйте інтерполяцію HTML лише з довіреним вмістом та **ніколи** не використовуйте на вмісті, що вводиться користувачем.</p>

### Атрибути

"Вуса" не можуть використовуватися всередині HTML-атрибутів. Замість цього використовуйте [директиву `v-bind`](../api/#v-bind):

``` html
<div v-bind:id="dynamicId"></div>
```

У випадку з булевими атрибутами, існування яких базується на значенні `true`, `v-bind` працює дещо по-іншому. Для прикладу:

``` html
<button v-bind:disabled="isButtonDisabled">Кнопка</button>
```

Якщо `isButtonDisabled` має значення `null`, `undefined`, або `false`, атрибут `disabled` навіть не буде промальовано в елементі `<button>`.

### Використання JavaScript виразів

До цього часу ми використовували зв'язування даних до звичайних властивостей в наших шаблонах. Але насправді Vue.js підтримує всю силу виразів JavaScript всередині зв'язаних даних:

``` html
{{ number + 1 }}

{{ ok ? 'ТАК' : 'НІ' }}

{{ message.split('').reverse().join('') }}

<div v-bind:id="'list-' + id"></div>
```

Ці вирази будуть обчислені як JavaScript в контексті даних у Vue екземплярі. Єдиним обмеженням є те, що кожне зв'язування даних повинно містити **один-єдиний вираз**, тому ось цей код **НЕ** працюватиме:

``` html
<!-- це твердження, а не вираз: -->
{{ var a = 1 }}

<!-- управління потоком теж не працюватиме, використовуйте тернарні вирази -->
{{ if (ok) { return message } }}
```

<p class="tip">Вирази в шаблонах ізольовані і можуть мати доступ до [чітко визначеного списку глобальних змінних](https://github.com/vuejs/vue/blob/v2.6.10/src/core/instance/proxy.js#L9), таких як `Math` та `Date`. Не намагайтеся звернутися до глобальних змінних, визначених користувачем з шаблону.</p>

## Директиви

Директиви є спеціальними атрибутами, які мають префікс `v-`. Очікуваним значенням такого атрибуту є **єдиний JavaScript вираз** (виключенням є `v-for`, про який ми поговоримо пізніше). Завданням директиви є реактивне додавання побічних ефектів до DOM, коли значення або вираз атрибуту змінюється. Переглянемо той самий приклад, який ми бачити у вступі:

``` html
<p v-if="seen">Тепер ти мене бачиш</p>
```

Тут директива `v-if` видаляє або додає елемент `<p>` залежно від того, чи вираз `seen` є правдивим.

### Аргументи

Деякі директиви можуть приймати певний аргумент, відділений двокрапкою після імені директиви. Для прикладу, директива `v-bind` використовується для реактивного оновлення HTML атрибутів:

``` html
<a v-bind:href="url"> ... </a>
```

Тут `href` є її аргументом, що вказує директиві `v-bind` зв'язати атрибут елемента `href` із значенням в значенні виразу, в даному випадку `url`.

Іншим зразком є директива `v-on`, яка "слухає" події DOM:

``` html
<a v-on:click="doSomething"> ... </a>
```

Тут аргументом є назва події, яку потрібно прослуховувати. Ми поговоримо детальніше про обробку подій теж.

### Динамічні аргументи

> Нове в 2.6.0+

Починаючи з версії 2.6.0, також стало можливим використовувати вирази JavaScript як аргумент директиви, оформленими у квадратні дужки:

``` html
<!--
Варто зазначити, що існують певні обмеження до таких виразів,
як це нижче пояснено в "Обмеження до виразів у динамічних аргументах".
-->
<a v-bind:[attributeName]="url"> ... </a>
```

Тут `attributeName` буде обчислено динамічно як вираз JavaScript expression, і його обчислене значення буде використовуватися як кінцеве значення для цього аргументу. Наприклад, якщо ваш Vue екземпляр має властивість `attributeName`, значення якої дорівнює `"href"`, тоді цей зв'язок буде еквівалентом до `v-bind:href`.

Таким же чином, ви можете використовувати динамічні аргументи для зв'язування обробників динамічних подій:

``` html
<a v-on:[eventName]="doSomething"> ... </a>
```

В даному прикладі, якщо значення `eventName` дорівнює `"focus"`, тоді вираз `v-on:[eventName]` буде рівнозначним до `v-on:focus`.

#### Обмеження до виразів у динамічних значеннях

Очікується, що динамічні аргументи будуть обчислені як рядкова величина, за винятком `null`. Спеціальне значення `null` може використовуватися для явного видалення зв'язування. Використання будь-якого іншого не-рядкового значення приведе до відповідного застереження.

#### Обмеження до виразів у динамічних аргументах

Вирази у динамічних аргументах мають деякі обмеження у їхньому синтаксисі, пов'язаними із певними символами, такими як пробіли, лапки. Їх використання всередині імен атрибутів та є помилковим. Для прикладу, ось цей вираз буде помилкою:

``` html
<!-- Це згенерує застереження -->
<a v-bind:['foo' + bar]="value"> ... </a>
```

Виходом із ситуації буде або використання без пробілів та лапок, або ж заміна такого складного виразу на обчислювану властивість.

Використовуючи in-DOM шаблони (шаблони, що вже знаходяться в DOM, написані безпосередньо в HTML файлі), ви повинні також уникати назв ключів у верхньому регістрі оскільки браузери перетворять їх на нижній регістр: 

``` html
<!--
Це буде перетворено в v-bind:[someattr] в in-DOM шаблонах.
Якщо ваш Vue еземпляр не має властивості "someattr", ваш код не працюватиме.
-->
<a v-bind:[someAttr]="value"> ... </a>
```

### Модифікатори

Модифікаторами є спеціальні суфікси, розділені крапкою, які вказують на те, що директива, в якій вони застосовуються повинна оброблятися відповідним чином. Для прикладу, модифікатор `.prevent` говорить директиві `v-on` автоматично викликати `event.preventDefault()` на тій чи іншій події:

``` html
<form v-on:submit.prevent="onSubmit"> ... </form>
```

Ви побачите приклади інших модифікаторів пізніше, [для `v-on`](events.html#Event-Modifiers) та [для `v-model`](forms.html#Modifiers), коли ми будемо розглядати їхні можливості.

## Скорочення

Префікс `v-` слугує як візуальна зачіпка для ідентифікації Vue-атрибутів у ваших шаблонах. Це корисно, якщо ви використовуєте Vue.js для задавання динамічної поведінки до наявного HTML, але це може бути занадто для деяких часто використовуваних директив. Разом з тим, необхідність в префіксі `v-` стає менш важливим, якщо ви розробляєте [SPA](https://uk.wikipedia.org/wiki/%D0%9E%D0%B4%D0%BD%D0%BE%D1%81%D1%82%D0%BE%D1%80%D1%96%D0%BD%D0%BA%D0%BE%D0%B2%D0%B8%D0%B9_%D0%B7%D0%B0%D1%81%D1%82%D0%BE%D1%81%D1%83%D0%BD%D0%BE%D0%BA), де Vue керує кожним шаблоном. Саме тому Vue надає спеціальні скорочення для двох найчастіше використовуваних директив, `v-bind` та `v-on`:

### Скорочення для `v-bind`

``` html
<!-- повний синтаксис -->
<a v-bind:href="url"> ... </a>

<!-- скорочений -->
<a :href="url"> ... </a>

<!-- скорочення із динамічним аргументом (починаючи з 2.6.0+) -->
<a :[key]="url"> ... </a>
```

### Скорочення для `v-on`

``` html
<!-- повний синтаксис -->
<a v-on:click="doSomething"> ... </a>

<!-- скорочений -->
<a @click="doSomething"> ... </a>

<!-- скорочення із динамічним аргументом (починаючи з 2.6.0+) -->
<a @[event]="doSomething"> ... </a>
```

Це може виглядати дещо незвично для HTML, але `:` та `@` є коректними символами для імен атрибутів для всіх браузерів, які підтримує Vue. Крім того, дані символи не потраплять до відмальованого HTML. Використання скорочень не є обов'язковим, але ймовірно, що він вам сподобається пізніше, коли ви їх вивчите та приклади застування таких скорочень.
