# SnapLearn - System Design Document

## 1. System Overview

**SnapLearn** is a cloud-native, serverless application built entirely on AWS infrastructure. It uses a microservices architecture with event-driven communication patterns to provide scalable, real-time educational assistance to millions of students across India.

---

## 2. High-Level Architecture

The system consists of 3 main layers:

```
┌─────────────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER                            │
│                   (React Native Mobile App)                      │
│         iOS & Android | Voice Capture | Offline Support          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                 API & BUSINESS LOGIC LAYER                       │
│    ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │
│    │ API Gateway  │  │   Lambda     │  │ EventBridge  │        │
│    └──────────────┘  └──────────────┘  └──────────────┘        │
│    ┌──────────────┐  ┌──────────────┐                          │
│    │Step Functions│  │   Cognito    │                          │
│    └──────────────┘  └──────────────┘                          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     DATA & AI LAYER                              │
│    ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │
│    │   Bedrock    │  │  Transcribe  │  │    Polly     │        │
│    └──────────────┘  └──────────────┘  └──────────────┘        │
│    ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │
│    │ Rekognition  │  │   DynamoDB   │  │      S3      │        │
│    └──────────────┘  └──────────────┘  └──────────────┘        │
└─────────────────────────────────────────────────────────────────┘
```

### Layer Descriptions

| Layer | Components | Responsibilities |
|-------|------------|------------------|
| Presentation | React Native App | User interface, voice capture, offline functionality |
| API & Logic | API Gateway, Lambda, EventBridge, Step Functions, Cognito | RESTful endpoints, serverless compute, event-driven architecture, authentication |
| Data & AI | Bedrock, Transcribe, Polly, Rekognition, DynamoDB, S3 | AI processing, data persistence, media storage |

---

## 3. Detailed Component Design

### 3.1 Mobile Application (React Native)

**Architecture Pattern:** Clean Architecture with Redux for state management

#### Key Components

##### a) Voice Input Module
- Uses device's native microphone API
- Records audio in .wav format (16kHz sampling rate)
- Real-time audio streaming to backend via WebSocket
- Visual feedback (waveform animation) during recording
- Handles permission requests for microphone access

##### b) Camera Module
- Native camera access for photo capture
- Image preprocessing (compression, format conversion to JPEG)
- Crop and rotate functionality before upload
- Gallery access for selecting existing photos

##### c) Offline Storage Module
- IndexedDB for storing downloaded lessons
- Service Worker for background sync
- Local cache for user profile and history
- Intelligent sync strategy when online

##### d) State Management (Redux)

| State Category | Contents |
|----------------|----------|
| User State | Profile, authentication, preferences |
| Learning State | Current topic, history, progress |
| UI State | Loading indicators, error messages |
| Offline Queue | Pending API requests |

##### e) Navigation
- React Navigation with tab-based layout
- Bottom navigation: Home, Chat, Practice, Profile
- Modal screens for homework upload and settings

#### UI Component Structure

```
/components
├── /common
│   ├── Button.js
│   ├── TextInput.js
│   ├── Card.js
│   └── LoadingSpinner.js
├── /voice
│   ├── VoiceButton.js          (large circular button with animation)
│   └── AudioWaveform.js        (visual feedback during recording)
├── /chat
│   ├── MessageBubble.js        (student vs AI differentiated styling)
│   └── ChatInput.js            (voice + text input)
├── /homework
│   ├── ImageUploader.js        (camera + gallery picker)
│   └── SolutionCard.js         (step-by-step display)
└── /dashboard
    ├── ProgressChart.js        (learning analytics)
    └── StreakCounter.js        (daily streak gamification)
```

---

### 3.2 API Gateway & Lambda Functions

#### API Design

##### Authentication Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/auth/signup` | Register new user |
| POST | `/auth/login` | Login with phone/email |
| POST | `/auth/refresh` | Refresh JWT token |
| POST | `/auth/logout` | Logout user |

