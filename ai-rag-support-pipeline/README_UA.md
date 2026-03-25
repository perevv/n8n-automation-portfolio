![n8n](https://img.shields.io/badge/n8n-automation-orange)
![AI](https://img.shields.io/badge/AI-Groq%20%7C%20Gemini-blue)
![Support](https://img.shields.io/badge/System-AI%20Support-green)

# AI Customer Support System with Knowledge Base (n8n Workflow)

🇺🇦 Українська | 🇺🇸 [English](README_EN.md)

---

## Огляд

Цей проєкт демонструє **AI automation workflow для обробки звернень клієнтів**, створений за допомогою **n8n**, **LLM (Gemini / Groq)** та **Supabase** як бази знань.

Система автоматично:

- приймає звернення клієнтів  
- визначає тему запиту  
- знаходить відповідь у базі знань  
- генерує відповідь клієнту  
- або створює тікет для менеджера  

Також реалізовано:

- fallback логіку для AI  
- обробку невідомих тем  
- логування запитів  
- rate limiting  

Проєкт створений як **learning / demo AI support system** для демонстрації:

- AI routing (topic classification)
- knowledge base integration
- fallback моделей
- automation support flows
- ticketing logic

---

## Архітектура Workflow

![Workflow](workflow.png)

Workflow складається з наступних етапів:

1. **Webhook** — отримання масиву звернень  
2. **customerMessage** — нормалізація даних + генерація chatId  
3. **Loop Over Items** — обробка кожного клієнта окремо  
4. **autoTopic (AI)** — визначення теми звернення  
5. **Fallback AI (Groq)** — резервна модель  
6. **topic parser** — отримання теми  
7. **topicUnknown check**
   - unknown → створення тікета  
   - known → пошук у базі знань  
8. **Supabase (Knowledge Base)** — пошук по темі  
9. **supabaseEmpty check**
   - empty → тікет  
   - found → формування knowledge  
10. **assistantSupport (AI)** — генерація відповіді  
11. **Fallback AI (Groq)** — резервна відповідь  
12. **answer parser** — отримання відповіді  
13. **answerUnknown check**
   - fallback → тікет  
   - success → відповідь клієнту  
14. **Telegram (newTicket)** — створення тікета  
15. **Google Sheets (logs)** — логування звернень  

---

## Як працює система

### 1. Webhook

Workflow отримує масив звернень:

```json
[
 {
  "name": "vlad",
  "message": "Яка вартість доставки?"
 },
 {
  "name": "pavlo",
  "message": "Мені потрібно оформити повернення покупки..."
 }
]
```

---

### 2. Нормалізація даних

Node **customerMessage**:

```javascript
const crypto = require('crypto');

const data = $input.all();

const customerInfo = data.flatMap(item =>
  item.json.body.map(bodyItem => ({
    chatId: crypto.randomUUID(),
    name: bodyItem.name,
    message: bodyItem.message,
    timestamp: new Date().toISOString()
  }))
);

return customerInfo;
```

---

### 3. AI визначення теми

Node **autoTopic** визначає тему:

Категорії:

- ціни  
- доставка  
- повернення  
- unknown  

Prompt:

```
Ти визначаєш тему звернення клієнта.

Відповідь тільки одним словом.
```

Fallback: **Groq model**

---

### 4. Парсинг теми

```javascript
const content = $json.content?.parts[0]?.text;
const text = $json.text;

// Захист від помилок та нормалізація для пошуку в БД
const topic = (content || text || 'unknown').toLowerCase().trim();

return { topic };
```

---

### 5. Unknown тема

Якщо:

```
topic = unknown
```

→ створюється **тікет у Telegram**

---

### 6. Пошук у базі знань

Запит до **Supabase**:

```
GET /rest/v1/{table}?topic=eq.{topic}
```

---

### 7. Перевірка результату

Якщо дані не знайдено:

→ створюється тікет  

Якщо знайдено:

→ формується knowledge

---

### 8. Формування knowledge

```javascript
const item = $input.first().json;

return {
  knowledge: `Topic: ${item.topic}\nContent: ${item.content}`
};
```

---

### 9. AI відповідь

Node **assistantSupport**:

- використовує knowledge base  
- відповідає лише на її основі  

Fallback: **Groq model**

---

### 10. Перевірка відповіді

Якщо відповідь містить:

```
менеджер / unknown / не знаю
```

→ створюється тікет  

Інакше → відповідь клієнту

---

### 11. Створення тікета

Telegram повідомлення:

```
Новий тікет!
ID: {{chatId}}
Клієнт: {{name}}
Повідомлення: {{message}}
```

---

### 12. Логування

Google Sheets зберігає:

- id  
- name  
- message  
- timestamp  

---

## Використані технології

- **n8n** — automation platform  
- **Google Gemini AI** — primary model  
- **Groq AI** — fallback model  
- **Supabase** — knowledge base  
- **Webhook** — отримання даних  
- **JavaScript** — обробка  
- **Telegram Bot API** — тікети  
- **Google Sheets API** — логування  

---

## Можливі покращення

- multi-language support  
- vector search (embeddings)  
- CRM integration  
- SLA tracking  
- AI confidence scoring  

---

## Setup Notes

Цей workflow є **демонстраційною AI support системою**.

Для використання потрібно:

- налаштувати **Gemini / Groq API**
- створити **Supabase table (knowledge base)**
- підключити **Telegram Bot**
- налаштувати **Google Sheets**
- імпортувати workflow у **n8n**
