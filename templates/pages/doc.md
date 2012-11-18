LESS będąc rozszerzeniem CSS jest z nim nie tylko wstecznie kompatybilny, ale też korzysta z jego istniejącej składni podczas opisu nowych struktur leksykalnych. To sprawia, że nauka LESS jest prosta i jeśli będziesz miał wątpliwości, pozwala ci powrócić do CSS.

Zmienne
---------

Poniższy kod jest dość oczywisty:

    @nice-blue: #5B83AD;
    @light-blue: (@nice-blue + #111);

    #header { color: @light-blue; }

Zostanie on przekonwertowany na:

    #header { color: #6c94be; }

Istnieje również możliwość definiowania zmiennych ze zmienną nazwą:

    @fnord: "I am fnord.";
    @var: 'fnord';
    content: @@var;

Co zostanie skompilowane do:

    content: "I am fnord.";

Zauważ, że zmienne w LESS są tak właściwie "stałymi", ponieważ mogą być zdefiniowane tylko raz.

Domieszki (mixins)
------

W LESS istnieje możliwość załączenia kilku właściwości z jednego zbioru reguł do innego. Załóżmy, że mamy taką klasę:

    .bordered {
      border-top: dotted 1px black;
      border-bottom: solid 2px black;
    }

I chcemy użyć tych właściwości wewnątrz innego zbioru reguł. Jedyne co musimy zrobić, to dodać nazwę klasy w dowolnym innym zbiorze reguł, jak poniżej:

    #menu a {
      color: #111;
      .bordered;
    }
    .post a {
      color: red;
      .bordered;
    }

Właściwości klasy `.bordered` będą teraz należeć również do `#menu a` oraz `.post a`:

    #menu a {
      color: #111;
      border-top: dotted 1px black;
      border-bottom: solid 2px black;
    }
    .post a {
      color: red;
      border-top: dotted 1px black;
      border-bottom: solid 2px black;
    }

Dowolny zbiór reguł *klasy* czy *id* może wyć wmieszany w ten sposób.

Parametryczne domieszki (parametric mixins)
-----------------

LESS ma specjalny typ zbioru reguł, który może być wmieszany tak jak klasy, ale przyjmując parametry. Tutaj jest wzorcowy przykład:

    .border-radius (@radius) {
      border-radius: @radius;
      -moz-border-radius: @radius;
      -webkit-border-radius: @radius;
    }

A tutaj jego użycie w różnych zbiorach reguł.

    #header {
      .border-radius(4px);
    }
    .button {
      .border-radius(6px);
    }

Parametryczne domieszki mogą również posiadać domyślne wartości argumentów:

    .border-radius (@radius: 5px) {
      border-radius: @radius;
      -moz-border-radius: @radius;
      -webkit-border-radius: @radius;
    }

Teraz możemy użyć jej w ten sposób:

    #header {
      .border-radius;
    }

I `#header` będzie posiadał 5px border-radius.

Możesz też używać parametrycznych domieszek, które nie przyjmują parametrów. Mogą być przydatne jeśli chcesz, aby zbiór reguł nie został dołączony do wyjściowego CSS, ale jego własności były załączone do innych zbiorów reguł:

    .wrap () {
      text-wrap: wrap;
      white-space: pre-wrap;
      white-space: -moz-pre-wrap;
      word-wrap: break-word;
    }

    pre { .wrap }

Co zostanie skompilowane do:

    pre {
      text-wrap: wrap;
      white-space: pre-wrap;
      white-space: -moz-pre-wrap;
      word-wrap: break-word;
    }

### Zmienna `@arguments`

`@arguments` ma specjalne znaczenie wewnątrz domieszek, zawiera wszystkie przekazane argumenty, podczas wywołania domieszki. Jest to użyteczne, jeśli nie musisz operować na poszczególnych zmiennych:

    .box-shadow (@x: 0, @y: 0, @blur: 1px, @color: #000) {
      box-shadow: @arguments;
      -moz-box-shadow: @arguments;
      -webkit-box-shadow: @arguments;
    }
    .box-shadow(2px, 5px);

