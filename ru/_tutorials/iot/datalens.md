# Мониторинг состояния географически распределенных устройств

В этом сценарии вы настроите мониторинг состояния устройств (например, вендинговых аппаратов), подключенных к сервису {{ iot-full-name }} и расположенных в разных точках города. Датчики будут эмулированы с помощью сервиса {{ sf-full-name }}. Если у вас есть подключенные датчики, используйте их. Вы можете наблюдать за состоянием автоматов на карте и графиках сервиса {{ datalens-full-name }}.

Исходный код этого сценария доступен на [GitHub](https://github.com/yandex-cloud/examples/tree/master/iot/Scenarios).

Чтобы настроить мониторинг показаний датчиков в серверной комнате:
1. [Подготовьте облако к работе](#before-you-begin).
1. [Необходимые платные ресурсы](#paid-resources).
1. [Создайте необходимые ресурсы {{ iot-full-name }}](#resources-step).
    1. [Создайте реестр](#registry-step).
    1. [Создайте устройства](#device-step).
1. [Создайте эмулятор устройства в {{ sf-full-name }}](#emulator-step).
    1. [Создайте функцию эмуляции отправки данных с устройств](#emulation-function).
    1. [Протестируйте функцию эмуляции отправки данных](#test-emulation-function).
    1. [Создайте триггер вызова функции эмуляции один раз в минуту](#minute-trigger).
1. [Создайте кластер в {{ mpg-full-name }}](#postgresql-step).
1. [Создайте функцию обработки данных в {{ sf-full-name }}](#processing-function-step).
    1. [Создайте функцию обработки принимаемых данных](#processing-function).
    1. [Протестируйте функцию обработки данных](#test-processing-function).
    1. [Просмотрите результат обработки данных в {{ mpg-short-name }}](#processing-function-results).
    1. [Создайте триггер вызова функции обработки данных](#processing-function-trigger).
    1. [Просмотрите результат работы триггера в {{ mpg-short-name }}](#processing-function-trigger-results).
1. [Настройте мониторинг в {{ datalens-full-name }}](#configure-datalens)
    1. [Настройте подключение к {{ mpg-short-name }}](#connect-mpg)
    1. [Создайте датасет](#create-dataset)
    1. [Создайте чарт по показателям температуры и напряжения сети](#create-chart)
    1. [Создайте чарт с картой](#create-chart-map)
    1. [Создайте дашборд](#create-dashboard)
  
Если созданные ресурсы вам больше не нужны, [удалите их](#cleanup).

## Подготовьте облако к работе {#before-you-begin}

{% include [before-you-begin](../_tutorials_includes/before-you-begin.md) %}

Если у вас еще нет интерфейса командной строки {{ yandex-cloud }}, [установите и инициализируйте его](../../cli/quickstart.md#install).


### Необходимые платные ресурсы {#paid-resources}

В стоимость входят:
* плата за количество сообщений сервиса {{ iot-full-name }} (см. [тарифы](../../iot-core/pricing.md));
* плата за количество вызовов функции сервиса {{ sf-full-name }} (см. [тарифы](../../functions/pricing.md));
* плата за вычислительные ресурсы и хранилище кластера в сервисе {{ mpg-full-name }} (см. [тарифы](../../managed-postgresql/pricing.md)).

## Создайте необходимые ресурсы {{ iot-short-name }} {#resources-step}

Для работы с сервисом {{ iot-short-name }} вам потребуется создать [реестр](../../iot-core/concepts/index.md#registry) и [устройства](../../iot-core/concepts/index.md#device), которые будут обмениваться данными и командами.

### Создайте реестр и настройте авторизацию по логину и паролю {#registry-step}

Чтобы создать реестр:
1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором вы выполняете сценарий.
1. Выберите сервис **{{ iot-short-name }}**.
1. Нажмите кнопку **Создать реестр**.
1. В поле **Имя** введите имя реестра. Например, `my-registry`.
1. В поле **Пароль** задайте пароль доступа к реестру.

    Пароль должен быть длиной не менее 14 символов, должен содержать строчные буквы, заглавные буквы и цифры.  
    Для создания пароля можно воспользоваться [генератором паролей](https://passwordsgenerator.net/).
1. Сохраните пароль локально или запомните его. Сервис не показывает пароли после создания.
1. (опционально) В поле **Описание** добавьте дополнительную информацию о реестре.
1. Нажмите кнопку **Создать**.

Вы также можете использовать авторизацию с помощью сертификатов. Подробнее [об авторизации в {{ iot-short-name }}](../../iot-core/concepts/authorization.md).

### Создайте устройство и настройте авторизацию по логину и паролю {#device-step}

Создайте три устройства: `my-device-1`, `my-device-2` и `my-device-3`.

Чтобы создать устройство:  
1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором вы выполняете сценарий.
1. Выберите сервис **{{ iot-short-name }}**.
1. Выберите реестр, созданный на предыдущем шаге.
1. В левой части окна выберите раздел **Устройства**.
1. Нажмите кнопку **Добавить устройство**.
1. В поле **Имя** введите имя устройства. Например, `my-device-1`.
1. В поле **Пароль** задайте пароль доступа к устройству.
    
    Для создания пароля можно воспользоваться [генератором паролей](https://passwordsgenerator.net/).  
    Пароль должен быть длиной не менее 14 символов, должен содержать строчные буквы, заглавные буквы и цифры.   
1. Сохраните пароль локально или запомните его. Сервис не показывает пароли после создания.
1. (опционально) В поле **Описание** добавьте дополнительную информацию об устройстве.
1. (опционально) Добавьте [алиасы](../../iot-core/concepts/topic/usage.md#aliases):
    1. Нажмите кнопку **Добавить алиас**.
    1. Нажмите кнопку **Изменить**.
    1. Заполните поля: введите алиас (например `events`) и тип топика после `$devices/<deviceID>` (например `events`).  
   
        Вы сможете использовать алиас `events` вместо топика `$devices/<deviceID>/events`.
    1. Повторите действия для каждого добавляемого алиаса.
1. Нажмите кнопку **Создать**.
1. Повторите действия для каждого устройства, которое необходимо создать. 

Вы также можете использовать авторизацию с помощью сертификатов. Подробнее [об авторизации в {{ iot-short-name }}](../../iot-core/concepts/authorization.md).

## Создайте эмулятор устройств в {{ sf-name }} {#emulator-step}

Эмулятор отправит данные с устройств в кластер {{ mpg-full-name }}.

Вам потребуется:
* создать и протестировать [функцию](../../functions/concepts/function.md) эмуляции отправки данных с датчиков каждого устройства;
* создать [триггер](../../functions/concepts/trigger/index.md) вызова функции эмуляции один раз в минуту.

### Создайте функцию эмуляции отправки данных с устройства {#emulation_function}

Чтобы создать функцию:
1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором вы выполняете сценарий.
1. Выберите сервис **{{ sf-name }}**.
1. В левой части окна выберите раздел **Функции**.
1. Нажмите кнопку **Создать функцию**.
1. В поле **Имя** введите имя функции. Например, `my-device-emulator-function`.
1. (опционально) В поле **Описание** добавьте дополнительную информацию о функции.
1. Нажмите кнопку **Создать**.
1. В открывшемся окне **Редактор** в списке **Среда выполнения** выберите `nodejs12`.
1. Выберите **Способ**: **Редактор кода**.
1. В левой части окна **Редактор кода** нажмите кнопку **Создать файл**.
1. В открывшемся окне **Новый файл** ведите имя файла `device-emulator.js`.
1. Нажмите кнопку **Создать**.
1. Выберите созданный файл в левой части окна **Редактор кода**.
1. В правой части окна **Редактор кода** вставьте код функции с [GitHub](https://github.com/yandex-cloud/examples/blob/master/iot/Scenarios/DashboardForGeoDistributedDevices/device-emulator.js).  
1. В поле **Точка входа** окна **Редактор** введите `device-emulator.handler`.
1. В поле **Таймаут, с** введите `10`.
1. В поле **Память** оставьте значение `128 МБ`.
1. Создайте сервисный аккаунт, от имени которого функция отправит данные в {{ iot-short-name }}:
    1. Нажмите кнопку **Создать аккаунт**.
    1. В открывшемся окне **Создание сервисного аккаунта** в поле **Имя** введите имя аккаунта. Например, `my-emulator-function-service-account`.
    1. Добавьте роли получения списка устройств и записи в ресурсы `viewer` и `iot.devices.writer`:
        1. Нажмите на значок ![image](../../_assets/plus-sign.svg).
        1. Выберите роль в списке.
        1. Нажмите кнопку **Создать**.
1. Настройте параметр **Переменные окружения** каждого датчика серверной комнаты:
    1. Нажмите кнопку **Добавить переменную окружения**.   
    1. Заполните поля **Ключ** и **Значение** переменных окружения:  
      
        Ключ | Описание | Значение
        :----- | :----- | :-----
        `CASH_DRAWER_SENSOR_VALUE` | Процент заполненности отсека купюр. | `67.89`
        `COIN_DRAWER_SENSOR_VALUE` | Процент заполненности отсека монет. | `67.89`
        `TEMPERATURE_SENSOR_VALUE` | Базовое значение температуры в отсеке выдачи товара. | `10.34`
        `POWER_SENSOR_VALUE` | Базовое значение напряжения сети. | `24.12`
        `SERVICE_DOOR_SENSOR_VALUE` | Показания датчика открытия сервисной дверцы. | `False`
        `ITEM1_SENSOR_VALUE` | Остаток товара типа 1. | `50.65`
        `ITEM2_SENSOR_VALUE` | Остаток товара типа 2. | `80.97`
        `ITEM3_SENSOR_VALUE` | Остаток товара типа 3. | `30.33`
        `ITEM4_SENSOR_VALUE` | Остаток товара типа 4. | `15.15`
        `REGISTRY_ID` | Идентификатор реестра, который вы создали | См. консоль управления <br>сервиса **{{ iot-short-name }}**
1. В правой верхней части окна нажмите кнопку **Создать версию**.

### Протестируйте функцию эмуляции {#test-emulation-function}

Чтобы протестировать функцию:
1. (опционально) Для получения подробной информации с датчиков, подпишите реестр на топик устройства {{ iot-full-name }}, где`$devices/<deviceID>/events/` — топик устройства, `<deviceID>` — ID устройства в сервисе:

    {% list tabs %}
    
    - CLI  
        
        Выполните команду: 
        
        ```
        $ yc iot mqtt subscribe \
              --username <ID реестра> \
              --password <пароль реестра> \
              --topic '$devices/<ID устройства>/events' \
              --qos 1
        ```
        Где:
        * `--username` и `--password` — параметры для авторизации с помощью логина и пароля.
        * `--topic` — топик устройства для отправки данных.
        * `--message` — текст сообщения.
        * `--qos` — уровень качества обслуживания (QoS).
    
    {% endlist %}

    Подробнее о [подписке на топики устройства в {{ iot-full-name }}](../../iot-core/operations/subscribe#one-device).

1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором вы выполняете сценарий.
1. Выберите сервис **{{ sf-name }}**.
1. В левой части окна выберите раздел **Тестирование**.
1. В списке **Тег версии** выберите `$latest` — последнюю созданную функцию.
1. Нажмите кнопку **Запустить тест**.

При успешном выполнении функции в поле **Состояние функции** отобразится статус **Выполнена** и в поле **Ответ функции** результат:

```
{
"statusCode" : 200
}
```

Если вы подписались на топик устройства {{ iot-short-name }}, вы получите JSON вида:

```json
{
"DeviceId":"arealt9f3jh445it1laq",
"TimeStamp":"2020-07-21T22:38:12Z",
"Values":[
    {"Type":"Bool","Name":"Service door sensor","Value":"false"},
    {"Type":"Float","Name":"Power Voltage","Value":"24.12"},
    {"Type":"Float","Name":"Temperature","Value":"10.34"},
    {"Type":"Float","Name":"Cash drawer fullness","Value":"67.89"},
    {"Type":"Float","Name":"Coin drawer fullness","Value":"67.89"},
    {"Items":[
       {"Type":"Float", "Id":"1","Name":"Item 1","Fullness":"50.65"},
       {"Type":"Float", "Id":"2","Name":"Item 2","Fullness":"80.97"},
       {"Type":"Float", "Id":"3","Name":"Item 3","Fullness":"30.33"},
       {"Type":"Float", "Id":"4","Name":"Item 4","Fullness":"15.15"},
    ]}
    ]
}
```

Подробнее об [MQTT-топиках в сервисе {{ iot-short-name }}](../../iot-core/concepts/topic/index.md).

### Создайте триггер вызова функции один раз в минуту {#minute-trigger}

Чтобы создать триггер:
1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором вы выполняете сценарий.
1. Выберите сервис **{{ sf-name }}**.
1. Выберите раздел **Триггеры**.
1. Нажмите кнопку **Создать триггер**.
1. В поле **Имя** введите имя триггера. Например, `my-emulator-function-trigger`.
1. (опционально) В поле **Описание** добавьте дополнительную информацию о триггере.
1. Выберите **Тип**: **Таймер**.
1. В поле **Cron-выражение** введите `* * * * ? *` (вызов один раз в минуту).
1. В блоке **Настройки функции** введите ранее заданные параметры функции:
    * **Функция**: `my-device-emulator-function`.
    * **Тег версии функции**: `$latest`.
    * **Сервисный аккаунт**: `my-emulator-function-service-account`.
1. (опционально) Настройте параметры блоков **Настройки повторных запросов** и **Настройки Dead Letter Queue**. Они обеспечивают сохранность данных.
    * **Настройки повторных запросов** позволяют повторно вызывать функцию, если текущий вызов функции завершается с ошибкой.
    * **Настройки Dead Letter Queue** позволяют перенаправлять сообщения, которые не смогли обработать получатели в обычных очередях. 
        В качестве DLQ очереди вы можете настроить стандартную очередь сообщений. Если вы еще не создавали очередь сообщений, [создайте ее в сервисе {{ message-queue-full-name }}](../../message-queue/operations/message-queue-new-queue.md).
1. Нажмите кнопку **Создать триггер**.

## Создайте кластер в {{ mpg-full-name }} {#postgresql-step}

В примере используются минимальные значения параметров хоста. Для реальных задач рекомендуется выбирать хосты с гарантированной долей vCPU 100%.

Чтобы создать кластер:
1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором вы выполняете сценарий.
1. Выберите сервис **{{ mpg-full-name }}**.
1. Выберите раздел **Кластеры**.
1. Нажмите кнопку **Создать кластер**.
1. В блоке **Базовые параметры** в поле **Имя кластера** введите, например, `my-pg-database`.
1. (опционально) В поле **Описание** добавьте дополнительную информацию о кластере.
1. В списке **Окружение** выберите `PRODUCTION`.
1. В списке **Версия** выберите `12`.
1. В блоке **Класс хоста** в списке **Платформа** выберите `Intel Ice Lake`.
1. Выберите тип виртуальной машины: на вкладке **burstable** тип **b2.nano**.
1. В блоке **Размер хранилища** выберите вкладку **network-hdd**.
1. Укажите размер хранилища 10 ГБ.
1. В блоке **База данных**:
    * В поле **Имя БД** введите `db1`.
    * В поле **Имя пользователя** введите `user1`.
    * В поле **Пароль** задайте пароль доступа к базе.  
        
        Не забудьте сохранить пароль, он вам понадобится.
1. Значения полей **Локаль сортировки** и **Локаль набора символов** оставьте без изменений. По умолчанию установлено значение `C`.
1. В блоке **Сеть** в списке выберите `default`.
1. В блоке **Хосты** настройте подключение к вашей базе данных через публичный IP-адрес (это необходимо для доступа к базе из {{ sf-full-name }}).
    1. Справа в правой части строки зоны доступа нажмите на значок ![image](../../_assets/pencil.svg).
    1. В открывшемся окне выберите **Зону доступности**. Например `{{ region-id }}-a`.
    1. Выберите **Подсеть**. Например `default-{{ region-id }}-a`.
    1. Включите **Публичный доступ**.
    1. Нажмите кнопку **Сохранить**.
1. Включите **Доступ из DataLens**.
1. Нажмите кнопку **Создать кластер**.

Кластер будет создаваться несколько минут. В результате отобразится окно с данными кластера. 

## Создайте функцию обработки данных в {{ sf-full-name }} {#processing-function-step}

Функция обрабатывает данные от устройств.

### Создайте функцию обработки принимаемых данных {#processing_function}

Чтобы создать функцию:
1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором вы выполняете сценарий.
1. Выберите сервис **{{ sf-name }}**.
1. В левой части окна выберите раздел **Функции**.
1. Нажмите кнопку **Создать функцию**.
1. В поле **Имя** введите имя функции. Например, `my-db-function`.
1. (опционально) В поле **Описание** добавьте дополнительную информацию о функции.
1. Нажмите кнопку **Создать**.
1. В открывшемся окне **Редактор** в списке **Среда выполнения** выберите `python37`.
1. Выберите **Способ**: нажмите на вкладку **Редактор кода**.
1. В левой части окна **Редактор кода** нажмите кнопку **Создать файл**.
1. В открывшемся окне **Новый файл** ведите имя файла `myfunction.py`.
1. Нажмите кнопку **Создать**.
1. В левой части окна **Редактор кода** выберите созданный файл.
1. В правой части окна вставьте код функции с [GitHub](https://github.com/yandex-cloud/examples/blob/master/iot/Scenarios/DashboardForGeoDistributedDevices/myfunction.py).
1. В поле **Точка входа** окна **Редактор** введите `myfunction.msgHandler`.
1. В поле **Таймаут, с** введите `10`.
1. В поле **Память** оставьте значение `128 МБ`.
1. Создайте сервисный аккаунт, от имени которого функция обработает данные, полученные от устройства:
    1. Нажмите кнопку **Создать аккаунт**.
    1. В открывшемся окне **Создание сервисного аккаунта** в поле **Имя** введите имя аккаунта. Например, `my-db-function-service-account`.
    1. Добавьте роли вызова функции и изменения ресурсов `serverless.functions.invoker` и `editor`:
        1. Нажмите на значок ![image](../../_assets/plus-sign.svg).
        1. Выберите роль в списке.
        1. Нажмите кнопку **Создать**.
1. Настройте параметр **Переменные окружения**. Параметры подключения к базе данных вы можете посмотреть в сервисе {{ mpg-full-name }}.
    1. Нажмите кнопку **Добавить переменную окружения**.   
    1. Заполните поля **Ключ** и **Значение** переменных окружения:
    
        Ключ | Описание | Значение
        :----- | :----- | :-----
        `VERBOSE_LOG` | Включение и отключение записи данных. | `True`
        `DB_HOSTNAME` | Имя хоста в {{ mpg-full-name }}. | См. в консоли управления<br>сервиса {{ mpg-full-name }} 
        `DB_PORT` | Порт подключения к кластеру в {{ mpg-full-name }}. | `6432`
        `DB_NAME` | Имя кластера в {{ mpg-full-name }}. | `db1`
        `DB_USER` | Имя пользователя для подключения к кластеру в {{ mpg-full-name }}. | `user1`
        `DB_PASSWORD` | Пароль подключения к кластеру в {{ mpg-full-name }}. | Пароль, который вы задали в {{ mpg-full-name }}.
1. В правой верхней части окна нажмите кнопку **Создать версию**.

### Протестируйте функцию обработки данных {#test-processing-function}

Чтобы протестировать функцию:
1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором вы выполняете сценарий.
1. Выберите сервис **{{ sf-name }}**.
1. В левой части окна выберите раздел **Тестирование**.
1. В списке **Тег версии** выберите `$latest` — последнюю созданную функцию.
1. В поле **Входные данные** вставьте данные:

    ```json
    {
        "messages": [
            {
                "event_metadata": {
                    "event_id": "160d239876d9714800",
                    "event_type": "yandex.cloud.events.iot.IoTMessage",
                    "created_at": "2020-05-08T19:16:21.267616072Z",
                    "folder_id": "b112345678910"
                },
                "details": {
                    "registry_id": "are1234567890",
                    "device_id": "are0987654321",
                    "mqtt_topic": "$devices/are0987654321/events",
                    "payload": "ewogICAgICAgICAgICAiRGV2aWNlSWQiOiJhcmU1NzBrZTA1N29pcjg1bDlmciIsCiAgICAgICAgICAgICJUaW1lU3RhbXAiOiIyMDIwLTA2LTExVDExOjA3OjIwWiIsCiAgICAgICAgICAgICJWYWx1ZXMiOlsKICAgICAgICAgICAgICAgIHsiVHlwZSI6IkJvb2wiLCJOYW1lIjoiU2VydmljZSBkb29yIHNlbnNvciIsIlZhbHVlIjoiRmFsc2UifSwKICAgICAgICAgICAgICAgIHsiVHlwZSI6IkZsb2F0IiwiTmFtZSI6IlBvd2VyIFZvbHRhZ2UiLCJWYWx1ZSI6IjI1LjA2In0sCiAgICAgICAgICAgICAgICB7IlR5cGUiOiJGbG9hdCIsIk5hbWUiOiJUZW1wZXJhdHVyZSIsIlZhbHVlIjoiMTEuMjEifSwKICAgICAgICAgICAgICAgIHsiVHlwZSI6IkZsb2F0IiwiTmFtZSI6IkNhc2ggZHJhd2VyIGZ1bGxuZXNzIiwiVmFsdWUiOiI2Ny44OSJ9LAogICAgICAgICAgICAgICAgeyJJdGVtcyI6WwogICAgICAgICAgICAgICAgICAgIHsiVHlwZSI6IkZsb2F0IiwgIklkIjoiMSIsIk5hbWUiOiJJdGVtIDEiLCJGdWxsbmVzcyI6IjUwLjY1In0sCiAgICAgICAgICAgICAgICAgICAgeyJUeXBlIjoiRmxvYXQiLCAiSWQiOiIyIiwiTmFtZSI6Ikl0ZW0gMiIsIkZ1bGxuZXNzIjoiODAuOTcifSwKICAgICAgICAgICAgICAgICAgICB7IlR5cGUiOiJGbG9hdCIsICJJZCI6IjMiLCJOYW1lIjoiSXRlbSAzIiwiRnVsbG5lc3MiOiIzMC4zMyJ9LAogICAgICAgICAgICAgICAgICAgIHsiVHlwZSI6IkZsb2F0IiwgIklkIjoiNCIsIk5hbWUiOiJJdGVtIDQiLCJGdWxsbmVzcyI6IjE1LjE1In0KICAgICAgICAgICAgICAgIF19CiAgICAgICAgICAgICAgICBdCiAgICAgICAgICAgIH0="
                }
            }
        ]
    }
    ```
1. Нажмите кнопку **Запустить тест**.

При успешном выполнении функции в поле **Состояние функции** отобразится статус **Выполнена**, а в поле **Ответ функции** результат:

```json
{
"statusCode" : 200 ,
    "headers" : {
        "Content-Type" : "text/plain"
    },
"isBase64Encoded" : false
}
```

### Просмотрите результат обработки данных в {{ mpg-short-name }} {#processing-function-results}

Чтобы просмотреть результат обработки данных:
1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором вы выполняете сценарий.
1. Выберите сервис **{{ mpg-full-name }}**.
1. Выберите раздел **SQL**.

В правой части окна отобразится таблица с результатами обработки данных.

### Создайте триггер вызова функции обработки данных {#processing-function-trigger}

Триггер вызовет функцию, когда в [топике устройства](../../iot-core/concepts/topic/devices-topic.md) появятся сообщения.

Чтобы создать триггер:
1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором вы выполняете сценарий.
1. Выберите сервис **{{ sf-name }}**.
1. Выберите раздел **Триггеры**.
1. Нажмите кнопку **Создать триггер**.
1. В поле **Имя** введите имя триггера. Например, `my-db-func-trigger`.
1. (опционально) В поле **Описание** добавьте дополнительную информацию о триггере.
1. Выберите **Тип**: **{{ iot-name }}**.
1. В блоке **Настройки сообщений {{ iot-name }}** введите ранее заданные параметры реестра и устройства:
    * **Реестр**: `my-registry`.
    * **Устройство**: `Любое устройство`.
    * **MQTT-топик**: `$devices/#`.
    
        Подробнее об [MQTT-топиках в сервисе {{ iot-short-name }}](../../iot-core/concepts/topic/index.md).    
1. В блоке **Настройки функции** введите ранее заданные параметры функции:
    * **Функция**: `my-database-function`.
    * **Тег версии функции**: `$latest`.
    * **Сервисный аккаунт**: `my-db-func-trigger-service-account`.
1. (опционально) Настройте параметры блоков **Настройки повторных запросов** и **Настройки Dead Letter Queue**. Они обеспечивают сохранность данных.
    * **Настройки повторных запросов** позволяют повторно вызывать функцию, если текущий вызов функции завершается с ошибкой.
    * **Настройки Dead Letter Queue** позволяют перенаправлять сообщения, которые не смогли обработать получатели в обычных очередях. 
        В качестве DLQ очереди вы можете настроить стандартную очередь сообщений. Если вы еще не создавали очередь сообщений, [создайте ее в сервисе {{ message-queue-full-name }}](../../message-queue/operations/message-queue-new-queue.md).
1. Нажмите кнопку **Создать триггер**.

### Просмотрите результат работы триггера в {{ mpg-short-name }} {#processing-function-trigger-results}

Через некоторое время после создания триггера вы можете проверить, как он работает.

Чтобы просмотреть результат работы триггера:
1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором вы выполняете сценарий.
1. Выберите сервис **{{ mpg-full-name }}**.
1. Выберите раздел **SQL**.

В правой части окна отобразится обновленная таблица с большим количеством данных.

## Настройте мониторинг в {{ datalens-full-name }}{#configure-datalens}

### Настройте подключение к {{ mpg-short-name }}{#connect-mpg}

Чтобы настроить подключение {{ datalens-full-name }} к {{ mpg-short-name }}:
1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором вы выполняете сценарий.
1. Выберите сервис **{{ datalens-full-name }}**.
1. Нажмите кнопку **Создать подключение**.
1. Выберите коннектор **PostgreSQL**.

    При подключении к внешнему источнику данных (который не является ресурсом {{ yandex-cloud }}) предоставьте доступ к источнику [для диапазонов IP-адресов сервиса {{ datalens-name }}](../../datalens/concepts/connection.md#changing-connection-ranges).
1. Задайте имя подключения: `MyPGConnection`.
1. Выберите **Подключение**: **Выбрать в облаке**.
1. В списке **Кластер** выберите `my-pg-database`.
1. В списке **Имя хоста** выберите хост, который вы создали в сервисе {{ mpg-full-name }}.
1. В поле **Порт** введите `6432`.
1. В списке **Имя базы данных** выберите `db1`.
1. В списке **Имя пользователя** выберите `user1`.
1. В поле **Пароль** введите пароль, который вы задали для доступа к кластеру в сервисе {{ mpg-full-name }}.
1. Нажмите кнопку **Создать**. Подключение появится в списке.

### Создайте датасет {#create-dataset}

Чтобы создать датасет:
1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором вы выполняете сценарий.
1. Выберите сервис **{{ datalens-full-name }}**.
1. Нажмите кнопку **Создать датасет**.
1. В левой части экрана нажмите **![image](../../_assets/plus-sign.svg) Добавить**.
1. Выберите подключение `MyPGConnection`.
1. В левой части окна выберите таблицы `public.iot_events` и `public.iot_position`, перетащите их вправо.
1. Нажмите кнопку **Сохранить**.
1. В открывшемся окне задайте имя датасета `My-pg-dataset` и нажмите **Создать**.

Датасет появится в списке.

### Создайте чарт по показателям температуры и напряжения сети {#create-chart}

Чтобы создать чарт по показателям температуры и напряжения сети:
1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором вы выполняете сценарий.
1. Выберите сервис **{{ datalens-full-name }}**.
1. Нажмите кнопку **Создать чарт**.
1. В разделе **Датасет** выберите датасет `My-pg-dataset`, который вы создали ранее.
1. Выберите тип чарта **Линейная диаграмма**.
1. Из блока **Измерения** в левой части окна перетащите измерения в блок **Линейная диаграмма**:
    * `event_datetime` в секцию **X**;
        
        В нижней части графика по оси X отобразится временная шкала.
    * `temperature` и `power_voltage` в секцию **Y**.
        
        По оси Y в виде графика отобразятся значения температуры и напряжения сети.
1. Нажмите кнопку **Сохранить**.
1. В открывшемся окне задайте имя чарта или используйте сгенерированное имя `My-pg-dataset — Линейная диаграмма` и нажмите **Сохранить**.

### Создайте чарт с картой {#create-chart-map}

Чтобы создать чарт с картой:
1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором вы выполняете сценарий.
1. Выберите сервис **{{ datalens-full-name }}**.
1. В левой части окна выберите раздел **Датасеты**.
1. В списке выберите датасет `My-pg-dataset`.
1. Выберите вкладку **Поля**.
1. В правой части окна нажмите кнопку **Добавить поле**.
1. В открывшемся окне в поле **Название поля** введите `Position`.
1. В поле **Формула** вставьте `GEOPOINT([latitude],[longitude])`.
1. Нажмите кнопку **Создать**.
1. Нажмите кнопку **Сохранить**.
1. В правом верхнем углу нажмите кнопку **Создать чарт**.
1. Выберите тип чарта **Карта**.
1. Из блока **Измерения** в левой части окна перетащите измерения в блок **Карта**:
    * `Position` в секцию **Геоточки**;
    * `item1_fullness`, `item2_fullness`, `item3_fullness`, `item4_fullness`, `cash_drawer` и `coin_drawer` в секцию **Тултипы**.

        В правой части окна отобразится масштабируемая карта, на которой вендинговые автоматы отмечены точками на карте, а тултипы  — строками легенды.
1. Нажмите кнопку **Сохранить**.
1. В открывшемся окне задайте имя чарта или используйте сгенерированное имя `My-pg-dataset — Карта` и нажмите **Сохранить**.

### Создайте дашборд {#create-dashboard}

Чтобы создать дашборд:
1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором вы выполняете сценарий.
1. Выберите сервис **{{ datalens-full-name }}**.
1. Нажмите кнопку **Создать дашборд**.
1. В открывшемся окне введите название дашборда `MyDash`.
1. Добавьте на дашборд чарты `My-pg-dataset — Карта` и `My-pg-dataset — Линейная диаграмма`, которые вы создали ранее:
    1. В верхней части окна в раскрывающемся списке **Добавить** выберите **Чарт**.
    1. В раскрывающемся списке **Чарт** выберите чарт `My-pg-dataset — Карта`.  
        
        Имя чарта подставится в поле **Название**.  
    1. (опционально) В поле **Описание** введите описание чарта.
    1. Нажмите кнопку **Добавить**.
    1. Повторите действия  — добавьте чарт `My-pg-dataset — Линейная диаграмма`.
1. Настройте селектор:
    1. В верхней части окна в раскрывающемся списке **Добавить** выберите **Селектор**.
    1. В поле **Название** введите **Устройство**.
    1. В списке **Датасет** выберите `My-pg-dataset`.
    1. В списке **Поле** выберите `device_id`.
    1. В списке **Значение по умолчанию** выберите идентификатор устройства, которое вы создали в сервисе {{ iot-short-name }}.
    1. Нажмите кнопку **Добавить**
1. Настройте связи:
    1. В верхней части окна нажмите кнопку **Связи**.
    1. В верхней части открывшегося окна в списке выберите чарт `My-pg-dataset — Карта`.
    1. В раскрывающемся списке **Вх.связь** выберите `Игнор`.  
        
        Для карты параметр **Связи** не действует.
    1. Нажмите кнопку **Сохранить**.
    1. В верхней части окна в списке выберите чарт `My-pg-dataset — Линейная диаграмма`.
    1. В раскрывающемся списке **Вх.связь** выберите `Вх.связь`.
    1. Нажмите кнопку **Сохранить**.
1. Нажмите кнопку **Сохранить**.

Подробнее [о дашбордах сервиса {{ datalens-full-name }}](../../datalens/concepts/dashboard.md)    