# Trainee Management System

A Dockerized ASP.NET Core microservices-based trainee management system.

The solution contains:

- **TaskManagement API** - main API for users, mentors, trainees, learning tasks, task assignments, submissions, reviews, file uploads, processing jobs and authentication.
- **TrainingDirectory API** - API that communicates with TaskManagement API using HttpClient, retry and circuit breaker policies.
- **SubmissionProcessor Worker** - background worker that consumes RabbitMQ messages and processes submitted files.
- **TaskManagement.Shared** - shared class library containing common domain models, enums, AppDbContext, messaging contracts, file storage services and shared options.
- **MySQL** - primary relational database.
- **Redis** - caching service.
- **RabbitMQ** - message broker for asynchronous submission processing.

---

## Technology Used

- ASP.NET Core Web API
- .NET 10
- Entity Framework Core
- Pomelo EntityFrameworkCore MySql
- MySQL
- Redis
- RabbitMQ
- Docker
- Docker Compose
- Swagger UI
- JWT Authentication
- BCrypt password hashing
- Polly retry and circuit breaker
- Background Worker Service
- Shared Docker volume for file storage

---

## Project Structure
```
.
├── TaskManagement.Shared
│   ├── Data
│   │   └── AppDbContext.cs
│   ├── Migrations 
│   ├── Models
│   │   ├── Entities
│   │   ├── Enums
│   │   └── Messaging
│   ├── Options
│   ├── Services
│   └── TaskManagement.Shared.csproj
│
├── taskmanagement
│   ├── Controllers
│   ├── Middlewares
│   ├── Models
│   │   ├── DTOs
│   ├── Services
│   ├── Validators
│   ├── Dockerfile
│   ├── Program.cs
│   ├── appsettings.json
│   └── taskmanagement.csproj
│
├── SubmissionProcessor
│   ├── Services
│   ├── Model
│   ├── Worker.cs
│   ├── Program.cs
│   ├── Dockerfile
│   ├── appsettings.json
│   └── SubmissionProcessor.csproj
│
├── TrainingDirectory.Api
│   ├── Clients
│   ├── Controllers
│   ├── Models
│   ├── Dockerfile
│   ├── Program.cs
│   ├── appsettings.json
│   └── TrainingDirectory.Api.csproj
│
├── docker-compose.yml
├── .dockerignore
└── TraineeManagementSln.slnx
```

## Configuration guide

1. Clone the repository
```bash
git clone https://github.com/smit2870/TrainingTasks.git  

```
2. Create a .env file in the root directory and set following values : 
```
DB_HOST=<HostName>>
DB_PORT=3306
DB_NAME=<DBNAME>
DB_USER=<DBUSER>
DB_PASSWORD=<DBPASSWORD>

REDIS_IMAGE=<REDISIMAGE>
MYSQL_IMAGE=<MYSQLIMAGE>
RABBITMQ_IMAGE=<RABBITMQIMAGE>

JWT_KEY=<Secret-key>

DOMAIN_OWNER=<your-domain-owner>
REGION=<your-region>
NUGET_REPO_NAME=<your-nuget-repository-name>
CODEARTIFACT_AUTH_TOKEN=<your-codeartifact-token>
```
3. Build all services

```bash
docker compose build
```

4. Start all services
```bash
docker compose up -d
```

5. To Stop All Services
```bash
docker compose down
```

--- 

## Services and Ports

| Service | Container Name | URL |
|---|---|---|
| TaskManagement API | `taskmanagement-api` | `http://localhost:5153` |
| TaskManagement Swagger | `taskmanagement-api` | `http://localhost:5153/swagger/index.html` |
| TrainingDirectory API | `trainingdirectory-api` | `http://localhost:5200` |
| TrainingDirectory Swagger | `trainingdirectory-api` | `http://localhost:5200/swagger/index.html` |
| RabbitMQ Management UI | `rabbitmq` | `http://localhost:15672` |
| MySQL | `mysql-db` | `localhost:3306` |
| Redis | `redis` | `localhost:6379` |

RabbitMQ login:
```
Username: rmquser
Password: rmqpass
```
---

---

## Database Setup

The database is created by the MySQL container using:
```
MYSQL_DATABASE: ${DB_NAME}
```

Check database tables:
```bash
docker exec -it mysql-db mysql -u <DBUSER> -p'<DBPASSWORD>' <DB_NAME> -e "SHOW TABLES;"
```

---

## RabbitMQ Submission Processing Flow

Queue name:
```
submission-processing
```

Flow:

- User uploads file using TaskManagement API.
- API saves file to /App/storage.
- API inserts metadata into SubmissionFiles.
- API creates a row in ProcessingJobs.
- API publishes a message to RabbitMQ.
- Worker receives the message.
- Worker reads the file from /App/storage.
- Worker updates job status, attempts, result, and errors.

