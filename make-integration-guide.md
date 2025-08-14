# Руководство по исправлению ошибок Make.com + Gupshup + Chatwoot

## Анализ ошибок из ваших изображений

### Ошибка 1: HTTP 415 "Unsupported Media Type"
**Причина**: Неправильный Content-Type заголовок
**Решение**: Убедитесь, что используете `Content-Type: application/json`

### Ошибка 2: HTTP 400 "Bad Request" 
**Причина**: Неправильный формат данных или отсутствие обязательных полей
**Решение**: Проверьте формат JSON и все обязательные поля

## Правильная конфигурация для Gupshup API

### 1. Отправка сообщения через Gupshup (Make.com -> Gupshup)

**HTTP модуль настройки:**
- **Method**: POST
- **URL**: `https://api.gupshup.io/sm/api/v1/msg`
- **Headers**:
  ```
  Content-Type: application/x-www-form-urlencoded
  apikey: YOUR_GUPSHUP_API_KEY
  ```

**Body (Form data):**
```
channel=whatsapp
source=YOUR_PHONE_NUMBER
destination=RECIPIENT_PHONE_NUMBER
message={"type":"text","text":"Ваше сообщение"}
src.name=YOUR_APP_NAME
```

### 2. Альтернативный формат для Gupshup (JSON)

**HTTP модуль настройки:**
- **Method**: POST  
- **URL**: `https://api.gupshup.io/sm/api/v1/msg`
- **Headers**:
  ```
  Content-Type: application/json
  apikey: YOUR_GUPSHUP_API_KEY
  ```

**Body (JSON):**
```json
{
  "channel": "whatsapp",
  "source": "YOUR_PHONE_NUMBER",
  "destination": "RECIPIENT_PHONE_NUMBER", 
  "message": {
    "type": "text",
    "text": "Ваше сообщение"
  },
  "src.name": "YOUR_APP_NAME"
}
```

## Правильная конфигурация для Chatwoot API

### 1. Отправка сообщения в Chatwoot

**HTTP модуль настройки:**
- **Method**: POST
- **URL**: `https://YOUR_CHATWOOT_DOMAIN/api/v1/accounts/ACCOUNT_ID/conversations/CONVERSATION_ID/messages`
- **Headers**:
  ```
  Content-Type: application/json
  api_access_token: YOUR_CHATWOOT_ACCESS_TOKEN
  ```

**Body (JSON):**
```json
{
  "content": "Текст сообщения",
  "message_type": "outgoing",
  "private": false
}
```

### 2. Получение webhook от Chatwoot

**Webhook URL в Chatwoot**: `https://hook.integromat.com/YOUR_WEBHOOK_ID`

**Ожидаемый формат данных от Chatwoot:**
```json
{
  "event": "message_created",
  "data": {
    "account": {
      "id": 1,
      "name": "Account Name"
    },
    "conversation": {
      "id": 123,
      "inbox_id": 456
    },
    "message": {
      "id": 789,
      "content": "Текст сообщения",
      "message_type": "incoming",
      "created_at": "2023-01-01T00:00:00.000Z",
      "sender": {
        "id": 1,
        "name": "Sender Name",
        "phone_number": "+1234567890"
      }
    }
  }
}
```

## Типичные ошибки и их решения

### 1. Ошибка 415 - Unsupported Media Type
**Причины:**
- Неправильный Content-Type заголовок
- Gupshup ожидает `application/x-www-form-urlencoded` или `application/json`
- Chatwoot ожидает `application/json`

**Решение:**
- Для Gupshup: используйте `application/x-www-form-urlencoded` для form data
- Для Chatwoot: всегда используйте `application/json`

### 2. Ошибка 400 - Bad Request
**Причины:**
- Отсутствуют обязательные поля
- Неправильный формат телефонного номера
- Некорректный JSON
- Специальные символы в тексте

**Решение:**
- Проверьте все обязательные поля
- Формат телефона: `+1234567890` (с кодом страны)
- Экранируйте специальные символы в JSON
- Удалите или замените проблемные символы (кавычки, переносы строк)

### 3. Проблемы с аутентификацией
**Для Gupshup:**
- Используйте заголовок `apikey: YOUR_API_KEY`

**Для Chatwoot:**
- Используйте заголовок `api_access_token: YOUR_TOKEN`

## Отладка в Make.com

### 1. Включите детальное логирование
- В настройках HTTP модуля включите "Log all states"
- Проверяйте Response в History

### 2. Тестирование запросов
Используйте Postman или curl для тестирования API отдельно:

```bash
# Тест Gupshup API
curl -X POST "https://api.gupshup.io/sm/api/v1/msg" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "apikey: YOUR_API_KEY" \
  -d "channel=whatsapp&source=YOUR_NUMBER&destination=RECIPIENT_NUMBER&message={\"type\":\"text\",\"text\":\"Test message\"}&src.name=YOUR_APP"

# Тест Chatwoot API  
curl -X POST "https://YOUR_DOMAIN/api/v1/accounts/ACCOUNT_ID/conversations/CONV_ID/messages" \
  -H "Content-Type: application/json" \
  -H "api_access_token: YOUR_TOKEN" \
  -d '{"content":"Test message","message_type":"outgoing","private":false}'
```

### 3. Обработка ошибок в Make.com
- Добавьте Error Handler route
- Используйте Filter для проверки успешности запроса
- Логируйте ошибки в Google Sheets или другую систему

## Рекомендуемая архитектура сценария

### Сценарий 1: Получение сообщений (Chatwoot -> Make -> Gupshup)
1. **Webhook** - получение от Chatwoot
2. **Filter** - проверка типа события (message_created)
3. **Text Parser** - извлечение данных сообщения
4. **HTTP** - отправка в Gupshup
5. **Error Handler** - обработка ошибок

### Сценарий 2: Отправка сообщений (Gupshup -> Make -> Chatwoot)  
1. **Webhook** - получение от Gupshup
2. **Filter** - проверка типа сообщения
3. **HTTP** - отправка в Chatwoot
4. **Error Handler** - обработка ошибок

## Чек-лист для устранения ошибок

- [ ] Проверены все API ключи и токены
- [ ] Правильно настроены Content-Type заголовки
- [ ] Все обязательные поля присутствуют
- [ ] Формат телефонных номеров корректен
- [ ] JSON синтаксически правильный
- [ ] Удалены проблемные символы из текста
- [ ] Настроена обработка ошибок
- [ ] Протестированы API отдельно от Make.com