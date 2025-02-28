# Powiązania klas i stylów {#class-and-style-bindings}

Częstą potrzebą w wiązaniu danych jest manipulowanie listą klas elementu i stylami inline. Ponieważ `class` i `style` są atrybutami, możemy użyć `v-bind`, aby dynamicznie przypisać im wartość tekstową, podobnie jak w przypadku innych atrybutów. Jednak generowanie tych wartości poprzez powiązanie łańcuchów znaków może być uciążliwe i podatne na błędy. Z tego powodu Vue zapewnia specjalne ulepszenia, gdy `v-bind` jest używany z `class` i `style`. Oprócz łańcuchów znaków, wyrażenia mogą również zwracać obiekty lub tablice.

## Wiązanie klas HTML {#binding-html-classes}

<div class="options-api">
  <VueSchoolLink href="https://vueschool.io/lessons/dynamic-css-classes-with-vue-3" title="Darmowa lekcja Vue.js o dynamicznych klasach CSS"/>
</div>

<div class="composition-api">
  <VueSchoolLink href="https://vueschool.io/lessons/vue-fundamentals-capi-dynamic-css-classes-with-vue" title="Darmowa lekcja Vue.js o dynamicznych klasach CSS"/>
</div>

### Wiązanie z obiektami {#binding-to-objects}

Możemy przekazać obiekt do `:class` (skrót od `v-bind:class`), aby dynamicznie przełączać klasy:

```vue-html
<div :class="{ active: isActive }"></div>
```

