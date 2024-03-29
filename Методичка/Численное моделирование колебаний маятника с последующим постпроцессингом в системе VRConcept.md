# Введение

Численное моделирование становится все более популярным методом решения различных технических задач. Связано это в первую очередь с ростом производительных возможностей даже персональных вычислительных систем. В данном методическом пособии рассмотрена методика моделирования колебательной системы с применением метода Рунге-Кутта 4-го порядка. Приведена программа на языке Python для такого моделирования с последующим постпроцессингом в программном пакете VRConcept.

Для реализации данной задачи потребуется:
1. Среда Python
2. Программная среда VRConcept

# Установка Python

Рассмотрим установку комплексную среду разработки Anaconda, включающую в себя необходимую нам среду разработки Spyder и сам интерпретатор Python. Anaconda содержит в себе намного большее количество различных модулей, каждый из которых заточен под решение некоторого круга задач.

## Скачивание установщика Anaconda

По этой **[ссылке](https://www.anaconda.com/download/#windows)** можно найти установщик Anaconda. Необходимо скачать и запустить установщик.

## Установка Anaconda
Я думаю все знакомы с классическим установщиком Windows. Единственное на чем стоит заострить внимание:![[Pasted image 20240223225525.png]]

Необходимо отметить поле «Add Anaconda to my PATH environment variable», чтобы связать Python системой и сделать возможным вызов Python команд через Windows консоль. В контексте нашей задачи, это требуется, чтобы не было никаких проблем при дальнейшей работе.

# Среда разработки Spyder

При первом Spyder, он предложит провести тур, в котором будут раскрыты основные блоки интерфейса, с которыми вы будете постоянно взаимодействовать по мере разработки.

## Редактор
![[Pasted image 20240307205016.png]]

В данном окне происходит само написание кода программы. Язык Python является высокоуровневым интерпретируемым языком, выполнение кода в котором идет построчно. То есть вы можете написать код, который производит суммирование и выполнить его, а может просто написать консольную команду
```
max(1,2,5,3,7)
```
и получить на выходе![[Pasted image 20240307205305.png]]
Такой подход к проектированию новых приложений и программ является более удобным и делает написание кода более быстрым в сравнении с низкоуровневыми языками программирования. Однако за это нам приходится расплачиваться быстродействием, ведь низкоуровневые языки при всем своем высоком пороге вхождения более эффективны при написании громоздких вычислительных программ.

Вышесказанное формирует нишу использования высокоуровневых языков программирования в контексте вычислительных задач - первичное проектирование. При помощи Python можно проверить концепцию, а вот при необходимости более точного решения нужно подключать тяжелую артиллерию в виде C, C++, C# и т.д.

## Консоль
![[Pasted image 20240307205951.png]]
Данное окно является неким аналогом командной строки, однако уже включено в среду разработки и, что самое главное, имеет прямую связь с значениями параметров в вашей программе и может быть использовано для изменения этих самых параметров.

## Обозреватель переменных
![[Pasted image 20240307210419.png]]
Данное окно позволяет получить информацию об используемых в программе переменных, типах их данных и значениях в этих переменных. Является мощнейшим инструментом отладки, позволяя без каких либо ухищрений просматривать всю информацию о переменной, массиве или любом другом типе данных в ходе работы программы.

## Основные кнопки
![[Pasted image 20240307210932.png]]

Обыкновенное для подобных программ меню, которое включает в себя различные функции для работы с кодом. Самая важная кнопка - зеленый треугольник, которая запускает работу программы.

# Связь Python и VRConcept

Связь Python и VRConcept обеспечивается при помощи протокола UDP. Сервер разворачиваемый в системе VRConcept слушает входящую информацию и сразу вносит изменения во внутренние переменные сцены VRConcept.

Рассмотрим базовый код, который позволит анимировать движение по одной оси. 

Подключаем необходимые модули

```
import socket # Модуль сетевого программирования
import time # Модуль для задержки по времени
import struct # Модуль работы с байтовым представлением
import numpy as np # Модуль для работы с математикой
```


Если каких из этих библиотек нет в вашей системе, поставьте их при помощи команды в консоли
```
pip insatll
```
![[Pasted image 20240308111635.png]]
Python очень удобен для установки новых модулей, ведь большая их часть находится на общедоступном сервере, что позволяет установить актуальную версию всего одной командой.

Инициализация базовых параметров для UDP, таких как локальный IP адрес и порт по которому происходит передача пакетов

```
print("Start TEST!") # Тестовое сообщение о старте программы

# Конфигурация сервера UDP
UDP_IP = "127.0.0.1" #IP Адрес сетевого подключения - локальный адрес
UDP_PORT = 6501 # Порт сетевого подключения

print("UDP_START!!!") # Тестовое сообщение о старте канала связи UDP
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM) # Создаём сокет для передачи массива в VRConcept

DAT=[1,2,3,4,5] #Массив из 4х чисел с плавающей точкой
MAXSTEPS=200 #Максимальное число шагов анимационного цикла
```

Потом в цикле прокручивается запись все новых данных в массив и их последующая передача в VRConcept. Задержка нужна как раз для настройки скорости анимации.

```
for i in range(MAXSTEPS+1):
    print("STEP=:", i) # Печатаем номер шага
    # готовим данные для пересылки
    koef = 0.1
    DAT[0]=np.cos(koef*i)*4
    print("DAT=",DAT)# Печатаем датаграмму

 # Пакуем данные val в байты функцией struct.pack('<d', val)
 # '<d' - это метод упаковки: порядок little-endian - от младшего байта кстаршему
 # тип данных - числа с плавающей точкой double диной 8 байт или массивтаких чисел
 # Подробнее упаковку структур см. https://tirinox.ru/python-struct/

 #Вариант с кодированием параметров из списка
    buf = bytes() #создаем переменную - буфер
 # Заполняем буфер
    for val in DAT:
        buf += struct.pack('<d', val)

    sock.sendto(buf, (UDP_IP, UDP_PORT)) # Отправляем данные серверу
    time.sleep(0.05)#Задержка по времени в сек. для удобства отображения
```

Вот весь код полностью:

```
import socket # Модуль сетевого программирования
import time #Модуль для задержки по времени
import struct # Модуль работы с байтовым представлением
import numpy as np

print("Start TEST!") # Тестовое сообщение о старте программы
# Конфигурация сервера UDP
UDP_IP = "127.0.0.1" #IP Адрес сетевого подключения
UDP_PORT = 6501 # Порт сетевого подключения

print("UDP_START!!!") # Тестовое сообщение о старте канала связи UDP
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM) # Создаём сокет
# Реализация алгоритма управления 3D-моделью
# Тестовый пример - смещение в цикле по шагам
DAT=[1,2,3,4,5] #Датаграмма из 4х чисел с плавающей точкой
MAXSTEPS=200 #Максимальное число шагов
#Цикл по шагам
for i in range(MAXSTEPS+1):
    print("STEP=:", i) # Печатаем номер шага
    # готовим данные для пересылки
    koef = 0.1
    DAT[0]=np.cos(koef*i)*4
    print("DAT=",DAT)# Печатаем датаграмму

 # Пакуем данные val в байты функцией struct.pack('<d', val)
 # '<d' - это метод упаковки: порядок little-endian - от младшего байта кстаршему
 # тип данных - числа с плавающей точкой double диной 8 байт или массивтаких чисел
 # Подробнее упаковку структур см. https://tirinox.ru/python-struct/

 #Вариант с кодированием параметров из списка
    buf = bytes() #создаем переменную - буфер
 # Заполняем буфер
    for val in DAT:
        buf += struct.pack('<d', val)

    sock.sendto(buf, (UDP_IP, UDP_PORT)) # Отправляем данные серверу
    time.sleep(0.05)#Задержка по времени в сек. для удобства отображения
```

Теперь рассмотрим настройку UDP связи со стороны VRConcept.
Для связи, необходимо в VRConcept подключить плагин SimulationManager
![[Pasted image 20240308102057.png]]
Данный плагин позволяет принимать данные снаружи и применять методы внутри VRConcept

Необходимо загрузить в сцену любую модель, задать ей свойство "Симуляция параметров"![[Pasted image 20240308102417.png]]
В окне свойств объекта теперь есть пункт "Симуляция параметров"![[Pasted image 20240308102535.png]]
Необходимо поменять 0 на 1. Это добавить окно "Настройка величин для симуляции"![[Pasted image 20240308102742.png]]
Каждая такая строка принимает определенный метод и параметры. После, приходит UDP пакет, который передает в VRConcept данные для данного метода.
Команды, используемые для симуляции
```
Сервер (VRConcept):
Вход:
В свойстве "Симуляция параметров"
Формат записи
<тип>:<индекс>:<пар2>:<пар3>:<пар4>
тип - что нужно выполнить при входящем пакете
индекс - индекс параметра входящего пакета

тип "move" - перемещение объекта
пар2 - коэфициэнты масштабирования (1 , 0)
пар3 - радиус-векторы осей, по которым будет перемещаться(0 , 1 , 2.0)
пар4 - пуст

тип "rotate" - повернуть объект
параметры как у move

тип "display" - вывести на окно значение
пар2 - количество символов после запятой (0,1,2,3 и т.д. до 15. После 15 числа будут округляться до 15ти знаков.)
пар3 - описания параметра ("текст")
пар4 - сдвиг значения от описания (1.0)

тип "sound" - вывести звук
пар2 - путь файла для проигрывания
пар3 - (если оставить пустым - музыка проигрывается циклично. 1 - музыка проигрывается один раз.
Команды для отправки: "индекс 0" - остановить воспроизведение. "индекс (от 1 до 100)" - громкость проигрывания (запуск проигрывания).
Чтобы запустить звук еще раз, после одиночного проигрывания, сначала остановите воспроизведение ("индекс 0") и отправьте запуск проигрывания еще раз ("индекс (от 1 до 100))
пар4 - пуст

тип "ambient","diffuse","specular","emission","shininess" - материал объекта
пар2 - пуст
пар3 - пуст
пар4 - пуст

Выход:
В свойстве "Симуляция параметров"
Формат записи
<тип>:<индекс>:<пар1>
тип "rot_angle" - отправить поворот объекта
пар1 - минимальный/максимальный угол (1 , 0)

<тип>:<индекс>
тип "local_position" - получить локальную позицию объекта
тип "global_position" - получить глобальную позицию объекта
тип "local_orientation" - получить локальную ориентацию
тип "global_orientation" - получить глобальную ориентацию
тип "local_scale" - получить локальный масштаб объекта
тип "global_scale" - получить глобальный масштаб объекта
тип "angle" - получить угол поворота
!ОБРАТИТЕ ВНИМАНИЕ!
тип "position" и "scale" больше не работают. Вместо них используется "local/global_position/scale"

Симулятор:
Начальная настройка - запустить программу, появится конфиг config.json
max log files - макс кол-во файлов лога
max log size - макс размер лога
output - кол-во выходных значений для симулятора
recv port - IP/UDP порт приема
send port - IP/UDP порт отправки
send url - IPv4 VRConcept'а

Формат записи
<индекс> <значение> - отправить значение по заданному индексу
quit - выход из программы

```

В нашем случае необходимо написать команду "move", поступательного движения со следующими параметрами
![[Безымянный-1 1.png]]
В нашем случае мы передаем массив с одним значением по нулевому индексу, масштаб приравниваем к 1 и применяем для вектора (1,0,0).
![[Pasted image 20240308105910.png]]

Открываем Spyder и VRConcept в таком виде
![[Pasted image 20240308105454.png]]
и запускаем скрипт по нажатию зеленой стрелки и наш объект начинает движение![[Pasted image 20240308105850.png]]
Мы научились связывать Python и VRConcept.
Однако мы хотим иметь возможность перемещать объект по всем степеням свободы - по 6ти. Для этого необходимо прописать для объекта 6 методов
```
move:1:1:1,0,0 # Поступательное перемещение по оси X
move:1:1:0,1,0 # Поступательное перемещение по оси Y
move:1:1:0,0,1 # Поступательное перемещение по оси Z
rotate:1:1:1,0,0 # Врашательное движение вокруг оси X
rotate:1:1:0,1,0 # Врашательное движение вокруг оси Y
rotate:1:1:0,0,1 # Врашательное движение вокруг оси Z
```
Теперь мы можем передавать массив из 6 индексов, каждый из которых отвечает за одну определенную степень свободы.

Теперь рассмотрим концепцию классов в Python.
В нашей задаче можно обойтись и без применения классов, однако их использование значительно упростит работу.
Класс - некий Python объект, который может содержать в себе огромное количество информации, относящееся только к этому объекту. Приведу простой пример простоты инициализации объектов при помощи классов.

Например, нужно задать объект мяч. Без применения классов нам бы пришлось прописывать каждый мяч с нуля, то есть
```
Color_ball1 = 'red'
Radius_ball1 = 10
Material_ball1 = 'rubber'

Color_ball2 = 'blue'
Radius_ball2 = 12
Material_ball2 = 'stone'

Color_ball1 = 'pink'
Radius_ball1 = 15
Material_ball1 = 'wool'
```

Как мы видим, для каждого нового объекта приходится писать однотипные конструкции. Также стоит отметить, что применение функций к таким самописным объектам является не самым удобным процессом.
Теперь рассмотрим решение предыдущей задачи при помощи классов
```
class Ball:
    def __init__(self, color = 'white', radius = 10, material = 'rubber', mass = 10):
    # Функция класса Ball, отвечающая за инициализацию нового объекта
        self.color = color
        self.radius = radius
        self.material = material
        self.mass = mass
        
    def __str__(self):
        return f'Color = {self.color}, radius = {self.radius}, material = {self.material}, mass = {self.mass}'
```
Здесь мы объявляем новый класс Ball, который содержит в себе значения по умолчанию:
1. color = 'white'
2. radius = 10
3. material = 'rubber'
4. mass = 10
Также, этот класс содержит две функции, __init__ и __str__ которые отвечают за инициализацию нового объекта и за вывод данного объекта в консоль
```
Ball1 = Ball()
Ball2 = Ball(color='blue', radius=100,mass = 100)
Ball3 = Ball(radius = 56)

print(Ball1)
print(Ball2)
print(Ball3)
```
Тут мы объявляем три объекта Ball1, Ball2, Ball3 и выводим информацию о них. Вы можете заметить, что можно использовать значения по умолчанию и вводить только те параметры, которые надо изменить.
```
runfile('C:/Users/Dimas/mmm/untitled2.py', wdir='C:/Users/Dimas/mmm')
Color = white, radius = 10, material = rubber, mass = 10
Color = blue, radius = 100, material = rubber, mass = 100
Color = white, radius = 56, material = rubber, mass = 10
```

Одной строкой можно объявить новый объект, применить функцию к этому объекту, вывести информацию об объекте. 
Использование классов показывает парадигму ОПП, когда мы за место того, чтобы писать каждый раз практически новый код, используем универсальные классы-объекты ООП, наследуем эти классы в другие классы, применяем правила записи в параметры классов и правила вывода из параметров классов. 

__Как говорится, разделяй и властвуй.__

Теперь воспользуемся новым методом для задания движения в VRConcept
