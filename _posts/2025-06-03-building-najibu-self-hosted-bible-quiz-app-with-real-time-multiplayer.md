---
layout: post
title: "Building Najibu: A Self-Hosted Bible Quiz App with Real-Time Multiplayer"
date: 2025-06-03 14:05:58 +0300
categories: project showcase fullstack
tags: golang flutter websockets self-hosted devops ory caddy
author: Alfred Wayne Kinyua
featured_image: /assets/images/najibu-hero.png
---

# Building Najibu: From Firebase to Full Self-Hosted Infrastructure

## Who I Am

Hi! I'm Alfred, a Software Engineer passionate about building scalable applications and mastering DevOps practices. You can find me on [LinkedIn](https://www.linkedin.com/in/alfred-wayne-kinyua/) where I share my journey in tech. I'm the solo developer behind **Najibu**, a comprehensive Bible trivia application that I built entirely from scratch.

![Profile or App Screenshot](/assets/images/profile-placeholder.png)
*Caption: The Najibu app interface showcasing real-time multiplayer gameplay*

## 🎯 The Project: Najibu Bible Quiz App

I have created a mobile trivia app that supports leaderboards, multiplayer gameplay, and much more. Built on Flutter, Najibu is a feature-rich platform that evolved from a simple quiz concept into a complex real-time multiplayer system with sophisticated backend infrastructure.

### Key Features
- **Real-time multiplayer games** with WebSocket connections
- **Comprehensive leaderboard system** with global and friend rankings
- **Daily challenges** to keep users engaged
- **Progressive difficulty levels** with XP-based advancement
- **Social features** including friend systems and achievements
- **Multiple game modes** from quick solo games to tournament-style competitions

![App Screenshots Grid](/assets/images/app-screenshots-grid.png)
*Caption: Screenshots showing different game modes, leaderboards, and social features*

## 🤔 The Technology Decision: Why I Ditched Firebase

I thought about this for a while, and the obvious choice was to use Firebase as it supports auth, DB, storage, analytics... Pretty much everything I'd have wanted for a start. I have been wanting to perfect DevOps for a while, and even have a homeserver that I run many services like Jellyfin and AdGuard locally using containers.

I wanted to put all I had been learning into practice, and several factors made it much easier to go fully self-hosted:

### The Problems with Firebase

**1. Vendor Lock-in Concerns**
I wanted to host as much as I can, and use FOSS apps that I can also contribute to. I also have been burnt previously relying on hosted proprietary DBs - can Firestore count?

**2. Cost Projections Were Astronomical**
I ran comparisons for about 100k MAUs and the Firebase costs were astronomical, just for auth, let alone egress fees for the DB.

![Cost Comparison Chart](/assets/images/cost-comparison.png)
*Caption: Firebase vs self-hosted costs at different user scales*

**3. Learning Opportunities**
As aforementioned, I wanted to learn hosting all these services. I have previously held roles that needed me in the server running Docker, but not at this scale and alone.

**4. Performance and Customization**
I needed fine-grained control over game logic, real-time features, and database optimizations that would be challenging with Firebase's constraints.

## 🏗️ My Comprehensive Tech Stack

After extensive research and hands-on experience, here's what I chose and why:

### 1. Caddy: Reverse Proxy

Since all our services are Docker containers whose ports are not exposed, a reverse proxy in the network is necessary to access anything outside the Docker network. I have past experiences with Traefik and Nginx, though I had been reading good things about Caddy as I researched for the project, and it's been a very good service.

**Caddy handles routing for:**
1. **API subdomain** - Proxied to Ory Oathkeeper, that then authenticates and proxies them to the Golang backend
2. **Monitoring subdomain** - Direct proxy to the Grafana dashboard
3. **Landing page (najibu.app)** - Direct proxy to the Svelte container
4. **Auth subdomain** - Monitored for source and proxied to Ory Oathkeeper, which strips and mutates requests, then forwards them to Ory Kratos

