## 1. Problem Understanding

Modern applications generate notifications from multiple services such as messages, reminders, alerts, promotions, and system updates.

This creates several challenges:

- Users receive excessive notifications (alert fatigue)
- Duplicate or near-duplicate notifications are sent
- Low-value notifications interrupt users
- Important notifications may be delayed or missed
- No clear explanation exists for decisions

The goal of this system is to design a smart and scalable Notification Prioritization Engine that classifies each notification into:

- **Now** – Send immediately  
- **Later** – Defer and schedule  
- **Never** – Suppress  

---

## 2. System Requirements

The system must:

- Classify notifications into **Now / Later / Never**
- Prevent exact and near duplicates
- Reduce alert fatigue
- Handle conflicting priorities
- Support configurable rules
- Log clear explanations for every decision
- Fail safely if AI or dependent services are unavailable
- Ensure important notifications are never silently lost

---

## 3. High-Level Architecture

The system follows a modular and scalable architecture.

### Components:

- **API Gateway**
  - Receives incoming notification events
  - Performs authentication and rate limiting

- **Validation & Preprocessing Layer**
  - Validates required fields
  - Normalizes data
  - Generates fallback dedupe keys

- **Duplicate Detection Module**
  - Checks exact duplicates using `dedupe_key`
  - Detects near duplicates using message hashing + time window

- **Decision Engine**
  - Applies rule-based logic
  - Uses AI scoring (if available)
  - Determines Now / Later / Never

- **User History Store (Redis/Database)**
  - Tracks recent notifications
  - Maintains counters and cooldown timestamps

- **Scheduler Service**
  - Handles deferred notifications

- **Audit Logging Service**
  - Logs decision and explanation

This modular design ensures scalability, reliability, and separation of concerns.

---

## 4. Decision Logic Design

The classification process follows structured evaluation steps:

### Step 1: Expiry Check
If `expires_at` is in the past → classify as **Never**.

### Step 2: Duplicate Check
If exact or near duplicate detected → classify as **Never**.

### Step 3: Priority Evaluation
Compute priority using:
- `priority_hint`
- `event_type`
- `source`
- metadata
- AI scoring (if available)

### Step 4: Alert Fatigue Check
Check recent notification count for the user:

- High frequency + High priority → **Now**
- High frequency + Medium priority → **Later**
- High frequency + Low priority → **Never**

### Step 5: Final Classification
Assign final decision:
- **Now**
- **Later**
- **Never**

---

## 5. Minimal Data Model

### Notification Event
- event_id
- user_id
- event_type
- message
- source
- priority_hint
- timestamp
- channel
- dedupe_key
- expires_at

### User Notification History
- user_id
- last_notification_time
- notification_count_last_10_min
- cooldown_end_time

### Decision Log
- event_id
- decision
- reason
- priority_score
- timestamp

---

## 6. API Design (Sample Endpoints)

### Submit Notification
`POST /notifications`

Response Example:

{
  "decision": "Later",
  "reason": "User exceeded frequency threshold",
  "scheduled_time": "2026-02-25T10:30:00Z"
}

### Get Decision Log
`GET /notifications/{event_id}/decision`

### Update Rules
`POST /rules/update`

Allows administrators to update:
- Cooldown thresholds
- Frequency caps
- Priority mappings

---

## 7. Duplicate Prevention Strategy

### Exact Duplicates
- Use provided `dedupe_key`
- If missing, generate hash using:
  user_id + message + event_type + timestamp window

### Near Duplicates
- Message similarity comparison
- Time window filtering (e.g., 5 minutes)

This reduces redundant notifications.

---

## 8. Alert Fatigue Strategy

To prevent user overload:

- Per-user cooldown periods
- Notification caps per time window
- Deprioritize promotional notifications during high activity
- Batch low-priority notifications into digest
- Avoid late-night delivery for non-urgent events

---

## 9. Fallback Strategy

If AI service is unavailable:

- Switch to rule-based engine
- Critical notifications default to **Now**
- Log fallback activation
- Ensure important alerts are not lost

This ensures reliability and fail-safe behavior.

---

## 10. Monitoring & Metrics

Key metrics:

- Notification distribution (Now / Later / Never)
- Duplicate suppression rate
- Average decision latency
- AI fallback frequency
- Error rate
- User notification frequency

Alerts are triggered if:
- Latency exceeds threshold
- Critical suppression rate increases abnormally

---

## 11. Trade-offs & Design Decisions

- Redis used for fast counters with periodic persistence
- Rule engine ensures reliability during AI failure
- Near-duplicate detection increases compute cost but improves UX
- Balances urgency and user fatigue

---

## Sample Input

{
  "user_id": "123",
  "event_type": "promotion",
  "message": "50% discount offer",
  "priority_hint": "low",
  "timestamp": "2026-02-26T10:00:00Z"
}

## Sample Output

{
  "decision": "LATER",
  "reason": "Low priority event and user exceeded notification threshold"
}

---

## Edge Case Handling

- If dedupe_key is missing → Generate hash using user_id + event_type
- If duplicate detected → Suppress notification
- If AI scoring fails → Use rule-based fallback
- If notification volume exceeds limit → Delay non-urgent alerts

---

## Future Improvements

- Machine learning based personalized scoring
- User preference learning system
- Real-time analytics dashboard
- Time-aware smart scheduling

## 12. Conclusion

This Notification Prioritization Engine:

- Reduces alert fatigue
- Prevents duplicate notifications
- Ensures important alerts are delivered
- Provides full explainability
- Scales for high-volume systems
- Maintains reliability under failure scenarios

The system is designed with real-world production constraints and emphasizes clarity, reliability, and user-centric decision making.
    sum = 0
    number=str(n)
    for digit in number:
    sum+=int(digit)**2
