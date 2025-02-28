---
outline: deep
---

# Renderowanie po stronie serwera (SSR) {#server-side-rendering-ssr}

## Wprowadzenie {#overview}

### Czym jest SSR? {#what-is-ssr}

Vue.js to framework do tworzenia aplikacji po stronie klienta. Domyślnie komponenty Vue generują i modyfikują DOM w przeglądarce. Jednak możliwe jest także renderowanie tych samych komponentów do kodu HTML na serwerze, wysyłanie go bezpośrednio do przeglądarki, a następnie "hydracja" statycznego kodu HTML, przekształcająca go w pełni interaktywną aplikację po stronie klienta.

Aplikacja Vue.js renderowana po stronie serwera może być również uznawana za "izomorficzną" lub "uniwersalną", ponieważ większość kodu aplikacji działa zarówno na serwerze **jak i** na kliencie.

### Dlaczego SSR? {#why-ssr}

W porównaniu do aplikacji jednostronicowej (SPA) po stronie klienta, zaleta SSR polega głównie na:

- **Szybszy czas do wyświetlenia treści**: jest to bardziej widoczne przy wolnym internecie lub na wolnych urządzeniach. Renderowany po stronie serwera kod HTML nie musi czekać, aż cały JavaScript zostanie pobrany i wykonany, aby zostać wyświetlonym, więc użytkownik zobaczy w pełni wyrenderowaną stronę szybciej. Dodatkowo pobieranie danych odbywa się po stronie serwera podczas pierwszej wizyty, co prawdopodobnie zapewnia szybsze połączenie z bazą danych niż po stronie klienta. Zwykle prowadzi to do poprawy wskaźników [Core Web Vitals](https://web.dev/vitals/), lepszej jakości doświadczenia użytkownika, a także może mieć kluczowe znaczenie w aplikacjach, gdzie czas do wyświetlenia treści ma bezpośredni wpływ na współczynnik konwersji.

- **Jednolity model mentalny**: masz możliwość używania tego samego języka i tego samego deklaratywnego, komponentowego modelu mentalnego do tworzenia całej aplikacji, zamiast przełączać się między systemem szablonów po stronie backendu a frameworkiem frontendowym.

- **Lepsze SEO**: roboty wyszukiwarek zobaczą bezpośrednio w pełni wyrenderowaną stronę.

  :::tip
  Na dzień dzisiejszy Google i Bing potrafią indeksować aplikacje JavaScript działające synchronicznie bez problemu. Słowo kluczowe to "synchronicznie". Jeśli twoja aplikacja zaczyna się od wskaźnika ładowania, a następnie pobiera treści za pomocą Ajaxa, robot nie poczeka, aż skończysz. Oznacza to, że jeśli pobierasz treści asynchronicznie na stronach, gdzie SEO ma znaczenie, SSR może być konieczne.
  :::

Istnieją także pewne ograniczenia, które należy wziąć pod uwagę przy używaniu SSR:

- Ograniczenia w procesie deweloperskim. Kod specyficzny dla przeglądarki może być używany tylko w wybranych hookach cyklu życia; niektóre zewnętrzne biblioteki mogą wymagać specjalnego traktowania, aby mogły działać w aplikacji renderowanej po stronie serwera..

- Bardziej skomplikowana konfiguracja kompilacji i wymagania dotyczące wdrożenia. W przeciwieństwie do w pełni statycznej aplikacji SPA, którą można wdrożyć na dowolnym serwerze plików statycznych, aplikacja renderowana po stronie serwera wymaga środowiska, w którym może działać serwer Node.js..

- Większe obciążenie po stronie serwera. Renderowanie pełnej aplikacji w Node.js będzie bardziej wymagające dla CPU niż tylko serwowanie plików statycznych, więc jeśli spodziewasz się dużego ruchu, przygotuj się na odpowiednie obciążenie serwera i mądrze stosuj strategie cache’owania.

Zanim zdecydujesz się na użycie SSR w swojej aplikacji, pierwsze pytanie, które powinieneś zadać, to czy naprawdę go potrzebujesz. W dużej mierze zależy to od tego, jak ważny jest czas do wyświetlenia treści w twojej aplikacji. Na przykład, jeśli tworzysz wewnętrzny panel, gdzie dodatkowe kilkaset milisekund przy początkowym ładowaniu nie ma dużego znaczenia, SSR będzie przesadą. Jednak w przypadkach, gdzie czas do wyświetlenia treści jest absolutnie krytyczny, SSR pomoże osiągnąć najlepszą możliwą wydajność początkowego ładowania.

### SSR vs. SSG {#ssr-vs-ssg}

**Statyczne generowanie stron (SSG)**, znane również jako pre-rendering, to kolejna popularna technika tworzenia szybkich stron internetowych. Jeśli dane potrzebne do renderowania strony po stronie serwera są takie same dla każdego użytkownika, zamiast renderować stronę za każdym razem, gdy przychodzi żądanie, możemy wyrenderować ją tylko raz, z wyprzedzeniem, podczas procesu budowania. Strony pre-renderowane są generowane i serwowane jako statyczne pliki HTML.

SSG zachowuje te same cechy wydajnościowe co aplikacje SSR: zapewnia świetną wydajność czasu do wyświetlenia treści. Jednocześnie jest tańsze i łatwiejsze w wdrożeniu niż aplikacje SSR, ponieważ wynikowy kod to statyczny HTML i zasoby. Słowo kluczowe tutaj to **statyczny**: SSG może być stosowane tylko do stron, które konsumują dane statyczne, tzn. dane, które są znane w czasie budowania i nie zmieniają się między publikacjami. Za każdym razem, gdy dane się zmieniają, potrzebna jest nowa publikacja.

Jeśli rozważasz SSR tylko w celu poprawy SEO kilku stron marketingowych (np.  `/`, `/about`, `/contact`, itp.), to prawdopodobnie lepszym wyborem będzie SSG zamiast SSR. SSG jest również świetne dla stron opartych na treści, takich jak strony dokumentacji czy blogi. W rzeczywistości ta strona, którą właśnie czytasz, jest generowana statycznie przy użyciu [VitePress](https://vitepress.dev/), generatora stron statycznych zbudowanego na Vue.

## Podstawowy Poradnik {#basic-tutorial}

### Renderowanie aplikacji {#rendering-an-app}

Przyjrzyjmy się najprostszemu przykładzie działania SSR w Vue.

1. Utwórz nowy katalog i wejdź do niego `cd`
2. Uruchom `npm init -y`
3. Dodaj `"type": "module"` w `package.json` aby Node.js działał w trybie [ES modules mode](https://nodejs.org/api/esm.html#modules-ecmascript-modules).
4. Uruchom `npm install vue`
5. Utwórz plik `example.js`:

```js
// this runs in Node.js on the server.
import { createSSRApp } from 'vue'
// Vue's server-rendering API is exposed under `vue/server-renderer`.
import { renderToString } from 'vue/server-renderer'

const app = createSSRApp({
  data: () => ({ count: 1 }),
  template: `<button @click="count++">{{ count }}</button>`
})

renderToString(app).then((html) => {
  console.log(html)
})
```

Następnie uruchom:

```sh
> node example.js
```

Powinno to wyświetlić następujący wynik w terminalu:

```
<button>1</button>
```

[`renderToString()`](/api/ssr#rendertostring) przyjmuje instancję aplikacji Vue i zwraca Promise, która zwraca wyrenderowany HTML aplikacji. Istnieje również możliwość strumieniowania renderowania za pomocą [Node.js Stream API](https://nodejs.org/api/stream.html) lub [Web Streams API](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API). Zajrzyj do [SSR API Reference](/api/ssr) po pełne szczegóły.

Następnie możemy przenieść kod SSR Vue do obsługi żądania serwera, który wstawi markup aplikacji w pełny HTML strony. Do kolejnych kroków będziemy używać [`express`](https://expressjs.com/):

- Uruchom `npm install express`
- Utwórz następujący plik `server.js`:

```js
import express from 'express'
import { createSSRApp } from 'vue'
import { renderToString } from 'vue/server-renderer'

const server = express()

server.get('/', (req, res) => {
  const app = createSSRApp({
    data: () => ({ count: 1 }),
    template: `<button @click="count++">{{ count }}</button>`
  })

  renderToString(app).then((html) => {
    res.send(`
    <!DOCTYPE html>
    <html>
      <head>
        <title>Vue SSR Example</title>
      </head>
      <body>
        <div id="app">${html}</div>
      </body>
    </html>
    `)
  })
})

server.listen(3000, () => {
  console.log('ready')
})
```

Na końcu uruchom `node server.js` i odwiedź `http://localhost:3000`. Powinieneś zobaczyć działającą stronę z przyciskiem.

[Try it on StackBlitz](https://stackblitz.com/fork/vue-ssr-example-basic?file=index.js)

### Hydratacja po stronie klienta {#client-hydration}

Jeśli klikniesz przycisk, zauważysz, że liczba się nie zmienia. HTML jest całkowicie statyczny po stronie klienta, ponieważ nie ładujemy Vue w przeglądarce.

Aby aplikacja po stronie klienta stała się interaktywna, Vue musi wykonać krok **hydracji**. Podczas hydratacji tworzy tę samą aplikację Vue, która była uruchomiona na serwerze, dopasowuje każdy komponent do odpowiednich węzłów DOM, którymi powinien zarządzać, i dołącza nasłuchiwacze zdarzeń DOM.

Aby zamontować aplikację w trybie hydratacji, musimy użyć [`createSSRApp()`](/api/application#createssrapp) zamiast `createApp()`:

```js{2}
// this runs in the browser.
import { createSSRApp } from 'vue'

const app = createSSRApp({
  // ...same app as on server
})

// mounting an SSR app on the client assumes
// the HTML was pre-rendered and will perform
// hydration instead of mounting new DOM nodes.
app.mount('#app')
```

### Struktura kodu {#code-structure}

Zauważ, że musimy ponownie używać tej samej implementacji aplikacji, co na serwerze. To tutaj musimy zacząć myśleć o strukturze kodu w aplikacji SSR – jak współdzielić ten sam kod aplikacji między serwerem a klientem?

Tutaj pokażemy najprostsze możliwe ustawienie. Na początek, podzielmy logikę tworzenia aplikacji na dedykowany plik, `app.js`:

```js
// app.js (współdzielony między serwerem a klientem)
import { createSSRApp } from 'vue'

export function createApp() {
  return createSSRApp({
    data: () => ({ count: 1 }),
    template: `<button @click="count++">{{ count }}</button>`
  })
}
```

Ten plik i jego zależności są współdzielone między serwerem a klientem – nazywamy go **kodem uniwersalnym**. Istnieje kilka rzeczy, na które musisz zwrócić uwagę podczas pisania kodu uniwersalnego, co [omówimy poniżej](#writing-ssr-friendly-code).

Plik wejściowy klienta importuje kod uniwersalny, tworzy aplikację i wykonuje montowanie:

```js
// client.js
import { createApp } from './app.js'

createApp().mount('#app')
```

A serwer używa tej samej logiki tworzenia aplikacji w obsłudze żądań:

```js{2,5}
// server.js (irrelevant code omitted)
import { createApp } from './app.js'

server.get('/', (req, res) => {
  const app = createApp()
  renderToString(app).then(html => {
    // ...
  })
})
```

Dodatkowo, aby załadować pliki klienta w przeglądarce, musimy również:

1. Udostępnić pliki klienta, dodając `server.use(express.static('.'))` w `server.js`.
2. Załadować plik wejściowy klienta, dodając `<script type="module" src="/client.js"></script>` do szablonu HTML.
3. Wspierać użycie takie jak `import * from 'vue'` w przeglądarce, dodając [Import Map](https://github.com/WICG/import-maps) do szablonu HTML.

[Wypróbuj gotowy przykład na StackBlitz](https://stackblitz.com/fork/vue-ssr-example?file=index.js). Teraz przycisk jest interaktywny!

## Rozwiązania Wyższego Poziomu {#higher-level-solutions}

Przejście od przykładu do aplikacji SSR gotowej do produkcji wiąże się z wieloma dodatkowymi krokami. Będziemy musieli:

- Obsługiwać Vue SFC i inne wymagania związane z procesem budowania. W rzeczywistości będziemy musieli koordynować dwa procesy budowania dla tej samej aplikacji: jeden dla klienta i jeden dla serwera.

  :::tip
  Komponenty Vue są kompilowane w inny sposób, gdy są używane do SSR – szablony są kompilowane do konkatenacji ciągów znaków, a nie funkcji renderujących Virtual DOM, co zapewnia bardziej efektywne renderowanie.
  :::

- W obsłudze żądań serwera, renderuj HTML z odpowiednimi linkami do zasobów po stronie klienta oraz optymalnymi wskazówkami dotyczącymi zasobów. Może być również konieczne przełączanie się między trybem SSR a SSG, a nawet łączenie obu w tej samej aplikacji.

- Zarządzaj routingiem, pobieraniem danych i magazynami stanu w sposób uniwersalny.

Pełna implementacja byłaby dość skomplikowana i zależy od wybranego narzędzia do budowy. Dlatego zdecydowanie zalecamy skorzystanie z rozwiązania wyższego poziomu, które upraszcza tę złożoność. Poniżej przedstawimy kilka rekomendowanych rozwiązań SSR w ekosystemie Vue.

### Nuxt {#nuxt}

[Nuxt](https://nuxt.com/) to framework wyższego poziomu zbudowany na ekosystemie Vue, który zapewnia uproszczone doświadczenie deweloperskie przy pisaniu uniwersalnych aplikacji Vue. Co więcej, możesz go również używać jako generator statycznych stron! Zdecydowanie polecamy spróbować.

### Quasar {#quasar}

[Quasar](https://quasar.dev) to kompleksowe rozwiązanie oparte na Vue, które pozwala na tworzenie aplikacji SPA, SSR, PWA, aplikacji mobilnych, aplikacji desktopowych oraz rozszerzeń do przeglądarek, wszystko przy użyciu jednej bazy kodu. Nie tylko obsługuje konfigurację procesu budowania, ale także oferuje pełną kolekcję komponentów UI zgodnych z Material Design.

### Vite SSR {#vite-ssr}

Vite zapewnia wbudowane [wsparcie dla renderowania po stronie serwera Vue](https://vitejs.dev/guide/ssr.html), ale jest to celowo rozwiązanie na niskim poziomie. Jeśli chcesz użyć Vite bezpośrednio, zapoznaj się z [vite-plugin-ssr](https://vite-plugin-ssr.com/), wtyczką społecznościową, która abstrahuje wiele trudnych szczegółów.

Możesz także znaleźć przykład projektu Vue + Vite SSR z ręczną konfiguracją [tutaj](https://github.com/vitejs/vite-plugin-vue/tree/main/playground/ssr-vue), który może służyć jako baza do dalszej budowy. Zauważ, że jest to zalecane tylko, jeśli masz doświadczenie z SSR / narzędziami do budowy i naprawdę chcesz mieć pełną kontrolę nad architekturą wyższego poziomu.

## Pisanie Kodu Przyjaznego dla SSR {#writing-ssr-friendly-code}

Niezależnie od wybranego narzędzia do budowy czy frameworka wyższego poziomu, istnieją pewne zasady, które mają zastosowanie we wszystkich aplikacjach Vue SSR.

### Reaktywność na Serwerze {#reactivity-on-the-server}

Podczas SSR każdy adres URL żądania mapuje się na pożądany stan naszej aplikacji. Nie ma interakcji użytkownika ani aktualizacji DOM, więc reaktywność na serwerze jest zbędna. Domyślnie reaktywność jest wyłączona podczas SSR, aby poprawić wydajność.

### Hooki cyklu życia komponentu {#component-lifecycle-hooks}

Ponieważ podczas SSR nie ma dynamicznych aktualizacji, hooki takie jak <span class="options-api">`mounted`</span><span class="composition-api">`onMounted`</span> lub <span class="options-api">`updated`</span><span class="composition-api">`onUpdated`</span> **NIE** będą wywoływane podczas SSR i będą uruchamiane tylko po stronie klienta.<span class="options-api"> Jedynymi hookami, które są wywoływane podczas SSR, są `beforeCreate` i `created`</span>

Należy unikać kodu, który powoduje efekty uboczne wymagające czyszczenia w <span class="options-api">`beforeCreate` i `created`</span><span class="composition-api">`setup()` lub w głównym zakresie `<script setup>`</span>. Przykładem takich efektów ubocznych jest ustawianie timerów za pomocą `setInterval`. Timer możemy ustawić tylko w kodzie działającym po stronie klienta i usunąć go w <span class="options-api">`beforeUnmount`</span><span class="composition-api">`onBeforeUnmount`</span> lub <span class="options-api">`unmounted`</span><span class="composition-api">`onUnmounted`</span>. Jednak ponieważ hooki usuwania nie są wywoływane podczas SSR, timery będą utrzymywać się w tle. Aby temu zapobiec, przenieś kod wywołujący efekty uboczne do <span class="options-api">`mounted`</span><span class="composition-api">`onMounted`</span>.

### Dostęp do API Specyficznych dla Platformy {#access-to-platform-specific-apis}

Kod uniwersalny nie może zakładać dostępu do API specyficznych dla platformy, więc jeśli Twój kod bezpośrednio używa globalnych zmiennych dostępnych tylko w przeglądarce, takich jak `window` lub `document`, spowoduje to błędy podczas wykonywania w Node.js i odwrotnie.

Dla zadań, które są wspólne dla serwera i klienta, ale korzystają z różnych API platform, zaleca się opakowanie implementacji specyficznych dla platformy w uniwersalne API lub używanie bibliotek, które robią to za Ciebie. Na przykład, możesz użyć [`node-fetch`](https://github.com/node-fetch/node-fetch) aby używać tego samego API fetch zarówno na serwerze, jak i na kliencie.

Dla API dostępnych tylko w przeglądarce, powszechnym podejściem jest leniwe ich ładowanie wewnątrz hooków działających tylko po stronie klienta, takich jak <span class="options-api">`mounted`</span><span class="composition-api">`onMounted`</span>.

Warto pamiętać, że jeśli biblioteka zewnętrzna nie została napisana z myślą o uniwersalnym użyciu, może być trudno ją zintegrować w aplikacji renderowanej po stronie serwera. Możesz spróbować uruchomić ją, stosując mockowanie niektórych globalnych zmiennych, ale to będzie rozwiązanie typu hack i może zakłócać kod detekcji środowiska innych bibliotek.

### Zanieczyszczenie stanu między żądaniami{#cross-request-state-pollution}

W rozdziale o zarządzaniu stanem zaprezentowaliśmy [prosty wzorzec zarządzania stanem przy użyciu API reaktywności](state-management#simple-state-management-with-reactivity-api). W kontekście SSR, ten wzorzec wymaga pewnych dodatkowych dostosowań.

Wzorzec ten deklaruje współdzielony stan w głównym zakresie modułu JavaScript. Tworzy to **singletony** – oznacza to, że istnieje tylko jedna instancja reaktywnego obiektu przez cały cykl życia aplikacji. Działa to zgodnie z oczekiwaniami w czystej aplikacji działającej po stronie klienta, ponieważ moduły naszej aplikacji są inicjowane na nowo przy każdym odwiedzeniu strony w przeglądarce.

Jednak w kontekście SSR moduły aplikacji są zazwyczaj inicjowane tylko raz na serwerze, gdy serwer uruchamia się. Te same instancje modułów będą wykorzystywane w wielu żądaniach serwera, a więc także nasze singletony stanu. Jeśli zmodyfikujemy współdzielony stan singletona danymi specyficznymi dla jednego użytkownika, mogą one przypadkowo zostać ujawnione w żądaniu innego użytkownika. Takie zjawisko nazywamy **zanieczyszczeniem stanu między żądaniami.**

Technicznie rzecz biorąc, możemy ponownie zainicjować wszystkie moduły JavaScript przy każdym żądaniu, tak jak robimy to w przeglądarkach. Jednak inicjalizacja modułów JavaScript może być kosztowna, więc wpłynęłoby to znacząco na wydajność serwera.

Zalecanym rozwiązaniem jest tworzenie nowej instancji całej aplikacji – w tym routera i globalnych magazynów – przy każdym żądaniu. Następnie, zamiast bezpośredniego importowania stanu w komponentach, udostępniamy współdzielony stan za pomocą [provide na poziomie aplikacji](/guide/components/provide-inject#app-level-provide) i wstrzykujemy go w komponentach, które go potrzebują:

```js
// app.js (współdzielony między serwerem a klientem)
import { createSSRApp } from 'vue'
import { createStore } from './store.js'

// wywoływane przy każdym żądaniu
export function createApp() {
  const app = createSSRApp(/* ... */)
  // tworzymy nową instancję store dla każdego żądania
  const store = createStore(/* ... */)
  // udostępniamy store na poziomie aplikacji
  app.provide('store', store)
  // udostępniamy store także w celu hydratacji
  return { app, store }
}
```

Biblioteki do zarządzania stanem, takie jak Pinia, zostały zaprojektowane z myślą o tym. Zajrzyj do [przewodnika SSR Pinia](https://pinia.vuejs.org/ssr/) po więcej szczegółów.

### Niezgodność Hydratacji {#hydration-mismatch}

Jeśli struktura DOM wstępnie renderowanego HTML nie pasuje do oczekiwanego wyniku aplikacji po stronie klienta, wystąpi błąd hydratacji. Niezgodność hydratacji najczęściej wprowadza jeden z poniższych powodów:

1. Szablon zawiera nieprawidłową strukturę zagnieżdżenia HTML, a renderowany HTML został "poprawiony" przez natywne zachowanie przeglądarki w zakresie parsowania HTML. Na przykład, częstym problemem jest, że [`<div>` nie może być umieszczony wewnątrz `<p>`](https://stackoverflow.com/questions/8397852/why-cant-the-p-tag-contain-a-div-tag-inside-it):

   ```html
   <p><div>cześć</div></p>
   ```

  Jeśli wygenerujemy to w naszym serwerowo renderowanym HTML, przeglądarka zakończy pierwszy tag `<p>`, gdy napotka `<div>` a następnie przekształci to w następującą strukturę DOM:

   ```html
   <p></p>
   <div>cześć</div>
   <p></p>
   ```

2. Dane używane podczas renderowania zawierają losowo generowane wartości. Ponieważ ta sama aplikacja będzie uruchomiona dwukrotnie - raz na serwerze, a raz po stronie klienta - nie ma gwarancji, że wartości losowe będą identyczne podczas obu uruchomień. Istnieją dwa sposoby, aby uniknąć niezgodności spowodowanych wartościami losowymi:

   1. Użyj `v-if` + `onMounted` aby renderować część zależną od wartości losowych tylko po stronie klienta. Twoje środowisko może również mieć wbudowane funkcje ułatwiające to zadanie, na przykład komponent `<ClientOnly>` w VitePress.

   2. Użyj biblioteki do generowania liczb losowych, która obsługuje generowanie z nasionami, i zapewnij, że zarówno uruchomienie na serwerze, jak i po stronie klienta używają tego samego nasienia (np. poprzez dołączenie nasienia w serializowanym stanie i pobranie go po stronie klienta).

3. Serwer i klient znajdują się w różnych strefach czasowych. Czasami chcemy przekształcić znacznik czasu na czas lokalny użytkownika. Jednak strefa czasowa podczas działania serwera i klienta nie zawsze jest taka sama, i możemy nie mieć pewności co do strefy czasowej użytkownika podczas działania serwera. W takich przypadkach konwersję czasu lokalnego należy wykonać tylko po stronie klienta.

Kiedy Vue napotka niezgodność hydratacji, spróbuje automatycznie odzyskać i dostosować wstępnie wyrenderowany DOM do stanu po stronie klienta. Doprowadzi to do pewnych strat wydajności renderowania, ponieważ niepoprawne węzły będą odrzucane, a nowe będą montowane. Niemniej jednak, w większości przypadków aplikacja powinna nadal działać zgodnie z oczekiwaniami. Niemniej jednak, najlepiej jest wyeliminować niedopasowania hydratacji podczas prac developerskich.

### Niestandardowe Dyrektywy {#custom-directives}

Ponieważ większość niestandardowych dyrektyw wiąże się z bezpośrednią manipulacją DOM, są one ignorowane podczas SSR. Jednak jeśli chcesz określić, jak niestandardowa dyrektywa powinna być renderowana (np. jakie atrybuty powinna dodać do renderowanego elementu), możesz użyć hooka dyrektywy `getSSRProps`:

```js
const myDirective = {
  mounted(el, binding) {
    // implementacja po stronie klienta:
    // bezpośrednia modyfikacja DOM
    el.id = binding.value
  },
  getSSRProps(binding) {
    // implementacja po stronie serwera:
    // zwrócenie właściwości do renderowania
    // getSSRProps otrzymuje tylko powiązanie dyrektywy
    return {
      id: binding.value
    }
  }
}
```

### Teleporty {#teleports}

Teleporty wymagają specjalnego traktowania podczas SSR. Jeśli renderowana aplikacja zawiera Teleporty, ich zawartość nie będzie częścią renderowanego ciągu znaków. Łatwiejszym rozwiązaniem jest warunkowe renderowanie Teleportu podczas montowania.

Jeśli musisz zhydratować zawartość teleportu, jest ona dostępna pod właściwością `teleports` obiektu kontekstu ssr:

```js
const ctx = {}
const html = await renderToString(app, ctx)

console.log(ctx.teleports) // { '#teleported': 'zawartość teleportu' }
```

Musisz wstrzyknąć znacznik Teleportu we właściwe miejsce w końcowym HTML-u strony, podobnie jak w przypadku głównego znacznika aplikacji.

:::tip
Unikaj kierowania Teleportów do `body` podczas korzystania z nich razem z SSR – zazwyczaj `<body>` zawiera inne renderowane po stronie serwera treści, co uniemożliwia Teleportom określenie prawidłowego punktu początkowego dla hydratacji.

Zamiast tego użyj dedykowanego kontenera, np. `<div id="teleported"></div>` który będzie zawierał wyłącznie teleportowaną zawartość.
:::
