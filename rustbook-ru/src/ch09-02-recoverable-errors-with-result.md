## Исправимые ошибки с `Result`

Множество ошибок не являются настолько критичными, чтобы полностью останавливать выполнение программы. Весьма часто необходима просто правильная их обработка. К примеру, при попытке открытия файла может произойти ошибка из-за отсутствия файла. Решением может быть создание нового файла вместо прерывания процесса.

Напомним из ["Обработка потенциального сбоя с помощью типа `Result` "]<comment> главы 2,  перечисление <code data-md-type="codespan">Result</code> определено следующим образом как имеющее два варианта <code data-md-type="codespan">Ok</code> и <code data-md-type="codespan">Err</code> :</comment>

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

Типы `T` и `E` являются параметрами обобщённого типа: мы обсудим обобщённые типы более подробно в главе 10. Все что вам нужно знать прямо сейчас, это то, что `T` представляет тип значения, которое будет возвращено в случае успеха внутри варианта `Ok`, а `E` представляет тип ошибки, которая будет возвращена при сбое внутри варианта `Err`. Так как тип `Result` имеет эти обобщённые параметры типа, мы можем использовать `Result` и функции, которые определила стандартная библиотека во многих различных ситуациях, когда могут отличаться успешное значение и значение ошибки, которое мы хотим вернуть.

Давайте вызовем функцию, которая возвращает значение типа `Result`, потому что такая функция может потерпеть неудачу. В листинге 9-3 мы пытаемся открыть файл.

<span class="filename">Файл: src/main.rs</span>

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");
}
```

<span class="caption">Листинг 9-3: Открытие файла</span>

Откуда мы знаем, что `File::open` возвращает `Result` ? Мы могли бы посмотреть в [документацию стандартной библиотеки по API ](../std/index.html)<comment> или мы могли бы спросить компилятор! Если мы назначим аннотацию типа у переменной <code data-md-type="codespan">f</code>, про которую мы знаем, что она <em data-md-type="emphasis">не</em> возвращает тип функции, а затем попытаемся скомпилировать код, компилятор скажет нам, что типы не совпадают. Сообщение об ошибке подскажет нам, каким типом <code data-md-type="codespan">f</code> <em data-md-type="emphasis">является</em> . Давай попробуем! Мы знаем, что возвращаемый тип `File::open` не относится к типу <code data-md-type="codespan">u32</code> , поэтому давайте изменим выражение <code data-md-type="codespan">let f</code> на следующее:</comment>

```rust,ignore,does_not_compile
let f: u32 = File::open("hello.txt");
```

Попытка компиляции выводит сообщение:

```text
error[E0308]: mismatched types
 --> src/main.rs:4:18
  |
4 |     let f: u32 = File::open("hello.txt");
  |                  ^^^^^^^^^^^^^^^^^^^^^^^ expected u32, found enum
`std::result::Result`
  |
  = note: expected type `u32`
  = note:    found type `std::result::Result<std::fs::File, std::io::Error>`
```

Ошибка говорит нам о том, что возвращаемым типом функции `File::open` является `Result<T, E>` . Общий параметр `T` здесь был заполнен типом успешного выполнения, значение является дескриптором файла `std::fs::File` . Тип `E` используемый в значении ошибки является `std::io::Error` .

Этот возвращаемый тип означает, что вызов `File::open` может завершиться успешно и вернуть дескриптор файла, который можно читать или писать. Вызов функции также может завершиться ошибкой: например, файл может не существовать или у нас может не быть прав на доступ к файлу. Функция `File::open` должна иметь способ сообщить нам, был ли он успешен или вызов  потерпел неудачу, и одновременно возвращает либо дескриптор файла либо информацию об ошибке. Эта информация - именно то, что возвращает перечисление `Result` .

Когда вызов `File::open` успешен, значение в переменной `f` будет экземпляром `Ok` внутри которого содержится дескриптор файла. Если вызов не успешный, значением переменной `f` будет экземпляр `Err` который содержит больше информации про то какая ошибка произошла.

Необходимо дописать в код листинга 9-3 выполнение разных действий в зависимости от значения, которое вернёт вызов `File::open`. Листинг 9-4 показывает один из способов обработки `Result` пользуясь базовыми инструментами языка, таким как выражение `match` рассмотренное в Главе 6.

<span class="filename">Файл: src/main.rs</span>

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("There was a problem opening the file: {:?}", error)
        },
    };
}
```

