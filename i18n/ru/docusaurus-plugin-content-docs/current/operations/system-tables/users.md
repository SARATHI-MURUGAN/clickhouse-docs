---
description: 'Системная таблица, содержащая список учетных записей пользователей, настроенных на сервере.'
keywords: ['системная таблица', 'пользователи']
slug: /operations/system-tables/users
title: 'system.users'
---


# system.users

Содержит список [учетных записей пользователей](../../guides/sre/user-management/index.md#user-account-management), настроенных на сервере.

Колонки:
- `name` ([String](../../sql-reference/data-types/string.md)) — Имя пользователя.

- `id` ([UUID](../../sql-reference/data-types/uuid.md)) — Идентификатор пользователя.

- `storage` ([String](../../sql-reference/data-types/string.md)) — Путь к хранилищу пользователей. Настроен в параметре `access_control_path`.

- `auth_type` ([Enum8](../../sql-reference/data-types/enum.md)('no_password' = 0, 'plaintext_password' = 1, 'sha256_password' = 2, 'double_sha1_password' = 3, 'ldap' = 4, 'kerberos' = 5, 'ssl_certificate' = 6, 'bcrypt_password' = 7)) — Показывает тип аутентификации. Существует несколько способов идентификации пользователей: без пароля, с обычным текстовым паролем, с паролем, закодированным с помощью [SHA256](https://en.wikipedia.org/wiki/SHA-2), с паролем, закодированным с помощью [double SHA-1](https://en.wikipedia.org/wiki/SHA-1), или с паролем, закодированным с помощью [bcrypt](https://en.wikipedia.org/wiki/Bcrypt).

- `auth_params` ([String](../../sql-reference/data-types/string.md)) — Параметры аутентификации в формате JSON в зависимости от `auth_type`.

- `host_ip` ([Array](../../sql-reference/data-types/array.md)([String](../../sql-reference/data-types/string.md))) — IP-адреса хостов, которым разрешено подключаться к серверу ClickHouse.

- `host_names` ([Array](../../sql-reference/data-types/array.md)([String](../../sql-reference/data-types/string.md))) — Имена хостов, которым разрешено подключаться к серверу ClickHouse.

- `host_names_regexp` ([Array](../../sql-reference/data-types/array.md)([String](../../sql-reference/data-types/string.md))) — Регулярное выражение для имен хостов, которым разрешено подключаться к серверу ClickHouse.

- `host_names_like` ([Array](../../sql-reference/data-types/array.md)([String](../../sql-reference/data-types/string.md))) — Имена хостов, которым разрешено подключаться к серверу ClickHouse, установленные с использованием предиката LIKE.

- `default_roles_all` ([UInt8](/sql-reference/data-types/int-uint#integer-ranges)) — Показывает, что все предоставленные роли установлены для пользователя по умолчанию.

- `default_roles_list` ([Array](../../sql-reference/data-types/array.md)([String](../../sql-reference/data-types/string.md))) — Список предоставленных ролей по умолчанию.

- `default_roles_except` ([Array](../../sql-reference/data-types/array.md)([String](../../sql-reference/data-types/string.md))) — Все предоставленные роли, установленные по умолчанию, за исключением перечисленных.

## See Also {#see-also}

- [SHOW USERS](/sql-reference/statements/show#show-users)
