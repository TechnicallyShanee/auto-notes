# AutoNotes

### AI-Powered Note-to-Action Automation

> Write it once. Let automation organize the rest.

## About the Project

I love writing things down, but I don't love having to organize the same information again across multiple apps.

An appointment written in my notes still needs to be added to my calendar. Something I need to buy still needs to be added to my shopping list. A reminder still needs to be created somewhere else.

I built **AutoNotes** to eliminate that duplicate work.

AutoNotes is an AI-powered workflow that interprets everyday notes, identifies actionable information, converts it into structured data, and prepares it for the appropriate productivity application.

Before anything is created, the system sends me a Daily Brief through Telegram where I can **Approve, Edit, or Reject** the proposed actions.

The idea is simple:

**Capture once → AI interprets → I review → Automation executes.**

---

## 💡 The Problem

My note-taking system is where I capture information throughout the day, including:

- Appointments
- Reminders
- Tasks
- Shopping items
- Family responsibilities
- Goals
- Expenses
- Other information I want to remember

The problem wasn't capturing information.

The problem was everything that had to happen **after** I captured it.

I was still manually transferring information from my notes into calendars, task lists, shopping lists, trackers, and other applications.

AutoNotes started with a question:

> **What if writing something down once was enough?**

---

## 🎯 Project Objectives

I designed AutoNotes to:

- Accept naturally written notes without requiring rigid formatting
- Use AI to understand the intent behind each note
- Convert unstructured information into structured JSON
- Categorize actionable information
- Generate a human-readable Daily Brief
- Require approval before executing actions
- Allow natural-language corrections
- Validate information before sending it to external applications
- Route approved information to the appropriate destination
- Maintain workflow state between Telegram interactions
- Reduce repetitive manual data entry

Duplicate prevention is also being developed to prevent previously processed information from being recreated.

---

## ⚙️ How It Works

AutoNotes follows a human-in-the-loop automation model.

```text
Daily Notes
     │
     ▼
AI Interpretation
     │
     ▼
Structured JSON
     │
     ▼
Daily Brief
     │
     ▼
Telegram Review
     │
     ├──── APPROVE ────► Validation ────► Application Routing
     │
     ├──── EDIT ───────► AI Revision ───► Revised Daily Brief
     │
     └──── REJECT ─────► Stop
```

---

## 🧠 Natural-Language Interpretation

AutoNotes is designed around natural input.

I don't need to manually label information before writing it.

For example:

| Note | Interpreted Action |
|---|---|
| Buy toothpaste | Shopping item |
| Dentist appointment Friday at 2 PM | Calendar event |
| Remind me to call the school tomorrow | Reminder |
| Finish documentation by Monday | Task |
| Save $5,000 by December | Goal |

Google Gemini acts as the interpretation layer.

Its responsibility is to understand natural-language input and convert it into predictable structured data that the workflow can process.

---

## 📦 Structured Data

Instead of allowing AI-generated text to directly control downstream applications, AutoNotes converts interpreted information into structured JSON.

Example:

```json
{
  "calendar": [
    {
      "title": "Dentist Appointment",
      "date": "2026-08-14",
      "time": "2:00 PM"
    }
  ],
  "shopping": [
    {
      "item": "Toothpaste"
    }
  ],
  "reminders": [
    {
      "title": "Call the school",
      "date": "2026-08-11"
    }
  ]
}
```

This gives the automation a predictable structure that can be parsed, validated, filtered, and routed.

---

## 🙋🏽‍♀️ Human-in-the-Loop Approval

I intentionally designed AutoNotes so that AI interpretation does not automatically result in actions being created.

Before anything reaches an external application, Telegram presents the interpreted information as a Daily Brief.

I can then choose:

### APPROVE

The structured information is validated and routed to the appropriate applications.

### EDIT

The workflow enters an editing state and waits for a natural-language correction.

### REJECT

Processing stops and the proposed actions are not created.

This keeps me in control while still allowing AI to handle the interpretation and organization of unstructured information.

---

## ✏️ Conversational Editing

The EDIT workflow allows corrections without restarting the entire process.

For example:

> Change the dentist appointment to Friday at 3 PM.

When an edit is requested:

1. The current Daily Brief enters an `awaiting_edit` state.
2. The next relevant Telegram message is captured as an edit instruction.
3. The existing structured JSON and edit instruction are sent to Gemini.
4. Gemini modifies the requested information while preserving unrelated data.
5. The revised structured data replaces the previous version.
6. A new Daily Brief is generated.
7. The revised brief must be reviewed again.

The workflow therefore supports an iterative approval cycle:

**EDIT → Instruction → AI Revision → Review → APPROVE / EDIT / REJECT**

---

## 🗃️ State Management

AutoNotes uses Make.com Data Stores to maintain workflow state across separate Telegram interactions.

Example states include:

- `pending`
- `awaiting_edit`
- `approved`
- `rejected`

State management allows the workflow to determine whether an incoming Telegram message is a new command, an editing instruction, or part of an existing approval process.

