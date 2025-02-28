# Obserwatory {#watchers}

## Podstawowy przykład {#basic-example}

Właściwości computed pozwalają nam deklaratywnie obliczać wartości pochodne. Istnieją jednak przypadki, w których musimy wykonywać "skutki uboczne" w reakcji na zmiany stanu – na przykład mutując DOM lub zmieniając inny element stanu na podstawie wyniku operacji asynchronicznej.

<div class="options-api">

Z Options API możemy użyć opcji [`watch`](/api/options-state#watch), aby wywołać funkcję za każdym razem, gdy zmieni się właściwość reaktywna:

```js
export default {
  data() {
    return {
      question: '',
      answer: 'Pytania zazwyczaj zawierają znak zapytania. ;-)',
      loading: false
    }
  },
  watch: {
    // za każdym razem, gdy pytanie ulegnie zmianie, ta funkcja zostanie    uruchomiona
    question(newQuestion, oldQuestion) {
      if (newQuestion.includes('?')) {
        this.getAnswer()
      }
    }
  },
  methods: {
    async getAnswer() {
      this.loading = true
      this.answer = 'Myślę...'
      try {
        const res = await fetch('https://yesno.wtf/api')
        this.answer = (await res.json()).answer
      } catch (error) {
        this.answer = 'Błąd! Nie udało się połączyć z API. ' + error
      } finally {
        this.loading = false
      }
    }
  }
}
```

```vue-html
<p>
  Ask a yes/no question:
  <input v-model="question" :disabled="loading" />
</p>
<p>{{ answer }}</p>
```

[Wypróbuj w playground](https://play.vuejs.org/#eNp9VE1v2zAM/SucLnaw1D70lqUbsiKH7rB1W4++aDYdq5ElTx9xgiD/fbT8lXZFAQO2+Mgn8pH0mW2aJjl4ZCu2trkRjfucKTw22jgosOReOjhnCqDgjseL/hvAoPNGjSeAvx6tE1qtIIqWo5Er26Ih088BteCt51KeINfKcaGAT5FQc7NP4NPNYiaQmhdC7VZQcmlxMF+61yUcWu7yajVmkabQVqjwgGZmzSuudmiX4CphofQqD+ZWSAnGqz5y9I4VtmOuS9CyGA9T3QCihGu3RKhc+gJtHH2JFld+EG5Mdug2QYZ4MSKhgBd11OgqXdipEm5PKoer0Jk2kA66wB044/EF1GtOSPRUCbUnryRJosnFnK4zpC5YR7205M9bLhyUSIrGUeVcY1dpekKrdNK6MuWNiKYKXt8V98FElDxbknGxGLCpZMi7VkGMxmjzv0pz1tvO4QPcay8LULoj5RToKoTN40MCEXyEQDJTl0KFmXpNOqsUxudN+TNFzzqdJp8ODutGcod0Alg34QWwsXsaVtIjVXqe9h5bC9V4B4ebWhco7zI24hmDVSEs/yOxIPOQEFnTnjzt2emS83nYFrhcevM6nRJhS+Ys9aoUu6Av7WqoNWO5rhsh0fxownplbBqhjJEmuv0WbN2UDNtDMRXm+zfsz/bY2TL2SH1Ec8CMTZjjhqaxh7e/v+ORvieQqvaSvN8Bf6HV0veSdG5fvSoo7Su/kO1D3f13SKInuz06VHYsahzzfl0yRj+s+3dKn9O9TW7HPrPLP624lFU=)

Opcja `watch` obsługuje również ścieżkę rozdzieloną kropkami jako klucz:

```js
export default {
  watch: {
    // Uwaga: tylko proste ścieżki. Wyrażenia nie są obsługiwane.
    'some.nested.key'(newValue) {
      // ...
    }
  }
}
```

</div>

<div class="composition-api">

Korzystając z Composition API, możemy użyć [funkcji `watch`](/api/reactivity-core#watch), aby uruchomić funkcję zwrotną, gdy zmieni się stan reaktywny:

```vue
<script setup>
import { ref, watch } from 'vue'

const question = ref('')
const answer = ref('Pytania zazwyczaj zawierają znak zapytania. ;-)')
const loading = ref(false)

// watch działa bezpośrednio na ref
watch(question, async (newQuestion, oldQuestion) => {
  if (newQuestion.includes('?')) {
    loading.value = true
    answer.value = 'Myślę...'
    try {
      const res = await fetch('https://yesno.wtf/api')
      answer.value = (await res.json()).answer
    } catch (error) {
      answer.value = 'Błąd! Nie udało się połączyć z API. ' + error
    } finally {
      loading.value = false
    }
  }
})
</script>

<template>
  <p>
    Zadaj pytanie tak/nie:
    <input v-model="question" :disabled="loading" />
  </p>
  <p>{{ answer }}</p>
</template>
```

[Wypróbuj w playground](https://play.vuejs.org/#eNp9U8Fy0zAQ/ZVFF9tDah96C2mZ0umhHKBAj7oIe52oUSQjyXEyGf87KytyoDC9JPa+p+e3b1cndtd15b5HtmQrV1vZeXDo++6Wa7nrjPVwAovtAgbh6w2M0Fqzg4xOZFxzXRvtPPzq0XlpNNwEbp5lRUKEdgPaVP925jnoXS+UOgKxvJAaxEVjJ+y2hA9XxUVFGdFIvT7LtEI5JIzrqjrbGozdOmikxdqTKqmIQOV6gvOkvQDhjrqGXOOQvCzAqCa9FHBzCyeuAWT7F6uUulZ9gy7PPmZFETmQjJV7oXoke972GJHY+Axkzxupt4FalhRcYHh7TDIQcqA+LTriikFIDy0G59nG+84tq+qITpty8G0lOhmSiedefSaPZ0mnfHFG50VRRkbkj1BPceVorbFzF/+6fQj4O7g3vWpAm6Ao6JzfINw9PZaQwXuYNJJuK/U0z1nxdTLT0M7s8Ec/I3WxquLS0brRi8ddp4RHegNYhR0M/Du3pXFSAJU285osI7aSuus97K92pkF1w1nCOYNlI534qbCh8tkOVasoXkV1+sjplLZ0HGN5Vc1G2IJ5R8Np5XpKlK7J1CJntdl1UqH92k0bzdkyNc8ZRWGGz1MtbMQi1esN1tv/1F/cIdQ4e6LJod0jZzPmhV2jj/DDjy94oOcZpK57Rew3wO/ojOpjJIH2qdcN2f6DN7l9nC47RfTsHg4etUtNpZUeJz5ndPPv32j9Yve6vE6DZuNvu1R2Tg==)

### Obserwowanie typów źródłowych {#watch-source-types}

Pierwszy argument `watch` może być różnymi typami reaktywnych „źródeł”: może to być odwołanie (w tym odwołania obliczeniowe (computed refs)), obiekt reaktywny, [funkcja getter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get#description) lub tablica wielu źródeł:

```js
const x = ref(0)
const y = ref(0)

// pojedynczy ref
watch(x, (newX) => {
  console.log(`x to ${newX}`)
})

// getter
watch(
  () => x.value + y.value,
  (sum) => {
    console.log(`sumą x + y jest: ${sum}`)
  }
)

// tablica wielu źródeł
watch([x, () => y.value], ([newX, newY]) => {
  console.log(`x to ${newX}, a y to ${newY}`)
})
```

Należy pamiętać, że nie można obserwować właściwości obiektu reaktywnego w ten sposób:

```js
const obj = reactive({ count: 0 })

// to nie zadziała, ponieważ przekazujemy liczbę do watch()
watch(obj.count, (count) => {
  console.log(`Liczba wynosi: ${count}`)
})
```

Zamiast tego użyj gettera:

```js
// zamiast tego użyj gettera:
watch(
  () => obj.count,
  (count) => {
    console.log(`Liczba wynosi: ${count}`)
  }
)
```

</div>

## Głębokie obserwatory (deep watchers) {#deep-watchers}

<div class="options-api">

`watch` is shallow by default: the callback will only trigger when the watched property has been assigned a new value - it won't trigger on nested property changes. If you want the callback to fire on all nested mutations, you need to use a deep watcher:

`watch` jest domyślnie płytki: funkcja zwrotna (callback) zostanie wywołana tylko wtedy, gdy obserwowana właściwość otrzyma nową wartość – nie zostanie wywołana przy zmianach w zagnieżdżonych właściwościach. Jeśli chcesz, aby funkcja zwrotna była wywoływana na wszystkich zagnieżdżonych mutacjach, musisz użyć głębokiego obserwatora:

```js
export default {
  watch: {
    someObject: {
      handler(newValue, oldValue) {
        // Uwaga: `newValue` będzie równe `oldValue` w przypadku
        // zagnieżdżonych mutacji, dopóki sam obiekt nie zostanie
        // zastąpiony.
      },
      deep: true
    }
  }
}
```

</div>

<div class="composition-api">

Gdy wywołasz `watch()` bezpośrednio na reaktywnym obiekcie, zostanie automatycznie stworzony głęboki obserwator – funkcja zwrotna zostanie wywołana przy każdej zmianie w zagnieżdżonych właściwościach:

```js
const obj = reactive({ count: 0 })

watch(obj, (newValue, oldValue) => {
  // wywołuje się przy mutacjach zagnieżdżonych właściwości
  // Uwaga: `newValue` będzie równe `oldValue`, ponieważ obydwa
  // wskazują na ten sam obiekt!
})

obj.count++
```

Należy to rozróżnić od getterów, które zwracają reaktywny obiekt – w takim przypadku funkcja zwrotna zostanie wywołana tylko wtedy, gdy getter zwróci inny obiekt:

```js
watch(
  () => state.someObject,
  () => {
    // wywołuje się tylko wtedy, gdy state.someObject zostanie zastąpione
  }
)
```

Możesz jednak wymusić, aby drugi przypadek działał jako głęboki watcher, używając opcji `deep`:

```js
watch(
  () => state.someObject,
  (newValue, oldValue) => {
    // Uwaga: `newValue` będzie równe `oldValue`, chyba że
    // state.someObject zostało zastąpione
  },
  { deep: true }
)
```

</div>

:::warning Używaj ostrożnie
Głębokie obserwowanie wymaga przejścia przez wszystkie zagnieżdżone właściwości w obserwowanym obiekcie i może być kosztowny w przypadku dużych struktur danych. Używaj go tylko wtedy, gdy jest to konieczne i uważaj na implikacje wydajnościowe.
:::

## Obserwatory natychmiastowe {#eager-watchers}

`watch` jest domyślnie leniwy (lazy): wywołanie zwrotne nie zostanie wywołane, dopóki obserwowane źródło nie ulegnie zmianie. Jednak w niektórych przypadkach możemy chcieć, aby ta sama logika wywołania zwrotnego była uruchamiana natychmiast — na przykład, gdy chcemy pobrać początkowe dane, a następnie ponownie pobrać je, gdy stan się zmieni.

<div class="options-api">

Możemy wymusić natychmiastowe wykonanie funkcji zwrotnej obserwatora, deklarując ją jako obiekt z funkcją `handler` oraz opcją `immediate: true`:

```js
export default {
  // ...
  watch: {
    question: {
      handler(newQuestion) {
        // ta funkcja zostanie uruchomiona natychmiast podczas tworzenia komponentu.
      },
      // wymusza natychmiastowe wykonanie funkcji zwrotnej
      immediate: true
    }
  }
  // ...
}
```

Początkowe wykonanie funkcji handlera nastąpi tuż przed wywołaniem haka `created`. Vue zdąży już przetworzyć opcje `data`, `computed` oraz `methods`, więc te właściwości będą dostępne przy pierwszym wywołaniu.

</div>

<div class="composition-api">

Możemy wymusić natychmiastowe wykonanie funkcji zwrotnej watchera, przekazując opcję `immediate: true`:

```js
watch(
  source,
  (newValue, oldValue) => {
    // wykona się natychmiast, a potem ponownie, gdy `source` się zmieni
  },
  { immediate: true }
)
```

</div>

## Obserwatory jednorazowe <sup class="vt-badge" data-text="3.4+" /> {#once-watchers}

Funkcja zwrotna watchera wykona się za każdym razem, gdy zmieni się obserwowane źródło. Jeśli chcesz, aby funkcja zwrotna została wywołana tylko raz, gdy źródło się zmieni, użyj opcji `once: true`.

<div class="options-api">

```js
export default {
  watch: {
    source: {
      handler(newValue, oldValue) {
        // gdy `source` się zmieni, funkcja wywoła się tylko raz
      },
      once: true
    }
  }
}
```

</div>

<div class="composition-api">

```js
watch(
  source,
  (newValue, oldValue) => {
    // gdy `source` się zmieni, wywoła się tylko raz
  },
  { once: true }
)
```

</div>

<div class="composition-api">

## `watchEffect()` \*\* {#watcheffect}

Często funkcja zwrotna obserwatora używa dokładnie tego samego reaktywnego stanu co źródło. Na przykład, rozważmy poniższy kod, który używa obserwatora do załadowania zasobów zdalnych, gdy zmienia się wartość `todoId`:

```js
const todoId = ref(1)
const data = ref(null)

watch(
  todoId,
  async () => {
    const response = await fetch(
      `https://jsonplaceholder.typicode.com/todos/${todoId.value}`
    )
    data.value = await response.json()
  },
  { immediate: true }
)
```

W szczególności zauważ, jak watcher używa `todoId` dwukrotnie: raz jako źródło, a następnie ponownie wewnątrz funkcji zwrotnej.

Można to uprościć za pomocą [`watchEffect()`](/api/reactivity-core#watcheffect). `watchEffect()` pozwala automatycznie śledzić zależności reaktywne funkcji zwrotnej. Powyższy watcher można zapisać w następujący sposób:

```js
watchEffect(async () => {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/todos/${todoId.value}`
  )
  data.value = await response.json()
})
```

Tutaj funkcja zwrotna wykona się natychmiast, nie trzeba używać `immediate: true`. Podczas jej wykonywania automatycznie zostanie śledzona `todoId.value` jako zależność (podobnie jak w przypadku właściwości obliczanych). Za każdym razem, gdy `todoId.value` się zmieni, funkcja zwrotna zostanie ponownie wywołana. Dzięki `watchEffect()` nie musimy jawnie przekazywać `todoId` jako źródła wartości.

Możesz zapoznać się z [tym przykładem](/examples/#fetching-data) użycia `watchEffect()` i reaktywnego pobierania danych w akcji.

Dla przykładów takich jak ten, z jedną zależnością, korzyści z użycia `watchEffect()` są stosunkowo niewielkie. Jednak w przypadku obserwatorów, które mają wiele zależności, użycie `watchEffect()` eliminuje konieczność ręcznego utrzymywania listy zależności. Ponadto, jeśli musisz śledzić wiele właściwości w zagnieżdżonej strukturze danych, `watchEffect()` może okazać się bardziej wydajny niż głęboki obserwator, ponieważ będzie śledził tylko te właściwości, które są używane w funkcji zwrotnej, zamiast rekurencyjnie śledzić wszystkie z nich.

:::tip
`watchEffect` śledzi zależności tylko podczas swojego **synchronicznego** wykonania. Podczas używania go z asynchronicznym wywołaniem zwrotnym, śledzone będą tylko właściwości, do których uzyskano dostęp przed pierwszym znacznikiem `await`.
:::

### `watch` vs. `watchEffect` {#watch-vs-watcheffect}

`watch` i `watchEffect` pozwalają na reaktywne wykonywanie efektów ubocznych. Ich główna różnica polega na sposobie śledzenia zależności reaktywnych:

- `watch` śledzi tylko jawnie obserwowane źródło. Nie śledzi niczego, co jest używane wewnątrz funkcji zwrotnej. Ponadto funkcja zwrotna wywołuje się tylko wtedy, gdy źródło faktycznie się zmieni. `watch` oddziela śledzenie zależności od efektów ubocznych, dając większą kontrolę nad tym, kiedy funkcja zwrotna powinna być wywołana.

- z kolei `watchEffect` łączy śledzenie zależności i efekt uboczny w jednej fazie. Automatycznie śledzi każdą reaktywną właściwość dostępną podczas swojej synchronizowanej egzekucji. Jest to wygodniejsze i zazwyczaj skutkuje zwięzłym kodem, ale sprawia, że ​​jego reaktywne zależności są mniej jednoznaczne.

</div>

## Czas wywołania funkcji zwrotnej {#callback-flush-timing}

Gdy modyfikujesz stan reaktywny, może to wywołać zarówno aktualizacje komponentów Vue, jak i funkcje zwrotne watchera, które utworzyłeś.

Podobnie jak w przypadku aktualizacji komponentów, wywołania zwrotne obserwatora tworzone przez użytkownika są grupowane, aby uniknąć duplikowania wywołań. Na przykład prawdopodobnie nie chcemy, aby obserwator uruchamiał się tysiąc razy, jeśli synchronicznie wprowadzimy tysiąc elementów do obserwowanej tablicy.

Domyślnie funkcja zwrotna obserwatora jest wywoływana **po** aktualizacjach komponentu nadrzędnego (jeśli takie są), a **przed** aktualizacjami DOM komponentu właściciela. Oznacza to, że jeśli spróbujesz uzyskać dostęp do DOM komponentu właściciela w wywołaniu zwrotnym obserwatora, DOM będzie w stanie przed aktualizacją.

### Obserwatory post {#post-watchers}

Jeśli chcesz uzyskać dostęp do DOM komponentu właściciela w wywołaniu zwrotnym obserwatora **po** jego zaktualizowaniu przez Vue, musisz określić opcję `flush: 'post'`:

<div class="options-api">

```js{6}
export default {
  // ...
  watch: {
    key: {
      handler() {},
      flush: 'post'
    }
  }
}
```

</div>

<div class="composition-api">

```js{2,6}
watch(source, callback, {
  flush: 'post'
})

watchEffect(callback, {
  flush: 'post'
})
```

Post-flush `watchEffect()` ma również wygodny alias, `watchPostEffect()`:

```js
import { watchPostEffect } from 'vue'

watchPostEffect(() => {
  /* wykonywane po aktualizacjach Vue */
})
```

</div>

### Obserwatory synchroniczne {#sync-watchers}

Można również utworzyć obserwator, który będzie uruchamiany synchronicznie, przed jakimikolwiek aktualizacjami zarządzanymi przez Vue:

<div class="options-api">

```js{6}
export default {
  // ...
  watch: {
    key: {
      handler() {},
      flush: 'sync'
    }
  }
}
```

</div>

<div class="composition-api">

```js{2,6}
watch(source, callback, {
  flush: 'sync'
})

watchEffect(callback, {
  flush: 'sync'
})
```

Synchroniczny `watchEffect()` ma również wygodny alias, `watchPostEffect()`:

```js
import { watchSyncEffect } from 'vue'

watchSyncEffect(() => {
  /* wykonywane synchronicznie po reaktywnej zmianie danych */
})
```

</div>

:::warning Używaj ostrożnie
Synchroniczne obserwatory nie mają dozowania i wyzwalają się za każdym razem, gdy zostanie wykryta reaktywna mutacja. Można ich używać do obserwowania prostych wartości logicznych, ale należy unikać używania ich w źródłach danych, które mogą być wielokrotnie mutowane synchronicznie, takich jak tablice.
:::

<div class="options-api">

## `this.$watch()` \* {#this-watch}

Możliwe jest również imperatywne tworzenie obserwatorów za pomocą [metody instancji `$watch()`](/api/component-instance#watch):

```js
export default {
  created() {
    this.$watch('question', (newQuestion) => {
      // ...
    })
  }
}
```

Jest to przydatne, gdy musisz warunkowo ustawić obserwator lub obserwować coś w odpowiedzi na interakcję użytkownika. Pozwala to również na wcześniejsze zatrzymanie obserwatora.

</div>

## Zatrzymywanie obserwatora {#stopping-a-watcher}

<div class="options-api">

Obserwatory zadeklarowane za pomocą opcji `watch` lub metody instancji `$watch()` są automatycznie zatrzymywane, gdy komponent właściciela jest usuwany, więc w większości przypadków nie musisz martwić się o zatrzymywanie obserwatora samodzielnie.

W rzadkich przypadkach, gdy musisz zatrzymać obserwator przed usunięciem komponentu właściciela, API `$watch()` zwraca w tym celu funkcję:

```js
const unwatch = this.$watch('foo', callback)

// ...gdy obserwator nie jest już potrzebny:
unwatch()
```

</div>

<div class="composition-api">

Obserwatory zadeklarowane synchronicznie w `setup()` lub `<script setup>` są związane z instancją komponentu właściciela i będą automatycznie zatrzymywane, gdy komponent właściciela zostanie usunięty. W większości przypadków nie musisz martwić się o zatrzymywanie obserwatora samodzielnie.

Kluczowe jest to, że obserwator musi być tworzony **synchronicznie**: jeśli obserwator zostanie stworzony w asynchronicznej funkcji zwrotnej, nie będzie związany z komponentem właściciela i będzie musiał zostać zatrzymany ręcznie, aby uniknąć wycieków pamięci. Oto przykład:

```vue
<script setup>
import { watchEffect } from 'vue'

// ten będzie automatycznie zatrzymany
watchEffect(() => {})

// ...ten nie będzie!

setTimeout(() => {
  watchEffect(() => {})
}, 100)
</script>
```

Aby ręcznie zatrzymać obserwator, użyj zwróconej funkcji obsługi. Działa to zarówno dla `watch`, jak i `watchEffect`:

```js
const unwatch = watchEffect(() => {})

// ...później, gdy nie jest już potrzebny
unwatch()
```

Należy zauważyć, że przypadków, w których trzeba tworzyć obserwatory asynchronicznie, powinno być bardzo mało, a tworzenie synchroniczne powinno być preferowane, gdy tylko to możliwe. Jeśli musisz poczekać na jakieś dane asynchroniczne, możesz zamiast tego sprawić, by logika obserwatora była warunkowa:

```js
// dane do załadowania asynchronicznie
const data = ref(null)

watchEffect(() => {
  if (data.value) {
    // wykonaj coś, gdy dane zostaną załadowane
  }
})
```

</div>
