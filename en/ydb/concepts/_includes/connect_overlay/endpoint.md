---
sourcePath: en/ydb/overlay/concepts/_includes/connect_overlay/endpoint.md
---
To find out the value of the DB endpoint parameter in {{ yandex-cloud }}:

{% list tabs %}

- Management console

  1. Go to the [management console]({{ link-console-main }}).
  1. Select the folder and go to **{{ ydb-full-name }}**.
  1. Select the desired database in the list of databases.
  1. On the **Overview** tab, go to the **YDB endpoint** section.

      The endpoint value is in the **Endpoint** line. To get the value of the DB connection parameter, add the protocol name at the beginning.

- CLI

  Run the [CLI {{ yandex-cloud }}](../../../../cli/index.yaml) command:

  ```bash
  yc ydb database list
  ```

  Result:

  ```bash
  +----------------------+------+-------------+--------------------------------------------------------------------------------------------------------------+---------------------+---------+
  |          ID          | NAME | DESCRIPTION |                                                   ENDPOINT                                                   |     CREATED AT      | STATUS  |
  +----------------------+------+-------------+--------------------------------------------------------------------------------------------------------------+---------------------+---------+
  | etn01lrprvnlnhv8v5kj | test |             | grpcs://ydb.serverless.yandexcloud.net:2135/?database=/ru-central1/b1g4ej5ju4rf5kelpk4b/etn01lrprvnlnhv8v5kj | 2021-11-26 12:06:55 | RUNNING |
  +----------------------+------+-------------+--------------------------------------------------------------------------------------------------------------+---------------------+---------+
  ```

  The parameter value is in the `ENDPOINT` column: `grpcs://ydb.serverless.yandexcloud.net:2135`.

{% endlist %}
