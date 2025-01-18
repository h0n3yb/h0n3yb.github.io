---
layout: post
title: "Building a Voice-Powered Personal AI Assistant"
description: "Step-by-step technical guide to building a voice AI assistant using ElevenLabs and n8n, with calendar integration and practical implementation tips."
date: 2025-01-18
image: /assets/images/social/helo-logo.jpg
image_alt: "Voice AI Assistant Implementation Guide - Technical Architecture Diagram"
author: Alex Ash
keywords: voice ai, elevenlabs, n8n, ai assistant, calendar integration, automation, agents, ai agent
tags: [AI, Voice Assistant, Tutorial, Integration, Agents]
---

# Step-by-Step Guide: Building a Voice-Powered Personal AI Assistant

I've recently built a personal voice AI assistant combining ElevenLabs and n8n. After debugging and resolving all the issues, I've recapped the details so you can build your own.

The solution consists of Twilio for handling phone communication, ElevenLabs for powering the voice AI interface, and n8n for managing backend automation and calendar integration. This creates a seamless voice-controlled personal assistant capable of managing your calendar efficiently.

You'll need a Twilio standard account (skip the trial), an ElevenLabs pro account, an n8n installation, and Google Calendar API access (OAuth2) through the Google Cloud Console.

## ElevenLabs Agent Configuration
- Navigate to Conversational AI > Agents
- Create a new agent
- Set welcome message (keep it brief and professional)
  Example: "Hi, my name is X. How can I help you?"
- Configure system prompt following pattern:
  * Persona
  * Instructions
  * Guidelines
  * Caveats
  * Closing remarks

Here is an example of a system prompt:

```plaintext
You are a helpful and enthusiastic personal assistant for a consulting firm.

You will receive calls from HELO executives and your job is to handle their requests.

If the user says "tomorrow", simply propagate the term to your tool call. Do not ask what day tomorrow is.

You have the following <tools> available:

<tools>
<check_availability>
If the caller wants to schedule a meeting, use the check_availability tool to see if the requested slot is available. You must always check availability first, before proceeding with the request.
</check_availability>
<book_meeting>
If the requested slot is available, use the book_meeting tool.
</book_meeting>
</tools>
Once an available slot is found and agreed upon, you must figure out the attendees.

If the caller doesn't mention attendees, ask them who will be attending. You will need their email addresses. The caller will tell you each attendee's email address, and you will confirm the email address by spelling it back to the caller letter-by-letter. Do not spell out the attendee's email domain, unless it is an unusual domain that is not a standard email provider.

Be courteous and reply to basic off-topic questions, but do not enter into an off-topic conversation with the caller.

Once all actions are done, ask if the caller needs anything else. If the caller responds that they have no further requests need anything else, wish them a nice rest of the their day and say goodbye.
```

- Set LLM to Gemini 1.5 Flash for optimal speed
- Configure temperature (0.7/0.8 for natural responses)


### Knowledge Base

Setup the knowledge base so that your assistant can access information that will be needed for repeated calls.

In my case, I wanted the agent to know my team's email, my own, and the notetaker's email.

- Add team member entries as text entries:
  * Title: Team member's name
  * Value: Their email address
- Add personal information:
  * Title: Your name
  * Value: Your email
- Include notetaker reference:
  * Title: Company name notetaker
  * Value: Notetaker email

### Tools
We'll set up two essential tools in the ElevenLabs interface for calendar management: one to check availability and another to book meetings. Both will integrate with Google Calendar through n8n.

#### Authentication

Generate an API key for your webhook authentication. This will be used to secure communication between ElevenLabs and n8n.Store this key safely as you'll need it for both tools. Add it to the headers as:

- Name: Authorization
- Value: Bearer {API_KEY} 

#### Tool 1: Check Meeting Availability

Navigate to the Tools section in your ElevenLabs agent. Click "Add Tool" and select "Webhook". 

- Name: check_availability
- Description: Check calendar for available meeting slots
- Method: POST
- URL: your_n8n_webhook_url/ai-assistant-voice

Headers:
- Authorization: Bearer your_api_key
- Content-Type: application/json

Query Parameters:
- Key: request
- Value: check_available_times
- Type: string
- Required: true
- Description: Identifier for availability check operation

