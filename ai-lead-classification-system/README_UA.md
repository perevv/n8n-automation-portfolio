![n8n](https://img.shields.io/badge/n8n-automation-orange)
![AI](https://img.shields.io/badge/AI-Google%20Gemini-blue)
![JavaScript](https://img.shields.io/badge/Code-JavaScript-yellow)

# AI Lead Classification System (n8n Workflow)

🇺🇦 Українська | 🇺🇸 [English](README_EN.md)

## Огляд

Цей проєкт демонструє **AI-workflow для обробки та класифікації лідів**, створений за допомогою **n8n**.

Workflow отримує дані ліда через webhook, обробляє їх за допомогою JavaScript, аналізує повідомлення клієнта через **Google Gemini AI**, зберігає результат у **Google Sheets** та відправляє сповіщення у **Telegram**.

Проєкт створений як **learning / demo automation system**.

---

## Архітектура Workflow

![Workflow](workflow.png)

Workflow складається з наступних етапів:

1. **Webhook** — отримання даних ліда  
2. **JavaScript обробка даних** — структуризація даних та визначення пріоритету  
3. **AI класифікація (Google Gemini)** — аналіз повідомлення клієнта  
4. **Парсинг відповіді AI** — витяг категорії з JSON відповіді  
5. **Google Sheets** — збереження оброблених даних  
6. **Telegram сповіщення** — повідомлення про нового ліда

---

## Як працює система

### 1. Webhook

Workflow отримує дані ліда через webhook.

Приклад payload:

```json
{
  "name": "John Doe",
  "email": "john@email.com",
  "phone": "+123456789",
  "source": "facebook",
  "message": "How much does your service cost?"
}
```

---

### 2. Обробка даних (JavaScript)

Node **Code**:

- витягує потрібні дані з webhook
- визначає пріоритет ліда
- створює timestamp

Логіка пріоритету:

- **Facebook / Instagram → Високий**
- **Website → Середній**
- **Інші джерела → Низький**

---

### 3. AI класифікація повідомлення

Повідомлення клієнта передається в **Google Gemini**, який повертає JSON з категорією повідомлення.

Категорії:

- **ЦІНА** — питання про вартість  
- **ПІДТРИМКА** — технічна проблема  
- **ІНФО** — загальна інформація  
- **ІНШЕ** — інші запити  

Приклад відповіді AI:

```json
{
  "category": "ЦІНА"
}
```

---

### 4. Парсинг відповіді AI

JavaScript node парсить JSON відповідь від Gemini та витягує категорію повідомлення.

---

### 5. Обробка помилок

Якщо AI не повернув валідну відповідь, workflow використовує **fallback**:

- категорія → **Невідомо**
- записується поле **error**

---

### 6. Збереження даних

Оброблені дані зберігаються у **Google Sheets**.

Зберігаються поля:

- name  
- email  
- phone  
- source  
- priority  
- message  
- timestamp  
- category  
- error (якщо виникла помилка)

---

### 7. Telegram сповіщення

Після обробки ліда відправляється повідомлення у **Telegram**.

Приклад:

```
🔔 Новий лід!

Пріоритет: 🔥 Високий
Імʼя: John Doe
Email: john@email.com
Телефон: +123456789
Джерело: facebook
Повідомлення: How much does your service cost?
Час: 12.04.2026 14:21
```

---

## Використані технології

- **n8n** — automation workflow platform  
- **Webhooks** — отримання лідів  
- **JavaScript (Code node)** — обробка даних  
- **Google Gemini AI** — AI класифікація повідомлень  
- **Google Sheets API** — збереження даних  
- **Telegram Bot API** — сповіщення  

---

## Можливі покращення

- інтеграція з CRM  
- перевірка лідів на дублікати  
- автоматичний розподіл лідів між менеджерами  
- retry-логіка для AI запитів  
- аналітичний dashboard

---

## Примітки щодо налаштування

Цей Workflow є **зразковою версією**.

Замініть такі значення:

- `YOUR_CHAT_ID`
- `YOUR_SPREADSHEET_ID`
