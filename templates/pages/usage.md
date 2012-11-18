Użycie po stronie klienta
=================

Interpretacja kodu w przeglądarce jest najprostszym sposobem na rozpoczęcie pracy z LESS i rozwijanie kodu w środowisku deweloperskim. W środowisku produkcyjnym, gdzie liczy się wydajność, zalecamy jednak wcześniejszą kompilację.

Załącz swój arkusz stylów `.less` z właściwością `rel` ustawioną na "`stylesheet/less`":

    <link rel="stylesheet/less" type="text/css" href="styles.less">

Następnie pobierz `less.js` z linku na początku strony i załącz go w elemencie `<head>` twojej strony, jak poniżej:

    <script src="less.js" type="text/javascript"></script>

Upewnij się, że załączasz arkusze stylów *przed* skryptem.

Tryb obserwatora (watch mode)
----------

*Tryb obserwatora* sprawia, że twoje style są odświeżane automatycznie kiedy tylko są zmienione.

Aby przejść do tego trybu, dodaj '`#!watch`' do adresu URL w przeglądarce, a następnie odśwież stronę. Inny sposób to wywołanie `less.watch()` z konsoli przeglądarki.

Użycie po stronie serwera
=================

Instalacja
------------

Najprostszym sposobem instalacji LESS na serwerze, jest skorzystanie z [npm](http://github.com/isaacs/npm), menadżera pakietów node:

    $ npm install -g less
	
Użycie z poziomu linii komend
------------------

Kiedy już zainstalujesz LESS, możesz uruchomić kompilator z linii komend:

    $ lessc styles.less

Tak uruchomiony kompilator zwróci skompilowany CSS do `stdout`, możesz jednak przekierować wyjście do dowolnego pliku:

    $ lessc styles.less > styles.css

Aby otrzymać zminimalizowany CSS, uruchom kompilator z opcją `-x`. Jeśli oczekujesz wyższego stopnia kompresji, możesz też skorzystać z [YUI CSS Compressor](http://developer.yahoo.com/yui/compressor/css.html) korzystając z opcji `--yui-compress`.

Aby zobaczyć wszystkie dostępne opcje kompilatora, uruchom lessc bez parametrów.

Użycie z poziomu kodu
-------------

Możesz wywołać kompilator w środowisku node, jak poniżej:

    var less = require('less');

    less.render('.class { width: (1 + 1) }', function (e, css) {
        console.log(css);
    });

co zwróci:

    .class {
      width: 2;
    }

możesz również wywołać parser i kompilator ręcznie:

    var parser = new(less.Parser);

    parser.parse('.class { width: (1 + 1) }', function (err, tree) {
        if (err) { return console.error(err) }
        console.log(tree.toCSS());
    });

Konfiguracja
-------------

Możesz przekazywać do kompilatora opcje:

    var parser = new(less.Parser)({
        paths: ['.', './lib'], // Określa ścieżki przeszukiwania dyrektyw @import
        filename: 'style.less' // Określa ścieżkę pliku, w celu lepszego raportowania błędów
    });

    parser.parse('.class { width: (1 + 1) }', function (e, tree) {
        tree.toCSS({ compress: true }); // Kompresuje wyjściowy CSS
    });



Inne narzędzia
=================

Poza oficjalnym kompilatorem, dostępne są też inne narzędzia, opisane na github wiki:

<a href="https://github.com/cloudhead/less.js/wiki/Command-Line-use-of-LESS">Narzędzia konsolowe</a>

<a href="https://github.com/cloudhead/less.js/wiki/GUI-compilers-that-use-LESS.js">Narzędzia z GUI</a>