Co w efekcie da:

      box-shadow: 2px 5px 1px #000;
      -moz-box-shadow: 2px 5px 1px #000;
      -webkit-box-shadow: 2px 5px 1px #000;

## Dopasowania wzorców (pattern-matching) i strażnicy (guard expressions)

Czasami możesz chcieć zmienić zachowanie domieszki w oparciu o parametry, które do niej przekazujesz. Zacznijmy od czegoś prostego:

    .mixin (@s, @color) { ... }

    .class {
      .mixin(@switch, #888);
    }

Teraz załóżmy, że chcemy aby `.mixin` zachowywał się różnie w zależności od wartości `@switch`. Możemy zdefiniować `.mixin` w następująco:

    .mixin (dark, @color) {
      color: darken(@color, 10%);
    }
    .mixin (light, @color) {
      color: lighten(@color, 10%);
    }
    .mixin (@_, @color) {
      display: block;
    }

Jeśli go teraz uruchomimy:

    @switch: light;

    .class {
      .mixin(@switch, #888);
    }

Otrzymamy poniższy CSS:

    .class {
      color: #a2a2a2;
      display: block;
    }

Gdzie kolor przekazany do `.mixin` został rozjaśniony. Jeżeli wartość `@switch` wynosiłaby `dark`, w efektcie otrzymalibyśmy ciemniejszy kolor.

Oto wyjaśnienie tego, co się stało:

- pierwsza definicja domieszki nie została dopasowana, ponieważ oczekiwana była wartość `dark` jako pierwszy argument,
- druga definicja domieszki pasowała, ponieważ oczekiwaną była wartość `light`,
- trzecia definicja domieszki pasowała, ponieważ oczekiwana była dowolna wartość.

Zostały użyte tylko te definicje, które pasowały. Zmienne pasują i wiążą do dowolnej wartości. Wszystko inne niż zmienna pasuje tylko do takiej samej wartości.

Możemy również dopasowywać poprzez arność, tutaj jest przykład:

    .mixin (@a) {
      color: @a;
    }
    .mixin (@a, @b) {
      color: fade(@a, @b);
    }

Jeżeli teraz wywołamy `.mixin` z jednym argumentem, otrzymamy pierwszą definicję, ale jeśli wywołamy go z dwoma argumentami, otrzymamy drugą definicję, mianowicie `@a` przechodzący w `@b`.

### Strażnicy (guards)

Strażnicy są przydatni, kiedy zamiast prostych wartości bądź arności, chcesz dopasować *wyrażenia*. Jeżeli miałeś styczność z programowaniem funkcyjnym, prawdopodobnie spotkałeś się już z nimi. 

Starając się trzymać tak bardzo jak to możliwe deklaratywnej natury CSS, w LESS wykonywanie warunkowych działań odbywa się poprzez strzeżone domieszki (guarded mixins) zamiast bloków if/else, w stylu istniejącego w CSS3 @media.

Zacznijmy od przykładu

    .mixin (@a) when (lightness(@a) >= 50%) {
      background-color: black;
    }
    .mixin (@a) when (lightness(@a) < 50%) {
      background-color: white;
    }
    .mixin (@a) {
      color: @a;
    }

Kluczowym jest słowo **`when`**, które rozpoczyna sekwencję strażnika (w tym przykładzie jest tylko jeden strażnik. Jeśli teraz uruchomimy następujący kod:

    .class1 { .mixin(#ddd) }
    .class2 { .mixin(#555) }


Otrzymamy:

    .class1 {
      background-color: black;
      color: #ddd;
    }
    .class2 {
      background-color: white;
      color: #555;
    }

Pełna lista operatorów porównania używanych w strażnikach to: **`> >= = =< <`++. Dodatkowo słowo kluczowe `true`
jest jedyną prawdziwą wartością, sprawiającą, że te dwie domieszki są równoważne z:

    .truth (@a) when (@a) { ... }
    .truth (@a) when (@a = true) { ... }

Każda wartość inna niż `true` jest fałszem:

    .class {
      .truth(40); // Will not match any of the above definitions.
    }

Strażnicy mogą być oddzielani przecinkiem '`,`'--jeżeli którykolwiek ze strażników nie ewaluuje do wartości true, uznaje się, że pasuje:

    .mixin (@a) when (@a > 10), (@a < -10) { ... }

Zauważ, że możesz także porównywać argumenty między sobą, lub z innymi wartościami:

    @media: mobile;

    .mixin (@a) when (@media = mobile) { ... }
    .mixin (@a) when (@media = desktop) { ... }

    .max (@a, @b) when (@a > @b) { width: @a }
    .max (@a, @b) when (@a < @b) { width: @b }

W końcu, jeśli chcesz dopasować domieszki bazujące na typie wartości, możesz użyć funkcji *is\**:

    .mixin (@a, @b: 0) when (isnumber(@b)) { ... }
    .mixin (@a, @b: black) when (iscolor(@b)) { ... }

Oto podstawowe funkcje sprawdzające typy:

- `iscolor`
- `isnumber`
- `isstring`
- `iskeyword`
- `isurl`

Jeżeli chcesz sprawdzić, czy wartość poza tym, że jest numeryczna, ma ponadto konkretną jednostkę, możesz użyć jednej z tych funkcji:

- `ispixel`
- `ispercentage`
- `isem`

W końcu możesz też użyć słowa kluczowego **`and`**, aby określić dodatkowe warunki wewnątrz strażnika:

    .mixin (@a) when (isnumber(@a)) and (@a > 0) { ... }

Oraz słowa kluczowego **`not`** do negowania warunków:

    .mixin (@b) when not (@b > 0) { ... }

Zagnieżdżone reguły
------------

LESS daje ci możliwość używania zagnieżdżeń zamiast kaskadowości, lub też w połączeniu z nią.
Załóżmy, że mamy poniższy CSS:

    #header { color: black; }
    #header .navigation {
      font-size: 12px;
    }
    #header .logo {
      width: 300px;
    }
    #header .logo:hover {
      text-decoration: none;
    }

W LESS możemy również zapisać to w ten sposób:

    #header {
      color: black;

      .navigation {
        font-size: 12px;
      }
      .logo {
        width: 300px;
        &:hover { text-decoration: none }
      }
    }

Lub w ten sposób:

    #header        { color: black;
      .navigation  { font-size: 12px }
      .logo        { width: 300px;
        &:hover    { text-decoration: none }
      }
    }

W efekcie kod jest bardziej zwięzły i lepiej odzwierciedla strukturę `drzewa DOM`.

Zauważ, że kombinator `&`--jest używany, kiedy chcesz aby zagnieżdżony selektor został dołączony do selektora jego rodzica, zamiast zachowywać się jak potomek. Jest to szczególnie przydatne w użyciu z pseudo-klasami takimi jak `:hover` czy `:focus`.

Na przykład:

    .bordered {
      &.float {
        float: left;
      }
      .top {
        margin: 5px;
      }
    }

Zwróci:

    .bordered.float {
      float: left;
    }
    .bordered .top {
      margin: 5px;
    }


Zaawansowane zastosowanie &
-------------------

Symbol & może być używany w selektorach do odwracania porządku zagnieżdżania i do kombinowania klas.

Na przykład:

    .child, .sibling {
	    .parent & {
		    color: black;
		}
		& + & {
		    color: red;
		}
	}
	
Zwróci:

    .parent .child,
    .parent .sibling {
	    color: black;
	}
	.child + .child,
    .child + .sibling,
	.sibling + .child,
	.sibling + .sibling {
	    color: red;
	}

Możesz też używać & w domieszkach, aby odnieść się do bloku na zewnątrz domieszki.

Operacje
----------

Każdy numer, kolor czy zmienna mogą być operandami. Operacje powinny być wykoniwane wewnątrz nawiasów. 
Poniżej kilka przykładów:

    @base: 5%;
    @filler: (@base * 2);
    @other: (@base + @filler);

    color: (#888 / 4);
    background-color: (@base-color + #111);
    height: (100% / 2 + @filler);

Rezultat jest prawdopodobnie zgodny z twoją intuicją—LESS rozróżnia kolory i jednostki. Jeżeli jednostka jest użyta w operacji tak jak tutaj:

    @var: (1px + 5);

LESS użyje tej jednostki w wyjściowym CSS—w tym wypadku `6px`.

Dodatkowe nawiasy również są dozwolone wewnątrz operacji:

    width: ((@var + 5) * 2);

Funkcje kolorów
---------------

LESS oferuje szereg funkcji przekształcających kolory. Kolory są najpierw konwertowane do przestrzeni barw *HSL*, a następnie zmieniane na poziomie kanałów:

    lighten(@color, 10%);    // zwraca kolor 10% jaśniejszy od @color
    darken(@color, 10%);     // zwraca kolor 10% ciemniejszy od @color

    saturate(@color, 10%);   // zwraca kolor o 10% bardziej nasycony niż @color
    desaturate(@color, 10%); // zwraca kolor o 10% mniej nasycony niż @color

    fadein(@color, 10%);     
	    // zwraca kolor o 10% mniej przezroczysty niż @color
    fadeout(@color, 10%);    
	    // zwraca kolor o 10% bardziej przezroczysty niż @color
    fade(@color, 50%);       // zwraca kolor o przezroczystości 50%

    spin(@color, 10);        // zwraca kolor o parametrze HUE większym o 10 stopni niż @color
    spin(@color, -10);       // zwraca kolor o parametrze HUE mniejszym o 10 stopni niż @color

    mix(@color1, @color2, @stosunek);  
	    // zwraca rezultat zmieszania kolorów @color1 i @color2, domyślnie w stosunku 50%
    contrast(@color1, @darkcolor, @lightcolor); 
	    // zwróci @darkcolor jeżeli @color1 parametr jasności jest większy niż 50%
		// w przeciwnym wypadku zwróci @lightcolor

Używanie tych funkcji jest bardzo proste:

    @base: #f04615;

    .class {
      color: saturate(@base, 5%);
      background-color: spin(lighten(@base, 25%), 8);
    }

Możesz też uzyskać informacje o kolorach:

    hue(@color);        // zwraca wartość kanału `hue`
    saturation(@color); // zwraca wartość kanału `saturation`
    lightness(@color);  // zwraca wartość kanału 'lightness'
    red(@color);        // zwraca wartość kanału 'red'
    green(@color);      // zwraca wartość kanału 'green'
    blue(@color);       // zwraca wartość kanału 'blue'
    alpha(@color);      // zwraca wartość kanału 'alpha'
    luma(@color);       // zwraca wartość 'luma'

Jest to przydatne, kiedy chcesz stworzyć nowy kolor, bazując na kanale innego koloru:

    @new: hsl(hue(@old), 45%, 90%);

`@new` będzie miało wartość *hue* koloru `@old` i będzie miał swoje własne nasycenie i jasność. Kolory są zawsze zwracane jako wartości RGB, więc wywoływanie funkcji `spin` na szarym kolorze nic nie zmieni.

Funkcje matematyczne
--------------

LESS oferuje kilka przydatnych funkcji matematycznych, których możesz używać na wartościach numerycznych:

    round(1.67); // zwraca `2`
    ceil(2.4);   // zwraca `3`
    floor(2.6);  // zwraca `2`

Jeśli chcesz zmienić ułamek na procent, możesz użyć funkcji `percentage`:

    percentage(0.5); // zwraca `50%`

Przestrzenie nazw
----------

Czasem możesz mieć potrzebę pogrupowania zmiennych i domieszek, z przyczyn organizacji kodu, albo po prostu, żeby dodać enkapsulację.
Możesz zrobić to w dość intuicyjny sposób. Załóżmy, że chesz powiązać jakieś domieszki i wartości wewnątrz `#bundle`, aby następnie ich użyć:

    #bundle {
      .button () {
        display: block;
        border: 1px solid black;
        background-color: grey;
        &:hover { background-color: white }
      }
      .tab { ... }
      .citation { ... }
    }

Jeśli teraz chcesz wmieszać klasę `.button` do naszego `#header a`, możesz to zrobić tak:

    #header a {
      color: orange;
      #bundle > .button;
    }

Zasięg
-----

Zasięg w LESS jest bardzo podobny do tego w językach programowania. Zmienne i domieszki są najpierws przeszukiwane lokalnie,
a następnie, jeśli nie zostały znalezione, kompilator będzie ich szukał w bloku rodzica i tak dalej.

    @var: red;

    #page {
      @var: white;
      #header {
        color: @var; // white
      }
    }

    #footer {
      color: @var; // red
    }

