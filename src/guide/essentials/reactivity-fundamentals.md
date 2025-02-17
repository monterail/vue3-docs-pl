---
outline: deep
---

# Podstawy reaktywności {#reactivity-fundamentals}

:::tip API Preference
Ta strona i wiele innych rozdziałów w dalszej części przewodnika zawiera różne treści dla Options API i Composition API. Twoim obecnym wyborem jest <span class="options-api">Options API</span><span class="composition-api">Composition API</span>. Możesz przełączać się między stylami API za pomocą przełączników „Preferowane API” u góry lewego paska bocznego.
:::

<div class="options-api">

## Deklarowanie reaktywnego stanu \* {#declaring-reactive-state}

W Options API używamy opcji `data`, aby zadeklarować reaktywny stan komponentu. Wartością tej opcji powinna być funkcja zwracająca obiekt. Vue wywoła tę funkcję podczas tworzenia nowej instancji komponentu i obejmie zwrócony obiekt swoim systemem reaktywności. Właściwości najwyższego poziomu tego obiektu są mapowane na instancję komponentu (`this` w metodach i hakach cyklu życia):

```js{2-6}
export default {
  data() {
    return {
      count: 1
    }
  },

  // `mounted` to hak cyklu życia, który omówimy później
  mounted() {
    // `this` odnosi się do instancji komponentu.
    console.log(this.count) // => 1

   // dane także mogą być modyfikowane
    this.count = 2
  }
}
```

