# Общие концепции

В следующих разделах подробно рассматриваются некоторые общие системы и концепции, с которыми вы столкнетесь при разработке дополнения XenForo. Если вы знакомы с разработкой XenForo 1.x, то многие из этих концепций покажутся вам знакомыми, хотя их стоит пересмотреть, поскольку есть несколько отличных новых инструментов и функций, которые помогут вам разрабатывать дополнения.

## Компоненты поставщика

XF2 не поддерживается специальной структурой, как XF1, однако мы использовали использование некоторых популярных, хорошо протестированных пакетов с открытым исходным кодом, чтобы помочь с конкретными задачами. Например, мы используем проект с именем [SwiftMailer](https://github.com/swiftmailer/swiftmailer) для отправки электронной почты и проект с именем [Guzzle](https://github.com/guzzle/guzzle) в качестве HTTP клиента. Все сторонние проекты загружаются из каталога `src/vendor`.

В настоящее время разработчики дополнений не могут добавлять свои собственные зависимости в это расположение.

## Интегрированная среда разработки (IDE)

Перед тем, как начать разработку XF2, вы можете потратить некоторое время на оценку приложения, с помощью которого вы фактически будете создавать и редактировать файлы PHP. Обычно это называется IDE. Доступен ряд опций, начиная от простого Блокнота и заканчивая чем-то вроде Sublime Text, который может быть расширен для лучшей поддержки PHP с дополнениями, до подходящей IDE, такой как [PhpStorm](https://www.jetbrains.com/phpstorm/). Внутри мы используем PhpStorm в качестве предпочитаемой IDE. Это коммерческий продукт премиум-класса, но могут быть доступны бесплатные альтернативы. В любом случае, никто не может сказать Вам, какое приложение лучше всего соответствует Вашим требованиям, и вам следует потратить некоторое время на ряд продуктов (даже бесплатных) и использовать этот опыт, чтобы выяснить свои предпочтения.

## Автозагрузчик

XF2 использует автозагрузчик, который автоматически генерируется Composer. Это позволяет автоматически включать весь код XF, код сторонних поставщиков и любой код разработчика дополнений по всему проекту без необходимости вручную `include/require` Ваши классы.

Корнем автозагрузки для всех дополнений XF является каталог `src/addons`. Это означает, что все имена Ваших классов будут относиться к этому базовому местоположению. Также стоит отметить, что XF2 использует строгий шаблон именования «класс на файл». Каждый файл должен содержать только один класс, и имя этого класса должно указывать точное расположение PHP-файла класса в файловой системе.

Например, если вы хотите создать новый класс в файле с именем `src/addons/Demo/Setup.php` (где `Demo` - ваш идентификатор дополнения), тогда этот класс будет называться `Demo\Setup`. И наоборот, если у вас есть класс с именем `Demo\Entity\Thing`, вы будете знать, что файл для этого класса находится по пути `src/addons/Demo/Entity/Thing.php`.

## Пространства имён

В XF мы используем [пространства имен](http://php.net/manual/en/language.namespaces.rationale.php), чтобы мы могли более кратко ссылаться на классы в том же пространстве имен. Рекомендуется, чтобы все дополнения также использовали пространства имен. В приведенном выше примере мы говорили о классе с именем `Demo\Setup`. Используя пространства имен, класс фактически будет называться просто `Setup`, но пространство имен будет установлено на `Demo`. В качестве более конкретного примера мы также говорили выше о классе с именем `Demo\Entity\Thing`. Посмотрим, как будет выглядеть PHP-код для этого класса:

```php
<?php

namespace Demo\Entity;

class Thing
{
	
}
```

Если бы был класс с именем `AnotherThing` в каталоге `src/addons/Demo/Entity`, мы могли бы ссылаться на этот класс в классе `Thing` просто как `AnotherThing`, потому что этот класс находится в том же `Demo\Entity` пространство имен.

## Краткие названия классов

Иногда классы, указанные в XF, сокращаются. Например, если вы хотите вызвать объект `User` (подробнее об объектах ниже), то вы можете увидеть, что имя класса обозначается просто `XF:User`. Использование коротких имен классов и полного имени класса, в которое они разрешаются, полностью зависит от контекста. Следовательно, в контексте вызова сущности короткое имя класса будет преобразовано в следующее полное имя класса `XF\Entity\User`. Часть `XF` указывает путь к файлу (на основе идентификатора дополнения), часть `Entity` подразумевается путем вызова сущности, а часть `User` указывает конкретную сущность. Точно так же, когда вы начнете создавать свои собственные классы, вы также будете использовать короткие имена классов для ссылки на свои собственные классы. Например, если вам нужно создать новую сущность `Thing` для дополнения `Demo`, вы должны написать следующее:

```php
\XF::em()->create('Demo:Thing');
```

Это разрешит класс `Demo\Entity\Thing`. Точно так же, если вы хотите получить доступ к репозиторию `Thing`, вы должны написать его следующим образом:

```php
\XF::repository('Demo:Thing');
```

Обратите внимание на идентичность коротких имен классов. Вызов репозитория фактически будет преобразован в `Demo\Repository\Thing`.

## Расширение классов

Большое количество классов в XF2 являются расширяемыми, что позволяет разработчикам расширять и переопределять основной код без необходимости напрямую редактировать его. Если вы знакомы с разработкой XF1, вы будете в некоторой степени знакомы со следующим процессом:

1. Создайте файл PHP прослушивателя
2. Создайте класс, который в конечном итоге расширит исходный класс 
3. Напишите функцию, которая соответствует ожидаемой сигнатуре обратного вызова для одного из событий `load_class` и добавляет имя Вашего расширенного класса
4. Добавьте «код прослушивателя события» в Admin CP, который указывает класс прослушивателя и имя метода для функции, упомянутой выше, и, при необходимости, подсказку относительно того, какой класс расширяется.

В XF2 мы удалили эти события в пользу специальной системы под названием «Расширения классов». Процесс выглядит следующим образом:

1. Создайте класс, который в конечном итоге расширит исходный класс
2. Добавьте «Расширение класса» в Admin CP, в котором указывается имя класса, который вы расширяете, и имя класса, который его расширяет.

Это явно сокращает количество шаблонов, необходимых для расширения классов, а также предоставляет специальный пользовательский интерфейс для просмотра и управления этими расширениями. Давайте посмотрим на процесс, расширив общедоступный контроллер `Member` и добавив новое действие, которое отображает простое сообщение.

Первое, что нужно сделать, это создать надстройку. Ранее мы описали, как это сделать, используя команду `xf-addon:create` [здесь](development-tools.md#creating-a-new-add-on). В этом примере мы предполагаем, что вы создали надстройку с идентификатором и названием «Демо».

Теперь у вас будет файл addon.json для этого дополнения в следующей папке `src/addons/Demo/addon.json`.

!!! note
	Хотя, строго говоря, вы можете размещать свои расширенные классы где угодно в каталоге надстроек, рекомендуется помещать расширенные классы в каталог, который легко идентифицирует а) надстройку, к которой принадлежит класс; б) тип класса расширяется и в) имя расширяемого класса. В следующих примерах мы расширяем общедоступный контроллер XF Member, поэтому мы разместим наш расширенный класс по следующему пути: `src/addons/Demo/XF/Pub/Controller/Member.php`.

Расширенный класс должен существовать до того, как мы добавим расширение класса в Admin CP. Итак, следуйте следующей инструкции:

1. Создайте новый каталог с именем `XF` внутри `src/addons/Demo`
2. Создайте новый каталог с именем `Pub` внутри `src/addons/Demo/XF`
3. Создайте новый каталог с именем `Controller` внутри `src/addons/Demo/XF/Pub`
4. Создайте новый файл с именем `Member.php` внутри `src/addons/Demo/XF/Pub/Controller`

Первоначальное содержимое Вашего файла PHP должно быть следующим:

```php
<?php

namespace Demo\XF\Pub\Controller;

class Member extends XFCP_Member
{
	
}
```

Если вы знакомы с расширением классов PHP в целом, но не знакомы с XF, приведенный выше пример поначалу может показаться запутанным. Причина этого в том, что Вы, возможно, ожидали расширения класса `XF\Pub\Controller\Member` напрямую, а не `XFCP_Member`. В XF мы используем систему "XenForo Class Proxy" (сокращенно XFCP) для построения «цепочки наследования», которая в конечном итоге позволяет расширить один класс несколькими надстройками. Соглашение заключается в том, чтобы ссылаться на фиктивный расширенный класс, который является текущим именем класса `Member`, и ставить перед ним префикс `XFCP_`.

Теперь, когда класс создан, мы можем создать расширение класса на странице "Admin CP > Development > Class extensions > Add class extension".

Все, что вам нужно сделать, это ввести имя базового класса (`XF\Pub\Controller\Member`) в первое поле и имя расширенного класса (которое вы только что создали) во втором поле (`Demo\XF\Pub\Controller\Member`) и нажмите кнопку «Сохранить».

Расширение Вашего класса теперь должно быть активным, но в настоящее время ничего не делает. Чтобы что-то произошло, нам нужно либо переопределить существующий метод в этом классе, создав метод с тем же именем, что и существующий, либо полностью добавить новый метод. Сделаем последнее:

```php
<?php

namespace Demo\XF\Pub\Controller;

class Member extends XFCP_Member
{
	public function actionHelloWorld()
	{
		return $this->message('Hello world!');
	}
}
```

Мы говорим больше о контроллерах, действиях и ответах на страницах [Основы работы с контроллерами](controller-basics.md), поэтому не беспокойтесь об этом прямо сейчас.

Теперь мы добавили код в наш расширенный контроллер, давайте посмотрим, как он работает. Просто введите следующий URL (относительно URL Вашего форума): `index.php?members/hello-world`. Теперь вы должны увидеть "Hello world!" сообщение отображается!

Как упоминалось ранее, также можно переопределить существующие методы в классе. Например, если мы изменим `actionHelloWorld()` на `actionIndex()`, то у вас больше не будет списка "Notable members", вместо этого будет отображаться «Hello world!» сообщение! Это не совсем правильный способ расширения существующего действия контроллера (или любого метода класса, на самом деле), но мы более подробно рассмотрим это в [Модификация ответа на действие контроллера (правильно)](controller-basics.md#modifying-a-controller-action-reply-properly). 

## Подсказка типов

Многие объекты в XF создаются с помощью фабричных методов. Например, если мы хотим создать конкретный репозиторий, мы должны написать следующее:

```php
$repo = \XF::repository('Demo:Thing');
```

Это очень удобный и последовательный способ создания экземпляра объекта. Мы знаем, просто глядя на него, какой объект будет создан. Результирующий код в этом методе знает, как вернуть правильный объект для того, что мы запросили.

К сожалению, Ваша IDE, вероятно, не имеет ни малейшего понятия (по крайней мере, по умолчанию). Что касается IDE, этот метод вернет экземпляр объекта `XF\Mvc\Entity\Repository`. В определенной степени это полезно, но потенциально существует множество методов, доступных в конкретном объекте `Demo\Repository\Thing`, о которых Ваша IDE не знает. В конечном итоге это означает, что когда вы пытаетесь использовать свой объект `$repo` в коде, Ваша IDE не сможет вносить предложения или автоматически заполнять имена методов и аргументы, которые для этого требуются.

Вот где подсказки типов становятся полезными, а синтаксис должен поддерживаться по стандарту большинством IDE и некоторыми текстовыми редакторами, поддерживающими PHP. Мы бы просто изменили вызов нашего репозитория следующим образом:

```php
/** @var \Demo\Repository\Thing $repo */
$repo = \XF::repository('Demo:Thing');
```

Подсказка типа над вызовом репозитория теперь сообщает IDE, что `$repo` относится к объекту, представленному классом `Demo\Repository\Thing`, а не к объекту, который он автоматически выводит изначально.

Подсказка типов также особенно полезна при расширении классов. Потенциальная проблема с нашими методами расширения класса заключается в том, что, по сути, Ваши классы не расширяют исходный класс, который вы хотите расширить, а вместо этого он проксируется через класс, который на самом деле не существует, например, `XFCP_Member`, такой как в [примере выше](#extending-classes).

Чтобы исправить эту проблему, мы автоматически создаем файл с именем `extension_hint.php` и сохраняем его в Вашем каталоге `_output`.

Это добавляет ссылку, которую IDE может читать, но PHP не может, поэтому теперь IDE понимает, что, когда мы используем `$this` внутри любого из методов в этом расширенном классе, он может предлагать и автоматически заполнять методы и свойства, доступные в элементе контроллер или один из его родителей.