---
title: "Чёрт побери, Git!?!"
source: "https://dangitgit.com/ru"
author:
published:
created: 2025-02-12
description:
tags:
  - "clippings"
---
Git сложен: легко всё испортить, и нереально понять как исправить. Документация Git - это финиш: чтобы найти решение, *тебе заранее надо знать название фишки*, которая вернет всё на место.

*Популярно* опишу несколько ситуаций, из которых мне пришлось выбираться.

## Чёрт побери, я накосячил, где у git волшебная машина времени!?!
```git
git reflog
# вы увидите список всего,
# что сделали в git, во всех ветках!
# у каждого элемента есть индекс HEAD@{индекс}
# найдите тот, перед которым всё сломалось
git reset HEAD@{index}
# волшебная машина времени
```

Используйте это чтобы вернуть случайно удалённые штуки, или убрать то чем Вы всё сломали, или восстановиться после неудачного слияния, или просто вернуться туда, когда всё работало. Я ОЧЕНЬ ЧАСТО использую `reflog`. Снимаю шляпу перед теми, кто предложил добавить это.

## Чёрт побери, я закоммитил и вспомнил, что кое-что забыл!
```git
# сделайте своё изменение
git add . # или добавьте файлы по отдельности
git commit --amend --no-edit
# теперь ваш последний коммит содержит это изменение!
# ПРЕДУПРЕЖДЕНИЕ: никогда не меняйте опубликованные коммиты!
```

Обычно я это использую когда коммичу, потом запускаю тесты/сканеры... и блин, я не поставил пробел после знака равно. Также можно сделать изменения в новом коммите и использовать `rebase -i` чтобы склеить оба коммита вместе, но так в миллион раз быстрее.

*Предупреждение: никогда не изменяйте коммиты, отправленные в публичную ветку! Изменяйте только коммиты в вашей локальной ветке, иначе Вам не поздоровится.*

## Чёрт побери, мне нужно изменить сообщение моего последнего коммита!
```git
git commit --amend
# открывает редактор для смены сообщения
```

Дурацкие требования по наименованию.

## Чёрт побери, Я случайно закоммитил что-то в мастер, хотя это должно быть в новой ветке!
```git
# создаст новую ветку из текущего состояния мастера
git branch какое-то-имя-новой-ветки
# удалит последний коммит из мастера
git reset HEAD~ --hard
git checkout какое-то-имя-новой-ветки
# ваш коммит теперь живёт в этой ветке :)
```

NB: это не будет работать, если вы уже отправили коммит в удалённую ветку, и если вы пробовали сделать это как-то по-другому, может помочь `git reset HEAD@{количество-коммитов-назад}` вместо `HEAD~`. Грусть-печаль. Так же многие люди предлагали сделать то же самое, но короче. Спасибо всем!

## Чёрт побери, я случайно закоммитил не в ту ветку!
```git
# отменяет последний коммит, но оставляет изменения доступными
git reset HEAD~ --soft
git stash
# переключиться на нужную ветку
git checkout имя-нужной-ветки
git stash pop
git add . # или добавьте отдельные файлы
git commit -m "ваше сообщение здесь"
# теперь ваши изменения на нужной ветке
```

Многие люди предлагали использовать `cherry-pick` в такой ситуации, так что выбирайте, то что вам больше нравится!

```git
git checkout имя-нужной-ветки
# скопировать последний коммит из мастера
git cherry-pick master
# удалить из мастера
git checkout master
git reset HEAD~ --hard
```

## Чёрт побери, я пытаюсь открыть diff, но ничего не происходит?!

Если вы уверены, что изменили файлы, но `diff` пуст, возможно вы индексировали изменения (`add`) и нужно добавить специальный флаг.

```git
git diff --staged
```

¯\\\_(ツ)\_/¯ (да, я знаю, что это не баг, а фича, но это неочевидно с первого раза!)

## Чёрт побери, мне нужно отменить коммит, который был 5 коммитов назад!
```git
# найдите коммит, который нужно отменить
git log
# используйте стрелочки, чтобы прокрутить историю
# сохраните хеш нужного коммита
git revert [сохранённый хеш]
# git создаст новый коммит, отменяющий выбранный
# отредактируйте сообщений коммита
# или просто сохраните
```

Вам не нужно откатываться назад и копипастить старый файл в существующий! Если вы закоммитили хрень, её можно убрать с `revert`.

