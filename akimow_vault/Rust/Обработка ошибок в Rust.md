---
title: "Обработка ошибок в Rust"
source: "https://habr.com/ru/companies/otus/articles/579538/"
author:
  - "[[Федченко Кирилл]]"
published: 2021-10-05
created: 2025-06-18
description: "Одним из факторов, влияющих на надёжность программного обеспечения, является способ обрабатывать ошибки, возникающие в процессе выполнения. Создатели Rust не стали повторять популярные методы, а..."
tags:
  - "clippings"
---
![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/644/c7b/cae/644c7bcaef8d62722bc57e574468efba.png)

Одним из факторов, влияющих на надёжность программного обеспечения, является способ обрабатывать ошибки, возникающие в процессе выполнения. Создатели Rust не стали повторять популярные методы, а выбрали другой способ, позволяющий описывать и обрабатывать ошибки более явно. В статье мы рассмотрим реализацию данного подхода, а также полезные библиотеки, упрощающие обработку ошибок.

## Содержание

1. [Что делать с ошибкой?](https://habr.com/ru/companies/otus/articles/579538/#error_kinds)
2. [Немного о синтаксисе Rust](https://habr.com/ru/companies/otus/articles/579538/#rust_syntax)
3. [Обработка ошибок в Rust](https://habr.com/ru/companies/otus/articles/579538/#rust_errors)
4. [Полезные библиотеки](https://habr.com/ru/companies/otus/articles/579538/#libs)
5. [Заключение](https://habr.com/ru/companies/otus/articles/579538/#summary)

## Что делать с ошибкой?

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/3c1/5ca/1d6/3c15ca1d687d0152158b82b2318fe8e5.jpg)

Для начала, порассуждаем о возможных вариантах действий при возникновении ошибки в ходе выполнения программы. Вариантов у нас, в конечном счёте, всего три:

1. **Завершить работу программы**. Это самый простой вариант, не требующий больших усилий от разработчика. Он применим в случаях, когда ошибка не позволяет программе корректно выполнять свои функции. В качестве примера можно рассмотреть приложение, представляющее собой обёртку над некоторой динамической библиотекой. Скажем, графический интерфейс. Приложение поставляется с этой библиотекой и не несёт какой-либо пользы в отрыве от неё. Разумно предположить, что приложение не должно работать без этой библиотеки. Поэтому, вполне обосновано, при ошибке загрузки библиотеки, прерывать работу приложения.
2. **Обработать ошибку**. Чтобы программа могла продолжить выполнение после возникновения ошибки, требуется отреагировать на эту ошибку так, чтобы корректная часть программы могла далее выполнять свои функции, потеряв, возможно, доступ к некоторым возможностям. Рассмотрим приложение, использующее модули в виде динамических библиотек. В данном случае, отсутствие библиотеки модуля, необходимого для выполнения выбранного пользователем действия - это повод отменить выполнение действия, а не прерывать программу. Как вариант, сообщим пользователю об отсутствии требуемого модуля и предложим другие варианты работы.
3. **Пропустить ошибку на более высокий уровень**. Далеко не всегда, в момент получения ошибки, есть возможность однозначно выбрать способ её обработки. В таких случаях можно передать ответственность по обработке ошибки выше по иерархии вызовов. Например, подсистема загрузки конфигурационных файлов может использоваться сразу в нескольких других системах приложения. Поэтому не разумно обрабатывать случай отсутствия запрошенного файла внутри неё, одинаково для всех обратившихся. Более подходящий вариант - предоставить каждой клиентской системе самой решать, как действовать в случае ошибки загрузки конфигурации.

Ошибки, после которых приложение должно завершить работу называют **неустранимыми**. Остальные - **устранимыми**. Тип конкретной ошибки не зависит от самой ошибки (некорректный ввод, файл не найден,...). Он зависит от решения разработчика: стоит ли продолжать работу программы при этой ошибке, или программа больше ничего не может сделать. Нужно искать компромисс, исходя из требований к надёжности системы и имеющимися ресурсами для разработки, так как восстановление после ошибки требует от разработчика некоторых усилий. В лучшем случае, достаточно просто сообщить о ней пользователю и продолжить работу. Но бывают ситуации, когда для восстановления от ошибки требуется создать целую резервную систему.

Механизм обработки ошибок в Rust требует явно указывать, как вы классифицируете каждую ошибку. Для того чтобы разобраться, как этот механизм устроен, давайте рассмотрим некоторые особенности синтаксиса Rust, которые в нём применяются.

## Немного о синтаксисе Rust

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/dc0/dbc/2de/dc0dbc2de5fda486caf5c6fc22ba6cb8.jpg)

Механизм обработки ошибок включает себя две особенности языка Rust: перечисления с данными и трейты.

## Трейты

Трейты схожи с концепцией интерфейсов в других языках. Их можно реализовывать на типах, расширяя их функционал. Также, функции могут накладывать ограничение на трейты принимаемых аргументов. Ограничения проверяются при компиляции. Например:

```rust
fn main() {
    let int_value: i32 = 42;
    let float_value: f32 = 42.0;

    print_value(int_value);
    // Строка ниже ломает компиляцию, так как для float не реализован трейт Print
    // print_value(float_value);
}

trait Print {
    fn print(&self);
}

impl Print for i32 {
    fn print(&self) {
        println!("Printing i32: {}", self)
    }
}

fn print_value<T: Print>(value: T) {
    value.print()
}
```

[Ссылка на Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=6499f8ebc47449c5e7fb962df3c30c13)

В данном примере мы определили трейт `Print` и реализовали его для встроенного целочисленного типа `i32`. Также, мы определили функцию `print_value()`, принимающую обобщённый (generic) аргумент `value`, ограничив варианты его типа только теми, которые реализуют трейт `Print`. Поэтому в `main()` мы можем вызвать `print_value()` только с `i32` аргументом.

Более того, при определённых условиях, можно создавать трейт объекты (trait objects). Это динамический объекты, которые могут быть созданы из любого типа, реализующего данный трейт. Конкретная реализация метода трейта выбирается динамически (dynamic dispatch). Например:

```rust
trait Animal {
    fn says(&self);
}

struct Cat {}
struct Dog {}

impl Animal for Cat {
    fn says(&self) {
        println!("Meow")
    }
}

impl Animal for Dog {
    fn says(&self) {
        println!("Woof")
    }
}

fn main() {
    let cat = Cat{};
    let dog = Dog{};
    
    say_something(&cat);
    say_something(&dog);
}

fn say_something(animal: &dyn Animal) {
    animal.says()
}
```

[Ссылка на Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=db0522372634d24c989627b45e854294)

В данном коде нет необходимости делать функцию `say_something()` обобщённой, так как конкретная реализация, скрытая за трейт объектом разрешается во время выполнения программы, а не при компиляции.

Также, стоит упомянуть о том, что трейты могут наследоваться. То что трейт `Mammal` унаследован от трейта `Animal` означает, что реализовать трейт `Mammal` может только тип, реализующий `Animal`.

```rust
trait Animal {}

trait Mammal: Animal {}

struct Cat {}
struct Dog {}

impl Animal for Cat {}
impl Mammal for Cat {}

impl Mammal for Dog {}
```

[Ссылка на Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=edeecc0f8e7285604dba90d9c6b43ffc)

Данный код не компилируется, так как мы пытаемся реализовать трейт `Mammal` на типе `Dog`, не реализовав `Animal`, от которого `Mammal` унаследован.

### Перечисления с данными

Данный элемент синтаксиса позволяет привязать данные разных типов к разным вариантам перечисления. Например, вы можете принимать в качестве аргумента IP адрес, не уточняя версию:

```rust
enum IpAddr {
    IPv4(u32),
    IPv6(String),
}

fn connect(addr: IpAddr) {
    match addr {
        IpAddr::IPv4(integer_address) => {...}
        IpAddr::IPv6(string_address) => {...}
    }
}
```

Ключевое слово `match` позволяет описать действия для различных вариантов перечисления и их содержимого.

Перечисления могут быть обобщенными:

```rust
struct DateFormatA {}
struct DateFormatB {}

enum Date<DateFormat> {
    InFormat(DateFormat),
    AsOffset(u32)
}

fn in_format_a() -> Date<DateFormatA> {
    Date::InFormat(DateFormatA {})
}

fn in_format_b() -> Date<DateFormatB> {
    Date::AsOffset(42)
}

fn main() {
    let _a = in_format_a();
    let _b = in_format_b();
}
```

[Ссылка на Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=33553578ff764ad068e82e24ebd44b24)

Разобравшись с типажами и перечислениями, можно переходить к механизму обработки ошибок.

## Обработка ошибок в Rust

В Rust есть два перечисления на которых строится, практически, вся обработка ошибок: `Option` и `Result`. Рассмотрим их подробнее.

### Option

Определение:

```rust
pub enum Option<T> {
    None,
    Some(T),
}
```

Семантика его проста: либо мы имеем некоторые данные, либо они отсутствуют. Таким образом, возвращая из функции `Option` мы, тем самым, выражаем мысль, что, возможно, мы не получим ожидаемый результат.

### Result

Определение:

```rust
pub enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

В отличие от `Option`, `Result` позволяет установить не только отсутствие данных, но и причину, в связи с которой они отсутствуют.

Рассмотрим теперь, как в Rust выразить три действия при ошибке, которые мы перечислили в начале статьи:

- Завершить работу приложения.
- Обработать ошибку.
- Пропустить ошибку на более высокий уровень.

### Завершаем работу приложения

Rust требует от разработчика явно демонстрировать своё намерение прервать программу в случае ошибки. Аварийное завершение работы программы в Rust называется паникой. Вызвать её можно с помощью макроса `panic!()`, позволяющего указать сообщения об ошибке для вывода.

```rust
fn main() {
    let broken = true;
    
    if broken {
        panic!("Program is broken!")
    }
}
```

[Ссылка на Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=0fff25619656d65689d8c36743631c60)

Так как для обработки ошибок, обычно, используются `Option` и `Result`, для завершения работы программы нужно писать что-то вроде:

```rust
match opt {
    Some(value) => value,
    None => panic!("No value in option!")
};

match res {
    Ok(value) => value,
    Err(error) => panic!("Error happaned: {}!", error)
};
```

Для удобства, `Option` и `Result` содержат ассоциированную функцию `unwrap()`, позволяющую не повторять приведённый выше код. Если перечисление находится в состоянии успеха, то `unwrap()` достаёт данные из перечисления и позволяет с ними работать. В случае ошибки, `unwrap()` вызывает панику. У `unwrap()` есть аналог, позволяющий добавить произвольный текст к выводу: `expect()`.

```rust
fn main() {
    let settings = read_settings().unwrap();
    let _service = create_service(settings).expect("Can't create service");
}

fn read_settings() -> Option<String> {
    // Some settings loading code
    None // Settings is not found
}

struct Service {}

#[derive(Debug)] // Generate implementation for Debug trait, required by .expect()
enum CreateServiceError {
    BadSettings,
    InternalError,
}

fn create_service(_settings: String) -> Result<String, CreateServiceError> {
    // Some service creation code
    let bad_settings = true;
    if bad_settings {
        Err(CreateServiceError::BadSettings)
    } else {
        Err(CreateServiceError::InternalError)
    }
}
```

[Ссылка на Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=df4a8e036b424c2785db1849926d736d)

### Обрабатываем ошибку

Вызывая функцию, которая может не сработать, мы получаем в качестве результата `Option` или `Result`. Если нам известно, что делать в случае неудачи, мы должны выразить свои намерения через конструкции языка. Рассмотрим пример:

```rust
fn main() {
    let some_settings = String::from("some settings");

    let s1 = match load_settings() {
        Some(s) => s,
        None => some_settings.clone(),
    };

    let s2 = load_settings().unwrap_or_default();

    let s3 = load_settings().unwrap_or(some_settings);
    
    let s4 = load_settings().unwrap_or_else(|| {String::from("new string")});
    
    println!("s1: {}", s1);
    println!("s2: {}", s2);
    println!("s3: {}", s3);
    println!("s4: {}", s4);
}

fn load_settings() -> Option<String> {
    None
}
```

[Ссылка на Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=ae260947c3c726d1f151abe967e5e82d)

В данном примере мы используем разные способы замены строки настроек, в случае неудачи при её получении:

- s1 - явно сопоставляем `Option` с шаблоном и указываем альтернативу.
- s2 - используем функцию `unwrap_or_default()`, которая в случае отсутствия данных возвращает значение по умолчанию (пустую строку).
- s3 - используем `unwrap_or()`, возвращающую свой аргумент в случае отсутствия данных.
- s4 - используем `unwrap_or_else()`, возвращающую результат вызова переданного в неё функтора в случае отсутствия данных. Такой подход позволяет вычислять значение резервного варианта не заранее, а только в случае пустого `Option`.

Перечисление `Result` предоставляет аналогичные методы.

### Пропускаем ошибку выше

Для начала, сделаем это вручную. Для `Option`:

```rust
fn main() {
    let module = init_module().unwrap();
}

struct Module {
    settings: String,
    dll: Dll,
}

struct Dll {}

fn init_module() -> Option<Module> {
    let settings = match load_settings() {
        Some(s) => s,
        None => return None,
    };

    let dll = match load_dll() {
        Some(dll) => dll,
        None => return None,
    };

    Some(Module { settings, dll })
}

fn load_settings() -> Option<String> {
    None
}

fn load_dll() -> Option<Dll> {
    None
}
```

[Ссылка на Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=686b0c12addfa8a67581467097bcfb92)

И для `Result`:

```rust
fn main() {
    let module = init_module();
}

struct Module {
    settings: String,
    dll: Dll,
}

struct Dll {}

enum InitModuleError {
    SettingsError(LoadSettingsError),
    DllError(LoadDllError),
}

fn init_module() -> Result<Module, InitModuleError> {
    let settings = match load_settings() {
        Ok(s) => s,
        Err(e) => return Err(InitModuleError::SettingsError(e)),
    };

    let dll = match load_dll() {
        Ok(dll) => dll,
        Err(e) => return Err(InitModuleError::DllError(e)),
    };

    Ok(Module { settings, dll })
}

struct LoadSettingsError {}

fn load_settings() -> Result<String, LoadSettingsError> {
    Err(LoadSettingsError {})
}

struct LoadDllError {}

fn load_dll() -> Result<Dll, LoadDllError> {
    Err(LoadDllError {})
}
```

[Ссылка на Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=1c24516d2ac386fecf80d216a37379a5)

Как видно в примерах, такой подход требует большого количества `match` конструкций. Это усложняет код, ухудшает его читабельность и добавляет разработчику дополнительной рутинной работы. Во избежание всего этого, создатели языка ввели оператор `?`. Расположенный после `Option` или `Result`, он заменяет собой `match` конструкцию. В случае наличия значения, он возвращает его для дальнейшего использования. В случае ошибки, возвращает её из функции. Воспользуемся им в наших примерах. Для `Option` всё очевидно:

```rust
fn main() {
    let module = init_module().unwrap();
}

struct Module {
    settings: String,
    dll: Dll,
}

struct Dll {}

fn init_module() -> Option<Module> {
    let settings = load_settings()?;
    let dll = load_dll()?;
    Some(Module { settings, dll })
}

fn load_settings() -> Option<String> {
    None
}

fn load_dll() -> Option<Dll> {
    None
}
```

[Ссылка на Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=8ca917c053dcbc4d904557aa780c4de6)

Для Result всё обстоит немного сложнее. Ведь в случае, если происходит `LoadDllError`, то компилятору нужно как-то преобразовать её в `InitModuleError` для возврата из функции. Для этого оператор `?` пытается найти способ преобразования для этих ошибок. Для того, чтобы создать такой способ, в стандартной библиотеке существует трейт `From`. Воспользуемся им:

```rust
fn main() {
    let module = init_module();
}

struct Module {
    settings: String,
    dll: Dll,
}

struct Dll {}

enum InitModuleError {
    SettingsError(LoadSettingsError),
    DllError(LoadDllError),
}

impl From<LoadSettingsError> for InitModuleError {
    fn from(e: LoadSettingsError) -> InitModuleError {
        InitModuleError::SettingsError(e)
    }
}

impl From<LoadDllError> for InitModuleError {
    fn from(e: LoadDllError) -> InitModuleError {
        InitModuleError::DllError(e)
    }
}

fn init_module() -> Result<Module, InitModuleError> {
    let settings = load_settings()?;
    let dll = load_dll()?;
    Ok(Module { settings, dll })
}

struct LoadSettingsError {}

fn load_settings() -> Result<String, LoadSettingsError> {
    Err(LoadSettingsError {})
}

struct LoadDllError {}

fn load_dll() -> Result<Dll, LoadDllError> {
    Err(LoadDllError {})
}
```

[Ссылка на Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=6824a92218bbb561428a5e687908509e)

Иными словами, Rust требует явно описывать способы преобразования ошибок друг в друга при передаче их верхним уровням иерархии вызовов.

### Динамические ошибки

В случае, если нет необходимости использовать конкретный тип ошибки, а достаточно просто иметь текстовое сообщение о ней, то можно передавать ошибку в виде трейт объекта `std::error::Error`, завёрнутого в умный указатель `Box` ([подробнее](https://doc.rust-lang.org/rust-by-example/trait/dyn.html)). Трейт `Error` определён так:

```rust
trait Error: Debug + Display {...}
```

Как видно из определения, он требует реализации трейтов `Debug` и `Display`. Таким образом, Rust вводит требования для всех типов реализующих `Error`: уметь выводить отладочную и текстовую информацию о себе. Рассмотрим на примере:

```rust
use std::fmt;
use std::error::Error;

fn main() {
    init_module().unwrap();
}

struct Module {
    settings: String,
    dll: Dll,
}

struct Dll {}

fn init_module() -> Result<Module, Box<dyn Error>> {
    let settings = load_settings()?;
    let dll = load_dll()?;
    Ok(Module { settings, dll })
}

#[derive(Debug)]
struct LoadSettingsError {}

impl Error for LoadSettingsError {}

impl fmt::Display for LoadSettingsError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "load settings error")
    }
}

