# Объекты store

Используя объекты store вы можете изолировать код своей модификации от других модов.

:::info
О том, как создавать свои store при инициализации, читайте в [этой статье](/docs/init#блок-python/аргумент-in).
:::

Все объекты сохранённые в именном store изначально не видны в общем store.

```renpy
init python in my_store:
    a = 10
init python:
     b = a # Выйдет "NameError: name 'a' is not defined"
```

Чтобы получить или изменить любой объект из именного store к нему нужно обращаться напрямую.

```renpy
init python in my_store:
    a = 10
init python:
    b = my_store.a # b будет равен a из my_store
```

:::warning
Импорт объектов прямо из объектов store на текущей версии движка игры **не работает** и будет выдавать ошибки.
:::

Используйте `renpy.store` чтобы получать или изменять значения стандартного store в именных store.

```renpy
init python:
    b = 10
init python in my_store:
    a = renpy.store.b
```

Импортируйте другие именные store чтобы использовать их в вашем именном store или взаимодействуйте с объектами из них напрямую.

```renpy
init python in other_store:
    c = 10
init python in my_store:
    from renpy.store import other_store
    a = other_store.c
    a1 = renpy.store.other_store.c
```
