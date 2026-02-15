# SnapLearn - Product Requirements Document

## 1. Project Overview

**SnapLearn** is a voice-first, multilingual AI-powered learning platform designed for 156 million students in rural and semi-urban India who cannot afford private tuition (₹50,000/year). It uses AWS AI services to provide personalized, context-aware tutoring in Hindi and regional languages.

---

## 2. Target Users

### 2.1 Students
- **Demographics:** Class 6-12 in rural/semi-urban India
- **Language Preference:** Prefer Hindi over English
- **Tech Proficiency:** Basic smartphone users
- **Device:** Android smartphones (Android 8+)

### 2.2 Teachers
- **Role:** Government school teachers
- **Challenge:** Overwhelmed with 1:35 student-to-teacher ratios
- **Need:** Tools to provide personalized attention at scale

### 2.3 Parents
- **Concern:** Education quality for their children
- **Constraint:** Cannot afford private tuition (₹50,000/year)
- **Need:** Visibility into child's learning progress

---

## 3. Problem Statement

| # | Problem | Impact |
|---|---------|--------|
| 1 | 260 million students in India, but only 104 million can access quality education support | 156 million underserved students |
| 2 | 1:35 teacher-student ratio in government schools | No personalized attention |
| 3 | Private tuition costs ₹50,000/year | Unaffordable for 60% of families |
| 4 | 70% of students prefer learning in mother tongue but textbooks are in English | Language barrier to learning |
| 5 | Rural students don't see themselves in urban-centric examples | Location barrier to relatability |

---

## 4. Core Solution

An AI study companion that:

- ✅ Understands Hindi/English code-switching naturally
- ✅ Remembers student's grade, location, and learning history
- ✅ Adapts explanations based on context (city vs village examples)
- ✅ Works via voice (no typing needed)
- ✅ Provides instant homework help via photo upload
- ✅ Detects emotional state and adjusts teaching approach
- ✅ Works offline after downloading weekly content

---

## 5. User Stories

### 5.1 Student User Stories

| ID | User Story | Acceptance Criteria |
|----|------------|---------------------|
| US-S01 | As a student, I want to ask questions in Hindi using my voice so that I don't struggle with typing English | Voice input works in Hindi, transcription accuracy > 95% |
| US-S02 | As a student, I want to upload photos of my homework so that I can get step-by-step solutions with explanations | Photo upload, OCR extraction, step-by-step solution generation |
| US-S03 | As a student, I want to learn through examples from my village/farm so that concepts are easier to understand | Location-aware examples based on user profile |
| US-S04 | As a student, I want to get practice questions based on my weak areas so that I can improve faster | Personalized practice question generation |
| US-S05 | As a student, I want to access the app without internet so that connectivity doesn't block my learning | Offline mode with downloaded content |
| US-S06 | As a student, I want to see how I'm progressing over time so that I stay motivated | Progress dashboard with trends |

### 5.2 Teacher User Stories

| ID | User Story | Acceptance Criteria |
|----|------------|---------------------|
| US-T01 | As a teacher, I want to see which topics my students struggle with so that I can focus class time effectively | Class-level analytics dashboard |
| US-T02 | As a teacher, I want to get AI-generated practice questions so that I save preparation time | Question generation by topic and difficulty |
| US-T03 | As a teacher, I want to track individual student progress so that I can provide targeted help | Individual student progress reports |
| US-T04 | As a teacher, I want to receive alerts when a student is falling behind so that I can intervene early | Automated alert system |

### 5.3 Parent User Stories

| ID | User Story | Acceptance Criteria |
|----|------------|---------------------|
| US-P01 | As a parent, I want to monitor my child's study time and topics covered so that I know they're learning | Parent dashboard with activity summary |
| US-P02 | As a parent, I want to receive reports on my child's strengths and weaknesses so that I can support them | Weekly progress reports |
| US-P03 | As a parent, I want to ensure the app is age-appropriate and safe so that I trust the platform | Content moderation, parental controls |

---

## 6. Functional Requirements

### FR1: Voice Input & Output

