# SnapLearn - AI for Bharat Hackathon Submission

AI-powered learning assistant for rural Indian students

## Overview

SnapLearn is a voice-first, multilingual AI-powered learning platform designed for 156 million students in rural and semi-urban India who cannot afford private tuition. It uses AWS AI services to provide personalized, context-aware tutoring in Hindi and regional languages.

## Key Features

- **Voice-First Learning** - Ask questions in Hindi using voice, no typing needed
- **Homework Helper** - Upload photos of homework for step-by-step solutions
- **Context-Aware** - Adapts examples based on student's location (rural/urban)
- **Offline Mode** - Download weekly content for areas with poor connectivity
- **Progress Tracking** - Track learning progress and identify weak areas

## Documentation

- [requirements.md](requirements.md) - Product requirements document
- [design.md](design.md) - System design and architecture

## Tech Stack

- **Frontend:** React (Vite)
- **AI Services:** Amazon Bedrock (Claude Sonnet 4.5), AWS Transcribe, AWS Polly, Amazon Rekognition
- **Backend:** AWS Lambda, API Gateway
- **Database:** Amazon DynamoDB
- **Storage:** Amazon S3, CloudFront CDN

## Team

SnapLearn

## Hackathon

AWS AI for Bharat 2025

## Getting Started

```bash
# Install dependencies
npm install

# Run development server
npm run dev

# Build for production
npm run build
```

## License

MIT