##### Learning Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/chat/message` | Send text/voice query |
| GET | `/chat/history` | Retrieve past conversations |
| POST | `/homework/upload` | Upload homework image |
| GET | `/homework/{id}` | Get homework solution |
| POST | `/practice/generate` | Generate practice questions |
| GET | `/topics` | Get available topics by grade/subject |

##### User Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/user/profile` | Get user profile |
| PUT | `/user/profile` | Update profile |
| GET | `/user/progress` | Get learning analytics |
| GET | `/user/streak` | Get daily streak data |

##### Content Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/lessons/{topicId}` | Get lesson content |
| POST | `/lessons/download` | Download offline package |
| GET | `/exams/predicted` | Get predicted exam questions |

#### Lambda Functions

##### 1. AuthHandler (Node.js 18)
- Integrates with Cognito for authentication
- Generates and validates JWT tokens
- Rate limiting: 10 requests/minute per user

##### 2. ChatHandler (Node.js 18)
- Main orchestrator for conversational AI
- Calls Bedrock with context-aware prompts
- Implements conversation memory (last 10 exchanges)
- Timeout: 30 seconds

##### 3. VoiceHandler (Python 3.11)
- Receives audio stream from client
- Calls AWS Transcribe for speech-to-text
- Detects language (Hindi/English)
- Returns transcribed text with confidence score

##### 4. ImageHandler (Python 3.11)
- Receives homework images from S3 event trigger
- Calls Rekognition for text extraction (OCR)
- Identifies question type using regex patterns
- Forwards to appropriate solver function

##### 5. SolutionGenerator (Node.js 18)
- Uses Bedrock to generate step-by-step solutions
- Formats output in structured JSON
- Stores solution in DynamoDB
- Returns solution ID to client

##### 6. ContextEngine (Python 3.11)
- Retrieves user profile and learning history from DynamoDB
- Determines user's location context (city/rural/state)
- Selects appropriate examples based on context
- Caches context for 1 hour to reduce DB calls

##### 7. EmotionAnalyzer (Python 3.11)
- Analyzes voice tone using Transcribe sentiment
- Detects frustration patterns (repeated questions, tone change)
- Updates teaching strategy flag in user session
- Triggers adaptive response in ChatHandler

##### 8. ProgressTracker (Node.js 18)
- Records learning events (questions asked, time spent, topics)
- Aggregates daily/weekly statistics
- Calculates knowledge gaps based on incorrect answers
- Generates progress reports

##### 9. OfflinePackager (Node.js 18)
- Bundles lessons, images, audio for offline use
- Compresses package using gzip
- Uploads to S3 with CloudFront distribution
- Returns download URL with expiry (24 hours)

#### Lambda Configuration

| Function | Memory | Timeout | Concurrency |
|----------|--------|---------|-------------|
| ChatHandler | 512 MB | 30s | Reserved: 100 |
| SolutionGenerator | 512 MB | 30s | Default |
| VoiceHandler | 256 MB | 15s | Default |
| ImageHandler | 256 MB | 15s | Default |
| Others | 256 MB | 15s | Default |

---

### 3.3 AI Integration Layer

#### Amazon Bedrock Integration

**Model:** Claude Sonnet 4.5 (claude-sonnet-4-5-20250929)

##### Prompt Engineering Strategy

**System Prompt Template:**
```
You are SnapLearn AI, a friendly and patient tutor for Indian students aged 12-18. 

STUDENT CONTEXT:
- Name: {student_name}
- Grade: {grade}
- Location: {city}, {state} ({urban/rural})
- Subjects: {subjects}
- Recent topics: {recent_topics}
- Struggle areas: {weak_topics}

TEACHING STYLE:
1. Always use simple, age-appropriate language
2. Adapt examples to student's location:
   - Rural students: Use farm, village, nature examples
   - Urban students: Use city, technology, sports examples
3. Encourage with positive reinforcement
4. Break complex topics into small steps
5. Ask clarifying questions if student seems confused
6. Use Hindi-English code-switching naturally

When explaining {topic}, remember the student is from {location_context}.
```

