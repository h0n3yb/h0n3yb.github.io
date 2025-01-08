---
layout: post
title: "Building Multi-Tool Voice AI Agents with ElevenLabs"
date: 2024-01-08
---

## Building Multi-Tool Voice AI Agents with ElevenLabs

Voice AI agents are extremely good now. They actually listen to your words, and respond with very little lag. This opens the door to many use cases which were previously not feasible with rigid robo-menu sounding systems. Combine this level of quality with the ability to use tools, and now all of a sudden we have a viable voice-powered AI agent which can execute complex tasks.

This guide explains how to build a voice agent that can independently access multiple tools and make decisions based on the conversation context during sales lead qualification, appointment scheduling, and customer support.

<img src="/assets/images/Core Architecture - visual selection.png" alt="Alt text" width="600"/>

The system combines several key components:
- ElevenLabs for voice interaction
- Vector database for information retrieval
- Google Calendar for appointment management
- Google Sheets for data storage
- Webhook system for kicking all of this off

<img src="/assets/images/agent-arch.png" alt="Alt text" width="600"/>

### ElevenLabs Voice Agent Setup
1. Create a new agent in the ElevenLabs Conversational AI menu

2. Configure initial greeting message. If your agent will be doing outreach, you can leave this blank, and the agent will speak once the user says: "Hello?".

3. Set your system prompt with tool definitions. You'll want to follow a generate pattern like this:

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

4. Configure webhook URL for external tool access -- this is agnostic to the tools you're using. Many automation tools will have a webhook system which you can use to trigger your agent.

5. Select GPT-4 or another model of similar quality (I like Claude 3.5 Sonnet v2).

### Vector Database Configuration
1. Prepare knowledge base documents.  Gather all your key resources like your website content, FAQs, and documentation - basically anything you want your agent to know about.

2. Process documents into embeddings. Transform your documents into a special format that helps your agent understand and work with them efficiently.

3. Store in vector database (e.g., Pinecone). Think of this as your agent's organized library of information.

4. Configure query endpoint. Set up the connection point where your agent can quickly search through all that knowledge.

5. Test retrieval with sample queries. Run some practice searches to make sure your agent can find what it needs.

<img src="/assets/images/Vector Database Configuration - visual selection.png" alt="Alt text" width="600"/>

### Calendar Integration
1. Set up Google Calendar API access. Get your agent connected to Google Calendar using oauth - it's like giving your agent permission to manage your schedule.

2. Create dedicated calendar for appointments. Give your agent its own calendar to keep things neat and organized.

3. Configure availability checking endpoint. Build a way for your agent to quickly check when you're free or busy.

4. Set up appointment creation endpoint. Create the tool your agent needs to actually schedule meetings.

5. Implement email notification system. Make sure everyone stays in the loop with automatic emails when appointments are booked.

### Customer Data Storage
1. Set up a customer database in Google Sheets with key info columns:
   - Name
   - Phone
   - Email
   - Appointment Time
   - Notes

2. Get your agent access to Google Sheets through their API

3. Create a simple way to add new customer records

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

### Tool Router Configuration
1. Build your central decision-making system

2. Map out key phrases to recognize:
   - Scheduling words: "schedule", "book", "availability"
   - Questions: "what", "how", "tell me about"
   - Contact management: "save", "store", "remember"

3. Connect these phrases to the right tools

4. Plan for when things don't go as expected

### Response Management
1. Make responses sound natural in spoken form

2. Structure your conversation flow:
   - Acknowledge what was asked
   - Handle the request
   - Confirm what was done
   - Guide to next steps

3. Keep conversations flowing naturally

4. Remember important details throughout the chat

### Testing
1. Check each tool works on its own

2. Make sure voice quality is crystal clear

3. Test complete conversations for:
   - Finding information
   - Booking appointments
   - Saving contact details

4. See how it handles mistakes

5. Make sure it can juggle multiple tasks

# Now it's your turn! 

Here are some final tips:

Build your agent like you're crafting an experience. Keep responses crisp and concise, but complete. Your tools are only as reliable as your weakest link. Log everything that happens in your flow. Simple systems survive; complex ones collapse, so make sure you have the visibility to see where things go wrong (spoiler: they will, but it's a natural part of the process). On security: your endpoints are your fortress walls - guard them well. Trust no input, verify everything. Audit your attack surface regularly.

Watch how your agent speaks and operates. Keep its knowledge fresh. The best systems aren't built - they're grown. Start small, iterate fast, break things, and I promise your expectations will converge with reality. 

Show me what you build on X and feel free to reach out with questions: (https://x.com/alexheloai).

# Want us to build something like this for you?

**Let's sync -- book a call @ [helosolutions.ai](https://helosolutions.ai)**