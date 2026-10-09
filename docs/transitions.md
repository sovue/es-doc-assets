:::outdated
:::

# Переходы

Переход определяет, как изображение на экране меняется: плавно растворяется, затемняется или сдвигается. Он применяется командой `with` после изменения сцены.

```renpy
label mymod_transition_example:
    scene bg ext_square_day
    with fade

    show sl smile pioneer at center
    with dissolve

    sl "Добро пожаловать!"

    hide sl
    with dissolve
    return
```

`scene` заменяет сцену и убирает прежние изображения на её слое. `show` показывает или заменяет отдельное изображение, а `hide` скрывает его. Сам эффект задаёт `with`.

## Один переход для нескольких изменений

Если фон и персонаж должны появиться одновременно, измените их до одной команды `with`:

```renpy
scene bg ext_square_day
show sl smile pioneer at center
with dissolve
```

Чтобы сначала заменить фон, а затем показать персонажа, примените два перехода:

```renpy
scene bg ext_square_day
with fade
show sl smile pioneer at center
with dissolve
```

Запись `show sl smile pioneer with dissolve` тоже допустима. Она соответствует отдельным командам `with None`, `show` и `with dissolve`: предыдущие накопленные изменения фиксируются перед показом персонажа. Для общего эффекта на несколько изменений удобнее отдельная команда `with`.

## Готовые переходы

| Имя | Эффект |
| --- | --- |
| `dissolve` | Плавное смешивание старой и новой сцены |
| `fade` | Затемнение до чёрного и появление новой сцены |
| `hpunch` | Горизонтальное встряхивание экрана |
| `vpunch` | Вертикальное встряхивание экрана |
| `moveinleft`, `moveinright` | Появление нового изображения с края экрана |
| `moveoutleft`, `moveoutright` | Уход скрываемого изображения за край экрана |
| `wipeleft`, `wiperight` | Замена сцены проходящей по экрану границей |

В «Бесконечном лете» также объявлены `dspr` — короткое растворение для спрайтов, `fade2` и `fade3` — более длинные затемнения, `flash` и `flash2` — переходы через белый цвет. Их определения находятся в `game/globals.rpy`; `flash` также объявлен в `game/media.rpy`, поэтому его длительность зависит от итогового порядка инициализации.

```renpy
show dv angry pioneer with dspr
dv "Эй!"
with hpunch
```

## Своя длительность

Создайте переход с уникальным префиксом мода, чтобы не менять общие эффекты игры:

```renpy
define mymod_slow_dissolve = Dissolve(1.5)
define mymod_fade = Fade(0.5, 0.3, 0.5)
define mymod_flash = Fade(0.1, 0.0, 0.4, color="#ffffff")

label mymod_custom_transition:
    scene bg ext_square_day
    with mymod_fade
    show sl smile pioneer
    with mymod_slow_dissolve
    return
```

Аргумент `Dissolve` — длительность в секундах. У `Fade` три длительности: исчезновение старой сцены, удержание одноцветного экрана и появление новой сцены. В примере `mymod_fade` занимает суммарно 1,3 секунды.

## Изменение без эффекта

`with None` фиксирует текущее состояние без анимации. Следующий переход начинается от этого состояния:

```renpy
scene bg ext_square_day
with None
show sl smile pioneer
with dissolve
```

Здесь фон появляется сразу, а персонаж — плавно. Если убрать `with None`, фон и персонаж могут попасть в один общий переход.

Переходы управляют сменой состояния сцены. Для непрерывного движения, изменения масштаба и других анимаций используйте трансформации и ATL.

Подробнее: [переходы](https://www.renpy.org/doc/html/transitions.html) и [команда with](https://www.renpy.org/doc/html/displaying_images.html#with-statement) в документации Ren'Py.
