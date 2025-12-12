import logging
import logging.handlers
import sys
import time
from pathlib import Path

# --- Настройки ---
LOG_FILE_PATH = Path("application.log")
LOG_LEVEL = logging.INFO # Минимальный уровень для записи и вывода
LOG_FORMAT = '%(asctime)s - %(levelname)s - %(name)s - %(funcName)s - %(message)s'
DATE_FORMAT = '%Y-%m-%d %H:%M:%S'

# --- 1. Основная функция настройки логгера ---

def setup_logging(log_file: Path, level: int = logging.INFO):
    """
    Конфигурирует основной логгер (root logger), добавляя обработчики 
    для файла и консоли.
    """
    
    # Создание основного объекта логгера
    root_logger = logging.getLogger()
    root_logger.setLevel(level)
    
    # Форматер для определения внешнего вида сообщений
    formatter = logging.Formatter(LOG_FORMAT, datefmt=DATE_FORMAT)

    # 1. File Handler (Обработчик для записи в файл)
    # 
    # RotatingFileHandler: автоматически создает новый файл, когда текущий 
    # достигает максимального размера, сохраняя старые файлы.
    file_handler = logging.handlers.RotatingFileHandler(
        log_file,
        maxBytes=1048576, # 1 MB
        backupCount=5,     # Хранить до 5 старых лог-файлов
        encoding='utf-8'
    )
    file_handler.setFormatter(formatter)
    root_logger.addHandler(file_handler)
    
    # 2. Console Handler (Обработчик для вывода в консоль)
    # StreamHandler выводит логи в стандартный поток вывода (stdout/stderr)
    console_handler = logging.StreamHandler(sys.stdout)
    console_handler.setFormatter(formatter)
    root_logger.addHandler(console_handler)
    
    # Сообщение о готовности системы
    root_logger.info("Система логирования успешно инициализирована.")
    
# --- 2. Функции для демонстрации логирования ---

# Получаем логгер для конкретного модуля (лучшая практика)
api_logger = logging.getLogger("API_Client")
db_logger = logging.getLogger("Database_Service")

def fetch_user_data(user_id):
    """Имитирует получение данных и логирует этапы."""
    api_logger.info(f"Начало запроса данных для пользователя ID: {user_id}")
    try:
        if user_id % 3 == 0:
            raise ConnectionError("Сетевое соединение прервано.")
            
        if user_id % 5 == 0:
            db_logger.warning(f"Медленный ответ от БД для ID {user_id}")
            time.sleep(0.5)
            
        api_logger.debug("Этот уровень не будет записан, так как уровень: INFO")
        api_logger.info(f"Успешно получены данные пользователя ID: {user_id}")
        return True

    except ConnectionError as e:
        # Логирование ошибок (уровень ERROR)
        api_logger.error(f"Критическая ошибка при получении данных ID {user_id}: {e}")
        return False

# --- Главный запуск программы ---

if __name__ == "__main__":
    
    print("--- 📝 ИНСТРУМЕНТ ПРОФЕССИОНАЛЬНОГО ЛОГИРОВАНИЯ ---")
    
    # 1. Настройка системы логирования
    setup_logging(LOG_FILE_PATH, LOG_LEVEL)
    
    # 2. Имитация работы системы
