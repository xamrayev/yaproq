# CyberLab v1 Architecture Specification

**Version:** 1.0
**Status:** Draft
**Goal:** MVP для платформы практического обучения Web Security в полностью браузерной среде.

---

# 1. Vision

## Миссия

CyberLab позволяет изучать веб-безопасность через реальные практические сценарии без установки локального окружения.

Принцип:

> Student should never leave the browser.

Студент получает:

```text
Login
 ↓
Workspace
 ↓
Lab
 ↓
Investigation
 ↓
Evidence
 ↓
Assessment
```

без установки:

- Kali Linux
- Burp Suite
- Docker
- Virtual Machines
- VPN
- локального веб-сервера

---

# 2. Product Scope v1

## Входит в v1

### Пользователи

- регистрация
- логин
- профиль

### Курсы

- список курсов
- список лабораторий
- прогресс

### Workspace

- Browser Panel
- LinuxOnTab
- Mission Panel
- HTTP History

### Labs

- IDOR-001

### Assessment

- автоматическая проверка
- XP
- статусы выполнения

### Infrastructure

- изолированные контейнерные лаборатории

---

## Не входит в v1

- Marketplace
- AI Tutor
- Teacher Dashboard
- Adaptive Learning
- Certificates
- Multi-user Labs
- Active Directory Labs
- Cloud Labs

---

# 3. System Context

```text
                    USER
                      │
                      ▼
                 FRONTEND
                      │
                      ▼
                API GATEWAY
                      │
     ┌────────────────┼─────────────────┐
     │                │                 │
     ▼                ▼                 ▼
   AUTH          LEARNING         LAB ENGINE
                                      │
                                      ▼
                               ASSESSMENT
                                      │
                                      ▼
                               INFRASTRUCTURE
                                      │
                                      ▼
                           LAB CONTAINERS
```

---

# 4. Core Domains

## Identity Domain

Отвечает за:

```text
Users
Authentication
Sessions
Roles
```

### Роли

```text
student
admin
```

---

## Learning Domain

Отвечает за:

```text
Courses
Modules
Labs
Progress
```

---

## Lab Domain

Отвечает за:

```text
Lab Definition
Environment
Objectives
Hints
Assessment
```

---

## Assessment Domain

Отвечает за:

```text
Evidence
Success Detection
XP
Completion State
```

---

# 5. Workspace

Workspace является главным интерфейсом студента.

---

## Layout

```text
┌─────────────────────────────────────────────┐
│ CyberLab                       XP: 120      │
├──────────────┬───────────────────┬──────────┤
│              │                   │          │
│  MISSION     │     BROWSER       │  HTTP    │
│              │                   │ HISTORY  │
│              │                   │          │
├──────────────┴───────────────────┴──────────┤
│                                              │
│                TERMINAL                      │
│                                              │
└──────────────────────────────────────────────┘
```

---

## Core Applications

### Browser

Назначение:

```text
Target interaction
```

Функции:

- открытие лабораторного сайта
- cookies
- session storage
- local storage

---

### Terminal

Основан на LinuxOnTab.

Профиль:

```yaml
profile: web-beginner

tools:
  - curl
  - wget
  - jq
  - python3
  - openssl
```

---

### HTTP History

Функции:

```text
Request list
Response view
Headers
Body
Cookies
Search
```

---

### Mission Panel

Отображает:

- цель
- описание
- подсказки
- статус

Пример:

```text
Objective

Access another user's profile.

Hints

Hint 1
Hint 2
Hint 3

Evidence

□ Request
□ Response
□ Explanation
```

---

# 6. LinuxOnTab Integration

## Назначение

Linux-машина атакующего запускается внутри браузера.

---

## Стек

```text
LinuxOnTab
    │
    ▼
WebAssembly
    │
    ▼
Linux Kernel
    │
    ▼
Alpine Linux
```

---

## Конфигурация

```yaml
cpu: browser-side
ram_limit: 512MB

filesystem:
  persistent: true

network:
  provider: wisp
```

---

## Snapshots

Сохраняются:

```text
filesystem
home directory
terminal history
```

Хранилище:

```text
IndexedDB
```

---

# 7. Course Model

## Структура

```text
Course
 └── Module
      └── Lab
```

---

## Пример

```text
Web Security Foundations

 ├── HTTP Basics
 ├── Authentication
 ├── Authorization
 └── IDOR Lab
```

---

# 8. Lab Model

Каждая лаборатория описывается декларативно.

## lab.yaml

```yaml
id: idor-001

metadata:
  title: The Wrong User
  category: authorization

environment:
  attacker:
    profile: web-beginner

  target:
    image: cyberlab/idor-demo:v1

objective:
  description: >
    Access another user's profile.

success:
  conditions:
    - type: http_request
      method: GET
      path: /api/users/42

xp: 150
```

---

# 9. Lab Engine

Центральный компонент системы.

---

## Responsibilities

Lab Engine отвечает за:

```text
Lab startup
Lab lifecycle
Objective tracking
Success detection
Evidence validation
Events
```