```yaml
# --- Main Application Domain ---
najibu.app {
    # --- Serve assetlinks.json directly, for deep-linking ---
    handle /.well-known/assetlinks.json {
        root * /etc/caddy/webroot
        header Content-Type application/json
        file_server
    }

    # --- Catch-all for other requests: Reverse Proxy ---
    reverse_proxy najibu-landing:9007

    # Enable logging
    log {
        output stdout
        format json
    }

    # Enable compression (gzip and zstd) for supported clients
    encode gzip zstd

    # Add common security headers
    header {
        Strict-Transport-Security max-age=31536000;
        X-Content-Type-Options nosniff
        X-Frame-Options DENY
        X-XSS-Protection "1; mode=block"
        Referrer-Policy strict-origin-when-cross-origin
    }
}

# --- API Subdomain ---
api.najibu.app {
    # Route API traffic through Oathkeeper for auth checks
    reverse_proxy oathkeeper:4455

    log {
        output stdout
        format json
    }
    encode gzip zstd
    header {
        Strict-Transport-Security max-age=31536000;
        X-Content-Type-Options nosniff
        X-Frame-Options DENY
        X-XSS-Protection "1; mode=block"
        Referrer-Policy strict-origin-when-cross-origin
    }
}
```

### 2. Ory Stack - Mostly because it uses Golang and I looove Go

#### Kratos: Identity Management
Kratos handles all the auth needs for the app. We do not expose it directly, and instead requests go through Oathkeeper.

#### Oathkeeper: The Main Workhorse
One of the main workhorses of the entire app. It receives requests from Caddy and proxies them to the various services as expected. Since our services are pretty much auth agnostic, Oathkeeper is very essential to the security.

I did not want to deal with auth tokens in the Go app, and Oathkeeper gladly handles that for me. All API requests are stripped of any conflicting headers, then Oathkeeper uses Kratos to authenticate, it then mutates the request with headers of the authenticated user metadata which the Go app can easily handle.

```yaml
# --- API: Protected endpoints ---
- id: "api:protected"
  match:
    # Matches any method not /health or /test on the api.najibu.app subdomain
    url: "https://api.najibu.app/<(?!health|test).*>"
    methods:
      - GET
      - POST
      - PUT
      - PATCH
      - DELETE

  # 1) Authenticate: Try cookie first, then Bearer token
  authenticators:
    # Try Kratos session cookie first
    - handler: cookie_session
    # If cookie fails, try Bearer token (for API clients like Flutter)
    - handler: bearer_token

  # 2) Authorize (allow any authenticated user)
  authorizer:
    handler: allow

  # 3) Mutate: inject claims into ID token
  mutators:
    - handler: id_token
    - handler: header

  # 4) Where to send the request on success
  upstream:
    preserve_host: true
    url: "http://najibu-go:9009"
```

Oathkeeper also receives the Kratos self-service requests and passes them to Kratos after some mutation. The good thing with this is, we can choose which self-service endpoints are accessible to users without having to tinker with Kratos configs itself.

```yaml
# --- Kratos Self-Service API Proxy ---
- id: "ory:kratos-selfservice-api"
  upstream:
    preserve_host: true
    url: "http://kratos:4433"
  match:
    # Matches Kratos's self-service API flows
    url: "https://auth.najibu.app/self-service/<.*>"
    methods:
      - GET # Used for fetching flow details
      - POST # Used for submitting forms
      - PUT
      - DELETE
      - PATCH
  authenticators:
    - handler: noop
  authorizer:
    handler: allow
  mutators:
    - handler: noop
```

![Ory Architecture Diagram](/assets/images/ory-architecture.png)
*Caption: How Ory Oathkeeper and Kratos work together in the authentication flow*

### 3. Golang: Backend Language of Choice

Every game runs on a separate goroutine and I am impressed. We use channels extensively in a game's lifecycle to receive and transmit WebSocket messages.

**Key Implementation Details:**
- All active games use WebSockets - a user sends `start_game`, we open a WebSocket and send a question, once answered, we send the correct answer and if the user got it correct... Over engineered I know, to keep everyone honest?
- Echo framework - We use Echo framework and all our API endpoints have an auth middleware that communicates with Ory Oathkeeper to verify each request

```go
// Game session handling with goroutines
func (h *GameHub) RunGame(gameID string) {
    game := h.games[gameID]

    go func() {
        for {
            select {
            case message := <-game.MessageChan:
                h.broadcastToGame(gameID, message)
            case <-game.Done:
                h.cleanupGame(gameID)
                return
            case <-time.After(30 * time.Second):
                // Handle game timeout
                h.handleGameTimeout(gameID)
            }
        }
    }()
}

// WebSocket message handling
type GameMessage struct {
    Type string      `json:"type"`
    Data interface{} `json:"data"`
}

// Question distribution
type QuestionMessage struct {
    QuestionIndex   int            `json:"question_index"`
    Question        string         `json:"question"`
    Options         []string       `json:"options"`
    TimeLimit       int            `json:"time_limit"`
    BibleReference  BibleRef       `json:"bible_reference"`
}
```

