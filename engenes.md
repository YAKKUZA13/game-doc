### Построчный разбор движков планет

---

#### **1. Движок планеты Волшебства (`magic_planet_engine`)**
```python
def magic_planet_engine() -> str:
    '''Движок планеты Волшебства'''
    hp = 5  # Здоровье игрока (изначально 5 единиц)
    name = "start"  # Начальная сцена
    while name != "end":  # Цикл, пока не достигнута сцена "end"
        scene = MAGIC_PLANET[name]  # Загружаем текущую сцену из словаря
        print_f(scene["text"])  # Выводим описание сцены
        if not scene.get("questions"):  # Если в сцене нет вопросов
            name = scene["next"]  # Переходим к следующей сцене
            continue  # Пропускаем оставшийся код цикла
        # Обработка вопросов в сцене:
        for question in scene["questions"]:
            match question["type"]:  # Определяем тип вопроса
                case "input":  # Текстовый ввод
                    resp = input_handler(question)  # Используем input_handler
                case "select":  # Выбор из вариантов
                    resp = select_handler(question)  # Используем select_handler
            if not resp[1]:  # Если ответ неверный (resp[1] = False)
                hp -= 2.5  # Уменьшаем здоровье на 2.5
            if hp <= 0:  # Если здоровье опустилось до 0
                return "fail"  # Игрок проиграл
        name = scene["next"]  # Переход к следующей сцене
    return "completed"  # Все сцены пройдены успешно
```

**Ключевые моменты**:
- **Здоровье (hp)**: Начинается с 5. За каждую ошибку отнимается 2.5.
- **Сцены**: Переход между ними происходит через ключ `"next"`.
- **Вопросы**: Обрабатываются в зависимости от типа (`input` или `select`).
- **Поражение**: Если `hp <= 0`, игра возвращает `"fail"`.

---

#### **2. Движок планеты Свинки Пеппы (`peppa_planet_engine`)**
```python
def peppa_planet_engine() -> str:
    '''Движок планеты Пеппы'''
    name = "start"  # Начальная сцена
    all_correct = True  # Флаг: все ответы верны
    while name != "end":  # Цикл до финальной сцены
        scene = PEPPA_PLANET[name]  # Загружаем текущую сцену
        print_f(scene["text"])  # Выводим текст сцены
        if not scene.get("questions"):  # Если вопросов нет
            name = scene["next"]  # Переход к следующей сцене
            continue
        # Проверка всех вопросов:
        for question in scene["questions"]:
            resp = input_handler(question)  # Текстовый ввод
            if not resp[1] and name == "peppa":  # Если ответ неверный
                all_correct = False  # Отмечаем ошибку
                break  # Прерываем цикл вопросов
        # Определение финальной сцены:
        scene["next"] = "final_good" if all_correct else "final_bad"
        name = scene["next"]  # Переход к финалу
    return "completed" if all_correct else "fail"  # Результат
```

**Ключевые моменты**:
- **Проверка всех ответов**: Если хотя бы один ответ неверный, `all_correct = False`.
- **Финальные сцены**: 
  - `final_good` — если все ответы верны.
  - `final_bad` — если есть ошибки.
- **Только текстовые вопросы**: Все вопросы типа `input`.

---

#### **3. Движок планеты Телепузиков (`tele_planet_engine`)**
```python
def tele_planet_engine() -> str:
    '''Движок планеты Телепузиков'''
    name = "start"  # Начальная сцена
    status = ""  # Статус завершения
    trust = 10  # Уровень доверия (изначально 10)
    while name != "end":  # Цикл до финала
        scene = TELE_PLANET[name]  # Загрузка сцены
        if name == "pre-final":  # Специальная сцена перед финалом
            name = "final_good" if trust > 5 else "final_bad"  # Определение исхода
            continue
        if name.startswith("final_"):  # Финал
            status = scene["status"]  # Сохраняем статус
        print_f(scene["text"])  # Вывод текста сцены
        if not scene.get("questions"):  # Если вопросов нет
            name = scene["next"]  # Переход
            continue
        # Обработка вопросов:
        for question in scene["questions"]:
            resp = select_scene_handler(question)  # Выбор сцены
            trust += question["influence"][resp]  # Изменение доверия
            name = question["next"][resp]  # Переход к выбранной сцене
    return status  # "completed" или "fail"
```

**Ключевые моменты**:
- **Доверие (trust)**: Начинается с 10. Уменьшается при негативных выборах.
- **Сцена `pre-final`**: Автоматически определяет финал на основе доверия.
- **Влияние выбора**: Каждый ответ изменяет `trust` через `influence`.
- **Финал**: Если `trust > 5` — успех, иначе — провал.

---

#### **4. Движок планеты Дома (`house_planet_engine`)**
```python
def house_planet_engine() -> str:
    '''Движок планеты Дома'''
    name = "start"  # Начальная сцена
    status = ""  # Статус завершения
    inventory = set()  # Инвентарь (собранные предметы)
    while name != "end":  # Цикл до финала
        scene = HOUSE_PLANET[name]  # Загрузка сцены
        if name == "pre-final":  # Проверка перед финалом
            print(inventory)  # Для отладки
            name = "final_good" if len(inventory) >= 2 else "final_bad"  # Нужно 2 предмета
            continue
        if name.startswith("final_"):  # Финал
            status = scene["status"]  # Сохраняем статус
        print_f(scene["text"])  # Вывод текста
        if not scene.get("questions"):  # Если вопросов нет
            name = scene["next"]  # Переход
            continue
        # Обработка вопросов:
        for question in scene["questions"]:
            resp = select_scene_handler(question)  # Выбор сцены
            if question["influence"][resp] != "":  # Если есть предмет
                inventory.add(question["influence"][resp])  # Добавляем в инвентарь
            name = question["next"][resp]  # Переход
    return status  # "completed" или "fail"
```

**Ключевые моменты**:
- **Инвентарь (inventory)**: Собирается через выборы с `influence`.
- **Условие победы**: Нужно собрать минимум 2 предмета.
- **Предметы**: Например, "Осколок" или "Шарик".
- **Финал**: 
  - `final_good` — если собрано 2+ предмета.
  - `final_bad` — если меньше.

---

### Итоговая логика для всех планет:
| **Планета**       | **Ключевой параметр** | **Условие победы**              |
|--------------------|------------------------|----------------------------------|
| Волшебства         | Здоровье (HP)          | HP > 0 после всех вопросов      |
| Свинки Пеппы       | Все ответы верны       | Ни одной ошибки                 |
| Телепузиков        | Доверие (Trust)        | Trust > 5                       |
| Дома               | Инвентарь (Items)      | Собрано 2+ предмета             |

