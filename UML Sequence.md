**Оформление заказа в Яндекс Такси**</br>
***Задача:***</br>
Пользователь мобильного приложения Яндекс Такси хочет заказать поездку и оплатить её через банковскую карту. При этом у пользователя есть возможность применить промокод для скидки на поездку. Промокод действует только на определённые типы поездок и ограничен по времени. Данные о промокодах хранятся в системе Яндекс Такси, включая информацию о самом промокоде, на какие поездки он распространяется, размер скидки, срок действия и статус (активен, использован, истёк). Процесс оплаты выполняется через платёжный шлюз, который списывает средства с карты клиента.</br>


***Последовательность шагов:***</br>
1.	Пользователь выбирает начальный и конечный пункты поездки.
2.	Приложение рассчитывает стоимость поездки и отображает её пользователю.
3.	Пользователь видит поле для ввода промокода и вводит промокод, если он доступен.
4.	Пользователь проверяет валидность промокода, нажав кнопку «Применить промокод».
5.	Если промокод действителен для выбранного маршрута и находится в состоянии «активен», стоимость пересчитывается с учётом скидки. В противном случае отображается ошибка с возможной причиной (промокод недействителен, истёк или не применяется к данному маршруту).
6.	Пользователь соглашается с пересчитанной или исходной стоимостью и нажимает кнопку «Заказать такси».
7.	В систему Яндекс Такси отправляется запрос с данными заказа и итоговой стоимостью. Если применён промокод, его статус меняется на «использован».
8.	Система отправляет запрос в платёжный шлюз с деталями заказа и суммой списания.
9.	Платёжный шлюз проводит проверку данных карты в системе «Антифрод».
10.	В случае успешной проверки шлюз списывает средства с карты, а статус оплаты передаётся в систему Яндекс Такси.
11.	Пользователь получает уведомление в приложении о статусе заказа и оплате (успешно или неуспешно).
12.	В случае успешной оплаты автомобиль приезжает в назначенное место для начала поездки.</br>


***Решение:***</br>
Разработаны диаграммы последовательности с учётом всех вышеперечисленных конструкций. Отразите основные шаги оформления заказа, проверки промокода и оплаты, применяя UML-конструкции для альтернативных сценариев, асинхронных вызовов и группировки участников.

***Для удобства восприятия диграммы разделены на 4 процесса:***
1) Заказ такси;
2) Выполнение поездки;
3) Оплата заказа;
4) Завершение поездки.

