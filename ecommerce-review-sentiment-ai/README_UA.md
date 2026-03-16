![Make](https://img.shields.io/badge/Make-automation-purple)
![AI](https://img.shields.io/badge/AI-Google%20Gemini-blue)
![Sentiment](https://img.shields.io/badge/AI-Sentiment%20Analysis-green)

# Ecommerce Review Sentiment AI (Make Automation)

🇺🇦 Українська | 🇺🇸 [English](README_EN.md)

## Огляд

Цей проєкт демонструє **AI automation workflow для аналізу відгуків товарів в e-commerce**, створений за допомогою **Make (Integromat)**.

Система автоматично отримує список товарів через API, аналізує відгуки клієнтів за допомогою **Google Gemini AI**, визначає рівень негативних відгуків та відправляє алерт у **Telegram**, якщо показник негативу перевищує встановлений поріг.

Результати аналізу зберігаються у **Google Sheets** для подальшого моніторингу.

Проєкт створений як **learning / demo automation system** для демонстрації:

- роботи з API
- AI аналізу тексту
- sentiment analysis
- фільтрації даних
- дедуплікації
- автоматичних алертів

---

## Архітектура Workflow

![Workflow](workflow.png)

Workflow складається з наступних етапів:

1. **Scheduler** — запуск сценарію кожні 180 хвилин  
2. **HTTP Request** — отримання списку товарів через API  
3. **Iterator** — перетворення масиву продуктів у окремі об'єкти  
4. **Filter (Rating < 4)** — відбір товарів з рейтингом менше 4  
5. **Google Gemini AI** — аналіз відгуків та визначення sentiment  
6. **JSON Parse** — парсинг відповіді AI  
7. **Filter (Negative Ratio)** — пропуск лише товарів з високим рівнем негативу  
8. **Deduplication (Google Sheets)** — перевірка, чи товар вже був оброблений  
9. **Google Sheets Insert** — запис результатів аналізу  
10. **Telegram Alert** — відправка повідомлення

---

## Як працює система

### 1. Scheduler

Сценарій запускається **кожні 180 хвилин**.

Це дозволяє регулярно перевіряти нові відгуки товарів без ручного запуску.

---

### 2. Отримання даних (HTTP Request)

Система отримує список товарів через API.

Для демонстрації використовується тестове API:

```
https://dummyjson.com/products
```

API повертає масив продуктів:

```json
{
  "products": [...]
}
```

---

### 3. Iterator

Модуль **Iterator** перетворює масив `products[]` у набір окремих об'єктів, щоб кожен товар можна було обробляти окремо.

---

### 4. Фільтрація за рейтингом

Фільтр пропускає тільки товари з рейтингом:

```
rating < 4
```

Це дозволяє аналізувати лише **проблемні або потенційно проблемні товари**.

---

### 5. AI аналіз відгуків

Відгуки передаються до **Google Gemini AI**, який виконує **sentiment analysis**.

Prompt:

```
You are an assistant that analyzes customer reviews of e-commerce products.

Do not include markdown.
Do not include explanations.
Do not wrap the response in code blocks.

Return ONLY valid JSON in this format:

{
 "sentiment": "positive | neutral | negative",
 "negative_ratio": number,
 "summary": "short explanation"
}

Product: {{title}}
Reviews: {{reviews}}
```

---

### 6. JSON Parse

Модуль **JSON Parse** перетворює відповідь Gemini у структуру даних.

Отримуються поля:

- sentiment
- negative_ratio
- summary

---

### 7. Фільтр негативу

Додатковий фільтр пропускає лише товари, у яких:

```
negative_ratio > 0.5
```

Це означає, що **більше половини відгуків є негативними**.

---

### 8. Дедуплікація

Перед записом даних виконується перевірка:

**Google Sheets → Search Rows**

Умова:

```
sku = {{sku}}
limit = 1
```

Далі застосовується фільтр:

```
Total number of bundles = 0
```

Це означає, що товар **ще не був оброблений раніше**.

---

### 9. Збереження даних

Якщо товар новий, дані записуються у **Google Sheets**.

Зберігаються поля:

- sku
- id
- brand
- price
- title
- ratingReviews
- category
- availabilityStatus
- sentiment
- negative_ratio
- summary

Це дозволяє створювати **таблицю проблемних товарів**.

---

### 10. Telegram Alert

Після запису даних система відправляє **алерт у Telegram**.

Приклад повідомлення:

```
⚠️ Negative Product Reviews Detected

Product: iPhone Case
Brand: Apple
Category: accessories
Negative ratio: 0.62

Summary:
Customers complain about poor durability and low quality materials.
```

---

## Використані технології

- **Make (Integromat)** — automation platform  
- **HTTP Request** — отримання даних з API  
- **Iterator** — обробка масивів  
- **Google Gemini AI** — аналіз тексту  
- **JSON Parse** — парсинг відповіді AI  
- **Google Sheets API** — збереження даних  
- **Telegram Bot API** — сповіщення  

---

## Можливі покращення

- інтеграція з реальним e-commerce API
- автоматичне створення тикетів для support команди
- dashboard для моніторингу sentiment
- відправка звітів у Slack
- класифікація типів проблем (quality / delivery / price)

---

## Setup Notes

Цей сценарій є **демонстраційним прикладом**.

Для використання потрібно:

- налаштувати **Google Gemini API**
- підключити **Google Sheets**
- додати **Telegram Bot Token**
- вказати `YOUR_CHAT_ID`
- вказати `YOUR_SPREADSHEET_ID`
