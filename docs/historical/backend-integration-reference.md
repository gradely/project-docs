# Historical backend integration reference

> Status: retained for historical context. This document contains examples and operational assertions that have **not** all been verified against current source or deployed environments. Use the verified architecture documents for current guidance. Do not treat URLs, ports, authentication mechanisms, database schema examples, deployment configuration, or code snippets here as production contracts.

Original material follows unchanged for reference.

---

# Gradely Backend Services - Technical Integration Guide

This document provides detailed technical information about how the three Gradely backend services integrate and communicate with each other.

## 🏗️ Service Architecture

### Service Overview

| Service                  | Technology      | Port      | Purpose           | Database      |
| ------------------------ | --------------- | --------- | ----------------- | ------------- |
| **Gradely API v2**       | PHP 7.4+ / Yii2 | 8080      | Legacy API & Auth | MySQL         |
| **Gradely-2.1**          | Go / Gin        | 8081      | Core Platform     | MySQL + Redis |
| **Notification Service** | Go / Gin        | 8080/4000 | Communications    | MySQL         |

## 🔄 Service Communication Flow

### 1. Authentication Flow

```
Client → Gradely API v2 → Gradely-2.1 → Database
```

### 2. Core Operations Flow

```
Client → Gradely-2.1 → Database
        ↓
   Notification Service
```

### 3. Notification Flow

```
Gradely-2.1 → Notification Service → External APIs (Firebase, Twilio, etc.)
```

## 🔧 Technical Integration Details

### Gradely-2.1 → Notification Service Integration

#### Configuration

```yaml
# config.yml in Gradely-2.1
notification:
  baseUrl: "http://localhost:8080" # Notification service URL
```

#### HTTP Client Implementation

```go
// service/notification/general.go
type NotificationModel struct {
    ActionName string                 `json:"action_name"`
    ReceiverID string                 `json:"receiver_id,omitempty"`
    ActionData map[string]interface{} `json:"action_data"`
}

func (payload *NotificationModel) SendNotification(cfg *config.Configuration) error {
    // Check blacklist first
    err := payload.CheckBlacklistContact(cfg)
    if err != nil {
        return err
    }

    // Send notification
    url := cfg.Notification.BaseUrl + "/notification/v2.1"
    response, err := payload.sendRequest("POST", url, payload)

    return err
}
```

#### API Endpoints Used

- `POST /notification/v2.1/` - Send notification
- `POST /notification/v2.1/check-blacklist-contact` - Check user preferences
- `POST /notification/v2.1/test` - Test notification delivery

### Gradely API v2 → Gradely-2.1 Integration

#### Configuration

```php
// config/var.php in Gradely API v2
'gradely21' => [
    'baseUrl' => 'http://localhost:8081',
    'apiKey' => 'your-api-key',
],
```

#### HTTP Client Usage

```php
// Example integration call
$response = Utility::MakeRequest(
    "http://localhost:8081/v2.1/notification/auth/tutor_joins_class",
    "POST",
    ['session_id' => $sessionId],
    Yii::$app->request->headers->get('Authorization')
);
```

## 📡 API Communication Patterns

### 1. Notification Service Integration

#### Request Format

```json
{
	"action_name": "new_homework_parent",
	"receiver_id": "12345",
	"action_data": {
		"student_name": "John Doe",
		"subject": "Mathematics",
		"homework_title": "Algebra Practice",
		"due_date": "2024-01-15",
		"teacher_name": "Mrs. Smith",
		"email": "parent@example.com",
		"phone": "+1234567890",
		"device_id": "firebase_device_token"
	}
}
```

#### Response Format

```json
{
	"status": "success",
	"message": "Notification sent successfully",
	"data": {
		"notification_id": "uuid",
		"channels_sent": ["email", "push"],
		"timestamp": "2024-01-15T10:30:00Z"
	}
}
```

### 2. Blacklist Check Integration

#### Request Format

```json
[
	{
		"contact_type": "email",
		"contact": "user@example.com",
		"action_name": "new_homework_parent"
	}
]
```

#### Response Format

```json
{
	"status": "success",
	"data": [
		{
			"contact_type": "email",
			"contact": "user@example.com",
			"blacklisted": false,
			"preference": "reminder",
			"blacklisted_at": null
		}
	]
}
```

## 🗄️ Database Integration

### Shared Database Schema

#### User Management

```sql
-- Shared across services
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(255) UNIQUE,
    phone VARCHAR(20),
    user_type ENUM('student', 'teacher', 'parent', 'school'),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### Notification Preferences

```sql
-- Notification service specific
CREATE TABLE user_preference (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    email BOOLEAN DEFAULT TRUE,
    sms BOOLEAN DEFAULT TRUE,
    push BOOLEAN DEFAULT TRUE,
    pause_all_notifications BOOLEAN DEFAULT FALSE,
    security BOOLEAN DEFAULT TRUE,
    weekly_progress_report BOOLEAN DEFAULT TRUE,
    product_update BOOLEAN DEFAULT TRUE,
    offer BOOLEAN DEFAULT TRUE,
    newsletter BOOLEAN DEFAULT TRUE,
    reminder BOOLEAN DEFAULT TRUE
);
```

### Database Connections

#### Gradely-2.1 Database Config

```yaml
database:
  driver: mysql
  dbname: gradely
  campaignDbname: gradely_campaign
  handlerDbname: gradely_handler
  gameDbname: gradely_game
  appDbname: gradely_apps
  username: admin
  password: YOUR_DATABASE_PASSWORD
  host: YOUR_DATABASE_HOST
  port: "3306"
  logmode: true
