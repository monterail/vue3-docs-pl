# Właściwości computed {#computed-properties}

<div class="options-api">
  <VueSchoolLink href="https://vueschool.io/lessons/computed-properties-in-vue-3" title="Darmowa lekcja o właściwościach computed w Vue.js"/>
</div>

<div class="composition-api">
  <VueSchoolLink href="https://vueschool.io/lessons/vue-fundamentals-capi-computed-properties-in-vue-with-the-composition-api" title="Darmowa lekcja o właściwościach cpmputed w Vue.js"/>
</div>

## Podstawowy przykład {#basic-example}

Wyrażenia w szablonie są bardzo wygodne, ale są przeznaczone do prostych operacji. Umieszczanie zbyt dużej ilości logiki do szablonów może sprawić, że staną się one przeładowane i trudne do utrzymania. Na przykład, jeśli mamy obiekt z zagnieżdżoną tablicą:

<div class="options-api">

```js
export default {
  data() {
    return {
      author: {
        name: 'John Doe',
        books: [
          'Vue 2 - Advanced Guide',
          'Vue 3 - Basic Guide',
          'Vue 4 - The Mystery'
        ]
      }
    }
  }
}
```

</div>
<div class="composition-api">

```js
const author = reactive({
  name: 'John Doe',
  books: [
    'Vue 2 - Advanced Guide',
    'Vue 3 - Basic Guide',
    'Vue 4 - The Mystery'
  ]
})
```

</div>

I chcemy wyświetlić różne komunikaty w zależności od tego, czy `author` ma już jakieś książki, czy nie:

```vue-html
<p>Czy wydał jakieś książki:</p>
<span>{{ author.books.length > 0 ? 'Tak' : 'Nie' }}</span>
```

W tym momencie szablon zaczyna być nieco przeładowany. Trzeba przez chwilę na niego popatrzeć, zanim zrozumiemy, że wykonuje on obliczenia w zależności od `author.books`. Co ważniejsze, prawdopodobnie nie chcemy powtarzać tego samego obliczenia w kilku miejscach w szablonie.

Dlatego w przypadku bardziej złożonej logiki, która obejmuje dane reaktywne, zaleca się użycie **właściwości computed**. Oto ten sam przykład, ale zrefaktoryzowany:

<div class="options-api">

```js
export default {
  data() {
    return {
      author: {
        name: 'John Doe',
        books: [
          'Vue 2 - Advanced Guide',
          'Vue 3 - Basic Guide',
          'Vue 4 - The Mystery'
        ]
      }
    }
  },
  computed: {
    // getter właściwości computed
    publishedBooksMessage() {
      // `this` odnosi się do instancji komponentu
      return this.author.books.length > 0 ? 'Tak' : 'Nie'
    }
  }
}
```

```vue-html
<p>Czy wydał jakieś książki:</p>
<span>{{ publishedBooksMessage }}</span>
```

