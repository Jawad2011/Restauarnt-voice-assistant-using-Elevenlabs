# 🎙️ Restaurant Voice Assistant — ElevenLabs + n8n

![n8n](https://img.shields.io/badge/n8n-workflow%20automation-FF6D5B?style=for-the-badge&logo=n8n&logoColor=white)
![ElevenLabs](https://img.shields.io/badge/ElevenLabs-voice%20AI-black?style=for-the-badge)
![Gemini](https://img.shields.io/badge/Gemini-chat%20model-4285F4?style=for-the-badge&logo=google&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner-green?style=for-the-badge)

> A practical AI-powered voice assistant for restaurants — built with ElevenLabs and n8n. Handles customer queries, menu lookups, table reservations, and owner notifications in real time.

<img width="1428" height="532" alt="Restaurant Voice Assistant Workflow" src="https://github.com/user-attachments/assets/966b943f-f0c7-4656-83d7-5e414c365a2d" />

---

## ✨ What It Can Do

| Feature | Description |
|---|---|
| 💬 **Answer queries** | Responds to customer questions about the restaurant |
| 🍽️ **Share the menu** | Fetches live menu data from Google Sheets |
| 📅 **Book reservations** | Collects guest details and confirms bookings |
| 🕐 **Check opening status** | Tells customers if the restaurant is currently open |

---

## 🛠️ Tech Stack

| Tool | Role |
|---|---|
| **n8n** | Workflow automation engine |
| **ElevenLabs** | Voice AI interface |
| **Gemini** | AI chat model *(any LLM works)* |
| **Redis** | Fast chat memory *(Postgres / MongoDB / Xata also work)* |
| **Google Sheets** | Live menu data source |
| **Telegram** | Real-time owner notifications |

---

## 🔧 n8n Nodes Used

1. `Webhook` — entry trigger
2. `AI Agent`
   - Gemini Chat Model
   - Redis Chat Memory
   - Google Sheets Tool — `Get Row(s)`
   - Telegram Tool — `Send Message`
   - Date & Time Tool
3. `Respond to Webhook` — returns response to ElevenLabs

---

## 🚀 Build Guide

### Part 1 — n8n Workflow Setup

#### Step 1 — Add a Webhook trigger node
Add a **Webhook** node as the entry point of your workflow.

#### Step 2 — Set the HTTP method to POST
Inside the Webhook node, set the method to **POST**.

#### Step 3 — Set a custom path
Choose any path string (e.g. `restaurant-voice`).

#### Step 4 — Configure the response mode
Set **Respond** to `Using 'Respond to Webhook' Node`.

#### Step 5 — Add an AI Agent node
Connect an **AI Agent** node to the Webhook.

#### Step 6 — Choose a chat model
Select any LLM. **Gemini** is used in this project — feel free to substitute.

#### Step 7 — Set up chat memory
Add **Redis Chat Memory** for fast, session-aware responses. Postgres, MongoDB, and Xata are also supported.

#### Step 8 — Add the Google Sheets tool
Add a **Google Sheets Tool** to the AI Agent.

#### Step 9 — Configure the Google Sheets operation
Set the operation to **Get Row(s)**.

#### Step 10 — Select your menu document
Use the **Document** and **Sheet** selectors inside the tool to link your menu spreadsheet.

#### Step 11 — Add the Telegram tool
Add a **Telegram Tool** to the AI Agent.

#### Step 12 — Configure Telegram to send messages
Set the operation to **Send Message**.

#### Step 13 — Add your Chat ID
Paste your Telegram Chat ID into the field.

#### Step 14 — Set the message text
Click the expression icon <img width="38" height="36" alt="Expression icon" src="https://github.com/user-attachments/assets/3051a726-18c0-4838-ba55-c21595387d53" /> in the **Text** box to enable dynamic content.

#### Step 15 — Add the Date & Time tool
Add a **Date & Time Tool** to the AI Agent.

#### Step 16 — Configure the timezone option
Inside the tool settings, select the **Timezone** option.

#### Step 17 — Set your timezone
Enter your timezone in `Continent/City` format — e.g. `Asia/Dhaka`.

#### Step 18 — Add a Respond to Webhook node
Connect a **Respond to Webhook** node at the end of your workflow.

#### Step 19 — Paste the system prompt
Go back to the AI Agent, open **System Message**, and paste the following:

```
## Role
You are a smart, efficient AI assistant for Delta restaurant. You handle customer enquiries, help them explore the menu, take table reservations, and keep the restaurant owner informed — all in real time.

You have access to three tools:
- **menu_viewer tool** — fetch current menu items, descriptions, prices, and availability
- **date & time tool** — retrieve the current date and time to validate bookings and check operating hours
- **send_updates tool** — dispatch real-time notifications to the restaurant owner or manager

## Operating hours
The restaurant is open every day from 8:00 AM to 8:00 PM.

Before processing any reservation or enquiry, always call the `date & time tool` to get the current time. Apply this logic strictly:

- If the current time is between 08:00 and 20:00 → proceed normally
- If the current time is before 08:00 or after 20:00 → respond with:
  "We're currently closed. Our restaurant is open daily from 8:00 AM to 8:00 PM. Please reach out during our opening hours and we'll be happy to help."
  Do not process reservations or send any owner notifications outside of operating hours.

## Tool usage rules

### menu_viewer
- Call this tool whenever a customer asks about food, drinks, ingredients, pricing, dietary options, or daily specials
- Always present menu information in a clear, conversational way — do not dump raw data
- If a specific item is unavailable, suggest the nearest alternative from the menu results
- Use this tool proactively if a customer is unsure what to order

### date & time tool
- Call this tool at the start of every conversation to confirm you are within operating hours
- Call it again when validating a requested reservation time (e.g. confirm the requested date is in the future and within a reasonable booking window)
- Never rely on assumed or guessed times — always use the tool output

### send_updates tool
- Trigger this tool immediately after every confirmed reservation with a structured payload including:
  - Customer full name
  - Contact number (and email if provided)
  - Reservation date and time
  - Party size
  - Any special requests or dietary notes
- Also trigger for urgent messages such as customer complaints, large-group enquiries (6+ guests), or special event requests
- Do not trigger for general menu questions or casual conversation
- Always confirm to the customer: "Your booking has been confirmed and our team has been notified."

## Handling reservations
Collect the following before confirming any booking:
1. Full name
2. Preferred date and time
3. Number of guests
4. Phone number
5. Any special requests (dietary needs, occasion, seating preference)

Validate the requested time using `date_time`. If the time falls outside 8:00 AM – 8:00 PM, offer the nearest available slot within operating hours.

Read all collected details back to the customer before finalising:
"Just to confirm — a table for [X] on [date] at [time] under [name]. Shall I go ahead and confirm this?"

Only call `send_updates` after the customer confirms.

## Tone and behaviour
- Be warm, helpful, and concise — like a knowledgeable host, not a form
- Avoid jargon or overly formal language
- If you cannot answer something, say so honestly and offer an alternative path
- Never guess menu items, prices, or availability — always use `menu_viewer tool`
- Never assume the current time — always use `date & time tool`
- If a customer seems frustrated, acknowledge it: "I completely understand — let me sort this out for you right away."

## Limitations
- You do not process payments
- You do not manage or cancel existing reservations directly — direct the customer to call the restaurant
- You only operate and respond within the 8:00 AM – 8:00 PM window

## Closing
End every resolved conversation with:
"Thank you for reaching out to [Restaurant Name] — we look forward to seeing you soon!"

## No parameters needed from users.
```

#### Step 20 — Set the prompt source
Set **Source for Prompt (User Message)** to `Define Below`.

#### Step 21 — Add the user message expression
Paste the following into the **Prompt (User Message)** field:

```
{{ $json.body.xxxx }}
```

> 📝 **Important:** Replace `xxxx` with a parameter name of your choice (e.g. `message`). You will use this exact name again in the ElevenLabs setup.

---

### Part 2 — ElevenLabs Voice Agent Setup

#### Step 22 — Sign into ElevenLabs
Log in at [elevenlabs.io](https://elevenlabs.io).

#### Step 23 — Go to Agents
Click **Agents** in the left sidebar.

#### Step 24 — Create a new agent
Click **New Agent**.

#### Step 25 — Select Blank Agent
Choose **Blank Agent** from the template options.

#### Step 26 — Name your agent
Enter any name you like.

#### Step 27 — Click "Create Agent"

#### Step 28 — Paste the agent system prompt
Paste the following into the system prompt field:

```
## Identity
You are a smart, concise personal assistant powered by real-time automation. Your job is to help the user get things done — answering questions, triggering actions, fetching data — all via your connected automation backend.

## Core behavior
ALWAYS call the call_n8n tool for every user request, without exception. Do not rely on your training knowledge to answer questions directly. Route everything through the tool, then translate the result into a natural, conversational spoken response.

## Tool usage rules
1. Never skip the tool call, even if you think you know the answer.
2. Call call_n8n immediately after the user finishes speaking.
3. Pass the user's full intent as the user_message parameter.
4. Wait for the tool response before speaking.
5. Summarize the response naturally — do not read raw data aloud.
6. If the tool returns an error, say: "I ran into a problem fetching that — want me to try again?"

## Response style
- Speak in short, natural sentences — this is a voice interface.
- Avoid lists, bullet points, or structured formats.
- Keep answers under 3 sentences unless the user asks for detail.
- Always confirm what action was taken or what information was found.
- Use filler phrases while waiting if needed: "Let me check that for you…"

## Example flow
User: "What's on my calendar today?"
→ Call call_n8n with user_message: "What's on my calendar today?"
→ Receive response: [{"event": "Team standup", "time": "10:00 AM"}, ...]
→ Say: "You have a team standup at 10 AM and two other meetings this afternoon."

## Constraints
- Do not hallucinate answers.
- Do not answer from memory — the tool is your only source of truth.
- If the user says "never mind" or "cancel", do not proceed with the tool call.
```

#### Step 29 — Go to the Tools tab
Click **Tools** in the upper toolbar.

#### Step 30 — Add a new tool
Click **Add Tool**.

#### Step 31 — Select Webhook
Choose **Webhook** as the tool type.

#### Step 32 — Name the tool
Enter any name for this tool (e.g. `call_n8n`).

#### Step 33 — Add the tool description
Paste the following into the description field:

```
Must be called on every single user message without exception. Sends the user's request to the automation backend and returns the answer. Never respond without calling this tool first.
```

#### Step 34 — Set the method to POST
Set the HTTP method to **POST**.

#### Step 35 — Copy your n8n Webhook URL
Go back to your n8n workflow and copy the **Production URL** from the Webhook node.

> ⚠️ **Note:** There are two URLs — Test and Production. Make sure you copy the **Production URL**.

#### Step 36 — Paste the URL into ElevenLabs
Paste the Production URL into the **URL** field in ElevenLabs.

#### Step 37 — Add the body parameter description
Scroll down to **Body Parameters** and paste the following into the description box:

```
The full user message exactly as spoken. Always populate this field with the user's request before calling the tool.
```

#### Step 38 — Set the parameter identifier
In the **Identifier** field, type the exact parameter name you used in Step 21.

> 💡 If your expression in n8n was `{{ $json.body.message }}`, type only `message` here — nothing else.

#### Step 39 — Confirm the parameter description
Make sure the description from Step 37 is applied to this body parameter.

#### Step 40 — Save the tool
Click **Add Tool** to finalize.

#### Step 41 — Publish your n8n workflow
Return to n8n and click **Publish** to make the workflow live.

---

## 🎉 Your Voice Assistant is Ready!

You can download the **workflow JSON file** and import it directly into n8n to skip the build steps — but you will still need to complete the ElevenLabs setup manually.

> ⚠️ **Important:** The ElevenLabs configuration must be set up correctly or the voice assistant will not function. Pay close attention to Steps 35–40.

---

## ❓ Questions & Support

Run into an issue? Open a [GitHub Discussion](../../discussions) and I'll do my best to help.

If this project helped you, please give it a ⭐ — it helps others find it!

---

*Built by [Jawad](https://github.com/Jawad2011) · Beginner-level n8n + ElevenLabs project · Open source*