**User Message Format:**
```json
{
  "query": "Photosynthesis kya hai?",
  "language": "hi-en",
  "context": {
    "grade": 9,
    "subject": "Biology",
    "location": "rural_bihar",
    "previous_topics": ["cell_structure", "plant_tissues"],
    "emotional_state": "neutral"
  }
}
```

**Response Parsing:**
- Extract main explanation
- Identify if follow-up questions needed
- Generate related practice problems
- Format with proper line breaks for voice synthesis

#### AWS Transcribe Configuration

| Setting | Value |
|---------|-------|
| Language | hi-IN (Hindi - India) |
| Custom vocabulary | Educational terms (photosynthesis → प्रकाश संश्लेषण) |
| Auto language detection | Enabled for code-switching |
| Output format | JSON with word-level timestamps |
| Confidence threshold | 0.8 (retry if below) |

**Real-time Streaming:**
- WebSocket connection to Transcribe Streaming API
- Chunk size: 0.5 seconds of audio
- Interim results: Display partial transcription
- Final result: Send to ChatHandler

#### AWS Polly Configuration

| Setting | Value |
|---------|-------|
| Voice | Aditi (Hindi, female, neural engine) |
| Speech rate | 100% (normal) |
| Volume | 100% |
| Output format | MP3, 24kbps |
| SSML support | Yes (for emphasis, pauses) |

**SSML Enhancement Example:**
```xml
<speak>
  <prosody rate="slow">
    Photosynthesis <break time="0.5s"/> ek process hai
  </prosody>
  <emphasis level="strong">jahaan plants</emphasis> 
  sunlight se apna khana banate hain.
</speak>
```

#### Amazon Rekognition Configuration

**Text Detection:**
| Setting | Value |
|---------|-------|
| Feature | DetectText API |
| Min confidence | 80% |
| Language hints | ['en', 'hi'] |
| Filters | Detect only printed text (ignore handwriting for MVP) |

**Object Detection:**
| Setting | Value |
|---------|-------|
| Feature | DetectLabels API |
| Max labels | 10 |
| Min confidence | 75% |
| Categories | Plants, Animals, Objects, Scenes |

**Custom Labels (Phase 2):**
- Train model on Indian textbook images
- Identify math equations, diagrams, charts
- Accuracy target: 95%+

---

### 3.4 Data Layer

#### DynamoDB Tables

##### Table 1: Users

| Attribute | Type | Description |
|-----------|------|-------------|
| **user_id** (PK) | String | UUID |
| name | String | User's full name |
| phone_number | String | Phone number (GSI 1) |
| email | String | Optional email (GSI 2) |
| grade | Number | Class/grade (6-12) |
| subjects | List<String> | Selected subjects |
| location | Map | {city, state, type: urban/rural} |
| created_at | Number | Timestamp |
| last_login | Number | Timestamp |
| subscription | String | free/premium |

##### Table 2: LearningHistory

| Attribute | Type | Description |
|-----------|------|-------------|
| **user_id** (PK) | String | User identifier |
| **timestamp** (SK) | Number | Event timestamp |
| session_id | UUID | Session identifier |
| topic | String | Topic studied |
| subject | String | Subject area |
| questions_asked | Number | Count of questions |
| time_spent | Number | Seconds |
| emotional_state | String | Detected emotion |
| comprehension_score | Number | 0-100 score |

##### Table 3: Conversations

| Attribute | Type | Description |
|-----------|------|-------------|
| **user_id** (PK) | String | User identifier |
| **timestamp** (SK) | Number | Message timestamp |
| message_id | UUID | Message identifier |
| sender | String | user/ai |
| content | String | Message text |
| audio_url | String | Optional S3 link |
| context | Map | Conversation context |

**TTL:** 30 days

##### Table 4: HomeworkSolutions