fn load_settings() -> Result<String, Box<dyn Error>> {
    Err(Box::new(LoadSettingsError {}))
}

#[derive(Debug)]
struct LoadDllError {}

impl Error for LoadDllError {}

impl fmt::Display for LoadDllError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "load dll error")
    }
}

fn load_dll() -> Result<Dll, Box<dyn Error>> {
    Err(Box::new(LoadDllError {}))
}
```

[Ссылка на Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=81aeab1463d86a3be652814d1c82fe63)

Как видно из примера, при использовании динамических ошибок, нет необходимости создавать промежуточные типы ошибок, объединяющие несколько типов ошибок нижнего уровня. У такого подхода есть свои недостатки. Во-первых, отсутствует возможность определить конкретный тип ошибки, произошедшей на нижнем уровне. Во-вторых, снижается производительность, так как для создания ошибки требуется аллокация в куче, а, при выводе сообщения об ошибке, используется динамическая диспетчеризация.

## Полезные библиотеки

Рассмотрим две популярные библиотеки, упрощающие обработку ошибок: **thiserror** и **anyhow**.

### thiserror

Данная библиотека предоставляет макросы, позволяющие упростить рутинные действия: описание способов конвертации ошибок через `From`, и реализация трейтов `Error` и `Display`. Рассмотрим на примере:

```rust
use thiserror::Error; // 1.0.29

