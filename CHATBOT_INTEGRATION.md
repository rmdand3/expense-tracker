# Chatbot Integration Guide

This guide explains how to integrate the Expense Tracker application with chatbot platforms like Microsoft Teams and WhatsApp to enable users to track expenses, debts, and savings through conversational interfaces.

## Table of Contents
1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Microsoft Teams Integration](#microsoft-teams-integration)
4. [WhatsApp Integration](#whatsapp-integration)
5. [API Endpoints for Chatbots](#api-endpoints-for-chatbots)
6. [Natural Language Processing](#natural-language-processing)
7. [Security Considerations](#security-considerations)

---

## Overview

Integrating the Expense Tracker with chatbots allows users to:
- Add expenses through natural language commands
- Check balance and summary statistics
- View recent transactions
- Add debts and savings
- Get spending reports by category

### Benefits
- **Convenience**: Add expenses on-the-go without opening the web app
- **Speed**: Quick commands for frequent actions
- **Accessibility**: Voice-based expense tracking
- **Notifications**: Proactive reminders for pending debts and budgets

---

## Architecture

```
┌─────────────────┐         ┌──────────────────┐         ┌─────────────────┐
│  Chatbot        │         │  Bot Backend     │         │  Expense        │
│  Platform       │◄───────►│  (Bot Framework) │◄───────►│  Tracker API    │
│  (Teams/WhatsApp│         │  + NLP Engine    │         │  (Flask)        │
└─────────────────┘         └──────────────────┘         └─────────────────┘
                                     │
                                     ▼
                            ┌──────────────────┐
                            │  User Auth       │
                            │  & Session Mgmt  │
                            └──────────────────┘
```

### Components Needed:
1. **Chatbot Backend**: Handles message parsing and routing
2. **NLP Engine**: Extracts intent and entities from user messages
3. **API Layer**: RESTful endpoints for chatbot communication
4. **Authentication**: Token-based auth for secure API access

---

## Microsoft Teams Integration

### Prerequisites
- Azure account
- Microsoft Teams admin access
- Bot Framework SDK
- Expense Tracker API hosted on public URL (ngrok for testing)

### Step 1: Create Azure Bot
1. Go to [Azure Portal](https://portal.azure.com)
2. Create a new **Bot Channels Registration**
3. Configure the messaging endpoint: `https://your-domain.com/api/messages`
4. Note the **App ID** and **App Secret**

### Step 2: Set Up Bot Framework

Create a new Python bot using Bot Framework:

```python
# bot.py
from botbuilder.core import BotFrameworkAdapter, TurnContext
from botbuilder.schema import Activity, ActivityTypes
import requests
import json

# Bot Configuration
APP_ID = "your-microsoft-app-id"
APP_PASSWORD = "your-microsoft-app-password"
EXPENSE_TRACKER_API = "https://your-expense-tracker-api.com"

# Create adapter
adapter = BotFrameworkAdapter(APP_ID, APP_PASSWORD)

# User session management
user_sessions = {}

async def on_message_activity(turn_context: TurnContext):
    user_id = turn_context.activity.from_property.id
    message = turn_context.activity.text.lower()

    # Get or create user session
    if user_id not in user_sessions:
        await turn_context.send_activity("Welcome! Please login: /login username password")
        return

    token = user_sessions[user_id]['token']

    # Parse command
    response = await process_command(message, token)
    await turn_context.send_activity(response)

async def process_command(message, token):
    """Process user commands and interact with Expense Tracker API"""

    # Add expense
    if message.startswith("add expense"):
        # Parse: "add expense 500 for lunch at restaurant"
        return await add_expense_command(message, token)

    # Check balance
    elif message in ["balance", "summary", "stats"]:
        return await get_balance(token)

    # Recent expenses
    elif message in ["recent", "last expenses", "show expenses"]:
        return await get_recent_expenses(token)

    # Add debt
    elif message.startswith("add debt"):
        return await add_debt_command(message, token)

    # Add saving
    elif message.startswith("add saving"):
        return await add_saving_command(message, token)

    else:
        return get_help_message()

async def add_expense_command(message, token):
    """Parse and add expense"""
    # Simple parser - can be enhanced with NLP
    # Example: "add expense 500 for lunch at restaurant"

    import re
    match = re.search(r'add expense (\d+\.?\d*) for (.+?) (?:at|in|category) (.+)', message)

    if match:
        amount = float(match.group(1))
        description = match.group(2).strip()
        category = match.group(3).strip().title()

        # Call Expense Tracker API
        headers = {'Authorization': f'Bearer {token}'}
        data = {
            'date': datetime.now().strftime('%Y-%m-%d'),
            'category': category,
            'description': description,
            'amount': amount,
            'payment_method': 'Other',
            'notes': 'Added via Teams bot'
        }

        response = requests.post(
            f'{EXPENSE_TRACKER_API}/api/add_expense',
            json=data,
            headers=headers
        )

        if response.status_code == 200:
            return f"✅ Expense added: ₹{amount} for {description} in {category}"
        else:
            return "❌ Failed to add expense. Please try again."
    else:
        return "Format: add expense [amount] for [description] at [category]"

async def get_balance(token):
    """Get user's financial summary"""
    headers = {'Authorization': f'Bearer {token}'}
    response = requests.get(f'{EXPENSE_TRACKER_API}/api/stats', headers=headers)

    if response.status_code == 200:
        stats = response.json()
        return f"""
📊 **Your Financial Summary**
💸 Total Expenses: ₹{stats['total_expenses']:.2f}
💳 Total Debts: ₹{stats['total_debts']:.2f}
🐷 Total Savings: ₹{stats['total_savings']:.2f}
⚖️ Net Balance: ₹{stats['net_balance']:.2f}
        """
    else:
        return "❌ Failed to retrieve balance."

def get_help_message():
    return """
🤖 **Expense Tracker Bot Commands**

📝 **Add Expense:**
`add expense [amount] for [description] at [category]`
Example: `add expense 500 for lunch at Food & Dining`

💰 **Check Balance:**
`balance` or `summary` or `stats`

📋 **Recent Expenses:**
`recent` or `last expenses`

💳 **Add Debt:**
`add debt [amount] from/to [name]`

🐷 **Add Saving:**
`add saving [amount] in [type]`

❓ **Help:**
`help` - Show this message
    """

# Flask endpoint to receive messages
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/api/messages', methods=['POST'])
def messages():
    if request.content_type == 'application/json':
        body = request.json
    else:
        return jsonify({'error': 'Invalid content type'}), 400

    activity = Activity().deserialize(body)
    auth_header = request.headers.get('Authorization', '')

    async def call_bot(turn_context):
        await on_message_activity(turn_context)

    task = adapter.process_activity(activity, auth_header, call_bot)
    return jsonify({}), 201

if __name__ == '__main__':
    app.run(port=3978)
```

### Step 3: Enable Teams Channel
1. In Azure Bot, go to **Channels**
2. Click on **Microsoft Teams** icon
3. Accept terms and save
4. Test the bot in Teams

### Step 4: Deploy
- Deploy bot backend to Azure App Service or your preferred hosting
- Update messaging endpoint in Azure Bot configuration
- Test all commands in Teams

---

## WhatsApp Integration

### Prerequisites
- Twilio account (or WhatsApp Business API access)
- Verified phone number
- Expense Tracker API hosted on public URL

### Step 1: Set Up Twilio WhatsApp Sandbox

1. Create account at [Twilio](https://www.twilio.com)
2. Go to **Messaging** → **Try it out** → **Send a WhatsApp message**
3. Note your WhatsApp sandbox number
4. Configure webhook URL: `https://your-domain.com/whatsapp/webhook`

### Step 2: Create WhatsApp Bot Handler

```python
# whatsapp_bot.py
from flask import Flask, request
from twilio.twiml.messaging_response import MessagingResponse
import requests
from datetime import datetime

app = Flask(__name__)

EXPENSE_TRACKER_API = "https://your-expense-tracker-api.com"

# In-memory session storage (use Redis in production)
user_sessions = {}

@app.route('/whatsapp/webhook', methods=['POST'])
def whatsapp_webhook():
    """Handle incoming WhatsApp messages"""
    incoming_msg = request.values.get('Body', '').strip().lower()
    from_number = request.values.get('From', '')

    # Create Twilio response
    resp = MessagingResponse()
    msg = resp.message()

    # Process command
    response_text = process_whatsapp_command(incoming_msg, from_number)
    msg.body(response_text)

    return str(resp)

def process_whatsapp_command(message, user_id):
    """Process WhatsApp commands"""

    # Authentication
    if message.startswith('login'):
        parts = message.split()
        if len(parts) == 3:
            username = parts[1]
            password = parts[2]
            return handle_login(user_id, username, password)
        else:
            return "❌ Format: login [username] [password]"

    # Check if user is logged in
    if user_id not in user_sessions:
        return "⚠️ Please login first: login [username] [password]"

    token = user_sessions[user_id]['token']

    # Commands
    if message.startswith('expense'):
        return handle_add_expense(message, token)

    elif message in ['balance', 'summary']:
        return handle_get_balance(token)

    elif message in ['recent', 'list']:
        return handle_recent_expenses(token)

    elif message.startswith('debt'):
        return handle_add_debt(message, token)

    elif message.startswith('saving'):
        return handle_add_saving(message, token)

    elif message == 'help':
        return get_whatsapp_help()

    else:
        return "❓ Unknown command. Send 'help' for available commands."

def handle_login(user_id, username, password):
    """Authenticate user"""
    response = requests.post(
        f'{EXPENSE_TRACKER_API}/login',
        json={'username': username, 'password': password}
    )

    if response.status_code == 200:
        data = response.json()
        user_sessions[user_id] = {
            'username': username,
            'token': data.get('token', 'session-token')
        }
        return f"✅ Welcome {username}! You are now logged in."
    else:
        return "❌ Login failed. Check your credentials."

def handle_add_expense(message, token):
    """Add expense from WhatsApp message"""
    # Parse: "expense 500 lunch Food & Dining"
    parts = message.split(maxsplit=3)

    if len(parts) < 4:
        return "❌ Format: expense [amount] [description] [category]"

    try:
        amount = float(parts[1])
        description = parts[2]
        category = parts[3]

        headers = {'Authorization': f'Bearer {token}'}
        data = {
            'date': datetime.now().strftime('%Y-%m-%d'),
            'category': category,
            'description': description,
            'amount': amount,
            'payment_method': 'Other',
            'notes': 'Added via WhatsApp'
        }

        response = requests.post(
            f'{EXPENSE_TRACKER_API}/api/add_expense',
            json=data,
            headers=headers
        )

        if response.status_code == 200:
            return f"✅ Expense added: ₹{amount}\n{description} - {category}"
        else:
            return "❌ Failed to add expense."

    except ValueError:
        return "❌ Invalid amount."

def handle_get_balance(token):
    """Get financial summary"""
    headers = {'Authorization': f'Bearer {token}'}
    response = requests.get(f'{EXPENSE_TRACKER_API}/api/stats', headers=headers)

    if response.status_code == 200:
        stats = response.json()
        return f"""📊 *Your Summary*

💸 Expenses: ₹{stats['total_expenses']:.2f}
💳 Debts: ₹{stats['total_debts']:.2f}
🐷 Savings: ₹{stats['total_savings']:.2f}
⚖️ Balance: ₹{stats['net_balance']:.2f}"""
    else:
        return "❌ Failed to get balance."

def get_whatsapp_help():
    return """🤖 *Expense Tracker Commands*

📝 *Add Expense:*
expense [amount] [description] [category]
Ex: expense 500 lunch Food & Dining

💰 *Balance:* balance

📋 *Recent:* recent

💳 *Add Debt:* debt [amount] [name]

🐷 *Add Saving:* saving [amount] [type]

🔐 *Login:* login [user] [pass]

❓ *Help:* help"""

if __name__ == '__main__':
    app.run(port=5001)
```

### Step 3: Configure Webhook
1. In Twilio console, set webhook URL
2. Choose **POST** method
3. Save configuration

### Step 4: Test
- Send "help" to your WhatsApp bot number
- Try: "login testuser password123"
- Try: "expense 100 coffee Food & Dining"
- Try: "balance"

---

## API Endpoints for Chatbots

### Required API Modifications

Add these endpoints to `app.py` for chatbot integration:

```python
# Add to app.py

@app.route('/api/auth/token', methods=['POST'])
def get_auth_token():
    """Generate authentication token for chatbot users"""
    data = request.json
    username = data.get('username')
    password = data.get('password')

    users = load_users()

    if username in users and check_password_hash(users[username]['password'], password):
        # Generate token (use JWT in production)
        token = generate_token(username)
        return jsonify({'success': True, 'token': token})

    return jsonify({'success': False, 'message': 'Invalid credentials'}), 401

@app.route('/api/expenses/add', methods=['POST'])
def api_add_expense_token():
    """Add expense with token authentication"""
    token = request.headers.get('Authorization', '').replace('Bearer ', '')
    user_id = verify_token(token)

    if not user_id:
        return jsonify({'success': False, 'message': 'Unauthorized'}), 401

    data = request.json
    try:
        add_expense(user_id, data['date'], data['category'],
                   data['description'], data['amount'],
                   data['payment_method'], data.get('notes', ''))
        return jsonify({'success': True})
    except Exception as e:
        return jsonify({'success': False, 'message': str(e)}), 500

# Token generation and verification helpers
import secrets
import time

tokens = {}  # Use Redis in production

def generate_token(username):
    token = secrets.token_urlsafe(32)
    tokens[token] = {
        'username': username,
        'created_at': time.time()
    }
    return token

def verify_token(token):
    if token in tokens:
        # Check if token is not expired (24 hours)
        if time.time() - tokens[token]['created_at'] < 86400:
            return tokens[token]['username']
    return None
```

---

## Natural Language Processing

### Option 1: Rule-Based Parser
Simple pattern matching for commands (shown in examples above)

### Option 2: Intent Recognition with Rasa

```python
# nlu.yml for Rasa
nlu:
- intent: add_expense
  examples: |
    - add expense 500 for lunch
    - spent 200 on groceries
    - paid 1000 for rent
    - expense 50 coffee

- intent: check_balance
  examples: |
    - what's my balance
    - show summary
    - how much have I spent
    - balance check

- intent: view_expenses
  examples: |
    - show my expenses
    - recent expenses
    - what did I spend on
    - list expenses
```

### Option 3: OpenAI GPT for Intent Extraction

```python
import openai

def parse_intent(message):
    prompt = f"""
    Extract expense information from this message: "{message}"
    Return JSON with: intent, amount, description, category
    """

    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[{"role": "user", "content": prompt}]
    )

    return json.loads(response.choices[0].message.content)
```

---

## Security Considerations

### 1. Authentication
- Use JWT tokens for API authentication
- Implement token expiration and refresh
- Store tokens securely (Redis, encrypted storage)

### 2. Rate Limiting
```python
from flask_limiter import Limiter

limiter = Limiter(app, key_func=lambda: request.remote_addr)

@app.route('/api/messages')
@limiter.limit("10 per minute")
def messages():
    # Handle bot messages
    pass
```

### 3. Data Validation
- Validate all input amounts and categories
- Sanitize user inputs to prevent injection
- Implement maximum expense limits

### 4. Encryption
- Use HTTPS for all API communication
- Encrypt sensitive data in transit
- Store bot credentials in environment variables

### 5. User Verification
- Implement 2FA for chatbot login
- Send OTP via email for verification
- Log all chatbot transactions

---

## Example Conversations

### Microsoft Teams Example
```
User: add expense 500 for lunch at Food & Dining
Bot: ✅ Expense added: ₹500.00 for lunch in Food & Dining

User: balance
Bot: 📊 Your Financial Summary
     💸 Total Expenses: ₹2,500.00
     💳 Total Debts: ₹1,000.00
     🐷 Total Savings: ₹10,000.00
     ⚖️ Net Balance: ₹6,500.00
```

### WhatsApp Example
```
User: expense 200 groceries Shopping
Bot: ✅ Expense added: ₹200
     groceries - Shopping

User: recent
Bot: 📋 Recent Expenses:
     1. ₹200 - groceries (Shopping)
     2. ₹500 - lunch (Food & Dining)
     3. ₹1000 - fuel (Transportation)
```

---

## Deployment Checklist

- [ ] Deploy Expense Tracker API to public URL
- [ ] Set up bot backend (Azure/Heroku/AWS)
- [ ] Configure webhooks in bot platforms
- [ ] Implement token-based authentication
- [ ] Add rate limiting and security measures
- [ ] Test all commands thoroughly
- [ ] Set up monitoring and logging
- [ ] Create user documentation
- [ ] Implement error handling
- [ ] Add analytics tracking

---

## Additional Resources

- [Microsoft Bot Framework Documentation](https://docs.microsoft.com/en-us/azure/bot-service/)
- [Twilio WhatsApp API](https://www.twilio.com/docs/whatsapp)
- [Rasa NLU Documentation](https://rasa.com/docs/)
- [Flask-Limiter for Rate Limiting](https://flask-limiter.readthedocs.io/)

---

## Support

For questions about chatbot integration, please open an issue in the repository or contact the maintainers.

---

Made with ❤️ in India | Currency: Indian Rupees (₹)
