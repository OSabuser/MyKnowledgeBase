---
title: "Pytest. Гайд для тех, кто ни разу про него не слышал"
source: "https://habr.com/ru/companies/beget/articles/948806/"
author:
  - "[[Андрей]]"
published: 2025-10-01
created: 2026-02-08
description: "Недавно на работе передо мной возникла задача максимально быстро погрузиться в автоматизированное тестирование с ранее мной не использовавшимся фреймворком pytest. Почитав порядка десяти статей на..."
tags:
  - "clippings"
---
Недавно на работе передо мной возникла задача максимально быстро погрузиться в автоматизированное тестирование с ранее мной не использовавшимся фреймворком pytest. Почитав порядка десяти статей на Хабре я понял, что в каждой из статей есть много всего интересного, а чтобы системно погрузиться — необходимо идти читать документацию. Я решил, в привычной мне манере, разобраться и систематизировать самый сок для того, чтобы быстро въехать в суть и важные тонкости положив основу для дальнейшего использования.

Всем интересующимся - добро пожаловать под кат!

![](https://habrastorage.org/r/w780/getpro/habr/upload_files/10a/174/0da/10a1740da160928a759129bc56177dd6.png)

**Дисклеймер.** Сразу же хотелось бы оставить за собой право на ошибки, а также размытые и не полные интерпретации вещей, о которых собираюсь рассказать т. к. я не являюсь профессиональным программистом и специалистом автоматизированном тестировании. Но с другой стороны, любые конструктивные замечания или исправления дадут почву для саморазвития и самокоррекции, и ваша обоснованная обратная связь будет очень ценной для меня.

## Что такое pytest?

**Pytest**  — это самый популярный фреймворк для тестирования на Python. **Pytest** появился, чтобы сделать тестирование в Python простым и приятным: меньше церемоний, больше читаемости и расширяемости. Он применяется везде — от библиотек и веб‑сервисов до ML‑проектов и инфраструктуры - и подходит как одиночным разработчикам, так и большим командам с CI/CD.

Первые массовые тесты в Python строились на unittest (xUnit‑подобный фреймворк из стандартной библиотеки). Он надежен, но в большей степени «церемониален»: классы, наследование, методы setUp/tearDown, громоздкие assert\*‑методы. Это тормозило внедрение тестов в повседневную практику: слишком много шаблонного кода ради простых проверок.

Со временем появлялись надстройки (например, nose), но им не хватило долгосрочной поддержки. **Pytest** предложил другой путь:

1. **Простота синтаксиса:** тест — это обычная функция и обычный assert.
2. **Лучшие сообщения об ошибках:** «расшифровка» выражений в assert и наглядные tracebacks.
3. **Фикстуры вместо классовой магии:** декларативная система подготовки/очистки состояния без наследования.
4. **Параметризация:** один тест — много входов/ожидаемых выходов.
5. **Плагины:** архитектура, которую можно расширять под любые сценарии (распараллеливание, ретраи, бенчмарки, отчеты и т. д.).

Иными словами, первопричина появления — снизить порог входа и стоимость владения тестовой базой, сохранив мощь для сложных проектов.

## Основные термины

Сразу приведу основные термины объясненные своим языком, поскольку они употребляются сразу же по ходу материала:

- **Assert-интроспекция** — это обычная инструкция Python вида assert <условие>\[, <сообщение>\], которая при падении показывает разбор выражения: левую/правую части, значения подвыражений, красивый дифф коллекций/строк.
- **Traceback** — это отчёт о том, по какой цепочке функций Python дошел до места где произошло исключение.
- **Фикстуры в pytest** — это функции, которые готовят ресурсы для тестов (данные, подключения, конфиги) и отдают его тесту как аргумент по имени и затем корректно убирают. Это удобный способ интегрировать в тест необходимые зависимости.
- **Mark‑функции** — это ярлыки для тестов в виде функций‑декораторов. Ими помечают функции тестов, чтобы исключать/выбирать при запуске, условно пропускать или ожидать падения, группировать и так далее
- **Хуки** — это специальный‑функции, с помощью которых можно настраивать поведение pytest без изменения самих тестов. Рассмотрим позже на примерах.
- **Моки** — это подменные объекты, которые имитируют поведение внешних зависимостей в тестах: БД, сеть, время, файл-система и т. п. С моками вы проверяете как ваш код взаимодействует с зависимостью (какие функции вызвал, с какими аргументами), не трогая реальный мир.

## В чем он лучше других?

1. Отсутствие избыточного кода, как это в unittest. Код тестов pytest состоит из коротких самодокументирующихся функций.
2. Достаточно простая подготовка окружения для работы, с явными повторно используемыми фикстурами с зависимостями.
3. Детально подсвечиваются различия, контекстов и значений assert для более полной диагностики падений.
4. Через @pytest.mark.parametrize можно использовать один и тот же тест с разными данными без дублирования.
5. Есть возможность делать параллельные запуски тестов, делать ретраи, таймауты и бенчмарки.
6. Единая, минималистичная идиома, которую легко читать и поддерживать.
7. Огромное количество разнообразных плагинов.

## Какой уровень входа?

Данный фреймворк вполне себе подходит для новичков, потому что для старта нужно знать лишь базовые функции Python и assert. Первые тесты пишутся на обычных assert — без классов и SetUp/tearDown. Необходимо понимать, плюсом к этому, что такое виртуальное окружение и установка пакетов из индекса pip.

То есть никаких особых знаний для его использования не требуется. Едем дальше.

## Как начать работу с pytest?

Первым делом необходимо установить pytest. Я в работе использую uv, т. к. он существенно быстрее простого pip. Рекомендую начать пользоваться им и вам.

Сначала установим uv:

```bash
pip install uv
uv --versionОбъяснить с
```

Создаем проект и виртуальное окружение:

```bash
mkdir pytest_tests && cd pytest_tests
uv init                           # создаст pyproject.toml и основу проекта
uv venv --python 3.12             # локальная .venv с нужной версией Python

# (можно зафиксировать нужную версию версию для проекта)
uv python pin 3.12                # создаст/обновит .python-versionОбъяснить с
```

uv автоматически находит.venv и использует её в следующих командах. Активировать окружение (не обязательно при использовании uv run) можно классическим способом:

```bash
source .venv/bin/activateОбъяснить с
```

Добавим pytest как dev-зависимость:

```bash
uv add --dev pytestОбъяснить с
```

Это добавит pytest в секцию зависимостей для разработчика и установит его в вашу.venv. По умолчанию uv синхронизирует dev-группу.

Далее настроим pytest в pyproject.toml. Добавьте секцию конфигурации (pytest читает её из tool.pytest.ini\_options) в созданный файл:

```coffeescript
[tool.pytest.ini_options]
minversion = "6.0"
testpaths = ["tests"]
python_files = test_*.py *_test.py
python_functions = test_*
addopts = [
  "-vv",                        # подробный вывод хода теста
  "--import-mode=importlib"      # меньше проблем с импортами
]Объяснить с
```

## Общая структура тестового окружения

Для того, чтобы тесты интегрировать в существующий проект необходимо соблюсти примерно вот такую структуру директорий:

```bash
project/
├─ src/ # Ваш исходный код (опционально)
├─ app/ # Или пакет приложения (Flask, etc.)
├─ tests/ # Все тесты здесь
│ ├─ test_smoke.py
│ ├─ test_api.py
│ └─ conftest.py # Общие фикстуры/хуки для tests/
└─ pytest.ini # Конфигурация pytestОбъяснить с
```

Сразу стоит сказать, что существуют правила обнаружения тестов: файлы test\_\*.py или *\_test.py, функции test\_*, классы Test\* без **init**.py.

Все очень просто. Идём дальше.

## Напишем первый тест

Переходим в директорию для тестов:

```bash
cd tests/Объяснить с
```

И сделаем первый тест test\_example.py:

```python
import pytest 

def add(a, b): 
    return a + b

def test_math():
    assert add(2, 3) == 5Объяснить с
```

Запустим тест через рекомендуемый способ, то есть через uv, чтобы гарантированно использовать среду проекта:

```bash
uv run pytest -vv tests/test_example.pyОбъяснить с
```

Или можно запустить с прямым указанием версии Python, кейс достаточно частый для проверки совместимости кода между версиями:

```bash
# или явно указать версию Python:
uv run --python 3.12 pytest -vv tests/test_example.pyОбъяснить с
```

При этом способе запуска нет необходимости активировать виртуальное окружение.venv. Помимо этого вы можете поэкспериментировать с флагами для запуска:

```bash
uv run pytest             # стандартный запуск
uv run pytest -q             # тише
uv run pytest -vv             # подробные имена кейсов
uv run pytest -k sum         # фильтр по выражению/подстроке имени теста
uv run pytest -m "not slow"     # запуск без помеченных slow
uv run pytest -x --maxfail=1     # остановиться на первом паденииОбъяснить с
```

И после получим вывод:

```bash
uv run pytest tests/*

============== test session starts ==============
...                                   
collected 1 item 
tests/test.py::test_math PASSED                                       [100%]

============== 1 passed in 0.07s ==============Объяснить с
```

Видим что, тест был найден и тест успешно пройден. Выведено соответствующее сообщение. Теперь изменим код для того чтобы код заведомо завалился:

```bash
uv run pytest tests/*

============== test session starts ==============
...
collected 1 item    

tests/test.py::test_math FAILED                                       [100%]

============== FAILURES ==============
______________ test_math _____________

    def test_math():
>       assert 2 + 3 == 6
E       assert (2 + 3) == 6

tests/test.py:2: AssertionError
============== short test summary info ==============
FAILED tests/test.py::test_math - assert (2 + 3) == 6
============== 1 failed in 0.09s ==============
Объяснить с
```

Сразу же видно, где происходит ошибка и pytest выводит соответствующее сообщение об этом. Выглядит все очень просто, немного усложним выходные условия.

## Разберемся с assert

Инструкция assert — это ключевой элемент тестового кейса. Именно он проверяет, есть ли соблюдение условий прохождения теста или нет. Проверки должны быть максимально читаемые и выглядеть просто как спецификация. При ее использовании нет необходимости выводить сообщения вручную почему упало. В трейслоге видно какие данные были переданы и где они в итоге не сошлись.

В assert работает стандартный набор рабочих выражений: ==,!=, <, >, in, is, составных выражений, списков/словарей/строк.

Но есть некоторые интересные кейс. Например в тестах вот такого вида:

```python
def test_list_diff():
    assert [1, 2, 3] == [1, 2, 4]   # покажет diff, что отличается последний элемент

def test_str_diff():
    assert "hello\nworld" == "hello\nWorld"  # построчный дифф с подсветкойОбъяснить с
```

В качестве результата будет выведен следующий лог:

```bash
collected 2 items                                                                                                             

tests/test_example.py::test_list_diff FAILED                                  [ 50%]
tests/test_example.py::test_str_diff FAILED                                   [100%]

============== FAILURES ==============
______________ test_list_diff ______________

    def test_list_diff():
>       assert [1, 2, 3] == [1, 2, 4]   # покажет diff, что отличается последний элемент
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
E       AssertionError: assert [1, 2, 3] == [1, 2, 4]
E         
E         At index 2 diff: 3 != 4
E         
E         Full diff:
E           [
E               1,
E               2,...
E         
E         ...Full output truncated (5 lines hidden), use '-vv' to show

tests/test_example.py:4: AssertionError
______________ test_str_diff ______________

    def test_str_diff():
>       assert "hello\nworld" == "hello\nWorld"  # построчный дифф с подсветкой
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
E       AssertionError: assert 'hello\nworld' == 'hello\nWorld'
E         
E           hello
E         - World
E         ? ^
E         + world
E         ? ^

tests/test_example.py:7: AssertionError
============== short test summary info ==============
FAILED tests/test_example.py::test_list_diff - AssertionError: assert [1, 2, 3] == [1, 2, 4]
FAILED tests/test_example.py::test_str_diff - AssertionError: assert 'hello\nworld' == 'hello\nWorld'
============== 2 failed in 0.09s ==============
Объяснить с
```

Будет подробно подсвечен конкретное место расхождения. Но это не всё, помимо этого можно производить проверку исключений:

```python
import pytest

def check_num(num):
    if not isinstance(num, int):
        raise ValueError(f"invalid value {num}")

def test_raises():
    with pytest.raises(ValueError, match=r"invalid value \d+"):
        check_num('123123')Объяснить с
```

Или предупреждений:

```python
def test_warns():
    with pytest.warns(UserWarning, match="deprecated"):
        warnings.warn("deprecated API", UserWarning)Объяснить с
```

Также есть возможность добавлять свои собственные сообщения в assert-ах:

```python
assert user.is_active, "Пользователь должен быть активирован перед входом"Объяснить с
```

Но имейте ввиду, что указывая сообщение, вы обычно теряете детальную «интроспекцию» условия. Если хотите именно своё описание — используйте выражение [pytest.fail](http://pytest.fail/) ("...") после явных проверок.

Вообще из хороших практик оформления теста можно взять за правило несколько моментов:

- Сравнивайте явно: assert value is None, assert not items, assert “ok” in resp.text.
- Для сложных объектов сделайте им информативный **repr** — отчёты станут гораздо понятнее.
- Один тест — одна идея. Несколько assert'ов конечно допустимы, но не перемешивайте разные сценарии в рамках одного кейса.

## Параметризация тестов

Часто получается так, что необходимо сделать одно и то же действие, но с разными входными данными. Очевидно, что раздувать тест из кучи копий функций с разными входными данными совершенно недопустимо - есть возможность задавать некий набор параметров для запуска. Рассмотрим самый простой вариант параметризации:

```python
import pytest

@pytest.mark.parametrize("a,b,expected", [
    (1, 1, 2),
    (2, 5, 7),
    (-1, 1, 0),
])
def test_add(a, b, expected):
    assert a + b == expectedОбъяснить с
```

Параметризация осуществлятся через декоратор @pytest.mark.parametrize. В качестве аргументов перечисляются имена параметров (строка с запятыми или список строк), второй аргумент - список наборов значений. По итогу получается каждый набор - это отдельный тест.

После запуска теста получается то, что тест перебирает каждый из вариантов:

```bash
uv run pytest 
============== test session starts ==============
...                                                 

tests/test_math.py::test_add[-1-1-0] PASSED                                    [ 33%]
tests/test_math.py::test_add[2-5-7] PASSED                                     [ 66%]
tests/test_math.py::test_add[1-1-2] PASSED                                     [100%]

============== 3 passed in 0.07s ==============Объяснить с
```

Помимо этого можно добавить описания кейсов вместо указания переданных данных:

```python
@pytest.mark.parametrize("a,b,expected", [
    (1, 1, 2),
    (2, 5, 7),
    (-1, 1, 0),
],
ids=["first", "second", "third"])Объяснить с
```

После запуска видим:

```bash
tests/test_math.py::test_add[third] PASSED                                     [ 33%]
tests/test_math.py::test_add[second] PASSED                                    [ 66%]
tests/test_math.py::test_add[first] PASSED                                     [100%]Объяснить с
```

В качестве имени кейса можно передать функцию. Такая функция получает **значение параметра** и должна вернуть **строку-имя кейса**. Это делает отчёт читаемым и помогает быстро понять, какой именно набор данных сломался.

```python
def idfn(v):
    if isinstance(v, int):
        return f"vid={v}"
    if isinstance(v, dict):
        return f'{v["name"]}:{v["role"]}'
    return repr(v)

@pytest.mark.parametrize("val", [
    1,
    4094,
    {"name": "alice", "role": "admin"},
], ids=idfn)

def test_example(val):
    assert val is not None
Объяснить с
```

Запуск покажет "говорящие" ID:

```python
tests/test_math.py::test_example[vid=1]PASSED                                  [ 33%]
tests/test_math.py::test_example[alice:admin]PASSED                            [ 66%]
tests/test_math.py::test_example[vid=4094]PASSED                               [100%]Объяснить с
```

Если функция вернет None - то Pytest возьмёт ID по умолчанию. И стоит в этом случае возвращать короткие строки.

## Параметризация через декартово произведение параметров

В pytest есть несколько удобных способов сделать "перебор всех значений". Для этого необходимо указать несколько параметров parametrize:

```python
@pytest.mark.parametrize("num1", [1, 2, 3])
@pytest.mark.parametrize("num2", [1, 2, 3])
def test_service(num1, num2):
    assert num1 + num2 == num1 + num2Объяснить с
```

После запуска получается следующий перебор вариантов:

```python
tests/test_math.py::test_service[3-3]PASSED                                  [ 11%]
tests/test_math.py::test_service[3-2]PASSED                                  [ 22%]
tests/test_math.py::test_service[3-1]PASSED                                  [ 33%]
tests/test_math.py::test_service[2-1]PASSED                                  [ 44%]
tests/test_math.py::test_service[1-1]PASSED                                  [ 55%]
tests/test_math.py::test_service[2-2]PASSED                                  [ 66%]
tests/test_math.py::test_service[1-3]PASSED                                  [ 77%]
tests/test_math.py::test_service[1-2]PASSED                                  [ 88%]
tests/test_math.py::test_service[2-3]PASSED                                  [100%]Объяснить с
```

Это позволяет проверить все возможные комбинации параметров и избежать дублирования огромного количества повторяющегося кода.

## Метки и фильтрация

Для создания групп тестов можно использовать специальные средства в pytest - маркеры. Они позволяют выбирать/исключать тесты при запуске, условно пропускать или ожидать падения, навешивать нужное поведение, логически группировать и запускать отдельными порциями и т.п.

Задается список маркеров в файле проекта pyproject.toml в секции \[tool.pytest.ini\_options\]. Например мы можем разделить тесты на группы:

```bash
markers = [
  "slow: долгие тесты",
  "integration: интеграционные тесты",
  "network: сетевые тесты"
]Объяснить с
```

Далее с помощью декоратора @pytest.mark можно разметить тестовые кейсы в соответствующие группы:

```python
@pytest.mark.slow
def test_long():
    ...

@pytest.mark.skipif(not has_device(), reason="нет стенда")
def test_needs_device():
    ...

@pytest.mark.xfail(reason="известный баг", strict=True)
def test_bug():
    assert 1 == 2Объяснить с
```

И можно потом запустить "медленные" тесты и без сетевых:

```bash
uv run pytest -m "slow and not network"Объяснить с
```

Чтобы вывести все маркеры можно выполнить комаду:

```bash
uv run pytest --markersОбъяснить с
```

Помимо пользовательских маркеров есть и встроенные, служебные:

- @pytest.mark.skip(reason="...") - пропустить тест всегда.
- @pytest.mark.skipif(условие, reason="...") - пропустить при выполнении условия.
- @pytest.mark.xfail(reason="...", strict=False) - ожидаемый провал; не ломает прогон. strict=True делает "неожиданный успех" (XPASS) ошибкой.
- @pytest.mark.parametrize(...) - параметризация теста/фикстуры; можно помечать отдельные параметры через pytest.param(..., marks=...).
- @pytest.mark.usefixtures("fix1", "fix2") - подцепить фикстуры без явных аргументов.
- @pytest.mark.filterwarnings("ignore::WarningType") - локально подавить/поднять уровень предупреждений.

Помимо маркировки отдельных функций тестов можно промаркировать класс или весь файл целиком:

```python
pytestmark = [pytest.mark.integration]

class TestAPI:
    pytestmark = pytest.mark.networkОбъяснить с
```

Также маркировке могут быть подвержены отдельные тест-кейсы созданные в параметризаторе:

```python
@pytest.mark.parametrize("mode", [
    "fast",
    pytest.param("slow", marks=pytest.mark.slow),
    pytest.param("offline", marks=pytest.mark.skip(reason="нет офлайна")),
])
def test_modes(mode):
    ...Объяснить с
```

Плюсом в файле conftest.py (который рассмотрим чуть позже) можно помечать тесты по имени файла/пути/тегам:

```python
def pytest_collection_modifyitems(config, items):
    for item in items:
        if "tests/integration/" in str(item.fspath):
            item.add_marker(pytest.mark.integration)Объяснить с
```

Вообще самый лучший путь использования маркировки заключается в том, что вы сразу планируете какие группы тестов могут быть, группируете их по признаку и отмечаете это в конфиге с понятными названиями. И после гибко управляете ходом тестирования. По мне - это классная функциональность.

## Файл conftest.py

Теперь разберемся с важным элементом pytest - файл conftest.py. Если коротко - это локальный плагин pytest, который автоматически подхватывается для каталога в котором лежит и для всех его подкаталогов. В нем обычно держат фикстуры, хуки, свои CLI-опции, общие для этой группы тестов настройки и прочее.

Важно отметить, что этот файл не импортируется из тестов напрямую - pytest подгружает его самостоятельно. Но в целом никто не мешает держать свои фикстуры и функции в отдельном модуле и импортировать вручную.

В conftest.py определенно не стоит класть тяжелые импорты и код с побочными эффектами на уровне модуля. Ресурсы рекомендуется загружать только с помощью фикстур. Также любые пользовательские утилиты не стоит туда класть и лучше вынести в отдельный импортируемый модуль.

То есть этот файл - это место где складываются “элементы инфраструктуры” для выполнения тестов на уровне текущей директории.

## Фикстуры в pytest

Для упрощения подготовки предварительных компонентов, данных, подключений, конфигов и прочего - используются фикстуры. Это главный инструмент подготовки к тестам. Эти данные отдаются в тест как аргумент по имени и затем корректно убирает после себя.

Как это работает? Если pytest видит, что в функции теста необходим аргумент с именем name - он идет искать фикстуру с таким именем. После осуществляет ее вызов (один раз на область видимости), кэширует результат и передает его через return/yield в тест. И после теста/модуля/сессии выполняет teardown, то есть всё что после yield или через addfinalizer.

Давайте сделаем простой пример теста с фикстурой для наглядности:

```python
@pytest.fixture
def ab():
    # фикстура готовит данные и возвращает их тестам
    return 2, 3

def test_add(ab):
    a, b = ab
    assert a + b == 5

def test_mul(ab):
    a, b = ab
    assert a * b == 6Объяснить с
```

Другой пример, более приближенный к реальным задачам. Пример открытия сессии SSH с использованием фикстур. Сделаем первую фикстуру в которой хранятся данные для авторизации:

```python
@pytest.fixture(scope="session")
def ssh_params():
    return {"host": "127.0.0.1", "user": "admin", "password": "pass", "port": "22"}Объяснить с
```

Сделаем вторую, для возврата объекта с подключением:

```python
@pytest.fixture(scope="session")
def ssh(ssh_params):
    client = paramiko.SSHClient()
    client.set_missing_host_key_policy(paramiko.AutoAddPolicy())

    kw = dict(
        hostname=ssh_params["host"],
        username=ssh_params["user"],
        port=ssh_params["port"],
        timeout=10,
        allow_agent=True,
        look_for_keys=False,
    )

    if ssh_params["password"]:
        kw["password"] = ssh_params["password"]

    client.connect(**kw)
    try:
        yield client            # передаем объект в тест
    finally:
        client.close()            # после теста закрываем сессию
Объяснить с
```

Немного отойдем в сторону. В параметре декоратора добавился параметр scope, который отвечает за область жизни фикстуры, то есть как часто ее нужно создавать и когда уничтожать. Pytest кэширует результат фикстуры в рамках ее scope. Выполнение кода после yield происходит когда область жизни заканчивается. Варианты scope:

- function (по умолчанию) - новая фикстура для каждого теста;
- class - одна фикстура на класс тестов (Test...);
- module - одна на файл с тестами;
- package - одна на пакет (каталог с **init**.py и его подпакеты);
- session - одна на весь прогон pytest;

Дам пару рекомендаций по поводу применения. Если это дешевый/одноразовый ресурс - то используем function. Если это "дорогие" подключения, типа SSH, к БД, HTTP-сессия - то лучше сделать module/section. Если это данные которые нельзя переиспользовать между тестами - то лучше оставить function или сделать функцию-конструктор, которая будет создавать свежий, изолированный экземпляр и не будет создавать "утечки состояния" между тест-кейсами.

Вернемся к кейсу с созданием фикстур для обмена данными по SSH. Добавим следующую фикстуру для исполнения действий с использованием вышеприведенной фикстуры:

```python
@pytest.fixture
def run_ssh(ssh):
    """Функция-запускалка команд по SSH: возвращает (rc, stdout, stderr)."""
    def _run(cmd, timeout=10):
        stdin, stdout, stderr = ssh.exec_command(cmd, timeout=timeout)
        out = stdout.read().decode(errors="replace").strip()
        err = stderr.read().decode(errors="replace").strip()
        rc = stdout.channel.recv_exit_status()
        return rc, out, err
    return _runОбъяснить с
```

Теперь это можно использовать в коде теста, передав данную фикстуру в аргументе:

```python
def test_hostname_matches_expected(run_ssh):
    expected_hostname = "localhost"
    rc, out, err = run_ssh("hostname")
    assert rc == 0, f"'hostname' завершилась с rc={rc}: {err}"
    assert out, "пустой вывод hostname"

    if expected_hostname:
        assert out == expected_hostname, f"ожидали {expected_hostname}, получили {out}"
    else:
        # если ожидание не задано — проверим «здравый» паттерн имени
        assert re.fullmatch(r"[A-Za-z0-9][A-Za-z0-9._-]*", out)Объяснить с
```

Проверим, запустив тест что у нас выйдет:

```bash
# uv run pytest

…

collected 1 item                                                                                         

tests/test_example.py::test_hostname_matches_expected FAILED             [100%]

============== FAILURES ==============
______________ test_hostname_matches_expected ______________

run_ssh = <function run_ssh.<locals>._run at 0x7d87ac928540>

    def test_hostname_matches_expected(run_ssh):
        expected_hostname = "localhost"
        rc, out, err = run_ssh("hostname")
        assert rc == 0, f"'hostname' завершилась с rc={rc}: {err}"
        assert out, "пустой вывод hostname"
    
        if expected_hostname:
>           assert out == expected_hostname, f"ожидали {expected_hostname}, получили {out}"
E           AssertionError: ожидали localhost, получили NB-1135-LNX
E           assert 'NB-1135-LNX' == 'localhost'
E             
E             - localhost
E             + NB-1135-LNX

tests/test_example.py:49: AssertionError
============== short test summary info ==============
FAILED tests/test_example.py::test_hostname_matches_expected - AssertionError: ожидали localhost, получили NB-1135-LNX
============== 1 failed in 0.49s ==============
Объяснить с
```

Думаю пример более чем показательный. Идём дальше.

## Хуки в pytest

Разберемся что такое хуки в pytest и как их использовать себе на пользу. Как писал выше в определениях - это специальные функции-обработчики жизненного цикла тестов. Pytest сам вызывает их в определённые моменты (старт сессии, парсинг CLI, коллекция тестов, параметризация, запуск, репортинг). С помощью них можно настраивать поведение pytest без изменения тестов: фильтровать/переименовывать тесты, добавлять опции командной строки, параметризовать «на лету», вмешиваться в отчеты и т.д. К слову parametrize - это частный образец такого хука.

Технически хуки реализованы через библиотеку pluggy; функции называются по шаблону pytest\_<имя\_хука> и пишутся в conftest.py или плагинах.

Приведу несколько примеров использования хуков. Объявление хуков производится в conftest.py.

Первый пример - **pytest\_addoption(parser)** позволяет объявить кастомные ключи для запуска тестов, например для нашего теста с SSH можно было бы добавить следующий хук:

```python
def pytest_addoption(parser):
    grp = parser.getgroup("ssh")
    grp.addoption("--ssh-host", help="SSH host")
    grp.addoption("--ssh-user", help="SSH username")
    grp.addoption("--ssh-password", help="SSH password")
    grp.addoption("--ssh-port", type=int, default=22, help="SSH port")
    grp.addoption("--expected-hostname", help="Expected hostname to assert")Объяснить с
```

А данные потом разобрать фикстурой:

```python
@pytest.fixture(scope="session")
def ssh_params(pytestconfig):
   host = pytestconfig.getoption("--ssh-host") 
   user = pytestconfig.getoption("--ssh-user") 
   password = pytestconfig.getoption("--ssh-password") 
   port = pytestconfig.getoption("--ssh-port") 
   expected = pytestconfig.getoption("--expected-hostname")

   if not host or not user:
      pytest.skip("Нужно задать --ssh-host и --ssh-user")

  return {"host": host, "user": user, "password": password, "port": port, "expected": expected}Объяснить с
```

Далее - **pytest\_configure(config) / pytest\_unconfigure(config)** используется для инициализации/освобождения глобальных объектов (например, лог-файла, временной папки и т.п.)

```python
# conftest.py
import tempfile, os

global_tmp_dir = None

def pytest_configure(config):
    """Выполняется при старте pytest — создаём временный каталог и сохраняем в объект config."""
    global global_tmp_dir
    global_tmp_dir = tempfile.mkdtemp(prefix="pytest_global_")
    config._global_tmp_dir = global_tmp_dir
    # Можно также регистрировать ini-lines: config.addinivalue_line(...)

def pytest_unconfigure(config):
    """Выполняется при завершении pytest — чистим временный каталог."""
    global global_tmp_dir
    if global_tmp_dir and os.path.isdir(global_tmp_dir):
        try:
            import shutil
            shutil.rmtree(global_tmp_dir)
        finally:
            global_tmp_dir = NoneОбъяснить с
```

Следующий хук - **pytest\_collection\_modifyitems(config, items)**. Он позволяет отфильтровать, изменить, переупорядочить найденные тесты. Например скипнуть тесты с выбранным маркером, если не передан никакой другой маркер:

```python
# conftest.py
def pytest_addoption(parser):
    parser.addoption("--run-network", action="store_true", help="run network tests")

def pytest_collection_modifyitems(config, items):
    """Если не передан --run-network, помечаем network-тесты как skip."""
    run_network = config.getoption("--run-network")
    if run_network:
        return
    skip = pytest.mark.skip(reason="use --run-network to run")
    for item in items:
        if "network" in item.keywords:
            item.add_marker(skip)Объяснить с
```

Также можно items.sort(key=...) чтобы переупорядочить запуск (например, быстрые тесты первыми).

```python
# conftest.py
def pytest_collection_modifyitems(config, items):
    """
    Простая сортировка: быстрые первыми, потом медленные (с маркером 'slow')
    Стабильность порядка обеспечивается сравнением по пути и имени.
    """
    items.sort(key=lambda it: ("slow" in it.keywords, str(it.fspath), it.name))
    
# tests/test_fast_slow.py
import pytest
import time

def test_fast_one():
    assert 1 + 1 == 2

def test_fast_two():
    assert "a".upper() == "A"

@pytest.mark.slow
def test_slow_one():
    # имитация медленной операции
    time.sleep(1)
    assert sum(range(10)) == 45

@pytest.mark.slow
def test_slow_two():
    time.sleep(0.05)
    assert "slow".startswith("s")Объяснить с
```

Следующий хук - **pytest\_generate\_tests(metafunc)**, он позволяет осуществлять динамическую параметризацию “на лету”. Например, если тест запрашивает аргументы, то мы передаем ему набор для теста:

```python
# conftest.py
def pytest_generate_tests(metafunc):
    """
    Если тест запрашивает a, b и expected — подставляем набор простых арифметических кейсов.
    """
    if {"a", "b", "expected"} <= set(metafunc.fixturenames):
        cases = [
            (1, 2, 3),
            (2, 2, 4),
            (10, 5, 15),
            (0, 5, 5),
            (-1, 1, 0),
        ]
        ids = [f"{a}+{b}={exp}" for a, b, exp in cases]
        metafunc.parametrize(("a", "b", "expected"), cases, ids=ids)

# tests/test_arith.py
def test_add(a, b, expected):
    assert a + b == expected
Объяснить с
```

Другие полезные хуки - это хуки-этапы запуска теста **pytest\_runtest\_setup(item), pytest\_runtest\_call(item), pytest\_runtest\_teardown(item).** С помощью них, например, можно логировать, менять окружение перед/после или прерывать тест:

```python
# conftest.py
import time

def pytest_runtest_setup(item):
    """
    Вызывается перед setup-частью теста.
    Здесь можно подготавливать окружение, проверять маркеры и т.п.
    Мы просто запомним время старта и напечатаем, что тест собирается запускаться.
    """
    item._runtest_start = time.time()
    print(f"\n[HOOK setup] Preparing to run: {item.nodeid}")

@pytest.hookimpl(hookwrapper=True)
def pytest_runtest_call(item):
    """
    Обёртка вокруг выполнения самого теста.
    Код до yield выполняется до теста, код после yield — после теста.
    Это удобное место для измерения времени, перехвата исключений и т.п.
    """
    print(f"[HOOK call] About to call test: {item.nodeid}")
    start = time.time()
    outcome = yield  # выполнение реального теста происходит здесь
    duration = time.time() - start

    # Получаем информацию об исполнении (исключение/успех) через outcome.get_result() не даёт report,
    # но мы можем посмотреть, упало ли исключение во время вызова (через outcome.exception() в new pytest?)
    # Простая проверка: если тест выбросил исключение, оно будет проброшено дальше, но мы всё равно логируем.
    print(f"[HOOK call] Finished call: {item.nodeid} (duration: {duration:.4f}s)")

def pytest_runtest_teardown(item, nextitem):
    """
    Вызывается после teardown-части теста.
    Здесь можно сделать финальную отчистку или логирование итоговой длительности.
    """
    total = None
    if hasattr(item, "_runtest_start"):
        total = time.time() - item._runtest_start
    print(f"[HOOK teardown] Completed: {item.nodeid} (total: {total:.4f}s)")
Объяснить с
```

Запустив тест с параметром -s можно увидеть следующий вывод:

```python
# uv run pytest -s
...
tests/test_example.py::test_quick [HOOK setup] Preparing to run: tests/test_example.py::test_quick
[HOOK call] About to call test: tests/test_example.py::test_quick
[HOOK call] Finished call: tests/test_example.py::test_quick (duration: 0.0001s)
PASSED[HOOK teardown] Completed: tests/test_example.py::test_quick (total: 0.0003s)
...
Объяснить с
```

Внимательный читатель заметит, что добавилась директива **@pytest.hookimpl(hookwrapper=True)** которая превращает весь хук в “обертку” вокруг всей цепочки реализаций этого же хука - то есть можно исполнить код до и после выполнения всех остальных обработчиков хука. Это делается с помощью yield и всё что до yield выполняется перед основной работой хука, а всё что после yield - после нее. То есть это своеобразный способ сделать “around” обёртку вокруг хука. С помощью него можно также контролировать порядок вызова хуков, не будем углубляться в это и идём дальше.

Следующих хук который я рассмотрю - это **pytest\_runtest\_makereport(item, call)**. Он позволяет добавить кастомный лог в объект отчета для каждого теста и например добавлять свои разделы. Рассмотрим пример с выдачей лога SSH при завале теста, чтобы сразу было видно почему тест завалился:

```python
# conftest.py
@pytest.fixture
def run_ssh(ssh, request):
    """
    Возвращает функцию запуска команды по SSH и параллельно
    пишет сырой DEBUG-лог библиотеки Paramiko в буфер.
    """
    # настраиваем capture лога Paramiko в StringIO
    buf = io.StringIO()
    handler = logging.StreamHandler(buf)
    handler.setLevel(logging.DEBUG)
    handler.setFormatter(logging.Formatter(
        "%(asctime)s %(name)s %(levelname)s: %(message)s"
    ))

    logger = logging.getLogger("paramiko")
    logger.setLevel(logging.DEBUG)        # критично: иначе DEBUG не пойдёт
    logger.addHandler(handler)
    logger.propagate = False              # чтобы не дублировалось в root

    def _run(cmd: str, timeout: int = 10):
        stdin, stdout, stderr = ssh.exec_command(cmd, timeout=timeout)
        out = stdout.read().decode(errors="replace").strip()
        err = stderr.read().decode(errors="replace").strip()
        rc = stdout.channel.recv_exit_status()
        return rc, out, err

    # прикрепим для хука
    _run._paramiko_buf = buf
    _run._paramiko_handler = handler
    _run._paramiko_logger = logger

    # снятие хендлера по окончании теста
    def fin():
        logger.removeHandler(handler)
        handler.flush()
    request.addfinalizer(fin)

    return _run

@pytest.hookimpl(hookwrapper=True)
def pytest_runtest_makereport(item, call):
    # hookwrapper даёт возможность выполнить код до/после получения rep
    outcome = yield
    rep = outcome.get_result()  # pytest.TestReport
    # интересует фаза "call" (сам тест), не setup/teardown
    if rep.when == "call" and rep.failed:
        # если тест использовал фикстуру 'run_ssh' — добавим её содержимое в секцию отчёта
        if "run_ssh" in getattr(item, "funcargs", {}):
            fn = item.funcargs.get("run_ssh")
            if fn and hasattr(fn, "_paramiko_buf"):
                text = fn._paramiko_buf.getvalue()
                rep.sections.append(("paramiko log", text if text.strip() else "<empty>"))
            else:
                rep.sections.append(("paramiko log", "run_ssh fixture not used"))Объяснить с
```

В этом случае можно будет увидеть в логе тестов свой вывод: \\

```bash
------------- paramiko log -------------
2025-09-14 01:33:21,394 paramiko.transport DEBUG: [chan 0] Max packet in: 32768 bytes
2025-09-14 01:33:21,586 paramiko.transport DEBUG: Received global request "hostkeys-00@openssh.com"
2025-09-14 01:33:21,586 paramiko.transport DEBUG: Rejecting "hostkeys-00@openssh.com" global request from server.
2025-09-14 01:33:21,628 paramiko.transport DEBUG: [chan 0] Max packet out: 32768 bytes
2025-09-14 01:33:21,628 paramiko.transport DEBUG: Secsh channel 0 opened.
2025-09-14 01:33:21,629 paramiko.transport DEBUG: [chan 0] Sesch channel 0 request ok
2025-09-14 01:33:21,631 paramiko.transport DEBUG: [chan 0] EOF received (0)
2025-09-14 01:33:21,631 paramiko.transport DEBUG: [chan 0] EOF sent (0)Объяснить с
```

Перейдем к следующим хукам - **pytest\_report\_header(config) / pytest\_terminal\_summary(terminalreporter, exitstatus, config)**, которые добавляют заголовок в начало и сводку в конце запуска. Позволяют снабдить вывод дополнительным сопровождением:

```python
def pytest_report_header(config):
    return f"Project: MyApp | ENV={config.getoption('--env') if config.getoption('--env', None) else 'default'}"

def pytest_terminal_summary(terminalreporter, exitstatus, config):
    tr = terminalreporter
    total = tr._numcollected if hasattr(tr, "_numcollected") else "?"
    passed = len(tr.stats.get("passed", []))
    failed = len(tr.stats.get("failed", []))
    skipped = len(tr.stats.get("skipped", []))
  tr.write_sep("-", f"Summary: collected={total} passed={passed} failed={failed} skipped={skipped}")
Объяснить с
```

Хук **pytest\_assertrepr\_compare(op, left, right)** позволяет переопределить вывод assert на вариант, который нужен именно вам. Сделаем простое расширение отладочного вывода на классической ошибке суммирования чисел с плавающей запятой:

```python
def pytest_assertrepr_compare(op, left, right):
    """
    Делает падения assert для чисел более информативными.
    Показываем левое/правое, разницу и относительную погрешность для float.
    """
    if op == "==" and isinstance(left, (int, float)) and isinstance(right, (int, float)):
        lines = ["numbers differ:"]
        lines.append(f"  left : {left!r}")
        lines.append(f"  right: {right!r}")
        diff = right - left
        lines.append(f"  diff (right - left): {diff!r}")
        if isinstance(left, float) or isinstance(right, float):
            denom = max(1.0, abs(right))  # чтобы не делить на 0
            rel = abs(diff) / denom
            lines.append(f"  abs diff: {abs(diff):.17g}, rel≈{rel:.3e} (vs right)")
            lines.append("  tip: for floats use pytest.approx(...)")
        return lines

def test_float_add_fails():
    # классическая проблема с двоичной арифметикой
    assert 0.1 + 0.2 == 0.3
Объяснить с
```

Будет выведен расширенный вывод:

```bash
FAILED tests/test_example.py::test_float_add_fails - assert numbers differ:
    left : 0.30000000000000004
    right: 0.3
    diff (right - left): -5.551115123125783e-17
    abs diff: 5.5511151231257827e-17, rel≈5.551e-17 (vs right)
    tip: for floats use pytest.approx(...)
Объяснить с
```

Есть несколько важных рекомендаций по использованию хуков. Во-первых их нужно делать **быстрыми** поскольку они вызываются очень часто. Всю логику связанную с ресурсами необходимо держать в фикстурах, а не в хуках. Хуки рекомендуется использовать только для глобальной политики проведения тестов, отчетности.

## Mock в pytest

Часто бывает так, что для тестов не доступны реальные объекты и необходимо сделать “заглушку”, для имитации поведения, возвращаемых значений, исключений или записывает, как объект вызывали для анализа.

В pytest есть такая возможность через модуль **pytest-mock**. Активируется при использовании автоматически без дополнительных импортов и передается через фикстуру mocker. Установка осуществляется стандартно:

```bash
uv add --dev pytest-mockОбъяснить с
```

Первое применение - это **патчинг**, т.е. временная подмена атрибута/функции/класса в точке использования на время теста. Чаще всего используется при замене “тяжелых” объектов и зависимостей. Главное правило патчинга - необходимо применять его там где объект используется (импортирован), а не там где он определен. То есть если в [app.py](http://app.py/) написано from math import sqrt, то цель будет "app.sqrt", а не "math.sqrt".

Приведу пример где мы пропатчим реальную функцию. Например, есть функция в коде, которая запрашивает функцию, которая возвращает JSON:

```python
# webapp.py
import requests

def get_title(url: str) -> str:
    resp = requests.get(url, timeout=1)
    resp.raise_for_status()
    return resp.json()["title"]Объяснить с
```

В директории с тестами сделаем файл с тестом:

```python
# tests/test_webapp.py
from webapp import get_title

def test_get_title_ok(mocker):
    # Патчим "там, где используется": webapp.requests.get
    mock_get = mocker.patch("webapp.requests.get")

    # Настраиваем фейковый ответ
    mock_resp = mocker.Mock()
    mock_resp.raise_for_status.return_value = None
    mock_resp.json.return_value = {"title": "Hello"}
    mock_get.return_value = mock_resp

    # Проверяем поведение
    assert get_title("https://example.com/api") == "Hello"
    mock_get.assert_called_once_with("https://example.com/api", timeout=1)
    mock_resp.raise_for_status.assert_called_once()
Объяснить с
```

В этом случае мы сделали импорт через import requests, то цель патчинга “webapp.requests.get”. В этом случае мы не ходим в сеть и просто возвращаем подготовленный мок-объект.

Следующее применение - **“шпион”**, который обвешивает вызываемую функцию средствами наблюдения без изменения реализации. Приведем простой пример:

```python
import math

def use_sqrt(x: float) -> float:
    return math.sqrt(x)

def test_spy(mocker):
    spy = mocker.spy(math, "sqrt")
    assert use_sqrt(9) == 3
    assert spy.call_count == 1
    assert spy.spy_return == 3
Объяснить с
```

Множество свойств для отслеживания вызываемой функции вы можете найти в документации самостоятельно.

Следующее применение - это **создание простой заглушки** для простых случаев. Допустим есть простой код возвращающий текущее время:

```python
# app_time.py
import time

def seconds_since_epoch() -> int:
    return int(time.time())Объяснить с
```

Сделаем простую заглушку для замены на фиксированное время.

```python
# tests/test_app_time.py
from app_time import seconds_since_epoch

def test_seconds_since_epoch_with_stub(mocker):
    time_stub = mocker.stub(name="time")
    time_stub.return_value = 1_700_000_000  # фиксированное «время»

    # В модуле app_time мы делали \`import time\`, значит патчим здесь:
    mocker.patch("app_time.time.time", time_stub)

    assert seconds_since_epoch() == 1_700_000_000
    time_stub.assert_called_once_with()
Объяснить с
```

Идея простая: mocker.stub — это крошечная вызываемая заглушка. Вы задаёте, что она возвращает (return\_value) или как себя ведёт (side\_effect), а дальше либо передаёте её как зависимость, либо ставите на место реальной функции через mocker.patch.

## В качестве заключения

Я решил дальше не продолжать описание pytest, чтобы не превращать ее в некое подобие руководства по этому очень мощному фреймворку. Целью для меня было бегло раскрыть самые основные функциональные возможности и привести пару-тройку примеров. И кажется рассмотреть все основные возможности получилось.

Вне этой статьи остались вопросы, которые вы можете поизучать после самостоятельно:

- работу с временные файлами/папками через tmp\_path;
- функции работы для захвата вывода capsys;
- другие способы подмены окружения и функций через monkeypatch;
- отчеты и покрытия через плагины;
- параллелизация тестов для их ускорения;
- бэнчмарки;
- способы интеграции с CI;

Ну и конечно же, интегрируйте это все в свои проекты и расширяйте тестовое покрытие, постепенно увеличивая покрытие кода тестами, а вслед за этим придет и опыт и знания.

Удачи!

+60