# Restauarnt-voice-assistant-using-Elevenlabs


Hello, everyone. In this project, I built a practical Restaurant voice assistant using elevenlabs. In this project we will see the beginner level use webhook tools. 

This tool can:

- Answer customer queries
- Share menus
- Book reservations
- Inform the customer about restaurant opening status

<img width="1428" height="532" alt="Image" src="https://github.com/user-attachments/assets/966b943f-f0c7-4656-83d7-5e414c365a2d" />



Let's see how to build this tool yourself.


The nodes used in this project:

1) Webhook

2) AI Agent

  > Gemini Chat Model
  
  > Redis chat memory
  
  > Google Sheets tool (Get Rows)
  
  > Telegram tool
  
  > Date & time tool

3) respond to webhook






Steps:


1) Take "webhook" trigger node.

2) Configure the webhook into "Post" Method.

3) Then, Choose any text as path

4) Select Respond into "Using 'Respond to webhook' Node"

5) Take an AI Agent node

6) Take Any model. In this project, I chose Gemini Chat Model. You may also use any other models.

7) Take any Memory among Redis, Postgres, MongoDB or Xata. I recommend using Redis because it is fas responding.

8) Take "Google Sheets Tool"

9) Configure the "Google Sheets tool" into "get Row(s)" operation

10) Choose the document using 'Document' and 'Sheet' inside "Google Sheets tool"

11) Take "Telegram tool"

12) Configure the operation of Telegram Tool into "Send Message"

13) Give your Chat ID.

14) Select the <img width="38" height="36" alt="Image" src="https://github.com/user-attachments/assets/3051a726-18c0-4838-ba55-c21595387d53" /> in the "Text" box.

15) Take "Date & Time Tool"

16) Configure "Date & Time Tool" by selecting an option named "timezone".

17) Select the timezone in 'Continent/City' format e.g. Europe/London

18) Take "Respond to Webhook" node

19) Go back to the AI Agent and paste this exact prompt into system message


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

20) Select "Source for Prompt (User Message)" into "Define Below"




21) Copy this Text into "Prompt (User Message)"

```
{{ $json.body.xxxx }}
```

N.B. in the place of xxxx, use any name as body parameter which will be used later to connect with Eleven labs





Now, Our workflow is finished but we need to connect it to Elevenlabs. So, let's go


22) Sign into Elevenlabs.

23) Go to Agents in the taskbar at the left

24) Select "New Agent"

25) After clicking "New Agent", you will select "Blank Agent"

26) Give any name.

27) Now, click "Create Agent"

28) Paste this exact System prompt

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


29) Select "Tools" in the upper taskbar.

30) Select "Add Tool"

31) After selecting "Add Tool", select "Webhook"

32) Give this tool any name.

33) Paste this exact prompt into description


```
Must be called on every single user message without exception. Sends the user's request to the automation backend and returns the answer. Never respond without calling this tool first.
```

34) Select the method into "Post"

35) Go back to your Workflow and copy the URL from the webhook node

N.B. There are two URLs. One is test and other one is production. Copy the Production URL.

36) Now, paste this URL into URL box in Elevenlabs

37) Scroll Down and Paste this prompt into the description box of "Body parameters"

```
The full user message exactly as spoken. Always populate this field with the user's request before calling the tool
```


38) In the "Identifier" box type that exact name you used as the name of parameter in the AI Agent's Source for Prompt (User Message). If you use {{ $json.body.xxxx }} on Source for Prompt (User Message), You only have to type the xxxx part, nothing else.

39) Paste the exact prompt mentioned on step 37.

40) Now Select "Add tool"

41) Return back to your n8n workflow and publish it.



Now your Restaurant AI Voice Assistant is ready to be used. You can download the  json file and paste it into your n8n to use it but you have to do the Elevenlabs setup part yourself. And Be careful about the Elevenlabs part. Your AI Voice Assistant may not function properly if the Elevenlabs isn't properly set up.

If you still run into an issue, You can start a discussion and I will try my best to help you with that. Thank You.


If you find this guide amazing make sure to star it so others can find it also.
