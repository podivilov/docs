#### Квоты {#load-balancer-quotas}
Вид ограничения | Значение
----- | -----
Количество балансировщиков в одном облаке | 2
Количество целевых групп в одном облаке | 2

#### Лимиты {#load-balancer-limits}
Вид ограничения | Значение
----- | -----
Количество ресурсов в целевой группе | 254
Количество портов обработчика | 10
Количество проверок состояния на подключенную целевую группу | 1
Протокол проверок состояния | TCP, HTTP

#### Прочие ограничения {#load-balancer-other-restrictions}
В одной целевой группе могут находиться целевые ресурсы только из одной облачной сети. 

В пределах одной зоны доступности в целевую группу могут входить ресурсы, подключенные к одной подсети.

После добавления в целевую группу ресурс не будет доступен напрямую по целевому порту.

Можно создать балансировщик без обработчика.

Проверки состояния передаются из диапазона адресов `198.18.235.0/24`.

При подключении ресурсов к балансировщику учитывайте [лимит](../vpc/concepts/limits.md#limits) на максимальное количество одновременно установленных TCP/UDP-соединений для одной виртуальной машины.