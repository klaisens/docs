---
sourcePath: en/ydb/ydb-docs-core/en/core/reference/ydb-cli/commands/_includes/discovery-list.md
---
# List of endpoints

The `discovery list` information command lets you get a list of [endpoints](../../../../concepts/connect.md#endpoint) for the {{ ydb-short-name }} cluster that you can establish connections with in order to access your database:

```bash
{{ ydb-cli }} [connection options] discovery list
```

{% include [conn_options_ref.md](conn_options_ref.md) %}

The output rows in the response contain the following information:

1. Endpoint, including protocol and port
2. Availability zone (in square brackets)
3. The `#` character is used for the list of {{ ydb-short-name }} services available on this endpoint

An endpoint discovery request to the {{ ydb-short-name }} cluster is executed in the {{ ydb-short-name }} SDK at driver initialization so that you can use the `discovery list` CLI command to localize connection issues.

## Example

```bash
$ ydb --profile db1 discovery list
grpcs://vm-etn01q5-ysor.etn01q5k.ydb.mdb.yandexcloud.net:2135 [sas] #table_service #scripting #discovery #rate_limiter #locking #kesus
grpcs://vm-etn01q5-arum.etn01ftr.ydb.mdb.yandexcloud.net:2135 [vla] #table_service #scripting #discovery #rate_limiter #locking #kesus
grpcs://vm-etn01q5beftr.ydb.mdb.yandexcloud.net:2135 [myt] #table_service #scripting #discovery #rate_limiter #locking #kesus
```