<span class="caption">Листинг 9-4: Использование выражения <code>match</code> для обработки <code>Result</code></span>

Обратите внимание, что перечисление `Result`, также как перечисление `Option` и их варианты входят в  автоматическое подключение прелюдии в область видимости, поэтому не нужно подключать варианты типа `Result::` перед использование `Ok` и `Err` в рукавах выражения `match`.

Здесь мы говорим Rust, что когда имеется результат типа `Ok` , то вернуть значение внутреннего `file` из варианта `Ok` , и затем мы присваиваем значение дескриптора файла переменной `f` . После `match` мы можем использовать дескриптор файла для чтения или записи.

Другая ветвь `match` обрабатывает случай, где мы получаем значение `Err` вызова `File::open`. В этом примере мы используем вызов макроса `panic!`, если в нашей текущей директории нет файла с именем *hello.txt*, где будет выполнен этот код. Мы увидим следующее сообщение макроса `panic!` :

```text
thread 'main' panicked at 'There was a problem opening the file: Error { repr:
Os { code: 2, message: "No such file or directory" } }', src/main.rs:8
```

Как обычно, данное сообщение точно говорит, что пошло не правильно.

### Обработка различных ошибок с помощью match

Код в листинге 9-4 будет `panic!` независимо от того, почему вызов `File::open` не удался. То что мы хотим сделать вместо этого, это предпринять разные действия для разных причин сбоя: если открытие `File::open` не удалось из-за отсутствия файла, то мы хотим создать файл и вернуть дескриптор нового файла. Если вызов `File::open` не удался по любой другой причине - например, потому что у нас не было разрешения открыть файл, то мы хотим выполнить `panic!` как делали в листинге 9-4. Посмотрите листинг 9-5, который добавляет внутреннее `match` выражение.

<span class="filename">Файл: src/main.rs</span>

<!-- ignore this test because otherwise it creates hello.txt which causes other
tests to fail lol -->

```rust,ignore
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(ref error) if error.kind() == ErrorKind::NotFound => match File::create("hello.txt") {
            Ok(fc) => fc,
            Err(e) => panic!("Tried to create file but there was a problem: {:?}", e),
        },
        Err(error) => panic!("There was a problem opening the file: {:?}", error),
    };
    print!("{:#?}",f);
}
```

<span class="caption">Листинг 9-5: Обработка различных ошибок различными способами</span>

Типом значения возвращаемого функцией `File::open` внутри `Err` варианта является `io::Error`, который представляет из себя структуру предоставляемую стандартной библиотекой. Данная структура  имеет метод  `kind`, который можно вызвать для получения значения `io::ErrorKind`. Перечисление `io::ErrorKind` предоставлено стандартной библиотекой и имеет варианты представляющие различные типы ошибок, которые могут появиться при операциях `ввода/вывода`. Вариант, который мы хотим использовать это `ErrorKind::NotFound`. Он даёт информацию, что данный файл ещё не существует. Так что мы попадаем на совпадение шаблона с переменной `f`, но также у нас есть внутренняя проверка совпадения для `error.kind()`.

Условие, которое мы проверяем на внутреннем match - это является ли возвращённое значение `error.kind()` вариантом  `NotFound` для перечисления `ErrorKind`. Если является, то пробуем создать новый файл с помощью `File::create`. Те не менее, так как `File::create` также может завершиться не успешно, то нам необходим второй рукав внутреннего `match` выражения. Когда файл не может быть создан, то печатаются разные сообщения. Второй рукав внешнего кода `match` остаётся тем же самым, так что программа "паникует" на любой другой ошибке кроме отсутствующего файла.

Достаточно про `match`! Код с `match` выражением является очень удобным, но также достаточно примитивным. В главе 13, вы узнаете про замыкания (closures) ; тип  `Result<T, E>` имеет много методов принимающих замыкание и реализованных с помощью выражения`match`. Использование данных методов сделает ваш код более лаконичным. Более опытные разработчики могут написать его как в листинге 9-5:

```rust,ignore
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt").unwrap_or_else(|error| {
        if error.kind() == ErrorKind::NotFound {
            File::create("hello.txt").unwrap_or_else(|error| {
                panic!("Problem creating the file: {:?}", error);
            })
        } else {
            panic!("Problem opening the file: {:?}", error);
        }
    });
}
```

Не смотря на то, что данный код имеет такое же поведение как в листинге 9-5, он не содержит ни одного выражения `match` и более чист при чтении. Вернёмся к этому примеру после главы 13 и рассмотрим метод `unwrap_or_else` из стандартной библиотеки. Многие их этих методов могут очистить код от больших, вложенных выражений `match` при обработке ошибок.

