---
layout: post
title: Как распарсить данные из сети, разбираемся
date: 2023-01-16
categories: ["dart"]
---

Коротко: 
``` dart

final response = await http.get(Uri.parse(
        'https://ваш_адрес'));
    if (response.statusCode == 200) {
      return response.body;
    } else {
      return 'error';
    }

final jsonList = jsonDecode(jsonString);
      if (jsonList.every((e) => e is Map)) {
        List<Map<String, dynamic>> mapList =
            jsonList.cast<Map<String, dynamic>>();
        List<ModelCity> cities =
            mapList.map((e) => ModelCity.fromJson(e)).toList();
}
```

Словами:

Сделаем запрос с помощью `http` пакета и получим ответ, который имеет тип данных класса `Response`.
У данного объекта есть свойство `body`, который имеет тип данных `String`, и фактически представляет из себя файл JSON, который на бекенде мы превратили в строку, то есть текст.

Теперь нам надо сделать обратное действие на стороне клиента(фронтенд) и превратить эту строку в тип данных, который есть в нашей программе, то есть необходимо сделать десериализацию (deserialization). Мы заранее создали класс, который имеет нужные нам поля (свойства класса), и все это сделано на языке Dart. 
Нам надо каким-то образом на основе полученного текста(он же тип данных String), создать объект, который понимает Dart. 

Для того, чтобы не делать всю эту работу вручную и улучшить читаемость кода, можно воспользоваться различными пакетами, например: `json_annotation package` и `json_serializable package`.
Но в нашем случае нам необходимо понять как все работает и как все устроено, поэтому будем делать все вручную.

Для того, чтобы понять всё нижеописанное, надо понимать, что такое словарь в контексте Dart и что с ним можно делать. 
Если это понимание есть, отлично. Двигаемся дальше. 

Если нет, то вот разбор данной структуры данных. 
[Словарь в Dart, он же Map](https://makires.github.io/articles/posts/map/). 

Итак, мы получаем строку в качестве ответа:

```dart
if (response.statusCode == 200) {
      return response.body;
    }
```

Нам необходимо ее декодировать в Json-объект, который фактически представляет из себя словарь, если говорить на языке Dart, и в котором ключ имеет тип данных `String`, а значение имеет тип `dynamic`.
Однако не все так просто.
Давайте разбираться.

```dart
final jsonList = jsonDecode(jsonString);
```

Если мы наведем курсор  в нашей среде разработки на переменную jsonList, то увидим, что тип переменной у нее `dynamic`. Это всё, что может сказать метод jsonDecode о возвращаемом типе данных, когда мы его вызываем и помещаем в качестве аргумента нашу строку. 
То есть, он не знает какого именно типа будут данные в этой строке.

>Заметка:
В этом конкретном случае использование типа dynamic вполне нормально. Однако иногда могут случаться проблемы,  особенно если вы забудете про то, какой тип данных используете(подразумеваете) в dynamic. Вы можете конечно запомнить, что под dynamic скрывается словарь или что-то другое. Но это не очень эффективно (с точки зрения когнитивной нагрузки).
Поэтому, можно указать настройки у анализатора в файле `analysis_options.yaml`, который находится в корневой папке вашего проекта. И добавить следующие строки кода(либо отредактировать их в таком виде)
```dart
analyzer: 
  strong-mode: 
    implicit-dynamic: false 
```
Но данное правило весьма занудное, учтите это. Оно будет постоянно напоминать вам, что необходимо указать тип данных или явно прописать его как `dynamic`. Но зато вы никогда не забудете какой тип данных используете в своем проекте.

Но если мы всё же посмотрим на содержимое строки, которая нам возвращается из сети, то увидим что это список `List`. Посмотреть это можно через команду `print(jsonString)`, которая выведет в терминал результат. Мы увидим что это не просто словарь, а последовательность словарей, которые расположены в массиве. 
Поэтому мы можем указать тип возвращаемых данных как `List<dynamic>`.

>Кто его там знает, что может бэкенд прислать. ;) Но так как мы сами написали его ([вот для этого приложения](https://whereishappyinrussia.web.app/), то мы-то уже точно знаем, что отправили клиенту. 

Если коротко, то код такой:

```dart
final List<dynamic> jsonList = jsonDecode(jsonString);
```

Теперь, чтобы превратить этот объект в нужный нам, а именно в массив из словарей, необходимо сделать проверку с помощью ключевого слова `is`. Это оператор, который позволяет проверить является ли наша переменная тем или иным объектом.

В нашем случае выглядит это так -

```dart
if (jsonList.every((e) => e is Map)) {
 // описание того, что мы хотим сделать
}
```
А если упростить и показать что происходит в сущности, то это так -

```dart
if (jsonList is Map<String, dynamic>) {
  print('Вы получили словарь, описание того, что хотите с ним сделать');
} else {
  print('Возможно ваш JSON имеет неверный формат');
}
```

Используя метод every() мы проходимся по этому списку и проверяем, является ли каждый элемент в этом списке словарем. Метод every() принимает в себя в качестве аргумента функцию, проверяющую наше условие и по итогу возвращает либо true, либо false.

```dart
every((e) => e is Map));
```

По сути мы делаем следующее: берём элемент, которому временно присваиваем переменную с названием `е` (можно назвать как угодно) и проверяем этот элемент на соответствие типа (ключевое словао `is`)
Если выражение `e is Map` вернёт `true`, то мы поместим этот элемент в наш новый массив с конкретным типом данных. 
Когда мы вызываем метод every(), по сути мы делаем цикл `for`:

```dart
List<Map<String, dynamic>> mapList = [];
for (var element in jsonList) {
  if (element is Map<String, dynamic>) {
    mapList.add(element);
  }
}
```
Но только записан он короче. 

Движемся дальше. 

Ну а далее сделаем `cast` для нужного нам типа и выглядеть это будет так:
```dart
List<Map<String, dynamic>> mapList =
            jsonList.cast<Map<String, dynamic>>();
```
Весьма многословно, но именно такой подход решил мою проблему с ошибкой 

```
Expected a value of type 'List<Map<String, dynamic>>', but got one of type 'List<dynamic>' 
```

В конечном счете пройдем по данному массиву словарей и конвертируем их в наш список объект c помощью заранее написанного конструктора.

```dart
List<ModelCity> cities =
            mapList.map((e) => ModelCity.fromJson(e)).toList();
```            

Подробнее о том, как работает конструктор fromJson можно прочитать [здесь](./another-page.html).