Check RabbitMQ queues:
```bash
docker exec -it rabbitmq rabbitmqctl list_queues name messages messages_ready messages_unacknowledged consumers
```
--- 

## API List

- Health
```http
GET    /api/health /live
GET    /api/health /ready
```

- Auth 
```http
POST   /api/auth/login 
```

- Trainee 
```http
GET    /api/trainees?pageNumber=1&pageSize=10&search=amit&status=Active 
GET    /api/trainees/{id} 
POST   /api/trainees 
PUT    /api/trainees/{id} 
DELETE /api/trainees/{id} 
```

- Mentor
```http
GET    /api/mentors 
GET    /api/mentors/{id} 
POST   /api/mentors 
PUT    /api/mentors/{id} 
DELETE /api/mentors/{id} 
```

- Learning Tasks
```http
GET    /api/learning-tasks 
GET    /api/learning-tasks/{id} 
POST   /api/learning-tasks 
PUT    /api/learning-tasks/{id} 
DELETE /api/learning-tasks/{id} 
```

- Processing Jobs
```http 
GET    /api/processing-jobs/{id} 
GET    /api/processing-jobs/{status}
```

- Task Assignments 
```http
POST   /api/task-assignments 
GET    /api/task-assignments 
GET    /api/task-assignments/{id} 
PUT    /api/task-assignments/{id}/status 
```

- Submissions 
```http
POST   /api/submissions 
GET    /api/submissions 
GET    /api/submissions/{id} 
```

- Reviews 
```http
POST   /api/reviews 
GET    /api/reviews 
GET    /api/reviews/{id}
```

- User
```http
GET    /api/users
GET    /api/users/{id} 
POST   /api/users 
PUT    /api/users/{id} 
DELETE /api/users/{id} 
```

## Swagger URLS

TaskManagement Swagger:
```http
http://localhost:5153/swagger/index.html
```

TrainingDirectory Swagger:
```http
http://localhost:5200/swagger/index.html
```

## API Sample 

### 1. Get All Trainees by Search (GET /api/trainee?search=\<string\>)
- Sample Request JSON:
```http
GET /api/trainee
```
- Sample Response JSON:

```json
[
  {
    "id": 1,
    "firstName": "Zeus",
    "lastName": "Learning",
    "email": "zeuslearning@email.com",
    "techStack": ["C#", "Dotnet"],
    "status": "Active",
    "createdAt": "2026-06-08T10:30:00Z",
    "updatedAt": "2026-06-08T10:30:00Z"
  }
]
```

### 2. Get Trainee By Id (GET /api/trainee/\<id\>)
- Sample Request JSON: 
```http
GET /api/trainee/1
```
- Sample Reponse JSON:
```json
[
  {
    "id": 1,
    "firstName": "Zeus",
    "lastName": "Learning",
    "email": "zeuslearning@email.com",
    "techStack": ["C#", "Dotnet"],
    "status": "Active",
    "createdAt": "2026-06-08T10:30:00Z",
    "updatedAt": "2026-06-08T10:30:00Z"
  }
]
```

### 3. Create Trainee (POST /api/trainee)
- Sample Request JSON:
```http
POST /api/trainee
Content-Type: application/json
```
```json
{
  "firstName": "Amit",
  "lastName": "Sharma",
  "email": "amit@email.com",
  "techStack": ["React", "NodeJS"],
  "status": "Busy"
}
```
- Sample Response JSON:
```json
{
  "id": 2,
  "firstName": "Amit",
  "lastName": "Sharma",
  "email": "amit@email.com",
  "techStack": ["React", "NodeJS"],
  "status": "Busy",
  "createdAt": "2026-06-08T11:00:00Z",
  "updatedAt": "2026-06-08T11:00:00Z"
}
```

### 4. Update Trainee (PUT /api/trainee/\<id\>)
- Sample Request JSON:
```http
PUT /api/trainee/2
Content-Type: application/json
```

```json
{
  "firstName": "Amit",
  "lastName": "Patel",
  "email": "amitpatel@email.com",
  "techStack": ["Angular", ".NET"],
  "status": "Active"
}
```
- Sample Response JSON:
```json
{
  "firstName": "Amit",
  "lastName": "Patel",
  "email": "amitpatel@email.com",
  "techStack": ["Angular", ".NET"],
  "status": "Active"
}
```

### 5. Delete Trainee (DELETE /api/trainee/\<id\>)
- Sample Request JSON:
```http
DELETE /api/trainee/2
```
- Sample Response JSON:
```http
204 No Content
```

---

## Known Limitations

1. Uploaded files are stored in a Docker volume. 
2. Failed messages are retried, but a production-grade dead-letter exchange and retry queue strategy can be added.


## Next Improvement Areas

- Add automatic database migration strategy.
- Add better retry strategy for file processing.
- Replace local file storage with cloud/object storage.
- Add integration tests for API, database, RabbitMQ, and worker.