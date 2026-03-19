![n8n](https://img.shields.io/badge/n8n-automation-orange)
![AI](https://img.shields.io/badge/AI-Google%20Gemini-blue)
![Email](https://img.shields.io/badge/Automation-Gmail-red)

# AI Helpdesk Email Automation (n8n Workflow)

🇺🇸 English | 🇺🇦 [Українська](README_UA.md) 

---

## Overview

This project demonstrates an **AI automation workflow for automatically responding to customer emails**, built using **n8n** and **Google Gemini AI**.

The workflow receives customer inquiries via a webhook, classifies them using AI, generates an appropriate HTML response template, and sends emails via **Gmail**.

The system also uses **rate limiting (Wait node)** to prevent sending too many emails at once.

This project was created as a **learning / demo AI automation system** to demonstrate:

- AI-based request classification
- automated email responses
- template-based messaging
- loop processing
- rate limiting
- personalized communication

---

## Workflow Architecture

![Workflow](workflow.png)

The workflow consists of the following stages:

1. **Webhook** — receiving an array of customer inquiries  
2. **customer-info** — converting the array into individual objects  
3. **Google Gemini AI** — classifying the customer message  
4. **category** — parsing the JSON response from the AI  
5. **Loop Over Items** — processing each customer individually  
6. **Switch** — selecting the response template based on category  
7. **HTML Template Generation** — building the email content  
8. **Gmail Send Message** — sending the email  
9. **Wait (3 sec)** — delay between sends (rate limiting)

---

## How the System Works

### 1. Webhook

The workflow receives customer data through a **Webhook**.

For testing purposes:

```
https://reqbin.com
```

Data format:

```json
{
  "body": [
    {
      "name": "John Doe",
      "phone": "123456789",
      "email": "john@email.com",
      "source": "website",
      "message": "How much does your service cost?"
    }
  ]
}
```

---

### 2. Array Processing (customer-info)

The node splits the array into individual objects.

```javascript
const data = $input.all();

const result = data.flatMap(item =>
  item.json.body.map(itemBody => ({
    name: itemBody.name,
    phone: itemBody.phone,
    email: itemBody.email,
    source: itemBody.source,
    message: itemBody.message
  }))
);

return result;
```

---

### 3. AI Classification

The customer message is sent to **Google Gemini AI**.

The AI determines the category:

- **PRICE**
- **SUPPORT**
- **INFO**
- **OTHER**

Prompt:

```
You classify customer messages.

Categories:
PRICE
SUPPORT
INFO
OTHER

Return only JSON:
{
 "category": "PRICE"
}
```

---

### 4. Response Parsing

The **category** node extracts the category from JSON.

```javascript
const gemini = $json.content.parts[0].text;
const parseText = JSON.parse(gemini);

return {
  category: parseText.category
};
```

---

### 5. Loop Processing

The **Loop Over Items** node processes each customer individually.

This allows:

- avoiding bulk email sending
- controlling system load
- adding delays between emails

---

### 6. Template Selection (Switch)

The **Switch** node selects the HTML template based on category:

- **PRICE** → pricing info  
- **SUPPORT** → support contacts  
- **INFO** → company information  
- **OTHER** → default response  

---

### 7. HTML Template (Default)

```html
<h2>Hello, {{ $('customer-info').item.json.name }}</h2>
<p>Thank you for your request. We have received your message:</p>
<blockquote>{{ $('customer-info').item.json.message }}</blockquote>
<p>Our manager will contact you shortly.</p>
<p>Best regards,<br>Support Team</p>
```

---

### 8. Sending Email

The **Gmail node** sends the email:

- **To:** customer email  
- **Subject:** personalized  
- **Message:** HTML template  

---

### 9. Rate Limiting

The **Wait (3 seconds)** node adds a delay between emails.

This helps to:

- avoid Gmail rate limits
- prevent blocking
- ensure stable sending

---

## Technologies Used

- **n8n** — automation workflow platform  
- **Webhook** — receiving requests  
- **JavaScript (Code nodes)** — data processing  
- **Google Gemini AI** — message classification  
- **Switch Node** — template routing  
- **Gmail API** — email sending  
- **Loop + Wait** — rate control  

---

## Possible Improvements

- saving requests to a CRM
- AI-generated responses (not just templates)
- multilingual support
- Slack or Telegram integration
- analytics dashboard

---

## Setup Notes

This workflow is a **demo AI email automation system**.

To use it:

- configure **Google Gemini API**
- connect **Gmail integration**
- import the workflow into **n8n**
