# RoadTech - Developer Documentation

## Table of Contents

1. [Project Architecture](#1-project-architecture)
2. [Backend Development](#2-backend-development)
3. [Frontend Development](#3-frontend-development)
4. [Database Schema](#4-database-schema)
5. [API Reference](#5-api-reference)
6. [WebSocket Integration](#6-websocket-integration)
7. [Security Implementation](#7-security-implementation)
8. [Testing Guide](#8-testing-guide)
9. [Deployment](#9-deployment)
10. [Troubleshooting](#10-troubleshooting)

---

## 1. Project Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Client Layer                            │
│  ┌─────────────────┐  ┌─────────────────┐  ┌────────────────┐  │
│  │   React SPA     │  │   Leaflet Maps  │  │  STOMP Client  │  │
│  │   (Vite + TS)   │  │                 │  │  (WebSocket)   │  │
│  └────────┬────────┘  └────────┬────────┘  └───────┬────────┘  │
└───────────┼─────────────────────┼──────────────────┼────────────┘
            │                     │                  │
            │        HTTP/REST    │                  │ WS
            ▼                     ▼                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                       API Gateway / Nginx                       │
└───────────────────────────────┬─────────────────────────────────┘
                                │
┌───────────────────────────────┼─────────────────────────────────┐
│                         Backend Layer                           │
│  ┌─────────────────┐  ┌─────────────────┐  ┌────────────────┐  │
│  │  REST Controllers│  │ WebSocket Config│  │ Security Filter│  │
│  └────────┬────────┘  └────────┬────────┘  └───────┬────────┘  │
│           │                    │                   │            │
│           ▼                    ▼                   ▼            │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    Service Layer                         │   │
│  │  AuthService │ ServiceRequestService │ MechanicService   │   │
│  └─────────────────────────────┬───────────────────────────┘   │
│                                │                                │
│  ┌─────────────────────────────┼───────────────────────────┐   │
│  │                    Repository Layer                      │   │
│  │  UserRepo │ ServiceRequestRepo │ MechanicProfileRepo     │   │
│  └─────────────────────────────┬───────────────────────────┘   │
└────────────────────────────────┼────────────────────────────────┘
                                 │
┌────────────────────────────────┼────────────────────────────────┐
│                          Data Layer                             │
│                    ┌───────────┴───────────┐                    │
│                    │      MySQL 8.0        │                    │
│                    └───────────────────────┘                    │
└─────────────────────────────────────────────────────────────────┘
```

### Directory Structure

```
roadTech/
├── backend/
│   ├── src/main/java/com/roadtech/
│   │   ├── RoadTechApplication.java      # Main entry point
│   │   ├── config/
│   │   │   ├── SecurityConfig.java       # Spring Security config
│   │   │   ├── WebSocketConfig.java      # STOMP WebSocket config
│   │   │   └── OpenApiConfig.java        # Swagger configuration
│   │   ├── controller/
│   │   │   ├── AuthController.java       # /auth/* endpoints
│   │   │   ├── UserController.java       # /users/* endpoints
│   │   │   ├── ServiceRequestController.java
│   │   │   └── MechanicController.java   # /mechanic/* endpoints
│   │   ├── dto/
│   │   │   ├── auth/                     # Auth request/response DTOs
│   │   │   ├── request/                  # Service request DTOs
│   │   │   └── mechanic/                 # Mechanic DTOs
│   │   ├── entity/
│   │   │   ├── User.java
│   │   │   ├── MechanicProfile.java
│   │   │   ├── ServiceRequest.java
│   │   │   └── RefreshToken.java
│   │   ├── exception/
│   │   │   ├── GlobalExceptionHandler.java
│   │   │   ├── BadRequestException.java
│   │   │   ├── ResourceNotFoundException.java
│   │   │   └── UnauthorizedException.java
│   │   ├── repository/
│   │   │   ├── UserRepository.java
│   │   │   ├── MechanicProfileRepository.java
│   │   │   ├── ServiceRequestRepository.java
│   │   │   └── RefreshTokenRepository.java
│   │   ├── security/
│   │   │   ├── JwtService.java           # JWT generation/validation
│   │   │   ├── JwtAuthFilter.java        # Request filter
│   │   │   ├── CustomUserDetails.java
│   │   │   └── CustomUserDetailsService.java
│   │   ├── service/
│   │   │   ├── AuthService.java
│   │   │   ├── ServiceRequestService.java
│   │   │   ├── MechanicService.java
│   │   │   ├── LocationService.java
│   │   │   └── NotificationService.java
│   │   └── websocket/
│   │       ├── WebSocketAuthInterceptor.java
│   │       └── LocationController.java
│   └── src/main/resources/
│       ├── application.yml               # Main config
│       └── schema.sql                    # Database schema
│
├── frontend/
│   ├── src/
│   │   ├── api/
│   │   │   ├── client.ts                 # Axios instance + interceptors
│   │   │   ├── auth.ts                   # Auth API calls
│   │   │   ├── serviceRequest.ts         # Request API calls
│   │   │   └── mechanic.ts               # Mechanic API calls
│   │   ├── components/
│   │   │   ├── common/                   # Reusable UI components
│   │   │   │   ├── Button.tsx
│   │   │   │   ├── Input.tsx
│   │   │   │   ├── Card.tsx
│   │   │   │   └── Badge.tsx
│   │   │   ├── layout/
│   │   │   │   ├── Header.tsx
│   │   │   │   └── Layout.tsx
│   │   │   ├── maps/
│   │   │   │   ├── LocationPicker.tsx
│   │   │   │   └── LiveTrackingMap.tsx
│   │   │   └── mechanic/
│   │   │       └── RequestCard.tsx
│   │   ├── contexts/
│   │   │   ├── AuthContext.tsx           # Authentication state
│   │   │   └── WebSocketContext.tsx      # WebSocket connection
│   │   ├── hooks/
│   │   │   └── useLocationTracking.ts
│   │   ├── pages/
│   │   │   ├── auth/
│   │   │   │   ├── Login.tsx
│   │   │   │   └── Register.tsx
│   │   │   ├── user/
│   │   │   │   ├── Dashboard.tsx
│   │   │   │   ├── RequestAssistance.tsx
│   │   │   │   ├── RequestTracking.tsx
│   │   │   │   └── RequestHistory.tsx
│   │   │   └── mechanic/
│   │   │       ├── Dashboard.tsx
│   │   │       └── JobDetails.tsx
│   │   ├── types/
│   │   │   └── index.ts                  # TypeScript interfaces
│   │   ├── utils/
│   │   │   └── cn.ts                     # Tailwind class merge
│   │   ├── App.tsx                       # Main app + routes
│   │   ├── main.tsx                      # Entry point
│   │   └── index.css                     # Global styles
│   ├── vite.config.ts
│   └── package.json
│
├── docker-compose.yml
└── README.md
```

---

## 2. Backend Development

### 2.1 Setting Up Development Environment

```bash
# Prerequisites
- Java 17+
- Maven 3.9+
- MySQL 8.0 (or Docker)

# Start MySQL with Docker
docker-compose up -d mysql

# Run backend in dev mode (uses H2 in-memory DB)
cd backend
./mvnw spring-boot:run -Dspring-boot.run.profiles=dev

# Run with MySQL
./mvnw spring-boot:run
```

### 2.2 Configuration

**application.yml structure:**
```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/roadtech
    username: ${DB_USERNAME:roadtech}
    password: ${DB_PASSWORD:roadtech123}

jwt:
  secret: ${JWT_SECRET:base64-encoded-secret-key}
  access-token-expiration: 900000    # 15 minutes
  refresh-token-expiration: 604800000 # 7 days
```

### 2.3 Adding New Entity

1. Create entity class in `entity/`:
```java
@Entity
@Table(name = "table_name")
@Getter @Setter
@NoArgsConstructor @AllArgsConstructor
@Builder
public class MyEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // fields...
}
```

2. Create repository in `repository/`:
```java
@Repository
public interface MyEntityRepository extends JpaRepository<MyEntity, Long> {
    // custom queries...
}
```

3. Create DTOs in `dto/`:
```java
@Data
@NoArgsConstructor @AllArgsConstructor
public class MyEntityDto {
    // fields...

    public static MyEntityDto fromEntity(MyEntity entity) {
        // mapping logic
    }
}
```

4. Create service in `service/`:
```java
@Service
@RequiredArgsConstructor
public class MyEntityService {
    private final MyEntityRepository repository;
    // business logic...
}
```

5. Create controller in `controller/`:
```java
@RestController
@RequestMapping("/my-entity")
@RequiredArgsConstructor
@Tag(name = "My Entity", description = "...")
public class MyEntityController {
    private final MyEntityService service;
    // endpoints...
}
```

### 2.4 Adding New Endpoint

```java
@RestController
@RequestMapping("/api/example")
@RequiredArgsConstructor
@Tag(name = "Example")
@SecurityRequirement(name = "bearerAuth")
public class ExampleController {

    @GetMapping("/{id}")
    @Operation(summary = "Get example by ID")
    public ResponseEntity<ExampleDto> getById(
            @AuthenticationPrincipal CustomUserDetails userDetails,
            @PathVariable Long id
    ) {
        // Implementation
    }

    @PostMapping
    @Operation(summary = "Create example")
    public ResponseEntity<ExampleDto> create(
            @AuthenticationPrincipal CustomUserDetails userDetails,
            @Valid @RequestBody CreateExampleDto dto
    ) {
        // Implementation
    }
}
```

### 2.5 Custom Exceptions

```java
// Throw in service layer
throw new ResourceNotFoundException("Entity", entityId);
throw new BadRequestException("Invalid input");
throw new UnauthorizedException("Access denied");
throw new ForbiddenException("Not allowed");

// Handled automatically by GlobalExceptionHandler
// Returns proper HTTP status codes and error messages
```

---

## 3. Frontend Development

### 3.1 Setting Up Development Environment

```bash
# Prerequisites
- Node.js 18+
- npm 9+

# Install dependencies
cd frontend
npm install

# Start development server
npm run dev

# Build for production
npm run build
```

### 3.2 Adding New Page

1. Create page component in `pages/`:
```tsx
// src/pages/example/ExamplePage.tsx
import { useQuery } from '@tanstack/react-query';
import { Card, CardHeader, CardTitle, CardContent } from '../../components/common';

export function ExamplePage() {
  const { data, isLoading } = useQuery({
    queryKey: ['example'],
    queryFn: exampleApi.getAll,
  });

  if (isLoading) return <div>Loading...</div>;

  return (
    <Card>
      <CardHeader>
        <CardTitle>Example</CardTitle>
      </CardHeader>
      <CardContent>
        {/* Content */}
      </CardContent>
    </Card>
  );
}
```

2. Add route in `App.tsx`:
```tsx
<Route
  path="example"
  element={
    <ProtectedRoute allowedRoles={['USER']}>
      <ExamplePage />
    </ProtectedRoute>
  }
/>
```

### 3.3 Adding New API Service

```typescript
// src/api/example.ts
import apiClient from './client';
import type { Example, CreateExampleDto } from '../types';

export const exampleApi = {
  getAll: async (): Promise<Example[]> => {
    const response = await apiClient.get<Example[]>('/example');
    return response.data;
  },

  getById: async (id: number): Promise<Example> => {
    const response = await apiClient.get<Example>(`/example/${id}`);
    return response.data;
  },

  create: async (data: CreateExampleDto): Promise<Example> => {
    const response = await apiClient.post<Example>('/example', data);
    return response.data;
  },
};
```

### 3.4 Using React Query

```tsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// Fetching data
const { data, isLoading, error } = useQuery({
  queryKey: ['examples', filters],
  queryFn: () => exampleApi.getAll(filters),
  staleTime: 60000, // 1 minute
});

// Mutations
const queryClient = useQueryClient();

const createMutation = useMutation({
  mutationFn: exampleApi.create,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['examples'] });
  },
});
```

### 3.5 Form Handling with React Hook Form + Zod

```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  name: z.string().min(2, 'Name is required'),
  email: z.string().email('Invalid email'),
});

type FormData = z.infer<typeof schema>;

function MyForm() {
  const { register, handleSubmit, formState: { errors } } = useForm<FormData>({
    resolver: zodResolver(schema),
  });

  const onSubmit = (data: FormData) => {
    // Handle submission
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <Input
        label="Name"
        error={errors.name?.message}
        {...register('name')}
      />
      <Button type="submit">Submit</Button>
    </form>
  );
}
```

### 3.6 Component Patterns

**Reusable Button:**
```tsx
<Button variant="primary" size="md" isLoading={loading}>
  Click Me
</Button>

// Variants: primary, secondary, outline, ghost, destructive
// Sizes: sm, md, lg
```

**Card Layout:**
```tsx
<Card>
  <CardHeader>
    <CardTitle>Title</CardTitle>
    <CardDescription>Description</CardDescription>
  </CardHeader>
  <CardContent>
    {/* Content */}
  </CardContent>
  <CardFooter>
    {/* Actions */}
  </CardFooter>
</Card>
```

---

## 4. Database Schema

### Tables

```sql
-- Users table
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(255) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    full_name VARCHAR(100) NOT NULL,
    phone VARCHAR(20) NOT NULL,
    role ENUM('USER', 'MECHANIC', 'ADMIN') NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_users_email (email),
    INDEX idx_users_role (role)
);

-- Mechanic profiles
CREATE TABLE mechanic_profiles (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL UNIQUE,
    specializations JSON,
    is_available BOOLEAN DEFAULT FALSE,
    current_latitude DECIMAL(10, 8),
    current_longitude DECIMAL(11, 8),
    rating DECIMAL(3, 2) DEFAULT 0.00,
    total_jobs INT DEFAULT 0,
    location_updated_at TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_mechanic_available (is_available)
);

-- Service requests
CREATE TABLE service_requests (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    mechanic_id BIGINT,
    issue_type VARCHAR(50) NOT NULL,
    description TEXT,
    latitude DECIMAL(10, 8) NOT NULL,
    longitude DECIMAL(11, 8) NOT NULL,
    address VARCHAR(500),
    status ENUM('PENDING', 'ACCEPTED', 'IN_PROGRESS', 'COMPLETED', 'CANCELLED'),
    estimated_arrival TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    accepted_at TIMESTAMP,
    started_at TIMESTAMP,
    completed_at TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (mechanic_id) REFERENCES users(id),
    INDEX idx_request_status (status)
);

-- Refresh tokens
CREATE TABLE refresh_tokens (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    token VARCHAR(255) NOT NULL UNIQUE,
    expires_at TIMESTAMP NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_refresh_token (token)
);
```

---

## 5. API Reference

### Authentication

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/api/auth/register` | POST | No | Register new user |
| `/api/auth/login` | POST | No | Login |
| `/api/auth/refresh` | POST | No | Refresh access token |
| `/api/auth/logout` | POST | No | Logout |

### User

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/api/users/me` | GET | Yes | Get current user |
| `/api/users/me` | PUT | Yes | Update current user |

### Service Requests

| Endpoint | Method | Auth | Roles | Description |
|----------|--------|------|-------|-------------|
| `/api/service-requests` | POST | Yes | USER | Create request |
| `/api/service-requests` | GET | Yes | USER | Get user's requests |
| `/api/service-requests/active` | GET | Yes | USER | Get active request |
| `/api/service-requests/{id}` | GET | Yes | USER/MECHANIC | Get request by ID |
| `/api/service-requests/{id}/cancel` | PUT | Yes | USER | Cancel request |

### Mechanic

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/api/mechanic/profile` | GET | Yes | Get profile |
| `/api/mechanic/profile` | PUT | Yes | Update profile |
| `/api/mechanic/availability` | PUT | Yes | Toggle availability |
| `/api/mechanic/location` | PUT | Yes | Update location |
| `/api/mechanic/requests/pending` | GET | Yes | Get pending requests |
| `/api/mechanic/requests` | GET | Yes | Get assigned requests |
| `/api/mechanic/requests/active` | GET | Yes | Get active requests |
| `/api/mechanic/requests/{id}/accept` | PUT | Yes | Accept request |
| `/api/mechanic/requests/{id}/reject` | PUT | Yes | Reject request |
| `/api/mechanic/requests/{id}/start` | PUT | Yes | Start service |
| `/api/mechanic/requests/{id}/complete` | PUT | Yes | Complete service |

---

## 6. WebSocket Integration

### Backend Configuration

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.enableSimpleBroker("/topic", "/queue");
        config.setApplicationDestinationPrefixes("/app");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws")
                .setAllowedOrigins("http://localhost:5173")
                .withSockJS();
    }
}
```

### Frontend Connection

```typescript
import { Client } from '@stomp/stompjs';
import SockJS from 'sockjs-client';

const client = new Client({
  webSocketFactory: () => new SockJS('/api/ws'),
  connectHeaders: {
    Authorization: `Bearer ${token}`,
  },
});

// Subscribe to topic
client.subscribe('/topic/request/123', (message) => {
  const data = JSON.parse(message.body);
  // Handle message
});

// Send message
client.publish({
  destination: '/app/location',
  body: JSON.stringify({ latitude, longitude }),
});
```

---

## 7. Security Implementation

### JWT Token Structure

```
Header: { "alg": "HS256", "typ": "JWT" }
Payload: { "sub": "user@email.com", "iat": 1234567890, "exp": 1234568790 }
Signature: HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```

### Security Filter Chain

```
Request → JwtAuthFilter → SecurityContext → Controller
                ↓
         Extract JWT from header
                ↓
         Validate token
                ↓
         Load UserDetails
                ↓
         Set Authentication
```

### Role-Based Access Control

```java
// In SecurityConfig
.requestMatchers("/mechanic/**").hasRole("MECHANIC")
.requestMatchers("/admin/**").hasRole("ADMIN")
.anyRequest().authenticated()

// In Controller (method level)
@PreAuthorize("hasRole('ADMIN')")
public ResponseEntity<?> adminOnly() { ... }
```

---

## 8. Testing Guide

### Backend Tests

```bash
# Run all tests
./mvnw test

# Run specific test class
./mvnw test -Dtest=AuthServiceTest

# Run with coverage
./mvnw test jacoco:report
```

### Frontend Tests

```bash
# Type checking
npm run build

# Lint
npm run lint
```

---

## 9. Deployment

### Docker Deployment

```bash
# Build all images
docker-compose build

# Start all services
docker-compose up -d

# View logs
docker-compose logs -f backend

# Stop all services
docker-compose down
```

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `JWT_SECRET` | (required) | JWT signing key (min 256 bits) |
| `SPRING_DATASOURCE_URL` | jdbc:mysql://... | Database URL |
| `SPRING_DATASOURCE_USERNAME` | roadtech | DB username |
| `SPRING_DATASOURCE_PASSWORD` | roadtech123 | DB password |

---

## 10. Troubleshooting

### Common Issues

**CORS errors:**
- Check `SecurityConfig.java` CORS configuration
- Ensure frontend URL is in allowed origins

**JWT token expired:**
- Frontend should automatically refresh using refresh token
- Check `client.ts` interceptor logic

**WebSocket connection fails:**
- Verify `/ws` endpoint is accessible
- Check authentication headers
- Ensure SockJS fallback is working

**Database connection issues:**
- Verify MySQL is running: `docker-compose ps`
- Check connection string in application.yml
- Ensure database exists: `CREATE DATABASE roadtech;`

### Logging

```yaml
# application.yml
logging:
  level:
    com.roadtech: DEBUG
    org.springframework.security: DEBUG
    org.hibernate.SQL: DEBUG
```

### Debug Mode

```bash
# Backend with debug port
./mvnw spring-boot:run -Dspring-boot.run.jvmArguments="-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005"

# Frontend with verbose output
npm run dev -- --debug
```
