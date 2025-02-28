# Referencje w szablonach {#template-refs}

Chociaż deklaratywny model renderowania Vue abstrahuje większość operacji na DOM, mogą zdarzyć się przypadki, w których potrzebujemy bezpośredniego dostępu do elementów DOM. Aby to osiągnąć, możemy użyć specjalnego atrybutu `ref`:

```vue-html
<input ref="input">
```

`ref` to specjalny atrybut, podobny do atrybutu `key` omawianego w rozdziale `v-for`. Pozwala nam uzyskać bezpośrednie referencje do konkretnego elementu DOM lub instancji komponentu dziecka po jej zamontowaniu. Może to być przydatne, gdy chcemy na przykład programowo ustawić fokus na inpucie podczas montowania komponentu lub zainicjować bibliotekę zewnętrzną na elemencie.

## Dostęp do referencji {#accessing-the-refs}

<div class="composition-api">

Aby uzyskać referencje do elementu przy użyciu Composition API, musimy zadeklarować ref o nazwie pasującej do wartości atrybutu ref w szablonie:

```vue
<script setup>
import { ref, onMounted } from 'vue'

// zadeklaruj ref do przechowywania referencji do elementu
// nazwa musi pasować do wartości atrybutu ref w szablonie
const input = ref(null)

onMounted(() => {
  input.value.focus()
})
</script>

<template>
  <input ref="input" />
</template>
```

Jeśli nie używasz `<script setup>`, upewnij się, że zwrócisz ref z funkcji `setup()`:

```js{6}
export default {
  setup() {
    const input = ref(null)
    // ...
    return {
      input
    }
  }
}
```

</div>
<div class="options-api">

Wynikowa referencja jest dostępne przez `this.$refs`:

```vue
<script>
export default {
  mounted() {
    this.$refs.input.focus()
  }
}
</script>

<template>
  <input ref="input" />
</template>
```

</div>

Należy pamiętać, że dostęp do referencji jest możliwy dopiero **po zamontowaniu komponentu**. Jeśli spróbujesz uzyskać dostęp do <span class="options-api">$refs.input</span><span class="composition-api">`input`</span> w wyrażeniu szablonu, będzie to <span class="options-api">`undefined`</span><span class="composition-api">`null`</span> przy pierwszym renderowaniu. Dzieje się tak, ponieważ element nie istnieje aż do pierwszego renderowania!

<div class="composition-api">

Jeśli próbujesz obserwować zmiany referencji w szablonie, upewnij się, że uwzględniasz przypadek, gdy referencja ma wartość `null`:

```js
watchEffect(() => {
  if (input.value) {
    input.value.focus()
  } else {
    // nie jest jeszcze zamontowany, lub element został odmontowany (np. przez v-if)
  }
})
```