[Wypróbuj w playground](https://play.vuejs.org/#eNpFUNFqhDAQ/JXBpzsoHu2j3B2U/oYPpnGtoetGkrW2iP/eRFsPApthd2Zndilex7H8mqioimu0wY16r4W+Rx8ULXVmYsVSC9AaNafz/gcC6RTkHwHWT6IVnne85rI+1ZLr5YJmyG1qG7gIA3Yd2R/LhN77T8y9sz1mwuyYkXazcQI2SiHz/7iP3VlQexeb5KKjEKEe2lPyMIxeSBROohqxVO4E6yV6ppL9xykTy83tOQvd7tnzoZtDwhrBO2GYNFloYWLyxrzPPOi44WWLWUt618txvASUhhRCKSHgbZt2scKy7HfCujGOqWL9BVfOgyI=)

Te właściwości instancji są dodawane tylko w momencie tworzenia instancji, więc musisz upewnić się, że wszystkie są obecne w obiekcie zwracanym przez funkcję `data`. W razie potrzeby użyj `null`, `undefined` lub innej wartości zastępczej dla właściwości, które nie mają jeszcze dostępnej wartości.

Możliwe jest dodanie nowej właściwości bezpośrednio do `this` bez umieszczania jej w data. Jednak właściwości dodane w ten sposób nie będą mogły wywoływać reaktywnych aktualizacji.

Vue używa prefiksu `$` do eksponowania własnych wbudowanych API poprzez instancję komponentu. Zarezerwowany jest także prefiks `_` dla wewnętrznych właściwości. Powinieneś unikać używania nazw rozpoczynających się od tych znaków dla właściwości `data`

### Proxy reaktywne vs. oryginalne \* {#reactive-proxy-vs-original}

W Vue 3 dane stają się reaktywne dzięki wykorzystaniu [JavaScript Proxies](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy). Użytkownicy przechodzący z Vue 2 powinni być świadomi następującego przypadku brzegowego:

```js
export default {
  data() {
    return {
      someObject: {}
    }
  },
  mounted() {
    const newObject = {}
    this.someObject = newObject

    console.log(newObject === this.someObject) // false
  }
}
```

Kiedy uzyskujesz dostęp do `this.someObject` po przypisaniu do niego nowej wartości, wartość ta jest reaktywną wersją oryginalnego `newObject`. **W przeciwieństwie do Vue 2, oryginalny `newObject` pozostaje nietknięty i nie staje się reaktywny: zawsze uzyskuj dostęp do reaktywnego stanu jako właściwości `this`**.

</div>

<div class="composition-api">

## Deklarowanie reaktywnego stanu \*\* {#declaring-reactive-state-1}

### `ref()` \*\* {#ref}

W API Kompozycji zalecanym sposobem deklarowania reaktywnego stanu jest użycie funkcji [`ref()`](/api/reactivity-core#ref):

```js
import { ref } from 'vue'

const count = ref(0)
```

`ref()` przyjmuje argument i zwraca go opakowanego w obiekt z właściwością `.value`:

```js
const count = ref(0)

console.log(count) // { value: 0 }
console.log(count.value) // 0

count.value++
console.log(count.value) // 1
```

> Zobacz także: [Typowanie odniesień (ref)](/guide/typescript/composition-api#typing-ref) <sup class="vt-badge ts" />

Aby uzyskać dostęp do ref w szablonie komponentu, należy je zadeklarować i zwrócić z funkcji `setup()`:

```js{5,9-11}
import { ref } from 'vue'

export default {
  // `setup` to specjalny hak dedykowany Composition API.
  setup() {
    const count = ref(0)

       // udostępnij odniesienie (ref) do szablonu
    return {
      count
    }
  }
}
```

```vue-html
<div>{{ count }}</div>
```

Zauważ, że w szablonie **nie** musieliśmy używać `.value`. Dla wygody, Vue automatycznie rozpakowuje ref w szablonach (z kilkoma [zastrzeżeniami](#caveat-when-unwrapping-in-templates)).

Możesz także bezpośrednio modyfikować ref w obsłudze zdarzeń:

```vue-html{1}
<button @click="count++">
  {{ count }}
</button>
```

W przypadku bardziej złożonej logiki można deklarować funkcje, które modyfikują ref w tym samym zakresie i zwracać je jako metody:

```js{7-10,15}
import { ref } from 'vue'

export default {
  setup() {
    const count = ref(0)

    function increment() {
      // użycie .value jest potrzebne JavaScript
      count.value++
    }

          // użycie .value jest potrzebne JavaScript
    return {
      count,
      increment
    }
  }
}
```

Udostępnione metody można następnie wykorzystać jako procedury obsługi zdarzeń:

```vue-html{1}
<button @click="increment">
  {{ count }}
</button>
```

Oto przykład na żywo w [Codepen](https://codepen.io/vuejs-examples/pen/WNYbaqo), bez użycia żadnych narzędzi do kompilacji.

### `<script setup>` \*\* {#script-setup}

Ręczne ujawnianie stanu i metod za pomocą `setup()` może być rozwlekłe. Na szczęście można tego uniknąć, używając [Single-File Components (SFCs)](/guide/scaling-up/sfc). Możemy uprościć użycie za pomocą `<script setup>`:

```vue{1}
<script setup>
import { ref } from 'vue'

const count = ref(0)

function increment() {
  count.value++
}
</script>

<template>
  <button @click="increment">
    {{ count }}
  </button>
</template>
```

[Wypróbuj w playground](https://play.vuejs.org/#eNo9jUEKgzAQRa8yZKMiaNcllvYe2dgwQqiZhDhxE3L3jrW4/DPvv1/UK8Zhz6juSm82uciwIef4MOR8DImhQMIFKiwpeGgEbQwZsoE2BhsyMUwH0d66475ksuwCgSOb0CNx20ExBCc77POase8NVUN6PBdlSwKjj+vMKAlAvzOzWJ52dfYzGXXpjPoBAKX856uopDGeFfnq8XKp+gWq4FAi)

Importy najwyższego poziomu, zmienne i funkcje zadeklarowane w `<script setup>` są automatycznie używalne w szablonie tego samego komponentu. Pomyśl o szablonie jako o funkcji JavaScript zadeklarowanej w tym samym zakresie - naturalnie ma dostęp do wszystkiego, co jest zadeklarowane obok niej.

:::tip
W dalszej części przewodnika będziemy używać głównie składni SFC + `<script setup>` dla przykładów kodu Composition API, ponieważ jest to najczęstsze użycie przez programistów Vue.

Jeśli nie używasz SFC, nadal możesz używać Composition API z opcją [`setup()`](/api/composition-api-setup)
:::

### Dlaczego używamy refs? \*\* {#why-refs}

Możesz się zastanawiać, dlaczego potrzebujemy refs z `.value` , a nie po prostu zwykłych zmiennych. Aby to wyjaśnić, musimy krótko omówić, jak działa system reaktywności w Vue.

Kiedy używasz ref w szablonie i później zmieniasz wartość tego ref, Vue automatycznie wykrywa tę zmianę i aktualizuje DOM. Jest to możliwe dzięki systemowi reaktywności opartemu na śledzeniu zależności. Kiedy komponent jest renderowany po raz pierwszy, Vue **śledzi** każdy ref, który został użyty podczas renderowania. Później, gdy ref zostaje zmodyfikowany, **wywołuje** to ponowne renderowanie komponentów, które go śledzą.

W standardowym JavaScript nie ma sposobu na wykrycie dostępu lub modyfikacji zwykłych zmiennych. Jednak możemy przechwycić operacje get i set na właściwościach obiektów za pomocą metod getter i setter.

Właściwość `.value` daje Vue możliwość wykrycia, kiedy ref został użyty lub zmodyfikowany. Wewnątrz Vue odbywa się śledzenie w jego getterze, a wywołanie ponownego renderowania w jego setterze. Koncepcyjnie, możesz myśleć o ref jako o obiekcie, który wygląda tak:

```js
// pseudokod, nie jest to rzeczywista implementacja
const myRef = {
  _value: 0,
  get value() {
    track()
    return this._value
  },
  set value(newValue) {
    this._value = newValue
    trigger()
  }
}
```

Inną zaletą refs jest to, że w przeciwieństwie do zwykłych zmiennych, możesz przekazywać refs do funkcji, zachowując dostęp do najnowszej wartości i połączenie reaktywności. Jest to szczególnie przydatne, gdy refaktoryzujesz złożoną logikę kodu, który może być wielokrotnie używany.

System reaktywności jest omawiany szczegółowo w sekcji [Reaktywność w szczegółach](/guide/extras/reactivity-in-depth).

</div>

<div class="options-api">

## Deklarowanie metod \* {#declaring-methods}

<VueSchoolLink href="https://vueschool.io/lessons/methods-in-vue-3" title="Bezpłatna lekcja o metodach Vue.js"/>

Aby dodać metody do instancji komponentu, używamy opcji `methods`. Powinien to być obiekt zawierający pożądane metody:

```js{7-11}
export default {
  data() {
    return {
      count: 0
    }
  },
  methods: {
    increment() {
      this.count++
    }
  },
  mounted() {
    // metody mogą być wywoływane w hakach cyklu życia lub innych metodach!
    this.increment()
  }
}
```

Vue automatycznie wiąże wartość `this` dla `methods`, aby zawsze odnosiła się do instancji komponentu. Dzięki temu metoda zachowa odpowiednią wartość `this`, jeśli będzie używana jako nasłuchiwacz zdarzeń lub funkcja zwrotna. Należy unikać używania funkcji strzałkowych przy definiowaniu `methods`, ponieważ uniemożliwia to Vue przypisanie odpowiedniej wartości `this`:

```js
export default {
  methods: {
    increment: () => {
      // ŹLE: brak dostępu do `this`!
    }
  }
}
```

Podobnie jak wszystkie inne właściwości instancji komponentu, `metody` są dostępne z wnętrza szablonu komponentu. W szablonie są najczęściej używane jako nasłuchiwacze zdarzeń:

```vue-html
<button @click="increment">{{ count }}</button>
```

[Wypróbuj w playground](https://play.vuejs.org/#eNplj9EKwyAMRX8l+LSx0e65uLL9hy+dZlTWqtg4BuK/z1baDgZicsPJgUR2d656B2QN45P02lErDH6c9QQKn10YCKIwAKqj7nAsPYBHCt6sCUDaYKiBS8lpLuk8/yNSb9XUrKg20uOIhnYXAPV6qhbF6fRvmOeodn6hfzwLKkx+vN5OyIFwdENHmBMAfwQia+AmBy1fV8E2gWBtjOUASInXBcxLvN4MLH0BCe1i4Q==)

In the example above, the method `increment` will be called when the `<button>` is clicked.

</div>

### Głęboka reaktywność {#deep-reactivity}

<div class="options-api">

W Vue stan jest domyślnie głęboko reaktywny. Oznacza to, że możesz oczekiwać, że zmiany będą wykrywane, nawet gdy modyfikujesz zagnieżdżone obiekty lub tablice:

```js
export default {
  data() {
    return {
      obj: {
        nested: { count: 0 },
        arr: ['foo', 'bar']
      }
    }
  },
  methods: {
    mutateDeeply() {
      // te operacje będą działać zgodnie z oczekiwaniami.
      this.obj.nested.count++
      this.obj.arr.push('baz')
    }
  }
}
```

</div>

<div class="composition-api">

Refs mogą przechowywać dowolny typ wartości, w tym głęboko zagnieżdżone obiekty, tablice lub wbudowane struktury danych JavaScript, takie jak `Map`.

Ref sprawi, że jego wartość będzie głęboko reaktywna. Oznacza to, że możesz oczekiwać, że zmiany będą wykrywane, nawet gdy modyfikujesz zagnieżdżone obiekty lub tablice:

```js
import { ref } from 'vue'

const obj = ref({
  nested: { count: 0 },
  arr: ['foo', 'bar']
})

function mutateDeeply() {
  // te operacje będą działać zgodnie z oczekiwaniami.
  obj.value.nested.count++
  obj.value.arr.push('baz')
}
```

Nieprymitywne wartości są zamieniane na reaktywne proxy za pomocą [`reactive()`](#reactive), które omawiamy poniżej.

Możliwe jest również wyłączenie głębokiej reaktywności za pomocą [płytkich odniesień(shallow refs)](/api/reactivity-advanced#shallowref). Shallow refs śledzą tylko dostęp do `.value` w reaktywności. Płytkie refs mogą być używane do optymalizacji wydajności, unikając kosztu obserwacji dużych obiektów lub w przypadkach, gdy wewnętrzny stan jest zarządzany przez zewnętrzną bibliotekę.

Dalsza lektura:

- [Redukowanie nadmiaru reaktywności dla dużych struktur niemutowalnych](/guide/best-practices/performance#reduce-reactivity-overhead-for-large-immutable-structures)
- [Integracja z systemami zewnętrznymi stanu](/guide/extras/reactivity-in-depth#integration-with-external-state-systems)

</div>

### Czas aktualizacji DOM {#dom-update-timing}

Kiedy modyfikujesz reaktywny stan, DOM jest automatycznie aktualizowany. Należy jednak zauważyć, że aktualizacje DOM nie są stosowane synchronicznie. Zamiast tego Vue buforuje je do „następnej klatki (next tick)” w cyklu aktualizacji, aby upewnić się, że każdy komponent jest aktualizowany tylko raz, niezależnie od tego, ile zmian stanu zostało dokonanych.

Aby poczekać na zakończenie aktualizacji DOM po zmianie stanu, możesz użyć globalnego API [nextTick()](/api/general#nexttick):

<div class="composition-api">

```js
import { nextTick } from 'vue'

async function increment() {
  count.value++
  await nextTick()
  // Teraz DOM jest aktualizowany
}
```

</div>
<div class="options-api">

```js
import { nextTick } from 'vue'

export default {
  methods: {
    async increment() {
      this.count++
      await nextTick()
      // Teraz DOM jest aktualizowany
    }
  }
}
```

</div>

<div class="composition-api">

## `reactive()` \*\* {#reactive}

Istnieje inny sposób deklarowania reaktywnego stanu za pomocą API `reactive()`. W przeciwieństwie do ref, które opakowuje wewnętrzną wartość w specjalny obiekt, `reactive()` sprawia, że obiekt sam w sobie staje się reaktywny:

```js
import { reactive } from 'vue'

const state = reactive({ count: 0 })
```

> Zobacz również: [Typowanie reactive](/guide/typescript/composition-api#typing-reactive) <sup class="vt-badge ts" />

Usage in template:

```vue-html
<button @click="state.count++">
  {{ state.count }}
</button>
```

Reaktywne obiekty to [JavaScript Proxies](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) i zachowują się jak zwykłe obiekty. Różnica polega na tym, że Vue jest w stanie przechwycić dostęp i modyfikację wszystkich właściwości reaktywnego obiektu w celu śledzenia reaktywności i wywoływania aktualizacji.

`reactive()` konwertuje obiekt głęboko: zagnieżdżone obiekty są również opakowywane w`reactive()`, gdy są dostępne. Jest również wywoływane wewnętrznie przez `ref()`, gdy wartość ref jest obiektem. Podobnie jak w przypadku płytkich odniesień (shallow refs), istnieje również API [`shallowReactive()`](/api/reactivity-advanced#shallowreactive) do rezygnacji z głębokiej reaktywności.

### Proxy reaktywne vs. oryginalne \*\* {#reactive-proxy-vs-original-1}

Ważne jest, aby zauważyć, że zwrócony przez `reactive()` obiekt to [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) oryginalnego obiektu, który nie jest równy oryginalnemu obiektowi:

```js
const raw = {}
const proxy = reactive(raw)

// proxy NIE jest równe oryginałowi.
console.log(proxy === raw) // false
```

Tylko proxy jest reaktywne - modyfikowanie oryginalnego obiektu nie wywoła aktualizacji. Dlatego najlepszą praktyką podczas pracy z systemem reaktywności Vue jest **wyłącznie używanie wersji proxy twojego stanu**.

Aby zapewnić spójny dostęp do proxy, wywołanie `reactive()` na tym samym obiekcie zawsze zwraca to samo proxy, a wywołanie `reactive()` na istniejącym proxy również zwraca to samo proxy:

```js
// wywołanie reactive() na tym samym obiekcie zwraca to samo proxy
console.log(reactive(raw) === proxy) // true

// wywołanie reactive() na proxy zwraca samo proxy
console.log(reactive(proxy) === proxy) // true
```

Ta zasada dotyczy również obiektów zagnieżdżonych. Ze względu na głęboką reaktywność, obiekty zagnieżdżone wewnątrz obiektu reaktywnego są również proxy:

```js
const proxy = reactive({})

const raw = {}
proxy.nested = raw

console.log(proxy.nested === raw) // false
```

### Ograniczenia `reactive()` \*\* {#limitations-of-reactive}

API `reactive()` ma kilka ograniczeń:

1. **Ograniczone typy wartości:** działa tylko dla typów obiektów (obiekty, tablice oraz [typy kolekcji](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects#keyed_collections) takie jak `Map` i `Set`). Nie obsługuje typów prymitywnych jak `string`, `number` czy `boolean`.

2. **Nie można zastąpić całego obiektu:** ponieważ śledzenie reaktywności Vue działa na dostępach do właściwości, musimy zawsze zachować tę samą referencję do reaktywnego obiektu. Oznacza to, że nie możemy łatwo „zastąpić” reaktywnego obiektu, ponieważ połączenie reaktywności z pierwszą referencją jest tracone:

   ```js
   let state = reactive({ count: 0 })

   // powyższa referencja ({ count: 0 }) nie jest już śledzona
   // (połączenie reaktywności jest utracone!)
   state = reactive({ count: 1 })
   ```

3. **Nieprzyjazne destrukturyzacji:** gdy destrukturalizujemy właściwość reaktywnego obiektu typu prymitywnego do lokalnych zmiennych lub przekazujemy tę właściwość do funkcji, tracimy połączenie reaktywności:

   ```js
   const state = reactive({ count: 0 })

   // count jest odłączony od state.count, gdy jest destrukturyzowane.
   let { count } = state
   // nie wpływa na oryginalny stan
   count++

   // funkcja otrzymuje zwykłą liczbę
   // i nie będzie w stanie śledzić zmian w state.count
   // musimy przekazać cały obiekt, aby zachować reaktywność
   callSomeFunction(state.count)
   ```

Z powodu tych ograniczeń zalecamy używanie `ref()` jako głównego API do deklarowania reaktywnego stanu.

## Dodatkowe szczegóły rozpakowywania ref \*\* {#additional-ref-unwrapping-details}

### Jako właściwość obiektu reaktywnego \*\* {#ref-unwrapping-as-reactive-object-property}

Ref jest automatycznie rozpakowywany, gdy jest używany jako właściwość obiektu reaktywnego. Innymi słowy, zachowuje się jak zwykła właściwość:

```js
const count = ref(0)
const state = reactive({
  count
})

console.log(state.count) // 0

state.count = 1
console.log(count.value) // 1
```

Jeśli nowy ref zostanie przypisany do właściwości powiązanej z istniejącym ref, zastąpi stary ref:

```js
const otherCount = ref(2)

state.count = otherCount
console.log(state.count) // 2
// oryginalny ref jest teraz odłączony od state.count
console.log(count.value) // 1
```

Rozpakowywanie ref zachodzi tylko wtedy, gdy jest zagnieżdżony w głęboko reaktywnym obiekcie. Nie ma to miejsca, gdy jest dostępny jako właściwość [płytkiego obiektu reaktywnego](/api/reactivity-advanced#shallowreactive).

### Ostrzeżenie dotyczące tablic i kolekcji \*\* {#caveat-in-arrays-and-collections}

W przeciwieństwie do obiektów reaktywnych, **nie** wykonuje się rozpakowywania, gdy odwołanie jest uzyskiwane jako element tablicy reaktywnej lub natywnego typu kolekcji, takiego jak `Map`:

```js
const books = reactive([ref('Vue 3 Guide')])
// tutaj potrzebne jest .value
console.log(books[0].value)

const map = reactive(new Map([['count', ref(0)]]))
// tutaj potrzebne jest .value
console.log(map.get('count').value)
```

### Ostrzeżenie dotyczące rozpakowywania w szablonach \*\* {#caveat-when-unwrapping-in-templates}

Rozpakowywanie ref w szablonach ma zastosowanie tylko wtedy, gdy ref jest właściwością najwyższego poziomu w kontekście renderowania szablonu.

W poniższym przykładzie `count` i `object` są właściwościami najwyższego poziomu, ale `object.id` nie jest:

```js
const count = ref(0)
const object = { id: ref(1) }
```

Zatem wyrażenie to działa zgodnie z oczekiwaniami:

```vue-html
{{ count + 1 }}
```

...podczas gdy ten **NIE**:

```vue-html
{{ object.id + 1 }}
```

Wyrenderowany wynik będzie `[object Object]1`, ponieważ `object.id` nie jest rozpakowywany podczas oceny wyrażenia i pozostaje obiektem ref. Aby to naprawić, możemy zdestrukturyzować `id` do właściwości najwyższego poziomu:

```js
const { id } = object
```

```vue-html
{{ id + 1 }}
```

Teraz wynik renderowania będzie wynosić `2`.

Another thing to note is that a ref does get unwrapped if it is the final evaluated value of a text interpolation (i.e. a <code v-pre>{{ }}</code> tag), so the following will render `1`:

Inną rzeczą, na którą należy zwrócić uwagę, jest to, że ref jest rozpakowywany, jeśli jest ostateczną wartością interpolacji tekstu (tj. znacznikiem <code v-pre>{{ }}</code>), więc poniższe wyrażenie spowoduje wyrenderowanie `1`:

```vue-html
{{ object.id }}
```

Jest to po prostu wygodna funkcja interpolacji tekstu i jest równoważna z <code v-pre>{{ object.id.value }}</code>.

</div>

<div class="options-api">

### Metody stanowe \* {#stateful-methods}

W niektórych przypadkach może być konieczne dynamiczne utworzenie funkcji metody, na przykład utworzenie obsługi zdarzeń debounced:

```js
import { debounce } from 'lodash-es'

export default {
  methods: {
    // Debouncing z użyciem Lodash
    click: debounce(function () {
      // ... reakcja na kliknięcie ...
    }, 500)
  }
}
```

Jednak takie podejście jest problematyczne w przypadku komponentów, które są ponownie wykorzystywane, ponieważ funkcja debounced jest **stanowa**: utrzymuje pewien stan wewnętrzny w czasie, który upłynął. Jeśli wiele instancji komponentów współdzieli tę samą funkcję debounced, będą one ze sobą kolidować.

Aby zachować niezależność funkcji debounced każdej instancji komponentu od pozostałych, możemy utworzyć wersję debounced w haku cyklu życia `created`:

```js
export default {
  created() {
    // każda instancja ma teraz własną kopię obsługi debounced
    this.debouncedClick = _.debounce(this.click, 500)
  },
  unmounted() {
    // dobrym pomysłem jest również wyłączenie timera
    // gdy komponent jest usuwany
    this.debouncedClick.cancel()
  },
  methods: {
    click() {
      // ... reakcja na kliknięcie ...
    }
  }
}
```

</div>
