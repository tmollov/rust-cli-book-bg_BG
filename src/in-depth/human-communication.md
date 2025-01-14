# Общуване с хора

Не забравяйте да прочетете [главата за изхода към терминала][output]
в първото ръководство.
Той обхваща как да запишете изход към терминала,
докато тази глава ще говори за _какво_ да се изведе.

[output]: ../tutorial/output.html

## Когато всичко е наред

Полезно е да докладвате за напредъка на приложението
дори когато всичко е наред.
Опитайте се да бъдете информативни и кратки в тези съобщения.
Не използвайте прекалено технически термини в регистрационните файлове.
Запомнете:
приложението не се срива
така че няма причина потребителите да търсят грешки.

Най-важното,
бъдете постоянни в стила на общуване.
Използвайте същите префикси и структура на изреченията
за да направите логовете лесни за разбиране.

Опитайте се да оставите изхода на вашето приложение да разказва история
за това какво прави
и как се отразява на потребителя.
Това може да включва показване на времева линия на включените стъпки
или дори лента за напредък и индикатор за дълготрайни действия.
Потребителят не трябва в нито един момент
да има чувството, че приложението прави нещо мистериозно
и че не могат да го следват.

## Когато е трудно да се каже какво се случва

Когато комуникирате неноминално състояние, важно е да сте последователни.
Приложение със силно логване, което не следва стриктни нива на регистриране
предоставя същото количество или дори по-малко информация
отколкото приложение без логове.

Заради това,
важно е да се определи сериозността на събитията
и съобщения, които са свързани с него;
след това използвайте последователни нива на логове за тях.
По този начин потребителите могат сами да избират количеството логове
чрез флагове `--verbose`
или променливи на средата (като `RUST_LOG`).

Често използваната библиотеки за логваме `log`
[дефинира][log-levels] следните нива
(подредени по нарастваща тежест):

- trace
- debug
- info
- warning
- error

Добра идея е да мислите за _info_ като ниво за логване по подразбиране.
Използвайте го като информативен изход.
(Някои приложения, които са клонни към по-тих стил на извеждане
може да показва само предупреждения и грешки по подразбиране.)

Допълнително,
винаги е добра идея да използвате подобни префикси
и структура на изреченията в съобщенията в логовете,
което улеснява използването на инструмент като `grep` за филтрирането им.
Самото съобщение трябва да предоставя достатъчно контекст
за да е полезен във филтрирани логовете,
но и в същото време не е и _твърде_ информативен.

[log-levels]: https://docs.rs/log/0.4.4/log/enum.Level.html

### Примерни логове

```console
error: could not find `Cargo.toml` in `/home/you/project/`
```

```console
=> Downloading repository index
=> Downloading packages...
```

Следният лог изход е взет от [wasm-pack]:

```console
 [1/7] Adding WASM target...
 [2/7] Compiling to WASM...
 [3/7] Creating a pkg directory...
 [4/7] Writing a package.json...
 > [WARN]: Field `description` is missing from Cargo.toml. It is not necessary, but recommended
 > [WARN]: Field `repository` is missing from Cargo.toml. It is not necessary, but recommended
 > [WARN]: Field `license` is missing from Cargo.toml. It is not necessary, but recommended
 [5/7] Copying over your README...
 > [WARN]: origin crate has no README
 [6/7] Installing WASM-bindgen...
 > [INFO]: wasm-bindgen already installed
 [7/7] Running WASM-bindgen...
 Done in 1 second
```

## При паникьосване

Един от аспектите, който често се забравя, е това, че
вашата програма също извежда нещо, когато се срине.
В Rust, "сривовете" най-често са "паника"
(т.е. „контролиран срив“
за разлика от това „операционната система да убе процеса“).
По подразбиране,
когато настъпи паника,
"паник манипулатор" ще отпечата някаква информация на конзолата

Например,
ако създадете нов бинарен проект
с командата `cargo new --bin foo`
и заменете съдържанието на `fn main` с `panic!("Hello World")`,
получавате това, когато стартирате вашата програма:

```console
thread 'main' panicked at 'Hello, world!', src/main.rs:2:5
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

Това е полезна информация за вас, разработчика.
(Изненада: програмата се срина поради ред 2 във вашия файл `main.rs`).
Но за потребител, който дори няма достъп до изходния код,
това не е много ценно.
Всъщност най-вероятно е просто объркващо.
Ето защо е добра идея да добавите персонализиран манипулатор на паника,
което осигурява малко по-фокусиран към крайния потребител резултат.

Една библиотека, която прави точно това и се нарича [human-panic].
За да го добавите към конзолния ви проект,
импортирайте го
и извикайте макрото `setup_panic!()`
в началото на вашата `main` функция:

```rust,ignore
use human_panic::setup_panic;

fn main() {
   setup_panic!();

   panic!("Hello world")
}
```

Това вече ще покаже много приятелско съобщение,
и казва на потребителя какво може да направи:

```console
Well, this is embarrassing.

foo had a problem and crashed. To help us diagnose the problem you can send us a crash report.

We have generated a report file at "/var/folders/n3/dkk459k908lcmkzwcmq0tcv00000gn/T/report-738e1bec-5585-47a4-8158-f1f7227f0168.toml". Submit an issue or email with the subject of "foo Crash Report" and include the report as an attachment.

- Authors: Your Name <your.name@example.com>

We take privacy seriously, and do not perform any automated error collection. In order to improve the software, we rely on people to submit reports.

Thank you kindly!
```

[human-panic]: https://crates.io/crates/human-panic
[wasm-pack]: https://crates.io/crates/wasm-pack