[Wypróbuj w playground](https://play.vuejs.org/#eNqFkN1KxDAQhV/l0JsqaFfUq1IquwiKsF6JINaLbDNui20S8rO4lL676c82eCFCIDOZMzkzXxetlUoOjqI0ykypa2XzQtC3ktqC0ydzjUVXCIAzy87OpxjQZJ0WpwxgzlZSp+EBEKylFPGTrATuJcUXobST8sukeA8vQPzqCNe4xJofmCiJ48HV/FfbLLrxog0zdfmn4tYrXirC9mgs6WMcBB+nsJ+C8erHH0rZKmeJL0sot2tqUxHfDONuyRi2p4BggWCr2iQTgGTcLGlI7G2FHFe4Q/xGJoYn8SznQSbTQviTrRboPrHUqoZZ8hmQqfyRmTDFTC1bqalsFBN5183o/3NG33uvoWUwXYyi/gdTEpwK)

Tutaj zadeklarowaliśmy właściwość computed `publishedBooksMessage`.

Spróbuj zmienić wartość tablicy `books` w danych aplikacji, a zobaczysz, jak `publishedBooksMessage` zmienia się odpowiednio.

Możesz powiązać właściwości computed w szablonach tak samo, jak zwykłe właściwości. Vue wie, że `this.publishedBooksMessage` zależy od `this.author.books`, więc zaktualizuje wszelkie powiązania zależne od `this.publishedBooksMessage`, gdy `this.author.books` się zmieni.

Zobacz także: [Typowanie wartości computed](/guide/typescript/options-api#typing-computed-properties) <sup class="vt-badge ts" />

</div>

<div class="composition-api">

```vue
<script setup>
import { reactive, computed } from 'vue'

const author = reactive({
  name: 'John Doe',
  books: [
    'Vue 2 - Advanced Guide',
    'Vue 3 - Basic Guide',
    'Vue 4 - The Mystery'
  ]
})

// computed ref
const publishedBooksMessage = computed(() => {
  return author.books.length > 0 ? 'Yes' : 'No'
})
</script>

<template>
  <p>Czy wydał jakieś książki:</p>
  <span>{{ publishedBooksMessage }}</span>
</template>
```

[Wypróbuj w playground](https://play.vuejs.org/#eNp1kE9Lw0AQxb/KI5dtoTainkoaaREUoZ5EEONhm0ybYLO77J9CCfnuzta0vdjbzr6Zeb95XbIwZroPlMySzJW2MR6OfDB5oZrWaOvRwZIsfbOnCUrdmuCpQo+N1S0ET4pCFarUynnI4GttMT9PjLpCAUq2NIN41bXCkyYxiZ9rrX/cDF/xDYiPQLjDDRbVXqqSHZ5DUw2tg3zP8lK6pvxHe2DtvSasDs6TPTAT8F2ofhzh0hTygm5pc+I1Yb1rXE3VMsKsyDm5JcY/9Y5GY8xzHI+wnIpVw4nTI/10R2rra+S4xSPEJzkBvvNNs310ztK/RDlLLjy1Zic9cQVkJn+R7gIwxJGlMXiWnZEq77orhH3Pq2NH9DjvTfpfSBSbmA==)

Tutaj zadeklarowaliśmy właściwość computed `publishedBooksMessage`. Funkcja `computed()` oczekuje [funkcji getter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get#description), a zwrócona wartość to **computed ref**. Podobnie jak normalne refy, możesz uzyskać dostęp do wyniku obliczeń jako `publishedBooksMessage.value`. Computed refy są także automatycznie rozpakowywane w szablonach, więc możesz się do nich odwoływać bez `.value` w wyrażeniach szablonu.

Właściwość computed automatycznie śledzi swoje reaktywne zależności. Vue wie, że obliczenie `publishedBooksMessage` zależy od `author.books`, więc zaktualizuje wszystkie powiązania zależne od `publishedBooksMessage`, gdy `author.books` się zmieni.

Zobacz także: [Typowanie właściwości computed](/guide/typescript/composition-api#typing-computed) <sup class="vt-badge ts" />

</div>

## Buforowanie computed vs. metody {#computed-caching-vs-methods}

Być może zauważyłeś, że ten sam rezultat możemy osiągnąć, wywołując metodę w wyrażeniu:

```vue-html
<p>{{ calculateBooksMessage() }}</p>
```

<div class="options-api">

```js
// w komponencie
methods: {
  calculateBooksMessage() {
    return this.author.books.length > 0 ? 'Yes' : 'No'
  }
}
```

</div>

<div class="composition-api">

```js
// w komponencie
function calculateBooksMessage() {
  return author.books.length > 0 ? 'Yes' : 'No'
}
```

</div>

Zamiast właściwości computed możemy zdefiniować tę samą funkcję jako metodę. W ostatecznym wyniku oba podejścia dają dokładnie ten sam efekt. Jednak różnica polega na tym, że **właściwości computed są buforowane na podstawie ich reaktywnych zależności**. Właściwość computed zostanie ponownie obliczona tylko wtedy, gdy zmienią się jej reaktywne zależności. Oznacza to, że tak długo, jak `author.books` nie ulegnie zmianie, każde kolejne odwołanie do `publishedBooksMessage` zwróci natychmiast wcześniej obliczony wynik, bez konieczności ponownego uruchamiania funkcji zwracającej wartość.

To także oznacza, że następująca obliczona właściwość nigdy się nie zaktualizuje, ponieważ `Date.now()` nie jest reaktywną zależnością:

<div class="options-api">

```js
computed: {
  now() {
    return Date.now()
  }
}
```

</div>

<div class="composition-api">

```js
const now = computed(() => Date.now())
```

</div>

Dla porównania, wywołanie metody **zawsze** uruchomi funkcję za każdym razem, gdy nastąpi ponowne renderowanie.

Dlaczego potrzebujemy buforowania? Wyobraź sobie, że mamy kosztowną właściwość computed `list`, która wymaga przetworzenia ogromnej tablicy i wykonania wielu operacji obliczeniowych. Jeśli inne obliczone właściwości zależą od `list`, to bez buforowania musielibyśmy uruchamiać funkcję zwracającą wartość `list` wiele razy więcej niż jest to konieczne! W przypadkach, w których nie chcesz korzystać z buforowania, zamiast tego użyj wywołania metody.

## Zapisywalne właściwości computed {#writable-computed}

Obliczone właściwości są domyślnie tylko do odczytu. Jeśli spróbujesz przypisać nową wartość do właściwości computed, otrzymasz ostrzeżenie w czasie działania. W rzadkich przypadkach, gdy potrzebujesz "zapisywalnej" właściwości computed, możesz ją utworzyć, definiując zarówno getter, jak i setter:

<div class="options-api">

```js
export default {
  data() {
    return {
      firstName: 'John',
      lastName: 'Doe'
    }
  },
  computed: {
    fullName: {
      // getter
      get() {
        return this.firstName + ' ' + this.lastName
      },
      // setter
      set(newValue) {
        // Zwróć uwagę, że używamy tutaj składni destrukturyzacji.
        ;[this.firstName, this.lastName] = newValue.split(' ')
      }
    }
  }
}
```

Teraz, gdy wykonasz `this.fullName = 'Jan Kowalski'`, wywołany zostanie setter i `this.firstName` oraz `this.lastName` zostaną odpowiednio zaktualizowane.

</div>

<div class="composition-api">

```vue
<script setup>
import { ref, computed } from 'vue'

const firstName = ref('John')
const lastName = ref('Doe')

const fullName = computed({
  // getter
  get() {
    return firstName.value + ' ' + lastName.value
  },
  // setter
  set(newValue) {
    // Zwróć uwagę, że używamy tutaj składni destrukturyzacji.
    ;[firstName.value, lastName.value] = newValue.split(' ')
  }
})
</script>
```

Teraz, gdy wykonasz `fullName.value = 'Jan Kowalski'`, wywołany zostanie setter i `firstName` oraz `lastName` zostaną odpowiednio zaktualizowane.

</div>

## Najlepsze praktyki {#best-practices}

### Gettery powinny być wolne od efektów ubocznych {#getters-should-be-side-effect-free}

Należy pamiętać, że funkcje zwracające wartość właściwości computed powinny wykonywać jedynie czystą kalkulację i być wolne od efektów ubocznych. **Nie modyfikuj innych stanów, nie wykonuj asynchronicznych żądań i nie manipuluj DOM-em wewnątrz getterów obliczonych właściwości!** Traktuj obliczoną właściwość jako deklaratywny sposób opisania, jak obliczyć wartość na podstawie innych wartości – jej jedynym zadaniem powinno być obliczenie i zwrócenie tej wartości. Później w przewodniku omówimy, jak możemy wykonywać efekty uboczne w reakcji na zmiany stanu za pomocą [obserwatorów (watchers)](./watchers).

### Unikaj modyfikowania wartości obliczonych {#avoid-mutating-computed-value}

Zwracana wartość z obliczonej właściwości jest stanem pochodnym. Traktuj ją jako tymczasowy zrzut danych – za każdym razem, gdy zmienia się stan źródłowy, tworzony jest nowy zrzut. Modyfikowanie takiego zrzutu nie ma sensu, dlatego wartość zwracana przez obliczoną właściwość powinna być traktowana jako tylko do odczytu i nigdy nie powinna być modyfikowana. Zamiast tego należy aktualizować stan źródłowy, od którego zależy obliczona właściwość, aby wywołać ponowne przeliczenie.
