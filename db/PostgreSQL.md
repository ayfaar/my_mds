# PostgreSQL

## Зайти под пользователем **postgres**

```bash
sudo -u postgres psql
```

## PASSWORD

### Password file

Файл **.pgpass**, находящийся в директории пользователя может содержать пароли, требуемые для подключения к БД. В Windows файл паролей называется: **%APPDATA\postgresql\pgpass.conf**. Вместо файла по умолчанию можно указать расположения файли импользуя параметр *passfile* при подключении к БД или используя переменную окружения **PGPASSFILE**.
Файл должен содержать строки следующего вида:

```properties
hostname:port:database:username:password
```

В файле можно писать строки коментарии, должны начинаться с символа *#*. Каждое из 4-х полей может быть заменено на символ '*', что значить любое совпадениею. Если в любом из полей требуется ввести символ ':' или '\', его следует экранировать символом '\'.
В Unix системах доступ к файлу паролей должен быть разрешен только его хозяину, что можно достичь при помощи команды **chmod 0600 ~/.pgpass**

### Изменить пароль пользователя(роли)

```sql
 \password postgres
```

где postgres - имя роли(пользователя)

## Версия БД

```bash
psql --verion
```

## Database list(Список БД)

```sql
\l
```

```sql
SELECT datname FROM pg_database;
```

## ROLEs

Роли экввивалентны пользователям или группам пользователейй, в зависимости от настроек роли

### Вывести все роли

```sql
\du+
```

```sql
SELECT rolname from pg_roles;
```

### Удалить роль

```sql
DROP ROLE name;
```

### Добавить роль pg_monitor

(право читать и выполнять различные представления и функции для мониторинга. Эта роль включена в роли pg_read_all_settings, pg_read_all_stats и pg_stat_scan_tables;) к роли zabbix

```sql
GRANT pg_monitor TO zabbix;
```

### Удалить роль pg_monitor из роли zabbix

```sql
REVOKE pg_monitor FROM zabbix;
```

### Изменение пароля роли(Change role password)

```sql
ALTER ROLE <roleName> WITH PASSWORD '<new_password>';
```
