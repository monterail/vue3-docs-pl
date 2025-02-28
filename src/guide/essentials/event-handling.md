# Obsługa zdarzeń {#event-handling}

<div class="options-api">
  <VueSchoolLink href="https://vueschool.io/lessons/user-events-in-vue-3" title="Darmowa lekcja Vue.js o zdarzeniach"/>
</div>

<div class="composition-api">
  <VueSchoolLink href="https://vueschool.io/lessons/vue-fundamentals-capi-user-events-in-vue-3" title="Darmowa lekcja Vue.js o zdarzeniach"/>
</div>

## Nasłuchiwanie zdarzeń {#listening-to-events}

Możemy użyć dyrektywy `v-on`, którą zazwyczaj skracamy do symbolu `@`, aby nasłuchiwać zdarzeń DOM i uruchamiać JavaScript, gdy zostaną one wywołane. Użycie wyglądałoby tak: `v-on:click="handler"` lub przy użyciu skrótu: `@click="handler"`.

Wartość handler może być jedną z następujących:

1. **Metody obsługi zdarzeń inline:** Kod JavaScript wykonywany bezpośrednio po wywołaniu zdarzenia (podobnie jak natywny atrybut `onclick`).

2. **Metody obsługi zdarzeń:** Nazwa właściwości lub ścieżka wskazująca na metodę zdefiniowaną w komponencie.

## Obsługa zdarzeń inline {#inline-handlers}

Metody obsługi zdarzeń inline są zwykle używane w prostych przypadkach, na przykład:

<div class="composition-api">

```js
const count = ref(0)
```

</div>
<div class="options-api">

```js
data() {
  return {
    count: 0
  }
}
```

</div>

```vue-html
<button @click="count++">Dodaj 1</button>
<p>Count is: {{ count }}</p>
```

<div class="composition-api">

