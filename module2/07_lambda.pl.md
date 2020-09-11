<!-- .slide: data-background="#111111" -->
# Wyrażenia lambda w skrócie

___

## Wyrażenia lambda

Lambda przydatna jest w programowaniu funkcjonalnym, funkcjach anonimowych, bardziej uniwersalnym przekazywaniu funkcji.
<!-- .element: class="fragment fade-in" -->

Wyrażenia lambda są definiowane bezpośrednio w miejscu ich użycia. Zwykle jest używany jako parametr innej funkcji, która oczekuje od wskaźnika do funkcji lub funktora - na ogół jest to obiekt wywoływalny.
<!-- .element: class="fragment fade-in" -->

Każde wyrażenie lambda powoduje, że kompilator tworzy unikalną klasę domknięcia (closure class), która implementuje operator funkcji z kodem z wyrażenia.
<!-- .element: class="fragment fade-in" -->

Domknięcie jest obiektem klasy domknięcia. W zależności od typu przechwytywanego obiekt ten przechowuje referencje lub kopie zmiennych lokalnych.
<!-- .element: class="fragment fade-in" -->

___

## Podstawowe wyrażenia lambda

```c++
[](){}; // pusta lambda
[] { std::cout << "hello world" << std::endl; } // anonimowa lambda

auto l = [] (int x, int y) { return x + y; };
auto result = l(2, 3); // result = 5
```

___

## Zwrócony typ lambdy

Z C++14 automatyczna dedukcja typu zwracanego z lambd działa całkiem nieźle i zwykle nie ma potrzeby bezpośredniego podawania zwracanego typu. Można to jednak zrobić za pomocą operatora strzałki.
<!-- .element: class="fragment fade-in" -->

```c++
[](bool condition) -> int {
    if (condition) {
        return 1;
    } else {
        return 2;
    }
}
```
<!-- .element: class="fragment fade-in" -->

___

## Predykaty

Wyrażenia lambda są zwykle używane do tworzenia predykatów i funktorów wymaganych przez algorytmy w bibliotece standardowej (np. `std::sort`).
<!-- .element: class="fragment fade-in" -->

```c++

std::array<double, 6> values = { 5.0, 4.0, -1.4, 7.9, -8.22, 0.4 };

std::sort(values.begin(), values.end(), [](double a, double b) {
    return std::abs(a) < std::abs(b); // posortuj wartości używając
                                      // wartości bezwzględnych
});
```
<!-- .element: class="fragment fade-in" -->

Wynik: `0.4, -1.4, 4.0, 5.0, 7.9, -8.22`
<!-- .element: class="fragment fade-in" -->

___

## Lista przechwytywania

Wewnątrz nawiasów `[]` możemy zawrzeć elementy, które lambda powinna wychwycić z zakresu, w jakim jest tworzona. Można również określić sposób ich przechwytywania.

* <!-- .element: class="fragment fade-in" --> puste nawiasy <code>[]</code> oznaczają, że wewnątrz lambdy nie można użyć żadnej zmiennej z zewnętrznego zakresu.
* <!-- .element: class="fragment fade-in" --> <code>[&]</code> oznacza, że ​​każda zmienna z zakresu zewnętrznego jest przechwytywana przez referencję, w tym wskaźnik <code>this</code>.
  * Funktor utworzony za pomocą wyrażenia lambda może czytać i zapisywać do dowolnej przechwyconej zmiennej i wszystkie są przechowywane wewnątrz przez referencję lambdy.
* <!-- .element: class="fragment fade-in" --> <code>[=]</code> oznacza, że ​​każda zmienna z zakresu zewnętrznego jest przechwytywana według wartości, w tym wskaźnik <code>this</code>.
  * Wszystkie używane zmienne z zewnętrznego zakresu są kopiowane do wyrażenia lambda i mogą być tylko odczytywane z wyjątkiem wskaźnika `this`.
  * wskaźnik `this` po skopiowaniu pozwala lambdzie zmodyfikować wszystkie zmienne, na które wskazuje.
  * Potrzeba słowa kluczowego `mutable`, aby zmodyfikować wartości przechwycone przez `=`.

___

## Lista przechwytywania

* <!-- .element: class="fragment fade-in" --> <code>[capture-list]</code> umożliwia jawne przechwytywanie zmiennych z zewnętrznego zakresu poprzez wymienienie ich nazw na liście.
  * Domyślnie wszystkie elementy są przechwytywane według wartości.
  * Jeśli zmienna ma zostać ujęta przez referencję, należy ją poprzedzić znakiem `&` co oznacza przechwytywanie przez referencję.
  * Przykład: `[a, &b]`
* <!-- .element: class="fragment fade-in" --> <code>[*this]</code> (C++17) przechwytuje ten wskaźnik według wartości (tworzy kopię tego obiektu).

___

## Lista przechwytywania

```c++
int a {5};
auto add5 = [=](int x) { return x + a; };

int counter {};
auto inc = [&counter] { counter++; };

int even_count = 0;
for_each(v.begin(), v.end(), [&even_count] (int n) {
    cout << n;
    if (n % 2 == 0)
        ++even_count;
});

cout << "There are " << even_count
     << " even numbers in the vector." << endl;
```

___
<!-- .slide: style="font-size: 0.95em" -->

## Generyczne lambdy (C++14)

W C++11 parametry wyrażenia lambda muszą być zadeklarowane przy użyciu określonego typu.

C++14 pozwala zadeklarować parametr jako `auto`.

Pozwala to kompilatorowi wydedukować typ parametru lambda w taki sam sposób, w jaki wyprowadza się parametry szablonów. W rezultacie kompilator generuje kod odpowiadający klasie zamknięcia podanej poniżej:

```c++
auto lambda = [](auto x, auto y) { return x + y; }

struct UnnamedClosureClass {// kod wygenerowany przez kompilator dla linii powyżej
    template <typename T1, typename T2>
    auto operator()(T1 x, T2 y) const {
        return x + y;
    }
};
auto lambda = UnnamedClosureClass();
```

___

## Wyrażenia przechwytywane w lambdzie (C++14)

Funkcje lambda w C++11 przechwytują zmienne zadeklarowane w ich zewnętrznych zakresach przez kopiowanie wartości lub referencję. Oznacza to, że elementy składowe wartości lambda nie mogą być typami tylko do przenoszenia.

C++14 umożliwia inicjalizację przechwyconych zmiennych za pomocą dowolnych wyrażeń. Umożliwia to zarówno przechwytywanie przez przenoszenie wartości, jak i deklarowanie dowolnych zmiennych lambdy, bez posiadania odpowiednio nazwanej zmiennej w zakresie zewnętrznym.

```c++
auto lambda = [value = 1] { return value; };

std::unique_ptr<int> ptr(new int(10));
auto anotherLambda = [value = std::move(ptr)] { return *value; };
```

___

## Zadanie

Zmień funkcje w `main.cpp` na lambdy (`sortByArea`, `perimeterBiggerThan20`, `areaLessThan10`)

Zmień lambdę `areaLessThan10` na lambdę `areaLessThanX`, które wymaga `x = 10` na liście przechwytywania. Jaki jest problem?

Posłuż się `std::function`, by rozwiązać problem.
