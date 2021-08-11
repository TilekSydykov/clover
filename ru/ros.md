ROS
===

Основная статья: http://wiki.ros.org

ROS – это широко используемый фреймворк для создания сложных и распределенных робототехнических систем.

Установка
---

Основная статья: http://wiki.ros.org/melodic/Installation/Ubuntu

ROS уже установлен на [образе для RPi](image.md).

Для использования ROS на компьютере рекомендуется ОС Ubuntu Linux (либо виртуальная машина, например [Parallels Desktop Lite](https://itunes.apple.com/ru/app/parallels-desktop-lite/id1085114709?mt=12) или [VirtualBox](https://www.virtualbox.org)).

> **Note** Для дистрибутива ROS Melodic рекомендуется Ubuntu версии 18.04.

Концепции
---

### Ноды

Основная статья: http://wiki.ros.org/Nodes

ROS-нода – это специальная программа (обычно написанная на Python или C++), которая взаимодействует с другими нодами посредством ROS-топиков и ROS-сервисов. Разделение сложных робототехнических систем на изолированные ноды дает определенные преимущества: понижается связанность кода, повышается переиспользуемость и надежность.

Очень многие робототехнические библиотеки и драйвера выполнены именно в виде ROS-нод.

Для того, чтобы превратить обычную программу в ROS-ноду, необходимо подключить к ней библиотеку `rospy` или `roscpp` и добавить инициализирующий код.

Пример ROS-ноды на языке Python:

```python
import rospy

rospy.init_node('my_ros_node')  # имя ROS-ноды

rospy.spin()  # входим в бесконечный цикл...
```

### Топики

Основная статья: http://wiki.ros.org/Topics

Топик – это именованная шина данных, по которой ноды обмениваются сообщениями. Любая нода может *опубликовать* сообщение в произвольный топик, а также *подписаться* на произвольный топик.

Пример публикации сообщения типа [`std_msgs/String`](http://docs.ros.org/api/std_msgs/html/msg/String.html) (строка) в топик `/foo` на языке Python:

```python
from std_msgs.msg import String

# ...

foo_pub = rospy.Publisher('/foo', String, queue_size=1)  # создаем Publisher'а

# ...

foo_pub.publish(data='Hello, world!')  # публикуем сообщение
```

Пример подписки на топик `/foo`:

```python
def foo_callback(msg):
    print(msg.data)

# Подписываемся. При получении сообщения в топик /foo будет вызвана функция foo_callback.
rospy.Subscriber('/foo', String, foo_callback)
```

Также, существует возможность работы с топиками с помощью утилиты `rostopic`. Например, с помощью следующей команды можно просматривать сообщения, публикуемые в топик `/mavros/state`:

```bash
rostopic echo /mavros/state
```

### Сервисы

Основная статья: http://wiki.ros.org/Services

Сервис – это некоторый аналог функции, которая может быть вызвана из одной ноды, а обработана в другой. У сервиса есть имя, аналогичное имени топика, и 2 типа сообщений: тип запроса и тип ответа.

Пример вызова ROS-сервиса из языка Python:

```python
from clover.srv import GetTelemetry

# ...

# Создаем обертку над сервисом get_telemetry пакета clover с типом GetTelemetry:
get_telemetry = rospy.ServiceProxy('get_telemetry', srv.GetTelemetry)

# Вызываем сервис и получаем телеметрию квадрокоптера:
telemetry = get_telemetry()
```

С сервисами можно также работать при помощи утилиты `rosservice`. Так можно вызвать сервис `/get_telemetry` из командной строки:

```bash
rosservice call /get_telemetry "{frame_id: ''}"
```

Больше примеров использования сервисов для автономных полетов квадрокоптера Клевер можно посмотреть в [документации ноды simple_offboard](simple_offboard.md).

Работа на нескольких машинах
---

Основная статья: http://wiki.ros.org/ROS/Tutorials/MultipleMachines.

Преимуществом использования ROS является возможность распределения нод на несколько машин в сети. Например, ноду, осуществляющую распознавание образом на изображении можно запустить на более мощном компьютере; ноду, управляющую коптером можно запустить непосредственно на Raspberry Pi, подключенном к полетному контроллеру и т. д.