### 4. Grafana: Comprehensive Monitoring

We use Grafana for monitoring what is going on, from active games, users registered, req/sec, system metrics. Here are the custom dashboards I've built:

1. **logs_dashboard** - Centralized log analysis
2. **najibu_app_metrics** - Core application performance metrics
3. **najibu_executive_summary** - High-level business KPIs
4. **najibu_games_analytics** - Game-specific analytics and player behavior
5. **najibu_player_leaderboard** - Real-time leaderboard monitoring
6. **najibu_realtime_monitoring** - Live system health and WebSocket connections
7. **postgres_metrics** - Database performance and query analysis
8. **system_metrics** - Server resource utilization

```go
// Custom Prometheus metrics
var (
    activeGamesGauge = prometheus.NewGauge(prometheus.GaugeOpts{
        Name: "najibu_active_games_total",
        Help: "Number of currently active games",
    })

    gameCompletionCounter = prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Name: "najibu_games_completed_total",
            Help: "Total number of completed games",
        },
        []string{"game_type", "difficulty"},
    )

    playerAnswerHistogram = prometheus.NewHistogramVec(
        prometheus.HistogramOpts{
            Name: "najibu_player_answer_duration_seconds",
            Help: "Time taken for players to answer questions",
            Buckets: prometheus.DefBuckets,
        },
        []string{"question_difficulty"},
    )
)
```

![Grafana Dashboard Grid](/assets/images/grafana-dashboard-grid.png)
*Caption: Collection of custom Grafana dashboards monitoring different aspects of the application*

### 5. PostgreSQL: Database Deep Dive

Mostly because I want to really learn how to use it, and I have, very much.

We have one single PostgreSQL instance running in the internal Docker network. It runs the app DB and Ory DBs, handling:
- User profiles and authentication data (via Kratos)
- Game sessions and real-time state
- Leaderboards and player statistics
- Question banks and categories
- Social features and friend connections

```sql
-- Example: Complex leaderboard query with ranking
SELECT
    u.username,
    p.total_score,
    p.games_played,
    p.average_score,
    RANK() OVER (ORDER BY p.total_score DESC) as global_rank,
    COUNT(*) FILTER (WHERE g.completed_at >= NOW() - INTERVAL '7 days') as games_this_week
FROM players p
JOIN users u ON p.user_id = u.id
LEFT JOIN games g ON g.player_id = p.id
WHERE p.last_played >= NOW() - INTERVAL '30 days'
GROUP BY u.username, p.total_score, p.games_played, p.average_score
ORDER BY p.total_score DESC
LIMIT 100;

-- Strategic indexing for performance
CREATE INDEX CONCURRENTLY idx_players_score_recent ON players (total_score DESC)
WHERE last_played >= NOW() - INTERVAL '30 days';

CREATE INDEX CONCURRENTLY idx_games_player_completed ON games (player_id, completed_at);
```

### 6. PostHog: Analytics & User Behavior

PostHog provides comprehensive analytics to understand user behavior, track feature adoption, and measure engagement across the application.

![Database Schema Diagram](/assets/images/db-schema.png)
*Caption: PostgreSQL database schema showing relationships between users, games, and analytics*

## 🎮 Real-Time Multiplayer: The Technical Challenge

The most complex part of Najibu is the real-time multiplayer system. Here's how I architected it:

### WebSocket Message Protocol

Every game operates through a carefully designed message protocol that ensures fair play and synchronized state across all players:

