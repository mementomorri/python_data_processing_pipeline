# Конвеер обработки данных на Python

### Краткое описание

Небольшой проект демонстрирующий мои навыки работы с Python и такими полезными библиотеками как pandas, numpy, pyodbc, sqlalchemy, openpyxl и т.д.\
Проект реализует обработку данных хранящихся на SQL Server, которые записываются туда с помощью SCADA системы Ignition. Входные данные сортируются по времени, медианное значение интервала между поступающими данными является нормой, а отклонения от нормы являются аномальными. Аномальные временные промежутки подсчитываются и восполняются с помощью линейной интрополяции, что делает график поступающих значений более гладким, без скачков в значениях и сестема контроля аварийных ситуаций не подает аварийный сигнал без необходимости.\
В результате, выполненный проект расширяет функционал стандартной библиотеки Ignition, дополняя его функционалом, который был желателен для клиента.
