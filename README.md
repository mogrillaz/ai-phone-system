# AI Phone System: Production Voice Agent

Real-time voice agent handling inbound calls via Twilio. Conducts natural conversations, extracts structured data, routes to correct department, and generates transcripts for follow-up.

## 🎯 The Problem

Manual call routing and intake took **20 hours per month**:
- Listen to calls, understand what caller needs
- Match to correct department/service area
- Document everything for handoff
- Repeat thousands of times

## ✅ The Solution

Automated voice agent:
1. **Receives call** via Twilio → triggered in n8n
2. **Conducts conversation** using GPT-4 (natural, contextual)
3. **Extracts data** (location, service type, urgency)
4. **Maps to department** using Excel region lookup
5. **Transcribes + summarizes** call into structured email
6. **Routes email** to correct team member
7. **Logs everything** for compliance + analytics

## 📊 Results

| Metric | Before | After | Impact |
|--------|--------|-------|--------|
| **Time per call intake** | 6 min manual | 2 min AI-assisted | 67% faster |
| **Monthly time spent** | 20 hours | 5 hours | **75% reduction** |
| **Accuracy** | ~90% (human error) | 95%+ | Fewer misroutes |
| **Availability** | 9–5 only | 24/7 | Round-the-clock |

## 🏗 Architecture

```
Inbound Call (Twilio)
    ↓
Twilio Webhook → n8n Workflow
    ↓
GPT-4 Agent (Speech-to-Text)
    ├─ Greeting + context setting
    ├─ Question: "What is your location (PLZ)?"
    ├─ Question: "What service do you need?"
    ├─ Optional: "Tell me more about your request"
    ↓
Data Extraction → Excel Lookup
    ├─ PLZ → Region mapping
    ├─ Service type → Department
    ├─ Urgency assessment
    ↓
Speech-to-Text Transcription
    ↓
GPT-4 Summarization
    ├─ Caller info
    ├─ Request summary
    ├─ Recommended action
    ↓
Email Composition + Routing
    ├─ To: [department_email]
    ├─ CC: [supervisor_if_urgent]
    ├─ Attachment: [transcript]
    ↓
Logging + Analytics
    ├─ BigQuery: call metadata
    ├─ Success/failure tracking
    ├─ Quality metrics
```

## 🔧 Tech Stack

| Component | Technology | Why |
|-----------|-----------|-----|
| **Phone** | Twilio Voice API | Reliable, flexible IVR |
| **Orchestration** | n8n | Workflow coordination, scheduling |
| **Speech-to-Text** | Google Cloud Speech-to-Text | Accurate German/English support |
| **LLM** | GPT-4 | Natural conversation, context retention |
| **Data Lookup** | Google Sheets + Excel | Fast, updatable region mapping |
| **Logging** | BigQuery | Analytics, compliance audit trail |

## 🎯 How It Works (Step-by-Step)

### **Inbound Call**
```
Customer dials → Twilio answers → Webhook hits n8n workflow
```

### **AI Conversation**
```json
{
  "timestamp": "2025-05-07T14:32:00Z",
  "caller_number": "+49123456789",
  "workflow": {
    "greeting": "Guten Tag, Sie sprechen mit unserem automatischen Service. Wie kann ich Ihnen helfen?",
    "question_1": "Bitte nennen Sie Ihre Postleitzahl.",
    "answer_1": "40211",
    "question_2": "Welchen Service benötigen Sie? Haushaltshilfe, Pflege, oder andere?",
    "answer_2": "Haushaltshilfe",
    "follow_up": "Können Sie kurz beschreiben, was Sie benötigen?",
    "answer_3": "Ich brauche Hilfe beim Putzen und Einkaufen, zweimal pro Woche.",
    "confidence_score": 0.96
  }
}
```

### **Data Extraction & Routing**
```
PLZ 40211 → Region: Düsseldorf-West
Service: Haushaltshilfe → Department: Team_DUS_West
Priority: Standard (no urgency flags)
→ Email to: anna.mueller@service.de
→ CC: supervisor@service.de
```

