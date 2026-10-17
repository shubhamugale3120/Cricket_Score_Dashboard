# 🏏 CricPulse — Real-Time Cricket Score Management & Simulation Platform

A scalable, real-time cricket score management system built with **Spring Boot 3**, **Spring WebSocket (STOMP)**, **Spring Data JPA**, and a modern **responsive stadium dark-neon frontend dashboard**.

---

## 📋 Table of Contents
1. [Prerequisites & What to Install](#-prerequisites--what-to-install)
2. [Project Requirements & Capabilities](#-project-requirements--capabilities)
3. [Folder Structure](#-folder-structure)
4. [Architecture Overview](#-architecture-overview)
5. [How to Run the Application](#-how-to-run-the-application)
6. [API Endpoints (REST & OpenAPI)](#-api-endpoints-rest--openapi)
7. [WebSocket Real-Time Subscriptions](#-websocket-real-time-subscriptions)
8. [Cricket Simulation Engine Details](#-cricket-simulation-engine-details)

---

## 🛠️ Prerequisites & What to Install

Before running the application, ensure the following software is installed on your system:

| Tool | Recommended Version | Verification Command | Download / Install Link |
| :--- | :--- | :--- | :--- |
| **Java Development Kit (JDK)** | **JDK 21 LTS** (or JDK 17+) | `java -version` | [Eclipse Adoptium Temurin 21](https://adoptium.net/) |
| **Apache Maven** | **3.9.x+** | `mvn -v` | [Maven Official Download](https://maven.apache.org/download.cgi) |
| **Modern Web Browser** | Chrome, Edge, Firefox, or Safari | N/A | Any current modern browser |
| *(Optional)* **Node.js** | 20+ (if developing external frontend) | `node -v` | [Node.js Official](https://nodejs.org/) |

> **Note:** The application uses an embedded **H2 in-memory database** and bundles all frontend assets directly in Spring Boot (`src/main/resources/static`). **No external database (like MySQL or PostgreSQL) or Node server is required to run the full application!**

---

## 🎯 Project Requirements & Capabilities

1. **Live Match Management**:
   - Create and track matches across **T20**, **ODI**, and **Test** formats.
   - Lifecycle tracking: `UPCOMING`, `LIVE`, `INNINGS_BREAK`, and `COMPLETED`.
   - Realistic team rosters (e.g. India, Australia, England, South Africa) with player profiles and career statistics.

2. **Real-Time Score Processing & Simulation**:
   - Probabilistic cricket simulation engine generating realistic ball-by-ball events (dots, singles, doubles, boundaries, sixes, wickets, extras).
   - Dynamic strike rotation on odd runs (1, 3) and at the end of each over.
   - Automatic bowler rotation from the fielding team at over completions.
   - Fall of wicket handling (retrieving the next batsman in batting order).
   - Inning transitions (calculating targets, required run rates, and final match results).

3. **Real-Time WebSocket Streaming (STOMP)**:
   - Low-latency real-time score dissemination via `/topic/match/{id}` and `/topic/live-matches`.
   - Live dashboard updates with zero page reloads.

4. **Dynamic Analytics & Win Probability**:
   - Real-time Current Run Rate (CRR) and Required Run Rate (RRR).
   - Dynamic **Win Probability Meter** calculating chasing dynamics based on wickets in hand, balls remaining, and target deficit.

5. **Manual Scorekeeper Controls & Simulation Deck**:
   - Controls to Pause, Resume, or Step through balls one-by-one.
   - Manual override pad to test custom balls (0, 1, 2, 3, 4, 6, Wicket, Wide, No-ball) with custom commentary.
   - Reset match to 0.0 overs for testing.

---

## 📂 Folder Structure

```
Live_Score_Cricket/
├── pom.xml                                     # Maven build definition and dependencies
├── README.md                                   # Comprehensive documentation & setup guide
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/cricket/livescore/
│   │   │       ├── LiveScoreCricketApplication.java   # Spring Boot Main Entry Point
│   │   │       │
│   │   │       ├── config/                            # Configurations
│   │   │       │   ├── WebSocketConfig.java           # STOMP Broker & SockJS endpoints
│   │   │       │   ├── OpenApiConfig.java             # Swagger 3 UI / OpenAPI setup
│   │   │       │   └── CorsConfig.java                # Global CORS filter
│   │   │       │
│   │   │       ├── controller/                        # REST Controllers
│   │   │       │   ├── MatchController.java           # Match lifecycle & scorecard endpoints
│   │   │       │   ├── SimulationController.java      # Engine controls & manual scoring
│   │   │       │   └── PlayerController.java          # Teams & player statistics
│   │   │       │
│   │   │       ├── model/                             # JPA Entities & Enums
│   │   │       │   ├── Match.java                     # Master match state & live scores
│   │   │       │   ├── Team.java                      # Team profile & squad mappings
│   │   │       │   ├── Player.java                    # Player bio, role, career stats
│   │   │       │   ├── PlayerScorecard.java           # Individual batting & bowling stats
│   │   │       │   ├── BallEvent.java                 # Granular ball-by-ball record
│   │   │       │   ├── Commentary.java                # Live commentary log entries
│   │   │       │   ├── MatchStatus.java               # UPCOMING, LIVE, INNINGS_BREAK, COMPLETED
│   │   │       │   ├── MatchFormat.java               # T20 (20 ov), ODI (50 ov), TEST (90 ov)
│   │   │       │   ├── PlayerRole.java                # BATSMAN, BOWLER, ALL_ROUNDER, WICKET_KEEPER
│   │   │       │   ├── BallOutcome.java               # DOT, SINGLE, FOUR, SIX, WICKET, WIDE, etc.
│   │   │       │   └── WicketType.java                # BOWLED, CAUGHT, LBW, RUN_OUT, STUMPED
│   │   │       │
│   │   │       ├── dto/                               # Data Transfer Objects
│   │   │       │   ├── MatchSummaryDto.java           # Lightweight card representation
│   │   │       │   ├── MatchDetailDto.java            # Comprehensive match payload for live view
│   │   │       │   ├── CreateMatchRequest.java        # Request schema for new matches
│   │   │       │   └── ManualBallRequest.java         # Payload for manual score entries
│   │   │       │
│   │   │       ├── repository/                        # Spring Data JPA Repositories
│   │   │       │   ├── MatchRepository.java
│   │   │       │   ├── TeamRepository.java
│   │   │       │   ├── PlayerRepository.java
│   │   │       │   ├── BallEventRepository.java
│   │   │       │   ├── CommentaryRepository.java
│   │   │       │   └── PlayerScorecardRepository.java
│   │   │       │
│   │   │       ├── service/                           # Business Logic & Core Engines
│   │   │       │   ├── MatchService.java              # State updates, overs, wickets, scorecards
│   │   │       │   ├── ScoreSimulationService.java    # Automated probabilistic match simulation
│   │   │       │   ├── ScoreCalculationService.java   # Win probabilities, situations, result logic
│   │   │       │   └── WebSocketBroadcastService.java # STOMP broadcasting helper
│   │   │       │
│   │   │       └── init/                              # Seed Data
│   │   │           └── DataInitializer.java           # Initial teams, squads, and tournament matches
│   │   │
│   │   └── resources/
│   │       ├── application.properties                 # Server, H2, and simulation configuration
│   │       └── static/                                # Frontend UI Assets (Embedded)
│   │           ├── index.html                         # Responsive stadium dark-neon dashboard
│   │           ├── css/
│   │           │   └── styles.css                     # Glassmorphic UI design system
│   │           └── js/
│   │               ├── websocket.js                   # SockJS + STOMP subscription wrapper
│   │               └── app.js                         # State management & dynamic UI renderer
```

---

## 🚀 How to Run the Application

### Option A: Using Maven (Recommended)
Open your terminal inside the project root (`Live_Score_Cricket`) and run:
```bash
mvn spring-boot:run
```

### Option B: Build and Run Executable JAR
```bash
mvn clean package -DskipTests
java -jar target/live-score-cricket-1.0.0.jar
```

### Accessing the Web Application
Once the application starts up:
- **Live Frontend Dashboard**: [http://localhost:8080](http://localhost:8080)
- **Interactive Swagger 3 API Docs**: [http://localhost:8080/swagger-ui.html](http://localhost:8080/swagger-ui.html)
- **H2 In-Memory Database Console**: [http://localhost:8080/h2-console](http://localhost:8080/h2-console)  
  *(JDBC URL: `jdbc:h2:mem:cricketdb`, User: `sa`, Password: empty)*

---

## 📡 API Endpoints (REST & OpenAPI)

### Matches & Scorecards
- `GET /api/matches` — Returns all matches (supports query filter `?status=LIVE` / `COMPLETED` / `UPCOMING`).
- `GET /api/matches/{id}` — Returns full match detail: on-crease batsmen, active bowler, win probability, recent balls, and scorecards.
- `POST /api/matches` — Creates a new cricket match.
- `POST /api/matches/{id}/reset` — Resets match to 0.0 overs to start fresh.
- `GET /api/matches/{id}/commentary` — Retrieves ball-by-ball commentary stream.

### Simulation & Manual Scoring
- `POST /api/matches/{id}/simulate/start?speedMs=2200` — Starts or resumes automatic simulation.
- `POST /api/matches/{id}/simulate/pause` — Pauses real-time simulation.
- `POST /api/matches/{id}/simulate/step` — Simulates exactly 1 ball.
- `POST /api/matches/{id}/ball` — Records a manual ball event with custom outcome (DOT, SINGLE, FOUR, SIX, WICKET, etc.).

### Teams & Players
- `GET /api/teams` — Returns registered teams and player rosters.
- `GET /api/players` — Returns all players and career averages/strike rates.

---

## ⚡ WebSocket Real-Time Subscriptions

The backend exposes a STOMP broker at `/ws` (with SockJS fallback):
- **`/topic/match/{id}`**: Emits real-time `MatchDetailDto` after every single ball event.
- **`/topic/live-matches`**: Emits updated `MatchSummaryDto` whenever any match state or score changes.

---

## 🎲 Cricket Simulation Engine Details

The built-in `ScoreSimulationService` implements realistic statistical probabilities calibrated against international T20/ODI frequencies:
- **Dot Ball**: ~32%
- **Single**: ~30%
- **Double**: ~10%
- **Three**: ~2%
- **Boundary Four**: ~12%
- **Maximum Six**: ~6%
- **Wicket**: ~5% (with random dismissal methods: Caught, Bowled, LBW, Run Out, Stumped)
- **Extras**: ~3% (Wides, No-balls, Byes)

Upon every legal delivery:
1. Active striker's balls and runs are updated.
2. Active bowler's figures (balls, maidens, runs, wickets) are updated.
3. If runs are odd (1, 3), strike is automatically rotated.
4. When 6 legal balls complete an over, strike rotates and a new bowler from the fielding team is assigned.
5. Inning ends automatically when maximum overs are bowled or 10 wickets fall. Target is computed and the 2nd innings begins seamlessly!
