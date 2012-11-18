Zmienne
---------

Zmienne pozwalają na zdefiniowanie wartości w jednym miejscu, a następnie na ich ponowne wykorzystanie w obrębie arkusza stylów, co pozwala na dokonywanie globalnych zmian poprzez zmianę jednej linijki kodu.

<table class="code-example" cellpadding="0">
  <tr><td>
  <pre class="less-example">
  <code>// LESS

@color: #4D926F;

#header {
  color: @color;
}
h2 {
  color: @color;
}</code></pre>
  </td><td>
  <pre class="css-output"><code>/* Skompilowany CSS */

#header {
  color: #4D926F;
}
h2 {
  color: #4D926F;
}</code></pre></td>
  </tr>
</table>

Domieszki (Mixins)
------

Dziedziczenie przez wmieszanie pozwalaja na zawarcie wszystkich właściwości jednej klasy do innej klasy, przez załączenie nazwy klasy jako jednej z właściwości. Działa podobnie do zmiennych, przy czym dotyczy całych klas. Domieszki mogą się ponadto zachowywać jak funkcje i pobierać argumenty, jak na poniższym przykładzie.

<table class="code-example" cellpadding="0">
  <tr><td>
  <pre class="less-example"><code>// LESS

.rounded-corners (@radius: 5px) {
  -webkit-border-radius: @radius;
  -moz-border-radius: @radius;
  -ms-border-radius: @radius;
  -o-border-radius: @radius;
  border-radius: @radius;
}

#header {
  .rounded-corners;
}
#footer {
  .rounded-corners(10px);
}</code></pre></td>

<td>
  <pre class="css-output"><code>/* Skompilowany CSS */

#header {
  -webkit-border-radius: 5px;
  -moz-border-radius: 5px;
  -ms-border-radius: 5px;
  -o-border-radius: 5px;
  border-radius: 5px;
}
#footer {
  -webkit-border-radius: 10px;
  -moz-border-radius: 10px;
  -ms-border-radius: 10px;
  -o-border-radius: 10px;
  border-radius: 10px;
}</code></pre>
  </td></tr>
</table>

Zagnieżdżone reguły (nested rules)
------------

Zamiast tworzyć długie nazwy selektorów w celu określenia dziedziczenia, w less można po prostu zagnieżdżać jedne selektory wewnątrz innych. Dzięki temu dziedziczenie jest bardziej przejrzyste, a arkusze stylów krótsze.

<table class="code-example" cellpadding="0">
  <tr><td>
  <pre class="less-example">
<code>// LESS

#header {
  h1 {
    font-size: 26px;
    font-weight: bold;
  }
  p { font-size: 12px;
    a { text-decoration: none;
      &:hover { border-width: 1px }
    }
  }
}

</code></pre></td>

<td>
  <pre class="css-output"><code>/* Skompilowany CSS */

#header h1 {
  font-size: 26px;
  font-weight: bold;
}
#header p {
  font-size: 12px;
}
#header p a {
  text-decoration: none;
}
#header p a:hover {
  border-width: 1px;
}

</code></pre>
  </td></tr>
</table>

Funkcje i operacje
----------------------

Czy niektóre z elementów w twoim arkuszu stylów są proporcjonalne do innych? Operacje pozwalają na dodawanie, odejmowanie, dzielenie i mnożenie wartości właściwości oraz kolorów, dając możliwość tworzenia złożonych zależności między właściwościami. Operacje nie powinny jednak być wykonywane wewnątrz nawiasów, aby zapewnić zgodność z CSS. Funkcje rz ?????????????????????????????????????????

<table class="code-example" cellpadding="0">
  <tr><td>
  <pre class="less-example">
<code>// LESS

@the-border: 1px;
@base-color: #111;
@red:        #842210;

#header {
  color: (@base-color * 3);
  border-left: @the-border;
  border-right: (@the-border * 2);
}
#footer {
  color: (@base-color + #003300);
  border-color: desaturate(@red, 10%);
}

</code></pre></td>

<td>
  <pre class="css-output"><code>/* Skompilowany CSS */

#header {
  color: #333;
  border-left: 1px;
  border-right: 2px;
}
#footer {
  color: #114411;
  border-color: #7d2717;
}

</code></pre>
  </td></tr>
</table>