### Сокращённые макросы обработки ошибок `unwrap` и `expect`

Использование `match` работает достаточно хорошо, но может быть многословным и не всегда хорошо передаёт намерение. Тип `Result<T, E>` имеет много вспомогательных методов определённых в нем, чтобы выполнять различные задачи. Одним из таких методов, который называется `unwrap`, является метод, который реализован так же , как выражение `match` описанный в листинге 9-4. Если значением `Result` является вариант `Ok`, то `unwrap` вернёт значение внутри `Ok`. Если `Result` является вариантом `Err`, `unwrap` вызовет макрос `panic!`. Вот пример `unwrap` в действии:

<span class="filename">Файл: src/main.rs</span>

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
    print!("{:#?}", f);
}
```

Если мы запустим этот код при отсутствии файла *hello.txt* , то увидим сообщение об ошибке из вызова `panic!` метода `unwrap` :

```text
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: Error {
repr: Os { code: 2, message: "No such file or directory" } }',
/stable-dist-rustc/build/src/libcore/result.rs:868
```

Другой метод похожий на `unwrap` это метод `expect`, позволяющий выбрать сообщение об ошибки для макроса `panic!`. Использование `expect` вместо `unwrap` с предоставлением хорошего сообщения об ошибке, выражает ваше намерение и делает более простым отслеживание источника ошибки паники. Синтаксис метода `expect` выглядит так:

<span class="filename">Файл: src/main.rs</span>

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
    print!("{:?}", f);
}
```

Использование `expect` такое же, как и `unwrap` : возврат дескриптора файла или вызова макроса `panic!` . Сообщение об ошибке будет параметром используемого метода `expect` при вызове внутреннего `panic!`, которое мы передадим в `expect` , а не сообщение по умолчанию для `panic!`, которое использует в `unwrap` . Вот как это выглядит:

```text
thread 'main' panicked at 'Failed to open hello.txt: Error { repr: Os { code:
2, message: "No such file or directory" } }', src/libcore/result.rs:906:4
```

Так как сообщение об ошибке начинается с нашего пользовательского текста: `Failed to open hello.txt`, то потом будет проще найти из какого места в коде данное сообщение приходит. Если использовать `unwrap` во множестве мест, то придётся потратить время для выяснения какой именно вызов `unwrap` вызывает "панику", так как все вызовы  `unwrap` генерируют одинаковое сообщение.

### Распространение генерируемых ошибок

Когда вы пишете функцию, реализация которой вызывает что-то, что может завершиться ошибкой, вместо обработки ошибки в этой функции, вы можете вернуть ошибку в вызывающий код, чтобы он мог решить, что с ней делать. Это известно как *распространение* ошибки и оно даёт больший контроль вызывающему коду, где может быть есть больше информации или логики, которая диктует, как ошибка должна обрабатываться, чем место в контексте текущего кода.

Например, код программы 9-5 читает имя пользователя из файла. Если файл не существует или не может быть прочтён, то функция возвращает ошибку в код, который вызвал данную функцию:

<span class="filename">Файл: src/main.rs</span>

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```

<span class="caption">Листинг 9-5: Функция, которая возвращает ошибки в вызывающий код, используя выражение<code>match</code></span>

Данную функцию можно записать гораздо короче, но чтобы изучить обработку ошибок, мы собираемся сделать многое вручную; в конце будет показан более короткий способ. Давайте, сначала рассмотрим тип возвращаемого значения `Result<String, io::Error>`. 
Оно означает, что функция возвращает значение типа `Result<T, E>` ; где шаблонный параметр `T` был заполнен конкретным типом `String` и шаблонный параметр `E` был заполнен конкретным типом `io::Error` . Если функция будет выполнена успешно, будет возвращено `Ok`, содержащее значение типа `String` , т.е. имя пользователя прочитанное функцией из файла. Если же при чтении файла будут какие-либо проблемы, то вызывающий код получит значение `Err` с экземпляром типа `io::Error`, которое содержит больше информации о проблеме. Тип `io::Error` для возвращаемого типа выбран потому, что он является типом ошибки возвращаемой из обоих операций, вызываемых в теле функции, которые могут не завершиться успешно. Это функции `File::open` и `read_to_string`.

Тело функции начинается с вызова `File::open`. Затем обрабатывается значение `Result` возвращаемое с помощью `match` аналогично коду `match` листинга 9-4, но только вместо вызова `panic!` для случая `Err`, делается ранний возврат из данной функции и передача значения ошибки из `File::open` обратно в вызывающий код, как значение ошибки уже текущей функции. Если `File::open` будет успешен, сохраняется дескриптор файла в переменной `f` и выполнение продолжается далее.

Затем мы создаём новую `String` в переменной `s` и вызываем `read_to_string` метод для дескриптора файла в переменной `f`, чтобы прочитать содержимое файла в `s`. Метод `read_to_string` также возвращает `Result`, потому что он может потерпеть неудачу, даже если `File::open` пройдёт успешно. Таким образом, нам нужно ещё одно выражение `match`, чтобы справиться с этим `Result`: если `read_to_string` выполнится успешно, то наша функция завершиться успешно и мы вернём имя пользователя из файла, который сейчас находится в `s` завёрнутый в вариант `Ok`. Если вызов `read_to_string` не успешен, мы возвращаем значение ошибки так же, как мы вернули значение ошибки в `match`, обработавшем возвращаемое значение `File::open`. Тем не менее, нам не нужно явно писать `return`, потому что это последнее выражение в функции.

Код вызывающий данный код, будет обрабатывать получение либо значения `Ok` содержащее имя пользователя или значение `Err` содержащее `io::Error` . Мы не знаем, что будет делать вызывающий код с этими значениями. Если вызывающий код получает значение `Err` , это  может вызвать `panic!` и сбой программы, используйте имя пользователя по умолчанию, или например, поищите имя  пользователя где-то в другом месте. На самом деле у нас недостаточно информации о том, что пытается сделать вызывающий код, поэтому мы распространяем всю информацию об успехе или ошибке наверх для её обработки соответствующим образом.

Такая схема распространения ошибок настолько распространена в Rust, что Rust предоставляет оператор вопросительный знак `?` для простоты.

#### Сокращённое описание для оператора распространения ошибки `?`

Код программы 9-6 показывает реализацию функции `read_username_from_file`, функционал которой аналогичен коду программы 9-5, но имеет сокращённое описание с использованием оператора `?` :

<span class="filename">Файл: src/main.rs</span>

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

<span class="caption">Листинг 9-6: Пример функции, которая возвращает ошибку,
используя символ <code>?</code></span>

Оператор `?` помещается после того, как значение `Result` определено для работы почти таким же образом
как выражение `match` , которое мы определили для обработки `Result` значений в листинге 9-6. Если значение `Result` равно `Ok` , значение внутри `Ok` будет возвращено из этого выражения и программа продолжит выполнение. Если значение является `Err` , то `Err` будет возвращено из всей функции, как если бы мы использовали ключевое слово `return` поэтому значение ошибки передаётся в вызывающий код.

Имеется разница между тем, что делает выражение `match` листинга 9-6 и оператор `?`. Ошибочные значения при выполнении методов с оператором `?` возвращаются через функцию `from`, определённую в типаже `From` стандартной библиотеки. Данный типаж используется для конвертирования ошибок одного типа в ошибки другого типа. Когда оператор `?` вызывает функцию `from`, то полученный тип ошибки конвертируется в тип ошибки, который определён для возврата в текущей функции. Это удобно, когда функция возвращает один тип ошибки для представления всех возможных вариантов из-за которых она может не завершиться успешно, даже если части кода функции могут не выполниться по разным причинам. Так как каждый стандартный тип ошибки реализует функцию `from` определяя, как конвертировать себя в возвращаемый тип ошибки, то оператор `?` позаботится о такой конвертации автоматически.

В коде примера 9-6 в первой строке функция `File::open` возвращает значения содержимого из перечисления `Ok` в переменную `f`. Если же в при работе этой функции происходит ошибка, будет возвращён экземпляр структуры `Err`. Те же самые действия произойдут при чтении текстовых данных из файла с помощью функции `read_to_string`.

Оператор `?` устраняет много шаблонов и делает эту реализацию функции проще. Мы могли бы даже сократить этот код, цепочкой вызовов методов сразу после `?` , как показано в листинге 9-8.

<span class="filename">Файл: src/main.rs</span>

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();

    File::open("hello.txt")?.read_to_string(&mut s)?;

    Ok(s)
}
```

<span class="caption">Листинг 9-8. Цепочка вызовов методов после оператора <code>?</code></span>

