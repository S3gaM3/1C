# Answer42 — MCP для форм 1С

Answer42 даёт агенту инструменты `start_session`, `open_form`, `click_button`, `ui_tree` и другие: открывать управляемые формы 1С, нажимать кнопки, заполнять поля.

Живые формы открываются только там, где установлена платформа **1С:Предприятие 8.3.27+** (у вас это Windows и база `C:\Users\user\Documents\1C\GroomingSalon`). На удалённом Linux-агенте без 1С пакет ставится, но `start_session` не запустит Конфигуратор.

Версия пакета: **answer42 0.5.3**.

---

## 1. Windows (Cursor Desktop рядом с 1С)

В PowerShell **от имени обычного пользователя**, не службы:

```powershell
py -3 -m pip install --user pipx
py -3 -m pipx ensurepath
```

Закройте и откройте терминал, затем:

```powershell
pipx install "answer42[screenshot,windows-window-control]"
answer42 --version
```

Должно быть `answer42 0.5.3`.

Создайте файл `%USERPROFILE%\.answer42-credentials.json` (права только у вас). Логин и пароль 1С сюда, **не** в чат и не в git:

```json
{
  "version": 2,
  "accounts": {
    "default": {
      "entries": [
        {
          "url": "C:\\Users\\user\\Documents\\1C\\GroomingSalon",
          "username": "",
          "password": "",
          "title": "grooming",
          "aliases": ["grooming-salon", "ГрумингСалон"]
        }
      ]
    }
  }
}
```

Если в базе уже есть пользователь, впишите его имя и пароль вместо пустых строк.

В репозитории уже есть `.cursor/mcp.json`. Перезапустите Cursor. В **Customize → MCP** должен появиться сервер `answer42`.

Проверка в чате агента (после перезапуска Cursor):

1. `credentials_check` с `base_url` `grooming` — запись найдена.
2. `start_session` с `base_url` `grooming` — откроется тонкий клиент тестирования.
3. `active_window` — видно окно 1С.
4. `stop_session` — клиент закрывается.

Не передавайте `username`/`password` в вызов `start_session`, если они уже в credentials-файле.

---

## 2. Что уже сделано на этом Linux-агенте

- пакет `answer42[screenshot,linux-window-control]` 0.5.3 (pipx + venv);
- `xvfb` для headless-дисплея;
- файл `~/.answer42-credentials.json` (вне git, режим `0600`);
- MCP: `~/.cursor/mcp.json` и `.cursor/mcp.json`;
- `answer42 install-skills` — в 0.5.3 это no-op, навыки встроены в описания MCP-инструментов.

Платформы 1С в `/opt/1cv8` нет, поэтому `start_session` здесь завершится ошибкой поиска `1cv8c` и `ibcmd`. Чтобы этот облачный агент открывал ваши формы, нужен **self-hosted worker** Cursor на том же Windows-ПК, где 1С.

---

## 3. Вызовы без секретов

```text
credentials_check(base_url="grooming")
start_session(base_url="grooming", idle_timeout_minutes=60)
active_window()
stop_session()
```

`base_url` может быть заголовком `grooming`, путём файловой базы `/F` или HTTP-публикацией. Для учебной файловой базы достаточно пути к каталогу с `1Cv8.1CD`.
