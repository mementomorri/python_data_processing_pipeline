# Конвеер обработки данных на Python

## Краткое описание

Небольшой проект демонстрирующий мои навыки работы с Python и такими полезными библиотеками как pandas, numpy, pyodbc, sqlalchemy, openpyxl и т.д.\
Проект реализует обработку данных хранящихся на SQL Server, которые записываются туда с помощью SCADA системы Ignition. Входные данные сортируются по времени, медианное значение интервала между поступающими данными является нормой, а отклонения от нормы являются аномальными. Аномальные временные промежутки подсчитываются и восполняются с помощью линейной интерполяции, что делает график поступающих значений более гладким, без скачков в значениях и сестема контроля аварийных ситуаций не подает аварийный сигнал без необходимости. Все действия выполняемые программой логгируются и записываются в лог-файл.\
В результате, выполненный проект расширяет функционал стандартной библиотеки Ignition, дополняя его функционалом, который был желателен для клиента. Окно с вызовом программы выглядит следующим образом.

![](https://github.com/mementomorri/python_data_processing_pipeline/blob/main/images/control_window.png)

## Конфигурация тестового окружения

Конфигурирование будет происходить в два этапа:
* выдача прав на чтение и запись в папку «test»;
* исполнение скрипта на создание хранимой процедуры;

### Выдача прав на чтение и запись

Перейдём к первому этапу конфигурирования. Разархивируем файлы архива «test.rar» по пути «C:\test\», предварительно создав папку «test» при необходимости.\
Выдать права MSSQL на чтение и запись в данной папке, чтобы MSSQL имел возможность вносить изменения в файлы находящиеся внутри папки. Сделать это можно с помощью bat скрипта «2_grant_permissions.bat» находящегося в папке «setup_SQLserver_env». Исполняемый файл можно запустить двойным щелчком мыши или через терминал Windows.\
Проверить результат работы программы можно в свойствах папки «test» в разделе «безопасность».

![](https://github.com/mementomorri/python_data_processing_pipeline/blob/main/images/check_permissions.png)

В случае, если выполнение скрипта невозможно, необходимо выдать права вручную.

### Создание хранимой процедуры

На втором этапе конфигурации ТО нужно выполнить скрипт «1_enable_ext_langs.bat», который даст SQL Server возможность испольнять команды с помощью внешних языков, таких как Python. Если выполнить скрипт не удалось, то можно выдать это разрешение выполнив следующий SQL-запрос к SQL Server:

` ` 
sp_configure 'external scripts enabled', 1;

RECONFIGURE WITH override;
` ` 

В случае успеха на оповестят следующим текстом.

![](https://github.com/mementomorri/python_data_processing_pipeline/blob/main/images/ext_langs_enabled.png)

Таким образом мы разрешили SQL серверу выполнять скрипты сторонних языков. Теперь нужно добавить хранимые процедуры в БД. Сделать это можно посредством скрипта «3_create_procedure.bat», либо вручную выполнив следующий SQL-запрос:

` ` 
CREATE PROCEDURE test_procedure_1
AS
EXECUTE sp_execute_external_script @language = N'Python'
, @script = N'
import sys
sys.path.append("C:\\test")
from main import main_call
main_call()
'
` ` 

Хранимая процедура успешно добавлена, протестируем работу процедуры выполнив её с помощью скрипта «4_start_procedure_1.bat». 

![](https://github.com/mementomorri/python_data_processing_pipeline/blob/main/images/test_procedure.png)

В результате выполнения процедуры мы можем отследить новые записи в лог файле по пути «C:\test\KIUS_Lodochnoe\log\KIUS.log».

![](https://github.com/mementomorri/python_data_processing_pipeline/blob/main/images/log_file.png)

И в выходных данных, в выходном файле экселя по пути «C:\test\KIUS_Lodochnoe\data_test_output\book1.xlsx.

![](https://github.com/mementomorri/python_data_processing_pipeline/blob/main/images/output_xl.png)

Как видно по скриншоту, в выходных данных имеется несколько новых столбцов с расчётами в отличие от исходных данных представленных далее.

![](https://github.com/mementomorri/python_data_processing_pipeline/blob/main/images/default_input.png)

Пути к исходным и выходным данным конфигурируются в конфигурационном файле по пути «C:\test\KIUS_Lodochnoe\config\config.ini». В разделе «Logging» можно указать путь к файлу конфигурации логгера в переменную «config_path ». В разделе «Input_output» указываются пути к файлам экселя считающимися входными «input_excel » и выходными «output_excel », если выходной файл не существует, то пайтон создаст его сам. В переменных «output_sheet » и «input_sheet » хранятся страницы на которые мы записываем выходные данные и читаем входные соответственно

![](https://github.com/mementomorri/python_data_processing_pipeline/blob/main/images/config_file.png)

## Инструкция к файлу конфигурации config.ini

Далее разберемкаждый из параметров файла конфигурации проекта:
[DBconnection]  - Раздел с параметрами подключения к БД
* connection_url = mssql+pyodbc:///?odbc_connect= - Строка формирующая подключение к БД с помощью pyodbc;
* driver = ODBC Driver 17 for SQL Server - Драйвер подключения к БД, в нашем случае SQL Server$
* servername = localhost - Символьное имя или IP адрес сервера;
* database = test - Имя подключаемой БД;
* table = sqlt_data_1_2022_02 - Таблица с данными, в которую вносятся новые записи;
* te = sqlth_te - Таблица с объявлением тегов Ignition;
* te_columns = id,tagpath,scid,datatype,querymode,created,retired - Заколовки в таблице с объявлением тегов;
* UID = sa
* PWD  = Sapassword__ - Авторизационные данные для подключения к БД;

[Logging] - Раздел с параметрами логгирования
* debugging_mode = False - Объявить режим дебаггинга, записывать больше информации в лог файл;
* limit_log_size = 10_000_000 - Ограничить размер лог файла указываемый в байтах, в нашем случае он указан в 10 мегабайт;
* config_path = \config\logging.conf - Путь к файлу конфигурации;

[IO_files] - Раздел с параметрами файлов конфмигурации
* default_columns = tagid,intvalue,floatvalue,stringvalue,datevalue,dataintegrity,t_stamp - Объявление стандартных заголовков колонок в таблице с данными;
* columns_with_data = human_readable_stamp,t_gap,median_gap,percent_of_gaps - Объявление колонок с рассчетными данными добавляемыми в ходе рачетов по временным пропускам, именно их мы будем использовать для записи выходных данных в книгу эксель;
* columns_with_bursts_data = t_stamp_,human_readable_stamp,t_gap,t_increment,t_speed,N_speed - Объявление колонок с рассчетными данными добавляемыми в ходе рачетов выбросов, именно их мы будем использовать для записи выходных данных в книгу эксель;
* n_columns_to_read = 7 - Количиство колонок считываемых из книги эксель в качестве исходных данных;
* input_excel = \data_test_input\test_book.xlsx - Путь к файлу эксель используемому в качестве входных данных;
* input_sheet = Sheet3 - Имя страницы эксель используемой в качестве входных данных для пропусков по времени;
* sheet_with_bursts= Sheet4 - Имя страницы эксель используемой в качестве входных данных для выбросов;
* output_excel = \data_test_output\book1.xlsx - Путь к файлу эксель используемому в качестве выходных данных;
* output_gaps_sheet = Sheet1 - Имя страницы эксель используемой в качестве выходных данных для пропусков по времени;
* output_interpolated_sheet = Sheet2 - Имя страницы эксель используемой в качестве выходных данных для пропусков по времени с интрополированными данными;
* output_sheet_with_bursts = Sheet5 - Имя страницы эксель используемой в качестве выходных данных для выбросов;
* output_interpolated_bursts = Sheet6 - Имя страницы эксель используемой в качестве выходных данных для выбросов с интрополированными данными;

[IO_tags] - Раздел с параметрами конфмигурации тегов
* tag_creation_time = 1643677200000 - Метка создания тега для таблицы “te”;
* tagid_gaps = 2 - id тега для записи количества временных пропусков;
* tagid_interpolated_data = 3 - id тега для записи интрополированных данных по временным пропускам;
* tagid_data_bursts_count = 7 - id тега для записи количества выбросов;
* tagid_data_bursts_interpolated = 8 - id тега для записи интрополированных данных по выбросам;
