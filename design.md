# Technical System Design Document

## Socratic Companion – AI Tutor VS Code Extension

**Version:** 1.0  
**Date:** February 15, 2026  
**Status:** Design Phase

---

## 1. System Overview

Socratic Companion is a cloud-connected VS Code extension that transforms coding education through intelligent, Socratic-method guidance. The system monitors developer behavior in real-time, detects struggle patterns, generates contextual hints using AI, tracks mistakes for pattern analysis, and delivers spaced repetition quizzes to reinforce learning.

### Key Capabilities

- Real-time struggle detection through keystroke and behavior analysis
- AI-powered hint generation using Amazon Bedrock
- Persistent mistake tracking and pattern recognition
- Automated spaced repetition quiz scheduling
- Offline-capable core features with cloud sync

### Design Principles

- **Privacy-first:** User code stays local unless explicitly shared
- **Low-latency:** Sub-second response for critical interactions
- **Resilient:** Graceful degradation when cloud services unavailable
- **Scalable:** Serverless architecture supporting 100K+ concurrent users
- **Cost-effective:** Optimized API calls and caching strategies

---

## 2. Architecture Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     VS Code Extension                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Struggle   │  │     Hint     │  │     Quiz     │      │
│  │   Detector   │  │   Manager    │  │   Manager    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│  ┌──────────────────────────────────────────────────┐      │
│  │          Local Storage (IndexedDB)                │      │
│  └──────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            │ HTTPS
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                      AWS Cloud Layer                         │
│                                                               │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              Amazon API Gateway                       │  │
│  │         (REST API + WebSocket for real-time)         │  │
│  └──────────────────────────────────────────────────────┘  │
│                            │                                 │
│         ┌──────────────────┼──────────────────┐            │
│         ▼                  ▼                  ▼            │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐     │
│  │   Lambda    │   │   Lambda    │   │   Lambda    │     │
│  │  HintGen    │   │  Mistake    │   │  QuizGen    │     │
│  └─────────────┘   └─────────────┘   └─────────────┘     │
│         │                  │                  │            │
│         ▼                  ▼                  ▼            │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐     │
│  │   Amazon    │   │  DynamoDB   │   │  DynamoDB   │     │
│  │   Bedrock   │   │  (Mistakes) │   │   (Quizzes) │     │
│  └─────────────┘   └─────────────┘   └─────────────┘     │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │         S3 (Documentation Cache & Assets)            │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │    CloudWatch (Logging & Monitoring)                 │  │
│  └─────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