| Attribute | Type | Description |
|-----------|------|-------------|
| **solution_id** (PK) | UUID | Solution identifier |
| user_id | String | User identifier (GSI 1) |
| image_url | String | S3 link to homework image |
| question_text | String | Extracted OCR text |
| solution_steps | List<Map> | Step-by-step solution |
| created_at | Number | Timestamp |

##### Table 5: ProgressMetrics

| Attribute | Type | Description |
|-----------|------|-------------|
| **user_id** (PK) | String | User identifier |
| **date** (SK) | String | YYYY-MM-DD format |
| total_time | Number | Seconds studied |
| topics_covered | List<String> | Topics list |
| questions_asked | Number | Question count |
| homework_completed | Number | Homework count |
| streak_count | Number | Current streak |

#### DynamoDB Optimization
- On-demand billing for MVP (switch to provisioned after analysis)
- Point-in-time recovery enabled
- Encryption at rest with AWS-managed keys
- DynamoDB Streams for analytics pipeline

#### S3 Bucket Structure

##### Bucket 1: snaplearn-user-content
```
/audio/{user_id}/{timestamp}.mp3
/images/homework/{user_id}/{timestamp}.jpg
/images/visual-learning/{user_id}/{timestamp}.jpg
```

**Lifecycle Policy:**
- Transition to Glacier after 90 days
- Delete after 1 year

##### Bucket 2: snaplearn-static-content
```
/lessons/{subject}/{topic}/
/practice-questions/{subject}/{topic}/
/offline-packages/{grade}/{subject}/
```

**CloudFront Distribution:**
- Origin: snaplearn-static-content
- Cache behavior: Cache for 24 hours
- Geo-restriction: India only
- HTTPS only

---

### 3.5 Authentication & Authorization

#### AWS Cognito User Pool

| Configuration | Value |
|---------------|-------|
| Sign-in | Phone number OR email |
| MFA | Optional SMS-based OTP |
| Password policy | Min 8 characters, 1 number |
| Account recovery | SMS to phone |

**User Attributes:**
- phone_number (required)
- email (optional)
- custom:grade (Number)
- custom:role (student/teacher/parent)

**Identity Pool:**
- Federated identity: Google Sign-In, Apple Sign-In
- Unauthenticated access: None (all features require login)

**JWT Token Structure:**
```json
{
  "sub": "user_id",
  "cognito:groups": ["students"],
  "custom:grade": "9",
  "custom:role": "student",
  "exp": 1234567890
}
```

**Authorization Logic:**
- API Gateway: Cognito Authorizer validates JWT
- Lambda: Check user role and permissions
- Rate limiting: 100 requests/minute per user

---

### 3.6 Event-Driven Architecture

#### EventBridge Rules

##### Rule 1: Daily Progress Summary
| Setting | Value |
|---------|-------|
| Schedule | cron(0 20 * * ? *) - 8 PM IST daily |
| Target | ProgressSummaryFunction |
| Action | Generate daily report, send to parent via email/SMS |

##### Rule 2: Homework Uploaded
| Setting | Value |
|---------|-------|
| Source | S3 PutObject event |
| Filter | Bucket = snaplearn-user-content, Prefix = /images/homework/ |
| Target | ImageHandler Lambda |
| Action | Process homework image |

##### Rule 3: Streak About to Break
| Setting | Value |
|---------|-------|
| Schedule | cron(0 19 * * ? *) - 7 PM IST daily |
| Target | StreakReminderFunction |
| Action | Check users with active streaks who haven't logged in, send reminder |

##### Rule 4: Low Comprehension Alert
| Setting | Value |
|---------|-------|
| Source | Custom event from ProgressTracker |
| Filter | comprehension_score < 50 |
| Target | AlertTeacherFunction |
| Action | Notify teacher about struggling student |

#### Step Functions Workflow (Complex Homework Solving)

**State Machine:** HomeworkSolver

