# Основы контроллера 

На базовом уровне контроллеры - это код, который выполняется, когда вы посещаете страницу в XF. Контроллеры обычно несут ответственность за обработку пользовательского ввода и передачу этого пользовательского ввода в соответствующее место, которое, как правило, должно выполнять какое-либо действие с базой данных (Модель) или загружать визуальный контент (Представление).

Когда пользователь кликает ссылку, запрошенный URL-адрес перенаправляется на конкретный контроллер и действие контроллера. Смотрите [Основы маршрутизации](routing-basics.md). Например, в XF, если вы перейдете по URL-адресу типа `index.php?conversations/add`, вы будете перенаправлены на контроллер `XF\Pub\Controller\Conversation` и на действие `add`.

Если вы посмотрите на этот класс в файловой системе (смотрите [Автозагрузчик](general-concepts.md#autoloader) для описания того, как классы и пути к файлам сопоставляются друг с другом), вы заметите, что существует ряд методов, названных с префиксом `action`. Все эти методы указывают на конкретное действие контроллера. Итак, чтобы увидеть код, задействованный при просмотре упомянутой выше страницы разговоров/добавления, поищите в этом файле `public function actionAdd()`.

Контроллеры XF отвечают за возврат объекта ответа, который обычно состоит из одного из следующих типов:

## Ответ представления

Это один из наиболее распространенных ответов, с которыми вы будете иметь дело во время разработки XF. Контроллер, который возвращает ответ представления, обычно требует передачи до трех аргументов. Класс представления (подробнее об этом ниже), имя шаблона и массив `$viewParams` , который является данными, которые должны быть доступны для шаблон.

Вот типичный пример действия контроллера, которое возвращает ответ Представления:

```php
public function actionExample()
{
    $hello = 'Hello';
    $world = 'world!';
    
    $viewParams = [
        'hello' => $hello,
        'world' => $world
    ];
    return $this->view('Demo:Example', 'demo_example', $viewParams);
}
```

Первый аргумент - это сокращенное имя класса для определенного класса View. Этот класс может существовать, а может и не существовать (часто в нем нет необходимости, мы рассмотрим классы представлений позже), но он должен иметь примерно уникальное имя для контроллера и действия. Как и в случае с другими [Сокращенными именами классов](general-concepts.md#short-class-names), конкретное короткое имя класса выше будет преобразовано в `Demo\Pub\View\Example`. Опять же, `Pub` автоматически выводится из типа контроллера.

Второй аргумент - это имя шаблона. В этом случае мы ищем шаблон с именем `demo_example`.

Третий аргумент - это массив параметров/переменных шаблона, которые должны быть доступны для представления. Этот массив обычно должен состоять из пар `key => value`. В приведенном выше примере в шаблон передаются два параметра шаблона. Часть массива `key` указывает имя переменной, доступной в шаблоне. Часть массива `value` указывает значение.

Итак, если бы у нас было следующее содержимое в шаблоне `demo_example`:

```html
{$hello} {$world}
```

Шаблон выведет следующее:

```plain
Hello world!
```

## Ответ редиректа

Этот ответ возвращается, когда вы хотите перенаправить пользователя на другой URL-адрес после того, как он выполнил какое-либо действие.

Типичный вариант использования здесь - после того, как пользователь отправил данные через форму, вы можете перенаправить их на другую страницу, например, вернуть пользователя к списку элементов.

Вот пример типичного действия контроллера, которое выполняет перенаправление:

```php
public function actionRedirect()
{
    return $this->redirect($this->buildLink('demo/example'), 'This is a redirect message.', 'permanent');
}
```

Первый аргумент - это URL-адрес для перенаправления. Этот пример перенаправит пользователя на URL-адрес `index.php?demo/example`.

Второй аргумент будет отображаться только в том случае, если форма отправлена по запросу AJAX, который предотвращает перенаправление. Результатом будет «флэш-сообщение», которое появится вверху экрана с выбранным Вами сообщением. вам не нужно предоставлять собственное сообщение. Если он не указан, по умолчанию будет установлено «Ваши изменения были сохранены».

Третий аргумент по умолчанию - `temporary`, но вы также можете выбрать постоянное значение, как в примере. Единственная разница здесь - это тип кода HTTP-ответа, предоставляемого сервером. В большинстве случаев идеально подходит временный режим, и он будет отвечать кодом 303. `permanent` выдаст код ответа 301.

Хотя таким образом вы можете вызвать постоянное перенаправление, на самом деле для этого есть специальный метод, который можно использовать следующим образом. Он также принимает аргумент 'message', но, как и выше, не является обязательным.

```php
public function actionRedirect()
{
    return $this->redirectPermanently($this->buildLink('demo/example'));
}
```

## Ответ об ошибке

Как следует из названия, вы вернете этот ответ, если вам нужно отобразить ошибку для пользователя. Это несколько просто, вот пример:

```php
public function actionError()
{
    return $this->error('Unfortunately the thing you are looking for could not be found.', 404);
}
```

Здесь поддерживаются только два аргумента. Первое - это сообщение об ошибке, которое вы хотите отобразить, а второе - это код ответа HTTP, который вы хотите, чтобы сервер отправил. 404 представляет собой соответствующий ответ, если что-то не было найдено.

## Ответ сообщение

Этот ответ очень похож на ответ об ошибке и поддерживает те же аргументы. Основное отличие заключается в том, что отображаемое сообщение не отображается как ошибка.

## Ответ исключение

Иногда необходимо прервать нормальный поток кода Вашего контроллера и вместо этого ответить исключением. Ответы на исключения не обязательно должны представлять ошибку; например, их можно использовать, чтобы заставить ваш контроллер выполнить перенаправление. Однако, как правило, они часто используются для остановки потока Вашего контроллера для отображения ошибки, как в следующем примере:

```php
public function actionException()
{
    throw $this->exception($this->error('An unexpected error occurred'));
}
```

Ответы на исключения принимают только один аргумент, и на самом деле этот аргумент должен быть какой-либо другой формой объекта Reply, например [Error reply](#error-reply). Этот конкретный пример вызывает исключение, и весь код контроллера в этот момент останавливается, и отображается стандартная ошибка.

Обратите внимание, что ответы об исключениях должны «выбрасываться» с помощью `throw`, а не с помощью `return`.

## Перенаправление ответа

При определенных условиях необходимо перенаправить пользователя на совершенно другой контроллер или действие в том же контроллере без выполнения полного перенаправления, без изменения URL-адреса, на который перешел пользователь, и без необходимости дублировать код целевого действия.

Это выглядит примерно так:

```php
public function actionReroute()
{
    return $this->rerouteController(__CLASS__, 'error');
}

public function actionError()
{
    return $this->error('Oops! Something went wrong!');
}
```

В этом конкретном примере, если пользователь перешел по URL-адресу `index.php?demo/reroute`, он увидит ответ об ошибке от метода `actionError()`. Они не будут перенаправлены, и URL-адрес в их браузере не изменится; они просто получат ответ от действия ошибки.

Ответ перенаправления также поддерживает третий аргумент, который позволяет передавать различные параметры от одного действия контроллера к другому. Это может быть либо массив, либо объект `ParameterBag` (подробнее об этом позже).

## Изменение ответа на действие контроллера (должным образом)

В разделе [Расширение классов](general-concepts.md#extending-classes) мы уже видели, насколько просто расширить класс, но при расширении уже существующего действия контроллера необходимо проявлять особую осторожность.

Если у вас нет особой необходимости полностью переопределить существующее действие и заменить его чем-то новым (что обычно не рекомендуется), вместо этого вам следует изменить существующий ответ родительского класса. Это делается довольно просто, в качестве примера давайте изменим ответ представления из приведенного выше примера [Ответ представление](#view-reply).

```php
public function actionExample()
{
    $reply = parent::actionExample();
    
    return $reply;
}
```

Предполагая, что указанное выше добавлено к расширенному контроллеру, в котором уже существует метод `actionExample()`, приведенное выше фактически не делает ничего, кроме возврата исходного ответа представления. Теперь давайте изменим существующий параметр `hello` на "Bonjour" вместо "Hello".

```php
public function actionExample()
{
    $reply = parent::actionExample();
    
    if ($reply instanceof \XF\Mvc\Reply\View)
    {
        $reply->setParam('hello', 'Bonjour');
    }
    
    return $reply;
}
```

Поскольку ответ контроллера на самом деле может представлять ряд различных объектов, которые имеют разное поведение и методы, крайне важно, чтобы мы пытались расширить только правильный тип ответа. Мы делаем это в приведенном выше примере, проверяя, действительно ли родительский объект `$reply` является типом `View`. Если бы мы этого не сделали, мы расширили это действие, и действие контроллера вместо этого отвечает перенаправлением, тогда, скорее всего, возникнет ошибка.

Перед расширением этого действия при посещении этой страницы будет отображаться "Hello world!". После его расширения в представлении теперь будет отображаться "Bonjour world!".
