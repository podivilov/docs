{% list tabs %}

- Консоль управления

   1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором находится реестр.
   1. Выберите сервис **{{ iot-short-name }}**.
   1. Выберите реестр.
   1. На странице **Обзор** перейдите к разделу **Сертификаты**.

- CLI

  {% include [cli-install](../cli-install.md) %}

  {% include [default-catalogue](../default-catalogue.md) %}

  Получите список сертификатов реестра:

  ```
  yc iot registry certificate list --registry-name my-registry
  ```

  Результат:

  ```
  +------------------------------------------+---------------------+
  |               FINGERPRINT                |     CREATED AT      |
  +------------------------------------------+---------------------+
  | 0f511ea32139178edf73afb953a9cc398f33adf1 | 2019-05-29 16:46:23 |
  | 589ce1605019eeff7bb0992f290be0cd708ecc6c | 2019-05-29 16:40:48 |
  +------------------------------------------+---------------------+
  ```

- API

  Получить список сертификатов реестра можно с помощью метода API [listCertificates](../../iot-core/api-ref/Registry/listCertificates.md).

{% endlist %}