1. [Создайте](../../vpc/operations/security-group-create.md) группу безопасности агента `agent-sg`.
1. [Добавьте правила](../../vpc/operations/security-group-update.md#add-rule):
    1. Правило для исходящего HTTPS-трафика к публичному API {{ load-testing-full-name }}:
        * диапазон портов: `443`;
        * протокол: `TCP`;
        * тип источника: `CIDR`;
        * назначение: `0.0.0.0/0`.

        Это позволит подключить агент к сервису {{ load-testing-name }}, чтобы управлять тестами из интерфейса и получать результаты тестирования.
    1. Правило для входящего SSH-трафика:
        * диапазон портов: `22`;
        * протокол: `TCP`;
        * тип источника: `CIDR`;
        * назначение: `0.0.0.0/0`.

        Это позволит подключаться к агенту по протоколу SSH и управлять тестами из консоли или собирать отладочную информацию.
    1. Правило для исходящего трафика при подаче нагрузки к цели тестирования:
        * диапазон портов: `{{ port-any }}`;
        * протокол: `Any`;
        * тип источника: `Группа безопасности`;
        * назначение: `Из списка`. Укажите группу безопасности, в которой находится нужная цель тестирования.

        Создайте такое правило для каждой цели тестирования с уникальной группой безопасности.