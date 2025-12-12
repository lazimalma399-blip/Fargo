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
    """Хранилище для токенов доступа и обновления, включая время истечения."""
    access_token: str
    refresh_token: Optional[str] = None
    expires_in: Optional[int] = None
    # Фиксируем время получения токена (для расчета срока действия)
    issued_at: float = field(default_factory=time.time) 
    
    @property
    def is_expired(self) -> bool:
        """Проверяет, истек ли access_token (с запасом 60 секунд)."""
        if self.expires_in is None:
            return False
        # Проверяем, осталось ли менее 60 секунд до истечения срока
        # 
        return (time.time() - self.issued_at) > (self.expires_in - 60) 

# --- 2. Улучшенный Класс API-Клиента ---

class AuthenaClient:
    """
    Класс-обертка для взаимодействия с Authena API с логикой обновления токена.
    """
    
    BASE_URL = "https://mock-authena-api.com/v1"
    
    def __init__(self, api_key: str, max_retries: int = 3):
        self.api_key = api_key
        self.session = self._create_resilient_session(max_retries)
        self.tokens: Optional[AuthTokens] = None
        
    # --- Служебные методы (Опущены для краткости, они те же, что и раньше) ---
    
    def _create_resilient_session(self, max_retries):
        """Создает requests.Session с логикой повторных попыток."""
        retry_strategy = Retry(total=max_retries, backoff_factor=1.0, status_forcelist=[500, 502, 503, 504])
        adapter = HTTPAdapter(max_retries=retry_strategy)
        session = requests.Session()
        session.mount("http://", adapter)
        session.mount("https://", adapter)
        return session
    
    def _get_auth_headers(self) -> Dict[str, str]:
        """Формирует заголовки, включая токен доступа."""
        headers = {"X-API-Key": self.api_key, "Content-Type": "application/json", "Accept": "application/json"}
        if self.tokens and self.tokens.access_token:
            headers["Authorization"] = f"Bearer {self.tokens.access_token}"
        return headers

    # --- 3. Метод обновления токена (НОВЫЙ) ---

    def refresh_access_token(self) -> bool:
        """
        Использует refresh_token для получения нового access_token.
        """
        if not self.tokens or not self.tokens.refresh_token:
            print("❌ ОШИБКА: Токен обновления не доступен. Требуется повторный вход.")
            return False

        url = f"{self.BASE_URL}/auth/refresh"
        payload = {"refreshToken": self.tokens.refresh_token}
        
        print(f"\n-> 🔄 Попытка обновления токена...")
        
        try:
            # Запрос на обновление
            response = self.session.post(url, json=payload, headers=self._get_auth_headers(), timeout=15)
            response.raise_for_status()
            
            response_data = response.json()
            new_