Мы перенесли в начало функции создание новой переменной `s` типа `String`; эта часть не изменилась. Вместо создания переменной `f` мы добавили вызов `read_to_string` непосредственно к результату `File::open("hello.txt")?`, У нас ещё есть `?` в конце вызова `read_to_string`, и мы по-прежнему возвращаем значение `Ok` содержащее имя пользователя в `s` когда оба метода: `File::open` и `read_to_string` успешны, а не возвращают ошибки. Функциональность снова такая же, как в листинге 9-6 и листинге 9-7; это просто другой, более эргономичный способ написания.

Обсуждая разные способы записи данной функции, на листинге 9-9 показан способ сделать его ещё короче.

<span class="filename">Файл: src/main.rs</span>

```rust
use std::io;
use std::fs;

fn read_username_from_file() -> Result<String, io::Error> {
    fs::read_to_string("hello.txt")
}
```

<span class="caption">Листинг 9-9: Использование <code>fs::read_to_string</code> вместо открытия и чтения файла</span>

Чтение файла в строку довольно распространённая операция, так что Rust предоставляет удобную функцию `fs::read_to_string` открытия файла, создание новой `String`, чтения содержимого файла, размещение содержимого в `String` и его возврат. Конечно, использование функции `fs::read_to_string` не даёт возможности объяснить обработку всех ошибок, поэтому мы сначала изучили длинный способ.

#### Оператор `?` можно использовать для функций возвращающих `Result`

Оператор `?` может использоваться в функциях, которые имеют возвращаемый тип `Result`, потому что он работает так же как выражение `match`  определённое в листинге 9-6. Та часть `match` которая требует возвращаемый тип `Result` является кодом `return Err(e)`, таким образом возвращаемый тип функции `Result` может быть совместимым с этим `return`.

Посмотрим что происходит, если использовать оператор `?` в коде функции `main`, которая как вы помните имеет возвращаемый тип `()`:

```rust,ignore,does_not_compile
use std::fs::File;

fn main() {
    let f = File::open("hello.txt")?;
}
```

При компиляции этого кода, мы получим следующее сообщение об ошибке:

```text
error[E0277]: the `?` operator can only be used in a function that returns
`Result` or `Option` (or another type that implements `std::ops::Try`)
 --> src/main.rs:4:13
  |
4 |     let f = File::open("hello.txt")?;
  |             ^^^^^^^^^^^^^^^^^^^^^^^^ cannot use the `?` operator in a
  function that returns `()`
  |
  = help: the trait `std::ops::Try` is not implemented for `()`
  = note: required by `std::ops::Try::from_error`
```

Эта ошибка указывает на то, что разрешено использовать оператор `?` только в функциях, которые возвращают `Result` или `Option` или другой тип, который реализует типаж `std::ops::Try` . При написании кода в функции, которая не возвращает один из этих типов, и если вы хотите использовать `?` при вызове других функций возвращающих `Result<T, E>` , у вас есть два варианта решения этой проблемы.
Один из методов - изменить тип возвращаемого значения вашей функции на `Result<T, E>` , при условии что у вас нет ограничений, препятствующих этому. Другая техника заключается в использовании `match` или одного из методов `Result<T, E>` для обработки `Result<T, E>` в любым подходящим способом.

Функция `main` является специальной и имеются ограничение, какой должен быть возвращаемый тип. Один из допустимых типов для main это `()`, другой допустимый возвращаемый тип  `Result<T, E>`, как в примере:

```rust,ignore
error[E0308]: mismatched types
 -->
  |
3 |     let f = File::open("hello.txt")?;
  |             ^^^^^^^^^^^^^^^^^^^^^^^^^ expected (), found enum
`std::result::Result`
  |
  = note: expected type `()`
  = note:    found type `std::result::Result<_, _>`
```

Тип `Box<dyn Error>` называется типаж объектом, о котором мы поговорим в разделе [«Использование типаж объектов, которые допускают значения различных типов»]<comment> главы 17. А пока вы можете читать обозначение `Box<dyn Error>` как «любая ошибка». Использование <code data-md-type="codespan">?</code> в <code data-md-type="codespan">main</code> функции разрешена с этим  возвращаемым типом.</comment>

Теперь, когда мы обсудили детали вызова `panic!` или возвращения `Result`, давайте вернёмся к теме о том, как решить, какой из случаев подходит для какой ситуации.


["Обработка потенциального сбоя с помощью типа `Result` "]: ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-the-result-type
[«Использование типаж объектов, которые допускают значения различных типов»]: ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types