# Vue i Web Components {#vue-and-web-components}

[Web Components](https://developer.mozilla.org/en-US/docs/Web/Web_Components) to ogólny termin określający zestaw natywnych API webowych, które pozwalają programistom tworzyć wielokrotnego użytku elementy niestandardowe.

Uważamy Vue i Web Components za przede wszystkim technologie komplementarne. Vue ma doskonałe wsparcie zarówno dla wykorzystywania, jak i tworzenia elementów niestandardowych. Niezależnie od tego, czy integrujesz elementy niestandardowe w istniejącej aplikacji Vue, czy używasz Vue do budowania i dystrybucji elementów niestandardowych, jesteś w dobrym towarzystwie.

## Używanie elementów niestandardowych w Vue {#using-custom-elements-in-vue}

Vue [zdobywa perfekcyjne 100% w testach Custom Elements Everywhere](https://custom-elements-everywhere.com/libraries/vue/results/results.html). Wykorzystywanie elementów niestandardowych wewnątrz aplikacji Vue działa w dużej mierze tak samo jak używanie natywnych elementów HTML, z kilkoma rzeczami, o których należy pamiętać:

### Pomijanie rozwiązywania komponentów {#skipping-component-resolution}

Domyślnie Vue będzie próbować rozwiązać nienatywny tag HTML jako zarejestrowany komponent Vue, zanim powróci do renderowania go jako elementu niestandardowego. Spowoduje to, że Vue wyemituje ostrzeżenie "failed to resolve component" podczas rozwoju. Aby poinformować Vue, że określone elementy powinny być traktowane jako elementy niestandardowe i pominąć rozwiązywanie komponentów, możemy określić opcję [`compilerOptions.isCustomElement`](/api/application#app-config-compileroptions).

Jeśli używasz Vue z konfiguracją budowania, opcja powinna być przekazana przez konfigurację budowania, ponieważ jest to opcja czasu kompilacji.

#### Przykład konfiguracji w przeglądarce {#example-in-browser-config}

```js
// Działa tylko przy kompilacji w przeglądarce.
// Jeśli używasz narzędzi do budowania, zobacz przykłady konfiguracji poniżej.
app.config.compilerOptions.isCustomElement = (tag) => tag.includes('-')
```

#### Przykładowa konfiguracja Vite {#example-vite-config}

```js
// vite.config.js
import vue from '@vitejs/plugin-vue'

export default {
  plugins: [
    vue({
      template: {
        compilerOptions: {
          // traktuj wszystkie tagi z myślnikiem jako elementy niestandardowe
          isCustomElement: (tag) => tag.includes('-')
        }
      }
    })
  ]
}
```

#### Przykładowa konfiguracja Vue CLI {#example-vue-cli-config}

```js
// vue.config.js
module.exports = {
  chainWebpack: config => {
    config.module
      .rule('vue')
      .use('vue-loader')
      .tap(options => ({
        ...options,
        compilerOptions: {
          // traktuj wszystkie tagi z myślnikiem jako elementy niestandardowe
          isCustomElement: tag => tag.startsWith('ion-')
        }
      }))
  }
}
```

### Przekazywanie właściwości DOM {#passing-dom-properties}

Ponieważ atrybuty DOM mogą być tylko ciągami znaków, musimy przekazywać złożone dane do elementów niestandardowych jako właściwości DOM. Podczas ustawiania właściwości (props) na elemencie niestandardowym, Vue 3 automatycznie sprawdza obecność właściwości DOM za pomocą operatora `in` i będzie preferować ustawienie wartości jako właściwość DOM, jeśli klucz jest obecny. Oznacza to, że w większości przypadków nie będziesz musiał o tym myśleć, jeśli element niestandardowy przestrzega [zalecanych najlepszych praktyk](https://web.dev/custom-elements-best-practices/).

Mogą jednak wystąpić rzadkie przypadki, w których dane muszą zostać przekazane jako właściwość DOM, ale element niestandardowy nie definiuje/nie odzwierciedla prawidłowo tej właściwości (powodując niepowodzenie sprawdzenia `in`). W takim przypadku możesz wymusić ustawienie wiązania `v-bind` jako właściwości DOM używając modyfikatora `.prop`:

```vue-html
<my-element :user.prop="{ name: 'jack' }"></my-element>

<!-- skrócony odpowiednik -->
<my-element .user="{ name: 'jack' }"></my-element>
```

## Budowanie niestandardowych elementów za pomocą Vue {#building-custom-elements-with-vue}

Główną zaletą niestandardowych elementów jest to, że mogą być używane z dowolnym frameworkiem lub nawet bez frameworka. Sprawia to, że są idealne do dystrybucji komponentów, gdzie końcowy użytkownik może nie używać tego samego stacku frontendowego lub gdy chcesz odizolować aplikację końcową od szczegółów implementacji używanych przez nią komponentów.

### defineCustomElement {#definecustomelement}

Vue wspiera tworzenie niestandardowych elementów przy użyciu dokładnie tych samych API komponentów Vue za pomocą metody [`defineCustomElement`](/api/general#definecustomelement). Metoda przyjmuje ten sam argument co [`defineComponent`](/api/general#definecomponent), ale zamiast tego zwraca konstruktor niestandardowego elementu, który rozszerza `HTMLElement`:

```vue-html
<my-vue-element></my-vue-element>
```

```js
import { defineCustomElement } from 'vue'

const MyVueElement = defineCustomElement({
  // tutaj normalne opcje komponentu Vue
  props: {},
  emits: {},
  template: `...`,

  // tylko defineCustomElement: CSS do wstrzyknięcia do shadow root
  styles: [`/* wbudowany css */`]
})

// Zarejestruj niestandardowy element.
// Po rejestracji wszystkie tagi `<my-vue-element>`
// na stronie zostaną zaktualizowane.
customElements.define('my-vue-element', MyVueElement)

// Możesz również programowo utworzyć instancję elementu:
// (można to zrobić tylko po rejestracji)
document.body.appendChild(
  new MyVueElement({
    // początkowe właściwości (opcjonalne)
  })
)
```

#### Cykl życia {#lifecycle}

- Niestandardowy element Vue zamontuje wewnętrzną instancję komponentu Vue wewnątrz swojego shadow root, gdy [`connectedCallback`](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements#using_the_lifecycle_callbacks) elementu zostanie wywołany po raz pierwszy.

- Kiedy `disconnectedCallback` elementu zostanie wywołany, Vue sprawdzi, czy element jest odłączony od dokumentu po wykonaniu microtaska.

- Jeśli element nadal jest w dokumencie, jest to przeniesienie i instancja komponentu zostanie zachowana;

- Jeśli element jest odłączony od dokumentu, jest to usunięcie i instancja komponentu zostanie odmontowana.

#### Props {#props}

- Wszystkie właściwości zadeklarowane za pomocą opcji `props` będą zdefiniowane w elemencie niestandardowym jako właściwości. Vue automatycznie obsłuży odzwierciedlenie między atrybutami / właściwościami tam, gdzie jest to odpowiednie.

- Atrybuty są zawsze odzwierciedlane do odpowiadających im właściwości.

- Właściwości z wartościami prymitywnymi (`string`, `boolean` lub `number`) są odzwierciedlane jako atrybuty.

- Vue automatycznie konwertuje również właściwości zadeklarowane z typami `Boolean` lub `Number` na pożądany typ, gdy są ustawiane jako atrybuty (które zawsze są ciągami znaków). Na przykład, mając następującą deklarację właściwości:

  ```js
  props: {
    selected: Boolean,
    index: Number
  }
  ```

  Użycie niestandardowego elementu:

  ```vue-html
  <my-element selected index="1"></my-element>
  ```

  W komponencie, `selected` zostanie przekształcone na `true` (boolean), a `index` zostanie przekształcone na `1` (liczba).

#### Zdarzenia {#events}

Zdarzenia emitowane za pomocą `this.$emit` lub `emit` w setup są wysyłane jako natywne [CustomEvents](https://developer.mozilla.org/en-US/docs/Web/Events/Creating_and_triggering_events#adding_custom_data_%E2%80%93_customevent) na elemencie niestandardowym. Dodatkowe argumenty zdarzeń (payload) będą dostępne jako tablica w obiekcie CustomEvent jako jego właściwość `detail`.

#### Sloty {#slots}

Wewnątrz komponentu sloty mogą być renderowane przy użyciu elementu `<slot/>` jak zwykle. Jednak podczas używania wynikowego elementu, akceptuje on tylko [natywną składnię slotów](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_templates_and_slots):

- [Sloty z zakresem](/guide/components/slots#scoped-slots) nie są wspierane.

- Podczas przekazywania nazwanych slotów użyj atrybutu `slot` zamiast dyrektywy `v-slot`:

  ```vue-html
  <my-element>
    <div slot="named">hello</div>
  </my-element>
  ```

#### Provide / Inject {#provide-inject}

[API Provide / Inject](/guide/components/provide-inject#provide-inject) i jego [odpowiednik w Composition API](/api/composition-api-dependency-injection#provide) działają również między niestandardowymi elementami zdefiniowanymi w Vue. Jednak należy zauważyć, że działa to **tylko między elementami niestandardowymi**. Tzn. niestandardowy element zdefiniowany w Vue nie będzie w stanie wstrzyknąć właściwości dostarczonych przez komponent Vue niebędący elementem niestandardowym.

### SFC jako element niestandardowy {#sfc-as-custom-element}

`defineCustomElement` działa również z komponentami jednoplikowymi Vue (SFC). Jednak przy domyślnej konfiguracji narzędzi, `<style>` wewnątrz SFC nadal będzie wyodrębniony i połączony w jeden plik CSS podczas budowania produkcyjnego. Podczas używania SFC jako elementu niestandardowego, często pożądane jest wstrzyknięcie tagów `<style>` do shadow root elementu niestandardowego.

Oficjalne narzędzia SFC obsługują importowanie SFC w "trybie elementu niestandardowego" (wymaga `@vitejs/plugin-vue@^1.4.0` lub `vue-loader@^16.5.0`). SFC załadowany w trybie elementu niestandardowego umieszcza swoje tagi `<style>` jako ciągi znaków CSS i udostępnia je w opcji `styles` komponentu. Zostanie to rozpoznane przez `defineCustomElement` i wstrzyknięte do shadow root elementu podczas tworzenia instancji.

Aby włączyć ten tryb, wystarczy zakończyć nazwę pliku komponentu na `.ce.vue`:

```js
import { defineCustomElement } from 'vue'
import Example from './Example.ce.vue'

console.log(Example.styles) // ["/* wbudowany css */"]

// przekształć w konstruktor elementu niestandardowego
const ExampleElement = defineCustomElement(Example)

// zarejestruj
customElements.define('my-example', ExampleElement)
```

Jeśli chcesz dostosować, które pliki powinny być importowane w trybie elementu niestandardowego (na przykład, traktowanie _wszystkich_ SFC jako elementów niestandardowych), możesz przekazać opcję `customElement` do odpowiednich wtyczek:

- [@vitejs/plugin-vue](https://github.com/vitejs/vite-plugin-vue/tree/main/packages/plugin-vue#using-vue-sfcs-as-custom-elements)
- [vue-loader](https://github.com/vuejs/vue-loader/tree/next#v16-only-options)

### Wskazówki dotyczące biblioteki elementów niestandardowych Vue {#tips-for-a-vue-custom-elements-library}

Podczas budowania elementów niestandardowych z Vue, elementy będą polegać na środowisku uruchomieniowym Vue. Istnieje bazowy koszt rozmiaru ~16kb w zależności od tego, ile funkcji jest używanych. Oznacza to, że nie jest to idealne rozwiązanie do używania Vue, jeśli dostarczasz pojedynczy element niestandardowy - możesz chcieć użyć czystego JavaScript, [petite-vue](https://github.com/vuejs/petite-vue) lub frameworków specjalizujących się w małym rozmiarze środowiska uruchomieniowego. Jednak bazowy rozmiar jest bardziej uzasadniony, jeśli dostarczasz kolekcję elementów niestandardowych ze złożoną logiką, ponieważ Vue pozwoli na tworzenie każdego komponentu przy użyciu znacznie mniejszej ilości kodu. Im więcej elementów dostarczasz razem, tym lepszy kompromis.

Jeśli elementy niestandardowe będą używane w aplikacji, która również używa Vue, możesz zdecydować się na zewnętrzne udostępnienie Vue z zbudowanej paczki, dzięki czemu elementy będą używać tej samej kopii Vue co aplikacja hostująca.

Zaleca się eksportowanie indywidualnych konstruktorów elementów, aby dać użytkownikom elastyczność w importowaniu ich na żądanie i rejestrowaniu ich z pożądanymi nazwami tagów. Możesz również wyeksportować wygodną funkcję do automatycznej rejestracji wszystkich elementów. Oto przykładowy punkt wejścia biblioteki elementów niestandardowych Vue:

```js
import { defineCustomElement } from 'vue'
import Foo from './MyFoo.ce.vue'
import Bar from './MyBar.ce.vue'

const MyFoo = defineCustomElement(Foo)
const MyBar = defineCustomElement(Bar)

// eksport poszczególnych elemntów
export { MyFoo, MyBar }

export function register() {
  customElements.define('my-foo', MyFoo)
  customElements.define('my-bar', MyBar)
}
```

Jeśli masz wiele komponentów, możesz również wykorzystać funkcje narzędzi buildowania, takie jak [glob import](https://vitejs.dev/guide/features.html#glob-import) Vite lub [`require.context`](https://webpack.js.org/guides/dependency-management/#requirecontext) webpacka do załadowania wszystkich komponentów z katalogu.

### Web Components i TypeScript {#web-components-and-typescript}

Jeśli tworzysz aplikację lub bibliotekę, możesz chcieć [sprawdzać typy](/guide/scaling-up/tooling.html#typescript) swoich komponentów Vue, włączając te zdefiniowane jako elementy niestandardowe.

Elementy niestandardowe są rejestrowane globalnie przy użyciu natywnych API, więc domyślnie nie będą miały wnioskowania typów podczas używania w szablonach Vue. Aby zapewnić wsparcie dla typów komponentów Vue zarejestrowanych jako elementy niestandardowe, możemy zarejestrować globalne typowania komponentów używając interfejsu [`GlobalComponents`](https://github.com/vuejs/language-tools/blob/master/packages/vscode-vue/README.md#usage) w szablonach Vue i/lub w [JSX](https://www.typescriptlang.org/docs/handbook/jsx.html#intrinsic-elements):

```typescript
import { defineCustomElement } from 'vue'

// komponent vue SFC
import CounterSFC from './src/components/counter.ce.vue'

// przekształć komponent w web component
export const Counter = defineCustomElement(CounterSFC)

// zarejestruj globalne typowania
declare module 'vue' {
  export interface GlobalComponents {
    'Counter': typeof Counter,
  }
}
```

## Web Components vs. Komponenty Vue {#web-components-vs-vue-components}

Niektórzy programiści uważają, że należy unikać modeli komponentów właściwych dla frameworków i że wyłączne używanie Custom Elements sprawia, że aplikacja jest "odporna na przyszłość". Postaramy się wyjaśnić, dlaczego uważamy to za nadmiernie uproszczone podejście do problemu.

Rzeczywiście istnieje pewien poziom nakładania się funkcji między Custom Elements a Komponentami Vue: oba pozwalają nam definiować komponenty wielokrotnego użytku z przekazywaniem danych, emitowaniem zdarzeń i zarządzaniem cyklem życia. Jednak API Web Components są stosunkowo niskopoziomowe i podstawowe. Aby zbudować rzeczywistą aplikację, potrzebujemy kilku dodatkowych możliwości, których platforma nie obejmuje:

- Deklaratywny i wydajny system szablonów;

- Reaktywny system zarządzania stanem, który ułatwia ekstrakcję i ponowne wykorzystanie logiki między komponentami;

- Wydajny sposób renderowania komponentów po stronie serwera i ich hydratacji po stronie klienta (SSR), co jest ważne dla SEO i [metryk Web Vitals, takich jak LCP](https://web.dev/vitals/). Natywny SSR elementów niestandardowych zazwyczaj wymaga symulowania DOM w Node.js, a następnie serializacji zmutowanego DOM, podczas gdy Vue SSR kompiluje się do konkatenacji ciągów znaków, gdy tylko jest to możliwe, co jest znacznie bardziej wydajne.

Model komponentów Vue został zaprojektowany z myślą o tych potrzebach jako spójny system.

Z kompetentnym zespołem inżynierów prawdopodobnie mógłbyś zbudować odpowiednik w oparciu o natywne Custom Elements - ale oznacza to również przejęcie długoterminowego obciążenia związanego z utrzymaniem własnego frameworka, tracąc jednocześnie korzyści ekosystemu i społeczności dojrzałego frameworka, takiego jak Vue.

Istnieją również frameworki zbudowane przy użyciu Custom Elements jako podstawy ich modelu komponentów, ale wszystkie nieuchronnie muszą wprowadzać własne rozwiązania dla wymienionych problemów. Korzystanie z tych frameworków wiąże się z akceptacją ich technicznych decyzji dotyczących rozwiązywania tych problemów - co, mimo tego co może być reklamowane, nie chroni automatycznie przed potencjalnymi przyszłymi zmianami.

Istnieją również obszary, w których uważamy, że elementy niestandardowe są ograniczające:

- Zachłanna ewaluacja slotów utrudnia kompozycję komponentów. [Sloty z zakresem](/guide/components/slots#scoped-slots) Vue są potężnym mechanizmem kompozycji komponentów, który nie może być obsługiwany przez elementy niestandardowe ze względu na zachłanną naturę natywnych slotów. Zachłanne sloty oznaczają również, że komponent odbierający nie może kontrolować kiedy lub czy renderować zawartość slotu.

- Dostarczanie elementów niestandardowych z CSS o zakresie shadow DOM wymaga obecnie osadzania CSS wewnątrz JavaScript, aby można było wstrzyknąć je do shadow roots w czasie wykonywania. Prowadzą one również do zduplikowanych stylów w znacznikach w scenariuszach SSR. W tym obszarze trwają prace nad [funkcjami platformy](https://github.com/whatwg/html/pull/4898/) - ale obecnie nie są one jeszcze powszechnie wspierane i nadal istnieją obawy dotyczące wydajności produkcyjnej / SSR. W międzyczasie Vue SFC zapewniają [mechanizmy określania zakresu CSS](/api/sfc-css-features), które obsługują wyodrębnianie stylów do zwykłych plików CSS.

Vue zawsze będzie aktualne względem najnowszych standardów platformy internetowej i chętnie wykorzystamy wszystko, co platforma oferuje, jeśli ułatwi to naszą pracę. Jednak naszym celem jest dostarczanie rozwiązań, które działają dobrze i działają dzisiaj. Oznacza to, że musimy włączać nowe funkcje platformy z krytycznym podejściem - a to wiąże się z wypełnianiem luk tam, gdzie standardy nie spełniają oczekiwań, dopóki tak jest.
