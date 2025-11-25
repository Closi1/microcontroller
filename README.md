# microcontroller

🎯 ЗАДАНИЕ №1: Подготовка среды сбора данных
📦 Часть 1: Установка всего необходимого
🛠️ Шаг 1.1: Установка Python
Открой браузер (Chrome, Firefox, Edge)

Перейди на сайт: python.org

Нажми большую жёлную кнопку "Download Python 3.12"

Запусти скачанный файл

ВАЖНО! Поставь галочку: ☑ "Add Python to PATH"

Нажми "Install Now"

Жди пока установится

https://docs.python.org/3/_images/win_installer.png

Проверка что установилось:

Нажми Win + R

Напиши cmd → Enter

Напиши python --version

Должно быть: Python 3.12.0

🛠️ Шаг 1.2: Установка Git
Перейди на сайт: git-scm.com

Скачай Git для Windows

Запусти установщик

Всё время нажимай "Next" (не меняй настройки)

В конце нажми "Finish"

🛠️ Шаг 1.3: Установка VS Code
Перейди на сайт: code.visualstudio.com

Скачай для Windows

Запусти установщик

При установке поставь галочки:

☑ "Add to PATH"

☑ "Create Desktop Icon"

Нажми "Install"

📁 Часть 2: Создаём проект
🗂️ Шаг 2.1: Создаём папку проекта
На рабочем столе нажми правую кнопку мыши

Выбери "Создать" → "Папку"

Назови папку: microcontroller_data_system

🗂️ Шаг 2.2: Открываем проект в VS Code
Запусти VS Code (синяя иконка на рабочем столе)

Нажми "File" вверху слева

Выбери "Open Folder"

Найди нашу папку microcontroller_data_system

Нажми "Select Folder"

https://code.visualstudio.com/assets/docs/getstarted/tips-and-tricks/Open_folder.png

🗂️ Шаг 2.3: Создаём структуру папок
В VS Code слева нажми на иконку "New Folder" и создай:

text
microcontroller_data_system/
├── microcontroller/     # Прошивка для микроконтроллера
├── server/             # Сервер для приёма данных
├── sync_service/       # Синхронизатор с RabbitMQ
└── docs/               # Документация
Как создать папки:

Наведи курсор на папку проекта слева

Нажми на иконку 📁 "New Folder"

Введи имя папки → Enter

Повтори для всех папок

🔌 Часть 3: Прошивка микроконтроллера (эмуляция)
💻 Шаг 3.1: Создаём эмулятор микроконтроллера
В папке microcontroller/ создай файл sensor_emulator.py:

В VS Code нажми правой кнопкой на папку microcontroller

Выбери "New File"

Назови файл: sensor_emulator.py

Скопируй этот код:

python
import socket
import time
import json
import random
from datetime import datetime

class MicrocontrollerEmulator:
    def __init__(self):
        self.device_id = "MCU_001"
        self.server_host = "localhost"
        self.server_port = 8888
        
    def generate_sensor_data(self):
        """Генерирует случайные данные с датчиков"""
        return {
            "device_id": self.device_id,
            "timestamp": datetime.now().isoformat(),
            "temperature": round(random.uniform(20.0, 30.0), 2),  # Температура 20-30°C
            "humidity": round(random.uniform(40.0, 80.0), 2),     # Влажность 40-80%
            "pressure": round(random.uniform(980.0, 1020.0), 2),  # Давление 980-1020 hPa
            "voltage": round(random.uniform(3.2, 3.8), 2)        # Напряжение 3.2-3.8V
        }
    
    def send_data_to_server(self):
        """Отправляет данные на сервер"""
        try:
            # Создаём сокет для подключения
            client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            client_socket.connect((self.server_host, self.server_port))
            
            # Генерируем данные
            sensor_data = self.generate_sensor_data()
            
            # Преобразуем в JSON и отправляем
            json_data = json.dumps(sensor_data)
            client_socket.sendall(json_data.encode('utf-8'))
            
            # Получаем ответ от сервера
            response = client_socket.recv(1024).decode('utf-8')
            print(f"📨 Отправлено: {sensor_data}")
            print(f"📩 Ответ сервера: {response}")
            
            client_socket.close()
            return True
            
        except Exception as e:
            print(f"❌ Ошибка отправки: {e}")
            return False
    
    def run(self):
        """Запускает эмулятор"""
        print("🚀 ЗАПУСК ЭМУЛЯТОРА МИКРОКОНТРОЛЛЕРА")
        print(f"📟 ID устройства: {self.device_id}")
        print(f"🌐 Сервер: {self.server_host}:{self.server_port}")
        print("=" * 50)
        
        counter = 0
        while True:
            counter += 1
            print(f"\n🔁 Цикл отправки #{counter}")
            
            success = self.send_data_to_server()
            
            if success:
                print("✅ Данные успешно отправлены!")
            else:
                print("❌ Ошибка отправки данных!")
            
            # Ждём 10 секунд перед следующей отправкой
            print("⏰ Ожидание 10 секунд...")
            time.sleep(10)