Request Body:
```
{
  "meeting_datetime": {
    "type": "string",
    "description": "ISO format datetime for the proposed meeting",
    "required": true
  }
}
```

#### Tool 2: Book a Meeting

Click "Add Tool" again:

- Name: book_meeting
- Description: Schedule a new meeting on the calendar
- Method: POST
- URL: your_n8n_webhook_url/ai-assistant-voice

Headers:
- Authorization: Bearer your_api_key
- Content-Type: application/json

Body:
```
{
  "meeting_datetime": {
    "type": "string",
    "description": "ISO format datetime for the meeting",
    "required": true
  },
  "attendees": {
    "type": "string",
    "description": "Comma-separated email addresses",
    "required": true
  },
  "title": {
    "type": "string",
    "description": "Meeting title (auto-generated from attendee names)",
    "required": true
  }
}
```

Response Handling:
```
{
  "success": {
    "type": "boolean",
    "description": "Whether the meeting was booked successfully"
  },
  "message": {
    "type": "string",
    "description": "Confirmation or error message"
  },
  "meeting_link": {
    "type": "string",
    "description": "Google Meet link (if successful)"
  }
}
```

#### System Prompt Integration:
Add these tool descriptions to your system prompt:

Available Tools:
1. check_availability: Verify if a time slot is free
2. book_meeting: Schedule a meeting with specified attendees

Tool Usage Guidelines:
- Always check availability before booking
- Format dates in ISO 8601 format
- Handle attendee names according to knowledge base

By following these steps, your ElevenLabs agent will be properly configured to handle both checking availability and booking meetings through your n8n webhook. The tools are secured with API key authentication and can handle various meeting scheduling scenarios. Remember to test each configuration step before moving forward, and maintain proper error handling throughout the implementation.

## ElevenLabs Voice Settings
For natural interaction:
- Stability: 0.5 (midway point)
- Similarity: 0.65
- Latency: Level 3 for quick response time
- Recommendation: Use a cloned voice with a high-quality microphone for best results

## n8n Implementation

### 1. Webhook Configuration
```javascript
Node: 'Webhook'
Path: '/ai-assistant-voice'
Method: POST
Authentication: Header Auth with API key
Response: Configure webhook response node
```

### 2. Nodes

#### AI Agent
1. Chat Model 
2. Tools


#### Chat Model
1. In the AI Agent node settings:
   - Locate "Chat Model" section
   - Select "OpenAI" from the dropdown
   - Add OpenAI credentials:
     - Click "Add Credential"
     - Enter your OpenAI API key
   - Select model: "GPT-4"


#### Google Calendar Integration:

Get Events Node:
- Operation: Get Many
- Authentication: OAuth2 (client ID + secret from Google Cloud Console)
- Calendar Selection: Gmail calendar
- Options:
  * after: $fromAI("one day before")
  * before: $fromAI("one day after")
  * Returns all meetings within specified range

Create Event Node:
- Operation: Create
- Fields:
  * start: $fromAI("start_time")
  * end: $fromAI("end_time")
  * attendees: $fromAI("attendees")
  * conferenceData: Google Meet
  * sendUpdates: "all" (ensures notifications)
  * summary: $fromAI("event_title")
```

## Edge Case Handling
Consider these critical scenarios:
- Back-to-back meeting detection (15-minute buffer)
- Meeting notification system (ensure sendUpdates: "all")
- Date interpretation (relative vs. absolute)
- Time zone handling
- System prompt specifications for conflict handling
- Meeting overlap detection with clear definitions

## Testing and Deployment Strategy
Follow this iterative testing approach:
1. Activate n8n workflow
2. Test voice interactions through ElevenLabs interface
3. Monitor executions view in n8n
4. Adjust system prompts as needed
5. Debug tool calling logic
6. Toggle between editor and executions view for real-time debugging

## Notes

ElevenLabs credits go quick (I burned through an hour of audio to make this guide), so test your backend via standard API calls when you can.

I regularly share detailed technical tutorials on AI implementation, automation workflows, and system integration. Follow me on X [@alexheloai](https://x.com/alexheloai) for more.

# Want us to build a more advanced agent for you? 

[Book a call with us](https://calendly.com/helo-solutions/30min)