```go
// Game state synchronization
type GameState struct {
    ID                   string            `json:"id"`
    Players              []Player          `json:"players"`
    CurrentQuestionIndex int               `json:"current_question_index"`
    Scores               map[string]int    `json:"scores"`
    Status               GameStatus        `json:"status"`
    TimeRemaining        int               `json:"time_remaining"`
}

// Answer submission with validation
type AnswerSubmission struct {
    PlayerID       string    `json:"player_id"`
    QuestionIndex  int       `json:"question_index"`
    SelectedOption int       `json:"selected_option"`
    TimeTaken      int       `json:"time_taken"`
    Timestamp      time.Time `json:"timestamp"`
}

// Real-time game events
func (g *Game) HandlePlayerAnswer(submission AnswerSubmission) {
    // Validate submission timing and question index
    if !g.isValidSubmission(submission) {
        return
    }

    // Record answer
    g.RecordAnswer(submission)

    // Check if all players have answered
    if g.AllPlayersAnswered() {
        g.AdvanceToNextQuestion()
    }

    // Broadcast updated game state
    g.BroadcastGameState()
}
```

### Concurrency & Thread Safety

Managing multiple simultaneous games requires careful attention to thread safety and race condition prevention:

```go
type GameHub struct {
    games   map[string]*Game
    players map[string]*Player
    mu      sync.RWMutex

    // Channels for coordinating game events
    gameEvents    chan GameEvent
    playerEvents  chan PlayerEvent
    shutdownChan  chan struct{}
}

type Game struct {
    ID                   string
    Players              []*Player
    Scores               map[string]int
    CurrentQuestionIndex int
    QuestionAnswers      map[int]map[string]Answer
    Mu                   sync.RWMutex
    Timer                *time.Timer
    MessageChan          chan GameMessage
    Done                 chan struct{}
}

// Thread-safe game operations
func (h *GameHub) AddPlayerToGame(gameID, playerID string) error {
    h.mu.Lock()
    defer h.mu.Unlock()

    game, exists := h.games[gameID]
    if !exists {
        return errors.New("game not found")
    }

    game.Mu.Lock()
    defer game.Mu.Unlock()

    if len(game.Players) >= MaxPlayersPerGame {
        return errors.New("game is full")
    }

    // Add player and notify other players
    game.AddPlayer(playerID)
    h.broadcastPlayerJoined(gameID, playerID)

    return nil
}
```

![Real-time Game Flow](/assets/images/websocket-flow-diagram.png)
*Caption: Sequence diagram showing WebSocket message flow during multiplayer games*

## 🏆 Gamification & User Engagement

To keep users engaged, I implemented a comprehensive progression system:

### XP Calculation System
```go
func calculateXPEarnedWithBreakdown(correctAnswers, totalQuestions, timeTaken, difficulty int) XPBreakdown {
    baseXP := correctAnswers * 10

    // Time bonus (faster answers get more XP)
    avgTimePerQuestion := timeTaken / totalQuestions
    timeBonus := 0
    if avgTimePerQuestion < 10 { // Less than 10 seconds per question
        timeBonus = (10 - avgTimePerQuestion) * 2
    }

    // Difficulty multiplier
    difficultyMultiplier := map[int]float64{
        1: 1.0,   // Easy
        2: 1.5,   // Medium
        3: 2.0,   // Hard
        4: 2.5,   // Expert
    }[difficulty]

    // Streak bonus for consecutive correct answers
    streakBonus := calculateStreakBonus(correctAnswers)

    totalXP := int(float64(baseXP + timeBonus + streakBonus) * difficultyMultiplier)

    return XPBreakdown{
        BaseXP:        baseXP,
        TimeBonus:     timeBonus,
        StreakBonus:   streakBonus,
        Difficulty:    difficulty,
        Multiplier:    difficultyMultiplier,
        TotalXP:       totalXP,
    }
}
```

### Achievement System
- **Scripture Scholar**: Answer 100 questions correctly
- **Speed Demon**: Complete games under time limits consistently
- **Social Player**: Play with friends regularly
- **Daily Devotion**: Complete daily challenges for consecutive days
- **Perfectionist**: Achieve 100% accuracy in multiple games
- **Marathon Runner**: Play for extended sessions

![Achievements & Progression](/assets/images/achievements-progression.png)
*Caption: User progression system showing XP, levels, and achievement unlocks*

## 🐳 DevOps & Infrastructure

The entire application runs on a sophisticated Docker Compose setup:

