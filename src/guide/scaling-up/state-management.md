# Zarządzanie stanem {#state-management}

## Czym jest zarządzanie stanem? {#what-is-state-management}

Z praktycznego punktu widzenia, każdy komponent "zarządza" swoim reaktywnym stanem. Weźmy na przykład prosty komponent licznika:

<div class="composition-api">

```vue
<script setup>
import { ref } from 'vue'

// state
const count = ref(0)

// actions
function increment() {
  count.value++
}
</script>

<!-- view -->
<template>{{ count }}</template>
```

</div>
<div class="options-api">

```vue
<script>
export default {
  // state
  data() {
    return {
      count: 0
    }
  },
  // actions
  methods: {
    increment() {
      this.count++
    }
  }
}
</script>

<!-- view -->
<template>{{ count }}</template>
```

</div>

Ten komponent to samodzielna jednostka składająca się z następujących części:

- **Stanu**, źródła prawdy które jest kluczowe dla naszej aplikacji;
- **Widoku**, deklaratywnie zmapowanego **stanu**;
- **Akcji**, czyli możliwych sposobów zmiany stanu poprzez akcje użytkownika w warstwie **widoku**.

Poniższy obrazek ilustruje uproszczoną koncepcję "jednokierunkowego przepływu danych":

<p style="text-align: center">
  <img alt="diagram przepływu danych" src="./images/state-flow.png" width="252px" style="margin: 40px auto">
</p>

Jednakże, sytuacja komplikuje się gdy mamy doczynienia z **wieloma komponentami współdzielącymi stan**:

1. Wiele widoków może polegać na tych samych danych.
2. Akcje z różnych widoków mogą modyfikować te same dane.

