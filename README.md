# Airbnb Real-Time AI Trip Planning & Collaboration Platform – Backend

> **Production-shaped prototype of a real-time, multiplayer trip-planning canvas** — where itineraries are live, concurrently-edited shared documents with optimistic concurrency control, group budget-split voting, and an explainable AI Concierge grounded in concrete venue catalog data, budget caps, and conflict-free scheduling.

---

## 🌟 Highlights & Core Capabilities

- **Live Multiplayer Itinerary Canvas**: Real-time concurrent document editing backed by WebSocket (STOMP) and Spring Kafka event pipelines. Collaborative drag/move and pin activity cards across Day 1, Day 2, and Unassigned Ideas backlog.
- **Optimistic Concurrency Control**: Entity-level version tracking (`@Version Long version`) on `ActivityCard` prevents lost updates during concurrent edits.
- **Trip Rooms & Instant Invite Links**: Create collaborative trip rooms with destination, date range, total budget cap, and unique human-readable invite codes (e.g. `TOKYO2026`).
- **Group Budget-Split Voting System**: Group travel involves financial decisions. Propose an activity card with cost splits (Equal or By Person). Participants vote (YES/NO/ABSTAIN). The system tallies votes in real time, reaches consensus resolution, automatically transitions cards to `PINNED` (or `REJECTED`), and deducts spend from the live trip budget cap.
- **Grounded & Explainable AI Concierge**: Solves AI hallucination by enforcing concrete constraint verification:
  1. **Venue Catalog Grounding**: Filters real database POIs (rating, price per person, coordinates, address, tags).
  2. **Budget Verification**: Validates cost against the group's remaining trip budget.
  3. **Schedule Conflict Checking**: Inspects existing pinned activities on the target day to ensure conflict-free time slots.
  4. **Operating Hours Verification**: Ensures venue is open during the requested slot.
  5. **Explainability Breakdown**: Produces transparent rationale for every recommendation and yields an actionable 1-click proposal.
- **Dual Session Testing Workbench**: A modern split-screen test client served directly by Spring Boot at `http://localhost:8080/`, letting you test Host (Alice) and Collaborator (Bob) side-by-side with live bi-directional sync.
- **Comprehensive Test Suite**: Automated JUnit 5 integration tests (`mvn test`) verifying end-to-end user journeys from auth to room creation, card moves, quorum resolution, and AI grounding.

---

## 🏗 Tech Stack & Architecture

| Layer | Technology |
|---|---|
| **Language & Platform** | Java 17 LTS |
| **Framework** | Spring Boot 3.3.4 (Spring MVC, Spring WebSocket, Spring Data JPA, Spring Security) |
| **Security & Auth** | Spring Security 6 + JJWT 0.12.5 (Stateless Bearer Tokens, STOMP Channel Interceptor) |
| **ORM & Persistence** | Hibernate 6 / JPA / HikariCP |
| **Databases** | **MySQL 8.0** (Production DDL included) + H2 (In-Memory zero-setup Dev profile) |
| **Event Streaming** | **Spring Kafka** (`KafkaTemplate`, `@KafkaListener`, Topics: `trip.activity.events`, `trip.voting.events`, `trip.ai.audit`) |
| **Real-Time Sync** | STOMP over WebSocket with SockJS fallback (`/ws-trip`, `/topic/trip/{tripId}/...`) |
| **Build & Tooling** | Apache Maven 3.9+ |

---

## 📂 Project Architecture

