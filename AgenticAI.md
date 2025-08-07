# 📜 Teams Transcription & Microsoft Graph Integration

This guide explains how to capture and retrieve Microsoft Teams **meeting transcripts** using **Microsoft Graph** and **Azure Functions**.

---

## 🎥 Meeting Storage Details

- **Standard Meetings**  
  &nbsp;&nbsp;&nbsp;&nbsp;📂 Transcripts and recordings are saved in the **organizer’s OneDrive**.

- **Channel Meetings**  
  &nbsp;&nbsp;&nbsp;&nbsp;📂 Transcripts and recordings are saved in the **Team’s SharePoint site**, not in an individual's OneDrive.

---

## 🌐 Microsoft Graph Overview

- Microsoft Graph is the **REST API layer** for all Microsoft 365 services (Teams, OneDrive, SharePoint, Outlook, etc.).
- It never runs on its own. Your **bot, app, or service is the caller**.
- When your app calls Graph (e.g., `Get transcripts`), Graph must **verify that the caller has the right permissions** to access that data.

---

## 🚀 Implementation Steps

### **A. Register an Azure AD App**

1. Go to **Azure Portal** → **Azure Active Directory** → **App registrations** → **New registration**.
2. Give it a name: `TeamsTranscriptionBot`.
3. Set **Redirect URI**:  
   &nbsp;&nbsp;&nbsp;&nbsp;`https://your-bot-domain.com/auth/callback`

---

### **B. Once the app is created**

- Note the **Application (client) ID**
- Note the **Directory (tenant) ID**
- Create a **Client Secret** (Certificates & Secrets)

---

### **C. Create API Permissions for the Bot**

1. In the App Registration → **API Permissions**, add:
   - `OnlineMeetings.Read` *(Delegated or Application)*  
   - `OnlineMeetings.ReadWrite` *(if you’ll modify)*  
   - `Calls.AccessMedia.All` *(optional)*  
   - `TeamsMessages.Read`  
   - `Files.Read.All` *(to access transcripts in OneDrive/SharePoint)*
2. **Grant admin consent** for these permissions.

---

### **D. Create a public endpoint to receive the transcript**

**Options (Cheapest → Most Expensive):**

- **Azure Functions (Consumption Plan – Pay as you go)**  
  &nbsp;&nbsp;💰 **CHEAPEST**  
  &nbsp;&nbsp;Includes HTTPS endpoint:  
  &nbsp;&nbsp;`https://<function>.azurewebsites.net/api/graph-webhook`  
  &nbsp;&nbsp;Free for the first **1M executions/month**, then **$0.20 per million executions** + tiny storage cost.

- **Azure Logic Apps (Consumption Plan)**  
  &nbsp;&nbsp;First 4,000 actions/month = ~$0.000025 per action.  
  &nbsp;&nbsp;Can get more expensive at scale (every action = cost).

- **Azure Bot Service (with App Service)**  
  &nbsp;&nbsp;If your bot already uses App Service (Standard tier), you don’t pay extra.  
  &nbsp;&nbsp;Otherwise, Basic B1 plan is **$13/month** (even for 1 notification).

- **Power Automate (Cloud Flows)**  
  &nbsp;&nbsp;Starts at **$15/user/month**.  
  &nbsp;&nbsp;No per-execution model → Expensive unless already licensed.

- **API Management / Gateways**  
  &nbsp;&nbsp;Starts at **$40–$50/month**. Overkill for this use case.

---

### **E. Recommended: Azure Functions (Cheapest)**

We will create **two Azure Functions**:

---

#### **1. `RegisterGraphSubscription` (Timer-triggered)**

- Runs **every 24 hours**.
- Registers or **renews the subscription** with Microsoft Graph.
- Sets Graph up to call your webhook whenever any **meeting changes**.

---

#### **2. `GraphWebhook` (HTTP-triggered)**

- This is the **public endpoint** Graph will call.
- Handles:
  - **Handshake validation** (on first subscription)
  - **Real-time notifications**

- After receiving the `meetingId` in the notification:
  - Check if the meeting ended.
  - Locate and retrieve the transcript from **OneDrive or SharePoint** using Graph API.

---

## 📊 Sequence Diagram

![Alt text](https://github.com/jinan-kordab/AIOps/blob/main/TeamsMeeting_Get_Transcript.png)