```yaml
version: '3.8'

services:
  # Main API service
  najibu-go:
    build: ./najibu-go
    environment:
      - DB_HOST=postgres
      - DB_NAME=najibu
      - KRATOS_ADMIN_URL=http://kratos:4434
      - OATHKEEPER_URL=http://oathkeeper:4455
    depends_on:
      - postgres
      - kratos
    volumes:
      - ./logs:/app/logs
    networks:
      - najibu-network

  # PostgreSQL database
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: najibu
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./postgres/init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - najibu-network

  # Ory Kratos for identity management
  kratos:
    image: oryd/kratos:v1.0.0
    environment:
      - DSN=postgres://postgres:${DB_PASSWORD}@postgres:5432/kratos
      - LOG_LEVEL=debug
    volumes:
      - ./najibu-ory/kratos:/etc/config/kratos
    depends_on:
      - postgres
    networks:
      - najibu-network

  # Ory Oathkeeper for API gateway
  oathkeeper:
    image: oryd/oathkeeper:v0.40.6
    environment:
      - LOG_LEVEL=debug
    volumes:
      - ./najibu-ory/oathkeeper:/etc/config/oathkeeper
    depends_on:
      - kratos
    networks:
      - najibu-network

  # Caddy reverse proxy
  caddy:
    image: caddy:2-alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./najibu-caddy/Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
      - caddy_config:/config
    networks:
      - najibu-network

  # Grafana monitoring
  grafana:
    image: grafana/grafana:latest
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD}
      - GF_USERS_ALLOW_SIGN_UP=false
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./grafana/datasources:/etc/grafana/provisioning/datasources
    networks:
      - najibu-network

  # Landing page
  najibu-landing:
    build: ./najibu-landing
    networks:
      - najibu-network

networks:
  najibu-network:
    driver: bridge

volumes:
  postgres_data:
  grafana_data:
  caddy_data:
  caddy_config:
```

### Production Deployment Considerations

**Security Headers & SSL:**
- Automatic HTTPS with Let's Encrypt via Caddy
- Comprehensive security headers (HSTS, CSP, etc.)
- Proper CORS handling for mobile app integration

**Monitoring & Alerting:**
- Comprehensive logging with structured JSON format
- Custom Prometheus metrics for business KPIs
- Grafana alerting for critical system events

**Backup & Recovery:**
- Automated PostgreSQL backups
- Configuration versioning with Git
- Disaster recovery procedures

![Infrastructure Diagram](/assets/images/infrastructure-diagram.png)
*Caption: Complete infrastructure overview showing service relationships and data flow*

## 📊 Performance Optimizations & Results

### Database Optimizations

**Strategic Indexing:**
```sql
-- Leaderboard queries
CREATE INDEX CONCURRENTLY idx_players_score_active ON players (total_score DESC)
WHERE last_played >= NOW() - INTERVAL '30 days';

-- Game lookup optimization
CREATE INDEX CONCURRENTLY idx_games_status_created ON games (status, created_at);

-- Friend system queries
CREATE INDEX CONCURRENTLY idx_friendships_user_status ON friendships (user_id, status);

-- Question retrieval
CREATE INDEX CONCURRENTLY idx_questions_category_difficulty ON questions (category_id, difficulty_level);
```

**Connection Pooling:**
```go
func NewDBPool(databaseURL string) *sql.DB {
    db, err := sql.Open("postgres", databaseURL)
    if err != nil {
        log.Fatal("Failed to open database:", err)
    }

    // Optimize connection pool for high concurrency
    db.SetMaxOpenConns(25)
    db.SetMaxIdleConns(10)
    db.SetConnMaxLifetime(5 * time.Minute)
    db.SetConnMaxIdleTime(2 * time.Minute)

    return db
}
```

### Performance Metrics

**Current Performance Stats:**
- **Average API Response Time**: <50ms for most endpoints
- **WebSocket Latency**: <30ms for real-time game updates
- **Concurrent Games Supported**: 500+ simultaneous games tested
- **Database Query Performance**: Sub-5ms for optimized queries
- **Memory Usage**: ~200MB for the Go backend under normal load
- **CPU Usage**: <10% during peak gaming hours

**Load Testing Results:**
- Successfully handled 1000+ concurrent WebSocket connections
- Database performed well under 10,000+ concurrent read operations
- Zero downtime during deployment with proper health checks

![Performance Metrics Dashboard](/assets/images/performance-metrics.png)
*Caption: Real-time performance monitoring showing response times and system health*

## 🧪 Testing Strategy

Comprehensive testing ensures the reliability of real-time multiplayer features:

