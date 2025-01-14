# Общуване с машини

Силата на инструментите на командния ред наистина излиза наяве
когато можете да ги комбинирате.
Това не е нова идея:
Всъщност това е изречение от [философията на Unix]:

> Очаквайте изхода на всяка програма да стане вход за друга, все още неизвестна програма.

[философията на Unix]: <https://en.wikipediaUnix> philosophy.org/wiki/Unix_philosophy

Ако нашите програми изпълнят това очакване,
нашите потребители ще бъдат доволни.
За да сте сигурни, че това работи добре,
ние трябва да осигурим не просто красив резултат за хората,
но също и версия, съобразена с това, от което се нуждаят други програми.
Нека да видим как можем да направим това.

<aside>

**Забележка:**
Не забравяйте да прочетете [главата за изход за хора и машини][output]
пo първото ръководство.
Той обхваща как да записвате изход към терминала.

[output]: ../tutorial/output.html

</aside>

## Кой чете това?

Първият въпрос, който трябва да зададете е:
Дали нашата продукция за човек пред цветен терминал,
или за друга програма?
За да отговорите на това,
можете да използвате библиоте като [is-terminal]:

[is-terminal]: https://crates.io/crates/is-terminal

```rust,ignore
use is_terminal::IsTerminal as _;

if std::io::stdout().is_terminal() {
    println!("I'm a terminal");
} else {
    println!("I'm not");
}
```

В зависимост от това кой ще прочете нашия резултат,
можем да добавим допълнителна информация.
Хората са склонни да харесват цветовете,
например,
ако стартирате `ls` в случаен Rust проект,
може да видите нещо подобно:

```console
$ ls
CODE_OF_CONDUCT.md   LICENSE-APACHE       examples
CONTRIBUTING.md      LICENSE-MIT          proptest-regressions
Cargo.lock           README.md            src
Cargo.toml           convey_derive        target
```

Тъй като този стил е създаден за хората,
в повечето конфигурации
дори ще отпечата някои от имената (като `src`) в цвят
за да покажат, че са директории.
Ако вместо това насочите това към файл,
или програма като `cat`,
`ls` ще адаптира своя изход.
Вместо да използвам колони, които отговарят на моя терминален прозорец
той ще отпечата всеки запис на отделен ред.
Освен това няма да излъчва никакви цветове.

```console
$ ls | cat
CODE_OF_CONDUCT.md
CONTRIBUTING.md
Cargo.lock
Cargo.toml
LICENSE-APACHE
LICENSE-MIT
README.md
convey_derive
examples
proptest-regressions
src
target
```

## Лесни изходни формати за машини

Исторически,
единственият тип изходен код от конзолните приложения бяха низове.
Това обикновено е добре за хората пред терминала,
които могат да четат текста
и разсъждават за значението му.
Други програми обаче обикновено нямат тази възможност:
Единственият начин те да разберат резултата от даден инструмент
като `ls`
е ако авторът на програмата е включил анализатор
за да работи с каквито и да е изходи на `ls`.

Това често означава, че
този изход бе ограничен до това, което е лесно за анализиране.
Формати като TSV (стойности, разделени с табулации),
където всеки запис е на отделен ред,
и всеки ред съдържа съдържание, разделено с табулации,
са много популярни.
Тези прости формати, базирани на редове текст
позволяват на инструменти като `grep`
да използват изхода на инструменти като `ls`.
Командата `| grep Cargo` не се интересува дали редовете ви са от `ls` или файл,
то просто ще филтрира ред по ред.

Недостатъкът на това е, че не можете да използвате
`grep` като лесно да го извикате за филтриране на всички директории, които `ls` ви подава.
За целта всеки елемент от директорията ще трябва да носи допълнителни данни.

## JSON изход за машини

Стойностите, разделени с разделители, са лесен начин
за извеждане на структурирани данни,
но изисква другата програма да знае кои полета да очаква
(и в какъв ред)
и е трудно да се извеждат съобщения от различни типове.
Например,
да кажем, че нашата програма иска да изпрати съобщение до потребителя
че в момента чака изтеглянето,
и след това ще изведе съобщение, описващо получените данни.
Това са много различни видове съобщения
и опита да ги обединим в TSV изход
ще изисква от нас да измислим начин да ги разграничим.
Същото, когато искаме да отпечатаме съобщение, което съдържа два списъка
от предмети с различна дължина.

Все още,
добра идея е да изберете формат, който лесно се анализира
в повечето програмни езици/среди.
По този начин,
през последните години много приложения придобиха възможността
за извеждане на техните данни в [JSON].
То е достатъчно прост формат, че практически всеки език има анализатори за него
но и достатъчно мощен, за да бъде полезен в много случаи.
Въпреки че е текстов формат, който може да се чете от хора,
много хора са работили върху имплементации, които са много бързи
при анализиране на JSON данни и сериализиране на данни в JSON.

[JSON]: https://www.json.org/