```
┌─────────────────┐
│  UploadToS3     │ → Store image
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  ExtractText    │ → Call Rekognition OCR
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│IdentifyQuestion │ → Regex parsing
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  ChoiceState    │ → Route to Math/Science/Essay solver
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│GenerateSolution │ → Call appropriate Bedrock prompt
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  FormatOutput   │ → Structure JSON response
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ StoreSolution   │ → Save to DynamoDB
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  NotifyUser     │ → Push notification
└─────────────────┘
```

**Error Handling:**
- Retry: 3 attempts with exponential backoff
- Catch: Send to Dead Letter Queue (DLQ) for manual review

---

## 4. Monitoring & Observability

### CloudWatch Dashboards

#### Dashboard 1: Application Health
| Widget | Metrics |
|--------|---------|
| Lambda Invocations | Invocation count (all functions) |
| Lambda Error Rate | Error rate (%) |
| API Gateway Errors | 4xx/5xx errors |
| DynamoDB Capacity | Read/write capacity |
| Response Time | Average response time |

#### Dashboard 2: User Engagement
| Widget | Metrics |
|--------|---------|
| Active Users | Hourly active users |
| Questions | Questions asked per hour |
| Homework | Homework uploads per hour |
| Session Duration | Average session duration |
| Conversion Funnel | Signup → First question |

#### Dashboard 3: AI Performance
| Widget | Metrics |
|--------|---------|
| Bedrock Latency | API latency |
| Transcribe Accuracy | Tracked via feedback |
| Polly Time | Synthesis time |
| Rekognition Scores | Confidence scores |

### CloudWatch Alarms

| Alarm | Metric | Threshold | Action |
|-------|--------|-----------|--------|
| High Error Rate | Lambda errors | > 5% in 5 minutes | SNS notification to dev team |
| Slow Response | API Gateway latency | > 5 seconds for 3 periods | SNS + auto-scale Lambda |
| DB Throttling | DynamoDB throttled requests | > 10 in 1 minute | Switch to on-demand mode |

### AWS X-Ray Tracing
- Enabled for all Lambda functions
- Traces end-to-end request flow
- Identifies bottlenecks and errors
- Retention: 30 days

### Logging Strategy
| Aspect | Configuration |
|--------|---------------|
| Format | Structured JSON |
| Log levels | INFO, WARN, ERROR |
| Sensitive data | Masked (phone numbers, email) |
| Retention | 7 days (hot), 90 days (cold in S3) |

---

## 5. Deployment & CI/CD

### Infrastructure as Code (AWS CDK)

**Stack Structure:**
```
/lib
├── network-stack.ts      (VPC, subnets if needed)
├── storage-stack.ts      (S3, DynamoDB)
├── compute-stack.ts      (Lambda, API Gateway)
├── ai-stack.ts           (Bedrock, Transcribe, Polly)
├── auth-stack.ts         (Cognito)
└── monitoring-stack.ts   (CloudWatch, X-Ray)
```

**CDK Benefits:**
- Version-controlled infrastructure
- Reusable constructs
- Automatic rollback on failures
- Cross-stack references

### CI/CD Pipeline (GitHub Actions)

```
┌─────────┐   ┌─────────┐   ┌─────────┐
│  Lint   │ → │  Test   │ → │  Build  │
└─────────┘   └─────────┘   └─────────┘
                               │
                               ▼
┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│ Deploy Dev  │ → │  E2E Tests  │ → │Deploy Stage │
└─────────────┘   └─────────────┘   └─────────────┘
                                          │
                                          ▼
                    ┌─────────────────────────────┐
                    │     Manual Approval         │
                    └─────────────┬───────────────┘
                                  │
                                  ▼
                    ┌─────────────┐   ┌─────────────┐
                    │ Deploy Prod │ → │ Smoke Tests │
                    └─────────────┘   └─────────────┘
```

**Stages:**
1. **Lint:** ESLint for JS/TS, Pylint for Python
2. **Test:** Unit tests with Jest, Integration tests
3. **Build:** Compile TypeScript, bundle Lambda functions
4. **Deploy Dev:** CDK deploy to development environment
5. **E2E Tests:** Run Cypress tests against dev
6. **Deploy Staging:** CDK deploy to staging
7. **Manual Approval:** Required for production
8. **Deploy Prod:** CDK deploy to production
9. **Smoke Tests:** Basic health checks

