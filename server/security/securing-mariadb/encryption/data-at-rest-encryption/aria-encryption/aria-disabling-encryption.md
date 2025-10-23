# Aria Disabling Encryption

The process involved in safely disabling data-at-rest encryption for your Aria tables is very similar to that of enabling encryption. To disable, you need to set the relevant system variables and then rebuild each table into an unencrypted state.

Don't remove the [Encryption Key Management](../../../securing-mariadb-encryption/encryption-data-at-rest-encryption/key-management-and-encryption-plugins/encryption-key-management.md) plugin from your configuration file until you have unencrypted all tables in your database. MariaDB cannot read encrypted tables without the relevant encryption key.

## Disabling Encryption on User-created Tables

With tables that the user creates, you can disable encryption by setting the [aria\_encrypt\_tables](../../../../../reference/storage-engines/aria/aria-system-variables.md#aria_encrypt_tables) system variable to `OFF`. Once this is set, MariaDB no longer encrypts new tables created with the Aria storage engine.

```sql
SET GLOBAL aria_encrypt_tables = OFF;
```

Unlike [InnoDB](../innodb-encryption/), Aria does not currently use background encryption threads. Before removing the [Encryption Key Management](../../../securing-mariadb-encryption/encryption-data-at-rest-encryption/key-management-and-encryption-plugins/encryption-key-management.md) plugin from the configuration file, you first need to manually rebuild each table to an unencrypted state.

To find the encrypted tables, query the Information Schema, filtering the [TABLES](../../../../../reference/system-tables/information-schema/information-schema-tables/information-schema-tables-table.md) table for those that use the Aria storage engine and the `PAGE` [ROW\_FORMAT](../../../../../reference/sql-statements/data-definition/create/create-table.md#row_format).

```sql
SELECT TABLE_SCHEMA, TABLE_NAME
FROM information_schema.TABLES
WHERE ENGINE = 'Aria'
  AND ROW_FORMAT = 'PAGE'
  AND TABLE_SCHEMA != 'information_schema';
```

Each table in the result-set was potentially written to disk in an encrypted state. Before removing the configuration for the encryption keys, you need to rebuild each of these to an unencrypted state. This can be done with an [ALTER TABLE](../../../../../reference/sql-statements/data-definition/alter/alter-table/) statement.

```sql
ALTER TABLE test.aria_table ENGINE = Aria ROW_FORMAT = PAGE;
```

Once all of the Aria tables are rebuilt, they're safely unencrypted.

## Disabling Encryption for Internal On-disk Temporary Tables

MariaDB routinely creates internal temporary tables. When these temporary tables are written to disk and the [aria\_used\_for\_temp\_tables](../../../../../server-usage/storage-engines/aria/aria-system-variables.md#aria_used_for_temp_tables) system variable is set to `ON`, MariaDB uses the Aria storage engine.

To decrypt these tables, set the [encrypt\_tmp\_disk\_tables](../../../../../ha-and-performance/optimization-and-tuning/system-variables/server-system-variables.md#encrypt_tmp_disk_tables) to `OFF`. Once set, all internal temporary tables that are created from that point on are written unencrypted to disk.

<sub>_This page is licensed: CC BY-SA / Gnu FDL_</sub>

{% @marketo/form formId="4316" %}