| ID | Requirement | Priority |
|----|-------------|----------|
| FR1.1 | Capture audio input via device microphone | P0 |
| FR1.2 | Convert Hindi/English speech to text using AWS Transcribe | P0 |
| FR1.3 | Support code-switching (mixing Hindi-English in same sentence) | P0 |
| FR1.4 | Convert text responses to natural Hindi voice using AWS Polly | P0 |
| FR1.5 | Provide playback controls (pause, replay, speed adjustment) | P1 |

### FR2: Context-Aware Learning Engine

| ID | Requirement | Priority |
|----|-------------|----------|
| FR2.1 | Store user profile (grade, location, subject preferences) in DynamoDB | P0 |
| FR2.2 | Track learning history (topics studied, questions asked, time spent) | P0 |
| FR2.3 | Detect user's geographic location (city/rural/specific state) | P0 |
| FR2.4 | Adapt explanation examples based on location context | P0 |
| FR2.5 | Remember past struggles and adjust difficulty level | P1 |
| FR2.6 | Maintain conversation context across multiple questions | P0 |

### FR3: Visual Learning Assistant

| ID | Requirement | Priority |
|----|-------------|----------|
| FR3.1 | Allow users to take photos of any object using camera | P1 |
| FR3.2 | Use Amazon Rekognition to identify objects in photos | P1 |
| FR3.3 | Generate educational content about identified objects | P1 |
| FR3.4 | Link object to relevant curriculum topics (e.g., neem tree → botany) | P1 |
| FR3.5 | Create interactive lessons from real-world objects | P2 |

### FR4: Homework Helper

| ID | Requirement | Priority |
|----|-------------|----------|
| FR4.1 | Accept photo uploads of homework questions | P0 |
| FR4.2 | Use Amazon Rekognition to extract text from images (OCR) | P0 |
| FR4.3 | Identify question type (math, science, essay, etc.) | P0 |
| FR4.4 | Generate step-by-step solutions using Amazon Bedrock | P0 |
| FR4.5 | Provide explanations for each step, not just answers | P0 |
| FR4.6 | Generate 3-5 similar practice problems | P1 |

### FR5: Emotion Detection & Adaptive Teaching

| ID | Requirement | Priority |
|----|-------------|----------|
| FR5.1 | Analyze voice tone using AWS Transcribe sentiment analysis | P1 |
| FR5.2 | Detect frustration, confusion, or boredom indicators | P1 |
| FR5.3 | Adjust explanation complexity based on emotional state | P1 |
| FR5.4 | Offer alternative explanation methods when confusion detected | P1 |
| FR5.5 | Provide encouragement and motivational messages | P1 |

### FR6: Exam Prediction Engine

| ID | Requirement | Priority |
|----|-------------|----------|
| FR6.1 | Store and analyze 5 years of CBSE/state board exam papers | P2 |
| FR6.2 | Identify high-frequency topics and question patterns | P2 |
| FR6.3 | Generate predicted questions for upcoming exams | P2 |
| FR6.4 | Create personalized practice tests based on student's syllabus | P1 |
| FR6.5 | Provide difficulty ratings for each predicted question | P2 |

### FR7: Peer Learning Network

| ID | Requirement | Priority |
|----|-------------|----------|
| FR7.1 | Identify students struggling with same topics in nearby schools | P2 |
| FR7.2 | Auto-create virtual study groups (max 5 students) | P2 |
| FR7.3 | Enable students to ask/answer peer questions | P2 |
| FR7.4 | Implement gamification (points for helping others) | P2 |
| FR7.5 | Moderate content to ensure quality and safety | P2 |

### FR8: Offline Mode

| ID | Requirement | Priority |
|----|-------------|----------|
| FR8.1 | Allow users to download weekly lesson packages | P0 |
| FR8.2 | Store downloaded content in local device storage (IndexedDB) | P0 |
| FR8.3 | Enable full functionality without internet for downloaded content | P0 |
| FR8.4 | Sync learning progress when internet reconnects | P0 |
| FR8.5 | Show offline status indicator clearly | P0 |

### FR9: Progress Tracking & Analytics

| ID | Requirement | Priority |
|----|-------------|----------|
| FR9.1 | Track daily study time and topics covered | P0 |
| FR9.2 | Generate weekly progress reports | P0 |
| FR9.3 | Identify knowledge gaps and weak areas | P1 |
| FR9.4 | Show improvement trends over time | P1 |
| FR9.5 | Provide parent-facing dashboard with summary stats | P1 |