### Environments

| Environment | Description |
|-------------|-------------|
| Development | Single AWS account, reduced resources |
| Staging | Mirrors production, separate AWS account |
| Production | Multi-AZ, auto-scaling enabled |

### Blue-Green Deployment
- Lambda Aliases: "blue" and "green"
- Traffic shifting: 10% → 50% → 100% over 1 hour
- Auto-rollback if error rate > 2%

---

## 6. Security Architecture

### Principles
- **Least privilege:** IAM roles grant minimum permissions
- **Defense in depth:** Multiple security layers
- **Encryption everywhere:** At rest and in transit
- **Audit all actions:** CloudTrail logging

### Security Layers

```
┌───────────────────────────────────────────────────────────────┐
│ Layer 1: NETWORK SECURITY                                      │
│ API Gateway + AWS WAF | Rate limiting | CORS                   │
├───────────────────────────────────────────────────────────────┤
│ Layer 2: AUTHENTICATION                                        │
│ Cognito | JWT tokens | MFA                                     │
├───────────────────────────────────────────────────────────────┤
│ Layer 3: AUTHORIZATION                                         │
│ IAM roles | Resource policies | DynamoDB conditions            │
├───────────────────────────────────────────────────────────────┤
│ Layer 4: DATA PROTECTION                                       │
│ S3 encryption | DynamoDB encryption | Secrets management       │
├───────────────────────────────────────────────────────────────┤
│ Layer 5: APPLICATION SECURITY                                  │
│ Input validation | Output encoding | Parameterized queries     │
└───────────────────────────────────────────────────────────────┘
```

| Layer | Security Control | Implementation |
|-------|------------------|----------------|
| Network | WAF rules | Block common attacks |
| Network | Rate limiting | 1000 requests/minute per IP |
| Network | CORS | Restrict to mobile app domains |
| Auth | JWT tokens | Short-lived (1 hour), refresh (30 days) |
| Auth | MFA | Optional but encouraged for premium |
| Authz | IAM roles | Lambda execution roles with specific permissions |
| Authz | Resource policies | S3 buckets accessible only by Lambda |
| Data | S3 encryption | Server-side encryption (SSE-S3) |
| Data | DynamoDB encryption | AWS-managed keys |
| Data | Secrets | Parameter Store, encrypted |
| App | Input validation | Sanitize all user inputs |
| App | Output encoding | Prevent XSS in responses |

### Compliance

| Regulation | Requirement | Implementation |
|------------|-------------|----------------|
| GDPR | User data deletion | On-request deletion capability |
| POCSO | Age verification | Users < 18 require parental consent |
| Indian IT Act | Data localization | Mumbai region |

---

## 7. Scalability & Performance

### Scaling Strategy

| Type | Service | Mechanism |
|------|---------|-----------|
| Horizontal | Lambda | Auto-scales to 1000 concurrent executions |
| Horizontal | DynamoDB | On-demand billing scales automatically |
| Horizontal | S3 | Infinite scalability |
| Vertical | Lambda | Increase memory (memory = CPU allocation) |

### Caching Strategy

```
┌─────────────────────────────────────────────────────────────────┐
│ Level 1: CLIENT-SIDE (Mobile App)                               │
│ User profile, recent conversations (1 hour TTL)                 │
│ Static content (lessons, images) - indefinite                   │
├─────────────────────────────────────────────────────────────────┤
│ Level 2: API GATEWAY CACHE                                      │
│ User-agnostic GET requests | TTL: 5 minutes | 0.5 GB capacity   │
├─────────────────────────────────────────────────────────────────┤
│ Level 3: LAMBDA INTERNAL CACHE                                  │
│ Frequently accessed DynamoDB items | TTL: 5 minutes             │
├─────────────────────────────────────────────────────────────────┤
│ Level 4: CLOUDFRONT CDN                                         │
│ Static content (images, audio, lessons) | TTL: 24 hours         │
└─────────────────────────────────────────────────────────────────┘
```

