Документация к DL4TD
======
[Ссылка на ноутбук в Google colab](https://colab.research.google.com/drive/1TDoXxlxGhrbeZfkID5xK0DNK7ikicbZC)

# О проекте

Dl4td - это автоматизированная система для создания и настройки рабочего процесса по подготовки данных для обучения ANN сети на распознавание таблиц в документах через Object Detection. Процесс подготовки данных стостоит из конвертации датасетов в унифицированный формат, преобразования изображений, аугментация методом афинных преобразования данных, некоторая валидация данных и создания входных файлов типа TF Records (train.record - обучающая выборка и val.record - тестовая выборка).  

<!--![WorkFlow](WorkFlow.png)-->

# Инициализация

Для работы проекта нужно установить  Python 3.6.X версии. Также установить все необходимы библиотеки, прописав в директории проекта команду:

```bash
pip install -r requirements.txt
```

После этого нужно установить на свою машину фреймворк TF Object Detection.
> Версия TensorFlow должна быть 1.5.x

[Установка Object Detection на Windows 10](https://medium.com/@marklabinski/installing-tensorflow-object-detection-api-on-windows-10-7a4eb83e1e7b)
[Установка OD и некоторых компонентов](https://tensorflow-object-detection-api-tutorial.readthedocs.io/en/latest/install.html#)

# Структура проекта (в GitHub'e)

| Директория/файл  | Описание |
| ------------------ | ---------- |
| `scripts/`| В этой папке хранятся все необходимые скрипты для преобразований, аугментации, конвертации датасетов и т.д. |
| `└─augmentation_data/main.py`| Скрипт выполняющий аугментацию (расширение) данных |
| `└─create_table_tf_record/create_tf_record.py`| Скрипт преобразовывающий данные из формата PASCAL VOC в формат TF Record (Создаёт два файла: `train.record` и `val.record`)|
| `└─icdar2017_to_pascalvoc/main.py`| Скрипт выполняющий конвертацию датасета из формата ICDAR2017 в формат PASCAL VOC|
| `└─icdar2017_to_pascalvoc/data_structure.py`| Вспомогательный модуль, хранящий функции для работы с данными ICDAR2017|
| `└─icdar2019сtdar_to_pascalvoc/main.py`| Скрипт выполняющий конвертацию датасета из формата ICDAR2019 cTDaR в формат PASCAL VOC|
| `└─image_transform/main.py`| Скрипт преобразовывающий изображения (бинаризует и применяет функцию DistanceTransform)|
| `└─marmot_to_pascalvoc/main.py`| Скрипт выполняющий конвертацию датасета из формата Marmot в формат PASCAL VOC|
| `└─marmot_to_pascalvoc/rect.py`| Вспомогательный модуль, хранящий функции для `marmot_to_pascalvoc/main.py`|
| `└─scitsr_to_pascalvoc/main.py`| Скрипт выполняющий конвертацию датасета из формата SciTSR в формат PASCAL VOC|
| `└─scitsr_to_pascalvoc/ignore.list`| Необходимый файл для корректной работы скрипта `scitsr_to_pascalvoc/main.py`|
| `└─unlv_to_pascalvoc/main.py`| Скрипт выполняющий конвертацию датасета из формата UNLV в формат PASCAL VOC|
| `config.ini`| Файл со всеми параметрами запуска всего процесса. Этот файл необходим для корректного запуска `main.py` скрипта|
| `main.py`| Управляющий скрипт. Этот скрипт является главным в проекте. Он поочерёдно запускает другие процессы и контролирует их|
| `requirements.txt`| Файл с необходимыми библиотеками (для pip install)|
| `transform_data.py`| Вспомогательный модуль для управляющего скрипта, содержащий необходимые функции|

# Настройка перед запуском

Все параметры для запуска находятся в файле `control.ini`. В этом файле параметры поделены на секции. Первая секция `datasets` отвечает за общие параметры для всех датасетов такие? как путь к папке, где будет унифицированный, преобразованный и расширенный датасет и путь к локальной директории, где хранятся временные файлы. Пример:

```ini
    [datasets]
    output_path = Data/output_dir
    local_path = Data/local
```

Далее "добавляем" датасет в наш рабочий процесс. Для этого добавляем секцию `data_NAME`, где `NAME` - название датасета (Только буквами и цифрами. Без пробелов).

В этой секции 4 параметра. Первый параметр `name` нужен для того, чтобы вывести в консоль сообщение о том, какой именно датасет конвертируется (этот параметр необходим для удобства чтения логов). Параметр `path_to_datasets` содержит путь к набору данных. `Script_to_convert` содержит путь к скрипту, который конвертирует данный датасет в формат PASCAL VOC. Параметр `enabled` указывает, следует ли использовать этот набор данных (Если false, то датасет игнорируется и не используется). Секцию `data_NAME` можно дублировать с разными параметрами, тем самым добавляя разные датасеты. 
Пример:

```ini
    [data_Marmot]
    name = Marmot dataset
    path_to_dataset = Data/Marmot
    script_to_convert = scripts/marmot_to_pascalvoc/main.py

    [data_Icdar2017]
    name = ICDAR2017 dataset
    path_to_dataset = Data/ICDAR2017
    script_to_convert = scripts/icdar2017_to_pascalvoc/main.py
```

Следующий раздел - `image_transform`. Этот раздел содержит параметр преобразования изображений `script_to_transform`, который содержит путь к скрипту.
Пример:

```ini
    [image-transform]
    script_to_transform = scripts/image_transform/main.py
```

Раздел `tuning_transform` содержит параметр для запуска сценария, который будет выполнять аугментацию данных. Этот параметр содержит путь к скрипту.
Пример:

```ini
    [tuning_transform]
    script_to_tuning = scripts/augmentation_data/main.py
```

Последний раздел - `records`. В этом разделе содержатся параметры для запуска скрипта, создающего входные файлы для нейронной сети типа TF Records.

Параметр `path_to_output` содержит путь к папке, в которой следует сохранять файлы типа записи. `Path_to_label_map` содержит путь к файлу `label_map.pbtxt`. Файл `label_map.pbtxt` содержит название и ID классов, которые необходимо научиться распознавать.
Пример:

```ini
    [records]
    script_to_create_tf_records = scripts/create_table_tf_record/create_tf_record.py
    path_to_output = Data/output_dir_rec
    path_to_label_map = Data/label_map.pbtxt
```

Если данные не нуждаются в каком-либо шаге (аугментация, конвертация и т.д.) и этот шаг нужно пропустить, то нужно в параметре `script_to_*` указать пустой путь к скрипту. Однако если указать пустой путь к датасету, то скрипт остановится с ошибкой.

# Запуск проекта

Для запуска проект нужно запустить `main.py` файл в любом ide под python. Если файл конфигурации расположен в той же директории, что и скрипт, то скрипт будет выполнять свою задачу в обычном режиме, в противном случае скрипт сгенерирует пустой файл конфигурации для дальнейшего заполнения пользователем.

## Коды ошибок

Скрипт может аварийно завершить выполнения по ряду причин. 

| Код ошибки  | Описание проблемы |
| ------------------ | ---------- |
| 1 | Путь к скрипту или не существует, или указан неверно. |
| 2 | Путь к директории указан неверно. Управляющий скрипт в выводе сообщит где именно (в какой секции config.ini файла) путь указан неверно. |
| 3 | Скрипт, который был запущен управляющем, завершился с ошибкой. Например, скрипт аугментации или конвертации. Управляющий скрипт в выводе сообщит о коде ошибки скрипта |
| 4 | Ошибка вспомогательного модуля. |

Об *успешном* окончании работы скрипт сообщит в выводе фразой "Script finished. File's train.record, val.record were created"

# Добавление своего датасета

Для добавления своего датасета необходимо написать скрипт для конвертации в формат PASCAL VOC.

## Особенность формата PASCAL VOC

PASCAL VOC - это формат, который представляет собой директорию с изображениями и с XML файлами. Также в этом формате присутствует файл trainval.txt, в котором хранятся названия всех файлов.

XML файлы хранят некоторую информацию о классе, который нужно распознать. Они хранят название файла (изображения), ширину и высоту изображения, параметр depth, параметр segmented и коллекцию объектов. В этой коллекции хранится название класса и координаты xmin, xmax, ymin, ymax. Эти координаты описывают, так называемую, область интереса (левый верхний край и нижний правый край).

Пример XML файла:

```xml
<annotation>
    <filename>cTDaR_s001.jpg</filename>
    <size>
        <width>4696</width>
        <height>3746</height>
        <depth>3</depth>
    </size>
    <segmented>0</segmented>
    <object>
        <name>table</name>
        <bndbox>
            <xmin>38</xmin>
            <ymin>36</ymin>
            <xmax>4575</xmax>
            <ymax>3687</ymax>
        </bndbox>
    </object>
</annotation>
```

## Скрипт по конвертации

Скрипт по конвертации должен записать данные размеченной области в XML файл в соответствии  с форматом PASCAL VOC (XML файлы хранятся в папке "annotations/xmls"). Также необходимо записать в файл trainval.txt названия файлов через знак переноса.

```xml
POD_0001tuned_0
POD_0001tuned_1
POD_0001tuned_2
```

Запуск скрипта осуществляется с помощью двух параметров запуска: входная директория с датасетом, директория сохранения нового датасета. Параметры определяются `-i <input folder>` и `-o <output folder>` соответственно.

Код для корректного (для данного проекта) чтения параметров:

```python
import getopt
import sys

argv = sys.argv[1:]

try:
    opts, args = getopt.getopt(argv, "hi:o:", ["input_folder=", "output_folder="])
except getopt.GetoptError:
    print('test.py -i <input folder> -o <output folder>')
    sys.exit(2)
for opt, arg in opts:
    if opt in ("-i", "--i"):
        input_path = arg
    elif opt in ("-o", "--o"):
        output_path = arg
```
