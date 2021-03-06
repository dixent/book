## Чтение файла

Теперь добавим возможность чтения файла, указанного как аргумент командной строки `filename`. Во-первых, нам нужен пример файла для тестирования: лучший тип файла для проверки работы `minigrep` это файл с небольшим количеством текста в несколько строк с несколькими повторяющимися словами. В листинге 12-3 представлено стихотворение Эмили Дикинсон, которое будет хорошо работать!

Создайте файл с именем *poem.txt* в корне вашего проекта и введите стихотворение «Я никто! Кто ты?"

<span class="filename">Файл: poem.txt</span>

```text
I'm nobody! Who are you?
Are you nobody, too?
Then there's a pair of us - don't tell!
They'd banish us, you know.

How dreary to be somebody!
How public, like a frog
To tell your name the livelong day
To an admiring bog!
```

<span class="caption">Листинг 12-3: Стихотворение Эмили Дикинсон “I’m nobody! Who are you?”</span>

Текст на месте, отредактируйте *src/main.rs* и добавьте код для чтения файла, как показано в листинге 12-4.

<span class="filename">Файл: src/main.rs</span>

```rust,should_panic
use std::env;
use std::fs;

fn main() {
#     let args: Vec<String> = env::args().collect();
#
#     let query = &args[1];
#     let filename = &args[2];
#
#     println!("Searching for {}", query);
    // --snip--
    println!("In file {}", filename);

    let contents = fs::read_to_string(filename)
        .expect("Something went wrong reading the file");

    println!("With text:\n{}", contents);
}
```

<span class="caption">Листинг 12-4: Чтение содержимого файла указанного во втором аргументе</span>

Во-первых, мы добавляем ещё одно объявление `use` чтобы подключить соответствующую часть стандартной библиотеки: нам нужен `std::fs` для обработки файлов.

В `main` мы добавили новый оператор: функция `fs::read_to_string` принимает `filename`, открывает этот файл и возвращает содержимое файла как `Result<String>`.

После этого выражения мы снова добавили временный вывод `println!` для печати значения `contents` после чтения файла, поэтому мы можем проверить, что программа работает.

Давайте запустим этот код с любой строкой в качестве первого аргумента командной строки (потому что мы ещё не реализовали поисковую часть) и файл *poem.txt* как второй аргумент:

```text
$ cargo run the poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep the poem.txt`
Searching for the
In file poem.txt
With text:
I'm nobody! Who are you?
Are you nobody, too?
Then there's a pair of us — don't tell!
They'd banish us, you know.

How dreary to be somebody!
How public, like a frog
To tell your name the livelong day
To an admiring bog!
```

Отлично! Этот код прочитал и затем печатал содержимое файла. Хотя наша программа решает поставленную задачу, она не лишена недостатков. Прежде всего, функция `main` решает множество задач. Такую функцию неудобно тестировать. Далее, не отслеживаются возможные ошибки ввода данных. Пока наша программа небольшая, то данными недочётами можно пренебречь. При увеличении размеров программы, такую программу будет всё сложнее и сложнее поддерживать. Хорошей практикой программирования является ранний рефакторинг кода (refactoring) по мере усложнения. Поэтому, далее мы улучшим наш код с помощью улучшения его структуры.