### Database Optimization
- **DynamoDB:** Efficient key design (minimize scans)
- **GSI:** For alternate access patterns (query by phone/email)
- **DynamoDB Accelerator (DAX):** Phase 2 for sub-millisecond reads

### Content Delivery Optimization
- **CloudFront:** Edge locations across India
- **S3 Transfer Acceleration:** For large uploads (videos in Phase 2)
- **Image optimization:** Compress JPEGs to 80% quality

### Load Testing
| Aspect | Configuration |
|--------|---------------|
| Tools | Artillery, Locust |
| Target | 10,000 concurrent users |
| Metrics | Response time, error rate, throughput |
| Frequency | Before each major release |

---

## 8. Disaster Recovery & Backup

### Backup Strategy

#### DynamoDB
- Point-in-time recovery: Enabled (restore to any point in last 35 days)
- On-demand backups: Weekly full backups
- Cross-region replication: Enabled to Singapore region

#### S3
- Versioning: Enabled for user-content bucket
- Cross-region replication: Enabled to Singapore region
- Lifecycle: Old versions archived to Glacier after 90 days

### Recovery Objectives

| Objective | Target |
|-----------|--------|
| RTO (Recovery Time Objective) | 4 hours |
| RPO (Recovery Point Objective) | 1 hour |

### Disaster Scenarios

| Scenario | Detection | Recovery | Impact |
|----------|-----------|----------|--------|
| Single Lambda failure | CloudWatch alarm (< 1 min) | Automatic retry (< 30 sec) | Single request fails, retried |
| DynamoDB throttling | CloudWatch alarm (< 1 min) | Auto-switch to on-demand (< 5 min) | Slow responses for 5 min |
| Regional outage (Mumbai) | Multi-region health checks (< 5 min) | Failover to Singapore (manual, 4 hr) | Service unavailable for 4 hr |

### Business Continuity
- **Runbook:** Documented procedures for common failures
- **On-call rotation:** 24/7 coverage
- **Communication plan:** User-facing status page, social media updates

---

## 9. Cost Optimization

### AWS Free Tier (First 12 months)

| Service | Free Tier Allocation |
|---------|---------------------|
| Lambda | 1M requests/month, 400,000 GB-seconds compute |
| DynamoDB | 25 GB storage, 25 WCU, 25 RCU |
| S3 | 5 GB storage, 20,000 GET requests |
| Bedrock | Pay-per-use (no free tier) |

### Estimated Costs (After Free Tier, 10K users)

| Service | Calculation | Monthly Cost |
|---------|-------------|--------------|
| Lambda | 10K users × 10 requests/day × 30 days = 3M requests | ~$5 |
| DynamoDB | 10 GB storage, 10M reads, 2M writes | ~$15 |
| S3 | 100 GB storage (audio, images) | ~$5 |
| Bedrock | 5M input tokens, 10M output tokens (Claude Sonnet) | ~$25 |
| Transcribe | 100K minutes/month | ~$50 |
| Polly | 50K requests/month | ~$20 |
| Rekognition | 50K image analyses/month | ~$50 |
| CloudFront | 500 GB data transfer, 5M requests | ~$40 |

**Total estimated cost: ~$210/month for 10K users = ₹0.70 per user/month**

### Cost Optimization Strategies

| Strategy | Savings |
|----------|---------|
| Aggressive caching | Reduce Bedrock calls by 50% |
| Image compression before S3 upload | Reduce storage by 60% |
| DynamoDB TTL for old conversations | Reduce storage |
| Reserved Lambda concurrency (peak hours only) | Reduce cost by 20% |
| CloudFront optimization | Minimize data transfer |

---

## 10. Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | February 2026 | SnapLearn Team | Initial design document |

---

*This document is part of the SnapLearn project for the AWS AI for Bharat 2025 Hackathon.*
