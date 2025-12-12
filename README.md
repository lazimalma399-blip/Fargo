import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry
import json
from dataclasses import dataclass, field
import time
from typing import Optional, Dict, Any

# --- 1. Модель данных для токена ---

@dataclass
class AuthTokens:
    """Хранилище для токенов доступа и обновления."""
    access_token: str
    refresh_token: Optional[str] = None
    expires_in: Optional[int] = None
    issued_at: float = field(default_factory=time.time)

# --- 2. Класс API-Клиента (модульный подход) ---

class AuthenaClient:
    """
    Класс-обертка для взаимодействия с Authena API, включая логику токенов.
    """
    
    BASE_URL = "https://mock-authena-api.com/v1"
    
    def __init__(self, api_key: str, max_retries: int = 3):
        self.api_key = api_key
        self.session = self._create_resilient_session(max_retries)
        self.tokens: Optional[AuthTokens] = None

    def _create_resilient_session(self, max_retries):
        """Создает requests.Session с логикой повторных попыток."""
        retry_strategy = Retry(
            total=max_retries,
            backoff_factor=1.0,
            status_forcelist=[500, 502, 503, 504],
        )
        adapter = HTTPAdapter(max_retries=retry_strategy)
        session = requests.Session()
        session.mount("http://", adapter)
        session.mount("https://", adapter)
        return session
    
    def _get_auth_headers(self) -> Dict[str, str]:
        """Формирует заголовки, включая токен доступа."""
        headers = {
            "X-API-Key": self.api_key, 
            "Content-Type": "application/json",
            "Accept": "application/json"
        }
        if self.tokens and self.tokens.access_token:
            # Если токен есть, добавляем его для защищенных запросов
            headers["Authorization"] = f"Bearer {self.tokens.access_token}"
        return headers

    def authenticate(self, email: str, password: str) -> bool:
        """
        Выполняет вход и сохраняет токены.
        """
        url = f"{self.BASE_URL}/auth/login"
        payload = {"email": email, "password": password}
        
        print(f"\n-> 🔑 Вход: {email}...")
        
        try:
            # Запрос без токена (используем только API-Key)
            response = self.session.post(url, json=payload, headers=self._get_auth_headers(), timeout=15)
            response.raise_for_status()
            
            response_data = response.json()
            
            # 
            access_token = response_data.get('accessToken')
            refresh_token = response_data.get('refreshToken') # Добавляем refresh_token
            expires_in = response_data.get('expiresIn')

            if access_token:
                self.tokens = AuthTokens(
                    access_token=access_token, 
                    refresh_token=refresh_token, 
                    expires_in=expires_in
                )
                print(f"✅ УСПЕХ: Токен получен и сохранен. Срок действия: {expires_in} с.")
                return True
            
            print("❌ ОШИБКА: Сервер не вернул токен.")
            return False

        except requests.exceptions.HTTPError as e:
            if e.response.status_code == 401:
                print("❌ ОШИБКА 401: Неверный email или пароль.")
            else:
                print(f"❌ HTTP Ошибка (Код {e.response.status_code}): {e.response.text}")
        except requests.exceptions.RequestException as e:
            print(f"❌ Сетевая ошибка: {e}")
        
        return False
        
    def get_user_profile(self) -> Optional[Dict[str, Any]]:
        """
        Выполняет защищенный GET-запрос к профилю пользователя.
        """
        if not self.tokens:
            print("⚠️ ОШИБКА: Сначала необходимо выполнить аутентификацию.")
            return None
            
        url = f"{self.BASE_URL}/user/profile"
        
        try:
            # Запрос автоматически включает токен из self.tokens
            response = self.session.get(url, headers=self._get_auth_headers(), timeout=10)
            response.raise_for_status()
            
            print(f"✅ Защищенный запрос к профилю успешен.")
            return response.json()
            
        except requests.exceptions.HTTPError as e:
             print(f"❌ ОШИБКА ДОСТУПА (Код {e.response.status_code}): Токен мог истечь.")
             # В реальной реализации здесь нужно вызвать функцию refresh_token()
        except requests.exceptions.RequestException as e:
            print(f"❌ Сетевая ошибка: {e}")
