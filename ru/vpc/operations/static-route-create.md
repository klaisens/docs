---
title: "Как создать статический маршрут. {{ vpc-short-name }}"
description: "Статический маршрут по умолчанию (0.0.0.0/0) действует на машины с публичными IP-адресами. Если вам требуется создать NAT-инстанс, создайте его в отдельной подсети. Чтобы создать таблицу маршрутизации и добавить в нее статические маршруты откройте раздел {{ vpc-name }} в каталоге, где требуется создать статический маршрут. Выберите сеть, в которой требуется создать таблицу маршрутизации. Нажмите Создать таблицу маршрутизации."
---

# Создать статический маршрут


{% note info %}

Статический маршрут по умолчанию (`0.0.0.0/0`) действует на машины с публичными IP-адресами. Если вам требуется [создать NAT-инстанс](../../tutorials/routing/nat-instance.md), создайте его в отдельной подсети.

{% endnote %}

{% list tabs %}

- Консоль управления

  Чтобы создать таблицу маршрутизации и добавить в нее [статические маршруты](../concepts/static-routes.md): 
  1. В [консоли управления]({{ link-console-main }}) перейдите в каталог, где требуется создать статический маршрут.
  1. В списке сервисов выберите **{{ vpc-name }}**.
  1. На панели слева выберите ![image](../../_assets/vpc/route-tables.svg) **Таблицы маршрутизации**.
  1. Нажмите кнопку **Создать**.
  1. Задайте имя таблицы маршрутизации.

     {% include [name-format](../../_includes/name-format.md) %}

  1. (опционально) Добавьте описание таблицы маршрутизации.
  1. Выберите сеть, в которой требуется создать таблицу маршрутизации.
  1. Нажмите кнопку **Добавить маршрут**.
  1. В открывшемся окне введите префикс подсети назначения в нотации CIDR.
  1. Укажите **next hop** — IP-адрес из [разрешенных диапазонов](../concepts/network.md#subnet).
  1. Нажмите кнопку **Добавить**.
  1. Нажмите кнопку **Создать таблицу маршрутизации**.

  Чтобы использовать статические маршруты, необходимо привязать таблицу маршрутизации к подсети:

  1. На панели слева выберите ![image](../../_assets/vpc/subnets.svg) **Подсети**.
  1. В строке нужной подсети нажмите кнопку ![image](../../_assets/options.svg).
  1. В открывшемся меню выберите пункт **Привязать таблицу маршрутизации**.
  1. В открывшемся окне выберите созданную таблицу в списке.
  1. Нажмите кнопку **Привязать**.

- CLI

  Чтобы создать таблицу маршрутизации и добавить в нее [статические маршруты](../concepts/static-routes.md): 
  1. Посмотрите описание команды CLI для создания таблиц маршрутизации:

     ```
     yc vpc route-table create --help
     ```

  1. Получите идентификаторы облачных сетей в вашем облаке:

     ```
     yc vpc network list
	 ```
	 Результат:
	 ```
     +----------------------+-----------------+
     |          ID          |      NAME       |
     +----------------------+-----------------+
     | enp34hbpj8dqebn3621l | yc-auto-subnet  |
     | enp846vf5fus0nt3lu83 | routes-test     |
     +----------------------+-----------------+
     ```

  1. Создайте таблицу маршрутизации в одной из сетей. Используйте следующие флаги:

     * `name` — имя таблицы маршрутизации.
     * `network-id` — идентификатор сети, в которой будет создана таблицы.
     * `route` — настройки маршрута, включают два параметра:
        * `destination` — префикс подсети назначения в нотации CIDR.
        * `next-hop` — внутренний IP-адрес виртуальной машины из [разрешенных диапазонов](../concepts/network.md#subnet), через которую будет направляться трафик.

     ```
     yc vpc route-table create --name=test-route-table --network-id=enp846vf5fus0nt3lu83 --route destination=0.0.0.0/0,next-hop=192.168.1.5
	 ```
	 Результат:
	 ```
     ...done
     id: enpsi6b08q2vfdmppsnb
     folder_id: b1gqs1teo2q2a4vnmi2t
     created_at: "2019-06-24T09:57:54Z"
     name: test-route-table
     network_id: enp846vf5fus0nt3lu83
     static_routes:
     - destination_prefix: 0.0.0.0/0
       next_hop_address: 192.168.1.5
     ```

  Чтобы использовать статические маршруты, необходимо привязать таблицу маршрутизации к подсети:

  1. Получите список подсетей в вашем облаке:

     ```
     yc vpc subnet list
	 ```
	 Результат:
	 ```
     +----------------------+------------------+----------------------+----------------+---------------+------------------+
     |          ID          |       NAME       |      NETWORK ID      | ROUTE TABLE ID |     ZONE      |      RANGE       |
     +----------------------+------------------+----------------------+----------------+---------------+------------------+
     | b0cf2b0u7nhl75gp1c9t | subnet-1         | enp846vf5fus0nt3lu83 |                | {{ region-id }}-c | [192.168.0.0/24] |
     | e2llnffvbakqu18hr170 | subnet-2         | enp846vf5fus0nt3lu83 |                | {{ region-id }}-b | [192.168.1.0/24] |
     +----------------------+------------------+----------------------+----------------+---------------+------------------+
     ```

  1. Привяжите таблицу маршрутизации к одной из подсетей:

     ```
     yc vpc subnet update b0cf2b0u7nhl75gp1c9t --route-table-id enp1sdveovdpdhaao5dq
	 ```
	 Результат:
	 ```
     ..done
     id: b0cf2b0u7nhl75gp1c9t
     folder_id: b1gqs1teo2q2a4vnmi2t
     created_at: "2019-03-12T13:27:22Z"
     name: subnet-1
     network_id: enp846vf5fus0nt3lu83
     zone_id: {{ region-id }}-c
     v4_cidr_blocks:
     - 192.168.0.0/24
     route_table_id: enp1sdveovdpdhaao5dq
     ```

{% endlist %}
