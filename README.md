import math
from typing import Tuple, List, Dict, Any, Optional
import time

# --- Настройки и константы ---

R_EARTH_KM = 6371.0 

# --- 1. Основные математические функции (повторное использование) ---

def degrees_to_radians(degrees: float) -> float:
    """Конвертирует градусы в радианы."""
    return degrees * (math.pi / 180)

def calculate_haversine_distance(
    lat1: float, lon1: float, lat2: float, lon2: float
) -> float:
    """
    Рассчитывает расстояние между двумя точками на сфере Земли 
    с использованием формулы гаверсинуса.
    """
    phi1 = degrees_to_radians(lat1)
    phi2 = degrees_to_radians(lat2)
    delta_phi = degrees_to_radians(lat2 - lat1)
    delta_lambda = degrees_to_radians(lon2 - lon1)

    a = (math.sin(delta_phi / 2)**2) + \
        (math.cos(phi1) * math.cos(phi2) * \
         (math.sin(delta_lambda / 2)**2))
        
    c = 2 * math.atan2(math.sqrt(a), math.sqrt(1 - a))
    
    distance_km = R_EARTH_KM * c
    return distance_km

# --- 2. Функция поиска ближайшего объекта (УЛУЧШЕНИЕ) ---

def find_nearest_location(
    current_coords: Tuple[float, float], 
    locations_list: List[Dict[str, Any]]
) -> Optional[Dict[str, Any]]:
    """
    Находит ближайший объект из списка к текущим координатам.
    
    Каждый объект в списке должен содержать ключи 'latitude' и 'longitude'.
    Возвращает словарь ближайшего объекта, дополненный ключом 'distance_km'.
    """
    if not locations_list:
        return None

    current_lat, current_lon = current_coords
    min_distance = float('inf')
    nearest_location = None
    
    start_time = time.time()
    
    # Итерация и расчет расстояния для каждого объекта
    for location in locations_list:
        try:
            loc_lat = location['latitude']
            loc_lon = location['longitude']
            
            # Расчет расстояния
            distance = calculate_haversine_distance(
                current_lat, current_lon, loc_lat, loc_lon
            )
            
            # Обновление ближайшего объекта
            if distance < min_distance:
                min_distance = distance
                nearest_location = location.copy() # Копируем данные
                
        except KeyError:
            print(f"⚠️ Пропущен объект с неверными ключами: {location}")
            continue

    end_time = time.time()
    
    if nearest_location:
        # Добавляем рассчитанное расстояние к результату
        nearest_location['distance_km'] = min_distance
        print(f"⏱️ Расчет завершен. Время выполнения: {(end_time - start_time) * 1000:.2f} мс")
        return nearest_location
    
    return None

# --- Главный запуск программы (Демонстрация) ---

if __name__ == "__main__":
    
    print("--- 📍 ГЕОЛОКАЦИОННЫЙ ПОИСК БЛИЖАЙШЕГО ОБЪЕКТА ---")
    
    # Текущее местоположение пользователя (Имитация: Центр Киева)
    USER_LOCATION = (50.4501, 30.5234) # Киев
    
    # Список потенциальных объектов (например, заправок/кафе)
    LOCATIONS_TO_CHECK: List[Dict[str, Any]] = [
        # Расстояние от Киева:
        {"name": "Кафе 'Центральное'", "latitude": 50.4401, "longitude": 30.5134}, # ~1.5 км
        {"name": "АЗС WOG (Трасса)", "latitude": 50.3200, "longitude": 30.8000},    # ~30 км
        {"name": "Музей (Далеко)", "latitude": 49.8397, "longitude": 24.0297},     # ~460 км (Львов)
        {"name": "Супермаркет 'У дома'", "latitude": 50.4510, "longitude": 30.5230}, # ~0.1 км
    ]
    
    print(f"\nВаше текущее местоположение: Широта {USER_LOCATION[0]:.4f}, Долгота {USER_LOCATION[1]:.4f}")
    
    # 1. Запуск поиска
    nearest = find_nearest_location(USER_LOCATION, LOCATIONS_TO_CHECK
