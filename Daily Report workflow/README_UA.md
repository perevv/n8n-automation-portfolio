![n8n](https://img.shields.io/badge/n8n-automation-orange)
![JavaScript](https://img.shields.io/badge/Code-JavaScript-yellow)
![Automation](https://img.shields.io/badge/Workflow-Daily%20Report-blue)

# Daily Report Workflow (n8n)

🇺🇦 Українська | 🇺🇸 [English](README_EN.md)

## Огляд

Цей проєкт демонструє **automation workflow для створення щоденного звіту**, створений за допомогою **n8n**.

Workflow автоматично запускається за розкладом, отримує дані з API, обробляє їх за допомогою JavaScript, формує підсумкову статистику та відправляє щоденний звіт у **Telegram**.

Проєкт створений як **learning / demo automation system** для демонстрації роботи з:

- scheduled automation
- HTTP API
- обробкою даних
- нодою Merge
- створенням автоматичних звітів

---

## Архітектура Workflow

![Workflow](workflow.png)

Workflow складається з наступних етапів:

1. **Schedule Trigger** — автоматичний запуск workflow щодня о 09:00  
2. **HTTP Request (Users)** — отримання списку користувачів з API  
3. **HTTP Request (Posts)** — отримання списку постів  
4. **JavaScript обробка даних** — підрахунок користувачів та постів  
5. **Merge** — об'єднання результатів двох гілок workflow  
6. **Summary** — формування підсумкового звіту  
7. **Telegram notification** — відправка щоденного звіту

---

## Як працює система

### 1. Schedule Trigger

Workflow запускається автоматично **щодня о 09:00** за допомогою ноди **Schedule Trigger**.

Це дозволяє створювати регулярні автоматичні звіти без ручного запуску.

---

### 2. Отримання даних (HTTP Requests)

Після запуску workflow виконує два **HTTP запити**:

1️⃣ Отримання списку користувачів  
2️⃣ Отримання списку постів

У цьому прикладі використовується тестове API:

```
https://jsonplaceholder.typicode.com
```

---

### 3. Обробка даних (JavaScript)

#### Node `totalUsers`

Підраховує загальну кількість користувачів.

```javascript
const data = $input.all();

return {
  totalUsers: data.length
};
```

---

#### Node `Code in JavaScript`

Підраховує:

- загальну кількість постів
- кількість постів від користувача з `userId = 1`

```javascript
const data = $input.all();

const countPosts = data.length;

const postsUserId = data.filter(item => item.json.userId === 1);

const countUser1 = postsUserId.length;

return {
  countPosts: countPosts,
  countUserId1: countUser1
};
```

---

### 4. Merge

Node **Merge** об'єднує результати двох гілок workflow.

Це дозволяє передати дані в один потік для подальшої обробки.

---

### 5. Формування звіту (Summary)

Node **summary** об'єднує всі отримані дані та формує фінальний об'єкт звіту.

```javascript
const data = $input.all();

const way1 = data[0].json;
const way2 = data[1].json;

return {
  totalUsers: way1.totalUsers,
  countPosts: way2.countPosts,
  countUserId1: way2.countUserId1,
  timestamp: new Date().toISOString(),
  timestampBoss: new Date().toLocaleString('uk-UA',{
    day: '2-digit',
    month: '2-digit',
    year: 'numeric',
    hour: '2-digit',
    minute: '2-digit',
    timeZone: 'Europe/Kyiv'
  })
};
```

---

### 6. Telegram сповіщення

Після обробки даних workflow відправляє **щоденний звіт у Telegram**.

Приклад повідомлення:

```
📊 Щоденний звіт

Користувачів: 10
Постів всього: 100
Постів від userId 1: 10
Дата: 12.04.2026 09:00
```

Це дозволяє автоматично отримувати актуальну статистику кожного дня.

---

## Використані технології

- **n8n** — automation workflow platform  
- **Schedule Trigger** — запуск workflow за розкладом  
- **HTTP Request** — отримання даних з API  
- **JavaScript (Code node)** — обробка даних  
- **Merge Node** — об'єднання потоків даних  
- **Telegram Bot API** — відправка звітів  

---

## Можливі покращення

- інтеграція з реальною базою даних
- збереження звітів у Google Sheets
- відправка звітів у Slack
- побудова графіків статистики
- створення dashboard

---

## Setup Notes

Цей workflow є **демонстраційним прикладом**.

Для використання потрібно:

- додати **Telegram Bot Token**
- вказати **CHAT_ID**
- імпортувати workflow у **n8n**