Komentarze
--------

Komentarze w CSS są dostępne również w LESS:

    /* Hej, jestem komentarzem w stylu CSS */
    .class { color: black }

Ponadto poprawne są również jednolinijkowe komentarze, jednak są 'ciche' (silent) i nie będą widoczne w wyjściowym kodzie CSS:

    // Cześć, jestem cichym komentarzem i nie będzie mnie w wyjściowym CSS
    .class { color: white }

Importowanie
---------

Możesz importować pliki `.less` i wszystkie zmienne oraz domieszki będą dostępne w głównym pliku.
Rozszerzenie `.less` nie jest konieczne, więc obie wersje są poprawne:

    @import "lib.less";
    @import "lib";

Jeśli chcesz importować plik CSS i nie chcesz, żeby LESS go przetwarzał, wystarczy że użyjesz rozszeżenia `.css`:

    @import "lib.css";

Dyrektywa pozostanie nienaruszona i znajdzie się w wyjściowym pliku CSS.

Interpolacja łańcuchów (string interpolation)
--------------------

Zmienne mogą być używane wewnątrz łańcuchów w sposób podobny do Rubiego czy PHP, wewnątrz `@{nazwa}`:

    @base-url: "http://assets.fnord.com";
    background-image: url("@{base-url}/images/bg.png");