```
airbnb-trip-planner/
├── pom.xml
├── README.md
├── src/main/resources/
│   ├── application.yml                 # Default dev configuration (H2 in MySQL mode, Decoupled event bus)
│   ├── application-mysql.yml           # Production configuration for native MySQL 8.0
│   ├── schema-mysql.sql                # Production MySQL DDL script
│   └── static/
│       └── index.html                  # Interactive Dual-Session Real-Time Workbench UI
└── src/main/java/com/airbnb/tripplanner/
    ├── AirbnbTripPlannerApplication.java
    ├── config/
    │   └── DataInitializer.java        # Pre-seeds demo users (Alice & Bob), realistic Tokyo venues, and seed trip room
    ├── controller/
    │   ├── AuthController.java          # POST /api/auth/login, /register, /me
    │   ├── TripRoomController.java      # POST /api/trips, /join, GET /{id}, /audit-logs
    │   ├── ActivityCardController.java  # GET /activities, POST, PUT /move, DELETE
    │   ├── VotingController.java        # POST /activities/{id}/vote
    │   ├── VenueController.java         # GET /api/venues (search by city, category, budget)
    │   └── AiConciergeController.java   # POST /api/concierge/ask (grounded & explainable)
    ├── dto/                            # Data Transfer Objects & Payloads
    ├── entity/                         # JPA Entities (User, TripRoom, Participant, Venue, ActivityCard, Vote, BudgetSplit, AuditLog)
    ├── kafka/
    │   ├── KafkaTopicConfig.java       # Topic definitions (trip.activity.events, trip.voting.events, trip.ai.audit)
    │   ├── TripEventProducer.java      # KafkaTemplate producer with resilient fallback
    │   └── TripEventConsumer.java      # @KafkaListener consumer & audit log persistence
    ├── repository/                     # Spring Data JPA Repositories
    ├── security/                       # SecurityConfig, JwtUtils, AuthTokenFilter, UserDetailsServiceImpl
    ├── service/                        # Core Domain Logic (TripRoomService, ActivityCardService, VotingService, AiConciergeService)
    └── websocket/                      # WebSocketConfig, RealtimeBroadcastService, CanvasMessageController
```

---

## 🚀 Quickstart Guide

### 1. Prerequisites
- **Java 17** (`java -version`)
- **Maven 3.8+** (`mvn -v`)

### 2. Run the Application Out-of-the-Box (Default Profile)
```bash
cd C:\Users\deepa\.gemini\antigravity\scratch\airbnb-trip-planner
mvn spring-boot:run
```
*The default profile starts with an in-memory MySQL-compatible database and pre-seeds:*
- **Host User**: `alice` / `password123`
- **Collaborator User**: `bob` / `password123`
- **Pre-seeded Trip Room**: `Tokyo Cyberpunk & Culinary Adventure` (Invite Code: `TOKYO2026`)
- **10+ Curated Real-World Tokyo Venues** (Shibuya Sky, Ichiran Ramen, teamLab Planets, Senso-ji, Tsukiji, Sushi Yoshitake, etc.)

