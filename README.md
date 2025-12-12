import math
from typing import Tuple

# --- Настройки и константы ---

# Радиус Земли в километрах (среднее значение)
R_EARTH_KM = 6371.0 
# Средняя скорость движения (км/ч) для оценки времени
AVG_SPEED_KMH = 60.0 

# --- 1. Основные математические функции ---

def degrees_to_radians(degrees: float) -> float:
    """Конвертирует градусы в радианы."""
    return degrees * (math.pi / 180)

def calculate_haversine_distance(
    lat1: float, lon1: float, lat2: float, lon2: float
) -> float:
    """
    Рассчитывает расстояние между двумя точками на сфере Земли 
    с использованием формулы гаверсинуса.
    

    Возвращает расстояние в километрах.
    """
    
    # Конвертация координат из градусов в радианы
    phi1 = degrees_to_radians(lat1)
    phi2 = degrees_to_radians(lat2)
    delta_phi = degrees_to_radians(lat2 - lat1)
    delta_lambda = degrees_to_radians(lon2 - lon1) # Разница долгот

    # Применение формулы гаверсинуса
    a = (math.sin(delta_phi / 2)**2) + \
        (math.cos(phi1) * math.cos(phi2) * \
         (math.sin(delta_lambda / 2)**2))
        
    c = 2 * math.atan2(math.sqrt(a), math.sqrt(1 - a))
    
    # Расстояние = Радиус Земли * c
    distance_km = R_EARTH_KM * c
    
    return distance_km

# --- 2. Функция расчета времени и маршрута ---

def get_route_summary(
    start_coords: Tuple[float, float], end_coords: Tuple[float, float], avg_speed: float
) -> Tuple[float, float]:
    """
    Рассчитывает расстояние и примерное время в пути.
    
    Возвращает (расстояние в км, время в часах).
    """
    
    lat1, lon1 = start_coords
    lat2, lon2 = end_coords
    
    # Шаг 1: Расчет расстояния
    distance_km = calculate_haversine_distance(lat1, lon1, lat2, lon2)
    
    # Шаг 2: Расчет времени (Время = Расстояние / Скорость)
    time_hours = distance_km / avg_speed
    
    return distance_km, time_hours

# --- Главный запуск программы (Демонстрация) ---

if __name__ == "__main__":
    
    print("--- 🗺️ ГЕОЛОКАЦИОННЫЙ РАСЧЕТ МАРШРУТА ---")
    
    # Координаты:
    # Точка А (Киев, Украина)
    KYIV_COORDS = (50.4501, 30.5234)
    # Точка Б (Одесса, Украина)
    ODESSA_COORDS = (46.4825, 30.7233)
    
    START_CITY = "Киев"
    END_CITY = "Одесса"

    # 1. Расчет
    distance, time_hours = get_route_summary(
        KYIV_COORDS, ODESSA_COORDS, AVG_SPEED_KMH
    )
    
    time_minutes = time_hours * 60
    
    # 2. Вывод результатов
    print(f"\n📍 Отправление: {START_CITY} (Ш: {KYIV_COORDS[0]:.4f}, Д: {KYIV_COORDS[1]:.4f})")
    print(f"🏁 Назначение: {END_CITY} (Ш: {ODESSA_COORDS[0]:.4f}, Д: {ODESSA_COORDS[1]:.4f})")
    print("-" * 50)
    
    print(f"📏 Расстояние по прямой (Воздушная линия): {distance:.2f} км")
    print(f"⏳ Примерное время в пути (при {AVG_SPEED_KMH} км/ч):")
    print(f"   {int(time_hours)} ч {int(time_minutes % 60)} мин")
    print("-" * 50)
    
    # 3. Имитация "В дороге" (Две очень близкие точки)
    NEAR_POINT_A = (40.7128, -74.0060) # Нью-Йорк, США
    NEAR_POINT_B = (40.7130, -74.0062)
    
    distance_short, _ = get_route_summary(NEAR_POINT_A, NEAR_POINT_B, AVG_SPEED_KMH)
    print(f"\n[Случай 'В дороге']: Расстояние между двумя близкими точками: {distance_short * 1000:.2f} метров")
