# Рендиране на документация за вашето конзолно приложение

Документации за конзолните приложения предимно се състои
в секция на командата `--help`
и страницата на `man`.

И двете могат да се генерират автоматично
когато използвате [`clap`](https://crates.io/crates/clap), чрез
библиотеката [`clap_mangen`](https://crates.io/crates/clap_mangen).

```rust,ignore
#[derive(Parser)]
pub struct Head {
    /// file to load
    pub file: PathBuf,
    /// how many lines to print
    #[arg(short = "n", default_value = "5")]
    pub count: usize,
}
```

Второ, трябва да използвате `build.rs`
за да генерирате ръчен файл по време на компилиране
от дефиницията на вашето приложение
в кода.

Трябва да имате предвид няколко неща
(като например как искате да опаковате своя изпълняем файл)
но засега
ние просто поставяме файла `man`
до нашата папка `src`.

```rust,ignore
use clap::CommandFactory;

#[path="src/cli.rs"]
mod cli;

fn main() -> std::io::Result<()> {
    let out_dir = std::path::PathBuf::from(std::env::var_os("OUT_DIR").ok_or_else(|| std::io::ErrorKind::NotFound)?);
    let cmd = cli::Head::command();

    let man = clap_mangen::Man::new(cmd);
    let mut buffer: Vec<u8> = Default::default();
    man.render(&mut buffer)?;

    std::fs::write(out_dir.join("head.1"), buffer)?;

    Ok(())
}
```

Когато сега компилирате вашето приложение
ще има файл `head.1`
в директорията на вашия проект.

Ако отворите това в `man`.
ще можете да се насладите на вашата безплатна документация.