W przypadku pierwszym, obejściem na ten problem jest "wyniesienie" wspólnego stanu do wspólnego komponentu rodzica wyżej, a następnie przekazywanie go do komponentów dzieci przez propsy. Trzeba mieć jednak na uwadze, że to podejście staje się uciążliwe wraz z bardziej skomplikowanymi hierarchiami komponentów, skutkując problemem zwanym ["Prop Drilling"](/guide/components/provide-inject#prop-drilling).

W drugim przypadku, możemy próbować odwoływać się bezpośrednio do instacji rodzica / dziecka przez template refs, lub modyfikować i synchronizować wiele kopii tego samego stanu poprzez emitowanie eventów. Oba podejścia są jedynie obejściami i bardzo szybko skutkują skomplikowanym i trudnym do utrzymania kodem.

Prostszym i bardziej przejrzystym rozwiązaniem jest wyodrębnienie wspólnego stanu poza komponenty, i zarządzanie nim z poziomu globalnego singletona. Dzięki temu całe nasze drzewo komponentów możemy uznać za jeden wielki "widok", gdzie każdy komponent ma dostęp do stanu oraz może go modyfikować niezależnie od jego miejsca w drzewie.

## Proste zarządzanie stanem z użyciem Reactivity API {#simple-state-management-with-reactivity-api}

<div class="options-api">

W Options API, reaktywne dane deklarowane są poprzez opcję `data()`. Następnie, obiekt zwrócony przez `data()` jest reaktywny dzięki wewnętrznemu zastosowaniu funkcji [`reactive()`](/api/reactivity-core#reactive), która jest również dostępna w publicznym API.

</div>

Jeśli masz kawałek stanu który musi być współdzielony między wieloma instancjami, możesz użyć funkcji [`reactive()`](/api/reactivity-core#reactive), aby stworzyć reaktywny obiekt a następnie zaimportować go do wielu komponentów:

```js
// store.js
import { reactive } from 'vue'

export const store = reactive({
  count: 0
})
```

<div class="composition-api">

```vue
<!-- ComponentA.vue -->
<script setup>
import { store } from './store.js'
</script>

<template>Komponent A: {{ store.count }}</template>
```

```vue
<!-- ComponentB.vue -->
<script setup>
import { store } from './store.js'
</script>

<template>Komponent B: {{ store.count }}</template>
```

</div>
<div class="options-api">

```vue
<!-- ComponentA.vue -->
<script>
import { store } from './store.js'

export default {
  data() {
    return {
      store
    }
  }
}
</script>

<template>Komponent A: {{ store.count }}</template>
```

```vue
<!-- ComponentB.vue -->
<script>
import { store } from './store.js'

export default {
  data() {
    return {
      store
    }
  }
}
</script>

<template>Komponent B: {{ store.count }}</template>
```

</div>

Teraz, gdy `store` zostanie zmodyfikowane, zarówno `<ComponentA>` jak i `<ComponentB>` zaktualizują swoje widoki automatycznie - mamy pojedyncze źródło prawdy.

Jednakże, oznacza to również że kazdy komponent importujący `store` może go dowolnie zmieniać:

```vue-html{2}
<template>
  <button @click="store.count++">
    Komponent B: {{ store.count }}
  </button>
</template>
```

Działa to bez zarzutu w prostych przypadkach, jednakże globalny stan mogący być dowolnie zmieniany przez każdy komponent nie będzie łatwy do utrzymania na dłuższą metę. Aby zapewnić zcentralizowanie modyfikacji stanu tak jak i sam stan, zalecane jest zdefiniowanie metod na stanie, z nazwami jasno określającymi intencje danych akcji:

```js{6-8}
// store.js
import { reactive } from 'vue'

export const store = reactive({
  count: 0,
  increment() {
    this.count++
  }
})
```

```vue-html{2}
<template>
  <button @click="store.increment()">
    Komponent B: {{ store.count }}
  </button>
</template>
```

<div class="composition-api">

[Wypróbuj w Piaskownicy](https://play.vuejs.org/#eNrNkk1uwyAQha8yYpNEiUzXllPVrtRTeJNSqtLGgGBsVbK4ewdwnT9FWWSTFczwmPc+xMhqa4uhl6xklRdOWQQvsbfPrVadNQ7h1dCqpcYaPp3pYFHwQyteXVxKm0tpM0krnm3IgAqUnd3vUFIFUB1Z8bNOkzoVny+wDTuNcZ1gBI/GSQhzqlQX3/5Gng81pA1t33tEo+FF7JX42bYsT1BaONlRguWqZZMU4C261CWMk3EhTK8RQphm8Twse/BscoUsvdqDkTX3kP3nI6aZwcmdQDUcMPJPabX8TQphtCf0RLqd1csxuqQAJTxtYnEUGtIpAH4pn1Ou17FDScOKhT+QNAVM)

</div>
<div class="options-api">

[Wypróbuj w Piaskownicy](https://play.vuejs.org/#eNrdU8FqhDAU/JVHLruyi+lZ3FIt9Cu82JilaTWR5CkF8d8bE5O1u1so9FYQzAyTvJnRTKTo+3QcOMlIbpgWPT5WUnS90gjPyr4ll1jAWasOdim9UMum3a20vJWWqxSgkvzTyRt+rocWYVpYFoQm8wRsJh+viHLBcyXtk9No2ALkXd/WyC0CyDfW6RVTOiancQM5ku+x7nUxgUGlOcwxn8Ppu7HJ7udqaqz3SYikOQ5aBgT+OA9slt9kasToFnb5OiAqCU+sFezjVBHvRUimeWdT7JOKrFKAl8VvYatdI6RMDRJhdlPtWdQf5mdQP+SHdtyX/IftlH9pJyS1vcQ2NK8ZivFSiL8BsQmmpMG1s1NU79frYA1k8OD+/I3pUA6+CeNdHg6hmoTMX9pPSnk=)

</div>

:::tip
Zwróć uwagę że handler eventu click używa `store.increment()` z nawiasami - jest to konieczne, aby wywołać tą metodę z prawidłowym kontekstem `this` gdyż nie jest to metoda komponentu.
:::

Podczas gdy w tych przypadkach używamy pojedynczego reaktywnego obiektu jako naszego globalnego stanu, warto pamiętać że możemy go równie dobrze utworzyć korzystając z innych [API reaktywności](/api/reactivity-core) takich jak `ref()` czy `computed()`, albo nawet zwrócić stan globalny z utworzonego [composable](/guide/reusability/composables):

```js
import { ref } from 'vue'

// stan globalny, utworzony w obrębie modułu
const globalCount = ref(1)

export function useCount() {
  // stan lokalny, tworzony per komponent
  const localCount = ref(1)

  return {
    globalCount,
    localCount
  }
}
```

Fakt, że moduł reaktywności w Vue jest wyodrębniony od samego modelu komponentów sprawia, że jest on bardzo elastyczny.

## Rozważania względem SSR {#ssr-considerations}

Jeśli budujesz aplikację, która wykorzystuje [renderowanie po stronie serwera (SSR)](./ssr) to powyższe podejście może prowadzić do różnego rodzaju problemów, ponieważ stan będącym takim singletonem jest współdzielony między zapytaniami do serwera. Omawiamy ten problem [dokładniej](./ssr#cross-request-state-pollution) w poradniku odnośnie SSR.

## Pinia {#pinia}

Podczas gdy powyższe podejście jest wystarczające w prostych przypadkach, istnieje wiele różnych czynników które warto rozważyć w przypadku produkcyjnych aplikacji o dużej skali:

- Ustandaryzowane konwencje dla pracy zespołowej
- Zintegrowanie z narzędziami deweloperskimi Vue, wliczając timeline, analizę w środku komponentu oraz tzw. "time-travel debugging"
- Hot Module Replacement
- Wsparcie dla renderowania po stronie serwera (SSR)

[Pinia](https://pinia.vuejs.org) to biblioteka do zarządzania stanem, która adresuje powyższe kwestie. Jest ona wspierana przez główny zespół Vue i wspiera zarówno Vue 2 jak i Vue 3.

Użytkownicy z dłuższym stażem, mogą kojarzyć również [Vuex](https://vuex.vuejs.org/), poprzednią oficjalną bibliotekę do zarządzania stanem w Vue. Pinia pełni tę samą rolę w ekosystemie, a Vuex jest obecnie w fazie maintenance. Nadal działa, ale nie otrzyma już żadnych nowych funkcjonalności. Rekomendujemy używanie Pinia w nowych aplikacjach.

Pinia zaczęła się jako eksploracja tego, jak kolejna iteracja Vuexa może wyglądać, wdrażając wiele z pomysłów z dyskusji zespołu co do Vuexa w wersji 5. Z czasem, zdaliśmy sobie sprawę, że Pinia już implementuje większość tego co chcieliśmy dostarczyć w tej wersji Vuexa i zdecydowaliśmy się by Pinia stała się nową rekomendacją.

W porównaniu do Vuexa, Pinia oferuje znacznie prostsze w użyciu API, oferuje API w stylu composition API i co najważniejsze - oferuje dobre wsparcie dla wnioskowania typów gdy jest używana razem z TypeScript.
