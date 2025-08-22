# From Inbox to Insights: AI-enhanced email analysis with dlt, Kestra, and Ollama

## Overview

This project demonstrates an automated workflow in Kestra that processes emails from Gmail using AI analysis. It uses `dlt` (Data Loading Tool) to extract emails, Ollama (local LLM) for AI analysis, BigQuery for data storage, and Slack for notifications.

![Overview of project demo](dlt-kestra-demo.png)

The workflow includes:
1. **Data Ingestion**: Extract emails from Gmail using `dlt`
2. **AI Analysis**: Summarize emails and analyze sentiment using Ollama (local LLM)
3. **Data Storage**: Store results in BigQuery
4. **Notifications**: Share insights via Slack <img width="854" height="419" alt="image" src="https://github.com/user-attachments/assets/eaa0ed2c-5f1d-4e5c-b7da-9fbaa35f0886" />


## 🚀 Key Features

- **Local AI Processing**: Uses Ollama with Llama2 model for privacy and cost-effectiveness
- **Automated Workflow**: Scheduled email processing with Kestra orchestration
- **Real-time Analysis**: Email summarization and sentiment analysis
- **Cloud Storage**: BigQuery integration for data persistence
- **Team Collaboration**: Slack notifications for processed emails

## 📋 Prerequisites

### Required Software
- **Docker Desktop** - For running Kestra and Ollama
- **Python 3.11** - For local development and testing
- **Git** - For version control

### Service Accounts & Credentials

