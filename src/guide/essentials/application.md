# Tworzenie aplikacji Vue {#creating-a-vue-application}

## Instancja aplikacji {#the-application-instance}

Każda aplikacja Vue zaczyna się od utworzenia nowej **instancji aplikacji** za pomocą funkcji [`createApp`](/api/application#createapp):

```js
import { createApp } from 'vue'

const app = createApp({
  /* opcje komponentu głównego */
})
```

## Komponent główny {#the-root-component}

Obiekt, który przekazujemy do `createApp`, jest w rzeczywistości komponentem. Każda aplikacja wymaga „komponentu głównego (root component)”, który może zawierać inne komponenty jako swoje elementy podrzędne.

Jeśli używasz komponentów pojedynczego pliku (Single-File Components), zazwyczaj importujemy komponent główny z innego pliku:

```js
import { createApp } from 'vue'
// importowanie komponentu głównego App z pliku pojedynczego komponentu
import App from './App.vue'

const app = createApp(App)
```

Chociaż wiele przykładów w tym przewodniku wymaga tylko jednego komponentu, większość rzeczywistych aplikacji jest zorganizowana w drzewo zagnieżdżonych, komponentów wielokrotnego użytku. Na przykład drzewo komponentów aplikacji Todo może wyglądać tak:

```
App (komponent główny)
├─ TodoList
│  └─ TodoItem
│     ├─ TodoDeleteButton
│     └─ TodoEditButton
└─ TodoFooter
   ├─ TodoClearButton
   └─ TodoStatistics
```

W kolejnych sekcjach przewodnika omówimy, jak definiować i komponować wiele komponentów razem. Zanim jednak to zrobimy, skupimy się na tym, co dzieje się wewnątrz pojedynczego komponentu.

## Montowanie aplikacji {#mounting-the-app}

Instancja aplikacji nie wyrenderuje niczego, dopóki nie zostanie wywołana jej metoda `.mount()`. Oczekuje ona argumentu „kontener”, którym może być rzeczywisty element DOM lub selektor:

```html
<div id="app"></div>
```

```js
app.mount('#app')
```

Zawartość komponentu głównego aplikacji zostanie wyrenderowana wewnątrz elementu kontenera. Sam element kontenera nie jest uważany za część aplikacji.

Metoda `.mount()` powinna być zawsze wywoływana po zakończeniu wszystkich konfiguracji aplikacji i rejestracji zasobów. Warto zauważyć, że zwracana wartość, w przeciwieństwie do metod rejestracji zasobów, to instancja komponentu głównego, a nie instancja aplikacji.

### Szablon komponentu głównego w DOM {#in-dom-root-component-template}

Szablon dla komponentu głównego jest zazwyczaj częścią samego komponentu, ale można go również dostarczyć osobno, wpisując go bezpośrednio w kontener montowania:

```html
<div id="app">
  <button @click="licznik++">{{ licznik }}</button>
</div>
```

```js
import { createApp } from 'vue'

const app = createApp({
  data() {
    return {
      licznik: 0
    }
  }
})

app.mount('#app')
```

Vue automatycznie użyje `innerHTML` kontenera jako szablonu, jeśli komponent główny nie ma już opcji `template`.

Szablony w DOM są często używane w aplikacjach, które [korzystają z Vue bez etapu kompilacji](/guide/quick-start.html#using-vue-from-cdn). Mogą być również używane w połączeniu z frameworkami po stronie serwera, gdzie główny szablon może być dynamicznie generowany przez serwer.

## Konfiguracja aplikacji {#app-configurations}

Instancja aplikacji udostępnia obiekt `.config`, który pozwala konfigurować kilka opcji na poziomie aplikacji, na przykład definiowanie globalnego handlera błędów:

```js
app.config.errorHandler = (err) => {
  /* obsługa błędu */
}
```

Instancja aplikacji oferuje także kilka metod do rejestrowania zasobów na poziomie aplikacji. Na przykład rejestrowanie komponentu:

```js
app.component('TodoDeleteButton', TodoDeleteButton)
```

To sprawia, że `TodoDeleteButton` jest dostępny do użycia w całej aplikacji. Omówimy rejestrację komponentów i innych typów zasobów w kolejnych sekcjach przewodnika. Możesz również przejrzeć pełną listę interfejsów API instancji aplikacji w jej [dokumentacji API](/api/application).

Upewnij się, że zastosowałeś wszystkie konfiguracje aplikacji przed jej zamontowaniem!

## Wiele instancji aplikacji {#multiple-application-instances}

Nie jesteś ograniczony do jednej instancji aplikacji na stronie. API `createApp` pozwala na współistnienie wielu aplikacji Vue na tej samej stronie, każda z własnym zakresem konfiguracji i zasobów globalnych:

```js
const app1 = createApp({
  /* ... */
})
app1.mount('#kontener-1')

const app2 = createApp({
  /* ... */
})
app2.mount('#kontener-2')
```

Jeśli używasz Vue do wzbogacenia renderowanego po stronie serwera HTML i chcesz, aby Vue kontrolowało tylko określone części dużej strony, unikaj montowania jednej instancji Vue na całej stronie. Zamiast tego utwórz wiele małych instancji aplikacji i zamontuj je na elementach, za które są odpowiedzialne.
