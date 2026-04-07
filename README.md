<div align="center">
<h1>RimWorld Mod Collector</h1>

Приложение на основе QT для преобразования коллекций модов RimWorld, взятых из Steam Workshop, в XML-файл, используемый RimPy.<br><br>
![Python](https://img.shields.io/badge/Python-3.10+-blue.svg)
![PyQt6](https://img.shields.io/badge/PyQt6-6.4+-green.svg)
![License](https://img.shields.io/badge/License-MIT-yellow.svg)
  
<img width="1920" height="1058" alt="image" src="https://github.com/user-attachments/assets/0e139c82-99cc-4ec7-8991-c6c9724c2958" />

</div>

# Описание
RimWorld Mod Collector — это инструмент для скачивания модификаций из коллекций Steam Workshop, а также извлечения и преобразования метаданных модов в нужный формат, требуемый RimPy — менеджера модификаций. Этот инструмент:
+ Загружает моды из заданной вами коллекции Steam Workshop при помощи [SteamCMD](https://developer.valvesoftware.com/wiki/SteamCMD)
+ После загрузки всех модов коллекции, алгоритм извлекает `packageId` из файла `About/About.xml` каждого мода
+ Полученные данные алгоритм использует для генерации XML-файла, используемого RimPy, а также кэширования этой информации для ускорения повторной обработки

### Особенности

- **Два режима работы**:
 
  > **Постоянный**: моды сохраняются в кэше
  > 
  > **Временный**: моды удаляются после обработки
- **Детальный вывод работы программы**
- **Асинхронная обработка**
- **Автосохранение настроек**

# Системные требования

- Python 3.10 или выше
- [SteamCMD](https://developer.valvesoftware.com/wiki/SteamCMD)

# Установка

**Клонируйте репозиторий:**
```bash
git clone https://github.com/Amorsev/rimworld-modbridge.git
cd rimworld-modbridge
```

**Создайте виртуальное окружение (настоятельно рекомендуется):**
```bash
python -m venv venv
```

**Активируйте созданное окружение**
```
# Windows
venv\Scripts\activate
# Linux
source venv/bin/activate
```

**Установите зависимости:**
```bash
pip install -r requirements.txt
```

**Скачайте и установите SteamCMD:**
   - [Инструкция для Windows](https://developer.valvesoftware.com/wiki/SteamCMD#Windows)
   - [Инструкция для Linux](https://developer.valvesoftware.com/wiki/SteamCMD#Linux)

**Запуск исполняемого файла** (убедитесь что вы находитесь в созданном вами окружении)

```bash
python main.py
```


# Использование

1. **Вставьте ссылку на коллекцию Steam Workshop в поле ввода**
   - Пример: `https://steamcommunity.com/sharedfiles/filedetails/?id=123456789`

2. **Укажите путь к исполняемому файлу SteamCMD**
   - Используйте кнопку "Обзор" для выбора файла

3. **Выберите папку для сохранения XML-файла**

4. **Введите имя файла** (без расширения)

5. **Выберите режим работы и соответствующую ему рабочую папку:**

- #### Режим 1 (Постоянный, с кэшем)
  - Моды скачиваются в указанную папку для постоянного хранения
  - Моды скачиваются только если папка с workshop ID отсутствует
  - После обработки моды остаются в каталоге
  - Идеально для повторного использования модов
  - Если папка не указана, используется папка steamcmd по умолчанию

- #### Режим 2 (Временный)
  - Моды скачиваются в папку для временного хранения
  - packageId извлекается, затем моды удаляются
  - Информация сохраняется в локальной базе данных
  - При повторной обработке данные берутся из кэша, а моды не скачиваются повторно
  - Экономит дисковое пространство
  - Если папка не указана, используется папка steamcmd по умолчаниюсо

6. **Нажмите "Запустить"**

# Структура проекта

```
rimworld_modbridge/
├── __init__.py          # Инициализация пакета
├── main.py              # Главный модуль с GUI
├── database.py          # Работа с SQLite базой данных
├── steam_handler.py     # Взаимодействие со Steam/steamcmd
├── xml_processor.py     # Обработка XML файлов
├── settings.py          # Управление настройками
├── styles.py            # Стили интерфейса
├── requirements.txt     # Зависимости Python
└── README.md            # Markdown файл проекта, который вы сейчас видите
```

# Архитектура проекта

### Диаграмма классов

```
┌─────────────────┐     ┌──────────────────┐
│   MainWindow    │────▶│   WorkerThread   │
│    (PyQt6)      │     │   (QThread)      │
└─────────────────┘     └────────┬─────────┘
                                 │
         ┌───────────────────────┼───────────────────────┐
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐     ┌──────────────────┐    ┌──────────────────┐
│   ModDatabase   │     │  SteamHandler    │    │  XmlProcessor    │
│    (SQLite)     │     │   (steamcmd)     │    │   (XML/RimPy)    │
└─────────────────┘     └──────────────────┘    └──────────────────┘
```

### Формат выходного XML

Генерируемый XML полностью совместим с RimPy и имеет следующий вид:

```xml
<?xml version="1.0" encoding="utf-8"?>
<ModList>
  <Name>MyModList</Name>
  <Version>1.0</Version>
  <Mods>
    <!-- Workshop ID: 123456789 | Mod Name -->
    <li>author.modname</li>
    <!-- Workshop ID: 987654321 | Another Mod -->
    <li>another.author.mod</li>
  </Mods>
</ModList>
```

### Настройки

Настройки автоматически сохраняются в `settings.json` и имеют следующий вид:

```json
{
  "steamcmd_path": "C:/steamcmd/steamcmd.exe",
  "output_path": "C:/RimWorld/ModLists",
  "xml_filename": "MyModList",
  "mods_download_path": "C:/RimWorld/Mods",
  "temp_download_path": "C:/Temp/RimWorldMods",
  "work_mode": 1,
  "log_font_size": 10,
  "verbose_logging": false
}
```

# Разработка

### Добавление новых функций

1. Создайте новый модуль в директории проекта
2. Импортируйте его в `__init__.py`
3. Интегрируйте с `WorkerThread` в `main.py`

### Тестирование

```bash
# Запуск отдельных модулей для тестирования
python database.py
python steam_handler.py
python xml_processor.py
```

# Поддержка

При неправильной работе программы, создайте ишью (Issue) в репозитории со следующим содержанием:
- Версия Python
- Операционная система
- Описание проблемы
- Шаги для воспроизведения


# 🤝 Помогите нам в развитии проекта!

1. Создайте ответвление (Fork) репозитория
3. Внесите нужные изменения (Commitы) в форкнутую ветвь (Branch)
5. Откройте запрос на слияние (Pull Request, PR) на этом проекте и ждите вердикта контрибьюторов!

---

**Сделано с ❤️ для сообщества RimWorld**
