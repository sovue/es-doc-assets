:::outdated
:::

# Работа с изображениями

## Позиционирование

При простом использовании `show <Спрайт>` он будет появляться просто по центру, это можно исправить, добавив в конец `at <Позиция>`: `show <Спрайт> at <Позиция>`.

Всего в игре 7 позиций, где может находиться спрайт:

- `fleft`
- `left`
- `cleft`
- `center`
- `cright`
- `right`
- `fright`

![Пять спрайтов на стандартных позициях у ночного автобуса](/docs/img/actions-positions.webp)

На скриншоте выше были использованы следующие команды:

```renpy
scene bg ext_bus_night

show dv angry pioneer2 at left
show un smile pioneer at right
show mi cry_smile pioneer at cleft
show sl serious pioneer at cright
show mt rage panama pioneer at center
```


:::tip
**ВНИМАНИЕ**

Также, если не хотите использовать стандартные расположения спрайтов, можно сделать свои собственные! Для этого необходимо показать спрайт, а затем прописать его расположение по координатам x и y. Для этого используем команду `center`, куда вписываем кортеж из двух float-чисел. Первое число — координата x, второе — координата y.

```renpy
label mod_start:
    show bg ext_square_day with dissolve
    show sl smile pioneer: # Славя стоит левее.
        center(0.4, 0.5)
    show dv normal pioneer: # Алиса стоит правее.
        center(0.6, 0.5)
```

Обращаем внимание, что позиционирование идёт по `center`, а не по `align` или `pos`.

:::


:::info
**ПОДСКАЗКА**

На всякий случай - координаты, используемые в дефолтных позициях:

```renpy
transform center:
    xalign 0.5
    xanchor 0.5
    yanchor 0.0

transform left:
    xalign 0.28
    xanchor 0.5
    yanchor 0.0

transform right:
    xalign 0.72
    xanchor 0.5
    yanchor 0.0

transform fleft:
    xalign 0.16
    xanchor 0.5
    yanchor 0.0

transform fright:
    xalign 0.84
    xanchor 0.5
    yanchor 0.0

transform cleft:
    xalign 0.355
    xanchor 0.5
    yanchor 0.0

transform cright:
    xalign 0.645
    xanchor 0.5
    yanchor 0.0
```

:::

Также можно показать персонажа ближе/дальше. Для этого можно воспользоваться атрибутом `close`/`far`.  


![Те же спрайты с атрибутами far и close: ближе к центру — крупнее](/docs/img/actions-distances.webp)

На скриншоте выше как раз были использованы эти атрибуты. Команды:

```renpy
scene bg ext_bus_night

show dv angry pioneer2 far at left
show un smile pioneer far at right
show mi cry_smile pioneer at cleft
show sl serious pioneer at cright
show mt rage panama pioneer close at center
```

## Анимации появления

### Плавность

Мы можем показать спрайты, БГ и ЦГ с некоторым замедлением, плавно.  

`scene <Название> with <Атрибут>`

Атрибутов для плавности шесть, и отличаются они лишь скоростью:

1. `dspr` - 0,2 сек. Самый быстрый, на практике не особо заметен. Оптимален для смены эмоций спрайтов.
2. `dissolve_fast` - 0,5 сек.
3. `dissolve` - также 0,5 сек. Оптимален для смены BG/CG.
4. `dissolve2` - 2 сек. Хороший вариант для длинных переходов между локациями лагеря.
5. `hell_dissolve` - 50 секунд. На практике не особо применим, но вдруг пригодится.
6. `dissolve_long` - 100 секунд.

Также можно группировать несколько объектов для одновременного начала эффекта:

```renpy
scene bg ext_bus_night
show sl smile pioneer far
with dissolve2
```

В приведённом примере мы попадём в ночной автобус, и перед нами предстанет улыбающаяся Славя. Всё это с задержкой в 2 секунды.

![Анимация: Славя плавно проявляется на фоне ночного автобуса за 2 секунды](/docs/img/actions-dissolve.webp)


:::info
**ПОДСКАЗКА**

Если все вышеприведённые примеры вас не устраивают, можно добавить свой атрибут плавности. Для этого пропишите его в блоке `init`.

Например:

```renpy
init:
    $ trisekundi = Dissolve(3.0)

label my_mod:
    scene bg ext_bus_night
    show sl smile pioneer far
    with trisekundi
```

:::

### Моргание

Мы более чем уверены, что вы видели эффект того, как иногда ГГ моргает или вовсе закрывает глаза. Мы можем реализовать подобную возможность!

- `show blink` - закрыть глаза
- `show unblink` - открыть глаза
- `show blinking` - моргание


:::warning
**Будьте осторожны**

`blink` и `unblink` - это разные эффекты, они не могут отменить друг друга.

:::

Пример:

```renpy
show blink
<Ваши события>
hide blink
show unblink
```

![Анимация: «веки» плавно закрываются, пауза, затем открываются — эффект blinking](/docs/img/actions-blinking.webp)


:::info
**ПОДСКАЗКА**

Также к эффектам моргания можно применить атрибут плавности.  

:::

## Добавление своих изображений

Вполне может быть, что по сюжету вашего мода не хватит имеющихся БГ, ЦГ и спрайтов. В таком случае мы добавим свои.

Для начала нам надо объявить изображение в блоке `init`. Делается это с помощью команды `image`. Например, если есть изображение `city.jpg`, и оно находится в папке `game/mods/my_mod/city.jpg`, то объявить его можно вот так:

```renpy
init:
    image gorod = "mods/my_mod/city.jpg"
```

И использовать его:

```renpy
init:
    image gorod = "mods/my_mod/city.jpg"

label my_mod:
    scene gorod with dissolve
```


:::info
**Тэги к изображениям**

Настоятельно рекомендуем использовать тэги. В приведённом ниже примере слово `bg` выступает в виде тэга к изображению. Если объявить несколько БГ с таким тэгом, то при показе следующего он автоматически заменит предыдущий.

:::

```renpy
init:
    image bg city = "mods/my_mod/city.jpg"
```

Более детальную информацию про **ATL** смотрите в продвинутом руководстве (ссылка появится позже).


