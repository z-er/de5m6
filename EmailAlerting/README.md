# 📧 Azure Function Email Alerting for Pipeline Failures

This project provides an Azure Function in Python that sends email alerts via Gmail SMTP when triggered by a pipeline failure (e.g., from Azure Data Factory or Synapse Pipelines).

It’s designed to be lightweight, secure, GitHub-friendly, and flexible — ideal for use in automated alerting for data engineering workflows.

---

## ✅ Why Use This Approach Instead of Logic Apps?

While **Azure Logic Apps** offer a no-code option for sending alerts, this **code-based Azure Function** approach has several important advantages:

| Feature                            | Azure Function (This Project) ✅ | Logic Apps ❌ |
|------------------------------------|----------------------------------|----------------|
| Full version control in GitHub     | ✅ Yes                           | ⚠️ Limited (via ARM templates) |
| Custom email formatting            | ✅ Full control (plain/HTML, dynamic logic) | 🚫 Limited UI-based formatting |
| Easily testable locally            | ✅ Yes                           | 🚫 No |
| Developer-friendly (Python-based)  | ✅ Yes                           | 🚫 Not code-centric |
| Reusable for other workflows       | ✅ Package and reuse as Python module | 🚫 No |
| Fast iterative development         | ✅ Git + CI/CD                   | 🚫 Slower iteration in UI |

This solution is ideal for teams who:
- Work in CI/CD environments
- Want complete control over email content
- Want reusable, testable alerting logic
- Use infrastructure-as-code or GitOps

---

## 🔍 What the Code Does

The main Azure Function is triggered by an HTTP POST request.

### 🔗 Trigger Input (Example)

```json
{
  "pipeline": "Sales_ETL_Pipeline",
  "error": "FileNotFoundError in step 3",
  "timestamp": "2025-08-07 14:03:12"
}
```

### 🧠 Code Flow

1. Parses the incoming JSON payload
2. Generates an email message with the pipeline name, error details, and timestamp
3. Sends the email via Gmail SMTP using `smtplib`

### 📤 Email Example

```
Subject: [ALERT] Sales_ETL_Pipeline Failed

Body:
ALERT: Pipeline Failure Detected 🚨

Pipeline: Sales_ETL_Pipeline  
Timestamp: 2025-08-07 14:03:12

Error Details:
FileNotFoundError in step 3

Please investigate immediately.

- DataOps Team
```

---

## 🔐 Gmail 2FA and App Passwords: Why It Was Needed

### ❌ Why Regular Gmail SMTP Login Fails

Google **no longer allows basic authentication (username + password)** for Gmail SMTP connections. Even if you turn off 2FA, Gmail **blocks access to "less secure apps"** by default.

Attempts to use your regular Gmail password for SMTP will result in:

```
(535, b'5.7.8 Username and Password not accepted...')
```

### ✅ Solution: Enable 2FA + App Password

1. **Enable 2-Step Verification** on your Gmail account  
   Go to: https://myaccount.google.com/security → *"2-Step Verification"*

2. **Generate an App Password**  
   After 2FA is enabled:
   - Go to the **App Passwords** section
   - Choose "Mail" as the app
   - Choose "Other" for the device name (e.g., "Azure Function SMTP")
   - Click "Generate"
   - Copy the 16-character password (e.g., `abcd efgh ijkl mnop`)

3. **Use App Password in the Function**
   This password acts like a special access token, allowing SMTP without full login.

---

## 📦 Environment Variables Used

These values are stored securely in the Azure Function Configuration (not in code):

| Variable         | Description                            |
|------------------|----------------------------------------|
| `EMAIL_SENDER`   | Gmail address used to send alerts      |
| `EMAIL_PASSWORD` | Gmail **App Password**, not real one   |
| `EMAIL_RECIPIENT`| Destination email address (or list)    |

---

## 🚀 Deployment Overview

1. Code is stored in GitHub and deployed via CI/CD or manual push
2. Azure Function is triggered via HTTP request (e.g., from a pipeline)
3. Sends an alert email via Gmail SMTP
4. Easily reusable for any other alerting use case

---

## 🔧 To Trigger the Alert from a Pipeline

In **Azure Data Factory or Synapse Pipelines**:
- Add a **Web Activity** after failure
- Configure it to POST to your Function URL with JSON body like:

```json
{
  "pipeline": "MyPipeline",
  "error": "Step X failed",
  "timestamp": "2025-08-07 16:42:00"
}
```

---

## 🛡 Security Notes

- The Gmail App Password is stored as a secure environment variable
- You can also store it in **Azure Key Vault** and retrieve it securely at runtime
- This setup avoids using any personal passwords or insecure practices

---

## 📁 Recommended Folder Structure

```
send_email_function/
├── SendEmail/              # Function code
│   ├── __init__.py
│   └── function.json
├── requirements.txt
├── local.settings.json     # For local testing (ignored in deployment)
├── host.json
└── README.md
```

---

## 🤝 Contributions

PRs welcome if you want to:
- Add HTML email support
- Add Key Vault integration
- Add logging/reporting
