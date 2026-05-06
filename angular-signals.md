# Czym jest Angular Signals?

**Angular Signals** to mechanizm reaktywności w Angularze służący do przechowywania i śledzenia stanu aplikacji. Najprościej: **signal to opakowanie na wartość**, które informuje Angulara, że ta wartość się zmieniła. Angular może wtedy dokładniej wiedzieć, gdzie dana wartość została użyta i zoptymalizować odświeżanie UI.

Oficjalna dokumentacja:

- [Angular Signals Guide](https://angular.dev/guide/signals)
- [Angular Signals Essentials](https://angular.dev/essentials/signals)
- [Angular RxJS Interop](https://angular.dev/ecosystem/rxjs-interop)

## Podstawowy przykład

```ts
import { signal, computed } from '@angular/core';

export class CounterComponent {
  count = signal(0);

  doubled = computed(() => this.count() * 2);

  increment(): void {
    this.count.update(value => value + 1);
  }

  reset(): void {
    this.count.set(0);
  }
}
```

W template:

```html
<p>Count: {{ count() }}</p>
<p>Doubled: {{ doubled() }}</p>

<button (click)="increment()">+</button>
<button (click)="reset()">Reset</button>
```

Ważna rzecz: **signal odczytujesz jak funkcję**, czyli `count()`, a nie jak zwykłą właściwość `count`. Dzięki temu Angular wie, że dany fragment kodu albo template’u zależy od tego signala.

## Podstawowe elementy

| Element | Do czego służy |
|---|---|
| `signal()` | Przechowuje stan, np. licznik, użytkownika, flagę UI |
| `.set()` | Ustawia nową wartość |
| `.update()` | Wylicza nową wartość na podstawie poprzedniej |
| `computed()` | Tworzy wartość pochodną zależną od innych signals |
| `effect()` | Uruchamia efekt uboczny, gdy zmieni się signal |

## `computed()`

`computed()` tworzy wartość pochodną, której nie ustawiasz ręcznie. Jej wartość zmienia się automatycznie, gdy zmienią się signale, z których korzysta.

```ts
firstName = signal('Jan');
lastName = signal('Kowalski');

fullName = computed(() => `${this.firstName()} ${this.lastName()}`);
```

Gdy zmienisz `firstName` albo `lastName`, `fullName` zostanie przeliczony automatycznie.

## `effect()`

`effect()` służy raczej do integracji ze światem zewnętrznym, na przykład do logowania, synchronizacji z `localStorage`, obsługi niestandardowego DOM albo bibliotek zewnętrznych.

Nie warto nadużywać `effect()`. Dla wartości pochodnych lepszy jest zwykle `computed()`.

```ts
import { effect, signal } from '@angular/core';

export class CounterComponent {
  count = signal(0);

  constructor() {
    effect(() => {
      localStorage.setItem('count', this.count().toString());
    });
  }
}
```

## Po co to w Angularze?

Signals rozwiązują problem bardziej precyzyjnego reagowania na zmianę stanu. Zamiast myśleć „co trzeba ręcznie odświeżyć?”, definiujesz zależności między danymi, a Angular wie, które miejsca zależą od których wartości.

Dzięki temu kod UI może być bardziej przewidywalny, a Angular dostaje więcej informacji do optymalizacji renderowania.

## Signals a RxJS

Signals **nie zastępują całkowicie RxJS**. Bardziej sensowne jest traktowanie ich jako dwóch narzędzi do różnych zastosowań.

**Signals** są bardzo wygodne do lokalnego i synchronicznego stanu UI:

```ts
isSidebarOpen = signal(false);
selectedUserId = signal<number | null>(null);
searchPhrase = signal('');
```

**RxJS** nadal dobrze pasuje do strumieni zdarzeń, HTTP, WebSocketów, debounce, cancelowania requestów i złożonych pipeline’ów asynchronicznych:

```ts
this.searchControl.valueChanges.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(query => this.usersService.search(query))
);
```

Angular ma też mechanizmy interoperacyjności z RxJS, więc oba podejścia mogą spokojnie współistnieć.

## Intuicyjne porównanie

Możesz myśleć o signalu trochę jak o prostszym `BehaviorSubject`, ale z innym API i głębszą integracją z Angularem.

```ts
// RxJS
count$ = new BehaviorSubject(0);
count$.next(1);

// Signal
count = signal(0);
count.set(1);
```

Różnica jest taka, że signal czytasz synchronicznie:

```ts
const currentValue = this.count();
```

Nie potrzebujesz `subscribe()` ani `async pipe`, żeby odczytać aktualną wartość.

## Kiedy używać signals?

Używaj signals szczególnie do lokalnego stanu komponentów i prostego stanu UI:

```ts
isLoading = signal(false);
errorMessage = signal<string | null>(null);
items = signal<Item[]>([]);
selectedItem = signal<Item | null>(null);
```

A `computed()` do wartości pochodnych:

```ts
hasItems = computed(() => this.items().length > 0);

visibleItems = computed(() =>
  this.items().filter(item => item.isVisible)
);
```

## Uwaga na tablice i obiekty

Przy obiektach i tablicach ważne jest, żeby nie mutować wartości bezpośrednio.

Źle:

```ts
this.items().push(newItem);
```

Dobrze:

```ts
this.items.update(items => [...items, newItem]);
```

Dzięki temu Angular dostaje jasny sygnał, że stan się zmienił.

## Podsumowanie

**Angular Signals to natywny, prosty mechanizm reaktywnego stanu w Angularze, oparty o `signal()`, `computed()` i `effect()`, który pozwala pisać bardziej przewidywalny kod UI i daje Angularowi więcej informacji do optymalizacji renderowania.**