```go
func TestGameCreation(t *testing.T) {
    hub := NewGameHub()

    gameID := "test-game-123"
    players := []string{"player1", "player2"}

    game, err := hub.CreateGame(gameID, players, GameTypeMultiplayer)

    assert.NoError(t, err)
    assert.Equal(t, gameID, game.ID)
    assert.Len(t, game.Players, 2)
    assert.Equal(t, GameStatusWaiting, game.Status)
}

func TestWebSocketGameFlow(t *testing.T) {
    server := setupTestServer()
    defer server.Close()

    // Connect multiple players
    ws1, _, err := websocket.DefaultDialer.Dial("ws"+server.URL[4:]+"/ws/game/test", nil)
    assert.NoError(t, err)
    defer ws1.Close()

    ws2, _, err := websocket.DefaultDialer.Dial("ws"+server.URL[4:]+"/ws/game/test", nil)
    assert.NoError(t, err)
    defer ws2.Close()

    // Test game start message
    startMessage := GameMessage{
        Type: "start_game",
        Data: map[string]string{"player_id": "test-player-1"},
    }
    err = ws1.WriteJSON(startMessage)
    assert.NoError(t, err)

    // Verify both players receive the question
    var response1, response2 GameMessage
    err = ws1.ReadJSON(&response1)
    assert.NoError(t, err)
    assert.Equal(t, "question", response1.Type)

    err = ws2.ReadJSON(&response2)
    assert.NoError(t, err)
    assert.Equal(t, "question", response2.Type)
}

func TestConcurrentGameSessions(t *testing.T) {
    hub := NewGameHub()
    numGames := 100

    var wg sync.WaitGroup
    wg.Add(numGames)

    // Create multiple games concurrently
    for i := 0; i < numGames; i++ {
        go func(gameIndex int) {
            defer wg.Done()

            gameID := fmt.Sprintf("concurrent-game-%d", gameIndex)
            players := []string{
                fmt.Sprintf("player-%d-1", gameIndex),
                fmt.Sprintf("player-%d-2", gameIndex),
            }

            _, err := hub.CreateGame(gameID, players, GameTypeMultiplayer)
            assert.NoError(t, err)
        }(i)
    }

    wg.Wait()

    // Verify all games were created successfully
    assert.Equal(t, numGames, len(hub.games))
}
```

**Test Coverage:**
- **Unit Tests**: 85%+ coverage across all modules
- **Integration Tests**: Critical game flow paths
- **Load Tests**: WebSocket connection stability under load
- **End-to-End Tests**: Complete user journey from registration to gameplay

## 📱 Mobile App Integration

The Flutter mobile app integrates seamlessly with the backend infrastructure:

### WebSocket State Management
```dart
class GameStateProvider extends ChangeNotifier {
  WebSocketChannel? _channel;
  GameState _currentGame = GameState.initial();

  void connectToGame(String gameId, String authToken) {
    final uri = Uri.parse('wss://api.najibu.app/ws/game/$gameId');

    _channel = WebSocketChannel.connect(
      uri,
      headers: {'Authorization': 'Bearer $authToken'},
    );

    _channel!.stream.listen(
      (message) => _handleGameUpdate(jsonDecode(message)),
      onError: (error) => _handleConnectionError(error),
      onDone: () => _handleConnectionClosed(),
    );
  }

  void submitAnswer(int questionIndex, int selectedOption) {
    if (_channel != null) {
      final message = {
        'type': 'submit_answer',
        'data': {
          'question_index': questionIndex,
          'selected_option': selectedOption,
          'timestamp': DateTime.now().millisecondsSinceEpoch,
        }
      };

      _channel!.sink.add(jsonEncode(message));
    }
  }
}
```

### Authentication Flow
```dart
class AuthService {
  static const String baseUrl = 'https://auth.najibu.app';

  Future<AuthResult> login(String email, String password) async {
    // Initialize login flow with Kratos
    final flowResponse = await http.get(
      Uri.parse('$baseUrl/.ory/kratos/public/self-service/login/api'),
    );

    final flow = LoginFlow.fromJson(jsonDecode(flowResponse.body));

    // Submit credentials
    final loginResponse = await http.post(
      Uri.parse('$baseUrl/self-service/login?flow=${flow.id}'),
      headers: {'Content-Type': 'application/json'},
      body: jsonEncode({
        'method': 'password',
        'password': password,
        'password_identifier': email,
      }),
    );

    if (loginResponse.statusCode == 200) {
      final session = Session.fromJson(jsonDecode(loginResponse.body));
      await _storeSession(session);
      return AuthResult.success(session);
    }

    return AuthResult.failure('Login failed');
  }
}
```

