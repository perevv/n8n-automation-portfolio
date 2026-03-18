![n8n](https://img.shields.io/badge/n8n-automation-orange)
![AI](https://img.shields.io/badge/AI-Google%20Gemini-blue)
![CRM](https://img.shields.io/badge/CRM-Airtable-yellow)

# Airtable CRM Lead Management (n8n Workflow)

🇺🇦 Українська | 🇺🇸 [English](README_EN.md)

## Огляд

Цей проєкт демонструє **automation workflow для управління лідами у CRM**, створений за допомогою **n8n** та **Airtable**.

Workflow отримує ліди через webhook, обробляє їх за допомогою JavaScript, визначає пріоритет ліда, класифікує звернення клієнта через **Google Gemini AI**, перевіряє наявність ліда в CRM та виконує **створення або оновлення запису в Airtable**.

Система також відправляє повідомлення у **Telegram**, якщо:

- створено нового клієнта
- оновлено інформацію про існуючого клієнта

Проєкт створений як **learning / demo CRM automation system** для демонстрації:

- webhook automation
- обробки масивів даних
- AI класифікації звернень
- дедуплікації лідів
- інтеграції з CRM
- автоматичних сповіщень

---

## Архітектура Workflow

![Workflow](workflow.png)

Workflow складається з наступних етапів:

1. **Webhook** — отримання масиву лідів із зовнішнього сервісу  
2. **inputData** — розбиття масиву на окремі об'єкти лідів  
3. **priority** — визначення пріоритету ліда та створення timestamp  
4. **Google Gemini AI** — аналіз повідомлення клієнта  
5. **geminiParse** — парсинг JSON відповіді від AI  
6. **allData** — об'єднання всіх даних ліда в один об'єкт  
7. **HTTP Request (Airtable API)** — перевірка існування запису в CRM  
8. **recordIDCheck** — перевірка чи знайдено існуючий запис  
9. **IF Condition**
   - **True** → оновлення існуючого запису  
   - **False** → створення нового запису  
10. **Airtable Update / Create** — оновлення або створення ліда в CRM  
11. **Telegram Notification** — повідомлення про нового або оновленого клієнта

---

## Як працює система

### 1. Webhook

Workflow отримує дані лідів через **Webhook**.

Для тестування використовується сервіс:

```
https://reqbin.com
```

Webhook приймає масив об'єктів:

```json
{
 "body": [
   {
     "name": "John Doe",
     "phone": "123456789",
     "email": "john@email.com",
     "source": "facebook",
     "message": "How much does your service cost?"
   }
 ]
}
```

---

### 2. Обробка масиву даних

Node **inputData** витягує об'єкти з масиву та перетворює їх у окремі записи.

```javascript
const data = $input.all();

const dataBody = data.flatMap(item =>
  item.json.body.map(itemBody => ({
    name: itemBody.name,
    phone: itemBody.phone,
    email: itemBody.email,
    source: itemBody.source,
    message: itemBody.message
  }))
);

return dataBody;
```

---

### 3. Визначення пріоритету

Node **priority** визначає рівень пріоритету залежно від джерела ліда.

Логіка:

- **Facebook / Instagram → Високий**
- **Website → Середній**
- **Інші джерела → Низький**

```javascript
const source = $json.source;

const priority = source === "facebook" | source === "instagram"
  ? "Високий"
  : source === "website"
    ? "Середній"
    : "Низький";

const timestamp = new Date().toISOString();

return {
  priority: priority,
  timestamp: timestamp
};
```

---

### 4. AI класифікація звернення

Повідомлення клієнта аналізується через **Google Gemini AI**.

AI визначає категорію звернення:

- **ЦІНА** — питання про вартість
- **ПІДТРИМКА** — технічна проблема
- **ІНФО** — загальна інформація
- **ІНШЕ** — інші запити

Prompt:

```
Ти класифікуєш звернення клієнтів тільки за такими напрямками:

ЦІНА
ПІДТРИМКА
ІНФО
ІНШЕ

Відповідай ЛИШЕ у валідному JSON форматі:
{
 "category": "ЦІНА"
}
```

---

### 5. Парсинг відповіді AI

Node **geminiParse** витягує категорію з JSON відповіді.

```javascript
const gemini = $json.content.parts[0].text;
const geminiParse = JSON.parse(gemini);

return {
  category: geminiParse.category
};
```

---

### 6. Об'єднання даних

Node **allData** формує повний об'єкт ліда.

```javascript
return {
  name: $('inputData').item.json.name,
  email: $('inputData').item.json.email,
  phone: $('inputData').item.json.phone,
  source: $('inputData').item.json.source,
  message: $('inputData').item.json.message,
  priority: $('priority').item.json.priority,
  category: $("geminiParse").item.json.category,
  timestamp: $("priority").item.json.timestamp
};
```

---

### 7. Перевірка дубліката

Workflow виконує **HTTP Request до Airtable API**.

Використовується параметр:

```
filterByFormula
```

Формула:

```
AND({Email} = "{{ $json.email }}", {Phone} = {{ $json.phone }})
```

Це дозволяє перевірити, чи вже існує клієнт у CRM.

---

### 8. Перевірка record ID

Node **recordIDCheck** визначає, чи існує запис.

```javascript
const records = $json.records;

return {
  exist: records.length > 0,
  recordId: records.length > 0 ? records[0].id : null
};
```

---

### 9. Умовна логіка

Node **IF** перевіряє:

```
recordId is not empty
```

---

#### Якщо запис існує

- запис **оновлюється в Airtable**
- статус змінюється на:

```
InProgress
```

- у Telegram надсилається повідомлення:

```
🔔 UPDATED CUSTOMER INFORMATION
```

---

#### Якщо запис не існує

- створюється **новий запис у Airtable**
- відправляється повідомлення:

```
🔔 NEW CUSTOMER
```

---

### 10. Запис у CRM

У **Airtable** зберігаються такі поля:

- Name
- Email
- Message
- Status
- Timestamp
- Phone
- Source
- Priority
- Category

---

## Використані технології

- **n8n** — automation workflow platform  
- **Webhook** — отримання лідів  
- **JavaScript (Code nodes)** — обробка даних  
- **Google Gemini AI** — AI класифікація звернень  
- **Airtable API** — CRM база даних  
- **HTTP Request** — інтеграція з Airtable  
- **Telegram Bot API** — сповіщення  

---

## Можливі покращення

- lead scoring система
- автоматичний розподіл лідів між менеджерами
- dashboard для аналітики лідів
- автоматичні follow-up повідомлення

---

## Setup Notes

Цей workflow є **демонстраційним прикладом CRM automation**.

Для використання потрібно:

- налаштувати **Airtable API Token**
- створити **Airtable Base**
- підключити **Telegram Bot**
- імпортувати workflow у **n8n**
