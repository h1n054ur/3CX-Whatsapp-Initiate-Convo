# 3CX-Whatsapp-Initiate-Convo

A simple relay to send WhatsApp messages by sending SMS to a Twilio number.  
It uses:

- Twilio (to receive SMS)  
- Azure Logic Apps (for automation)  
- Meta WhatsApp Cloud API (to send WhatsApp messages)  
- 3CX (for handling replies)

---

## 🧭 Overview

### Problem

3CX’s WhatsApp integration doesn’t allow initiating conversations — only replying to customers who message first.

### Solution

Use Meta’s WhatsApp Cloud API to proactively message up to **1000 users/day**. Once the customer replies, 3CX takes over as normal.

**This setup uses:**

- **Two Facebook Developer Apps**
  - App A: used by 3CX (handles replies, regular setup by directly linking Meta to 3CX — _Don't use the Twilio WhatsApp API!_)
  - App B: used for outbound API calls (initiates messages)

- **Two Twilio Numbers**
  - Main business number (used by 3CX)
  - Secondary service number (used to send SMS commands)

---

## ⚙️ How It Works

1. Send an SMS to the **Twilio service number** in this format:

    ```
    <whatsapp_number> | <message>
    ```

2. Azure Logic App parses the message.

3. Meta API sends a WhatsApp message to the number.

4. If they reply, 3CX receives it via its existing integration.

---

## 📋 Prerequisites

- Verified WhatsApp Business Account  
- Meta access token (from App B)  
- Azure subscription  
- Twilio account with 2 numbers  
- Phone number ID linked to both apps (App A + B)

---

## 🚀 Setup Guide

### 1️⃣ Twilio

- Buy a second Twilio number  
- Under “Messaging” → “A message comes in”  
  Set method to `POST` and point to your Logic App URL  

---

### 2️⃣ Azure Logic App (Consumption Plan)

- Trigger: `When an HTTP request is received`  
- Leave schema blank  
- Add the following actions:

#### Filter Array

**From:**
```
@triggerBody()?['$formdata']
```

**Condition:**
```
item()?['key'] equals 'Body'
```

---

#### Compose (Extract Message Body)

```
first(body('Filter_array'))?['value']
```

---

#### Compose (Split Into Parts)

```
split(outputs('Compose'), '|')
```

---

#### Initialize Variables

| Name  | Type   | Value                                  |
|-------|--------|----------------------------------------|
| var1  | String | `trim(first(outputs('Compose_2')))`    |
| var2  | String | `trim(last(outputs('Compose_2')))`     |

---

### 3️⃣ HTTP Action (Send to Meta API)

**Method:** POST  
**URI:**

```
https://graph.facebook.com/v22.0/<your_phone_number_id>/messages
```

**Headers:**

```
Authorization: Bearer <your_access_token>  
Content-Type: application/json
```

**Body:**

```json
{
  "messaging_product": "whatsapp",
  "to": "@{variables('var1')}",
  "type": "text",
  "text": {
    "body": "@{variables('var2')}"
  }
}
```


---

### 4️⃣ Test the Flow

Send an SMS like:

```
+11234567890 | Hello, this is a test message!
```

✅ Logic App parses the input and sends the WhatsApp message.  
✅ If the user replies, 3CX handles the conversation.

---

## 🔍 Architecture

```
User ➡ SMS ➡ Twilio (Service Number)
           ➡ Azure Logic App ➡ Meta API ➡ WhatsApp Message
                                                ⬇
                                               3CX
```

---

## ✅ Key Points

- Meta allows 1000 outbound conversations/day  
- Logic Apps on the Consumption Plan are very cost-efficient  
- Replies handled by 3CX (App A), messages initiated via API (App B)  
- Two Twilio numbers to avoid webhook collisions  
- Freeform text requires a valid WhatsApp session (24h rule)

---

## 📚 Resources

- https://developers.facebook.com/docs/whatsapp  
- https://portal.azure.com/  
- https://console.twilio.com/

---
