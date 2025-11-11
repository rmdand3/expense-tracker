# Expense Tracker Web Application

A comprehensive expense tracking web application built with Python Flask that allows multiple users to manage their expenses, debts, and savings with Excel-based storage.

## Features

### Core Features
- **Multi-User Support**: Secure user registration and authentication system
- **Expense Management**: Track daily expenses with categories and payment methods
- **Debt Tracking**: Monitor money owed to you and money you owe to others
- **Savings Tracking**: Record savings across different investment types
- **Excel Storage**: All data stored in user-specific Excel files for easy access and portability

### Additional Features
- **Dashboard Analytics**: Visual representation of expenses by category using charts
- **Category Management**: 12 predefined expense categories (Food & Dining, Transportation, Shopping, etc.)
- **Payment Methods**: Track expenses by Cash, Credit Card, Debit Card, UPI, Net Banking, or Other
- **Summary Statistics**: Real-time calculation of total expenses, debts, savings, and net balance
- **Responsive Design**: Mobile-friendly interface for on-the-go expense tracking
- **Currency Support**: All amounts displayed in Indian Rupees (₹)

## Tech Stack

- **Backend**: Python Flask
- **Frontend**: HTML, CSS, JavaScript
- **Data Storage**: Excel (openpyxl)
- **Authentication**: Werkzeug password hashing
- **Charts**: Chart.js
- **Icons**: Font Awesome

## Installation

### Prerequisites
- Python 3.8 or higher
- pip (Python package manager)

### Setup Instructions

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd expense-tracker
   ```

2. **Create a virtual environment**
   ```bash
   python -m venv venv
   ```

3. **Activate the virtual environment**
   - On Windows:
     ```bash
     venv\Scripts\activate
     ```
   - On macOS/Linux:
     ```bash
     source venv/bin/activate
     ```

4. **Install required packages**
   ```bash
   pip install -r requirements.txt
   ```

5. **Run the application**
   ```bash
   python app.py
   ```

6. **Access the application**
   Open your browser and navigate to:
   ```
   http://localhost:5000
   ```

## Usage

### First Time Setup

1. **Register a new account**
   - Navigate to the registration page
   - Enter username, email, and password
   - Click "Register"

2. **Login**
   - Enter your credentials
   - Click "Login"

3. **Start tracking**
   - Use the dashboard to add expenses, debts, or savings
   - View your financial summary in real-time

### Adding Entries

#### Add Expense
1. Click "Add Expense" button
2. Fill in the details:
   - Date
   - Category (select from 12 categories)
   - Description
   - Amount in ₹
   - Payment method
   - Optional notes
3. Click "Add Expense"

#### Add Debt
1. Click "Add Debt" button
2. Fill in the details:
   - Date
   - Creditor/Debtor name
   - Type (Owed to me or I owe)
   - Amount in ₹
   - Status (Pending/Partially Paid/Paid)
   - Due date (optional)
   - Optional notes
3. Click "Add Debt"

#### Add Saving
1. Click "Add Saving" button
2. Fill in the details:
   - Date
   - Type (FD, Mutual Fund, Stocks, PPF, etc.)
   - Description
   - Amount in ₹
   - Goal (optional)
   - Optional notes
3. Click "Add Saving"

## Data Storage

- All user data is stored in Excel files located in the `data/` directory
- Each user has their own Excel file: `data/<username>_expense_tracker.xlsx`
- Excel files contain separate sheets for:
  - **Summary**: Overview of financial status
  - **Expenses**: Detailed expense records
  - **Debts**: Debt tracking information
  - **Savings**: Savings records
  - **Budgets**: Budget planning (future feature)

## Security

- Passwords are hashed using Werkzeug's security features
- Session-based authentication
- User isolation - each user can only access their own data
- **Important**: Change the `app.secret_key` in `app.py` for production use

## Project Structure

```
expense-tracker/
├── app.py                      # Main Flask application
├── requirements.txt            # Python dependencies
├── README.md                   # This file
├── CHATBOT_INTEGRATION.md      # Chatbot integration guide
├── data/                       # User data storage
│   ├── users.json             # User credentials
│   └── *_expense_tracker.xlsx # User Excel files
└── app/
    ├── templates/             # HTML templates
    │   ├── base.html
    │   ├── login.html
    │   ├── register.html
    │   ├── dashboard.html
    │   ├── expenses.html
    │   ├── debts.html
    │   └── savings.html
    └── static/                # Static files
        ├── css/
        │   └── style.css      # Stylesheet
        └── js/
            └── main.js        # JavaScript functions
```

## Expense Categories

The application supports the following expense categories:
1. Food & Dining
2. Transportation
3. Shopping
4. Entertainment
5. Bills & Utilities
6. Healthcare
7. Education
8. Travel
9. Groceries
10. Rent/EMI
11. Investment
12. Other

## Payment Methods

- Cash
- Credit Card
- Debit Card
- UPI (Unified Payments Interface)
- Net Banking
- Other

## Future Enhancements

- Budget planning and alerts
- Recurring expense management
- Export to PDF reports
- Email notifications for due debts
- Multi-currency support
- Mobile app integration
- Data visualization improvements
- Expense sharing between users
- Integration with banking APIs

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is open source and available under the MIT License.

## Support

For issues, questions, or suggestions, please open an issue in the repository.

## Chatbot Integration

For integrating this expense tracker with Microsoft Teams or WhatsApp chatbots, please refer to [CHATBOT_INTEGRATION.md](CHATBOT_INTEGRATION.md).

---

Made with ❤️ in India | Currency: Indian Rupees (₹)
