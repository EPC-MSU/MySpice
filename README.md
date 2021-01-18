## Установка
#### Ubuntu и другие debian-like системы
Установка проверялась на Ubuntu 19.04. PySpice по умолчанию требует ngspice как разделяемую библиотеку. Ngspice в свою очередь в репозитории такую библиотеку не поставляет, нужно собирать самому. Ставим тулчейн и необходимые dev пакеты:

`sudo apt install -y libreadline-dev make build-essential wget python3-pip`

Собираем ngspice:

```
wget http://sourceforge.net/projects/ngspice/files/ng-spice-rework/30/ngspice-30.tar.gz
tar -xvzf ./ngspice-30.tar.gz
cd ./ngspice-30/
./configure --prefix=/usr/local --enable-xspice --disable-debug --enable-cider --with-readline=yes --enable-openmp --with-ngshared
make -j4
sudo make install
sudo ldconfig
```
Устанавливаем MySpice:

`pip3 install git+git://github.com/LukyanovM/MySpice.git`

#### Windows
- Скачиваем ngspice для windows https://sourceforge.net/projects/ngspice/files/ng-spice-rework/30/ngspice-30_dll_64.zip/download. Распаковываем, папку Spice64_dll помещаем в C:\Program Files .


- Устанавливаем miniconda https://repo.anaconda.com/miniconda/Miniconda3-latest-Windows-x86_64.exe . Запускаем *Anaconda Prompt (Miniconda3)* и ставим git:

`conda install git`

- Если без Miniconda3, то  установить Python 3.6 64 бит для всех пользователей. Далее установить зависимости с помощью ***pip install -r requirements.txt***

- И далее ставим уже MySpice:


`pip install git+git://github.com/LukyanovM/MySpice.git`

Если устанавливать из локального источника то:

`python setup.py install`

Так же можно не устанавливать восе. Для этого просто подключить модуль, с извесным местом расположения, к нужному файлу. Например если модуль лежит в каталоге запускаемого приложения, в папке MySpice то импорт в приложение:

`import MySpice.MySpice`

Удаление пакета:

`pip uninstall MySpice`

## Состав модуля

`setup.py`

`MySpice`

​	`__init__`.py

​	`MySpice.py`

`README.md`

`LICENSE`

## Формат cir файла

Файлы можно получать экспортом схемы из Qucs нажатием клавиши F2. 

Поскольку в редакторе используется тот же симулятор, что и при построении ВАХ в python все модели электронных компонентов доступные в редакторе будут моделироваться и в ПО.

Сохраняются они в `\user\.qucs\spice4qucs\spice4qucs.cir`

Пример:

```
* Scheme
C1 _net1 0 10000N 
D_1 _net1 0 DMOD_D_1N4148_1
D_2 0 _net1 DMOD_D_1N4148_1
R1 Input _net1  1000
```

`*  Scheme`  - первая строка обязательно коментарий. 
Далее произвольная схема соединения.  

Проводник `Input` и `0` должны быть обязательно.




## Интерфейс

1. Функция `LoadFile()`. 

   `circuit = spice.LoadFile('пример1.cir')` принимает путь к файлу в формате spice (.cir). Файлы для тестов генерировались в qucs-s версии 0.21. Реализован базовый функционал, секции .include, .subckt, .control игнорируются. Результат схема соединения.

2. Функция `Init_Data()`.

   `input_data = spice.Init_Data(частота, напряжение, предустановленный_резистор, шум)`

3. Функция `CreateCVC()`. 

   analysis = spice.CreateCVC(circuit, input_data, lendata, cycle=1)` проводит анализ переходного процесса на цикле определяемом `номер_периода` длительностью определяемой частотой в струкутре `input_data`. Возвращает экземпляр класса `PySpice.CircuitSimulation`, к которому можно обратиться напрямую для получения напряжения - analysis.input_dummy, для получения силы тока - analysis.VCurrent.`

4. Функция `CreateCVC1()`. 

   analysis = spice.CreateCVC1(circuit, input_data, lendata, name="input_dummy", cycle=1)` проводит анализ переходного процесса на цикле определяемом `номер_периода` длительностью определяемой частотой в струкутре `input_data`. Возвращает экземпляр класса `PySpice.CircuitSimulation`, к которому можно обратиться напрямую для получения напряжения - analysis.input_dummy, для получения силы тока - analysis.VCurrent.` Отличается от предыдущей функции наличием параметра  **name**. Если параметр равен параметру по умолчанию **name="input_dummy"**, то поведение функции полностью аналогично предыдущей. Данный параметр может быть равен имени любого проводника существующего в схеме. Если параметр равен  **name="input"** то массив напряжений для ВАХ будет получен без учета падения напряжения на токоограничивающем резисторе. 

5. Функция SaveFile().

   `spice.SaveFile(analysis, "пример1.csv")` сохраняет данные в формате csv с разделителем ";". Первая строка - напряжение, вторая - сила тока.

   

   Пример кода:

   ```
   import matplotlib.pyplot as plt
   
   import PySpice.Logging.Logging as Logging
   from PySpice.Probe.Plot import plot
   
   from MySpice import MySpice as spice
   
   logger = Logging.setup_logging()
   # Загрузка файла со схемой
   circuit = spice.LoadFile('пример1.cir')
   # Загрузка данных генератора
   input_data = spice.Init_Data(10, 5, 0, 80)
   # Получение ВАХ
   analysis = spice.CreateCVC(circuit, input_data, 100, 10)
   # Сохранение файла со значением тока и напряжения
   spice.SaveFile(analysis, "пример1.csv")
   
   # Отображение графика ВАХ
   figure1 = plt.figure(1, (20, 10))
   plt.grid()
   plt.plot(analysis.input_dummy, analysis.VCurrent)
   plt.xlabel('Напряжение [В]')
   plt.ylabel('Сила тока [А]')
   plt.show()
   ```

   