### 3. Open the Interactive Real-Time Workbench
Open your browser to:
👉 **[http://localhost:8080/](http://localhost:8080/)**

The workbench opens in **Dual Session Mode**:
- **Left Window**: Session 1 (Alice Chen - Host)
- **Right Window**: Session 2 (Bob Martinez - Collaborator)
- Watch live bi-directional sync as you move cards, cast budget votes, and query the AI Concierge!

---

## 🗄 Running with Native MySQL 8.0

To run against a local or remote MySQL 8.0 database:

1. **Create the Database** (or execute `src/main/resources/schema-mysql.sql`):
   ```sql
   CREATE DATABASE airbnb_trip_planner CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   ```
2. **Start Spring Boot with MySQL profile**:
   ```bash
   mvn spring-boot:run -Dspring-boot.run.profiles=mysql -Dspring-boot.run.arguments="--SPRING_DATASOURCE_URL=jdbc:mysql://localhost:3306/airbnb_trip_planner?createDatabaseIfNotExist=true&useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC --SPRING_DATASOURCE_USERNAME=root --SPRING_DATASOURCE_PASSWORD=your_password"
   ```

---

## ⚡ Running with Apache Kafka

To stream live events through an external Kafka broker (e.g. `localhost:9092`):
```bash
mvn spring-boot:run -Dspring-boot.run.arguments="--app.kafka.enabled=true --KAFKA_BOOTSTRAP_SERVERS=localhost:9092"
```
The application will automatically initialize partitions for:
- `trip.activity.events`
- `trip.voting.events`
- `trip.ai.audit`

---

## 🧪 Automated Testing

Execute the complete automated test suite covering authentication, multiplayer canvas synchronization, optimistic locking, budget-split voting resolution, and AI Concierge constraint grounding:

```bash
mvn test
```

Sample output:
```
[INFO] Running com.airbnb.tripplanner.AuthControllerTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 16.34 s
[INFO] Running com.airbnb.tripplanner.TripCollaborationIntegrationTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.620 s
[INFO] Running com.airbnb.tripplanner.VotingAndBudgetTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.263 s
[INFO] Results:
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0
[INFO] BUILD SUCCESS
```

---

## 📡 REST API Reference

### 1. Authentication (`/api/auth`)
| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/auth/login` | Authenticate with username & password, returns JWT token |
| `POST` | `/api/auth/register` | Register new user account |
| `GET` | `/api/auth/me` | Fetch authenticated user profile |

### 2. Collaborative Trip Rooms (`/api/trips`)
| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/trips` | Create new Trip Room with budget cap & currency |
| `POST` | `/api/trips/join` | Join existing trip room via unique invite code |
| `GET` | `/api/trips/{id}` | Get trip room metadata, budget summary, and participants |
| `GET` | `/api/trips` | List all trips the current user participates in |
| `GET` | `/api/trips/{id}/audit-logs` | Retrieve chronological audit trail of collaborative actions |

### 3. Itinerary Canvas Activities (`/api/trips/{id}/activities`)
| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/trips/{id}/activities` | Get all activity cards organized by Day and Sort Order |
| `POST` | `/api/trips/{id}/activities` | Pin or propose new activity card (triggers STOMP & Kafka) |
| `PUT` | `/api/trips/{id}/activities/{cardId}/move` | Move activity card across days/time slots with version check |
| `DELETE` | `/api/trips/{id}/activities/{cardId}` | Delete activity card and refund budget if previously approved |

### 4. Group Budget-Split Voting (`/api/trips/{id}/activities/{cardId}/vote`)
| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/trips/{id}/activities/{cardId}/vote` | Cast vote (`YES`, `NO`, `ABSTAIN`) with optional comment. Triggers quorum resolution. |

### 5. Grounded AI Concierge (`/api/concierge/ask`)
| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/concierge/ask` | Submit constrained travel query, returns grounded recommendation with full explainability breakdown. |

---

## 🤖 Grounded AI Concierge Explanation Matrix

When a user submits a query such as:
> *"Find a dinner spot near Shibuya for under $40/person on Day 1 evening between 19:00 and 21:00 that doesn't exceed our remaining budget"*

The AI Concierge validates 4 hard constraints against the live database:

```json
{
  "query": "Find a dinner spot near Shibuya for under $40/person on Day 1 evening between 19:00 and 21:00",
  "summaryAnswer": "I recommend Ichiran Ramen Shibuya (DINING) in Tokyo! At USD 14.00 per person (USD 28.00 total for 2 travelers), it fits neatly within your remaining budget of USD 2,462.00 without exceeding caps. It is open from 10:00 to 23:00, and your proposed slot of 19:00 - 20:30 on Day 1 has zero schedule conflicts with your current itinerary.",
  "recommendedVenue": {
    "name": "Ichiran Ramen Shibuya",
    "category": "DINING",
    "pricePerPerson": 14.00,
    "rating": 4.7,
    "openingTime": "10:00:00",
    "closingTime": "23:00:00",
    "address": "1-22-7 Jinnan, Shibuya City, Tokyo"
  },
  "reasoningBreakdown": {
    "budgetCheck": "PASS: USD 14.00/person (USD 28.00 total for 2 travelers). Fits within USD 2,462.00 remaining trip budget.",
    "scheduleConflictCheck": "PASS: Slot 19:00 - 20:30 on Day 1 is completely conflict-free. No overlapping activities.",
    "operatingHoursCheck": "PASS: Ichiran Ramen Shibuya is open from 10:00 to 23:00. Selected slot (19:00 - 20:30) is fully within operational hours.",
    "transitProximityCheck": "PASS: Located at 1-22-7 Jinnan, Shibuya City, Tokyo. Convenient transit access from prior activities in Tokyo.",
    "overallGroundedStatus": "GROUNDED_AND_VERIFIED"
  },
  "actionableProposal": {
    "venueId": 2,
    "title": "Ichiran Ramen Shibuya",
    "dayNumber": 1,
    "startTime": "19:00:00",
    "endTime": "20:30:00",
    "estimatedCostPerPerson": 14.00,
    "estimatedCostTotal": 28.00,
    "splitType": "EQUAL"
  }
}
```

---

## 🔒 Security & Concurrency Design

- **Stateless Authentication**: Passwords hashed with **BCrypt**. Requests authenticated via cryptographically signed JWT tokens with 24-hour validity.
- **WebSocket Security**: Handshake channel interceptor extracts the Bearer token from the STOMP `CONNECT` frame and authenticates the STOMP user session.
- **Optimistic Concurrency Control**:
  ```java
  @Version
  @Column(nullable = false)
  private Long version;
  ```
  Every time a collaborator moves or updates an activity card, the version is incremented. The authoritative server state is immediately broadcast to all clients via `/topic/trip/{tripId}/canvas`.
#   A i r b n b  
 