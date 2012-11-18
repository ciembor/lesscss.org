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

Strażnicy są przydatni, kiedy zamiast prostych wartości, czy arności, chcesz dopasować *wyrażenia*. Jeżeli miałeś styczność z programowaniem funkcyjnym, prawdopodobnie spotkałeś się już z nimi. 

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

Guards can be separated with a comma '`,`'--if any of the guards evaluates to true, it's
considered as a match:

    .mixin (@a) when (@a > 10), (@a < -10) { ... }

Note that you can also compare arguments with each other, or with non-arguments:

    @media: mobile;

    .mixin (@a) when (@media = mobile) { ... }
    .mixin (@a) when (@media = desktop) { ... }

    .max (@a, @b) when (@a > @b) { width: @a }
    .max (@a, @b) when (@a < @b) { width: @b }

Lastly, if you want to match mixins based on value type, you can use the *is\** functions:

    .mixin (@a, @b: 0) when (isnumber(@b)) { ... }
    .mixin (@a, @b: black) when (iscolor(@b)) { ... }

Here are the basic type checking functions:

- `iscolor`
- `isnumber`
- `isstring`
- `iskeyword`
- `isurl`

If you want to check if a value, in addition to being a number, is in a specific unit, you may use one of:

- `ispixel`
- `ispercentage`
- `isem`

Last but not least, you may use the **`and`** keyword to provide additional conditions inside a guard:

    .mixin (@a) when (isnumber(@a)) and (@a > 0) { ... }

And the **`not`** keyword to negate conditions:

    .mixin (@b) when not (@b > 0) { ... }

Nested rules
------------

LESS gives you the ability to use *nesting* instead of, or in combination with cascading.
Lets say we have the following CSS:

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

In LESS, we can also write it this way:

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

Or this way:

    #header        { color: black;
      .navigation  { font-size: 12px }
      .logo        { width: 300px;
        &:hover    { text-decoration: none }
      }
    }

The resulting code is more concise, and mimics the structure of your `DOM tree`.

Notice the `&` combinator--it's used when you want a nested selector to be concatenated to its parent selector, instead
of acting as a descendant. This is especially important for pseudo-classes like `:hover` and `:focus`.

For example:

    .bordered {
      &.float {
        float: left;
      }
      .top {
        margin: 5px;
      }
    }

Will output

    .bordered.float {
      float: left;
    }
    .bordered .top {
      margin: 5px;
    }
	
Advanced Usage of &
-------------------

The & symbol can be used in selectors in order to reverse the ordering of the nesting and to multiply classes.

For example:

    .child, .sibling {
	    .parent & {
		    color: black;
		}
		& + & {
		    color: red;
		}
	}
	
Will output

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
	
You can also use & in mixins in order to reference nesting that is outside of your mixin.

Operations
----------

