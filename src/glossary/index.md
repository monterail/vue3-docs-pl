# Słownik {#glossary}

Ten słownik ma na celu dostarczenie wskazówek dotyczących znaczenia technicznych terminów, które są powszechnie używane podczas rozmowy o Vue. Ma on charakter *opisowy* tego, jak terminy są powszechnie używane, a nie *normatywnej* specyfikacji tego, jak muszą być używane. Niektóre terminy mogą mieć nieco inne znaczenia lub niuanse w zależności od otaczającego kontekstu.

[[TOC]]

## komponent asynchroniczny {#async-component}

*Komponent asynchroniczny* jest wrapperem wokół innego komponentu, który umożliwia leniwe ładowanie komponentu opakowanego. Jest to zazwyczaj wykorzystywane jako sposób na zmniejszenie rozmiaru zbudowanych plików `.js`, pozwalając na podzielenie ich na mniejsze części, które są ładowane tylko wtedy, gdy są wymagane.

Vue Router posiada podobną funkcję do [leniwego ładowania komponentów tras](https://router.vuejs.org/guide/advanced/lazy-loading.html), choć nie wykorzystuje ona funkcji komponentów asynchronicznych Vue.

Aby uzyskać więcej szczegółów zobacz:
- [Przewodnik - Komponenty Asynchroniczne](/guide/components/async.html)

## makro kompilatora {#compiler-macro}

*Makro kompilatora* to specjalny kod, który jest przetwarzany przez kompilator i konwertowany w coś innego. Są to w rzeczywistości sprytne formy zamiany ciągów znaków.

Kompilator [SFC](#single-file-component) Vue wspiera różne makra, takie jak `defineProps()`, `defineEmits()` i `defineExpose()`. Makra te są celowo zaprojektowane tak, aby wyglądały jak normalne funkcje JavaScript, dzięki czemu mogą wykorzystywać ten sam parser i narzędzia do wnioskowania typów dla JavaScript / TypeScript. Jednak nie są to rzeczywiste funkcje, które są uruchamiane w przeglądarce. Są to specjalne ciągi znaków, które kompilator wykrywa i zamienia na prawdziwy kod JavaScript, który zostanie faktycznie uruchomiony.

Makra mają ograniczenia w użyciu, które nie dotyczą normalnego kodu JavaScript. Na przykład, można by pomyśleć, że `const dp = defineProps` pozwoli utworzyć alias dla `defineProps`, ale w rzeczywistości spowoduje to błąd. Istnieją również ograniczenia dotyczące wartości, które można przekazać do `defineProps()`, ponieważ 'argumenty' muszą być przetwarzane przez kompilator, a nie w czasie wykonania.

Aby uzyskać więcej szczegółów zobacz:
- [`<script setup>` - `defineProps()` & `defineEmits()`](/api/sfc-script-setup.html#defineprops-defineemits)
- [`<script setup>` - `defineExpose()`](/api/sfc-script-setup.html#defineexpose)

## komponent {#component}

Termin *komponent* nie jest unikalny dla Vue. Jest wspólny dla wielu frameworków UI. Opisuje fragment interfejsu użytkownika, taki jak przycisk lub pole wyboru. Komponenty mogą być również łączone w celu tworzenia większych komponentów.

Komponenty są głównym mechanizmem dostarczanym przez Vue do podziału interfejsu użytkownika na mniejsze części, zarówno w celu poprawy możliwości utrzymania, jak i umożliwienia ponownego wykorzystania kodu.

Komponent Vue jest obiektem. Wszystkie właściwości są opcjonalne, ale szablon lub funkcja renderująca są wymagane, aby komponent mógł się wyrenderować. Na przykład, następujący obiekt byłby prawidłowym komponentem:

```js
const HelloWorldComponent = {
  render() {
    return 'Witaj świecie!'
  }
}
```

W praktyce, większość aplikacji Vue jest pisana przy użyciu [Komponentów Jednoplikowych](#single-file-component) (pliki `.vue`). Choć komponenty te mogą na pierwszy rzut oka nie wyglądać jak obiekty, kompilator SFC przekonwertuje je na obiekt, który jest używany jako domyślny eksport pliku. Z zewnętrznej perspektywy, plik `.vue` jest po prostu modułem ES, który eksportuje obiekt komponentu.

Właściwości obiektu komponentu są zwykle określane jako *opcje*. Stąd właśnie pochodzi nazwa [Options API](#options-api).

Opcje komponentu definiują sposób tworzenia instancji tego komponentu. Komponenty są koncepcyjnie podobne do klas, chociaż Vue nie używa rzeczywistych klas JavaScript do ich definiowania.

Termin komponent może być również używany w luźniejszym znaczeniu w odniesieniu do instancji komponentów.

Aby uzyskać więcej szczegółów zobacz:
- [Przewodnik - Podstawy Komponentów](/guide/essentials/component-basics.html)

Słowo 'komponent' występuje również w kilku innych terminach:
- [komponent asynchroniczny](#async-component)
- [komponent dynamiczny](#dynamic-component)
- [komponent funkcyjny](#functional-component)
- [Web Component](#web-component)

## composable {#composable}

Termin *composable* opisuje powszechny wzorzec użycia w Vue. Nie jest to oddzielna funkcja Vue, to po prostu sposób wykorzystania [Composition API](#composition-api) frameworka.

* Composable jest funkcją.
* Composables są używane do enkapsulacji i ponownego wykorzystania logiki stanowej.
* Nazwa funkcji zazwyczaj zaczyna się od `use`, aby inni programiści wiedzieli, że jest to composable.
* Oczekuje się, że funkcja zostanie wywołana podczas synchronicznego wykonywania funkcji `setup()` komponentu (lub równoważnie, podczas wykonywania bloku `<script setup>`). Wiąże to wywołanie composable z aktualnym kontekstem komponentu, np. poprzez wywołania `provide()`, `inject()` lub `onMounted()`.
* Composables zazwyczaj zwracają zwykły obiekt, nie obiekt reaktywny. Ten obiekt zwykle zawiera refs i funkcje i oczekuje się, że zostanie zdestrukturyzowany w kodzie wywołującym.

Jak w przypadku wielu wzorców, mogą pojawić się pewne rozbieżności co do tego, czy konkretny kod kwalifikuje się do tej nazwy. Nie wszystkie funkcje narzędziowe JavaScript są composables. Jeśli funkcja nie używa Composition API, to prawdopodobnie nie jest composable. Jeśli nie oczekuje się, że zostanie wywołana podczas synchronicznego wykonywania `setup()`, to prawdopodobnie nie jest composable. Composables są specjalnie używane do enkapsulacji logiki stanowej, nie są to tylko konwencje nazewnictwa funkcji.

Zobacz [Przewodnik - Composables](/guide/reusability/composables.html), aby uzyskać więcej szczegółów na temat pisania composables.

## Composition API {#composition-api}

*Composition API* to zbiór funkcji używanych do pisania komponentów i composables w Vue.

Termin ten jest również używany do opisania jednego z dwóch głównych stylów używanych do pisania komponentów, drugim jest [Options API](#options-api). Komponenty napisane przy użyciu Composition API używają albo `<script setup>` albo jawnej funkcji `setup()`.

Zobacz [FAQ Composition API](/guide/extras/composition-api-faq), aby uzyskać więcej szczegółów.

## custom element {#custom-element}

*Custom element* jest funkcją standardu [Web Components](#web-component), która jest zaimplementowana w nowoczesnych przeglądarkach internetowych. Odnosi się do możliwości używania niestandardowego elementu HTML w znacznikach HTML w celu umieszczenia Web Component w danym miejscu strony.

Vue ma wbudowane wsparcie dla renderowania custom elements i pozwala na ich bezpośrednie użycie w szablonach komponentów Vue.

Custom elements nie należy mylić z możliwością używania komponentów Vue jako znaczników w szablonie innego komponentu Vue. Custom elements są używane do tworzenia Web Components, nie komponentów Vue.

Aby uzyskać więcej szczegółów zobacz:
- [Przewodnik - Vue i Web Components](/guide/extras/web-components.html)

## dyrektywa {#directive}

Termin *dyrektywa* odnosi się do atrybutów szablonu zaczynających się od prefiksu `v-` lub ich równoważnych skrótów.

Wbudowane dyrektywy obejmują `v-if`, `v-for`, `v-bind`, `v-on` i `v-slot`.

Vue wspiera również tworzenie własnych dyrektyw, chociaż są one zazwyczaj używane tylko jako 'furtka bezpieczeństwa' do bezpośredniej manipulacji węzłami DOM. Własne dyrektywy generalnie nie mogą być używane do odtworzenia funkcjonalności wbudowanych dyrektyw.

Aby uzyskać więcej szczegółów zobacz:
- [Przewodnik - Składnia Szablonów - Dyrektywy](/guide/essentials/template-syntax.html#directives)
- [Przewodnik - Własne Dyrektywy](/guide/reusability/custom-directives.html)

## komponent dynamiczny {#dynamic-component}

Termin *komponent dynamiczny* jest używany do opisania przypadków, gdy wybór komponentu potomnego do wyrenderowania musi być dokonany dynamicznie. Zazwyczaj osiąga się to za pomocą `<component :is="type">`.

Komponent dynamiczny nie jest specjalnym typem komponentu. Każdy komponent może być użyty jako komponent dynamiczny. To wybór komponentu jest dynamiczny, a nie sam komponent.

Aby uzyskać więcej szczegółów zobacz:
- [Przewodnik - Podstawy Komponentów - Komponenty Dynamiczne](/guide/essentials/component-basics.html#dynamic-components)

## efekt {#effect}

Zobacz [efekt reaktywny](#reactive-effect) i [efekt uboczny](#side-effect).

## zdarzenie {#event}

Używanie zdarzeń do komunikacji między różnymi częściami programu jest wspólne dla wielu różnych obszarów programowania. W Vue termin ten jest powszechnie stosowany zarówno do natywnych zdarzeń elementów HTML, jak i zdarzeń komponentów Vue. Dyrektywa `v-on` jest używana w szablonach do nasłuchiwania obu typów zdarzeń.

Aby uzyskać więcej szczegółów zobacz:
- [Przewodnik - Obsługa Zdarzeń](/guide/essentials/event-handling.html)
- [Przewodnik - Zdarzenia Komponentów](/guide/components/events.html)

## fragment {#fragment}

Termin *fragment* odnosi się do specjalnego typu [VNode](#vnode), który jest używany jako rodzic dla innych VNodes, ale sam nie renderuje żadnych elementów.

Nazwa pochodzi z podobnej koncepcji [`DocumentFragment`](https://developer.mozilla.org/en-US/docs/Web/API/DocumentFragment) w natywnym API DOM.

Fragmenty są używane do obsługi komponentów z wieloma węzłami głównymi. Chociaż takie komponenty mogą wydawać się mieć wiele korzeni, w tle używają węzła fragment jako pojedynczego korzenia, będącego rodzicem węzłów 'głównych'.

Fragmenty są również używane przez kompilator szablonów jako sposób na opakowywanie wielu węzłów dynamicznych, np. tych utworzonych przez `v-for` lub `v-if`. Pozwala to na przekazanie dodatkowych wskazówek do algorytmu łatania [VDOM](#virtual-dom). Większość tego jest obsługiwana wewnętrznie, ale jednym miejscem, gdzie możesz się z tym bezpośrednio spotkać, jest używanie `key` na znaczniku `<template>` z `v-for`. W tym scenariuszu, `key` jest dodawany jako [prop](#prop) do VNode fragmentu.

Węzły fragmentów są obecnie renderowane do DOM jako puste węzły tekstowe, choć jest to szczegół implementacyjny. Możesz napotkać te węzły tekstowe, jeśli używasz `$el` lub próbujesz przejść przez DOM używając wbudowanych API przeglądarki.

## komponent funkcyjny {#functional-component}

Definicja komponentu jest zazwyczaj obiektem zawierającym opcje. Może nie wyglądać to w ten sposób, jeśli używasz `<script setup>`, ale komponent wyeksportowany z pliku `.vue` nadal będzie obiektem.

*Komponent funkcyjny* jest alternatywną formą komponentu, która jest deklarowana przy użyciu funkcji zamiast obiektu. Ta funkcja działa jako [funkcja renderująca](#render-function) dla komponentu.

Komponent funkcyjny nie może mieć własnego stanu. Nie przechodzi też przez normalny cykl życia komponentu, więc nie można używać haków cyklu życia. Sprawia to, że są one nieco lżejsze niż normalne komponenty stanowe.

Aby uzyskać więcej szczegółów zobacz:
- [Przewodnik - Funkcje Renderujące i JSX - Komponenty Funkcyjne](/guide/extras/render-function.html#functional-components)

## hoisting {#hoisting}

Termin *hoisting* jest używany do opisania uruchamiania sekcji kodu przed jej osiągnięciem, przed innym kodem. Wykonanie jest 'podciągane' do wcześniejszego punktu.

JavaScript używa hoistingu dla niektórych konstrukcji, takich jak `var`, `import` i deklaracje funkcji.

W kontekście Vue, kompilator szablonów stosuje *static hoisting* w celu poprawy wydajności. Podczas konwertowania szablonu na funkcję renderującą, VNodes odpowiadające statycznej zawartości mogą być utworzone raz i następnie ponownie użyte. Te statyczne VNodes są opisywane jako hoisted, ponieważ są tworzone poza funkcją renderującą, przed jej uruchomieniem. Podobna forma hoistingu jest stosowana do statycznych obiektów lub tablic, które są generowane przez kompilator szablonów.

Aby uzyskać więcej szczegółów zobacz:
- [Przewodnik - Mechanizm Renderowania - Static Hoisting](/guide/extras/rendering-mechanism.html#static-hoisting)

## szablon w-DOM {#in-dom-template}

Istnieją różne sposoby określania szablonu dla komponentu. W większości przypadków szablon jest dostarczany jako ciąg znaków.

Termin *szablon w-DOM* odnosi się do scenariusza, w którym szablon jest dostarczany w formie węzłów DOM, zamiast ciągu znaków. Vue następnie konwertuje węzły DOM na ciąg znaków szablonu za pomocą `innerHTML`.

Zazwyczaj szablon w-DOM zaczyna się jako znaczniki HTML napisane bezpośrednio w HTML strony. Przeglądarka następnie parsuje je do węzłów DOM, które Vue wykorzystuje do odczytania `innerHTML`.

Aby uzyskać więcej szczegółów zobacz:
- [Przewodnik - Tworzenie Aplikacji - Szablon Komponentu Głównego w-DOM](/guide/essentials/application.html#in-dom-root-component-template)
- [Przewodnik - Podstawy Komponentów - Uwagi dotyczące parsowania szablonów w-DOM](/guide/essentials/component-basics.html#in-dom-template-parsing-caveats)
- [Opcje: Renderowanie - template](/api/options-rendering.html#template)

## inject {#inject}

Zobacz [provide / inject](#provide-inject).

## hooki cyklu życia {#lifecycle-hooks}

Instancja komponentu Vue przechodzi przez cykl życia. Na przykład, jest tworzona, montowana, aktualizowana i odmontowywana.

*Hooki cyklu życia* są sposobem na nasłuchiwanie tych zdarzeń cyklu życia.

W Options API, każdy hook jest dostarczany jako osobna opcja, np. `mounted`. Composition API używa zamiast tego funkcji, takich jak `onMounted()`.

Aby uzyskać więcej szczegółów zobacz:
- [Przewodnik - Hooki Cyklu Życia](/guide/essentials/lifecycle.html)

## makro {#macro}

Zobacz [makro kompilatora](#compiler-macro).

## nazwany slot {#named-slot}

Komponent może mieć wiele slotów, rozróżnianych po nazwie. Sloty inne niż domyślny slot są określane jako *nazwane sloty*.

Aby uzyskać więcej szczegółów zobacz:
- [Przewodnik - Sloty - Nazwane Sloty](/guide/components/slots.html#named-slots)

## Options API {#options-api}

Komponenty Vue są definiowane przy użyciu obiektów. Właściwości tych obiektów komponentów są znane jako *opcje*.

Komponenty mogą być pisane w dwóch stylach. Jeden styl używa [Composition API](#composition-api) w połączeniu z `setup` (albo poprzez opcję `setup()` albo `<script setup>`). Drugi styl wykorzystuje bardzo mało bezpośrednio Composition API, zamiast tego używając różnych opcji komponentu do osiągnięcia podobnego rezultatu. Opcje komponentu, które są używane w ten sposób, są określane jako *Options API*.

Options API zawiera opcje takie jak `data()`, `computed`, `methods` i `created()`.

Niektóre opcje, takie jak `props`, `emits` i `inheritAttrs`, mogą być używane podczas tworzenia komponentów z dowolnym API. Ponieważ są to opcje komponentu, mogłyby być uznane za część Options API. Jednak ponieważ te opcje są również używane w połączeniu z `setup()`, zazwyczaj bardziej użyteczne jest myślenie o nich jako wspólnych dla obu stylów komponentów.

Sama funkcja `setup()` jest opcją komponentu, więc *mogłaby* być opisana jako część Options API. Jednak nie tak termin 'Options API' jest normalnie używany. Zamiast tego, funkcja `setup()` jest uważana za część Composition API.

## plugin {#plugin}

Podczas gdy termin *plugin* może być używany w różnych kontekstach, Vue ma specyficzną koncepcję pluginu jako sposobu na dodawanie funkcjonalności do aplikacji.

Pluginy są dodawane do aplikacji poprzez wywołanie `app.use(plugin)`. Sam plugin jest albo funkcją, albo obiektem z funkcją `install`. Ta funkcja otrzyma instancję aplikacji i może wtedy zrobić wszystko, co jest potrzebne.

Aby uzyskać więcej szczegółów zobacz:
- [Przewodnik - Pluginy](/guide/reusability/plugins.html)

## prop {#prop}

Istnieją trzy powszechne zastosowania terminu *prop* w Vue:

* Props komponentu
* Props VNode
* Props slotów

*Właściwości komponentu* (ang. Component props) to to, co większość osób rozumie jako props. Są one jawnie zdefiniowane przez komponent przy użyciu `defineProps()` lub opcji `props`.

Termin *właściwości VNode* (ang. VNode props) odnosi się do właściwości obiektu przekazywanego jako drugi argument do `h()`. Mogą one zawierać właściwości komponentu, ale także zdarzenia komponentu, zdarzenia DOM, atrybuty DOM i właściwości DOM. Zazwyczaj napotkasz właściwości VNode tylko wtedy, gdy pracujesz z funkcjami renderującymi w celu bezpośredniej manipulacji VNodes.

*Właściwości slotu* (ang. Slot props) to właściwości przekazywane do slotu z zakresem.

We wszystkich przypadkach props to właściwości, które są przekazywane z zewnątrz.

Podczas gdy słowo props pochodzi od słowa *properties* (właściwości), termin props ma znacznie bardziej konkretne znaczenie w kontekście Vue. Powinieneś unikać używania go jako skrótu od properties.

Aby uzyskać więcej szczegółów, zobacz:
- [Przewodnik - Props](/guide/components/props.html)
- [Przewodnik - Funkcje renderujące i JSX](/guide/extras/render-function.html)
- [Przewodnik - Sloty - Sloty z zakresem](/guide/components/slots.html#scoped-slots)

## provide / inject {#provide-inject}

`provide` i `inject` są formą komunikacji między komponentami.

Kiedy komponent *udostępnia* wartość, wszystkie komponenty potomne mogą następnie zdecydować się na pobranie tej wartości za pomocą `inject`. W przeciwieństwie do props, komponent udostępniający nie wie dokładnie, który komponent otrzymuje wartość.

`provide` i `inject` są czasami używane w celu uniknięcia *prop drilling*. Mogą być również używane jako niejawny sposób komunikacji komponentu z zawartością jego slotów.

`provide` może być również używane na poziomie aplikacji, udostępniając wartość wszystkim komponentom w tej aplikacji.

Aby uzyskać więcej szczegółów, zobacz:
- [Przewodnik - provide / inject](/guide/components/provide-inject.html)

## efekt reaktywny {#reactive-effect}

*Efekt reaktywny* jest częścią systemu reaktywności Vue. Odnosi się do procesu śledzenia zależności funkcji i ponownego uruchamiania tej funkcji, gdy wartości tych zależności ulegają zmianie.

`watchEffect()` jest najbardziej bezpośrednim sposobem tworzenia efektu. Różne inne części Vue używają efektów wewnętrznie, np. aktualizacje renderowania komponentów, `computed()` i `watch()`.

Vue może śledzić zależności reaktywne tylko w ramach efektu reaktywnego. Jeśli wartość właściwości jest odczytywana poza efektem reaktywnym, "straci" reaktywność w tym sensie, że Vue nie będzie wiedzieć, co zrobić, jeśli ta właściwość następnie się zmieni.

Termin pochodzi od "efektu ubocznego" (ang. side effect). Wywołanie funkcji efektu jest efektem ubocznym zmiany wartości właściwości.

Aby uzyskać więcej szczegółów, zobacz:
- [Przewodnik - Reaktywność w szczegółach](/guide/extras/reactivity-in-depth.html)

## reaktywność {#reactivity}

Ogólnie rzecz biorąc, *reaktywność* odnosi się do zdolności automatycznego wykonywania działań w odpowiedzi na zmiany danych. Na przykład, aktualizacja DOM lub wykonanie żądania sieciowego, gdy zmienia się wartość danych.

W kontekście Vue, reaktywność jest używana do opisania zbioru funkcji. Te funkcje łączą się, tworząc *system reaktywności*, który jest udostępniany poprzez [API Reaktywności](#reactivity-api).

Istnieją różne sposoby implementacji systemu reaktywności. Na przykład, mogłoby to być zrobione poprzez statyczną analizę kodu w celu określenia jego zależności. Jednak Vue nie wykorzystuje tego rodzaju systemu reaktywności.

Zamiast tego system reaktywności Vue śledzi dostęp do właściwości w czasie wykonywania. Robi to przy użyciu zarówno wrapperów Proxy, jak i funkcji [getter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get#description)/[setter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/set#description) dla właściwości.

Aby uzyskać więcej szczegółów, zobacz:
- [Przewodnik - Podstawy reaktywności](/guide/essentials/reactivity-fundamentals.html)
- [Przewodnik - Reaktywność w szczegółach](/guide/extras/reactivity-in-depth.html)

## API Reaktywności {#reactivity-api}

*API Reaktywności* to zbiór podstawowych funkcji Vue związanych z [reaktywnością](#reactivity). Mogą być używane niezależnie od komponentów. Zawiera funkcje takie jak `ref()`, `reactive()`, `computed()`, `watch()` i `watchEffect()`.

API Reaktywności jest podzbiorem API Kompozycyjnego.

Aby uzyskać więcej szczegółów, zobacz:
- [API Reaktywności: Podstawowe](/api/reactivity-core.html)
- [API Reaktywności: Narzędzia](/api/reactivity-utilities.html)
- [API Reaktywności: Zaawansowane](/api/reactivity-advanced.html)

## ref {#ref}

> Ten wpis dotyczy użycia `ref` do reaktywności. Informacje o atrybucie `ref` używanym w szablonach znajdują się w sekcji [template ref](#template-ref).

`ref` jest częścią systemu reaktywności Vue. Jest to obiekt z pojedynczą reaktywną właściwością o nazwie `value`.

Istnieją różne typy ref. Na przykład, refy mogą być tworzone przy użyciu `ref()`, `shallowRef()`, `computed()` i `customRef()`. Funkcja `isRef()` może być używana do sprawdzenia, czy obiekt jest refem, a `isReadonly()` może być używana do sprawdzenia, czy ref pozwala na bezpośrednie przypisanie jego wartości.

Aby uzyskać więcej szczegółów, zobacz:
- [Przewodnik - Podstawy reaktywności](/guide/essentials/reactivity-fundamentals.html)
- [API Reaktywności: Podstawowe](/api/reactivity-core.html)
- [API Reaktywności: Narzędzia](/api/reactivity-utilities.html)
- [API Reaktywności: Zaawansowane](/api/reactivity-advanced.html)

## funkcja renderująca {#render-function}

*Funkcja renderująca* jest częścią komponentu, która generuje VNodes używane podczas renderowania. Szablony są kompilowane do funkcji renderujących.

Aby uzyskać więcej szczegółów, zobacz:
- [Przewodnik - Funkcje renderujące i JSX](/guide/extras/render-function.html)

## planista {#scheduler}

*Planista* jest częścią wewnętrznych mechanizmów Vue, która kontroluje czas wykonywania [efektów reaktywnych](#reactive-effect).

Gdy stan reaktywny się zmienia, Vue nie wywołuje natychmiast aktualizacji renderowania. Zamiast tego grupuje je razem, używając kolejki. Zapewnia to, że komponent renderuje się tylko raz, nawet jeśli wprowadzono wiele zmian w bazowych danych.

[Obserwatory](/guide/essentials/watchers.html) są również grupowane przy użyciu kolejki planisty. Obserwatory z `flush: 'pre'` (domyślnie) będą uruchamiane przed renderowaniem komponentu, podczas gdy te z `flush: 'post'` będą uruchamiane po renderowaniu komponentu.

Zadania w planiście są również używane do wykonywania różnych innych wewnętrznych zadań, takich jak wyzwalanie niektórych [hooków cyklu życia](#lifecycle-hooks) i aktualizacja [referencji szablonu](#template-ref).

## slot z zakresem {#scoped-slot}

Termin *slot z zakresem* (ang. scoped slot) jest używany do określenia [slotu](#slot), który otrzymuje [właściwości](#prop).

Historycznie Vue wprowadzało znacznie większe rozróżnienie między slotami z zakresem i bez zakresu. W pewnym stopniu można je było traktować jako dwie oddzielne funkcje, zjednoczone za pomocą wspólnej składni szablonu.

W Vue 3 API slotów zostało uproszczone, aby wszystkie sloty zachowywały się jak sloty z zakresem. Jednak przypadki użycia slotów z zakresem i bez zakresu często się różnią, więc termin nadal jest przydatny jako sposób odnoszenia się do slotów z właściwościami.

Właściwości przekazane do slotu mogą być używane tylko w określonym obszarze szablonu nadrzędnego, odpowiedzialnym za zdefiniowanie zawartości slotu. Ten region szablonu zachowuje się jak zakres zmiennych dla właściwości, stąd nazwa "slot z zakresem".

Aby uzyskać więcej szczegółów, zobacz:
- [Przewodnik - Sloty - Sloty z zakresem](/guide/components/slots.html#scoped-slots)

## SFC {#sfc}

Zobacz [Komponent Jednoplikowy](#single-file-component).

## efekt uboczny {#side-effect}

Termin *efekt uboczny* (ang. side effect) nie jest specyficzny dla Vue. Jest używany do opisania operacji lub funkcji, które robią coś poza ich lokalnym zakresem.

Na przykład, w kontekście ustawiania właściwości jak `user.name = null`, oczekuje się, że zmieni to wartość `user.name`. Jeśli robi również coś innego, jak wyzwalanie systemu reaktywności Vue, to zostałoby to opisane jako efekt uboczny. Jest to źródło terminu [efekt reaktywny](#reactive-effect) w Vue.

Kiedy mówi się, że funkcja ma efekty uboczne, oznacza to, że funkcja wykonuje jakiegoś rodzaju działanie, które jest obserwowalne poza funkcją, oprócz samego zwracania wartości. Może to oznaczać, że aktualizuje wartość w stanie lub wywołuje żądanie sieciowe.

Termin jest często używany przy opisywaniu renderowania lub właściwości obliczanych. Uważa się za najlepszą praktykę, aby renderowanie nie miało efektów ubocznych. Podobnie, funkcja getter dla właściwości obliczanej nie powinna mieć efektów ubocznych.

## Komponent Jednoplikowy {#single-file-component}

Termin *Komponent Jednoplikowy* (ang. Single-File Component), lub SFC, odnosi się do formatu pliku `.vue`, który jest powszechnie używany dla komponentów Vue.

Zobacz także:
- [Przewodnik - Komponenty Jednoplikowe](/guide/scaling-up/sfc.html)
- [Specyfikacja składni SFC](/api/sfc-spec.html)

## slot {#slot}

Sloty są używane do przekazywania treści do komponentów potomnych. Podczas gdy props są używane do przekazywania wartości danych, sloty są używane do przekazywania bogatszej treści składającej się z elementów HTML i innych komponentów Vue.

Aby uzyskać więcej szczegółów, zobacz:
- [Przewodnik - Sloty](/guide/components/slots.html)

## referencja szablonu {#template-ref}

Termin *referencja szablonu* (ang. template ref) odnosi się do użycia atrybutu `ref` na tagu w szablonie. Po wyrenderowaniu komponentu atrybut ten jest używany do wypełnienia odpowiedniej właściwości elementem HTML lub instancją komponentu, która odpowiada tagowi w szablonie.

Jeśli używasz API Opcji, to referencje są udostępniane poprzez właściwości obiektu `$refs`.

W przypadku API Kompozycyjnego, referencje szablonu wypełniają reaktywny [ref](#ref) o tej samej nazwie.

Referencji szablonu nie należy mylić z reaktywnymi refami znajdującymi się w systemie reaktywności Vue.

Aby uzyskać więcej szczegółów, zobacz:
- [Przewodnik - Referencje szablonu](/guide/essentials/template-refs.html)

## VDOM {#vdom}

Zobacz [wirtualny DOM](#virtual-dom).

## wirtualny DOM {#virtual-dom}

Termin *wirtualny DOM* (VDOM) nie jest unikalny dla Vue. Jest to powszechne podejście używane przez kilka frameworków webowych do zarządzania aktualizacjami interfejsu użytkownika.

Przeglądarki używają drzewa węzłów do reprezentowania aktualnego stanu strony. To drzewo i interfejsy API JavaScript używane do interakcji z nim są określane jako *model obiektowy dokumentu* lub *DOM*.

Manipulowanie DOM-em jest głównym wąskim gardłem wydajności. Wirtualny DOM dostarcza jedną ze strategii zarządzania tym.

Zamiast bezpośrednio tworzyć węzły DOM, komponenty Vue generują opis węzłów DOM, które chciałyby utworzyć. Te deskryptory są zwykłymi obiektami JavaScript, znanymi jako VNodes (węzły wirtualnego DOM). Tworzenie VNodes jest stosunkowo tanie.

Za każdym razem, gdy komponent renderuje się ponownie, nowe drzewo VNodes jest porównywane z poprzednim drzewem VNodes, a wszelkie różnice są następnie aplikowane do rzeczywistego DOM. Jeśli nic się nie zmieniło, DOM nie musi być modyfikowany.

Vue używa podejścia hybrydowego, które nazywamy [Wirtualnym DOM-em z informacją kompilatora](/guide/extras/rendering-mechanism.html#compiler-informed-virtual-dom). Kompilator szablonów Vue jest w stanie zastosować optymalizacje wydajności na podstawie statycznej analizy szablonu. Zamiast wykonywać pełne porównanie starych i nowych drzew VNode komponentu w czasie wykonywania, Vue może wykorzystać informacje wyodrębnione przez kompilator do ograniczenia porównania tylko do tych części drzewa, które faktycznie mogą się zmienić.

Aby uzyskać więcej szczegółów, zobacz:
- [Przewodnik - Mechanizm renderowania](/guide/extras/rendering-mechanism.html)
- [Przewodnik - Funkcje renderujące i JSX](/guide/extras/render-function.html)

## VNode {#vnode}

*VNode* to *węzeł wirtualnego DOM*. Mogą być tworzone przy użyciu funkcji [`h()`](/api/render-function.html#h).

Zobacz [wirtualny DOM](#virtual-dom), aby uzyskać więcej informacji.

## Web Component {#web-component}

Standard *Web Components* to zbiór funkcji zaimplementowanych w nowoczesnych przeglądarkach internetowych.

Komponenty Vue nie są Web Components, ale `defineCustomElement()` może być użyte do utworzenia [elementu niestandardowego](#custom-element) z komponentu Vue. Vue obsługuje również używanie elementów niestandardowych wewnątrz komponentów Vue.

Aby uzyskać więcej szczegółów, zobacz:
- [Przewodnik - Vue i Web Components](/guide/extras/web-components.html)