---

## 🛡️ Validation

AI interpretation is followed by deterministic workflow validation.

For example, an AI response may technically contain a calendar array while not containing enough information to create a valid calendar event.

AutoNotes therefore validates individual items before they reach their destination modules.

### Calendar Events

- Event title must exist
- Required date information must exist
- Date and time values must be valid before event creation

### Shopping Items

- A valid item must exist before a shopping task is created

### Reminders & Tasks

- Required task information must exist before the item continues through the workflow

Invalid or incomplete items are prevented from continuing through the relevant route.

This separates two important responsibilities:

**AI → Interpret the information**

**Workflow → Validate and execute the action**

---

## 🔀 Application Routing

After approval and validation, items are routed according to their intended action.

| Information | Destination |
|---|---|
| Calendar events | Microsoft 365 Calendar |
| Tasks | Microsoft To Do |
| Reminders | Microsoft To Do |
| Shopping | Microsoft To Do |
| Goals | Microsoft To Do |
| Family tasks | Microsoft To Do |
| Expenses | Microsoft Excel / OneDrive |

Not every piece of information needs to leave the original note-taking environment.

The workflow is designed to create external actions only when doing so provides value.

---

## 🛠️ Tech Stack

| Technology | Purpose |
|---|---|
| Make.com | Workflow orchestration |
| Google Gemini | Natural-language interpretation & structured output |
| Telegram Bot | Approval & conversational editing interface |
| Make.com Data Stores | Workflow state management |
| Microsoft 365 Calendar | Calendar event creation |
| Microsoft To Do | Tasks, reminders, shopping & goals |
| Microsoft Excel | Expense tracking |
| OneDrive | File storage |
| JSON | Structured data exchange |

---

## 🧩 Engineering Challenges

### Conversational State

Telegram messages arrive as separate events. The workflow therefore needed a way to determine whether a message represented a new command or a continuation of an existing interaction.

I introduced persistent workflow states to distinguish between pending approvals and edit instructions.

### Edit Routing

During testing, edit acknowledgement messages were triggered multiple times because multiple records were allowed through part of the workflow.

I refined the routing and filtering logic to ensure the correct record continued through the edit process.

### AI Output Validation

Structured AI output can contain empty categories or incomplete objects.

Additional validation was introduced at the workflow level so that the existence of a category alone does not trigger an external action.

### Date & Time Handling

Natural-language dates and times must ultimately be converted into formats accepted by destination applications.

The workflow includes transformation logic between AI interpretation and application modules.

### Preventing Data Loss During Editing

Editing needed to change only the requested information without unintentionally modifying unrelated items.

The AI editing instructions therefore require the existing structure to be preserved while applying targeted changes.

---

## 🔁 Duplicate Prevention

**Status: In Development**

Because source notes may continue to contain information that has already been processed, AutoNotes needs to distinguish between new and previously handled items.

The planned approach uses deterministic item identifiers and a processed-item ledger.

```text
New Item
   │
   ▼
Generate Identifier
   │
   ▼
Previously Processed?
   │
   ├── YES → Skip
   │
   └── NO → Create Action → Store Identifier
```

This feature is currently being implemented and tested.

---

## 🚧 Project Status

AutoNotes is an active personal project.

### Implemented

- Natural-language note interpretation
- AI-generated structured output
- JSON parsing
- Daily Brief generation
- Telegram integration
- APPROVE workflow
- EDIT workflow
- REJECT workflow
- Conversational editing
- Revised approval cycle
- Data Store state management
- Category routing
- Calendar integration
- Microsoft To Do integration
- Excel expense tracking
- Item-level validation

### In Development

- Duplicate prevention
- Additional error handling
- Expanded validation
- Workflow optimization
- End-to-end reliability testing

### Future Ideas

- Processing history
- Audit logging
- Habit tracking
- Enhanced conflict detection
- Confidence-based review
- Reporting & analytics
- Additional productivity integrations

---

## 🔐 Privacy & Security

AutoNotes interacts with personal productivity applications, so sensitive information is intentionally excluded from this repository.

This repository does **not** contain:

- API keys
- Access tokens
- Telegram bot tokens
- Webhook URLs
- Microsoft credentials
- Gemini credentials
- Personal notes
- Real calendar information
- Personal financial information
- Private account identifiers

All examples and screenshots published in this repository use sanitized or fictional data.

---

## 📚 Concepts Demonstrated

This project explores:

- Workflow automation
- AI integration
- Prompt engineering
- Natural-language processing
- Structured JSON
- Data transformation
- Conditional routing
- Data validation
- State management
- Human-in-the-loop AI
- Application integrations
- Error handling
- Process improvement
- Requirements-driven solution design

---

## 📌 Project Notes

AutoNotes is a personal project that I am actively developing, testing, and improving.

The project started as a solution to an everyday problem: reducing the amount of repetitive work required after taking notes.

As the project develops, I will continue documenting the architecture, challenges, improvements, and lessons learned in this repository.
