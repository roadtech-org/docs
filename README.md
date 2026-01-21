# RoadTech - Roadside Assistance System

A production-ready web application that connects users in need of roadside assistance with nearby mechanics. Built with React + Vite + Tailwind CSS (Frontend) and Spring Boot (Backend).

## Features

- **User Features**
  - Register/Login with JWT authentication
  - Request roadside assistance with live GPS location
  - Real-time tracking of assigned mechanic
  - View request history

- **Mechanic Features**
  - Go online/offline to receive requests
  - Accept/reject incoming assistance requests
  - Navigate to customer location
  - Mark jobs as started/completed

- **Technical Features**
  - Real-time updates via WebSocket (STOMP)
  - JWT authentication with refresh tokens
  - Interactive maps with Leaflet
  - Responsive design with Tailwind CSS
  - API documentation with Swagger/OpenAPI
  - Docker containerization

## Tech Stack

### Frontend
- React 18 with TypeScript
- Vite for build tooling
- Tailwind CSS for styling
- React Router v6 for routing
- TanStack Query for server state
- React Hook Form + Zod for forms
- Leaflet for maps
- STOMP.js for WebSocket

### Backend
- Java 17
- Spring Boot 3.2
- Spring Security with JWT
- Spring Data JPA
- Spring WebSocket
- MySQL 8
- Springdoc OpenAPI (Swagger)

## Project Structure

```
roadTech/
├── frontend/                # React application
│   ├── src/
│   │   ├── api/            # API client
│   │   ├── components/     # Reusable components
│   │   ├── contexts/       # React contexts
│   │   ├── hooks/          # Custom hooks
│   │   ├── pages/          # Route pages
│   │   ├── types/          # TypeScript types
│   │   └── utils/          # Utility functions
│   └── Dockerfile
│
├── backend/                 # Spring Boot application
│   ├── src/main/java/com/roadtech/
│   │   ├── config/         # Configuration
│   │   ├── controller/     # REST controllers
│   │   ├── dto/            # Data transfer objects
│   │   ├── entity/         # JPA entities
│   │   ├── exception/      # Exception handling
│   │   ├── repository/     # Data repositories
│   │   ├── security/       # Security config
│   │   ├── service/        # Business logic
│   │   └── websocket/      # WebSocket handlers
│   └── Dockerfile
│
└── docker-compose.yml
```

## Getting Started

### Prerequisites

- Node.js 18+
- Java 17+
- Maven 3.9+
- Docker & Docker Compose (optional)
- MySQL 8 (or use Docker)

### Development Setup

#### 1. Start MySQL Database

Using Docker:
```bash
docker-compose up -d mysql
```

Or install MySQL locally and create the database:
```sql
CREATE DATABASE roadtech;
CREATE USER 'roadtech'@'localhost' IDENTIFIED BY 'roadtech123';
GRANT ALL PRIVILEGES ON roadtech.* TO 'roadtech'@'localhost';
```

#### 2. Start Backend

```bash
cd backend

# Using Maven wrapper (recommended)
./mvnw spring-boot:run -Dspring-boot.run.profiles=dev

# Or with Maven installed
mvn spring-boot:run -Dspring-boot.run.profiles=dev
```

The backend will start at `http://localhost:8080`

#### 3. Start Frontend

```bash
cd frontend
npm install
npm run dev
```

The frontend will start at `http://localhost:5173`

### Docker Deployment

Build and run all services:

```bash
docker-compose up --build
```

Services:
- Frontend: http://localhost:80
- Backend API: http://localhost:8080/api
- Swagger UI: http://localhost:8080/api/swagger-ui.html
- MySQL: localhost:3306

## API Documentation

Once the backend is running, access the Swagger UI at:
- Development: http://localhost:8080/api/swagger-ui.html
- Docker: http://localhost:8080/api/swagger-ui.html

### Main Endpoints

#### Authentication
- `POST /api/auth/register` - Register new user
- `POST /api/auth/login` - Login
- `POST /api/auth/refresh` - Refresh token
- `POST /api/auth/logout` - Logout

#### Service Requests (User)
- `POST /api/service-requests` - Create request
- `GET /api/service-requests` - Get user's requests
- `GET /api/service-requests/active` - Get active request
- `PUT /api/service-requests/{id}/cancel` - Cancel request

#### Mechanic
- `GET /api/mechanic/profile` - Get profile
- `PUT /api/mechanic/availability` - Toggle availability
- `PUT /api/mechanic/location` - Update location
- `GET /api/mechanic/requests/pending` - Get pending requests
- `PUT /api/mechanic/requests/{id}/accept` - Accept request
- `PUT /api/mechanic/requests/{id}/start` - Start service
- `PUT /api/mechanic/requests/{id}/complete` - Complete service

## Environment Variables

### Backend
```yaml
JWT_SECRET: your-super-secret-key-minimum-256-bits
SPRING_DATASOURCE_URL: jdbc:mysql://localhost:3306/roadtech
SPRING_DATASOURCE_USERNAME: roadtech
SPRING_DATASOURCE_PASSWORD: roadtech123
```

### Frontend (via Vite proxy in development)
No additional environment variables needed for development.

## Testing

### Backend
```bash
cd backend
./mvnw test
```

### Frontend
```bash
cd frontend
npm run build  # Type checking + build
```

## Production Deployment

1. Update environment variables with secure values
2. Build Docker images:
   ```bash
   docker-compose build
   ```
3. Deploy using your preferred orchestration tool

### Security Considerations

- Change default JWT secret
- Use HTTPS in production
- Configure CORS for specific domains
- Secure MySQL credentials
- Enable rate limiting

## License

MIT License