### **Generated Email**
```
Subject: Neue Anfrage – Haushaltshilfe Düsseldorf-West (40211)

Anrufer: +49123456789
Postleitzahl: 40211 (Düsseldorf-West)
Service: Haushaltshilfe
Datum: 2025-05-07 14:32 UTC
Priorität: Standard

Anfrage:
"Ich brauche Hilfe beim Putzen und Einkaufen, zweimal pro Woche."

Nächste Schritte:
1. Kontaktieren Sie Anrufer bis spätestens morgen
2. Bedarfscheck: Umfang? Termine?
3. Verfügbarkeit prüfen & Angebot machen
4. Bestätigung an Anrufer

Transkript: [vollständige Aufzeichnung angehängt]
Confidence: 96% (hohe Datenqualität)
```

## 🎓 Technical Decisions & Learnings

### **Why GPT-4, not cheaper model?**
- Conversation quality matters – misunderstandings = repeat calls
- Supports context retention across 5–10 exchanges
- Handles edge cases (caller gets emotional, goes off-topic, etc.)
- Cost per call ~$0.03–0.05 << savings from manual labor

### **Why Google Speech-to-Text?**
- Accurate German support (important for accents, dialect)
- Real-time streaming (low latency)
- Integration with n8n is native

### **Why n8n for orchestration?**
- Visual workflow = easy to debug & modify
- Native integrations: Twilio, Google Cloud, email
- Scheduling + error handling built-in
- No code needed for 80% of logic

### **Why Excel lookup instead of database?**
- Business team updates regions themselves (no dev dependency)
- Fast enough for real-time lookups (<50ms)
- Versionable, auditable

## 🚨 Error Handling & Edge Cases

**What if the caller doesn't give PLZ?**
- Agent asks 2x, then routes to generic queue
- Escalation flag added to email

**What if speech is unclear?**
- Confidence score < 70% → adds "REVIEW NEEDED" flag
- Supervisor gets CC'd

**What if call drops?**
- Partial transcript logged
- Follow-up email sent with available info
- Caller can call back, system has history

**What if API is down?**
- Fallback: Call routed to human answering machine
- n8n retries every 5 min for 24h
- Alert sent to ops team

## 📈 Metrics & Monitoring

**Daily Dashboard:**
- Calls received: [count]
- Success rate: [%]
- Avg call duration: [minutes]
- Misroutes: [count]
- Average confidence score: [%]

**Monthly Reports:**
- Total time saved: [hours]
- Cost per call: [€]
- Cost savings: [€]
- Customer satisfaction (if surveyed): [%]

## 🔐 Privacy & Compliance

- **GDPR:** Calls only stored encrypted, deleted after 30 days (configurable)
- **Data Access:** Only authorized team members can view transcripts
- **Audit Trail:** All accesses logged in BigQuery
- **Consent:** Caller hears "Call will be recorded" message at start

## 🚀 Next Steps / Roadmap

- [ ] Add sentiment analysis (escalate if caller is frustrated)
- [ ] Integrate with CRM (look up existing customer history)
- [ ] A/B test different greeting scripts
- [ ] Multilingual support (add Italian, Turkish for target regions)
- [ ] Callback scheduling (if queue is full, offer callback instead)

## 📝 How to Deploy Your Own

1. **Set up Twilio account** + get phone number
2. **Connect n8n workflow** (webhook receives calls)
3. **Add GPT-4 API key** (OpenAI)
4. **Create Google Sheets** with PLZ ↔ Department mapping
5. **Configure email routing** (Gmail or your mail provider)
6. **Set up BigQuery** for logging
7. **Test** with test calls before going live

(Detailed setup guide in `SETUP.md`)

## 💡 Key Insights

- **Production AI ≠ ChatGPT in a loop.** Real systems need error handling, monitoring, fallbacks.
- **Cost matters.** Model selection (GPT-4 vs. Llama vs. Haiku) changes business economics.
- **User experience is everything.** A voice agent that misunderstands wastes time, not saves it.
- **Data quality gates everything.** Confidence scoring + escalation flags are more important than perfect automation.

---

**This system runs daily, handling real calls, real money, real impact.**