```

#### Notification Service Database Config

```yaml
database:
  driver: mysql
  dbname: gradely_notifications
  username: admin
  password: YOUR_DATABASE_PASSWORD
  host: YOUR_DATABASE_HOST
  port: 3306
  logmode: false
```

## 🔐 Authentication & Security

### JWT Token Flow

1. **Gradely API v2** generates JWT tokens
2. **Gradely-2.1** validates tokens for core operations
3. **Notification Service** uses API keys for service-to-service communication

### API Key Authentication

```go
// Notification service authentication
req.Header.Add("X-API-Key", "your-api-key")
req.Header.Add("Content-Type", "application/json")
```

### CORS Configuration

```go
// Gradely-2.1 CORS middleware
r.Use(middleware.CORS())
r.Use(middleware.Security())
r.Use(middleware.MyLimit(cfg))
```

## 📊 Monitoring & Logging

### Health Check Endpoints

- **Gradely API v2**: `GET /docs/health`
- **Gradely-2.1**: `GET /health`
- **Notification Service**: `GET /`

### Logging Configuration

```yaml
# Gradely-2.1
params:
  environment: production
  debug: false
  sentrydsn: "your-sentry-dsn"

# Notification Service
params:
  environment: production
  debug: false
  sentrydsn: "your-sentry-dsn"
```

## 🚀 Deployment Configuration

### Docker Compose Example

```yaml
version: "3.8"
services:
  gradely-api:
    build: ./gradely-api
    ports:
      - "8080:8080"
    environment:
      - DB_HOST=mysql
      - REDIS_HOST=redis

  gradely-2.1:
    build: ./gradely-2.1
    ports:
      - "8081:8081"
    environment:
      - DB_HOST=mysql
      - REDIS_HOST=redis
      - NOTIFICATION_BASE_URL=http://notification:8080

  notification-service:
    build: ./notification-v2.1
    ports:
      - "8080:8080"
    environment:
      - DB_HOST=mysql
      - GRADELY21_URL=http://gradely-2.1:8081

  mysql:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=password
      - MYSQL_DATABASE=gradely

  redis:
    image: redis:7-alpine
```

### Environment Variables

#### Gradely-2.1

```bash
DB_HOST=localhost
DB_PORT=3306
DB_USER=admin
DB_PASSWORD=your_password
REDIS_HOST=localhost
REDIS_PORT=6379
NOTIFICATION_BASE_URL=http://localhost:8080
JWT_SECRET=your_jwt_secret
```

#### Notification Service

```bash
DB_HOST=localhost
DB_PORT=3306
DB_USER=admin
DB_PASSWORD=your_password
GRADELY21_URL=http://localhost:8081
FIREBASE_PROJECT_ID=your_project_id
TWILIO_ACCOUNT_SID=your_twilio_sid
TWILIO_AUTH_TOKEN=your_twilio_token
```

## 🔧 Development Tools

### API Testing

```bash
# Test Gradely-2.1 endpoints
curl -X GET http://localhost:8081/v2.1/general/user \
  -H "Authorization: Bearer <jwt_token>"

# Test Notification Service
curl -X POST http://localhost:8080/notification/v2.1/ \
  -H "Content-Type: application/json" \
  -H "X-API-Key: your-api-key" \
  -d '{"action_name": "test_notification", "action_data": {}}'
```

### Database Migrations

```bash
# Gradely-2.1
make migrate_up

# Notification Service
go run main.go migrate
```

## 🐛 Troubleshooting

### Common Issues

#### Service Communication Failures

```bash
# Check service connectivity
curl -f http://localhost:8081/health
curl -f http://localhost:8080/
```

#### Database Connection Issues

```bash
# Test MySQL connection
mysql -h localhost -u admin -p -e "SELECT 1"

# Test Redis connection
redis-cli ping
```

#### Notification Delivery Issues

```bash
# Check notification service logs
docker logs notification-service

# Test Firebase configuration
curl -X POST http://localhost:8080/notification/v2.1/test \
  -H "Content-Type: application/json" \
  -d '{"action_name": "test_notification_on_device", "action_data": {"device_id": "test_token"}}'
```

## 📈 Performance Optimization

### Caching Strategy

- **Redis**: Session management and API response caching
- **Database**: Query optimization and indexing
- **HTTP**: Connection pooling and keep-alive

### Rate Limiting

```go
// Gradely-2.1 rate limiting
r.Use(middleware.MyLimit(cfg)) // 100 requests per minute
```

### Background Processing

```go
// Notification service background processing
go func() {
    // Process notifications asynchronously
    processNotificationQueue()
}()
```

---

**For more detailed information about each service, please refer to their individual repositories and documentation.**