1. **Gmail Credentials**
   - IMAP server hostname (default: `imap.gmail.com`)
   - Gmail account email
   - App password (not regular password)
   
   > For app password setup, see [Gmail's official guidelines](https://support.google.com/mail/answer/185833?hl=en)

2. **BigQuery Credentials**
   - Project ID
   - Private key (JSON format)
   - Client email
   
   > Learn more in [dlt's BigQuery documentation](https://dlthub.com/docs/dlt-ecosystem/destinations/bigquery)

3. **Slack Webhook URL**
   - Create a webhook in your Slack workspace
   
   > Follow [Slack's webhook guidelines](https://api.slack.com/messaging/webhooks)

## 🛠️ Installation & Setup

### 1. Clone the Repository
```bash
git clone <your-repository-url>
cd DEProject
```

### 2. Install Ollama

#### Windows:
1. Download from: https://ollama.ai/download
2. Install and start Ollama from Start menu

#### Docker (Alternative):
```bash
docker run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

### 3. Download Llama2 Model
```bash
ollama pull llama2:7b
```

### 4. Test Ollama Installation
```bash
ollama run llama2:7b "Hello, how are you?"
```

### 5. Create Environment File
Create a `.env` file in your project root with base64-encoded credentials:

```env
# Gmail Credentials (Base64 encoded)
SECRET_GMAIL_HOST=imap.gmail.com
SECRET_GMAIL_EMAIL_ACCOUNT=your-email@gmail.com
SECRET_GMAIL_PASSWORD=your-app-password

# BigQuery Credentials (Base64 encoded)
SECRET_BIGQUERY_PROJECT_ID=your-project-id
SECRET_BIGQUERY_PRIVATE_KEY=your-private-key
SECRET_BIGQUERY_CLIENT_EMAIL=your-service-account@your-project.iam.gserviceaccount.com

# Slack Webhook URL (Base64 encoded)
SECRET_SLACK_WEBHOOK_URL=your-slack-webhook-url

# GCP Service Account (for BigQuery plugin)
SECRET_GCP_SA={"type":"service_account","project_id":"your-project-id","private_key_id":"key-id","private_key":"-----BEGIN PRIVATE KEY-----\nyour-private-key\n-----END PRIVATE KEY-----\n","client_email":"your-service-account@your-project.iam.gserviceaccount.com","client_id":"client-id","auth_uri":"https://accounts.google.com/o/oauth2/auth","token_uri":"https://oauth2.googleapis.com/token","auth_provider_x509_cert_url":"https://www.googleapis.com/oauth2/v1/certs","client_x509_cert_url":"https://www.googleapis.com/robot/v1/metadata/x509/your-service-account%40your-project.iam.gserviceaccount.com"}
```

### 6. Setup Docker Compose
Download and configure Docker Compose:
```bash
curl -o docker-compose.yml https://raw.githubusercontent.com/kestra-io/kestra/develop/docker-compose.yml
```

Update the `docker-compose.yml` to include your `.env` file:
```yaml
kestra:
    image: kestra/kestra:develop-full
    env_file:
        - .env
    restart: always
```

### 7. Start Services
```bash
docker compose up -d
```

### 8. Access Kestra UI
Open http://localhost:8080/ in your browser

## 📁 Project Structure

```
DEProject/
├── .env                    # Environment variables (create this)
├── .gitignore             # Git ignore rules
├── docker-compose.yml     # Docker services configuration
├── mainflow.yml          # Main Kestra workflow
├── subflow.yml           # Subflow for email processing
├── requirements.txt      # Python dependencies
├── inbox/               # dlt inbox source configuration
│   ├── __init__.py
│   ├── helpers.py
│   ├── settings.py
│   └── README.md
├── OLLAMA_SETUP.md      # Ollama setup instructions
└── README.md           # This file
```

## 🔧 Configuration

### Kestra Flows Setup

1. **Create Main Flow**:
   - Navigate to Flows → Create in Kestra UI
   - `mainflow.yml`
   - Save as `dlt-kestra-demo`

2. **Create Subflow**:
   - Create another flow
   - `subflow.yml`
   - Save as `process_email`

### Workflow Configuration

The main flow (`dlt-kestra-demo`) includes:
- **dlt_pipeline**: Extracts emails from Gmail to BigQuery
- **check_load_status**: Checks for new emails
- **get_new_emails**: Retrieves new email data
- **sequential**: Processes each email through subflow

The subflow (`process_email`) includes:
- **process_email_ai**: Uses Ollama for AI analysis
- **dlt_load_result**: Stores processed data in BigQuery
- **send_to_slack**: Sends notifications

## 🚀 Usage

### Manual Execution
1. Open Kestra UI (http://localhost:8080/)
2. Navigate to your flow
3. Click "Execute" to run manually

### Scheduled Execution
The workflow runs automatically:
- **Schedule**: Every hour, 9 AM - 6 PM, Monday-Friday
- **Trigger**: Cron expression `0 9-18 * * 1-5`

### Monitoring
- View execution logs in Kestra UI
- Check BigQuery for stored data
- Monitor Slack for notifications

## 📊 Data Flow

1. **Email Extraction**: dlt connects to Gmail and extracts emails
2. **Data Storage**: Raw emails stored in BigQuery `messages_data.my_inbox`
3. **AI Processing**: Ollama analyzes each email for:
   - Summary (30 words max)
   - Sentiment (positive/negative/neutral)
4. **Results Storage**: Processed data stored in `messages_data.processed_emails`
5. **Notifications**: Slack receives formatted messages with insights

## 🔍 Troubleshooting

### Ollama Issues
```bash
# Check if Ollama is running
curl http://localhost:11434/api/tags

# Restart Ollama
ollama serve

# Test model
ollama run llama2:7b "Test message"
```

### Kestra Issues
```bash
# Check Docker containers
docker ps

# View logs
docker compose logs kestra

# Restart services
docker compose restart
```

### BigQuery Issues
- Verify service account permissions
- Check project ID and dataset names
- Ensure credentials are properly encoded

## 📈 Benefits of Ollama Integration

✅ **Cost Effective**: No API costs - runs locally  
✅ **Privacy**: Data stays on your machine  
✅ **Offline**: Works without internet connection  
✅ **Customizable**: Can use different models  
✅ **Fast**: Local processing reduces latency  

## 📝 Version Information

- **Python**: 3.11
- **dlt**: >=0.3.19, <0.4
- **Kestra**: develop-full (latest)
- **Ollama**: Latest stable
- **Llama2 Model**: 7B parameter version
- **Docker**: Latest stable


**Note**: This project uses local AI processing with Ollama, ensuring your email data remains private and secure while providing powerful analysis capabilities.
