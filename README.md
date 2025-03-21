### Детальный разбор функций игры:

---

#### **1. `print_f(text: str, delimeter: bool = True)`**
**Назначение**:  
Выводит текст на экран и сохраняет его в файл `output_script.txt`.  
**Параметры**:
- `text` — текст для отображения.
- `delimeter` — если `True`, добавляет разделитель `-----` перед текстом (по умолчанию включен).

**Логика**:
1. Проверяет, нужно ли добавить разделитель.
2. Печатает текст в консоль.
3. Открывает файл в режиме добавления (`"a+"`) и записывает текст с переносом строки.

**Пример**:
```python
print_f("Вы в стартовой зоне", delimeter=True)
```
**Вывод в файле**:
```
-----
Вы в стартовой зоне
```

---

#### **2. `input_f(descr: str) -> str`**
**Назначение**:  
Запрашивает ввод у игрока и сохраняет ответ в файл.  
**Параметры**:
- `descr` — текст-подсказка для ввода (например, `"Введите имя: "`).

**Логика**:
1. Использует `input()`, чтобы получить ответ.
2. Записывает ответ в файл `output_script.txt`.
3. Возвращает введенную строку.

**Пример**:
```python
name = input_f("Ваше имя: ")
```
**Запись в файле**:
```
Ваше имя: Гарри
```

---

#### **3. `select_handler(script: dict) -> tuple[str, bool]`**
**Назначение**:  
Обрабатывает выбор из вариантов ответов с проверкой правильности.  
**Параметры**:
- `script` — словарь с ключами:
  - `"question"` — текст вопроса.
  - `"answers"` — список вариантов ответов.
  - `"correct_idx"` — правильный индекс ответа (например, `"0"`).
  - `"retries"` — количество попыток.

**Логика**:
1. Выводит вопрос и варианты ответов.
2. Запускает цикл попыток (от `retries` до 1).
3. При неверном ответе сообщает об оставшихся попытках.
4. Возвращает кортеж: `(ответ_игрока, результат_проверки)`.

**Пример**:
```python
script = {
    "question": "Где купить палочку?",
    "answers": ["0. Лавка", "1. Рынок"],
    "correct_idx": "0",
    "retries": 2
}
answer, is_correct = select_handler(script)
```

---

#### **4. `select_scene_handler(script: dict) -> str`**
**Назначение**:  
Обрабатывает выбор сцены без проверки правильности (для навигации).  
**Параметры**:
- `script` — словарь с ключами:
  - `"question"` — текст вопроса.
  - `"answers"` — список вариантов ответов.

**Логика**:
1. Выводит вопрос и варианты.
2. Возвращает введенный игроком номер ответа (например, `"1"`).

**Пример**:
```python
choice = select_scene_handler({
    "question": "Куда идти?",
    "answers": ["0. Лес", "1. Пещера"]
})
```

---

#### **5. `input_handler(script: dict) -> tuple[str, bool]`**
**Назначение**:  
Обрабатывает текстовый ввод с проверкой по списку допустимых ответов.  
**Параметры**:
- `script` — словарь с ключами:
  - `"question"` — текст вопроса.
  - `"answers"` — список правильных ответов (например, `["снитч"]`).
  - `"retries"` — количество попыток.

**Логика**:
1. Приводит ответ игрока к нижнему регистру (`answer.lower()`).
2. Проверяет, есть ли ответ в списке `answers`.
3. Если `answers` не указан, любой ответ считается правильным.

**Пример**:
```python
script = {
    "question": "Что нужно поймать в квиддиче?",
    "answers": ["снитч", "золотой снитч"],
    "retries": 2
}
answer, is_correct = input_handler(script)
```

---

#### **6. Движки планет (например, `magic_planet_engine()`)**
**Назначение**:  
Управляют сюжетом на конкретной планете.  
**Логика**:
- Переключают сцены через ключ `"next"` в словаре планеты.
- Используют обработчики (`select_handler`, `input_handler`) для вопросов.
- Отслеживают параметры:
  - **Здоровье (HP)** — уменьшается при ошибках (`magic_planet_engine`).
  - **Доверие** — влияет на финал (`tele_planet_engine`).
  - **Инвентарь** — собираемые предметы (`house_planet_engine`).

**Пример**:
```python
def magic_planet_engine():
    hp = 5
    current_scene = "start"
    while current_scene != "end":
        scene = MAGIC_PLANET[current_scene]
        print_f(scene["text"])
        # Обработка вопросов...
        if not result:
            hp -= 2.5
    return "completed" if hp > 0 else "fail"
```

---

#### **7. `main()`**
**Назначение**:  
Запускает игру, управляет выбором планет и финалом.  
**Логика**:
1. Создает словарь `planets` с данными о доступных планетах.
2. В цикле запрашивает выбор планеты через `select_scene_handler`.
3. Запускает движок выбранной планеты.
4. После прохождения всех планет предлагает выбрать финальную для проживания.

**Пример**:
```python
planets = {
    "magic": {
        "name": "Планета Волшебства",
        "engine": magic_planet_engine,
        "status": ""
    },
    # ...
}
```

---

#### **8. Словари планет (например, `MAGIC_PLANET`)**
**Структура**:
- **Ключи** — названия сцен (например, `"start"`, `"main"`).
- **Значения** — словари с данными сцены:
  - `"text"` — описание локации.
  - `"next"` — следующая сцена по умолчанию.
  - `"questions"` — список вопросов (если есть).

**Пример**:
```python
MAGIC_PLANET = {
    "start": {
        "next": "main",
        "text": "Вы прибыли на планету Волшебства...",
        "questions": [
            {
                "type": "input",
                "question": "Во сколько лет пришло письмо?",
                "answers": ["11"],
                "retries": 2
            }
        ]
    }
}
```

---

### Взаимодействие функций:
1. **Игрок выбирает планету** → `main()` вызывает `select_scene_handler`.
2. **Движок планеты** загружает сцены из словаря → использует `print_f`, `input_f`, обработчики вопросов.
3. **Результаты ответов** влияют на параметры (HP, доверие, инвентарь).
4. **После всех планет** → `main()` выводит итоги и предлагает выбрать финальную локацию.

### Ключевые особенности:
- **Сценарии отделены от кода**. Изменяя словари планет, можно добавлять новые диалоги и сцены.
- **Логика ветвления** определяется через ключ `"next"` и выбор игрока.
- **Все действия игрока** сохраняются в файл для анализа.

Этот подход позволяет легко модифицировать игру, не меняя основной код — достаточно редактировать словари сценариев.