Также можно отменить только один файл вместо целого коммита! Но конечно, (как всегда у git\`а) это совершенно другой набор чёртовых команд...

## Чёрт побери, мне нужно отменить изменения в файле!
```git
# найти хеш коммита, до которого нужно откатиться
git log
# используйте стрелочки, чтобы прокрутить историю
# сохраните хеш нужного коммита
git checkout [сохранённый хеш] -- путь/к/файлу
# старая версия файла окажется в вашем индексе
git commit -m "Ого, теперь не придётся копипастить, чтобы отменить изменения!"
```

Когда до меня это дошло, это было НЕВЕРОЯТНО. НЕ-ВЕ-РО-ЯТ-НО. Если серьезно, то где в этой вселенной `checkout --` лучший способ отменять изменения? :угрожает-линусу-торвальдсу:

## К чёрту всё, я сдаюсь.
```git
cd ..
sudo rm -r чёртов-репозиторий-git
git clone https://some.github.url/чёртов-репозиторий-git.git
cd чёртов-репозиторий-git
```

Спасибо Eric V. за эту подсказку. Все жалобы по поводу использованию `sudo` в этой шутке могут быть направлены сразу ему.

Вообще говоря, если ваша ветка настооолько загажена, что нужно вернуться к удалённому состоянию в "git-корректном стиле" попробуйте это, но это необратимо!

```git
# получить последнее состояние origin
git fetch origin
git checkout master
git reset --hard origin/master
# удалить неиндексированные файлы
git clean -d --force
# повторить checkout/reset/clean для каждой испорченной ветки
```

\*Предупреждение: Этот сайт не является исчерпывающим руководством. И да, есть другие способы сделать тоже самое, но я пришел к этому методом проб и ошибок, и теперь делюсь этим с большой дозой легкомыслия и ругани. Примите это или уйдите!

Большое спасибо этим людям, которые перевели этот сайт на другие языки, вы крутые! [Michael Botha](https://github.com/michaeljabotha) ([af](https://dangitgit.com/af)) · [Khaja Md Sher E Alam](https://github.com/sheralam) ([bn](https://dangitgit.com/bn)) · [Eduard Tomek](https://github.com/edee111) ([cs](https://dangitgit.com/cs)) · [Moritz Stückler](https://github.com/pReya) ([de](https://dangitgit.com/de)) · [Franco Fantini](https://github.com/francofantini) ([es](https://dangitgit.com/es)) · [Hamid Moheb](https://github.com/hamidmoheb1) ([fa](https://dangitgit.com/fa)) · [Senja Jarva](https://github.com/sjarva) ([fi](https://dangitgit.com/fi)) · [Michel](https://github.com/michelc) ([fr](https://dangitgit.com/fr)) · [Alex Tzimas](https://github.com/Tzal3x) ([gr](https://dangitgit.com/gr)) · [Elad Leev](https://github.com/eladleev) ([he](https://dangitgit.com/he)) · [Aryan Sarkar](https://github.com/aryansarkar13) ([hi](https://dangitgit.com/hi)) · [Ricky Gultom](https://github.com/quellcrist-falconer) ([id](https://dangitgit.com/id)) · [fedemcmac](https://github.com/fedemcmac) ([it](https://dangitgit.com/it)) · [Meiko Hori](https://github.com/meih) ([ja](https://dangitgit.com/ja)) · [Zhunisali Shanabek](https://github.com/zshanabek) ([kk](https://dangitgit.com/kk)) · [Gyeongjae Choi](https://github.com/ryanking13) ([ko](https://dangitgit.com/ko)) · [Rahul Dahal](https://github.com/rahuldahal) ([ne](https://dangitgit.com/ne)) · [Martijn ten Heuvel](https://github.com/MartijntenHeuvel) ([nl](https://dangitgit.com/nl)) · [Łukasz Wójcik](https://github.com/lwojcik) ([pl](https://dangitgit.com/pl)) · [Davi Alexandre](https://github.com/davialexandre) ([pt\_BR](https://dangitgit.com/pt_BR)) · [Catalina Focsa](https://github.com/catalinafox) ([ro](https://dangitgit.com/ro)) · [Daniil Golubev](https://github.com/dadyarri) ([ru](https://dangitgit.com/ru)) · [Nemanja Vasić](https://github.com/GoodbyePlanet) ([sr](https://dangitgit.com/sr)) · [Björn Söderqvist](https://github.com/cybear) ([sv](https://dangitgit.com/sv)) · [Kitt Tientanopajai](https://github.com/kitt-tientanopajai) ([th](https://dangitgit.com/th)) · [Taha Paksu](https://github.com/tpaksu) ([tr](https://dangitgit.com/tr)) · [Andriy Sultanov](https://github.com/LastGenius-edu) ([ua](https://dangitgit.com/ua)) · [Tao Jiayuan](https://github.com/taojy123) ([zh](https://dangitgit.com/zh)) . С дополнительной помощью от [Allie Jones](https://github.com/alliejones) · [Artem Vorotnikov](https://github.com/vorot93) · [David Fyffe](https://github.com/davidfyffe) · [Frank Taillandier](https://github.com/DirtyF) · [Iain Murray](https://github.com/imurray) · [Lucas Larson](https://github.com/LucasLarson) · [Myrzabek Azil](https://github.com/mvrzvbvk)

Если вы хотите помочь добавить перевод на свой язык, отправьте PR на [GitHub](https://github.com/ksylor/ohshitgit)