# Thread-Based Reply Tracking System (n8n)

## Overview
Tracks email replies and updates CRM using:
- Gmail Thread ID
- Email fallback

## Features
- First reply detection
- Follow-up tracking
- Reply history storage
- Auto lead creation

## Tech Stack
- n8n
- Gmail API
- Zoho CRM API

## Workflow Logic
1. Capture email
2. Search by thread
3. Fallback to email
4. Update CRM

## Use Case
Sales teams tracking conversations automatically

📘 WORKFLOW 10 DOCUMENTATION
Thread-Based Reply Tracking System (Upgrade from Email-Based)

🧠 1. PROJECT CONTEXT (READ THIS FIRST)
❌ Problem (Old System - Workflow 9)
	• Email-based tracking 
	• Issues: 
		○ Duplicate replies 
		○ Cannot track conversations properly 
		○ Multiple threads treated as same lead 

✅ Solution (Workflow 10)
Use:

Thread ID (Primary Identifier)
Email (Fallback Identifier)

🎯 Final Goal
Automatically:
	1. Capture incoming emails 
	2. Identify correct lead using: 
		○ Thread ID (primary) 
		○ Email (fallback) 
	3. Update CRM with: 
		○ Reply count 
		○ Reply history 
		○ Last reply time 
	4. Create lead if not found 

🧩 2. WORKFLOW ARCHITECTURE
🔥 Core Logic

Gmail Trigger
   ↓
Edit Fields (normalize data)
   ↓
Search CRM by Thread ID
   ↓
IF (Thread Found?)
   ├── YES → Update CRM (First / Follow-up)
   └── NO → Search by Email
                 ↓
             IF (Email Found?)
             ├── YES → Update CRM
             └── NO → Create Lead

⚙️ 3. NODE-BY-NODE CONFIG (READY TO COPY)

🟢 NODE 1: Gmail Trigger
Purpose:
Capture incoming emails
Config:
	• Poll: Every minute 
	• Mode: Full data (not simple) 

🟢 NODE 2: Edit Fields
Purpose:
Normalize incoming data
Fields:

thread_id = {{ $json.threadId }}

email = {{ $json.from.value[0].address }}

body = {{ $json.text }}

normalized_email = {{ $json.from.value[0].address.toLowerCase().trim() }}

🟢 NODE 3: Thread ID Search
HTTP URL:

={{ "https://www.zohoapis.in/crm/v2/Leads/search?criteria=(Thread_ID:equals:'" + $json.thread_id + "')" }}
Important:
	• Enable FULL RESPONSE 
	• Enable NEVER ERROR 

🟢 NODE 4: IF (Thread Found)
Condition:

{{ $json.statusCode === 200 }}

🔥 THREAD FLOW (MAIN FLOW)

🟢 NODE 5: IF2 (First vs Follow-up)
Condition:

{{ ($json.body.data[0].Reply_Count ?? 0) === 0 }}

🟢 NODE 6A: FIRST REPLY UPDATE
Key Fields:

Thread_ID = {{ $node["Edit Fields"].json.thread_id }}

Reply_Count = {{ ($json.body.data[0].Reply_Count || 0) + 1 }}

Reply_Status = "First Reply"

Reply History Logic:

{{ 
(() => {
  const existing = $json.body.data[0].Reply_History || "";
  const newReply = $node["Edit Fields"].json.body;

  const formattedNew = 
    "-----\n" +
    "Time: " + new Date().toLocaleString() + "\n" +
    "Reply:\n" + newReply;

  const combined = existing 
    ? existing + "\n" + formattedNew 
    : formattedNew;

  const parts = combined.split("-----");
  const lastFive = parts.slice(-5).join("-----");

  return lastFive.trim();
})()
}}

🟢 NODE 6B: FOLLOW-UP UPDATE
Same as above, but:

Reply_Status = "Follow-up"

🔁 EMAIL FALLBACK FLOW

🟢 NODE 7: Email Search
URL:

=https://www.zohoapis.in/crm/v2/Leads/search?criteria=(Email:equals:{{ $node["Edit Fields"].json.normalized_email }})

🟢 NODE 8: IF1 (Email Found?)

{{ $json.data && $json.data.length > 0 }}

🔥 CRITICAL NODE
🟢 NODE 9: IF_EMAIL_SAFE
Condition:

{{ $json.data[0].Thread_ID }}

🔥 CORRECT LOGIC (IMPORTANT)
Branch	Meaning	Action
TRUE	Thread exists in CRM	Update CRM
FALSE	Thread missing	Update CRM (initialize)
👉 BOTH must update CRM

🟢 NODE 10A: Update lead1 (TRUE branch)
Used when:
	• Email match 
	• Thread already exists 

🟢 NODE 10B: Update lead (FALSE branch)
Used when:
	• Email match 
	• Thread NOT exists 

🟢 NODE 11: Create Lead
If email not found:

Company = QuickSolveAI

Last Name = {{ email }}

🧪 4. TESTING

TEST CASE 1
Input:
	• Existing lead 
	• Same thread 
Expected:
	• Reply_Count +1 
	• History updated 

TEST CASE 2
Input:
	• Existing lead 
	• New thread 
Expected:
	• Thread updated 
	• Reply tracked 

TEST CASE 3
Input:
	• New email 
Expected:
	• New lead created 

⚠️ 5. COMMON ERRORS

❌ Error 1: Invalid Query
Cause:

Wrong Zoho syntax
Fix:

Use quotes OR no quotes correctly

❌ Error 2: Empty Response
Cause:

Wrong email mapping
Fix:

{{ $node["Edit Fields"].json.normalized_email }}

❌ Error 3: Reply Count Not Increasing
Cause:

Wrong JSON path
Fix:

$json.data[0].Reply_Count

🔍 6. POST EXECUTION CHECK
After workflow runs:
	• CRM updated? 
	• Reply_Count correct? 
	• Reply_History appended? 
	• Thread_ID saved? 



🚀 8. DEPLOYMENT
	• Enable workflow 
	• Keep n8n active 
	• Use cloud or VPS 

🎯 9. CLIENT HANDOVER
Give:
	• Workflow JSON 
	• README 
	• Demo video 
	• Credentials guide 

🎤 10. CLIENT DEMO SCRIPT
Say this:
	“Whenever a customer replies, system automatically identifies the correct lead using thread ID. If not found, it intelligently falls back to email. Every reply is tracked with history and count.”

✅ 11. SIGN-OFF CHECK
Ask client:
	• Replies updating? 
	• Duplicate leads avoided? 
	• History visible? 

🔐 12. CREDENTIALS REQUIRED
	• Gmail OAuth 
	• Zoho CRM OAuth 

🔒 13. SECURITY
	• Never expose tokens 
	• Use environment variables 
	• Limit API access 

✅ 14. COMPLETION CHECK
Project is DONE when:
	• All 3 test cases pass 
	• No duplicate leads 
	• Reply tracking works 100% 

