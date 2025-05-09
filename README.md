📲 SMS-to-WhatsApp Relay
A simple solution for sending WhatsApp messages by sending an SMS to a Twilio number.
This uses:

Twilio (to receive SMS)

Azure Logic Apps (for automation)

Meta WhatsApp Cloud API (to send WhatsApp messages)

3CX (for handling replies)

🧭 Overview
🔹 Problem:
3CX’s native WhatsApp integration doesn’t allow initiating conversations — you can only reply to customers who message you first.

🔹 Solution:
Use the Meta WhatsApp Cloud API to send proactive messages (up to 1000 conversations/day) and let 3CX handle replies.

⚙️ How It Works
Send an SMS with the customer’s WhatsApp number + message to a Twilio “service number.”

An Azure Logic App parses the SMS.

The Logic App calls the Meta WhatsApp Cloud API to send the message.

If the customer replies, 3CX receives the reply (via your existing integration).

📋 Prerequisites
Twilio account with 2 phone numbers:

Business number (used by 3CX)

Service number (used for sending SMS commands)

Azure account (Logic App - Consumption plan)

2 Facebook Developer Apps:

App 1: Integrated with 3CX (handles replies)

App 2: Linked to the same WhatsApp number for outbound messaging

Verified WhatsApp Business Account

Meta Phone Number ID & Access Token

Approved WhatsApp message template (for initiating conversations)

🚀 Setup Guide
1️⃣ Twilio Setup
Purchase/configure two Twilio numbers:

Business number (existing, used by 3CX)

Service number (used for sending SMS commands)

In the Twilio Console, configure the service number:

Messaging → “A message comes in”

Method: POST

URL: (your Logic App URL from Step 2)

2️⃣ Azure Logic App Setup
Create a new Logic App (Consumption plan).

Add the trigger:

vbnet
Copy
Edit
When an HTTP request is received
Leave the Request Body JSON Schema blank.

Add the following actions:

🔸 Filter Array (to extract SMS Body)
css
Copy
Edit
From: @triggerBody()?['$formdata']
Condition:

css
Copy
Edit
item()?['key'] is equal to Body
🔸 Compose (to retrieve Body content)
less
Copy
Edit
first(body('Filter_array'))?['value']
This returns:

diff
Copy
Edit
+614xxxxxxxx | Your message here
🔸 Compose (to split WhatsApp number & message)
bash
Copy
Edit
split(outputs('Compose'), '|')
🔸 Initialize Variables (optional for cleaner references)
Name	Type	Value
var1	String	trim(first(outputs('Compose_2')))
var2	String	trim(last(outputs('Compose_2')))

🔸 HTTP (to send WhatsApp message)
Method: POST
URI:

bash
Copy
Edit
https://graph.facebook.com/v22.0/YOUR_PHONE_NUMBER_ID/messages
Headers:

json
Copy
Edit
{
  "Authorization": "Bearer YOUR_ACCESS_TOKEN",
  "Content-Type": "application/json"
}
Body:

json
Copy
Edit
{
  "messaging_product": "whatsapp",
  "to": "@{variables('var1')}",
  "type": "template",
  "template": {
    "name": "your_template_name",
    "language": {
      "code": "en_US"
    }
  }
}
🔸 Notes:

Replace YOUR_PHONE_NUMBER_ID with your Meta phone number ID.

Use your approved template name (e.g., hello_world).

Your access token must be generated from the Meta app linked to your WhatsApp number.

3️⃣ Test the Workflow
Send an SMS from your business Twilio number to your service Twilio number in this format:

kotlin
Copy
Edit
+614xxxxxxxx | Hello, this is a test message!
✅ The Logic App will:

Parse the number/message.

Send the WhatsApp message via Meta’s API.

If the customer replies, 3CX will handle the conversation as normal.

🔍 How It Looks Behind the Scenes
scss
Copy
Edit
Business Number (SMS) ➡️ Service Twilio Number ➡️ Logic App ➡️ Meta API ➡️ Customer (WhatsApp)
                                  ⬆️                                          ⬇️
                                3CX ⬅️ (Replies handled by existing integration)
✅ Key Points & Considerations
Meta allows 1000 outbound conversations/day (can be increased with higher tiers).

Logic Apps on the Consumption Plan are very cost-effective (pay-per-run).

Two Meta apps used:

App 1: connected to 3CX (replies).

App 2: for sending outbound messages via API.

Two Twilio numbers used:

Business number: customer-facing / 3CX integration.

Service number: for triggering Logic App with SMS.

📚 Resources
Twilio Console

Azure Logic Apps

Meta for Developers

WhatsApp Business API Docs

Let me know if you need help setting up message templates, error handling, or logging!

yaml
Copy
Edit

---

### ✅ How to Use:
1. Copy-paste this into your `README.md`.
2. Replace placeholders (`YOUR_PHONE_NUMBER_ID`, `YOUR_ACCESS_TOKEN`, etc.) with your actual values.
3. Add code blocks in GitHub rendering automatically.

Let me know if you need further adjustments!

2/2