Escaping
--------

Może się zdażyć, że będziesz musiał skorzystać z wartości, która nie jest zgodna ze składnią CSS, lub składnią nierozpoznawaną przez LESS.

Aby wypisać taką wartość, umieszczamy ją wewnątrz łańcucha poprzedzonego przez `~`:

    .class {
      filter: ~"ms:alwaysHasItsOwnSyntax.For.Stuff()";
    }

Po angielsku wartość taką nazywa się "escaped value", w rezultacie otrzymamy:

    .class {
      filter: ms:alwaysHasItsOwnSyntax.For.Stuff();
    }
	
Interpolacja selektorów (selector interpolation)
----------------------

Jeśli chcesz używać zmiennych less wewnątrz selektorów, możesz to zrobić poprzez odniesienie się do zmiennej za pomocą `@{selektor}`, tak samo jak przy interpolacji łańcuchów znaków. Na przykład:

    @name: blocked;
	.@{name} {
	    color: black;
	}
	
zwróci:

    .blocked {
	    color: black;
	}
	
Uwaga: do LESS w wersji 1.3.1 wyrażenie `(~"@{nazwa}")` było dozwolone. Będzie ono jednak usunięte w niedalekiej przyszłości.

wykonywanie kodu JavaScript
---------------------

Wyrażenia JavaScript mogą być wywoływane tak jak wartości wewnątrz plików .less. Zalecamy ostrożność podczas korzystania z tej możliwości,
ponieważ taki kod może nie być przenośny i trudniejszy w rozwijaniu. Jeżeli to możliwe, spróbuj pomyśleć o funkcji, która może być użyta
w celu uzyskania tego efektu i zapytaj o nią w serwisie GitHub. Mamy w planach zezwolenie na rozszerzanie domyślnego zbioru funkcji.
Jednak jeśli ciągle chcesz używać JavaScriptu wewnątrz .less, możesz to zrobić otaczając wyrażenie grawisami (back-ticks):

    @var: `"cześć".toUpperCase() + '!'`;

Zostanie zamienione na:

    @var: "CZEŚĆ!";

Możesz też używać interpolacji i "escapingu" tak jak w łańcuchach znaków:

    @str: "cześć";
    @var: ~`"@{str}".toUpperCase() + '!'`;

Zostanie zamienione na:

    @var: CZEŚĆ!;

Możesz również odnieść się do zmiennych z środowiska JavaScript:

    @height: `document.body.clientHeight`;

Jeśli chcesz użyć łańcucha znaków z JavaScript jako koloru w zapisie szesnastkowym, możesz skorzystać z funkcji `color`:

    @color: color(`window.colors.baseColor`);
    @darkcolor: darken(@color, 10%);


