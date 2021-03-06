----
title: О песочнице
prevChapter: /ru/prepare/about-hackage.html
nextChapter: /ru/about-haskell/index.html
----

Как вы помните, создание нашего проекта началось с этой команды:

```bash
$ cabal sandbox init
```

Эта команда создаёт песочницу для проекта. Пришло время узнать, что это за песочница такая и зачем она нужна.

## Проблема

Давным-давно, до выхода `cabal 1.18.0`, существовала проблема, которую хаскелисты назвали `Cabal Hell`. Причина проблемы стара как IT-шный мир: зависимости между пакетами. Вот вам иллюстрация.

В 2012 году мы создали Haskell-проект под названием `Real`, зависящий от целого ряда пакетов из Hackage. В числе прочих находился уже известный вам пакет `text`, версии `1.0.0.1`. И всё было прекрасно до тех пор, пока на этом же компьютере в 2013 году мы не создали второй Haskell-проект под названием `Unreal`, который также использует пакет `text`. Однако проблема в том, что проект `Unreal` использует значительно более свежую версию пакета `text`. Утилита `cabal` радостно обновляет этот пакет с версии `1.0.0.1` до версии `1.1.0.0` - и вуаля! - проект `Real` перестаёт собираться. То есть, конечно, этого может и не произойти (в том случае, если новая версия пакета `text` сохранила полную обратную совместимость), но гарантировать этого не может, к сожалению, никто. Как вы поняли, корень проблемы в том, что экземпляр пакета `text` - один на всех.

## Решение

Решением стала песочница. Суть её чрезвычайно проста: все пакеты, необходимые для конкретного проекта, устанавливаются в его песочницу, то есть локально. Посмотрите внимательно внутрь каталога `Real`:

```bash
$ ls -a | grep sandbox
.cabal-sandbox
cabal.sandbox.config
```

Видите скрытый каталог `.cabal-sandbox`? Это и есть наша песочница: все пакеты, которые мы установим для нашего проекта, будут сохранены внутри этого каталога. Таким образом, если мы, находясь в каталоге `Real`, выполним команду

```bash
cabal install text
```

текущая версия пакета `text` установится не глобально, а в его песочницу. Другие наши проекты ничего не будут знать об этой версии пакета `text`, ведь песочница каждого проекта скрыта от остального мира. Следовательно, когда у нас будет два проекта, каждый из них будет использовать свою песочницу. И если один проект будет зависеть от пакета `text 1.2.0.0`, а другой проект - от пакета `text 1.0.0.1`, никаких конфликтов мы не поймаем.

Разумеется, мы заплатим за это дисковым пространством, ведь если у нас будет десять проектов, зависящих от пакета `text`, то и песочниц будет десять, а значит и экземпляров данного пакета будет столько же. К счастью, мы живём в такую эпоху, когда экономить место на диске уже не нужно. ;-)

Вот и всё, теперь вы знаете о песочнице. Разумеется, это лишь самые азы песочницы, но в большинстве случаев вам и не понадобится знать больше. Если же вы хотите подробностей - вот [очень неплохое руководство](http://coldwa.st/e/blog/2013-08-20-Cabal-sandbox.html).