# Запускаем эмулятор
if __name__ == "__main__":
    emulator = MicrocontrollerEmulator()
    emulator.run()
Как сохранить:

Нажми Ctrl + S

Или Файл → Save

🖥️ Часть 4: Сервер для приёма данных
💻 Шаг 4.1: Создаём сервер
В папке server/ создай файл data_receiver.py:

python
import socket
import json
import sqlite3
from datetime import datetime
import threading

class DataReceiver:
    def __init__(self):
        self.host = "localhost"
        self.port = 8888
        self.db_path = "sensor_data.db"
        self.setup_database()
    
    def setup_database(self):
        """Создаёт базу данных SQLite и таблицу"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS sensor_data (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                device_id TEXT NOT NULL,
                timestamp TEXT NOT NULL,
                temperature REAL,
                humidity REAL,
                pressure REAL,
                voltage REAL,
                received_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                synced INTEGER DEFAULT 0
            )
        ''')
        
        conn.commit()
        conn.close()
        print(f"✅ База данных создана: {self.db_path}")
    
    def save_to_database(self, data):
        """Сохраняет данные в SQLite"""
        try:
            conn = sqlite3.connect(self.db_path)
            cursor = conn.cursor()
            
            cursor.execute('''
                INSERT INTO sensor_data 
                (device_id, timestamp, temperature, humidity, pressure, voltage)
                VALUES (?, ?, ?, ?, ?, ?)
            ''', (
                data['device_id'],
                data['timestamp'],
                data['temperature'],
                data['humidity'],
                data['pressure'],
                data['voltage']
            ))
            
            conn.commit()
            conn.close()
            print(f"💾 Данные сохранены в базу от устройства: {data['device_id']}")
            return True
            
        except Exception as e:
            print(f"❌ Ошибка сохранения в базу: {e}")
            return False
    
    def handle_client(self, client_socket, address):
        """Обрабатывает подключение клиента"""
        try:
            print(f"🔌 Подключился клиент: {address}")
            
            # Получаем данные
            data = client_socket.recv(1024).decode('utf-8')
            print(f"📥 Получены данные: {data}")
            
            # Парсим JSON
            sensor_data = json.loads(data)
            
            # Сохраняем в базу
            success = self.save_to_database(sensor_data)
            
            # Отправляем ответ клиенту
            if success:
                response = "OK: Data saved successfully"
            else:
                response = "ERROR: Failed to save data"
                
            client_socket.send(response.encode('utf-8'))
            
        except json.JSONDecodeError:
            print("❌ Ошибка: Неверный JSON формат")
            client_socket.send("ERROR: Invalid JSON format".encode('utf-8'))
        except Exception as e:
            print(f"❌ Ошибка обработки: {e}")
            client_socket.send("ERROR: Processing failed".encode('utf-8'))
        finally:
            client_socket.close()
    
    def start_server(self):
        """Запускает сервер"""
        server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        
        try:
            server_socket.bind((self.host, self.port))
            server_socket.listen(5)
            
            print("🖥️ СЕРВЕР ДЛЯ ПРИЁМА ДАННЫХ ЗАПУЩЕН")
            print(f"📍 Адрес: {self.host}:{self.port}")
            print(f"💾 База данных: {self.db_path}")
            print("=" * 50)
            print("⏳ Ожидаем подключения микроконтроллеров...")
            
            while True:
                client_socket, address = server_socket.accept()
                
                # Обрабатываем каждого клиента в отдельном потоке
                client_thread = threading.Thread(
                    target=self.handle_client, 
                    args=(client_socket, address)
                )
                client_thread.daemon = True
                client_thread.start()
                
        except KeyboardInterrupt:
            print("\n🛑 Сервер остановлен")
        except Exception as e:
            print(f"❌ Ошибка сервера: {e}")
        finally:
            server_socket.close()

# Запускаем сервер
if __name__ == "__main__":
    receiver = DataReceiver()
    receiver.start_server()
💻 Шаг 4.2: Создаём просмотрщик базы данных
В папке server/ создай файл database_viewer.py:

python
import sqlite3
import pandas as pd
from datetime import datetime

class DatabaseViewer:
    def __init__(self):
        self.db_path = "sensor_data.db"
    
    def show_all_data(self):
        """Показывает все данные из базы"""
        try:
            conn = sqlite3.connect(self.db_path)
            
            # Используем pandas для красивого вывода
            df = pd.read_sql_query("SELECT * FROM sensor_data", conn)
            
            print("📊 ВСЕ ДАННЫЕ ИЗ БАЗЫ:")
            print("=" * 80)
            
            if len(df) > 0:
                print(df.to_string(index=False))
            else:
                print("База данных пуста")
                
            print(f"\n📈 Всего записей: {len(df)}")
            
            conn.close()
            
        except Exception as e:
            print(f"❌ Ошибка чтения базы: {e}")
    
    def show_statistics(self):
        """Показывает статистику по данным"""
        try:
            conn = sqlite3.connect(self.db_path)
            cursor = conn.cursor()
            
            # Общее количество записей
            cursor.execute("SELECT COUNT(*) FROM sensor_data")
            total_records = cursor.fetchone()[0]
            
            # Количество устройств
            cursor.execute("SELECT COUNT(DISTINCT device_id) FROM sensor_data")
            unique_devices = cursor.fetchone()[0]
            
            # Последняя запись
            cursor.execute("SELECT MAX(received_at) FROM sensor_data")
            last_record = cursor.fetchone()[0]
            
            # Несинхронизированные записи
            cursor.execute("SELECT COUNT(*) FROM sensor_data WHERE synced = 0")
            unsynced_records = cursor.fetchone()[0]
            
            print("\n📈 СТАТИСТИКА БАЗЫ ДАННЫХ:")
            print("=" * 40)
            print(f"📋 Всего записей: {total_records}")
            print(f"📟 Уникальных устройств: {unique_devices}")
            print(f"🔄 Несинхронизировано: {unsynced_records}")
            print(f"🕒 Последняя запись: {last_record}")
            
            conn.close()
            
        except Exception as e:
            print(f"❌ Ошибка статистики: {e}")

if __name__ == "__main__":
    viewer = DatabaseViewer()
    viewer.show_all_data()
    viewer.show_statistics()
🚀 Часть 5: Запускаем систему
🖥️ Шаг 5.1: Устанавливаем зависимости
Создай файл requirements.txt в корне проекта:

txt
pandas
pika
Как установить зависимости:

Открой командную строку (Win+R → cmd → Enter)

Перейди в папку проекта:

cmd
cd Desktop\microcontroller_data_system
Установи зависимости:

cmd
pip install pandas pika
🎯 Шаг 5.2: Запускаем систему
Нужно открыть ТРИ окна командной строки:

Окно 1 - Сервер:

cmd
cd Desktop\microcontroller_data_system
python server\data_receiver.py
Окно 2 - Эмулятор микроконтроллера:

cmd
cd Desktop\microcontroller_data_system  
python microcontroller\sensor_emulator.py
Окно 3 - Просмотр базы данных:

cmd
cd Desktop\microcontroller_data_system
python server\database_viewer.py
👀 Шаг 5.3: Что ты должен увидеть
В окне Сервера:

text
🖥️ СЕРВЕР ДЛЯ ПРИЁМА ДАННЫХ ЗАПУЩЕН
📍 Адрес: localhost:8888
💾 База данных: sensor_data.db
==================================================
⏳ Ожидаем подключения микроконтроллеров...
🔌 Подключился клиент: ('127.0.0.1', 54321)
📥 Получены данные: {"device_id": "MCU_001", ...}
💾 Данные сохранены в базу от устройства: MCU_001
В окне Эмулятора:

text
🚀 ЗАПУСК ЭМУЛЯТОРА МИКРОКОНТРОЛЛЕРА
📟 ID устройства: MCU_001
🌐 Сервер: localhost:8888
==================================================
🔁 Цикл отправки #1
📨 Отправлено: {'device_id': 'MCU_001', ...}
📩 Ответ сервера: OK: Data saved successfully
✅ Данные успешно отправлены!
⏰ Ожидание 10 секунд...
В окне Просмотрщика:

text
📊 ВСЕ ДАННЫЕ ИЗ БАЗЫ:
================================================================================
id device_id timestamp                  temperature humidity pressure voltage received_at           synced
1  MCU_001   2024-01-15T10:30:00.123456 25.67       65.43    1001.23  3.65    2024-01-15 10:30:01   0

📈 Всего записей: 1

📈 СТАТИСТИКА БАЗЫ ДАННЫХ:
========================================
📋 Всего записей: 1
📟 Уникальных устройств: 1
🔄 Несинхронизировано: 1
🕒 Последняя запись: 2024-01-15 10:30:01
✅ Чек-лист задания №1:
✅ Python установлен

✅ VS Code установлен

✅ Проект создан с правильной структурой

✅ Эмулятор микроконтроллера создан

✅ Сервер приёма данных создан

✅ База данных SQLite работает

✅ Данные сохраняются в базу

✅ Можно просматривать данные