**UML Sequence "Заказ такси"** </br>
![Заказ такси](https://github.com/user-attachments/assets/5a0dbbb0-3c7a-4e7f-a05f-b8e0658c0e01)
```
title Заказ такси 

actor Пользователь as U

participant "Приложение" as Ui
participant "Сервер" as Server
participant "Приложение для водителя" as Driver

U -> Ui: Выбирает начальный и конечный пункты поездки
activate U
activate Ui
U -> Ui: Выбирает класс такси
deactivate U
Ui -> Server: Отправка данных маршрута
activate Server
Server-> Server: Расчет стоимости поездки
Server--> Ui: Получение стоимости поездки
deactivate Server
Ui-> U: Отображение рассчитаной стоимости поездки
deactivate Ui
alt Пользователь оформляет заказ
    U -> Ui: Соглашается с ценой и нажимает "Заказать такси"
    activate Ui
    Ui-> Server: Поиск автомобиля
    activate Server
    note over Server: Поиск машины может занять до 3-х минут
    activate Driver
    Server-> Driver: Новая заявка
    alt Заявка принята
    Driver-> Driver: Водитель принял заявку
    Driver -->Server: Результат поиска такси
    deactivate Driver
    Server--> Ui: Отображение результата поиска автомобиля
    deactivate Server
    Ui-> U: Отображение информации о найденом автомобиле и времени прибытия
    
    else Заявка отклонена
    activate Driver
    Driver-> Driver: Ни один водитель не принял заявку
    activate Server
    Driver -->Server: Результат поиска такси
    deactivate Driver
    Server--> Ui: Отображение результата поиска автомобиля
    deactivate Server
    Ui-> U: Сообщение: Свободных машин рядом нет, продолжаем поиск
    deactivate Ui
    end
    
else Пользователь изменяет параметры заказа
    U -> Ui: Изменение начального/конечого пунктов поездки и/или класса такси
    activate Server
    activate Ui
    Ui -> Server: Отправка данных маршрут
    Server-> Server: Расчет стоимости поездки
    Server--> Ui: Получение стоимости поездки
    deactivate Server
    Ui-> U: Отображение рассчитаной стоимости поездки
    deactivate Ui

else Пользователь отменяет заказ
    U -> Ui: Нажимает кнопку "Отмена"
    activate Ui
    activate Server
    Ui -> Server: Актуализация статуса заказа  
    Server--> Ui: Заказ отменен
    deactivate Server
    Ui-> U: Заказ успешно отменен
    deactivate Ui
end
```

**UML Sequence "Выполнение поездки"** </br>
![Выполнение поездки](https://github.com/user-attachments/assets/79fa0465-6852-4d36-99a9-2af352c97943)
```
title Выполнение поездки

actor Пользователь as U

participant "Приложение" as Ui
participant "Сервер" as Server
participant "Приложение для водителя" as Driver

Driver -> Server: Такси прибыло
activate Driver
activate Server
Server -> Server: Старт бесплатного ожидания
Server -> Driver: Отображение времени ожидания
deactivate Driver
activate Ui
Server -> Ui: Отображение времени ожидания  и местоположения авто
Ui-> U: Уведомление: Такси вас ожидает

Driver -> Server: Начало поездки
activate Driver
Server -> Ui: Отображение маршрута
deactivate Ui
Server--> Driver: Отображение нового маршрута и рассчитаной стоимости поездки
deactivate Server
deactivate Driver

alt Пользователь изменяет маршрут
    U -> Ui: Изменение конечого пункта и/или добавление остановки
    activate Ui
    Ui -> Server: Отправка данных маршрута
    deactivate Ui
    activate Server

    else Водитель изменяет маршрут
    Driver -> Server: Изменение конечого пункта и/или добавление остановки
    end

    Server-> Server: Расчет нового маршрута
    Server-> Server: Перерасчет стоимости поездки
    Server--> Driver: Отображение нового маршрута и рассчитаной стоимости поездки
    Server--> Ui: Отображение нового маршрута и рассчитаной стоимости поездки
    activate Ui
    deactivate Server
    Ui-> U: Уведомление об изменение маршрута и стоимости поездки
    deactivate Ui
```

**UML Sequence "Оплата поездки"** </br>
![Оплата](https://github.com/user-attachments/assets/8bc8c84d-8aae-42e6-acf7-776c7f7d4f43)
```
title Оплата заказа


actor Пользователь as U
participant "Приложение Яндекс Такси" as Taxi
participant "Сервер" as Server
participant "Платежный шлюз" as Gateway
participant "Антифрод-система" as Antifraud


U -> Taxi: Отмечает наличие промокода
activate Taxi
activate U
Taxi -> U: Открывается поле ввода промокода
U -> Taxi: Вводит промокод и нажимает "Применить промокод"
Taxi-> Server: Запрос на проверку валидности промокода
deactivate Taxi
deactivate U
activate Server
Server-> Server: Проверка валидности промокода


alt Промокод невалиден
    Server--> Taxi: Отображение результата проверки
    activate Taxi
    deactivate Server
    Taxi -> U: Вывод сообщения: "промокод не применен" и описания ошибки
    deactivate Taxi
    
else Промокод валиден
    activate Server
    Server-> Server: Перерасчет стоимости поездки с учетом промокода
    Server--> Taxi: Отображение результата проверки
    activate Taxi
    deactivate Server
    Taxi -> U: Вывод сообщения: "промокод успешно применен" и пересчитанной стоимости поездки
    activate U
    U -> Taxi: Соглашается с ценой и нажимает "Заказать такси"
    deactivate U
    Taxi -> Server: Изменение статуса промокода на "использован"
    activate Gateway
    Taxi-> Gateway: Отправляет запрос на оплату
    deactivate Taxi
    activate Antifraud
    Gateway -> Antifraud: Проверка заказа на мошенничество
    Antifraud --> Gateway: Результаты проверки
    deactivate Antifraud
    Gateway -> Gateway: Списание средств с карты
    alt Оплата неуспешна
        Gateway -> Taxi: Статус неуспешной оплаты
        activate Taxi
        Taxi-> U: Платеж неуспешен
        deactivate Taxi
    else Оплата успешна
        Gateway -> Taxi: Статус успешной оплаты
        activate Taxi
        deactivate Gateway
        Taxi-> U: Платеж успешен
        deactivate Taxi
    end
end
```

**UML Sequence "Завершение поездки"** </br>
![Завершение поездки](https://github.com/user-attachments/assets/d2052e0d-acf3-422a-9da3-af8de655b3d5)
```
title Завершение поездки

actor Пользователь as U

participant "Приложение" as Ui
participant "Сервер" as Server
participant "Приложение для водителя" as Driver
participant "Платежный шлюз" as Gateway
participant "Антифрод-система" as Antifraud


Driver -> Server: Прибытие в точку назначения
activate Server
Server -> Server: Расчет итоговой стоимости поездки
Server -> Gateway: Запрос на оплату
deactivate Server

activate Gateway
    Gateway -> Antifraud: Проверка заказа на мошенничество
    activate Antifraud
    Antifraud --> Gateway: Результаты проверки
    deactivate Antifraud
    Gateway -> Gateway: Списание средств с карты
    alt Оплата неуспешна
        Gateway -> Server: Статус неуспешной оплаты
        activate Server
        Server -> Ui: Статус заказа: ожидает оплаты
        activate Ui
        Server -> Driver: Платеж неуспешен
        deactivate Server
        Ui-> U: Платеж неуспешен
        deactivate Ui
    else Оплата успешна

        Gateway -> Server: Статус успешной оплаты
        activate Server
        deactivate Gateway
        Server -> Ui: Статус заказа: завершен
        activate Ui
        Server -> Driver: Поездка оплачена
        deactivate Server
        Ui-> U: Платеж успешен
        Ui -> U: Спасибо за поездку!
        deactivate Ui
    end
```


Все диаграммы разработаны с использованием: https://www.websequencediagrams.com/app

