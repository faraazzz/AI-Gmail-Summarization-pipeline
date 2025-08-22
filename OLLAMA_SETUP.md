# Ollama Setup Guide

This project now uses Ollama (local LLM) instead of OpenAI for email analysis.

## Step 1: Install Ollama

### Windows:
1. Download from: https://ollama.ai/download
2. Install the Windows version
3. Start Ollama from the Start menu

### Alternative (Docker):
```bash
docker run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

## Step 2: Download Llama2 Model

Open a terminal/command prompt and run:
```bash
ollama pull llama2
```

This will download the Llama2 model (about 4GB).

## Step 3: Test Ollama

Test that Ollama is working:
```bash
ollama run llama2 "Hello, how are you?"
```

You should get a response from the model.

## Step 4: Verify API Access

Test the API endpoint:
```bash
curl -X POST http://localhost:11434/api/generate -d '{
  "model": "llama2",
  "prompt": "Hello, how are you?",
  "stream": false
}'
```

## Step 5: Update Environment Variables

Since we're no longer using OpenAI, you don't need the OpenAI API key. Your `.env` file should now look like:

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

## Benefits of Using Ollama:

✅ **No API costs** - runs locally  
✅ **Privacy** - data stays on your machine  
✅ **Offline capability** - works without internet  
✅ **Customizable** - can use different models  

## Troubleshooting:

**If Ollama doesn't start:**
- Check if Docker is running (if using Docker version)
- Restart Ollama application
- Check Windows Defender/firewall settings

**If model doesn't download:**
- Check internet connection
- Try: `ollama pull llama2:7b` (smaller model)
- Check available disk space (need ~4GB)

**If API calls fail:**
- Verify Ollama is running on port 11434
- Check: `http://localhost:11434/api/tags`
- Restart Ollama service
