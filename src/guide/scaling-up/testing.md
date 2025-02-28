<script setup>
import { VTCodeGroup, VTCodeGroupTab } from '@vue/theme'
</script>
<style>
.lambdatest {
  background-color: var(--vt-c-bg-soft);
  border-radius: 8px;
  padding: 12px 16px 12px 12px;
  font-size: 13px;
  a {
    display: flex;
    color: var(--vt-c-text-2);
  }
  img {
    background-color: #fff;
    padding: 12px 16px;
    border-radius: 6px;
    margin-right: 24px;
  }
  .testing-partner {
    color: var(--vt-c-text-1);
    font-size: 15px;
    font-weight: 600;
  }
}
</style>

# Testowanie {#testing}

## Po co testować? {#why-test}

Testy automatyczne pomagają Tobie i Twojemu zespołowi budować złożone aplikacje Vue szybko i z większą pewnością poprzez zapobieganie regresji i zachęcanie do dzielenia aplikacji na testowalne funkcje, moduły, klasy i komponenty. Jak z każdą aplikacją, Twoja aplikacja Vue może zepsuć się na wiele sposobów i ważnym jest by móc szybko te problemy zauważać i naprawiać przed release'ami.

W tym poradniku omówimy podstawową terminologię oraz damy Ci nasze rekomendacje co do tego, które narzędzia wybrać dla Twojej aplikacji Vue 3.

