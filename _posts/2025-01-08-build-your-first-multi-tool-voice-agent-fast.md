---
layout: post
title: "Build Your First Multi-Tool Voice Agent Fast"
date: 2025-01-08
---

## Build Your First Multi-Tool Voice Agent Fast

Voice AI agents are extremely good now. They actually listen to your words, and respond with very little lag. Combine this level of quality with the ability to use tools, and we have ourselves a viable voice-powered AI agent which can execute complex tasks.

This guide explains how to build a voice agent that can independently access multiple tools and make decisions during sales outreach, customer support, and internal operations.

<img src="/assets/images/Core Architecture - visual selection.png" alt="Alt text" width="600"/>

The system combines a couple of components:

<img src="/assets/images/agent-arch.png" alt="Alt text" width="600"/>

### ElevenLabs Voice Agent Setup
1. Create a new agent in the ElevenLabs Conversational AI menu

2. Configure the initial greeting message. If your agent will be doing outreach, you can leave this blank, and the agent will speak once the user says "Hello?".

3. Create a system prompt that includes tool definitions. 

4. Configure the webhook URL for external tool access -- this is agnostic to the tools you're using. Many automation tools will have a webhook system which you can use to trigger your agent.

5. Select a frontier-quality LLM. I've found Gemini Flash 1.5 to have the best latency:quality ratio thus far.

For the system prompt, you'll want to follow a general pattern like this:

```text
You are a support agent with access to:
- Calendar for checking appointment availability
- Vector database for retrieving information
- Google Sheets for storing customer data

Your job is to help the user with their request.

Here's how to use your tools:

- Use the calendar if the user is asking about scheduling.
- Use the vector database if the user is asking about information.
- Use Google sheets to catalog what happened in the conversation.
```

### Vector Database Configuration
1. Prepare your knowledge base documents.  Gather all your key resources like your website content, FAQs, and documentation - basically anything you want your agent to know about.

2. Process your documents into embeddings. Transform your documents into a special format that helps your agent understand and work with them efficiently.

3. Store the embeddings in a vector database (e.g., Pinecone). Think of this as your agent's organized library of information.

4. Configure the query endpoint. Set up the connection point where your agent can quickly search through your knowledgebase.

5. Test retrieval with sample queries. Run some practice searches to make sure your agent can find what it needs.

**NOTE**: You'll use embeddings if you're rolling your own workflow. If you prefer to stick with ElevenLabs, you can skip this step and use the Knowledge Base section in your Agent configuration. Just know you will trade off some flexibility for ease of use.

<img src="/assets/images/Vector Database Configuration - visual selection.png" alt="Alt text" width="600"/>

### Calendar Integration
1. Set up Google Calendar API access. Get your agent connected to Google Calendar using oauth - it's like giving your agent permission to manage your schedule.

2. Create a dedicated calendar for appointments. Give your agent its own calendar to keep things neat and organized.

3. Configure the availability checking endpoint. Build a way for your agent to quickly check when you're free or busy.

4. Set up the appointment creation endpoint. Create the tool your agent needs to actually schedule meetings.

5. Implement an email notification system. Make sure everyone stays in the loop with automatic emails when appointments are booked.

### Customer Data Storage
1. Set up a customer database in Google Sheets with key info columns:
   - Name
   - Phone
   - Email
   - Appointment Time
   - Notes

2. Get your agent access to Google Sheets through their API.

3. Run some practice queries to make sure your agent can add new customer records.

### Webhook Setup

<img src="/assets/images/webhook- visual selection.png" alt="Alt text" width="600"/>

1. Build your main connection point

2. Give it a clear address: `/voice-agent`

3. Make it ready to receive POST requests

4. Use this format for incoming messages:
```json
{
  "query": "user's request",
  "context": {
    "name": "user name",
    "phone": "phone number"
  }
}
```

### Tool Usage
1. Map out key phrases for the agent to recognize:
   - Scheduling words: "schedule", "book", "availability"
   - Questions: "what", "how", "tell me about"
   - Contact management: "save", "store", "remember"

3. Connect these phrases to the right tools by associating the tool name with the phrase in your ElevenLabs agent's system prompt.

### Testing
1. Make sure each tool works on its own

2. If you can, use a cloned voice instead of a stock one from ElevenLabs. This will help your agent sound more human and natural, but make sure your microphone is good, because the cloned voice quality will sound like your microphone's quality.

3. Test complete conversations for:
   - Finding information
   - Booking appointments
   - Saving contact details
<br>
4. See how it handles mistakes

# Your turn

Here are some final tips:

Build your agent like you're crafting an experience. Keep responses crisp and concise, but complete. Your tools are only as reliable as your weakest link. Log everything that happens in your flow. Make sure you have sufficient visibility to understand where things go wrong (it'll happen). 

On security: guard your endpoints well. Validate all inputs, and audit your attack surface regularly.

Start small, break things, iterate fast, and I promise your expectations will converge with reality. 

Feel free to reach out with questions [@alexheloai](https://x.com/alexheloai).

# Let us build it for you

**Let's sync -- book a call @ [helosolutions.ai](https://helosolutions.ai)**