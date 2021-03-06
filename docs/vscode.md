# Visual Studio Code

[Visual Studio Code](https://code.visualstudio.com/) - отличный редактор кода от Microsoft.

По умолчанию он предлагает подсветку синтаксиса и определенную степень интеллектуальности кода, но легко устанавливаемые расширения могут превратить его в очень мощную среду разработки.

Начните с загрузки и установки [VS Code](https://code.visualstudio.com/Download).

## Папка рабочей области

В качестве отправной точки вы можете указать VS Code на свою папку `www`.

В меню `File` выберите `Open Folder...` и укажите в ней папку `www`.

Теперь любые файлы, которые вы помещаете в эту папку, будут мгновенно отображаться на панели проводника VS Code.

## Расширения

Перейдите к представлению `Extensions` на левой панели интерфейса - он выглядит как группа из трех блоков с четвертым, плавающим рядом.

### Отключить встроенный PHP

Вы должны отключить встроенные функции языка PHP, поскольку они заменены функциями PHP Intelephense, расширения, которое мы установим на следующем шаге.

Для этого введите `@builtin php` в поле поиска вверху списка расширений. Это вызовет два компонента: основы языка PHP и функции языка PHP. Используйте значок шестеренки, чтобы **Отключить** Функции языка PHP, но не трогайте Основы языка PHP, так как мы по-прежнему будем использовать элементы подсветки синтаксиса этого компонента.

### Установить расширения

Вам нужно будет установить набор расширений, чтобы включить расширенные функции, которые превращают VS Code в отличную среду разработки.

Используйте поле поиска, чтобы найти и установить следующие расширения:

* PHP Intelephense
* PHP Debug
* Visual Studio Intellicode
* SQLTools MySQL/MariaDB

После завершения установки перечисленных здесь расширений убедитесь, что они полностью активированы, выйдя из VS Code и перезапустив его. Это помогает очистить все кешированные настройки и повторно инициализировать движок.

## Отладка

[Отладка](https://code.visualstudio.com/docs/editor/debugging) - это мощный инструмент программирования, который позволяет остановить программу в любой момент и проверить ее среду, состояние и контекст, а затем перейти к следующему коду.

### Добавить Xdebug helper

Большая часть выполняемой вами отладки будет запускаться из вашего браузера, поэтому вам необходимо установить расширение, которое будет инструктировать отладчик о запуске.

Для простоты мы подробно опишем [Xdebug helper для Google Chrome](https://chrome.google.com/webstore/detail/xdebug-helper/eadndfjplgieldjbigjakmdgkmoaaaoc), но [документация Xdebug](https://xdebug.org/docs/remote#browser-extensions) содержит список расширений для других браузеров.

Загрузите расширение и установите его, затем переключите его в режим **Debug**, как описано на веб-странице расширения.

### Конфигурация отладки

Прежде чем мы сможем начать использовать Xdebug в VS Code, нам нужно добавить конфигурацию отладки.

В меню `Run` выберите `Add Configuration...` и выберите `PHP` в следующем диалоговом окне. Это создаст файл `launch.json` с двумя конфигурациями - 'Listen for XDebug' и 'Launch currently open script'.

На левой панели переключитесь в режим `Run` - он выглядит как треугольник с наложенной ошибкой.

В верхней части панели вы найдете гаджет со значком 'Play' (треугольник) и текстом 'Listen for XDebug'. Нажмите треугольник, и вы заметите, что строка состояния в нижней части окна VS Code меняет цвет, показывая, что он прослушивает.

Вы можете в любой момент включить или выключить режим прослушивания.

Чтобы начать отладку скрипта, разместите в коде точки останова, добавив точку в поле слева от номеров строк в скрипте.

Когда сценарий запускается, выполнение приостанавливается при достижении точки останова, что позволяет вам проверить состояние и контекст приложения в этой точке.

### Пример отладки

Откройте скрипт `index.php`, который Laragon поместил в папку `www`.

Поместите точку останова в строке 2 - `if (!empty($_GET['q'])) {` и убедитесь, что VS Code находится в режиме 'Listen for XDebug'.

Откройте новую вкладку браузера и установите помощник Xdebug в режим **Debug**, затем введите URL-адрес `http://localhost/?q=info` в адресную строку.

Xdebug должен запускать VS Code для перехода в режим отладки и приостановки выполнения непосредственно перед запуском операторов в строке 2.

На панели **VARIABLES**, разверните группу **Superglobals**, и вы увидите, что `$_GET` отмечен как `array(1)`, что означает, что у него есть один член. Разверните свернутый раздел `$_GET`, и вы увидите, что его член - это `q` со значением `info`, как и следовало ожидать, просто передав `?q=info` через GET скрипту.

Теперь вы можете использовать кнопку `Step Over` (или нажать F10 на клавиатуре) на панели инструментов отладки, чтобы выполнение продолжалось по одной строке за раз, пока весь PHP не будет завершен и страница не загрузится полностью в браузере.

### Расширенная отладка

Подробное описание того, как использовать Step Debugger, выходит за рамки этого руководства, но в Интернете есть множество ресурсов, которые помогут вам понять суть.

* [Отладка Visual Studio](https://code.visualstudio.com/docs/editor/debugging) (Visual Studio Code Docs)
* [Пошаговая отладка](https://www.fourkitchens.com/blog/article/step-step-through-debugging/) (Four Kitchens)

## Ссылки на ресурсы

* [Скрипт требований XenForo](https://xenforo.com/purchase/requirements-zip)
* [XenForo](https://xenforo.com/purchase/)