import json
from jsonschema import validate, exceptions
from typing import Dict, Any

# --- 1. Определение JSON Схемы ---

# Схема определяет структуру, типы данных и обязательность полей.
# Это "контракт" между клиентом и сервером.
USER_PROFILE_SCHEMA: Dict[str, Any] = {
    "type": "object",
    "properties": {
        "id": {"type": "integer", "description": "Уникальный ID пользователя."},
        "email": {"type": "string", "format": "email", "description": "Email пользователя."},
        "firstName": {"type": "string", "minLength": 2, "description": "Имя пользователя."},
        "lastName": {"type": "string", "description": "Фамилия пользователя."},
        "company": {"type": "string", "description": "Название компании."},
        "isActive": {"type": "boolean", "description": "Статус активации аккаунта."},
        "roles": {
            "type": "array",
            "items": {"type": "string"},
            "minItems": 1
        }
    },
    # Обязательные поля
    "required": ["id", "email", "firstName", "isActive", "roles"],
    "additionalProperties": False # Запрет на лишние поля, не указанные в схеме
}

# --- 2. Имитация данных, полученных от API ---

# Пример 1: Корректный ответ от API
VALID_USER_DATA: Dict[str, Any] = {
    "id": 101,
    "email": "user@authena.com",
    "firstName": "Алекс",
    "lastName": "Смит",
    "company": "Tech Solutions",
    "isActive": True,
    "roles": ["admin", "user"]
}

# Пример 2: Некорректный ответ (нарушение схемы)
INVALID_USER_DATA: Dict[str, Any] = {
    "id": "101", # Ошибка: должен быть INTEGER
    "email": "invalid-email", # Ошибка: не соответствует формату "email"
    "firstName": "А", # Ошибка: minLength = 2
    "isActive": 1, # Ошибка: должен быть BOOLEAN
    "roles": [] # Ошибка: minItems = 1
}

# --- 3. Основная функция валидации ---

def validate_json_data(data: Dict[str, Any], schema: Dict[str, Any], schema_name: str) -> bool:
    """
    Проверяет, соответствует ли словарь данных заданной JSON-схеме.
    """
    print(f"\n--- 🔎 Валидация данных по схеме '{schema_name}' ---")
    
    try:
        # 
        validate(instance=data, schema=schema)
        print("✅ ВАЛИДАЦИЯ УСПЕШНА: Данные полностью соответствуют схеме.")
        return True
        
    except exceptions.ValidationError as err:
        print("❌ ОШИБКА ВАЛИДАЦИИ:")
        
        # Вывод детальной информации об ошибке
        print(f"   Поле: {list(err.path)} (Путь к ошибке)")
        print(f"   Ошибка: {err.message}")
        print(f"   Схема ожидает: {err.schema}")
        
        return False
    except Exception as e:
        print(f"❌ Непредвиденная ошибка валидации: {e}")
        return False

# --- Главный запуск программы ---

if __name__ == "__main__":
    
    print("--- 🔬 ИНСТРУМЕНТ ВАЛИДАЦИИ JSON-СХЕМЫ ---")
    
    # 1. Проверка корректных данных
    is_valid_1 = validate_json_data(VALID_USER_DATA, USER_PROFILE_SCHEMA, "Профиль пользователя (Корректный)")
    
    print("-" * 60)
    
    # 2. Проверка некорректных данных
    is_valid_2 = validate_json_data(INVALID_USER_DATA, USER_PROFILE_SCHEMA, "Профиль пользователя (Некорректный)")
    
    print("-" * 60)
    print(f"Итого: Корректные данные прошли проверку: {is_valid_1}")
    print(f"Итого: Некорректные данные прошли проверку: {is_valid_2}")
