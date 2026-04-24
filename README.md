
# Wazuh to Telegram Alert Automation using n8n

## 📌 Overview
This project demonstrates a real-time Security Operations Center (SOC) alerting mechanism. It captures security alerts from **Wazuh** via webhooks and instantly forwards them to a **Telegram** group or private chat using **n8n** (a node-based workflow automation tool). This ensures security engineers receive critical alerts directly on their mobile devices.

## 🚀 Prerequisites
* **Docker** installed on your system.
* A **Telegram Account**.
* **Wazuh Manager** (optional, for final integration).

---

## 🛠️ Step-by-Step Implementation Guide

### Step 1: n8n Docker Setup
Run n8n locally using Docker. Execute the following command in your terminal:

```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -e N8N_ENFORCEMENT_AGREEMENT=true \
  -v n8n_data:/home/node/.n8n \
  n8nio/n8n
```

### Step 2: Create a Telegram Bot & Get IDs
To send messages to Telegram, you need a Bot and a Chat ID.

**1. Create the Bot via BotFather:**
* Open Telegram and search for `@BotFather`.
* Send `/newbot` and follow the prompts to give your bot a name and username.
* Once created, BotFather will give you an **API Token**. Copy and save this securely.

**2. Get Your Chat ID:**
* Search for `@userinfobot` or `@GetIDs Bot` on Telegram and click Start to get your personal Chat ID.
* *(Alternatively, for a group, add the bot to the group, send a test message, and use the Telegram API to fetch the Group Chat ID).*

### Step 3: Access n8n & Create a Webhook
1. Open your browser and navigate to `http://localhost:5678`.
2. Create a new workflow.
3. Click the **+** button, search for **Webhook**, and add the node.
4. Set the **HTTP Method** to `POST`.
5. Copy the **Test URL**.
6. Click on **Listen for test event** to keep it active.

### Step 4: Integrate Telegram in n8n Workflow
Now, configure n8n to format the incoming Wazuh data and push it to Telegram.

**1. Add the Telegram Node:**
* Click the **+** icon next to your Webhook node, search for **Telegram**, and add it.

**2. Authenticate the Connection:**
* In the Telegram node settings, click to create new credentials.
* Paste the **API Token** you got from BotFather and save.

**3. Configure the Action:**
* **Resource:** Select `Message`.
* **Operation:** Select `Send Message`.
* **Chat ID:** Enter your Telegram Chat ID obtained in Step 2.

**4. Map the Dynamic Data:**
* In the **Text** field, use n8n expressions to format the incoming JSON data from Wazuh. You can use a structured template like this:

```text
🚨 *Critical Wazuh Alert!* 🚨
*Rule Description:* {{$json.body.rule.description}}
*Severity Level:* {{$json.body.rule.level}}
*Agent Name:* {{$json.body.agent.name}}
```

* Save the node and toggle the workflow at the top right to **Active**.

### Step 5: Configure Wazuh to Trigger n8n
To make Wazuh send alerts to n8n automatically, add this integration block to your Wazuh Manager's `ossec.conf` file:

```xml
<integration>
  <name>custom-n8n</name>
  <hook_url>http://YOUR_N8N_IP:5678/webhook/YOUR_WEBHOOK_ID</hook_url>
  <level>7</level>
  <alert_format>json</alert_format>
</integration>
```
*Restart the Wazuh manager to apply the changes.*

### Step 6: Test the Pipeline via API (cURL)
To verify that the webhook is receiving data and forwarding it to Telegram, send a POST request using your terminal.

*Replace `YOUR_WEBHOOK_ID` with the actual ID from your n8n Webhook Test URL.*

```bash
curl -X POST http://localhost:5678/webhook-test/YOUR_WEBHOOK_ID \
     -H "Content-Type: application/json" \
     -d '{
           "rule": {
             "description": "Critical: SSH Brute Force Attempt Detected",
             "level": 10
           },
           "agent": {
             "name": "Web-Server-01",
             "ip": "192.168.1.100"
           }
         }'
```
If successful, your Telegram bot will instantly send you the formatted alert message!

---

## 👨‍💻 Author
**Md. Shafiur Rahman**
* GitHub: [@MdShafiurRahman0](https://github.com/MdShafiurRahman0)
* Email: shafiur.rahman.shadat@gmail.com
```