---

## Lifecycle

```text
Created
 ↓
Starting
 ↓
Running
 ↓
Solved
 ↓
Stopped
 ↓
Destroyed
```

---

# 10. Infrastructure Model

Каждая лаборатория разворачивается в отдельном namespace.

---

## Example

```text
lab-123
│
├── web
├── api
└── db
```

---

## Isolation

```text
Internet Egress = Deny
```

По умолчанию.

---

## TTL

```yaml
ttl: 2h
inactive_timeout: 30m
```

---

# 11. Event Model

Каждое действие студента создаёт событие.

---

## Event Schema

```json
{
  "id": "uuid",
  "student_id": 42,
  "lab_id": "idor-001",
  "timestamp": "...",
  "type": "http_request"
}
```

---

## Supported Events

### Browser

```text
page_opened
form_submitted
```

### HTTP

```text
http_request
http_response
```

### Terminal

```text
command_executed
```

### Learning

```text
hint_opened
lab_started
lab_completed
```

### Assessment

```text
evidence_submitted
```

---

# 12. Assessment Engine

Оценивает выполнение лаборатории.

---

## Success Conditions

Пример:

```yaml
success:

  conditions:

    - type: http_request

      method: GET

      path: /api/users/42
```

Если событие совпадает:

```text
Lab Solved
```

---

## XP Rules

```yaml
base_xp: 150

hint_penalty:
  per_hint: 10
```

---

# 13. Evidence Model

## Evidence Types

### Request

```json
{
  "method": "GET",
  "path": "/api/users/42"
}
```

---

### Response

```json
{
  "status": 200
}
```

---

### Text Explanation

```text
Why is this vulnerability possible?
```

В v1 текст не оценивается автоматически.

Только сохраняется.

---

# 14. Database Schema

## users

```sql
id
email
password_hash
created_at
```

---

## courses

```sql
id
title
slug
```

---

## labs

```sql
id
slug
title
difficulty
xp
```

---

## user_labs

```sql
user_id
lab_id
started_at
completed_at
status
xp_earned
```

---

## events

```sql
id
user_id
lab_id
type
payload
created_at
```

---

## evidence

```sql
id
user_id
lab_id
type
content
created_at
```

---

# 15. API Contracts

## Start Lab

```http
POST /api/labs/{id}/start
```

Response:

```json
{
  "instance_id": "lab-123"
}
```

---

## Submit Evidence

```http
POST /api/labs/{id}/evidence
```

---

## Get Progress

```http
GET /api/progress
```

---

## HTTP Events Stream

```http
GET /api/labs/{id}/events
```

(WebSocket)

---

# 16. Repository Structure

```text
cyberlab/

├── apps/
│   ├── web/
│   └── api/
│
├── packages/
│   ├── ui/
│   ├── protocol/
│   └── types/
│
├── services/
│   ├── lab-engine/
│   ├── assessment/
│   └── scheduler/
│
├── infrastructure/
│   ├── postgres/
│   ├── redis/
│   └── kubernetes/
│
├── labs/
│   └── idor-001/
│
└── docs/
```

---

# 17. Technology Stack

## Frontend

```text
Next.js
React
TypeScript
Tailwind
Monaco
xterm.js
TanStack Query
```

---

## Backend

```text
Go
Gin/Fiber
gRPC
WebSockets
```

---

## Data

```text
PostgreSQL
Redis
```

---

## Infrastructure

```text
Kubernetes
containerd
Helm
```

---

## Observability

```text
OpenTelemetry
Prometheus
Grafana
Loki
```

---

# 18. Reference Lab: IDOR-001

## Objective

```text
Access another user's profile.
```

---

## Scenario

Студент входит как:

```text
user01
```

и обнаруживает запрос:

```http
GET /api/users/1
```

Изменяя идентификатор:

```http
GET /api/users/2
```

получает данные другой учётной записи.

---

## Success

```yaml
success:

  conditions:

    - type: http_request
      method: GET
      path: /api/users/2

    - type: response_status
      value: 200
```

---

# 19. Definition of Done

CyberLab v1 считается завершённым, если работают:

✅ Логин пользователя
✅ Каталог курсов
✅ IDOR-001 лаборатория
✅ LinuxOnTab интеграция
✅ Browser panel
✅ HTTP history
✅ Event collection
✅ Automatic success detection
✅ XP and progress tracking
✅ Isolated lab deployment

---

# 20. Roadmap After v1

```text
v1
 └── IDOR

v1.1
 └── XSS

v1.2
 └── SQL Injection

v1.3
 └── Authentication

v1.4
 └── SSRF

v2
 └── Competency Engine

v3
 └── AI Tutor

v4
 └── Adaptive Learning

v5
 └── Full Cyber Range
```

**Главная цель CyberLab v1:** доказать, что веб-безопасности можно эффективно обучаться в единой браузерной рабочей среде, где Linux-машина атакующего, цель, практическая лаборатория и система оценки объединены в один продукт.