### FR10: Multi-language Support

| ID | Requirement | Priority |
|----|-------------|----------|
| FR10.1 | Support Hindi as primary language | P0 |
| FR10.2 | Support English for code-switching | P0 |
| FR10.3 | Phase 2: Add Tamil, Telugu, Bengali, Marathi, Gujarati | P2 |
| FR10.4 | Allow users to switch language mid-conversation | P1 |
| FR10.5 | Maintain consistent UI/UX across languages | P1 |

---

## 7. Non-Functional Requirements

### NFR1: Performance

| ID | Requirement | Target |
|----|-------------|--------|
| NFR1.1 | Voice-to-text latency | < 2 seconds |
| NFR1.2 | AI response generation (text) | < 3 seconds |
| NFR1.3 | AI response generation (voice) | < 5 seconds |
| NFR1.4 | Image recognition | < 4 seconds |
| NFR1.5 | App launch time | < 3 seconds |
| NFR1.6 | Concurrent user support | 100,000 users |

### NFR2: Scalability

| ID | Requirement | Target |
|----|-------------|--------|
| NFR2.1 | User scaling capacity | 10 million users |
| NFR2.2 | Database sharding strategy | DynamoDB partitioning |
| NFR2.3 | CDN for content delivery | CloudFront |
| NFR2.4 | Auto-scaling | Lambda-based |
| NFR2.5 | Regional deployments | Mumbai, Delhi, Bangalore |

### NFR3: Reliability & Availability

| ID | Requirement | Target |
|----|-------------|--------|
| NFR3.1 | Uptime SLA | 99.9% |
| NFR3.2 | Graceful degradation | If AI services unavailable |
| NFR3.3 | Automatic failover | For critical services |
| NFR3.4 | Data backup frequency | Every 6 hours |
| NFR3.5 | Disaster recovery RTO | 4 hours |

### NFR4: Security & Privacy

| ID | Requirement | Description |
|----|-------------|-------------|
| NFR4.1 | Encryption | End-to-end encryption for voice data |
| NFR4.2 | Compliance | GDPR and Indian data protection |
| NFR4.3 | PII handling | No PII stored without explicit consent |
| NFR4.4 | Access control | Role-based (student/teacher/parent) |
| NFR4.5 | Authentication | AWS Cognito |
| NFR4.6 | Content moderation | For peer interactions |
| NFR4.7 | Audit logs | For all data access |

### NFR5: Usability

| ID | Requirement | Description |
|----|-------------|-------------|
| NFR5.1 | Onboarding | No tutorial needed - intuitive first use |
| NFR5.2 | Input method | Voice-first UI - minimal typing |
| NFR5.3 | UI elements | Large buttons for low-resolution displays |
| NFR5.4 | Data usage | Works on 2G/3G (< 100KB/min) |
| NFR5.5 | Device support | Devices with 2GB RAM |
| NFR5.6 | Accessibility | Screen reader compatibility |

### NFR6: Maintainability

| ID | Requirement | Description |
|----|-------------|-------------|
| NFR6.1 | Architecture | Modular microservices |
| NFR6.2 | Documentation | Comprehensive API docs |
| NFR6.3 | Testing | Unit, integration, E2E |
| NFR6.4 | CI/CD | GitHub Actions pipeline |
| NFR6.5 | Infrastructure | AWS CDK (IaC) |
| NFR6.6 | Monitoring | CloudWatch and X-Ray |

### NFR7: Localization

| ID | Requirement | Description |
|----|-------------|-------------|
| NFR7.1 | RTL support | Future: Urdu |
| NFR7.2 | Currency | Indian Rupees (₹) |
| NFR7.3 | Timezone | Indian Standard Time (IST) |
| NFR7.4 | Cultural sensitivity | In examples and content |
| NFR7.5 | Festivals | Festival-specific messages |

---

## 8. Technical Stack

### AWS AI Services
| Service | Purpose |
|---------|---------|
| Amazon Bedrock | Claude Sonnet 4.5 for conversational AI and content generation |
| AWS Transcribe | Speech-to-text with Hindi language pack |
| AWS Polly | Text-to-speech with Aditi voice (Hindi) |
| Amazon Rekognition | Image analysis and OCR |

