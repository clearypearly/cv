# Пошаговое исправление ошибок Make.com

## Шаг 1: Исправление ошибки 415 "Unsupported Media Type"

### Проблема из Image 1:
Ошибка HTTP 415 указывает на неправильный Content-Type заголовок.

### Решение:

1. **Откройте HTTP модуль в Make.com**
2. **Проверьте настройки заголовков:**
   - Для **Gupshup API**: измените Content-Type на `application/x-www-form-urlencoded`
   - Для **Chatwoot API**: используйте `application/json`

3. **Правильные настройки для Gupshup:**
   ```
   Headers:
   - Content-Type: application/x-www-form-urlencoded
   - apikey: ВАШ_GUPSHUP_API_КЛЮЧ
   ```

4. **Правильные настройки для Chatwoot:**
   ```
   Headers:
   - Content-Type: application/json
   - api_access_token: ВАШ_CHATWOOT_ТОКЕН
   ```

## Шаг 2: Исправление ошибки 400 "Bad Request"

### Проблема из Image 3:
Ошибка HTTP 400 указывает на неправильный формат данных или отсутствие обязательных полей.

### Решение:

1. **Проверьте формат тела запроса для Gupshup:**
   
   **Неправильно (может вызывать 400):**
   ```json
   {
     "message": "простой текст"
   }
   ```
   
   **Правильно:**
   ```
   Body Type: application/x-www-form-urlencoded
   
   Fields:
   - channel: whatsapp
   - source: 919876543210
   - destination: 919876543211
   - message: {"type":"text","text":"Ваше сообщение"}
   - src.name: YourAppName
   ```

2. **Проверьте формат номеров телефонов:**
   - **Правильно**: `919876543210` (код страны + номер, БЕЗ знака +)
   - **Неправильно**: `+91 98765 43210`, `+919876543210`, `98765 43210`

3. **Удалите проблемные символы из текста сообщения:**
   - Кавычки: `"` → `\"`
   - Переносы строк: удалите или замените на пробелы
   - Эмодзи: могут вызывать проблемы в некоторых случаях

## Шаг 3: Настройка HTTP модуля в Make.com для Gupshup

### Полная конфигурация:

1. **URL**: `https://api.gupshup.io/sm/api/v1/msg`
2. **Method**: POST
3. **Headers**:
   - Name: `Content-Type`, Value: `application/x-www-form-urlencoded`
   - Name: `apikey`, Value: `ваш_api_ключ_gupshup`

4. **Body Type**: выберите "application/x-www-form-urlencoded"

5. **Fields** (добавьте каждое поле отдельно):
   - Name: `channel`, Value: `whatsapp`
   - Name: `source`, Value: `{{ваш_номер_источника}}`
   - Name: `destination`, Value: `{{номер_получателя}}`
   - Name: `message`, Value: `{"type":"text","text":"{{текст_сообщения}}"}`
   - Name: `src.name`, Value: `ИмяВашегоПриложения`

## Шаг 4: Настройка HTTP модуля в Make.com для Chatwoot

### Полная конфигурация:

1. **URL**: `https://ваш-домен-chatwoot.com/api/v1/accounts/{{account_id}}/conversations/{{conversation_id}}/messages`
2. **Method**: POST
3. **Headers**:
   - Name: `Content-Type`, Value: `application/json`
   - Name: `api_access_token`, Value: `ваш_токен_chatwoot`

4. **Body Type**: выберите "application/json"

5. **JSON Body**:
   ```json
   {
     "content": "{{текст_сообщения}}",
     "message_type": "outgoing",
     "private": false
   }
   ```

## Шаг 5: Отладка и тестирование

### 1. Включите детальное логирование:
- В настройках HTTP модуля включите все опции логирования
- После выполнения проверьте History в Make.com
- Изучите Request и Response для выявления проблем

### 2. Тестирование вне Make.com:

**Тест Gupshup через Postman или curl:**
```bash
curl -X POST "https://api.gupshup.io/sm/api/v1/msg" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "apikey: ВАШ_API_КЛЮЧ" \
  -d "channel=whatsapp&source=919876543210&destination=919876543211&message={\"type\":\"text\",\"text\":\"Тест\"}&src.name=TestApp"
```

**Тест Chatwoot через Postman или curl:**
```bash
curl -X POST "https://ваш-домен.com/api/v1/accounts/1/conversations/123/messages" \
  -H "Content-Type: application/json" \
  -H "api_access_token: ВАШ_ТОКЕН" \
  -d '{"content":"Тест","message_type":"outgoing","private":false}'
```

### 3. Проверьте ответы:
- **Успешный ответ Gupshup**: HTTP 202 с messageId
- **Успешный ответ Chatwoot**: HTTP 200 с данными сообщения

## Шаг 6: Добавление обработки ошибок

### Настройте Error Handler в Make.com:

1. **Добавьте Error Handler route** к вашему HTTP модулю
2. **Добавьте модуль фильтрации** для обработки разных типов ошибок
3. **Настройте уведомления** об ошибках (email, Slack, etc.)
4. **Добавьте повторные попытки** для временных ошибок

### Пример фильтра для обработки ошибок:
```
Condition: {{1.statusCode}} не равен 200 И {{1.statusCode}} не равен 202
```

## Частые ошибки и их исправления:

### ❌ Ошибка: "Invalid phone number format"
**Решение**: Используйте формат `919876543210` (без +, пробелов, дефисов)

### ❌ Ошибка: "Missing required field 'apikey'"
**Решение**: Добавьте заголовок `apikey` с вашим API ключом Gupshup

### ❌ Ошибка: "Unauthorized" (401)
**Решение**: Проверьте правильность API ключей и токенов

### ❌ Ошибка: "Conversation not found" 
**Решение**: Убедитесь, что conversation_id существует в Chatwoot

### ❌ Ошибка: JSON parse error
**Решение**: Проверьте синтаксис JSON, экранируйте кавычки

## Проверочный чек-лист:

- [ ] Content-Type заголовок правильный для каждого API
- [ ] Все обязательные поля присутствуют
- [ ] Формат номеров телефонов корректен
- [ ] API ключи и токены действительны
- [ ] JSON синтаксически правильный
- [ ] Тестирование вне Make.com работает
- [ ] Настроена обработка ошибок
- [ ] Включено детальное логирование

После выполнения всех шагов ваши сценарии должны работать без ошибок 415 и 400.