В описанието по-горе,
говорихме за "съобщения", написани от нашата програма.
Това е добър начин да мислите за изхода:
Вашата програма не извежда непременно само един блок от данни
но всъщност може да излъчва много различна информация
докато работи.
Един лесен начин за поддържане на този подход при извеждане на JSON
е да напишете един JSON документ на съобщение
и да поставяте всеки JSON документ на нов ред
(понякога наричан [JSON с разделени редове][jsonlines]).
Това може да направи имплементациите толкова прости, колкото използването на обикновен `println!`.

[jsonlines]: https://en.wikipedia.org/wiki/JSON_streaming#Line-delimited_JSON

Ето един прост пример,
използвайки макро `json!` от библиотеката [serde_json]
за бързо записване на валиден JSON във вашия изходен код на Rust:

[serde_json]: https://crates.io/crates/serde_json

```rust,ignore
{{#include machine-communication.rs}}
```

Ето и изхода:

```console
$ cargo run -q
Hello world
$ cargo run -q -- --json
{"content":"Hello world","type":"message"}
```

(Стартирането на `cargo` с `-q` потиска обичайния му изход.
Аргументите след „--“ се предават на нашата програма.)

### Практически пример: ripgrep

_[ripgrep]_ е заместител на _grep_ или _ag_, написан на Rust.
По подразбиране ще генерира изход по следния начин:

[ripgrep]: https://github.com/BurntSushi/ripgrep

```console
$ rg default
src/lib.rs
37:    Output::default()

src/components/span.rs
6:    Span::default()
```

Но ако подадем `--json` то ще принтира:

```console
$ rg default --json
{"type":"begin","data":{"path":{"text":"src/lib.rs"}}}
{"type":"match","data":{"path":{"text":"src/lib.rs"},"lines":{"text":"    Output::default()\n"},"line_number":37,"absolute_offset":761,"submatches":[{"match":{"text":"default"},"start":12,"end":19}]}}
{"type":"end","data":{"path":{"text":"src/lib.rs"},"binary_offset":null,"stats":{"elapsed":{"secs":0,"nanos":137622,"human":"0.000138s"},"searches":1,"searches_with_match":1,"bytes_searched":6064,"bytes_printed":256,"matched_lines":1,"matches":1}}}
{"type":"begin","data":{"path":{"text":"src/components/span.rs"}}}
{"type":"match","data":{"path":{"text":"src/components/span.rs"},"lines":{"text":"    Span::default()\n"},"line_number":6,"absolute_offset":117,"submatches":[{"match":{"text":"default"},"start":10,"end":17}]}}
{"type":"end","data":{"path":{"text":"src/components/span.rs"},"binary_offset":null,"stats":{"elapsed":{"secs":0,"nanos":22025,"human":"0.000022s"},"searches":1,"searches_with_match":1,"bytes_searched":5221,"bytes_printed":277,"matched_lines":1,"matches":1}}}
{"data":{"elapsed_total":{"human":"0.006995s","nanos":6994920,"secs":0},"stats":{"bytes_printed":533,"bytes_searched":11285,"elapsed":{"human":"0.000160s","nanos":159647,"secs":0},"matched_lines":2,"matches":2,"searches":2,"searches_with_match":2}},"type":"summary"}
```

Както виждате,
всеки JSON документ е обект (шаблон) съдържащо поле `type`.
Това ще ни позволи да напишем прост интерфейс за `rg`
който чете тези документи, когато влязат и показват съвпаденията
(както и файловете, в които се намират)
дори докато _ripgrep_ все още търси.

<aside>

**Забелжка:**
Ето как Visual Studio Code използва _ripgrep_ за своето търсене в кода.

</aside>

## Как да се справим с подадената информация

Да приемем, че имаме програма, която чете броя на думите във файл:

``` rust,ignore
{{#include machine-communication-wc.rs}}
```

Той взема пътя до файл, чете го ред по ред и брои броя
думи, разделени с интервал.

Когато го стартирате, той извежда общия брой думи във файла:

``` console
$ cargo run README.md
Words in README.md: 47
```

Но какво ще стане, ако искаме да преброим броя на думите, въведени в програмата?
Програмите на Rust могат да четат данни, предадени чрез stdin [със структурата Stdin](https://doc.rust-lang.org/std/io/struct.Stdin.html), която можете да получите чрез [функцията stdin](https://doc.rust-lang.org/std/io/fn.stdin.html)
от стандартната библиотека. Подобно на четенето на редовете от файл, той може да чете
и редовете от stdin.

Ето една програма, която брои думите на това, което е въведено чрез stdin

``` rust,ignore
{{#include machine-communication-stdin.rs}}
```

Ако стартирате тази програма с текст, въведен по канал, с `-`, представляващ намерението
за четене от `stdin`, тя ще изведе броя на думите:

``` console
$ echo "hi there friend" | cargo run -- -
Words from stdin: 3
```

Изисква се това че stdin не е интерактивно, защото очакваме вход, което
предаден до програмата, а не текст, въведен по време на изпълнение. Ако stdin е
a `tty`, извежда помощните документи, така че е ясно защо не работи.