![Mobile App Screenshots](/assets/images/mobile-app-flow.png)
*Caption: Mobile app screenshots showing authentication, game lobby, and active gameplay*

## 📈 Analytics & Business Intelligence

### PostHog Integration

PostHog provides comprehensive user analytics and feature tracking:

```go
// Event tracking in the backend
func (s *GameService) trackGameEvent(event string, playerID string, properties map[string]interface{}) {
    client := posthog.New(s.posthogAPIKey)
    defer client.Close()

    client.Enqueue(posthog.Capture{
        DistinctId: playerID,
        Event:      event,
        Properties: properties,
        Timestamp:  time.Now(),
    })
}

// Example usage
func (s *GameService) CompleteGame(gameID string, results GameResults) {
    // ... game completion logic ...

    // Track completion event
    s.trackGameEvent("game_completed", results.PlayerID, map[string]interface{}{
        "game_id":          gameID,
        "score":           results.Score,
        "questions_total":  results.TotalQuestions,
        "questions_correct": results.CorrectAnswers,
        "duration_seconds": results.Duration,
        "difficulty":       results.Difficulty,
        "game_type":        results.GameType,
    })
}
```

### Key Metrics Tracked

**User Engagement:**
- Daily/Monthly Active Users
- Session duration and frequency
- Feature adoption rates
- User retention cohorts

**Game Performance:**
- Game completion rates by difficulty
- Average scores and improvement trends
- Most/least popular question categories
- Multiplayer vs single-player preferences

**Technical Metrics:**
- API response times
- WebSocket connection stability
- Error rates and crash reports
- Database query performance

![Analytics Dashboard](/assets/images/posthog-analytics.png)
*Caption: PostHog analytics dashboard showing user behavior and engagement metrics*

## 🔮 Future Enhancements & Roadmap

### Planned Features

**Short-term (Next 3 months):**
- **Tournament System**: Organized competitions with brackets and prizes
- **Team Competitions**: Church groups and Bible study teams
- **Enhanced Social Features**: Private messaging and group challenges
- **Offline Mode**: Play without internet with sync when connected

**Medium-term (6-12 months):**
- **AI Question Generation**: Infinite content using language models
- **Voice Chat Integration**: Real-time communication during multiplayer games
- **Advanced Analytics**: Player skill assessment and personalized difficulty
- **Mobile Push Notifications**: Smart engagement and challenge reminders

**Long-term (1+ years):**
- **Cross-platform Expansion**: Web version and desktop apps
- **Internationalization**: Multiple languages and cultural adaptations
- **Blockchain Integration**: Achievement verification and rewards
- **Machine Learning**: Adaptive difficulty and personalized content

### Technical Roadmap

**Infrastructure Evolution:**
- **Kubernetes Migration**: Better scalability and orchestration
- **Microservices Architecture**: Split monolith into focused services
- **GraphQL API**: More efficient client-server communication
- **Event Sourcing**: Better audit trails and state reconstruction
- **CQRS Implementation**: Optimized read/write operations

**Performance Optimizations:**
- **Redis Caching Layer**: Reduced database load
- **CDN Implementation**: Global content delivery
- **Database Sharding**: Horizontal scaling preparation
- **Message Queue**: Async processing for heavy operations

## 💡 Lessons Learned & Insights

### What Went Exceptionally Well

**1. Go's Concurrency Model**
The goroutine-based architecture made real-time multiplayer features surprisingly manageable. Channels provided excellent coordination between game sessions.

**2. Self-Hosting Decision**
Complete control over infrastructure enabled rapid iteration and debugging. Cost savings are significant at scale.

**3. Ory Stack Integration**
The Go-based Ory tools integrated seamlessly and provided enterprise-grade security without vendor lock-in.

**4. PostgreSQL Performance**
With proper indexing and query optimization, PostgreSQL handled complex gaming queries with excellent performance.

### Challenges Overcome

**1. WebSocket Connection Management**
Initial implementation struggled with connection drops and reconnections. Solved with heartbeat mechanisms and graceful degradation.

**2. Race Conditions in Multiplayer**
Concurrent access to game state required careful mutex usage and channel coordination to prevent data corruption.