Istnieje również sekcja bardziej specyficzna dla Vue - skupiająca się na composables. Zobacz [testowanie composables](#testing-composables) by dowiedzieć się więcej.

## Kiedy testować {#when-to-test}

Zacznij testować jak najwcześniej! Zalecamy by pisać testy tak wcześnie jak to możliwe. Im dłużej zwlekasz z dodaniem testów do Twojej aplikacji tym większa ilość zależności znajdzie się w niej i tym trudniej będzie zacząć.

## Typy testów {#testing-types}

Podczas planowania strategii na testowanie swojej aplikacji Vue, warto rozważyć następujące typy testów:

- **Jednostkowe** (Unit tests): Sprawdzają czy dane wejściowe danej funkcji, klasy czy komponentu odpowiadają oczekiwanym danym wyjściowej lub oczekiwanym skutkom ubocznym.
- **Komponentów**: Sprawdzają czy Twoje komponenty montują się, renderują się, mogą zachodzić w interakcje i zachowują  się zgodnie z oczekiwaniami. Testy te wymagają większej ilości kodu, są również bardziej złożone i wymagają większej ilości czasu na wykonanie.
- **End-to-end** (E2E): Sprawdzają czy funkcjonalności, które pokrywają wiele stron i wykonują faktyczne zapytania do serwerów związanych z Twoją aplikacją. Testy te wymagają postawienia bazy danych czy backendu.

Każdy typ testów odgrywa swoją rolę w strategii na testowanie Twojej aplikacji i każdy z nich zabezpiecza Cię przed różnymi typami problemów.

## Przegląd {#overview}

Poniżej krótko omówimy każdy z tych typów, jak je zaimplementować w aplikacjach Vue oraz wylistujemy ogólne rekomendacje.

## Testy jednostkowe {#unit-testing}

Testy jednostkowe są pisane po to by zweryfikować czy małe, wyizolowane kawałki kodu działają zgodnie z oczekiwaniami. Test jednostkowy zazwyczaj pokrywa jedną funkcję, klasę, composable lub moduł. Testy jednostkowe skupiają się na poprawnej logice i mają wgląd jedynie w bardzo mały kawałek funkcjonalności danej aplikacji. Mogą mockować duże kawałki środowiska aplikacji (jak na przykład stan początkowy, skomplikowane moduły, zależności czy zapytania do serwerów i usług).

Na ogół, testy jednostkowe wyłapują problemy związane z logiką biznesową danej funkcji czy samą jej logiczną poprawnością.

Weźmy na przykład poniższą funkcję `increment`:

```js
// helpers.js
export function increment(current, max = 10) {
  if (current < max) {
    return current + 1
  }
  return current
}
```

Ponieważ jest ona bardzo niezależna, łatwo wywołać funkcję `increment` i sprawdzić czy zwraca to, czego oczekujemy, a więc napiszemy dla niej test jednostkowy.

Jeśli któraś asercja nie powiedzie się, będzie to oznaczało jasny i wyraźny problem zlokalizowany w funkcji `increment`.

```js{4-16}
// helpers.spec.js
import { increment } from './helpers'

describe('increment', () => {
  test('zwiększa licznik o 1', () => {
    expect(increment(0, 10)).toBe(1)
  })

  test('nie zwiększa obecnej wartości powyżej maksimum', () => {
    expect(increment(10, 10)).toBe(10)
  })

  test('ma domyślne maksimum równe 10', () => {
    expect(increment(10)).toBe(10)
  })
})
```

Jak mówiliśmy wcześniej, testy jednostkowe są zazwyczaj aplikowane do bardzo niezależnych kawałków logiki biznesowej, komponentów, klas, modułów lub funkcji, które nie mają styczności z renderowaniem UI, zapytaniami do serwerów czy innymi zależnościami związanymi ze środowiskiem.

Są to kawałki kodu najczęściej pisane w prostym JavaScript lub TypeScript, niezależne od samego Vue. W praktyce, pisanie testów jednostkowych dla logiki biznesowej w Vue nie różni się znacząco od aplikacji pisanych w innych frameworkach.

Istnieją dwa przypadki w których testujemy funkcjonalności mocno powiązane z samym Vue:

1. Composables
2. Komponenty

### Composables {#composables}

Pierwszą z grup funkcji specyficznych dla aplikacji Vue są [Composables](/guide/reusability/composables), które wymagają innego podejścia podczas pisania testów.
Zobacz [Testowanie Composables](#testing-composables) poniżej, aby dowiedzieć się więcej.

### Testy jednostkowe komponentów {#unit-testing-components}

Komponent może być przetestowany na dwa sposoby:

1. Testy białej skrzynki (Whitebox tests)

   Testy "białej skrzynki" są pisane ze świadomością szczegółów implementacji i zależności komponentu. Skupiają się na **wyizolowaniu** komponentu, który testujemy. Testy te zazwyczaj mockują niektóre, lub nawet wszystkie dzieci komponentu, setupując również wszelkie pluginy czy zależności (np. Pinia).

2. Testy czarnej skrzynki (Blackbox tests)

   Testy "czarnej skrzynki" pisane są bez ingerowania w szczegóły implementacji komponentu. Mockują one możliwie minimum, aby przetestować pełną integrację komponentu w ramach całego systemu. Zazwyczaj renderują również wszystkie dzieci komponentu i mogą być rozważane bardziej jako "testy integracyjne". Zobacz nasze [zalecenia testowania komponentów](#component-testing) poniżej.

### Rekomendacje {#recommendation}

- [Vitest](https://vitest.dev/)

  Oficjalne, rekomendowane konfigurowanie nowych projektów przez `create-vue` korzysta z [Vite](https://vitejs.dev/), w związku z czym zalecamy również framework do testów jednostkowych, który wykorzystuje tę samą konfigurację i pipeline transformacji kodu bezpośrednio z Vite. [Vitest](https://vitest.dev/) jest frameworkiem testów jednostkowych zaprojektowanym dokładnie w tym celu, został on utworzony i jest wspierany przez członków zespołów Vue i Vite. Integruje się z projektami Vite z minimum potrzebnej konfiguracji oraz jest on bardzo szybki.

### Inne opcje {#other-options}

- [Jest](https://jestjs.io/) jest popularnym frameworkiem testów jednostkowych. Jednakże, zalecamy używać Jesta jedynie w przypadku gdy masz już istniejące testy napisane w Jeście, które planujesz zmigrować do projektu używającego Vite, w związku z tym, że Vitest oferuje o wiele prostszą integrację i znacznie lepszy performance.

## Testowanie komponentów {#component-testing}

Komponenty są głównymi klockami z których budujemy interfejsy w aplikacjach Vue. Naturalnym jest więc, aby testować komponenty w izolacji podczas testowania zachowania Twojej aplikacji. Z perspektywy granularności, testy komponentów są nieco ponad testami jednostkowymi i można uznać je za pewną formę testów integracyjnych. Większość Twojej aplikacji może być pokryta testami komponentów i rekomendujemy, aby każdy komponent Vue posiadał swój odrębny plik z testami.

Testy komponentów służą wyłapywaniu problemów związanych z jego propsami, emitowanymi zdarzeniami, slotami jakie dostarcza, stylami, klasami, hookami cyklu życia i innymi.

Testy komponentów nie powinny mockować dzieci tego komponentu, a jedynie interakcje między nimi w taki sposób, w jaki robiłby to użytkownik. Na przykład, test komponentu powinien kliknąć na element tak jak użytkownik zamiast wywoływać interakcję w sposób programowy.

Testy komponentów powinny skupiać się na ich publicznych interfejsach niż szczegółach implementacji. Dla większości komponentów, publiczny interfejs ogranicza się do: emitowanych zdarzeń, propsów oraz slotów. Podczas pisania testów pamiętaj by **testować to co komponent robi, nie to jak to robi**.

**ZALECENIA**

- Dla logiki **wizualnej**: sprawdzaj prawidłowo wyrenderowany output na podstawie propsów i slotów.
- Dla logiki **zachowania**: sprawdzaj prawidłowe aktualizacje treści czy emitowane zdarzenia na podstawie interakcji przychodzący od strony użytkownika.

  W poniższym przykładzie, demonstrujemy przykład testów komponenta "Licznik", który ma element DOM oznaczony jako "inkrementuj", który możemy kliknąć. Przekazujemy propa `max` określającego maksymalną wartość licznika równą `2`, a więc gdy klikniemy w przycisk 3 razy, interfejs powinien pokazywać `2`.

  Nie znamy szczegółowej implementacji Licznika, jedynie że możemy przekazać do niego propa `max` oraz "wyjście" jakim jest stan drzewa DOM w taki sposób jaki widzi go użytkownik.

<VTCodeGroup>
  <VTCodeGroupTab label="Vue Test Utils">

```js
const valueSelector = '[data-testid=stepper-value]'
const buttonSelector = '[data-testid=increment]'

const wrapper = mount(Stepper, {
  props: {
    max: 1
  }
})

expect(wrapper.find(valueSelector).text()).toContain('0')

await wrapper.find(buttonSelector).trigger('click')

expect(wrapper.find(valueSelector).text()).toContain('1')
```

  </VTCodeGroupTab>
  <VTCodeGroupTab label="Cypress">

```js
const valueSelector = '[data-testid=stepper-value]'
const buttonSelector = '[data-testid=increment]'

mount(Stepper, {
  props: {
    max: 1
  }
})

cy.get(valueSelector)
  .should('be.visible')
  .and('contain.text', '0')
  .get(buttonSelector)
  .click()
  .get(valueSelector)
  .should('contain.text', '1')
```

  </VTCodeGroupTab>
  <VTCodeGroupTab label="Testing Library">

```js
const { getByText } = render(Stepper, {
  props: {
    max: 1
  }
})

getByText('0') // Zakładamy że "0" znajduje się w środku wyrenderowanego komponentu

const button = getByRole('button', { name: /increment/i })

// Wywołujemy event click na naszym przycisku do inkrementacji
await fireEvent.click(button)

getByText('1')

await fireEvent.click(button)
```

  </VTCodeGroupTab>
</VTCodeGroup>

- **PRZECIWWSKAZANIA**

  Nie sprawdzaj wewnętrznego, prywatnego stanu komponentu czy też jego metod. Testowanie szczegółów implementacyjnych nie jest dobrym pomysłem, gdyż te znacznie częściej ulegają zmianom (które również musimy potem wprowadzać w testach).

  Fundamentalnym działaniem komponentu, jest wyrenderowanie odpowiednich treści w drzewie DOM, a więc testowanie, które sprawdza DOM dostarcza taki sam stopień pewności co do poprawnego działania (o ile nie większy) podczas gdy dostarcza znacznie większą odporność na zmiany implementacji komponentu.

  Nie polegaj wyłącznie na testach z wykorzystaniem snapshotów. Asercje dotyczące ciągów znaków HTML nie opisują poprawności. Pisz testy z wyraźnie zdeklarowanymi celami.

  Jeśli dana metoda musi być dokładnie przetestowana, warto rozważyć wyodrębnienie jej do samodzielnej, reużywalnej funkcji, z własnymi dedykowanymi testami jednostkowymi. Jeśli nie może być łatwo wyodrębniona, może być przetestowana jako część testu komponentu, integracyjnego lub end-to-end, który ją pokryje.

### Rekomendacje {#recommendation-1}

- [Vitest](https://vitest.dev/) dla komponentów lub composables, które renderujemy headlessowo (np. funkcja [`useFavicon`](https://vueuse.org/core/useFavicon/#usefavicon) w VueUse). Komponenty i DOM mogą być testowane przy użyciu [`@vue/test-utils`](https://github.com/vuejs/test-utils).

- [Testy komponentów w Cypress](https://on.cypress.io/component) dla komponentów, których zachowanie polega na prawidłowym renderowaniu styli, lub wywoływaniu zdarzeń natywnych DOM. Może być również używany razem z Testing Library poprzez [@testing-library/cypress](https://testing-library.com/docs/cypress-testing-library/intro).

Główną różnicą między Vitestem a narzędziami wykonującymi testy przy użyciu przeglądarki są szybkość i kontekst wykonania. W skrócie, narzędzia oparte o przeglądarkę jak Cyppress, mogą wyłapać problemy których narzędzia oparte o node, jak Vitest, nie potrafią (np. problemy związane ze stylem, prawdziwymi, natywnymi zdarzeniami w DOM, cookies, local storage czy problemami z połączeniem do sieci), ale narzędzia te są _rzędy wielkości wolniejsze niż Vitest_ ponieważ otwierają faktyczną przeglądarkę, kompilują style i więcej. Cypress jest narzędziem opartym o przeglądarkę, które wspiera testy komponentów. Zachęcamy do przeczytania [o porównaniu Vitesta do innych narzędzi](https://vitest.dev/guide/comparisons.html#cypress) w tym również Cypressa.

### Biblioteki do montowania {#mounting-libraries}

Testowanie komponentów często wymaga również montowania komponentu, który będzie testowany w izolacji, wyzwalania symulowanych akcji użytkownika czy dokonywanie asercji na wyjściowym stanie DOM. Istnieją dedykowane biblioteki które znacząco ułatwiają te zadania.

- [`@vue/test-utils`](https://github.com/vuejs/test-utils) to oficjalna biblioteka niskiego poziomu do testowania komponentów, która napisana była, aby dostarczyć użytkownikom dostęp do specyficznych API Vue. W oparciu o tą bibliotekę napisana została biblioteka `@testing-library/vue`.

- [`@testing-library/vue`](https://github.com/testing-library/vue-testing-library) to biblioteka do testów Vue, skupiona wokół testowania komponentów bez potrzeby znania szczegółów implementacji. Jej fundamentalnym założeniem jest to, że im więcej testów napiszemy w sposób przypominający to w jaki korzystamy z implementowanych funkcjonalności, tym większą pewność będą one dawać co do poprawnego ich działania.

Zalecamy używanie `@vue/test-utils` do testów komponentów w aplikacjach. `@testing-library/vue` nie radzi sobie niestety z testami asynchronicznych komponentów korzystających z Suspense, powinna być więc używana ostrożnie.

### Inne opcje {#other-options-1}

- [Nightwatch](https://nightwatchjs.org/) to narzędzie do testów E2E ze wsparciem do testów komponentów Vue. ([Przykładowy projekt](https://github.com/nightwatchjs-community/todo-vue))

- [WebdriverIO](https://webdriver.io/docs/component-testing/vue) to narzędzie do testów komponentów w różnych przeglądarkach, które polega na natywnych interakcjach użytkownika, opartych o standardową automatyzację. Może być również używane razem z Testing Library.

## Testy E2E {#e2e-testing}

Podczas gdy testy jednostkowe dostarczają nam pewien poziom pewności poprawnego działania, testy jednostkowe i komponentów są ograniczone w swoich możliwościach do jedynie do pokrycia holistycznego aplikacji gdy wypuszczamy ją na produkcję. Skutkiem tego, testy end-to-end (E2E) mają na celu pokrycie w pewnym sensie najważniejszego aspektu aplikacji: co się dzieje gdy użytkownicy faktycznie używają Twojej aplikacji.

Testy end-to-end skupiają się wokół zachowania użytkownika na przekroju wielu stron, wykonują zapytania na faktycznej aplikacji Vue zbudowanej pod cele produkcyjne. Wymagają one zatem postawienia właściwej bazy danych czy usług backendowych i mogą być wywołane na aplikacji żyjącej na środowisku stagingowym.

Testy te mogą często wyłapać błędy związane z routerem, biblioteką zarządzania stanem, komponentami najwyższego poziomu (np. App czy Layout), publicznymi assetami czy obsługą zapytań sieciowych. Jak nadmieniliśmy wyżej wykrywają one krytyczne problemy, które mogą być niemożliwe do wychwycenia przez testy jednostkowe czy komponentowe.

Testy end-to-end nie importują w ogóle kodu Twojej aplikacji Vue, a jedynie opierają się kompletnie na testowaniu aplikacji poprzez odwiedzanie stron w rzeczywistej przeglądarce.

Testy end-to-end sprawdzają poprawność wielu wartsw aplikacji na raz. Mogą celować w aplikację zbudowaną lokalnie, czy nawet aplikację na środowisku stagingowym. Testowanie aplikacji na środowisku stagingowym testuje nie tylko kod frontendowy czy serwer statyczny ale też również całą infrastrukturę i usługi backendowe.

> Im więcej testów napiszemy w sposób przypominający to w jaki korzystamy z implementowanych funkcjonalności, tym większą pewność będą one dawać co do poprawnego ich działania. - [Kent C. Dodds](https://twitter.com/kentcdodds/status/977018512689455106) - Autor biblioteki Testing Library

Testy E2E, sprawdzające jak akcje użytkownika wpływają na aplikację są często kluczowe by mieć wysoką pewność co do tego czy nasza aplikacja działa poprawnie lub nie.

### Wybór rozwiązania do testów E2E {#choosing-an-e2e-testing-solution}

Testy end-to-end (E2E) zbudowały z czasem nieco negatywną reputację z powodu zawodzących testów i spowalniania procesu deweloperskiego, jednakże współczesne narzędzia do testów E2E znacząco zaadresowały te problemy i dostarczają znacznie bardziej niezawodne, interaktywne i pomocne testy. Poniższe sekcje mają na celu nakreślić czynniki, które warto mieć na uwadze podczas wyboru narzędzi do testów E2E dla Twojej aplikacji.

#### Testowanie cross-browser {#cross-browser-testing}

Jedną z głównych zalet, z których znane są testy end-to-end (E2E) jest możliwość spsrawdzenia jak działa Twoja aplikacja na różnych przeglądarkach. Pomimo, że możemy chcieć mieć 100% pokrycie sprawdzenia jak aplikacja działa na różnych przeglądarkach, istotne jest mieć świadomość, że korzyści maleją wraz ze wzrostem kosztów zasobów i czasu koniecznego by wykonywać te testy. Ważnym jest więc mieć rozwagę i świadomość kosztów związanych z ilością testów między różnymi przeglądarkami.

#### Sybsza informacja zwrotna {#faster-feedback-loops}

Głównym problemem testów end-to-end (E2E) jest fakt, że wykonanie pełnego zestawu testów zajmuje dużo czasu. W typowym przypadku, dzieje się to jedynie podczas procesu continuous integration oraz deploymentu (CI/CD). Współczesne frameworki testów E2E pomagają rozwiązać ten problem poprzez funkcjonalności takie jak paralelizacja, która pozwala pipeline'om CI/CD działać rzędy wielkości szybciej. W dodatku, podczas pracy lokalnej, możliwośc uruchomienia jednego konkretnego testu dla strony nad którą obecnie pracujemy przy jednoczesnym wsparciu hot reloadingu tych testów znacznie usprawniają rytm pracy i produktywność deweloperów.

#### Pierwszorzędne doświadczenie debugowania {#first-class-debugging-experience}

Podczas gdy zazwyczaj deweloperzy przeszukiwali logi w oknie terminala tekstowego celem analizy co poszło nie tak podczas wykonywania testów, współczesne frameworki testów end-to-end (E2E) pozwalają deweloperom wykorzystać narzędzia z którymi są już zaznajomieni jak np. narzędzia deweloperskie przeglądarek.

#### Podgląd w trybie headless {#visibility-in-headless-mode}

Podczas gdy testy end-to-end (E2E) wykonywane są w ramach pipeline'ów continuous integration czy deploymentu, zazwyczaj wykonywane są w przeglądarkach typu headless (tj. nie ma widocznej przeglądarki którą użytkownik mógłby widzieć). Krytyczną funkcjonalnością współczesnych frameworków E2E jest możliwość zobaczenia snapshotów i/lub filmów wideo aplikacji podczas testów, dostarczając konieczny kontekst i informacje by zrozumieć czemu wykrywamy błędy. W przeszłości, utrzymywanie tych integracji było bardzo uciążliwe.

### Rekomendacje {#recommendation-2}

- [Cypress](https://www.cypress.io/)

  Biorąc wszystko pod uwagę, uważamy, że Cypress dostarcza najbardziej pełne rozwiązanie do testów E2E z funkcjonalnościami takimi jak pełen przydatnych informacji interfejs graficzny, świetne wsparcie w debugowaniu, wbudowane asercje, stuby, flake-resistence, paralelizacja i snapshoty. Jak wspominaliśmy wyżej, dostarcza wsparcie do [testów komponentów](https://docs.cypress.io/guides/component-testing/introduction). Wspiera przeglądarki oparte o Chromium, Firefoxa czy Electron. Support WebKita jest również dostępny, ale wymieniony jako eksperymentalny.

<div class="lambdatest">
  <a href="https://lambdatest.com" target="_blank">
    <img src="/images/lambdatest.svg">
    <div>
      <div class="testing-partner">Sponsor działu o Testach</div>
      <div>Lambdatest to platforma chmurowa do wykonywania testów E2E, dostępności i wizualnych testów regresji na wszystkich głównych przeglądarkach oraz prawdziwych urządzeniach, ze wsparciem generowania testów przez AI!</div>
    </div>
  </a>
</div>

### Inne opcje {#other-options-2}

- [Playwright](https://playwright.dev/) jest również dobrym rozwiązaniem do testów E2E, które wspiera wszystkie współczesne silniki renderowania wliczając Chromium, WebKit czy Firefoxa. Umożliwia testy na Windowsie, Linuxie, macOS, lokalnie czy też w ramach CI, headlessowo lub nie oraz z natywną emulacją mobilną Google Chrome dla Androida i Safari w wersji mobilnej.

- [Nightwatch](https://nightwatchjs.org/) to rozwiązanie do testów E2E oparte o [Selenium WebDriver](https://www.npmjs.com/package/selenium-webdriver). Daje to mu najszersze wsparcie przeglądarek, wliczając również natywne testy mobilne. Rozwiązania oparte o Selenium będą jednak wolniejsze niż Playwright czy Cypress.

- [WebdriverIO](https://webdriver.io/) to framework automatyzacji testów dla webu i urządzeń mobilnych opartym o protokół WebDriver.

## Przepisy {#recipes}

### Dodawanie Vitest do projektu {#adding-vitest-to-a-project}

W projekcie Vue opartym o Vite uruchom:

```sh
> npm install -D vitest happy-dom @testing-library/vue
```

Następnie, zmodyfikuj konfigurację Vite dodając block opcji `test`:

```js{6-12}
// vite.config.js
import { defineConfig } from 'vite'

export default defineConfig({
  // ...
  test: {
    // włącz globalne API testów podobnie jak jest
    globals: true,
    // symuluj DOM przy użyciu happy-dom
    // (wymaga instalacji happy-dom jako peer dependency)
    environment: 'happy-dom'
  }
})
```

:::tip
Jeśli używasz TypeScript, dodaj `vitest/globals` do pola `types` w Twoim `tsconfig.json`.

```json
// tsconfig.json

{
  "compilerOptions": {
    "types": ["vitest/globals"]
  }
}
```

:::

Następnie, utwórz plik z nazwą kończącą się na `*.test.js` w Twoim projekcie. Wszystkie pliki testów możesz umieścić w folderze test w głównym folderze projektu lub w folderach z testami obok plików z kodem źródłowym. Vitest automatycznie odnajdzie te testy po konwencji nazewnictwa.

```js
// MyComponent.test.js
import { render } from '@testing-library/vue'
import MyComponent from './MyComponent.vue'

test('powinien działać', () => {
  const { getByText } = render(MyComponent, {
    props: {
      /* ... */
    }
  })

  // assert output
  getByText('...')
})
```

Na koniec, zaktualizuj `package.json` dodając skrypt do testów i uruchom go:

```json{4}
{
  // ...
  "scripts": {
    "test": "vitest"
  }
}
```

```sh
> npm test
```

### Testowanie Composables {#testing-composables}

> Ta sekcja zakłada wcześniejsze zaznajomienie się z działem dedykowanym [composables](/guide/reusability/composables).

Rozważając testowanie composables, możemy podzielić je na dwie kategorie: composables które nie wymagają hostującej instancji komponentu, oraz composables, które tego nie potrzebują.

Composable wymaga hostującej instancji komponentu, gdy używa następujących API:

- Hooków cyklu życia
- Provide / Inject

Jeśli composable używa jedynie API reaktywności to możemy je testować bezpośrednio i wykonywać asercje na podstawie zwróconego stanu i metod:

```js
// counter.js
import { ref } from 'vue'

export function useCounter() {
  const count = ref(0)
  const increment = () => count.value++

  return {
    count,
    increment
  }
}
```

```js
// counter.test.js
import { useCounter } from './counter.js'

test('useCounter', () => {
  const { count, increment } = useCounter()
  expect(count.value).toBe(0)

  increment()
  expect(count.value).toBe(1)
})
```

Composable który polega na hookach cyklu życia lub Provide / Inject musi być owrapowany hostującym komponentem, aby być poprawnie przetestowanym. Możemy utworzyć helpera jak poniżej:

```js
// test-utils.js
import { createApp } from 'vue'

export function withSetup(composable) {
  let result
  const app = createApp({
    setup() {
      result = composable()
      // pomiń ostrzeżenia o braku template
      return () => {}
    }
  })
  app.mount(document.createElement('div'))
  // zwróć resultat i instancję aplikacji
  // aby testować provide/unmount
  return [result, app]
}
```

```js
import { withSetup } from './test-utils'
import { useFoo } from './foo'

test('useFoo', () => {
  const [result, app] = withSetup(() => useFoo(123))
  // zmockowany provide by testować wstrzyknięcia
  app.provide(...)
  // wykonujemy asercje
  expect(result.foo.value).toBe(1)
  // wywołujemy hook onMounted jeśli potrzeba
  app.unmount()
})
```

Dla bardziej złożonych composables, może być łatwiejsze napisać testy dla komponentu wrapującego używając technik z [testowania komponentów](#component-testing).

<!--
TODO more testing recipes can be added in the future e.g.
- How to set up CI via GitHub actions
- How to do mocking in component testing
-->