[Wypróbuj w playground](https://play.vuejs.org/#eNo9jssKgzAURH/lko0tgrbbEqX+Q5fZaLxiqHmQ3LgJ+fdqFZcD58xMYp1z1RqRvRgP0itHEJCia4VR2llPkMDjBBkmbzUUG1oII4y0JhBIGw2hh2Znbo+7MLw+WjZ/C4TaLT3hnogPkcgaeMtFyW8j2GmXpWBtN47w5PWBHLhrPzPCKfWDXRHmPsCAaOBfgSOkdH3IGUhpDBWv9/e8vsZZ/gFFhFJN)

</div>
<div class="options-api">

[Wypróbuj w playground](https://play.vuejs.org/#eNo9jcEKgzAQRH9lyKlF0PYqqdR/6DGXaLYo1RjiRgrivzepIizLzu7sm1XUzuVLIFEKObe+d1wpS183eYahtw4DY1UWMJr15ZpmxYAnDt7uF0BxOwXL5Evc0kbxlmyxxZLFyY2CaXSDZkqKZROYJ4tnO/Tt56HEgckyJaraGNxlsVt2u6teHeF40s20EDo9oyGy+CPIYF1xULBt4H6kOZeFiwBZnOFi+wH0B1hk)

</div>

## Metody obsługi zdarzeń {#method-handlers}

Jednak w wielu przypadkach logika obsługi zdarzeń będzie bardziej złożona i nie nadaje się do umieszczenia w metodach inline. Dlatego `v-on` może również przyjąć nazwę lub ścieżkę metody komponentu, którą chcesz wywołać.

Na przykład:

<div class="composition-api">

```js
const name = ref('Vue.js')

function greet(event) {
  alert(`Hello ${name.value}!`)
  // `event` to natywne zdarzenie DOM
  if (event) {
    alert(event.target.tagName)
  }
}
```

</div>
<div class="options-api">

```js
data() {
  return {
    name: 'Vue.js'
  }
},
methods: {
  greet(event) {
    // `this` wewnątrz metod odnosi się do aktualnej instancji komponentu

    alert(`Cześć ${this.name}!`)
    // `event` to natywne zdarzenie DOM
    if (event) {
      alert(event.target.tagName)
    }
  }
}
```

</div>

```vue-html
<!-- `greet` to nazwa metody zdefiniowanej powyżej -->
<button @click="greet">Greet</button>
```

<div class="composition-api">

[Wypróbuj w playground](https://play.vuejs.org/#eNpVj0FLxDAQhf/KMwjtXtq7dBcFQS/qzVMOrWFao2kSkkkvpf/dJIuCEBgm771vZnbx4H23JRJ3YogqaM+IxMlfpNWrd4GxI9CMA3NwK5psbaSVVjkbGXZaCediaJv3RN1XbE5FnZNVrJ3FEoi4pY0sn7BLC0yGArfjMxnjcLsXQrdNJtFxM+Ys0PcYa2CEjuBPylNYb4THtxdUobj0jH/YX3D963gKC5WyvGZ+xR7S5jf01yPzeblhWr2ZmErHw0dizivfK6PV91mKursUl6dSh/4qZ+vQ/+XE8QODonDi)

</div>
<div class="options-api">

[Wypróbuj w playground](https://play.vuejs.org/#eNplUE1LxDAQ/StjEbYL0t5LXRQEvag3Tz00prNtNE1CMilC6X83SUkRhJDJfLz3Jm8tHo2pFo9FU7SOW2Ho0in8MdoSDHhlXhKsnQIYGLHyvL8BLJK3KmcAis3YwOnDY/XlTnt1i2G7i/eMNOnBNRkwWkQqcUFFByVAXUNPk3A9COXEgBkGRgtFDkgDTQjcWxuAwDiJBeMsMcUxszCJlsr+BaXUcLtGwiqut930579KST1IBd5Aqlgie3p/hdTIk+IK//bMGqleEbMjxjC+BZVDIv0+m9CpcNr6MDgkhLORjDBm1H56Iq3ggUvBv++7IhnUFZfnGNt6b4fRtj5wxfYL9p+Sjw==)

</div>

Metody obsługi zdarzeń automatycznie otrzymują natywny obiekt zdarzenia DOM, które je wywołało. W powyższym przykładzie mamy dostęp do elementu, który wywołał zdarzenie, poprzez `event.target`.

<div class="composition-api">

Zobacz także: [Typowanie obsługi zdarzeń](/guide/typescript/composition-api#typing-event-handlers) <sup class="vt-badge ts" />

</div>
<div class="options-api">

Zobacz także: [Typowanie obsługi zdarzeń](/guide/typescript/options-api#typing-event-handlers) <sup class="vt-badge ts" />

</div>

### Metoda vs. wykrywanie inline {#method-vs-inline-detection}

Kompilator szablonów Vue wykrywa obsługę metod poprzez sprawdzanie, czy wartość w `v-on` jest poprawnym identyfikatorem JavaScript lub ścieżką dostępu do właściwości. Na przykład `foo`, `foo.bar` oraz `foo['bar']` są traktowane jako metody obsługi zdarzeń, natomiast `foo()` i `count++` są traktowane jako obsługa inline.

## Wywoływanie metod w obsłudze inline {#calling-methods-in-inline-handlers}

Zamiast bezpośredniego przypisywania nazwy metody, możemy także wywołać metodę wewnątrz obsługi inline. Dzięki temu możemy przekazać do niej niestandardowe argumenty zamiast natywnego obiektu zdarzenia:

<div class="composition-api">

```js
function say(message) {
  alert(message)
}
```

</div>
<div class="options-api">

```js
methods: {
  say(message) {
    alert(message)
  }
}
```

</div>

```vue-html
<button @click="say('hello')">Powiedz cześć</button>
<button @click="say('bye')">Powiedz pa</button>
```

<div class="composition-api">

[Wypróbuj w playground](https://play.vuejs.org/#eNp9jTEOwjAMRa8SeSld6I5CBWdg9ZJGBiJSN2ocpKjq3UmpFDGx+Vn//b/ANYTjOxGcQEc7uyAqkqTQI98TW3ETq2jyYaQYzYNatSArZTzNUn/IK7Ludr2IBYTG4I3QRqKHJFJ6LtY7+zojbIXNk7yfmhahv5msvqS7PfnHGjJVp9w/hu7qKKwfEd1NSg==)

</div>
<div class="options-api">

[Wypróbuj w playground](https://play.vuejs.org/#eNptjUEKwjAQRa8yZFO7sfsSi57B7WzGdjTBtA3NVC2ldzehEFwIw8D7vM9f1cX742tmVSsd2sl6aXDgjx8ngY7vNDuBFQeAnsWMXagToQAEWg49h0APLncDAIUcT5LzlKJsqRBfPF3ljQjCvXcknEj0bRYZBzi3zrbPE6o0UBhblKiaKy1grK52J/oA//23IcmNBD8dXeVBtX0BF0pXsg==)

</div>

## Dostęp do argumentu zdarzenia w obsłudze inline{#accessing-event-argument-in-inline-handlers}

Czasami musimy uzyskać dostęp do oryginalnego zdarzenia DOM w obsłudze inline. Można to zrobić, przekazując je do metody za pomocą specjalnej zmiennej `$event` lub używając funkcji strzałkowej inline:

```vue-html
<!-- użycie specjalnej zmiennej $event -->
<button @click="warn('Formularz nie może zostać wysłany.', $event)">
  Wyślij
</button>

<!-- użycie funkcji strzałkowej inline -->
<button @click="(event) => warn('Formularz nie może zostać wysłany.', event)">
  Wyślij
</button>
```

<div class="composition-api">

```js
function warn(message, event) {
  // teraz mamy dostęp do natywnego zdarzenia
  if (event) {
    event.preventDefault()
  }
  alert(message)
}
```

</div>
<div class="options-api">

```js
methods: {
  warn(message, event) {
    // teraz mamy dostęp do natywnego zdarzenia
    if (event) {
      event.preventDefault()
    }
    alert(message)
  }
}
```

</div>

## Modyfikatory zdarzeń {#event-modifiers}

Bardzo często w obsłudze zdarzeń musimy wywołać `event.preventDefault()` lub `event.stopPropagation()`. Chociaż możemy to zrobić bezpośrednio w metodach, lepiej byłoby, gdyby metody zajmowały się wyłącznie logiką danych, a nie szczegółami obsługi zdarzeń DOM.

Aby rozwiązać ten problem, Vue udostępnia **modyfikatory zdarzeń** dla `v-on`. Modyfikatory to sufiksy dyrektyw oznaczone kropką.

- `.stop`
- `.prevent`
- `.self`
- `.capture`
- `.once`
- `.passive`

```vue-html
<!-- propagacja zdarzenia kliknięcia zostanie zatrzymana -->
<a @click.stop="doThis"></a>

<!-- zdarzenie submit nie przeładuje strony -->
<form @submit.prevent="onSubmit"></form>

<!-- modyfikatory mogą być łączone -->
<a @click.stop.prevent="doThat"></a>

<!-- sam modyfikator -->
<form @submit.prevent></form>

<!-- obsługa zdarzenia tylko wtedy, gdy event.target to sam element -->
<!-- czyli nie element potomny -->
<div @click.self="doThat">...</div>
```

::: tip
Kolejność ma znaczenie przy stosowaniu modyfikatorów, ponieważ odpowiedni kod jest generowany w tej samej kolejności. Dlatego użycie `@click.prevent.self` zapobiegnie **domyślnej akcji kliknięcia zarówno na elemencie, jak i jego elementach potomnych**, natomiast `@click.self.prevent` zapobiegnie tylko domyślnej akcji kliknięcia na samym elemencie.
:::

Modyfikatory `.capture`, `.once` i `.passive` odpowiadają [opcjom natywnej metody addEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener#options):

```vue-html

<!-- użyj trybu przechwytywania podczas dodawania nasłuchiwacza zdarzeń -->
<!-- tj. zdarzenie skierowane do elementu wewnętrznego jest obsługiwane -->
<!-- zanim zostanie obsłużone przez ten element                         -->
<div @click.capture="doThis">...</div>

<!-- zdarzenie kliknięcia zostanie wywołane co najwyżej raz -->
<a @click.once="doThis"></a>

<!-- nastąpi domyślne zachowanie zdarzenia (przewijanie)         -->
<!-- natychmiast, zamiast czekać na zakończenie `onScroll`       -->
<!-- w przypadku, gdy zawiera `event.preventDefault()`           -->
<div @scroll.passive="onScroll">...</div>
```

The `.passive` modifier is typically used with touch event listeners for [improving performance on mobile devices](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener#improving_scroll_performance_using_passive_listeners).

::: tip
Nie używaj `.passive` i `.prevent` razem, ponieważ `.passive` informuje przeglądarkę, że nie zamierzasz zapobiegać domyślnemu zachowaniu zdarzenia, co może skutkować ostrzeżeniem.
:::

## Modyfikatory klawiszy {#key-modifiers}

Podczas nasłuchiwania zdarzeń klawiatury często chcemy sprawdzać, czy naciśnięto określony klawisz. Vue umożliwia dodanie modyfikatorów klawiszy do `v-on` lub `@`:

```vue-html
<!-- wywołaj `submit` tylko po naciśnięciu `Enter` -->
<input @keyup.enter="submit" />
```

Możesz używać dowolnych nazw klawiszy zgodnych z [`KeyboardEvent.key`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key/Key_Values), konwertując je do kebab-case.

```vue-html
<input @keyup.page-down="onPageDown" />
```

W powyższym przykładzie procedura obsługi zostanie wywołana tylko wtedy, gdy `$event.key` będzie równe `'PageDown'`.

### Aliasy klawiszy {#key-aliases}

Vue udostępnia aliasy dla najczęściej używanych klawiszy:

- `.enter`
- `.tab`
- `.delete` (obsługuje zarówno "Delete", jak i "Backspace")
- `.esc`
- `.space`
- `.up`
- `.down`
- `.left`
- `.right`

### Modyfikatory klawiszy systemowych {#system-modifier-keys}

Możesz użyć następujących modyfikatorów, aby uruchomić nasłuchiwanie zdarzeń myszy lub klawiatury tylko wtedy, gdy zostanie naciśnięty odpowiedni klawisz modyfikujący:

- `.ctrl`
- `.alt`
- `.shift`
- `.meta`

::: tip Note
Na klawiaturach Macintosh meta to klawisz command (⌘). Na klawiaturach Windows meta to klawisz Windows (⊞). Na klawiaturach Sun Microsystems meta jest oznaczone jako pełny romb (◆). Na niektórych klawiaturach, szczególnie na klawiaturach maszyn MIT i Lisp oraz ich następcach, takich jak klawiatura Knight, klawiatura Space-Cadet, meta jest oznaczone jako „META”. Na klawiaturach Symbolics meta jest oznaczone jako „META” lub „Meta”.
:::

For example:

```vue-html
<!-- Alt + Enter -->
<input @keyup.alt.enter="clear" />

<!-- Ctrl + Click -->
<div @click.ctrl="doSomething">Zrób coś</div>
```

::: tip
Należy pamiętać, że klawisze modyfikujące różnią się od zwykłych klawiszy i gdy są używane ze zdarzeniami `keyup`, muszą zostać naciśnięte, gdy zdarzenie jest emitowane. Innymi słowy, `keyup.ctrl` zostanie wyzwolone tylko wtedy, gdy zwolnisz klawisz, przytrzymując `ctrl`. Nie zostanie wyzwolone, jeśli zwolnisz sam klawisz `ctrl`.
:::

### Modyfikator `.exact` {#exact-modifier}

Modyfikator `.exact` pozwala kontrolować dokładną kombinację modyfikatorów systemowych potrzebnych do wywołania zdarzenia.

```vue-html
<!-- to zostanie wywołane nawet jeśli naciśnięty zostanie również klawisz Alt lub Shift -->

<button @click.ctrl="onClick">A</button>

<!-- to zostanie wywołane tylko wtedy, gdy naciśnięty zostanie klawisz Ctrl i nie zostanie naciśnięty żaden inny klawisz -->

<button @click.ctrl.exact="onCtrlClick">A</button>

<!-- to zostanie wywołane tylko wtedy, gdy nie zostaną naciśnięte żadne modyfikatory systemowe -->
<button @click.exact="onClick">A</button>
```

## Modyfikatory przycisków myszy {#mouse-button-modifiers}

- `.left`
- `.right`
- `.middle`

Te modyfikatory ograniczają obsługę do zdarzeń wyzwalanych przez określony przycisk myszy.