Powyższa składnia oznacza, że obecność klasy `active` będzie zależała od [prawdziwości (truthiness)](https://developer.mozilla.org/en-US/docs/Glossary/Truthy) właściwości danej `isActive`.

Możesz przełączać wiele klas, dodając więcej pól do obiektu. Dodatkowo dyrektywa `:class` może współistnieć ze zwykłym atrybutem `class`. Przyjmując następujący stan:

<div class="composition-api">

```js
const isActive = ref(true)
const hasError = ref(false)
```

</div>

<div class="options-api">

```js
data() {
  return {
    isActive: true,
    hasError: false
  }
}
```

</div>

Oraz następujący szablon:

```vue-html
<div
  class="static"
  :class="{ active: isActive, 'text-danger': hasError }"
></div>
```

Wyrenderuje to:

```vue-html
<div class="static active"></div>
```

Kiedy wartości `isActive` lub `hasError` się zmienią, lista klas zostanie odpowiednio zaktualizowana. Na przykład, jeśli `hasError` zmieni się na `true`, lista klas stanie się `"static active text-danger"`.

Obiekt powiązany nie musi być zapisany inline:

<div class="composition-api">

```js
const classObject = reactive({
  active: true,
  'text-danger': false
})
```

</div>

<div class="options-api">

```js
data() {
  return {
    classObject: {
      active: true,
      'text-danger': false
    }
  }
}
```

</div>

```vue-html
<div :class="classObject"></div>
```

This will render:

```vue-html
<div class="active"></div>
```

Możemy także powiązać klasę z [właściwością computed](./computed), która zwraca obiekt. Jest to częsty i skuteczny wzorzec:

<div class="composition-api">

```js
const isActive = ref(true)
const error = ref(null)

const classObject = computed(() => ({
  active: isActive.value && !error.value,
  'text-danger': error.value && error.value.type === 'fatal'
}))
```

</div>

<div class="options-api">

```js
data() {
  return {
    isActive: true,
    error: null
  }
},
computed: {
  classObject() {
    return {
      active: this.isActive && !this.error,
      'text-danger': this.error && this.error.type === 'fatal'
    }
  }
}
```

</div>

```vue-html
<div :class="classObject"></div>
```

### Wiązanie z tablicami {#binding-to-arrays}

Możemy powiązać `:class` z tablicą, aby zastosować listę klas:

<div class="composition-api">

```js
const activeClass = ref('active')
const errorClass = ref('text-danger')
```

</div>

<div class="options-api">

```js
data() {
  return {
    activeClass: 'active',
    errorClass: 'text-danger'
  }
}
```

</div>

```vue-html
<div :class="[activeClass, errorClass]"></div>
```

Co wyrenderuje:

```vue-html
<div class="active text-danger"></div>
```

Jeśli chcesz warunkowo dodać klasę do listy, możesz użyć wyrażenia trójskładnikowego (ternary expression):

```vue-html
<div :class="[isActive ? activeClass : '', errorClass]"></div>
```

To zawsze będzie dotyczyć `errorClass`, ale `activeClass` będzie stosowane tylko wtedy, gdy `isActive` będzie true.

Jednak może to być trochę rozwlekłe, jeśli masz wiele klas warunkowych. Dlatego możliwe jest również użycie składni obiektu wewnątrz składni tablicy:

```vue-html
<div :class="[{ [activeClass]: isActive }, errorClass]"></div>
```

### Z komponentami {#with-components}

> Ta sekcja zakłada znajomość [komponentów](/guide/essentials/component-basics). Możesz ją pominąć i wrócić do niej później.

Kiedy używasz atrybutu `class` w komponencie z pojedynczym elementem root, te klasy zostaną dodane do elementu root komponentu i połączone z już istniejącymi klasami.

Na przykład, jeśli mamy komponent o nazwie `MyComponent` z następującym szablonem:

```vue-html
<!-- szablon komponentu dziecka -->
<p class="foo bar">Hi!</p>
```

Następnie dodajemy klasy przy jego użyciu:

```vue-html
<!-- podczas używania komponentu  -->
<MyComponent class="baz boo" />
```

Wygenerowany HTML będzie wyglądał następująco:

```vue-html
<p class="foo bar baz boo">Cześć!</p>
```

To samo dotyczy powiązań klas:

```vue-html
<MyComponent :class="{ active: isActive }" />
```

Kiedy `isActive` jest wartością truthy, wygenerowany HTML będzie wyglądał następująco:

```vue-html
<p class="foo bar active">Cześć!</p>
```

Jeśli Twój komponent ma wiele elementów root, musisz określić, który element otrzyma tę klasę. Możesz to zrobić używając właściwości `$attrs` komponentu:

```vue-html
<!--  Szablon komponentu MyComponent z użyciem $attrs -->
<p :class="$attrs.class">Cześć!</p>
<span>This is a child component</span>
```

```vue-html
<MyComponent class="baz" />
```

Wyrenderuje to:

```html
<p class="baz">Cześć!</p>
<span>To jest komponent dziecka</span>
```

Możesz dowiedzieć się więcej o dziedziczeniu atrybutów komponentów w sekcji [atrybuty przechodzące](/guide/components/attrs).

## Wiązanie stylów inline {#binding-inline-styles}

### Wiązanie z obiektami {#binding-to-objects-1}

`:style` obsługuje wiązanie z wartościami obiektów JavaScript — odpowiada [właściwości `style` elementu HTML](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/style):

<div class="composition-api">

```js
const activeColor = ref('red')
const fontSize = ref(30)
```

</div>

<div class="options-api">

```js
data() {
  return {
    activeColor: 'red',
    fontSize: 30
  }
}
```

</div>

```vue-html
<div :style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
```

Chociaż zalecane są klucze pisane z użyciem camelCase, `:style` obsługuje również klucze właściwości CSS w formacie kebab (odpowiada to sposobowi ich użycia w rzeczywistym CSS) - na przykład:

```vue-html
<div :style="{ 'font-size': fontSize + 'px' }"></div>
```

Często dobrym pomysłem jest bezpośrednie powiązanie z obiektem stylu, aby szablon był czystszy:

<div class="composition-api">

```js
const styleObject = reactive({
  color: 'red',
  fontSize: '30px'
})
```

</div>

<div class="options-api">

```js
data() {
  return {
    styleObject: {
      color: 'red',
      fontSize: '13px'
    }
  }
}
```

</div>

```vue-html
<div :style="styleObject"></div>
```

Ponownie, powiązanie stylu obiektu jest często używane w połączeniu z obliczonymi właściwościami, które zwracają obiekty.

Dyrektywy `:style` mogą również współistnieć ze zwykłymi atrybutami stylów, tak jak `:class`.

Szablon:

```vue-html
<h1 style="color: red" :style="'font-size: 1em'">cześć</h1>
```

Wyrenderuje:

```vue-html
<h1 style="color: red; font-size: 1em;">cześć</h1>
```

### Wiązanie z tablicami {#binding-to-arrays-1}

Możemy powiązać `:style` z tablicą wielu obiektów stylu. Te obiekty zostaną scalone i zastosowane do tego samego elementu:

```vue-html
<div :style="[baseStyles, overridingStyles]"></div>
```

### Automatyczne dodawanie prefiksu {#auto-prefixing}

Gdy używasz właściwości CSS, która wymaga [prefiksu dostawcy](https://developer.mozilla.org/en-US/docs/Glossary/Vendor_Prefix) w `:style`, Vue automatycznie doda odpowiedni prefiks. Vue robi to, sprawdzając w czasie wykonywania, które właściwości stylu są obsługiwane w bieżącej przeglądarce. Jeśli przeglądarka nie obsługuje określonej właściwości, zostaną przetestowane różne warianty z prefiksem, aby spróbować znaleźć taką, która jest obsługiwana.

### Wiele wartości {#multiple-values}

Możesz podać tablicę wielu (z prefiksem) wartości do właściwości stylu, na przykład:

```vue-html
<div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
```

To spowoduje wyświetlenie tylko ostatniej wartości w tablicy obsługiwanej przez przeglądarkę. W tym przykładzie zostanie wyświetlona wartość `display: flex` dla przeglądarek obsługujących wersję flexbox bez prefiksu.