fn main() {
    let module = init_module().unwrap();
}

struct Module {
    settings: String,
    dll: Dll,
}

struct Dll {}

#[derive(Debug, Error)]
enum InitModuleError {
    #[error("init module settings error")]
    SettingsError(#[from] LoadSettingsError),
    #[error("init module dll error")]
    DllError(#[from] LoadDllError),
}

fn init_module() -> Result<Module, InitModuleError> {
    let settings = load_settings()?;
    let dll = load_dll()?;
    Ok(Module { settings, dll })
}

#[derive(Debug, Error)]
#[error("load settings error")]
struct LoadSettingsError {}

fn load_settings() -> Result<String, LoadSettingsError> {
    Err(LoadSettingsError {})
}

#[derive(Debug, Error)]
#[error("load dll error")]
struct LoadDllError {}

fn load_dll() -> Result<Dll, LoadDllError> {
    Err(LoadDllError {})
}
```

[Ссылка на Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=4a53abc9a932f7275bfa66e4694222ec)

В данном примере, трейт `Error` реализуется автоматически с помощью макроса `#[derive(Error)]`. Используя макрос `#[error("text to display")]` генерируем реализацию трейта Display. Макрос `#[from]` создаёт реализацию трейта From для конвертации ошибки нижнего уровня в ошибку текущего.

Данные макросы значительно сокращают объём boilerplate кода для обработки ошибок.

### anyhow

Данную библиотеку удобно использовать, когда единственное, что интересует нас в ошибке - её текстовое описание. anyhow предоставляет структуру `Error`. В неё может быть сконвертирован любой объект, реализующий трейт `std::Error`, что значительно упрощает распространение ошибки по иерархии вызовов. Помимо этого, `anyhow::Error` позволяет добавлять текстовое описание контекста, в котором произошла ошибка. Эта библиотека сочетается с thiserror. Пример:

```rust
use thiserror::Error; // 1.0.29
use anyhow; // 1.0.43;
use anyhow::Context; // 1.0.43;

fn main() {
    let module = init_module().unwrap();
}

struct Module {
    settings: String,
    dll: Dll,
}

struct Dll {}

fn init_module() -> anyhow::Result<Module> {
    let dll = load_dll().context("module initialization")?;
    let settings = load_settings()?;
    Ok(Module { settings, dll })
}

#[derive(Debug, Error)]
#[error("load settings error")]
struct LoadSettingsError {}

fn load_settings() -> Result<String, LoadSettingsError> {
    Err(LoadSettingsError {})
}

fn load_dll() -> anyhow::Result<Dll> {
    anyhow::bail!("load dll error")
}
```

[Ссылка на Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=5e0c872a4e9a853f09bc987d37a56e75)

Макрос `anyhow::bail!()` в примере создаёт `anyhow::Error` с заданным описанием и возвращает её из функции. Псевдоним `anyhow::Result` определяется так:

```rust
type Result<T, E = Error> = Result<T, E>;
```

## Заключение

В начале статьи мы рассмотрели три возможных варианта действий, при получении ошибки: завершить работу программы, обработать ошибку и передать ошибку вверх по иерархии вызовов. Далее, разобравшись с особенностями синтаксиса, мы разобрались на примерах, как выразить наши намерения по отношению к ошибке на языке Rust. Мы увидели, что любой из вариантов поведения должен быть выражен явно. Такой подход повышает надёжность приложения, так как не позволяет разработчику случайно проигнорировать ошибку. С другой стороны, явное описание своих намерений требует дополнительных усилий. Минимизировать эти усилия позволяют библиотеки thiserror и anyhow.

Благодарю за внимание. Поменьше вам ошибок!

---

Статья написана в преддверии старта курса [Rust Developer](https://otus.pw/nisT/). Приглашаю всех желающих на [бесплатный урок](https://otus.pw/nisT/), в рамках которого на примере построения простого веб сервиса рассмотрим популярный веб-фреймворк actix-web в связке с MongoDB + Redis и другие полезные библиотеки для backend разработки.

- [Записаться на бесплатный урок](https://otus.pw/nisT/)

+36

Сайт

[otus.ru](https://otus.ru/)

Дата регистрации

Дата основания

Численность

101–200 человек

Местоположение

Россия

Представитель

[OTUS](https://habr.com/ru/users/MaxRokatansky/)

[![](https://habrastorage.org/getpro/habr/widget/6ba/2ac/0d7/6ba2ac0d76e059ad282b6cb1f9af45d7.jpg)](https://t.me/+nmxTyyOhV-ozYjUy)

Код свободы: говорим об open source

Узнать секреты облаков

Годное из блогов компаний

Вперёд в будущее

Реклама