Zobacz także: [Typowanie referencji w szablonie](/guide/typescript/composition-api#typing-template-refs) <sup class="vt-badge ts" />

</div>

## Referencje wewnątrz `v-for` {#refs-inside-v-for}

> Wymaga wersji v3.2.25 lub wyższej

<div class="composition-api">

When `ref` is used inside `v-for`, the corresponding ref should contain an Array value, which will be populated with the elements after mount:

Gdy `ref` jest używane wewnątrz `v-for`, odpowiednia referencja powinna zawierać wartość tablicy, która zostanie wypełniona elementami po zamontowaniu:

```vue
<script setup>
import { ref, onMounted } from 'vue'

const list = ref([
  /* ... */
])

const itemRefs = ref([])

onMounted(() => console.log(itemRefs.value))
</script>

<template>
  <ul>
    <li v-for="item in list" ref="itemRefs">
      {{ item }}
    </li>
  </ul>
</template>
```

[Wypróbuj w playground](https://play.vuejs.org/#eNpFjs1qwzAQhF9l0CU2uDZtb8UOlJ576bXqwaQyCGRJyCsTEHr3rGwnOehnd2e+nSQ+vW/XqMSH6JdL0J6wKIr+LK2evQuEhKCmBs5+u2hJ/SNjCm7GiV0naaW9OLsQjOZrKNrq97XBW4P3v/o51qTmHzUtd8k+e0CrqsZwRpIWGI0KVN0N7TqaqNp59JUuEt2SutKXY5elmimZT9/t2Tk1F+z0ZiTFFdBHs738Mxrry+TCIEWhQ9sttRQl0tEsK6U4HEBKW3LkfDA6o3dst3H77rFM5BtTfm/P)

</div>
<div class="options-api">

Gdy `ref` jest używane wewnątrz `v-for`, wynikowa wartość referencji będzie tablicą zawierającą odpowiednie elementy:

```vue
<script>
export default {
  data() {
    return {
      list: [
        /* ... */
      ]
    }
  },
  mounted() {
    console.log(this.$refs.items)
  }
}
</script>

<template>
  <ul>
    <li v-for="item in list" ref="items">
      {{ item }}
    </li>
  </ul>
</template>
```

[Wypróbuj w playground](https://play.vuejs.org/#eNpFjk0KwjAQha/yCC4Uaou6kyp4DuOi2KkGYhKSiQildzdNa4WQmTc/37xeXJwr35HEUdTh7pXjszT0cdYzWuqaqBm9NEDbcLPeTDngiaM3PwVoFfiI667AvsDhNpWHMQzF+L9sNEztH3C3JlhNpbaPNT9VKFeeulAqplfY5D1p0qurxVQSqel0w5QUUEedY8q0wnvbWX+SYgRAmWxIiuSzm4tBinkc6HvkuSE7TIBKq4lZZWhdLZfE8AWp4l3T)

</div>

Należy zauważyć, że tablica referencji **nie** gwarantuje tej samej kolejności, co tablica źródłowa.

## Referencje funkcyjne {#function-refs}

Zamiast nazwy klucza, atrybut `ref` może być również powiązany z funkcją, która będzie wywoływana przy każdej aktualizacji komponentu i daje pełną elastyczność w przechowywaniu referencji do elementu. Funkcja ta otrzymuje referencje do elementu jako pierwszy argument:

```vue-html
<input :ref="(el) => { /* przypisz el do właściwości lub referencji */ }">
```

Zauważ, że używamy dynamicznego powiązania `:ref`, aby przekazać funkcję zamiast ciągu znaków jako nazwę referencji. Gdy element zostanie odmontowany, argument będzie `null`. Oczywiście można użyć metody zamiast funkcji w linii.

## Referencje na komponencie {#ref-on-component}

> Ta sekcja zakłada znajomość [komponentów](/guide/essentials/component-basics). Jeśli chcesz, możesz ją pominąć i wrócić później.

`ref` może być również używane na komponencie dziecka. W takim przypadku referencja będzie dotyczyła instancji komponentu:

<div class="composition-api">

```vue
<script setup>
import { ref, onMounted } from 'vue'
import Child from './Child.vue'

const child = ref(null)

onMounted(() => {
  // child.value będzie przechowywać instancję <Child />
})
</script>

<template>
  <Child ref="child" />
</template>
```

</div>
<div class="options-api">

```vue
<script>
import Child from './Child.vue'

export default {
  components: {
    Child
  },
  mounted() {
    // this.$refs.child będzie przechowywać instancję <Child />
  }
}
</script>

<template>
  <Child ref="child" />
</template>
```

</div>

<span class="composition-api">Jeśli komponent dziecka używa Options API lub nie korzysta ze `<script setup>`,</span><span class="options-api">Wtedy</span> referowana instancja będzie identyczna z `this` komponentu dziecka, co oznacza, że komponent nadrzędny będzie miał pełny dostęp do wszystkich właściwości i metod komponentu dziecka. Ułatwia to tworzenie ściśle powiązanych szczegółów implementacyjnych między rodzicem a dzieckiem, więc referencje do komponentów należy stosować tylko wtedy, gdy jest to absolutnie konieczne – w większości przypadków należy spróbować zaimplementować interakcje rodzic-dziecko przy użyciu standardowych interfejsów props i emit.

<div class="composition-api">

Wyjątkiem są komponenty używające `<script setup>`, które są **domyślnie prywatne**: komponent nadrzędny, który odwołuje się do komponentu dziecka używającego `<script setup>`, nie będzie miał dostępu do niczego, chyba że komponent dziecka zdecyduje się ujawnić publiczny interfejs za pomocą makro `defineExpose`:

```vue
<script setup>
import { ref } from 'vue'

const a = 1
const b = ref(2)

// Makra kompilatora, takie jak defineExpose, nie muszą być importowane
defineExpose({
  a,
  b
})
</script>
```

Kiedy komponent nadrzędny uzyska instancję tego komponentu przez referencje do szablonu, zwrócona instancja będzie miała postać `{ a: number, b: number }` (referencje są automatycznie rozpakowywane tak jak na zwykłych instancjach).

Należy pamiętać, że defineExpose musi być wywołane przed jakąkolwiek operacją await. W przeciwnym razie właściwości i metody ujawnione po operacji await nie będą dostępne.

Zobacz także: [Typowanie referencji w szablonach komponentów](/guide/typescript/composition-api#typing-component-template-refs) <sup class="vt-badge ts" />

</div>
<div class="options-api">

Opcja `expose` może być użyta, aby ograniczyć dostęp do instancji komponentu:

```js
export default {
  expose: ['publicData', 'publicMethod'],
  data() {
    return {
      publicData: 'foo',
      privateData: 'bar'
    }
  },
  methods: {
    publicMethod() {
      /* ... */
    },
    privateMethod() {
      /* ... */
    }
  }
}
```

W powyższym przykładzie, komponent nadrzędny, który odwołuje się do tego komponentu za pomocą referencji w szablonie, będzie miał dostęp tylko do `publicData` i `publicMethod`.

</div>