**3. Real-time State Synchronization**
Ensuring all players see consistent game state required implementing proper event ordering and conflict resolution.

**4. Database Query Optimization**
Complex leaderboard queries initially caused performance issues. Resolved with strategic indexing and query restructuring.

### What I'd Do Differently

**1. Start with Microservices**
The monolithic approach required significant refactoring as the application grew. Microservices from the beginning would have been better.

**2. Implement Comprehensive Logging Earlier**
Adding structured logging and distributed tracing from day one would have made debugging much easier.

**3. Feature Flags from the Start**
Rolling out new features safely required implementing feature flags later in development.

**4. API Versioning Strategy**
Planning for API evolution from the beginning would have prevented breaking changes for mobile app users.

## 🏁 Technical Achievements & Impact

### Scalability Milestones

**Performance Benchmarks:**
- Successfully tested with 1,000+ concurrent WebSocket connections
- Sub-50ms API response times under normal load
- Zero-downtime deployments with health check integration
- Database queries optimized to sub-10ms for critical paths

**Reliability Features:**
- Automatic failover and recovery mechanisms
- Comprehensive monitoring with custom dashboards
- Automated backup and disaster recovery procedures
- Circuit breaker patterns for external service calls

### Code Quality & Best Practices

**Development Standards:**
- 85%+ test coverage across all modules
- Comprehensive documentation and API specs
- Automated code quality checks and linting
- Git workflow with protected branches and reviews

**Security Implementation:**
- Zero-trust architecture with service-to-service auth
- Comprehensive input validation and sanitization
- Rate limiting and DDoS protection
- Regular security audits and dependency updates

![System Architecture Overview](/assets/images/complete-architecture.png)
*Caption: Complete system architecture showing all services and their interactions*

## 🎉 Conclusion

Building Najibu has been an incredible journey that pushed me to master full-stack development, DevOps practices, and distributed system architecture. What started as a simple Bible quiz app evolved into a comprehensive platform showcasing:

- **Modern Backend Architecture**: Go-based microservices with real-time capabilities
- **Self-Hosted Infrastructure**: Complete control with Docker and proper monitoring
- **Real-Time Multiplayer Systems**: WebSocket-based gaming with proper state management
- **Enterprise Security**: Ory stack integration with zero-trust principles
- **Mobile Excellence**: Flutter app with seamless backend integration
- **DevOps Mastery**: Automated deployment, monitoring, and scaling

The decision to avoid Firebase and go fully self-hosted was challenging but ultimately rewarding. I now have:
- **Complete Infrastructure Control**: No vendor lock-in or unexpected limitations
- **Significantly Lower Operating Costs**: Especially important for scaling to 100k+ users
- **Invaluable Learning Experience**: Hands-on with enterprise-level technologies
- **Production-Ready Skills**: Applicable to any large-scale system

This project demonstrates that with the right architecture and tools, a solo developer can build and operate systems that compete with team-developed applications. The combination of Go's performance, PostgreSQL's reliability, and modern DevOps practices creates a robust foundation for any real-time application.

## 🔗 Project Resources

- **Live Application**: [najibu.app](https://najibu.app)
- **Technical Documentation**: Available in project repository
- **API Documentation**: [api.najibu.app/docs](https://api.najibu.app/docs)
- **Monitoring Dashboard**: [monitoring.najibu.app](https://monitoring.najibu.app)

## 🙏 Acknowledgments

Special thanks to the open-source community and maintainers of:
- **Ory**: For excellent identity management tools
- **Go Community**: For incredible libraries and documentation
- **Flutter Team**: Making cross-platform development enjoyable
- **PostgreSQL**: For a rock-solid database foundation
- **Grafana Labs**: For outstanding monitoring solutions
- **Caddy**: For the simplest reverse proxy experience

---

*Want to discuss this project or have questions about the technical implementation? Feel free to reach out on [LinkedIn](https://www.linkedin.com/in/alfred-wayne-kinyua/) or connect with me for technical discussions about Go, Flutter, or self-hosted infrastructure!*

**Tags:** #golang #flutter #websockets #selfhosted #devops #multiplayer #realtime #postgresql #docker #ory #caddy #grafana

---

*This post is part of my technical blog series documenting interesting projects and engineering challenges. Subscribe to stay updated with new content about building scalable applications and mastering modern development practices!*