Any number, color or variable can be operated on. Operations should be performed
within parentheses. Here are a couple of examples:

    @base: 5%;
    @filler: (@base * 2);
    @other: (@base + @filler);

    color: (#888 / 4);
    background-color: (@base-color + #111);
    height: (100% / 2 + @filler);

The output is pretty much what you expect—LESS understands the difference between colors and units. If a unit is used in an operation, like in:

    @var: (1px + 5);

LESS will use that unit for the final output—`6px` in this case.

Extra parentheses are also authorized in operations:

    width: ((@var + 5) * 2);

Color functions
---------------

LESS provides a variety of functions which transform colors. Colors are first converted to
the *HSL* color-space, and then manipulated at the channel level:

    lighten(@color, 10%);    // return a color 10 percentage points *lighter* than @color
    darken(@color, 10%);     // return a color 10 percentage points *darker* than @color

    saturate(@color, 10%);   // return a color 10 percentage points *more* saturated than @color
    desaturate(@color, 10%); // return a color 10 percentage points *less* saturated than @color

    fadein(@color, 10%);     
	    // return a color 10 percentage points *less* transparent than @color
    fadeout(@color, 10%);    
	    // return a color 10 percentage points *more* transparent than @color
    fade(@color, 50%);       // return @color with 50% transparency

    spin(@color, 10);        // return a color with a 10 degree larger in hue than @color
    spin(@color, -10);       // return a color with a 10 degree smaller hue than @color

    mix(@color1, @color2, @weight);  
	    // return a mix of @color1 and @color2, default weight 50%
    contrast(@color1, @darkcolor, @lightcolor); 
	    // return @darkcolor if @color1 is >50% luma (i.e. is a light color), 
		// otherwise return @lightcolor

Using them is pretty straightforward:

    @base: #f04615;

    .class {
      color: saturate(@base, 5%);
      background-color: spin(lighten(@base, 25%), 8);
    }

You can also extract color information:

    hue(@color);        // returns the `hue` channel of @color
    saturation(@color); // returns the `saturation` channel of @color
    lightness(@color);  // returns the 'lightness' channel of @color
    red(@color);        // returns the 'red' channel of @color
    green(@color);      // returns the 'green' channel of @color
    blue(@color);       // returns the 'blue' channel of @color
    alpha(@color);      // returns the 'alpha' channel of @color
    luma(@color);       // returns the 'luma' value (perceptual brightness) of @color

This is useful if you want to create a new color based on another color's channel, for example:

    @new: hsl(hue(@old), 45%, 90%);

`@new` will have `@old`'s *hue*, and its own saturation and lightness. Colors are always returned as RGB values, so applying `spin` to a grey value will do nothing.

Math functions
--------------

LESS provides a couple of handy math functions you can use on number values:

    round(1.67); // returns `2`
    ceil(2.4);   // returns `3`
    floor(2.6);  // returns `2`

If you need to turn a value into a percentage, you can do so with the `percentage` function:

    percentage(0.5); // returns `50%`

Namespaces
----------

Sometimes, you may want to group your variables or mixins, for organizational purposes, or just to offer some encapsulation.
You can do this pretty intuitively in LESS—say you want to bundle some mixins and variables under `#bundle`, for later re-use, or for distributing:

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

Now if we want to mixin the `.button` class in our `#header a`, we can do:

    #header a {
      color: orange;
      #bundle > .button;
    }

Scope
-----

Scope in LESS is very similar to that of programming languages. Variables and mixins are first looked up locally,
and if they aren't found, the compiler will look in the parent scope, and so on.

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

Comments
--------

CSS-style comments are preserved by LESS:

    /* Hello, I'm a CSS-style comment */
    .class { color: black }

Single-line comments are also valid in LESS, but they are 'silent',
they don't show up in the compiled CSS output:

    // Hi, I'm a silent comment, I won't show up in your CSS
    .class { color: white }

Importing
---------

You can import `.less` files, and all the variables and mixins in them will be made available to the main file.
The `.less` extension is optional, so both of these are valid:

    @import "lib.less";
    @import "lib";

If you want to import a CSS file, and don't want LESS to process it, just use the `.css` extension:

    @import "lib.css";

The directive will just be left as is, and end up in the CSS output.

String interpolation
--------------------

Variables can be embeded inside strings in a similar way to ruby or PHP, with the `@{name}` construct:

    @base-url: "http://assets.fnord.com";
    background-image: url("@{base-url}/images/bg.png");

Escaping
--------

Sometimes you might need to output a CSS value which is either not valid CSS syntax,
or uses proprietary syntax which LESS doesn't recognize.

To output such value, we place it inside a string prefixed with `~`, for example:

    .class {
      filter: ~"ms:alwaysHasItsOwnSyntax.For.Stuff()";
    }

This is called an "escaped value", which will result in:

    .class {
      filter: ms:alwaysHasItsOwnSyntax.For.Stuff();
    }
	
Selector Interpolation
----------------------

If you want to use less variables inside selectors, you can do this by referencing the variable using `@{selector}` as 
in string interpolation. For example:

    @name: blocked;
	.@{name} {
	    color: black;
	}
	
will output

    .blocked {
	    color: black;
	}
	
Note: prior to less 1.3.1 a `(~"@{name}")` type of selector was supported. Support for this will be removed in the near future.

JavaScript evaluation
---------------------

JavaScript expressions can be evaluated as values inside .less files. We reccomend using caution with this feature
as the less will not be compilable by ports and it makes the less harder to mantain. If possible, try to think of a
function that can be added to achieve the same purpose and ask for it on github. We have plans to allow expanding the
default functions available. However, if you still want to use JavaScript in .less, this is done by wrapping the expression
with back-ticks:

    @var: `"hello".toUpperCase() + '!'`;

Becomes:

    @var: "HELLO!";

Note that you may also use interpolation and escaping as with strings:

    @str: "hello";
    @var: ~`"@{str}".toUpperCase() + '!'`;

Becomes:

    @var: HELLO!;

It is also possible to access the JavaScript environment:

    @height: `document.body.clientHeight`;

If you want to parse a JavaScript string as a hex color, you may use the `color` function:

    @color: color(`window.colors.baseColor`);
    @darkcolor: darken(@color, 10%);


