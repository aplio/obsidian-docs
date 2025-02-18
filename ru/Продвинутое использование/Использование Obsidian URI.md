Obsidian поддерживает модифицированную версию URI протокола `obsidian://`, который можно использовать для запуска различных действий в приложении. Это обычно используется в MacOS и мобильных приложениях для автоматизации и выстраивания процессов работы между несколькими приложениями.

Если у вас установлен Obsidian, эта ссылка откроет приложение на вашем устройстве: [Нажмите здесь](obsidian://open)

## Установка Obsidian URI

Чтобы убедиться, что ваша операционная система перенаправляет URI `obsidian://` в приложение Obsidian, вам могут потребоваться дополнительные шаги.

- В Windows достаточно запустить приложение один раз. Это зарегистрирует обработчик пользовательского протокола `obsidian://` в реестре Windows.
- В MacOS однократного запуска приложения должно быть достаточно, однако ваше приложение **должно** иметь версию установщика 0.8.12 или новее.
- В Linux процесс гораздо более сложный:
	- Во-первых, убедитесь, что вы создали файл `obsidian.desktop`. [Более детально смотрите здесь](https://developer.gnome.org/integration-guide/stable/desktop-files.html.en)
	- Убедитесь, что в файле рабочего стола в поле `Exec` указано `Exec=executable %u`. Параметр `%u` используется для передачи URI `obsidian://` приложению.
	- Если вы используете установщик AppImage, вам, возможно, придется распаковать его с помощью `Obsidian-x.y.z.AppImage --appimage-extract`. Затем убедитесь, что директива `Exec` указывает на распакованный исполняемый файл.

## Использование Obsidian URI

Obsidian URIs обычно имеют следующий формат:

```
obsidian://action?param1=value&param2=value
```

- `action` - это действие, которое вы хотите выполнить.

### Кодировка

==Важно==

Убедитесь, что ваши значения правильно закодированы в URI. Например, символы прямой косой черты `/` должны кодироваться как `%2F`, а символы пробела должны кодироваться как `%20`.

Это особенно важно, потому что неправильно закодированный «зарезервированный» символ может нарушить интерпретацию URI. [Подробнее смотри здесь](https://en.wikipedia.org/wiki/Percent-encoding)

### Доступные действия

#### Действие `open` (открыть)

Описание: открывает хранилище Obsidian и, возможно, открывает файл в этом хранилище. 

Возможные параметры:

- `vault` может быть либо именем хранилища, либо ID хранилища.
	- Имя хранилища - это просто имя папки хранилища.
	- ID хранилища - это случайный 16-значный код, присвоенный хранилищу. Этот идентификатор уникален для каждой папки на вашем компьютере. Пример: `ef6ca3e3b524d22f`. Пока нет простого способа найти этот идентификатор, он будет предложен позже в переключателе хранилища. В настоящее время его можно найти в `%appdata%/obsidian/obsidian.json` в Windows. Для MacOS, замените `%appdata%` на `~/Library/Application Support/`. Для Linux, замените `%appdata%` на `~/.config/`.
- `file` может быть либо именем файла, либо путем от корня хранилища к указанному файлу.
	- Для разрешения целевого файла Obsidian использует ту же систему разрешения ссылок, что и обычные `[[викиссылки]]` в хранилище.
	- Если расширение файла - `md`, расширение можно не указывать.
- `path` - абсолютный путь файловой системы к файлу.
	- Использование этого параметра переопределит как параметр `vault`, так и `file`.
	- Это заставит приложение искать наиболее конкретное хранилище, содержащее указанный путь к файлу.
	- Затем оставшаяся часть пути заменяет параметр `file`.

Примеры:

- `obsidian://open?vault=my%20vault`
	Откроет хранилище `my vault`. Если хранилище уже открыто, произойдет фокусировка на окне.

- `obsidian://open?vault=ef6ca3e3b524d22f`
	Это открывает хранилище с ID `ef6ca3e3b524d22f`.

- `obsidian://open?vault=my%20vault&file=my%20note`
	Откроет заметку `my note` в хранилище `my vault`, при условии, что `my note` существует и название файла - `my note.md`.

- `obsidian://open?vault=my%20vault&file=my%20note.md`
	Также откроет заметку `my note` в хранилище `my vault`.

- `obsidian://open?vault=my%20vault&file=path%2Fto%2Fmy%20note`
	Откроет заметку, расположенную по пути `path/to/my note` в хранилище `my vault`.

- `obsidian://open?path=%2Fhome%2Fuser%2Fmy%20vault%2Fpath%2Fto%2Fmy%20note`
	Это будет искать любое хранилище, содержащее путь `/home/user/my vault/path/to/my note`. Затем остальная часть пути передается параметру `file`. Например, если хранилище существует в `/home/user/my vault`, то это будет эквивалентно параметру `file` установленному в `path/to/my note`.

- `obsidian://open?path=D%3A%5CDocuments%5CMy%20vault%5CMy%20note`
	Это будет искать любое хранилище, содержащее путь `D:\Documents\My vault\My note`. Затем остальная часть пути передается параметру `file`. Например, если хранилище существует в `D:\Documents\My vault`, то это будет эквивалентно `file`, установленному в `My note`.
	
#### Действие `search` (поиск)

Описание: открывает панель поиска для хранилища и, при необходимости, выполняет поисковый запрос. 

Возможные параметры:

- `vault` может быть либо именем хранилища, либо идентификатором хранилища. То же, что и в действии `open`.
- `query` (опциональный параметр) - выполняемый поисковый запрос.

Примеры:

- `obsidian://search?vault=my%20vault`
	Это откроет хранилище `my vault` и откроет поисковую панель.

- `obsidian://search?vault=my%20vault&query=MOC`
	Это откроет хранилище `my vault`, откроет поисковую панель и выполнит поисковый запрос `MOC`.
	
#### Действие `new` (новый)

Описание: создает новую заметку в хранилище, при желании с некоторым содержанием. 

Возможные параметры:

- `vault`  может быть либо именем хранилища, либо идентификатором хранилища. То же, что и в действии `open`.
- `name` имя создаваемого файла. Если это указано, расположение файла будет выбрано на основе ваших настроек "Местоположение по умолчанию для новых заметок".
- `file` абсолютный путь к хранилищу, включая имя. Если указано, переопределит `name`.
- `path` глобально абсолютный путь. Работает аналогично параметру `path` в действии `open`, которое переопределяет как `vault`, так и `file`.
- `content` (опциональный параметр)содержимое заметки.
- `silent` (опциональный параметр) установите это, если вы не хотите открывать новую заметку.

Примеры:

- `obsidian://new?vault=my%20vault&name=my%20note`
	Это откроет хранилище `my vault`, создаст новую заметку под названием `my note`.
- `obsidian://new?vault=my%20vault&path=path%2Fto%2Fmy%20note`
	Откроет хранилище `my vault`, создаст новую заметку с расположением `path/to/my note`.
	
#### Действие `hook-get-address` (получить )

Описание: предназначено для использования с [Hook](https://hookproductivity.com/). Копирует markdown-ссылку на текущую активную заметку в буфер обмена в виде URL-адреса`obsidian://open`. Использование: `obsidian://hook-get-address`

Возможные параметры:

- `vault` (опциональный параметр) может быть либо именем хранилища, либо идентификатором хранилища. Если не указан, будет использоваться текущее или последнее выделенное хранилище.

## Сокращенные форматы

Помимо вышеперечисленных форматов, для открытия хранилищ и файлов доступны еще два сокращенных формата записи:

- `obsidian://vault/my vault/my note` эквивалентно `obsidian://open?vault=my%20vault&file=my%20note`
- `obsidian:///absolute/path/to/my note` эквивалентно `obsidian://open?path=%2Fabsolute%2Fpath%2Fto%2Fmy%20note`