### Backend Infrastructure
| Service | Purpose |
|---------|---------|
| AWS Lambda | Serverless functions for API logic |
| Amazon DynamoDB | NoSQL database for user data and learning history |
| Amazon S3 | Media storage (audio, images, lesson content) |
| Amazon CloudFront | CDN for content delivery |
| AWS API Gateway | RESTful API management |
| Amazon Cognito | User authentication and authorization |

### Frontend
| Technology | Purpose |
|------------|---------|
| React Native | Cross-platform mobile app (iOS & Android) |
| Redux | State management |
| WebRTC | Real-time voice capture |
| IndexedDB | Local offline storage |

### DevOps
| Tool | Purpose |
|------|---------|
| GitHub | Version control |
| GitHub Actions | CI/CD pipeline |
| AWS CloudWatch | Monitoring and logging |
| AWS X-Ray | Distributed tracing |
| AWS CDK | Infrastructure as Code |

---

## 9. Success Metrics

### 9.1 User Engagement

| Metric | Target |
|--------|--------|
| Daily Active Users (DAU) | 50% of registered users |
| Average session time | 25+ minutes |
| Questions asked per session | 5+ questions |
| 30-day retention rate | 60% |

### 9.2 Learning Outcomes

| Metric | Target |
|--------|--------|
| Concept comprehension improvement | 85%+ students report better understanding |
| Test score improvement | 15-20% average increase |
| Homework completion rate | 80%+ students complete daily homework |
| Practice question attempts | 10+ per week per student |

### 9.3 Platform Performance

| Metric | Target |
|--------|--------|
| Voice recognition accuracy (Hindi) | 95%+ |
| Response relevance | 90%+ rated "helpful" |
| App crash rate | < 0.1% |
| Average response time | < 3 seconds |

### 9.4 Business Metrics

| Metric | Target |
|--------|--------|
| User acquisition cost | < ₹50 per student |
| Conversion to premium | 15% within 3 months |
| School partnerships | 500 schools by Year 1 |
| Net Promoter Score (NPS) | 70+ |

---

## 10. Constraints & Assumptions

### 10.1 Constraints

| Category | Constraint |
|----------|------------|
| Budget | ₹11 lakhs for MVP (3 months development) |
| Infrastructure | AWS free tier limits for first 6 months |
| Team | 2 full-stack developers, 1 UI/UX designer |
| Timeline | MVP in 3 months, full launch in 6 months |
| Regulatory | Must comply with Indian IT Act 2000 and POCSO Act |

### 10.2 Assumptions

| # | Assumption |
|---|------------|
| 1 | Students have access to smartphones (Android 8+) |
| 2 | Internet connectivity available at least once per week for content download |
| 3 | Teachers willing to adopt technology for classroom support |
| 4 | Parents supportive of digital learning tools |
| 5 | AWS services remain stable and pricing consistent |

---

## 11. Risks & Mitigation

| Risk | Impact | Probability | Mitigation Strategy |
|------|--------|-------------|---------------------|
| Low adoption due to distrust of technology | High | Medium | Partner with trusted local NGOs, conduct free workshops in schools |
| Content quality concerns (AI hallucinations) | High | Medium | Human-in-the-loop review for curriculum content, feedback mechanism |
| Data privacy concerns from parents | Medium | High | Transparent privacy policy, local data storage option, no ads |
| AWS costs spiral with scale | High | Medium | Aggressive caching, optimize API calls, negotiate enterprise pricing |
| Competition from established players | Medium | High | Focus on unique features (voice-first, hyperlocal, offline), serve underserved market |

---

## 12. Future Enhancements (Phase 2)

| Enhancement | Description |
|-------------|-------------|
| Video Explanations | AI-generated video explanations |
| AR/VR | Augmented/Virtual reality for science experiments |
| Teacher Tools | Collaboration tools and dashboards |
| Parent Communication | Parent-teacher communication channel |
| Exam Mode | AI-powered doubt resolution during exams (supervised) |
| DIKSHA Integration | Integration with DIKSHA/NCERT platforms |
| Gamification | Badges, leaderboards, achievements |
| Live Sessions | Live doubt-solving sessions with AI tutors |

---

## 13. Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | February 2026 | SnapLearn Team | Initial requirements document |

---

*This document is part of the SnapLearn project for the AWS AI for Bharat 2025 Hackathon.*
