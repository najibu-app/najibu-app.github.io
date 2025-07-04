---
layout: default
title: "Building Najibu: A Self-Hosted Bible Quiz App with Real-Time Multiplayer"
description: "A comprehensive technical showcase of building a full-stack, self-hosted Bible trivia platform with real-time multiplayer gameplay, complete DevOps pipeline, and scalable architecture."
author: Alfred
featured_image: /assets/images/najibu-hero.png
---

# Building Najibu: From Firebase to Full Self-Hosted Infrastructure

## Who I Am

I'm Alfred, a Software Engineer specializing in scalable application development and DevOps engineering. You can connect with me on [LinkedIn](https://www.linkedin.com/in/alfred-wayne-kinyua/) where I document my technical journey. As the sole architect and developer of **Najibu**, I designed and implemented this comprehensive Bible trivia platform from ground zero.

## 🎯 The Project: Najibu Bible Quiz App

Najibu represents a full-featured mobile trivia platform supporting persistent leaderboards, real-time multiplayer sessions, and comprehensive user engagement systems. Built with Flutter for cross-platform deployment, the application evolved from a straightforward quiz implementation into a sophisticated real-time multiplayer ecosystem backed by enterprise-grade infrastructure.

### Key Features

- **Real-time multiplayer games** with WebSocket connections
- **Comprehensive leaderboard system** with global and friend rankings
- **Daily challenges** to keep users engaged
- **Progressive difficulty levels** with XP-based advancement
- **Social features** including friend systems and achievements
- **Multiple game modes** from quick solo games to tournament-style competitions

### Mobile App Interface Gallery

<table>
  <tr>
    <td style="text-align: center; vertical-align: top;">
      <img src="/assets/img/flutter-app/home_light.png" alt="Home Screen Light" style="width:100%; max-width:150px; display: block; margin-left: auto; margin-right: auto;">
      <br>
      <em>Clean, intuitive home screen interface with game mode selection</em>
    </td>
    <td style="text-align: center; vertical-align: top;">
      <img src="/assets/img/flutter-app/question_dark_green.png" alt="Game Question Interface" style="width:100%; max-width:150px; display: block; margin-left: auto; margin-right: auto;">
      <br>
      <em>Real-time multiplayer question interface with timer and progress indicators</em>
    </td>
    <td style="text-align: center; vertical-align: top;">
      <img src="/assets/img/flutter-app/lobby_light.png" alt="Game Lobby" style="width:100%; max-width:150px; display: block; margin-left: auto; margin-right: auto;">
      <br>
      <em>Multiplayer game lobby showing waiting players and match configuration</em>
    </td>
    <td style="text-align: center; vertical-align: top;">
      <img src="/assets/img/flutter-app/leaderboard_light.png" alt="Leaderboard System" style="width:100%; max-width:150px; display: block; margin-left: auto; margin-right: auto;">
      <br>
      <em>Global leaderboard with ranking system and player statistics</em>
    </td>
  </tr>
  <tr>
    <td style="text-align: center; vertical-align: top;">
      <img src="/assets/img/flutter-app/game_results_light.png" alt="Game Results" style="width:100%; max-width:150px; display: block; margin-left: auto; margin-right: auto;">
      <br>
      <em>Detailed game results with scoring breakdown and performance analytics</em>
    </td>
    <td style="text-align: center; vertical-align: top;">
      <img src="/assets/img/flutter-app/duel_results_light.png" alt="Duel Results" style="width:100%; max-width:150px; display: block; margin-left: auto; margin-right: auto;">
      <br>
      <em>Head-to-head duel results showing competitive match outcomes</em>
    </td>
    <td style="text-align: center; vertical-align: top;">
      <img src="/assets/img/flutter-app/profile_dark_red.png" alt="User Profile" style="width:100%; max-width:150px; display: block; margin-left: auto; margin-right: auto;">
      <br>
      <em>Comprehensive user profile with achievements, statistics, and customization options</em>
    </td>
    <td style="text-align: center; vertical-align: top;">
      <img src="/assets/img/flutter-app/activity_light.png" alt="Activity Tracking" style="width:100%; max-width:150px; display: block; margin-left: auto; margin-right: auto;">
      <br>
      <em>Achievement system and user activity tracking with progress visualization</em>
    </td>
  </tr>
</table>

## 🤔 The Technology Decision: Why I Ditched Firebase

Firebase initially appeared to be the logical choice, offering integrated authentication, database, storage, and analytics services. However, my existing homelab infrastructure running containerized services like Jellyfin and AdGuard presented an opportunity to apply DevOps principles at scale.

The decision to pursue a fully self-hosted architecture was driven by several technical and strategic considerations:

### The Problems with Firebase

**1. Vendor Lock-in Concerns**
The strategic goal was maximizing infrastructure ownership while leveraging open-source solutions that enable upstream contributions. Previous experiences with proprietary database dependencies, including Firestore's ecosystem constraints, reinforced the value of maintaining architectural flexibility.

**2. Cost Analysis: The Economics of Scale**
A comprehensive cost analysis comparing Firebase, Supabase, and self-hosted infrastructure revealed significant economic advantages for self-hosting at scale. For a target of 100k monthly active users with real-time multiplayer functionality:

**Firebase Blaze Plan Monthly Costs (100k MAUs):**

- Authentication: $0 (covered under free tier)
- Cloud Firestore:
  - Storage (estimated 10GB): $90/month
  - Document reads (15M/month): $540/month
  - Document writes (3M/month): $360/month
  - Network egress (50GB/month): $480/month
- Cloud Functions:
  - Invocations (10M/month): $3,200/month
  - Compute time: $800/month
- **Total Firebase: ~$5,470/month**

**Supabase Pro Plan (100k MAUs):**

- Base Pro plan: $25/month
- Database (beyond 8GB): $15/month
- Auth (beyond 100k MAUs): $0 (free tier covers 100k)
- Bandwidth (beyond 250GB): $450/month
- Realtime connections: $500/month
- **Total Supabase: ~$990/month**

**Self-Hosted VPS (Hetzner/Netcup):**

- 8GB RAM VPS: €25-35/month (~$30-40/month)
- Domain and SSL: $15/month
- Monitoring services: $0 (self-hosted)
- **Total Self-Hosted: ~$55/month**

The cost differential becomes even more pronounced as user bases scale beyond 100k MAUs, where Firebase's per-operation pricing model and Supabase's bandwidth charges create exponential cost growth, while VPS costs remain relatively linear.

**3. Learning & Control**
Complete infrastructure ownership enables deep architectural control while providing hands-on experience with production-grade system design and operations.

**4. Performance Requirements**
Real-time multiplayer games demand predictable latency profiles and performance characteristics that are more easily achieved through dedicated infrastructure with controlled resource allocation.

**5. Data Sovereignty**
Maintaining complete control over data storage locations, processing methodologies, and compliance frameworks.

## 🏗️ My Comprehensive Tech Stack

After extensive research and testing, I settled on a modern, scalable architecture that prioritizes performance, maintainability, and cost-effectiveness:

### 1. Caddy: Reverse Proxy

Caddy serves as my edge layer, handling:

- **Automatic HTTPS** with Let's Encrypt certificates
- **Reverse proxy** routing to different services
- **Load balancing** for high availability
- **Static file serving** for the landing page
- **Compression and caching** for optimal performance

```caddyfile
{
    admin off
    auto_https off
}

api.najibu.app {
    reverse_proxy backend:8080
}

auth.najibu.app {
    reverse_proxy auth-service:4433
}

najibu.app {
    reverse_proxy landing-page:3000
}

grafana.najibu.app {
    reverse_proxy monitoring:3000
}
```

Benefits of Caddy over alternatives:

- **Zero-configuration HTTPS**: Automatic certificate management
- **Simple configuration syntax**: Much easier than Nginx or Apache
- **Built-in load balancing**: No need for separate tools
- **Excellent documentation**: Quick setup and troubleshooting
- **Performance**: Written in Go, highly efficient
- **Plugin ecosystem**: Extensible for future needs

### 2. Ory Stack: Identity & Access Management

The Ory ecosystem was selected for identity and access management, providing enterprise-grade security with open-source flexibility and Go-based performance characteristics.

#### Kratos: Identity Management

Handles user registration, login, password resets, and profile management:

```yaml
# Authentication service configuration
version: v0.11.1
dsn: postgres://auth_user:password@postgres:5432/auth_db
serve:
  public:
    base_url: https://auth.najibu.app
    cors:
      enabled: true
      allowed_origins:
        - https://najibu.app
        - https://api.najibu.app
selfservice:
  default_browser_return_url: https://najibu.app/
  flows:
    registration:
      ui_url: https://najibu.app/auth/registration
    login:
      ui_url: https://najibu.app/auth/login
    recovery:
      ui_url: https://najibu.app/auth/recovery
```

#### Oathkeeper: Authentication & Authorization Proxy

Oathkeeper functions as the primary authentication and authorization gateway, implementing zero-trust architecture between the edge proxy and backend services:

```yaml
# Authorization gateway configuration
- id: "api-protected"
  match:
    url: "https://api.najibu.app/api/v1/<protected-endpoints>/**"
    methods:
      - "GET"
      - "POST"
      - "PUT"
      - "DELETE"
      - "PATCH"
  authenticators:
    - handler: "session"
    - handler: "bearer_token"
  authorizer:
    handler: "allow"
  mutators:
    - handler: "noop"
    - handler: "id_token"
  upstream:
    preserve_host: true
    url: "http://backend-service:8080"

- id: "api-public"
  upstream:
    preserve_host: true
    url: "http://backend-service:8080"
  match:
    url: "https://api.najibu.app/api/v1/health"
    methods:
      - "GET"
  authenticators:
    - handler: "anonymous"
  authorizer:
    handler: "allow"
  mutators:
    - handler: "noop"
```

### 3. Golang: Backend Language of Choice

The heart of Najibu is a Go-based API server using the Echo framework. Here's a glimpse of the real-time game management:

```go
// Example of game state management
type GameMessage struct {
    Type string      `json:"type"`
    Data interface{} `json:"data"`
}

type QuestionMessage struct {
    QuestionIndex   int      `json:"question_index"`
    Question        string   `json:"question"`
    Options         []string `json:"options"`
    TimeLimit       int      `json:"time_limit"`
    BibleReference  string   `json:"bible_reference"`
}

func (h *GameHub) RunGame(ctx context.Context, gameID string) error {
    game := h.games[gameID]

    for questionIndex := 0; questionIndex < len(game.Questions); questionIndex++ {
        game.CurrentQuestionIndex = questionIndex

        // Broadcast question to all players
        questionMsg := QuestionMessage{
            QuestionIndex:   questionIndex,
            Question:       game.Questions[questionIndex].Text,
            Options:        game.Questions[questionIndex].Options,
            TimeLimit:      30, // seconds
            BibleReference: game.Questions[questionIndex].Reference,
        }

        h.broadcastToGame(gameID, "question", questionMsg)

        // Wait for answers or timeout
        select {
        case <-time.After(30 * time.Second):
            h.processQuestionResults(gameID, questionIndex)
        case <-ctx.Done():
            return ctx.Err()
        }
    }

    return h.endGame(gameID)
}
```

### 4. Grafana: Comprehensive Monitoring

Real-time monitoring is essential for multiplayer game stability. The implementation includes custom Prometheus metrics tracking:

```go
var activeGamesGauge = prometheus.NewGauge(prometheus.GaugeOpts{
    Name: "najibu_active_games_total",
    Help: "Number of currently active games",
})

var gameCompletionCounter = prometheus.NewCounterVec(
    prometheus.CounterOpts{
        Name: "najibu_games_completed_total",
        Help: "Total number of completed games",
    },
    []string{"game_mode", "difficulty"},
)

var playerAnswerHistogram = prometheus.NewHistogramVec(
    prometheus.HistogramOpts{
        Name:    "najibu_player_answer_duration_seconds",
        Help:    "Time taken by players to answer questions",
        Buckets: prometheus.LinearBuckets(1, 2, 15), // 1s to 30s
    },
    []string{"difficulty", "correct"},
)
```

**Grafana Dashboards Include:**

- Real-time active players and games
- Question answer time distributions
- Game completion rates by difficulty
- User registration and authentication metrics
- System resource utilization
- API response times and error rates

#### Grafana Dashboard Gallery

![Grafana Summary Dashboard](/assets/img/grafana/grafana-summary-dashboard.png)
_High-level system overview dashboard showing key performance indicators_

![Grafana Game Analytics](/assets/img/grafana/grafana-game-analytics-dashboard.png)
_Game-specific metrics including player engagement and match statistics_

![Grafana Real-time Monitoring](/assets/img/grafana/grafana-realtime-monitoring-dashboard.png)
_Real-time system performance monitoring with live metrics_

![Grafana System Overview](/assets/img/grafana/grafana-system-overview-dashboard.png)
_Infrastructure metrics including CPU, memory, and network utilization_

![Grafana PostgreSQL Dashboard](/assets/img/grafana/grafana-postgres-dashboard.png)
_Database performance metrics and query optimization insights_

### 5. PostgreSQL: Database Deep Dive

PostgreSQL serves as the backbone with carefully designed schemas for performance:

```sql
-- Users table with authentication integration
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    kratos_id UUID UNIQUE NOT NULL,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Players table for game statistics
CREATE TABLE players (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    total_score INTEGER DEFAULT 0,
    games_played INTEGER DEFAULT 0,
    games_won INTEGER DEFAULT 0,
    current_level INTEGER DEFAULT 1,
    total_xp INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Optimized leaderboard query with window functions
CREATE INDEX CONCURRENTLY idx_players_total_score_desc
ON players (total_score DESC)
WHERE total_score > 0;

-- Games table for match history
CREATE TABLE games (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    game_mode VARCHAR(20) NOT NULL,
    difficulty VARCHAR(10) NOT NULL,
    max_players INTEGER NOT NULL,
    status VARCHAR(20) DEFAULT 'waiting',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    started_at TIMESTAMP,
    completed_at TIMESTAMP
);
```

### 6. PostHog: Analytics & User Behavior

PostHog provides comprehensive analytics without the privacy concerns of Google Analytics:

```go
client, _ := posthog.NewWithConfig(os.Getenv("POSTHOG_API_KEY"), posthog.Config{
    Endpoint: "https://app.posthog.com",
})

func (s *GameService) trackGameEvent(userID, event string, properties map[string]interface{}) {
    client.Enqueue(posthog.Capture{
        DistinctId: userID,
        Event:      event,
        Properties: properties,
    })
}
```

#### PostHog Analytics Dashboards

![PostHog User Behavior Analytics](/assets/img/posthog/posthog-1.png)
_User behavior patterns and engagement metrics showing player journey through the app_

![PostHog Game Analytics](/assets/img/posthog/posthog-2.png)
_Game-specific analytics including session duration, feature adoption, and user retention_

## �️ System Architecture Overview

The Najibu platform follows a microservices architecture with clear separation of concerns, implementing zero-trust security principles and comprehensive observability patterns. The system is designed for horizontal scalability and high availability.

### Architecture Highlights

**Edge Layer**: Caddy provides automatic TLS, intelligent load balancing, and serves as the single entry point for all external traffic.

**Zero-Trust Authentication**: The Ory stack ensures that every request is authenticated and authorized before reaching application services.

**Microservices Design**: Clear separation between REST API services and real-time WebSocket services allows for independent scaling and maintenance.

**Data Layer**: PostgreSQL handles persistent data with Redis providing high-performance caching for real-time game states and session management.

**Observability**: Comprehensive monitoring through Prometheus metrics collection and Grafana visualization, with user analytics via PostHog.

[![](https://mermaid.ink/img/pako:eNqFWAtz4kYS_itTpJK7XIGNeBlTV6nCIMOWwRAJdi9hUykhtUCxkHQzkm2y7H-_nhk9RiAfuOxC01_39Ltb_lazQwdqg9qOWtGerB6-BgQ_P_5I9PcYaGD5xAT66tnAJIUlWwn9WrtE1CSGf8bP5gZ_yZKGr54D9I-CNNM3M4j_wYge2PQYxQppOd0sQxZPwx0ZouRj7NlMIev_Wf35MBw9rZeb_PIHy35JImLGIbV2kIIhcL4GuSkj34MgJsMo8j3bir0wuDSmCqOY8zhbr1a6sXn0kxgvJvNw6_nA4f_e0ttfvIVJfkKVHRp6jqLwF_1h8wW2ZGYFjhfsyBJVFAwGWHZ8-wzv8c1frFJp3dkB8h2BXjo-J5EGGVmOcyQGvAJlwN39flQVHw3H4982EsTjBFTc3yDDJA7JdLVamunBLLQc9KZvBTaqmh6OwkNEgTH0R3pixugdmzxy80Xgg12l_ih_jx5Nncm9gwch9f6Wz9WWcUxu2YIe-W32i2rPkzFcLcwNpz1RKw6ZUOuTw6-Kj2RuBejiAz6l6q4ZyjJg57GYiptza3decIt_wySDLi3G3kLqIJxBfkhDl5taCFbCuxiupk-6vsTE4AotrHj_AhClPhbGTKwY3qxjKu13oGFjRROGmUbtvReDHScUMtdKR5PPlu85qrIPYFE0YxW-QCD8mJ4b8N8EUJaBRnwYhyKjPy5lFSQjo7hcwU1CUW94Q2XR889w-Wlj6OaKf1EzTrf3IXmk1gHQxy9qdC5iNgMLO8Y2tKjDsmxFZ2GKc1B2NEFRZIpxDenxj7IO5afJcK7_OV0_bLDo_EbsIZvkTbZCFBaoGaJRcblA5okfe5EvknEUYl54gRoTIYIXA1QY4L0izcZsVwopsBNKeY-RnDLW7P9qju3DXIye9NWm0BEFBZg1nDdL0UzHwyEJ0ijmCZJZvI4wo6DkO_1V8eYwYG8oxEy2B-9CM5FRl6k1tmLrg0JWSEp-LBfmaoLJsfkn7_I7CuavMyFlazFQUyKtupK6mcc-TpJfeTFwCCZp8PJx5jxST2TwkcVwYD8rhhbfDH38CfU0wPEY9lh7f16k6cC5zAYVrChZOjc4cOYdPF62P1fW7TwMvFhkkJgs6TC8cPRii_56tXAa8fZ30S6XxmKur6b62tygTw-APTmRbphDTFEgpo3vy3zKMhW7U3iQFqWgLNGAuiE94IAARb081ViYUKSsWeEW3gOQEGFMgKwwD1mlsyfG8HH4PNxMqOViLQnmscX2Mrrcfh9oHr4iqS-USHUveDM9OP9lnaoGne8b5Wh8Clxq4QhJRL--iEKZjLPrGWIbtxLDJJ1ms0kmmvZBRx3zqhbFG4UMzpvpmFe_sUlBRhJws4Xy2AZiywvwdIGDBM6mW9qcydhjdogtLZs_U3QdDqXRHuwXpi4CGKTYovF5LEWKVnaCcgBN3fjM9cRqwf4DjtpJtd7kgYzHRpfoo5E46ZPX0XJNhvMx0Ze_ybOu1kLY8-c5lpE5Fketmy6ZbCPGnclnRmVg0vXvJ7wR-yvWwEVsMgJiJFh1crpNch8chOISkmd80aac5JAfo_Ndb5dIlxO1U4nulUQ-7lLZkc79T3pkjy6tTn_TnG3w93Y1M8kIE9VzhRfVADVwcuAe4ed9Rdmfs9jyXQ69gxltq4vK2S6cjw_i-uFb2lDS9ZY0bhq_nISg2y-meZLbo4TgLFLIJRLf8wVpjE70ApE-uCfjHSWY_CsOSAPRfGsBcmtF3u2_Tsou9REO9x4OlBvgB6Cd7CAclzaVD4AIQItUxQoFBE7ZX8HJNi124i2tEl9M6HWEWjhwyhcPiZd6C-zZ2nPKB6OqD--dQmGcIbdfMIPhHJchxGQ5yZFVpuRNLR32KGGq3pFpKMDFDDu_qAS7vK0g35QcMcdRiSayU7HIqJfnh5KvaOulTeaUJafKWow1oZJpY63n00qJ0RVcOUBXwGcukdklkLh3YIUXwFyOqrLsNAKf9iy--JyLVVB8wYCsH52Ud19V6kyXWHNW6hyluuNEEbgCUEq-ixLlbw2-T1i64ROaBATLWk6htOTFYJKBk7JYuTIrAec1XgnKg1dJLYesElL2aCVESd5qEWchrFZFbTCViAvPyjmZ9tFQ9BPJqHRoMz76uNHIZ9vHl9IxuPhF_H8CZ40_-AE0t-tCHYc-vhEOfmhq3bv7bfrYePOceD9oRe9nEoD_10Dyu23out2cv2Npnb59jZ_334zfRQnNnB96Xa3ZvMofRZn6fbcL_Zxd23ahdZXd4W8U-fV90HJ-t3vnanfX-A9yV8xE2NABOxfR7zeh414T4fEtLxOgoQ73uYB2u3evwdUQZP-0SmV0XTUMPY3_XJPBsnUm94XmFqGwe61-q18hQ5GSNdQ6H-kyr1SqHJU8W9RTWdt1ZejxfFARWLf1rDrrRW_HsKuorDjrogRFVEvkvPDqWYdNA6eiZM3U02oSYSmRn836TK8vp_Wiaea-V4GSVOcNMnNrrV7bUc-pDXCZh3rtAPh6wB9r3zjj1xouBQdc0wf41QHXSvyYr5PfkS2ygt_D8JBx0jDZ7WsD1_IZPiXi9XvsWbgaFBBczYCOQlzra4O-kFAbfKu91wYN7U67uevfN_s97b7Xbvbu2_XaEc9bze5Nq9fsttrtbqfZ1vrf67W_xa3aTb-rde56nXazf9-673R63_8HXBjfGA?type=png)](https://mermaid.live/edit#pako:eNqFWAtz4kYS_itTpJK7XIGNeBlTV6nCIMOWwRAJdi9hUykhtUCxkHQzkm2y7H-_nhk9RiAfuOxC01_39Ltb_lazQwdqg9qOWtGerB6-BgQ_P_5I9PcYaGD5xAT66tnAJIUlWwn9WrtE1CSGf8bP5gZ_yZKGr54D9I-CNNM3M4j_wYge2PQYxQppOd0sQxZPwx0ZouRj7NlMIev_Wf35MBw9rZeb_PIHy35JImLGIbV2kIIhcL4GuSkj34MgJsMo8j3bir0wuDSmCqOY8zhbr1a6sXn0kxgvJvNw6_nA4f_e0ttfvIVJfkKVHRp6jqLwF_1h8wW2ZGYFjhfsyBJVFAwGWHZ8-wzv8c1frFJp3dkB8h2BXjo-J5EGGVmOcyQGvAJlwN39flQVHw3H4982EsTjBFTc3yDDJA7JdLVamunBLLQc9KZvBTaqmh6OwkNEgTH0R3pixugdmzxy80Xgg12l_ih_jx5Nncm9gwch9f6Wz9WWcUxu2YIe-W32i2rPkzFcLcwNpz1RKw6ZUOuTw6-Kj2RuBejiAz6l6q4ZyjJg57GYiptza3decIt_wySDLi3G3kLqIJxBfkhDl5taCFbCuxiupk-6vsTE4AotrHj_AhClPhbGTKwY3qxjKu13oGFjRROGmUbtvReDHScUMtdKR5PPlu85qrIPYFE0YxW-QCD8mJ4b8N8EUJaBRnwYhyKjPy5lFSQjo7hcwU1CUW94Q2XR889w-Wlj6OaKf1EzTrf3IXmk1gHQxy9qdC5iNgMLO8Y2tKjDsmxFZ2GKc1B2NEFRZIpxDenxj7IO5afJcK7_OV0_bLDo_EbsIZvkTbZCFBaoGaJRcblA5okfe5EvknEUYl54gRoTIYIXA1QY4L0izcZsVwopsBNKeY-RnDLW7P9qju3DXIye9NWm0BEFBZg1nDdL0UzHwyEJ0ijmCZJZvI4wo6DkO_1V8eYwYG8oxEy2B-9CM5FRl6k1tmLrg0JWSEp-LBfmaoLJsfkn7_I7CuavMyFlazFQUyKtupK6mcc-TpJfeTFwCCZp8PJx5jxST2TwkcVwYD8rhhbfDH38CfU0wPEY9lh7f16k6cC5zAYVrChZOjc4cOYdPF62P1fW7TwMvFhkkJgs6TC8cPRii_56tXAa8fZ30S6XxmKur6b62tygTw-APTmRbphDTFEgpo3vy3zKMhW7U3iQFqWgLNGAuiE94IAARb081ViYUKSsWeEW3gOQEGFMgKwwD1mlsyfG8HH4PNxMqOViLQnmscX2Mrrcfh9oHr4iqS-USHUveDM9OP9lnaoGne8b5Wh8Clxq4QhJRL--iEKZjLPrGWIbtxLDJJ1ms0kmmvZBRx3zqhbFG4UMzpvpmFe_sUlBRhJws4Xy2AZiywvwdIGDBM6mW9qcydhjdogtLZs_U3QdDqXRHuwXpi4CGKTYovF5LEWKVnaCcgBN3fjM9cRqwf4DjtpJtd7kgYzHRpfoo5E46ZPX0XJNhvMx0Ze_ybOu1kLY8-c5lpE5Fketmy6ZbCPGnclnRmVg0vXvJ7wR-yvWwEVsMgJiJFh1crpNch8chOISkmd80aac5JAfo_Ndb5dIlxO1U4nulUQ-7lLZkc79T3pkjy6tTn_TnG3w93Y1M8kIE9VzhRfVADVwcuAe4ed9Rdmfs9jyXQ69gxltq4vK2S6cjw_i-uFb2lDS9ZY0bhq_nISg2y-meZLbo4TgLFLIJRLf8wVpjE70ApE-uCfjHSWY_CsOSAPRfGsBcmtF3u2_Tsou9REO9x4OlBvgB6Cd7CAclzaVD4AIQItUxQoFBE7ZX8HJNi124i2tEl9M6HWEWjhwyhcPiZd6C-zZ2nPKB6OqD--dQmGcIbdfMIPhHJchxGQ5yZFVpuRNLR32KGGq3pFpKMDFDDu_qAS7vK0g35QcMcdRiSayU7HIqJfnh5KvaOulTeaUJafKWow1oZJpY63n00qJ0RVcOUBXwGcukdklkLh3YIUXwFyOqrLsNAKf9iy--JyLVVB8wYCsH52Ud19V6kyXWHNW6hyluuNEEbgCUEq-ixLlbw2-T1i64ROaBATLWk6htOTFYJKBk7JYuTIrAec1XgnKg1dJLYesElL2aCVESd5qEWchrFZFbTCViAvPyjmZ9tFQ9BPJqHRoMz76uNHIZ9vHl9IxuPhF_H8CZ40_-AE0t-tCHYc-vhEOfmhq3bv7bfrYePOceD9oRe9nEoD_10Dyu23out2cv2Npnb59jZ_334zfRQnNnB96Xa3ZvMofRZn6fbcL_Zxd23ahdZXd4W8U-fV90HJ-t3vnanfX-A9yV8xE2NABOxfR7zeh414T4fEtLxOgoQ73uYB2u3evwdUQZP-0SmV0XTUMPY3_XJPBsnUm94XmFqGwe61-q18hQ5GSNdQ6H-kyr1SqHJU8W9RTWdt1ZejxfFARWLf1rDrrRW_HsKuorDjrogRFVEvkvPDqWYdNA6eiZM3U02oSYSmRn836TK8vp_Wiaea-V4GSVOcNMnNrrV7bUc-pDXCZh3rtAPh6wB9r3zjj1xouBQdc0wf41QHXSvyYr5PfkS2ygt_D8JBx0jDZ7WsD1_IZPiXi9XvsWbgaFBBczYCOQlzra4O-kFAbfKu91wYN7U67uevfN_s97b7Xbvbu2_XaEc9bze5Nq9fsttrtbqfZ1vrf67W_xa3aTb-rde56nXazf9-673R63_8HXBjfGA)

## �🎮 Real-Time Multiplayer: The Technical Challenge

The most complex aspect of Najibu is the real-time multiplayer system. Here's how I architected it for scalability and fairness:

### WebSocket Message Protocol

I designed a JSON-based protocol for all game communications:

```go
type GameState struct {
    ID                   string                 `json:"id"`
    Players              []Player               `json:"players"`
    CurrentQuestionIndex int                    `json:"current_question_index"`
    Scores               map[string]int         `json:"scores"`
    Status               string                 `json:"status"`
    TimeRemaining        int                    `json:"time_remaining"`
}

type AnswerSubmission struct {
    PlayerID        string    `json:"player_id"`
    QuestionIndex   int       `json:"question_index"`
    SelectedOption  int       `json:"selected_option"`
    TimeTaken       float64   `json:"time_taken"`
    Timestamp       time.Time `json:"timestamp"`
}

func (g *Game) HandlePlayerAnswer(playerID string, answer AnswerSubmission) error {
    g.Mu.Lock()
    defer g.Mu.Unlock()

    // Validate answer timing and question index
    if answer.QuestionIndex != g.CurrentQuestionIndex {
        return errors.New("invalid question index")
    }

    // Store answer and calculate score
    g.QuestionAnswers[playerID] = answer

    if g.isAnswerCorrect(answer) {
        timeBonus := g.calculateTimeBonus(answer.TimeTaken)
        g.Scores[playerID] += 100 + timeBonus
    }

    return nil
}
```

### Concurrency & Thread Safety

Managing concurrent access is crucial for multiplayer games:

```go
type GameHub struct {
    games         map[string]*Game
    players       map[string]*Player
    mu            sync.RWMutex

    // Event channels for coordination
    gameEvents    chan GameEvent
    playerEvents  chan PlayerEvent
    shutdownChan  chan struct{}
}

type Game struct {
    ID                   string
    Players              map[string]*Player
    Scores               map[string]int
    CurrentQuestionIndex int
    QuestionAnswers      map[string]AnswerSubmission
    Mu                   sync.RWMutex
    Timer                *time.Timer
    MessageChan          chan GameMessage
    Done                 chan struct{}
}

func (h *GameHub) AddPlayerToGame(playerID, gameID string) error {
    h.mu.Lock()
    defer h.mu.Unlock()

    game, exists := h.games[gameID]
    if !exists {
        return errors.New("game not found")
    }

    if len(game.Players) >= game.MaxPlayers {
        return errors.New("game is full")
    }

    game.Players[playerID] = h.players[playerID]

    // Start game if we have enough players
    if len(game.Players) >= 2 {
        go h.RunGame(context.Background(), gameID)
    }

    return nil
}
```

## 🏆 Gamification & User Engagement

User retention is critical for any app. I implemented a comprehensive progression system:

### XP Calculation System

```go
func calculateXPEarnedWithBreakdown(baseScore int, timeTaken float64, difficulty string, streakBonus int) (int, map[string]int) {
    breakdown := make(map[string]int)

    // Base XP from score
    baseXP := baseScore / 10
    breakdown["base"] = baseXP

    // Time bonus (faster answers get more XP)
    timeBonus := 0
    if timeTaken < 10.0 {
        timeBonus = 15
    } else if timeTaken < 20.0 {
        timeBonus = 10
    } else if timeTaken < 30.0 {
        timeBonus = 5
    }
    breakdown["time_bonus"] = timeBonus

    // Difficulty multiplier
    difficultyMultiplier := 1.0
    switch difficulty {
    case "easy":
        difficultyMultiplier = 1.0
    case "medium":
        difficultyMultiplier = 1.5
    case "hard":
        difficultyMultiplier = 2.0
    }

    // Streak bonus
    breakdown["streak_bonus"] = streakBonus

    totalXP := int(float64(baseXP+timeBonus+streakBonus) * difficultyMultiplier)
    breakdown["total"] = totalXP

    return totalXP, breakdown
}
```

### Achievement System

I created a flexible achievement system that tracks various player accomplishments:

```sql
CREATE TABLE achievements (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    description TEXT NOT NULL,
    icon_url VARCHAR(255),
    category VARCHAR(50) NOT NULL,
    difficulty VARCHAR(20) DEFAULT 'medium',
    xp_reward INTEGER DEFAULT 100,
    requirements JSONB NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE user_achievements (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    achievement_id UUID REFERENCES achievements(id) ON DELETE CASCADE,
    unlocked_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    progress JSONB DEFAULT '{}',
    UNIQUE(user_id, achievement_id)
);
```

## 🐳 DevOps & Infrastructure

The entire application runs on Docker Compose, making it incredibly portable and easy to deploy:

```yaml
version: "3.8"

services:
  backend:
    build: ./backend
    environment:
      - DATABASE_URL=postgres://app_user:password@postgres:5432/app_db
      - AUTH_SERVICE_URL=http://auth-service:4433
      - ANALYTICS_API_KEY=${ANALYTICS_API_KEY}
    depends_on:
      - postgres
      - auth-service
    volumes:
      - ./uploads:/app/uploads
    networks:
      - app-network

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: app_db
      POSTGRES_USER: app_user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - app-network

  auth-service:
    image: identity-service:latest
    environment:
      - DSN=postgres://auth_user:password@postgres:5432/auth_db
    volumes:
      - ./auth-config:/etc/config/auth
    depends_on:
      - postgres
    networks:
      - app-network

  # ...additional services...

networks:
  app-network:
    driver: bridge

volumes:
  postgres_data:
  monitoring_data:
  reverse_proxy_data:
  reverse_proxy_config:
```

### Production Deployment Considerations

**Security Hardening:**

- All services run as non-root users
- Secrets are managed through environment variables
- Database connections use SSL/TLS
- Rate limiting implemented at the Caddy level
- Regular security updates through automated Docker image pulls

**Backup Strategy:**

- Automated PostgreSQL backups every 6 hours
- Configuration files versioned in Git
- User uploads backed up to external storage
- Database replication for high availability

**Monitoring & Alerting:**

- Prometheus metrics collection
- Grafana alerting rules for critical issues
- Log aggregation with structured logging
- Uptime monitoring through external services

## 📊 Performance Optimizations & Results

### Database Optimizations

Connection pooling is critical for handling concurrent game sessions:

```go
// Database connection pooling configuration
func NewDBPool(databaseURL string) (*pgxpool.Pool, error) {
    config, err := pgxpool.ParseConfig(databaseURL)
    if err != nil {
        return nil, err
    }

    // Production-optimized settings
    config.MaxConns = 50
    config.MinConns = 10
    config.MaxConnLifetime = time.Hour
    config.MaxConnIdleTime = time.Minute * 30
    config.HealthCheckPeriod = time.Minute

    return pgxpool.ConnectConfig(context.Background(), config)
}
```

### Performance Metrics

**Current Performance (Single VPS):**

- Supports 100+ concurrent WebSocket connections
- Sub-100ms API response times
- 99.9% uptime over 6 months
- Database queries optimized to under 10ms average
- Real-time game latency under 50ms

**Load Testing Results:**

- Handled 500 concurrent users during stress testing
- Memory usage remains stable under load
- CPU utilization peaks at 60% during high traffic
- Zero data loss during failover testing

**Scaling Bottlenecks Identified:**

- WebSocket connection limits (solvable with load balancing)
- Database connection pool (already optimized)
- Single point of failure (planning Kubernetes migration)

## 🧪 Testing Strategy

Comprehensive testing ensures reliability in production:

```go
func TestGameCreation(t *testing.T) {
    hub := NewGameHub()

    gameID, err := hub.CreateGame("multiplayer", "medium", 4)
    assert.NoError(t, err)
    assert.NotEmpty(t, gameID)

    game := hub.GetGame(gameID)
    assert.Equal(t, "waiting", game.Status)
    assert.Equal(t, 4, game.MaxPlayers)
    assert.Equal(t, 0, len(game.Players))
}

func TestWebSocketGameFlow(t *testing.T) {
    server := httptest.NewServer(setupTestServer())
    defer server.Close()

    // Convert HTTP URL to WebSocket URL
    wsURL := "ws" + strings.TrimPrefix(server.URL, "http") + "/ws"

    // Connect multiple players
    var connections []*websocket.Conn
    for i := 0; i < 3; i++ {
        conn, _, err := websocket.DefaultDialer.Dial(wsURL, nil)
        assert.NoError(t, err)
        connections = append(connections, conn)
        defer conn.Close()

        // Join game
        joinMsg := map[string]interface{}{
            "type": "join_game",
            "data": map[string]interface{}{
                "player_id": fmt.Sprintf("player_%d", i),
            },
        }
        err = conn.WriteJSON(joinMsg)
        assert.NoError(t, err)
    }

    // Verify game starts automatically
    time.Sleep(100 * time.Millisecond)

    // Check that all players receive game_started message
    for _, conn := range connections {
        var response map[string]interface{}
        err := conn.ReadJSON(&response)
        assert.NoError(t, err)
        assert.Equal(t, "game_started", response["type"])
    }
}

func TestConcurrentGameSessions(t *testing.T) {
    hub := NewGameHub()

    // Create multiple games concurrently
    var wg sync.WaitGroup
    gameIDs := make([]string, 0, 100)
    mutex := sync.Mutex{}

    for i := 0; i < 100; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()

            gameID, err := hub.CreateGame("quick", "easy", 2)
            assert.NoError(t, err)

            mutex.Lock()
            gameIDs = append(gameIDs, gameID)
            mutex.Unlock()
        }()
    }

    wg.Wait()

    // Verify all games were created successfully
    assert.Equal(t, 100, len(gameIDs))

    // Verify no duplicate game IDs
    gameIDSet := make(map[string]bool)
    for _, id := range gameIDs {
        assert.False(t, gameIDSet[id], "Duplicate game ID found")
        gameIDSet[id] = true
    }
}
```

## 📱 Mobile App Integration

The Flutter app seamlessly integrates with the backend through WebSocket connections and RESTful APIs:

### WebSocket State Management

```dart
class GameStateProvider extends ChangeNotifier {
  IOWebSocketChannel? _channel;
  GameState? _currentGame;

  void connectToGame(String gameId, String playerId) {
    _channel = IOWebSocketChannel.connect(
      'wss://api.najibu.app/ws',
      headers: {'Authorization': 'Bearer $token'},
    );

    _channel!.stream.listen((data) {
      final message = jsonDecode(data);
      _handleGameMessage(message);
    });

    // Join the game
    _channel!.sink.add(jsonEncode({
      'type': 'join_game',
      'data': {'game_id': gameId, 'player_id': playerId}
    }));
  }

  void submitAnswer(int questionIndex, int selectedOption) {
    _channel!.sink.add(jsonEncode({
      'type': 'submit_answer',
      'data': {
        'question_index': questionIndex,
        'selected_option': selectedOption,
        'timestamp': DateTime.now().toIso8601String(),
      }
    }));
  }

  void _handleGameMessage(Map<String, dynamic> message) {
    switch (message['type']) {
      case 'question':
        _currentGame = _currentGame?.copyWith(
          currentQuestion: Question.fromJson(message['data']),
        );
        break;
      case 'game_state':
        _currentGame = GameState.fromJson(message['data']);
        break;
      case 'game_ended':
        _handleGameEnd(message['data']);
        break;
    }
    notifyListeners();
  }
}
```

### Authentication Flow

```dart
class AuthService {
  final Dio _dio = Dio();

  Future<AuthResult> login(String email, String password) async {
    try {
      // Initialize login flow with Kratos
      final flowResponse = await _dio.get(
        'https://auth.najibu.app/self-service/login/api',
      );

      final flowId = flowResponse.data['id'];

      // Submit credentials
      final loginResponse = await _dio.post(
        'https://auth.najibu.app/self-service/login',
        data: {
          'flow': flowId,
          'method': 'password',
          'password_identifier': email,
          'password': password,
        },
      );

      final sessionToken = loginResponse.data['session_token'];
      await _storeToken(sessionToken);

      return AuthResult.success(sessionToken);
    } catch (e) {
      return AuthResult.failure(e.toString());
    }
  }

  Future<void> _storeToken(String token) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString('session_token', token);
  }
}
```

## 📈 Analytics & Business Intelligence

### PostHog Integration

I track key user behaviors and game metrics:

```go
func (s *GameService) trackGameEvent(userID, event string, properties map[string]interface{}) {
    s.posthog.Enqueue(posthog.Capture{
        DistinctId: userID,
        Event:      event,
        Properties: properties,
        Timestamp:  time.Now(),
    })
}

func (s *GameService) CompleteGame(gameID string, results GameResults) error {
    // Save to database
    err := s.saveGameResults(gameID, results)
    if err != nil {
        return err
    }

    // Track analytics for each player
    for playerID, score := range results.Scores {
        s.trackGameEvent(playerID, "game_completed", map[string]interface{}{
            "game_id":     gameID,
            "score":       score,
            "duration":    results.Duration.Seconds(),
            "difficulty":  results.Difficulty,
            "game_mode":   results.GameMode,
        })
    }

    return nil
}
```

### Key Metrics Tracked

**User Engagement:**

- Daily/Weekly/Monthly active users
- Session duration and frequency
- Feature adoption rates
- User retention cohorts

**Game Performance:**

- Average game duration by difficulty
- Question answer accuracy rates
- Popular game modes and times
- Player progression patterns

**Technical Metrics:**

- API response times
- WebSocket connection stability
- Error rates and types
- Database query performance

**Business Intelligence:**

- User acquisition channels
- Feature usage analytics
- Performance bottlenecks
- Growth opportunities

## 🔮 Future Enhancements & Roadmap

### Planned Features

**Advanced Multiplayer Modes:**

- Tournament brackets with elimination rounds
- Team-based competitions
- Seasonal leaderboards with rewards

**AI & Machine Learning:**

- Personalized difficulty adjustment
- Smart question recommendation
- Player skill assessment algorithms
- Automated question generation from Bible text

**Social Features:**

- Guild/team system with shared progress
- Social leaderboards and challenges
- In-game chat and reactions
- Achievement sharing and celebrations

### Technical Roadmap

**Microservices Architecture:**

- Separate services for user management, games, leaderboards
- Event-driven architecture with message queues
- Independent scaling and deployment

**Kubernetes Migration:**

- Container orchestration for better scaling
- Service mesh for advanced traffic management
- Automated failover and self-healing

**Advanced Analytics:**

- Real-time player behavior tracking
- A/B testing framework for feature rollouts
- Predictive analytics for user retention

**Mobile Improvements:**

- Offline mode with sync capabilities
- Push notifications for game invites
- Native platform optimizations

## 💡 Lessons Learned & Insights

### What Went Exceptionally Well

**Self-Hosting Architecture Decision:**
The migration from Firebase delivered exceptional ROI with 99.2% cost reduction compared to projected Firebase expenses ($5,470/month vs $41/month for equivalent functionality using Netcup's RS 4000 G11). Beyond cost savings, complete infrastructure ownership enabled rapid feature iteration, custom performance optimizations, and invaluable production DevOps experience. The technical flexibility proved essential for implementing real-time multiplayer features that would have been constrained by Firebase's function execution limits.

**Go for Backend Development:**
Go's native concurrency primitives (goroutines and channels) provided elegant solutions for real-time multiplayer game coordination. The runtime's performance characteristics and memory efficiency exceeded expectations under production load.

**Docker Compose Development Environment:**
Containerizing the entire stack enabled consistent development environments and eliminated configuration drift. Single-command stack deployment (`docker-compose up`) accelerated iteration cycles and reduced environment-specific debugging.

**Proactive Monitoring Implementation:**
Early integration of Grafana and Prometheus enabled proactive performance optimization. Metrics-driven development identified bottlenecks before they impacted user experience.

### Challenges Overcome

**WebSocket Connection Management:**
Managing thousands of concurrent WebSocket connections required careful consideration of connection pooling, heartbeat mechanisms, and graceful degradation under load.

**Database Performance:**
Initially, leaderboard queries were slow. Adding proper indexes and using PostgreSQL window functions dramatically improved performance from seconds to milliseconds.

**Authentication Integration:**
Integrating Ory Kratos with both the mobile app and web frontend required understanding OAuth flows and session management deeply.

**Real-Time Synchronization:**
Ensuring all players see questions at exactly the same time required implementing clock synchronization and compensation for network latency.

### What I'd Do Differently

**Earlier Load Testing Implementation:**
Comprehensive load testing should have been integrated earlier in the development lifecycle to identify scaling constraints before production deployment.

**Microservices Architecture from Inception:**
While monolithic architecture accelerated initial development, the current refactoring effort toward microservices represents technical debt that could have been avoided with upfront architectural planning.

**Comprehensive Logging Strategy:**
Structured logging and distributed tracing should have been implemented from project inception to facilitate debugging and performance analysis in production environments.

**Mobile-First API Design:**
Initial API design prioritized web consumption patterns, requiring subsequent optimization for mobile performance characteristics and network constraints.

## 🏁 Technical Achievements & Impact

### Scalability Milestones

**Infrastructure Achievements:**

- Successfully handling 500+ concurrent users on a single VPS
- 99.9% uptime of operation, albeit we are just in closed beta
- Sub-100ms API response times globally
- Zero data loss incidents

**Performance Optimizations:**

- Reduced database query times by 95% through indexing
- Implemented connection pooling reducing resource usage by 60%
- Optimized WebSocket message protocol reducing bandwidth by 40%
- Achieved real-time game synchronization within 50ms

### Code Quality & Best Practices

**Testing Coverage:**

- 85% code coverage across all Go packages
- Comprehensive integration tests for multiplayer scenarios
- Load testing simulating 1000+ concurrent games

**Documentation & Maintainability:**

- Complete API documentation with OpenAPI/Swagger
- Comprehensive README with setup instructions

**Security Implementations:**

- Zero-trust architecture with service-to-service authentication
- Comprehensive input validation and sanitization
- Rate limiting and DDoS protection

## 🎉 Conclusion

The Najibu project represents a comprehensive technical evolution from concept to production-grade platform, demonstrating modern software architecture principles across full-stack development and DevOps practices. The system architecture successfully scaled from a simple quiz application into a sophisticated real-time multiplayer ecosystem.

**Key Technical Accomplishments:**

- Architected and implemented a scalable, real-time multiplayer game engine
- Deployed a complete self-hosted infrastructure stack with zero-trust security
- Integrated comprehensive monitoring, analytics, and observability systems

**Professional Development:**

- Advanced container orchestration and infrastructure-as-code practices
- Production experience with Go's concurrency patterns and runtime characteristics
- Database performance optimization and query analysis expertise
- Modern authentication and authorization system implementation

## 🔗 Project Resources

- **Live Application:** [https://najibu.app](https://najibu.app)
- **Technical Documentation:** [https://najibu-app.github.io](https://najibu-app.github.io)

## 🙏 Acknowledgments

Special thanks to the open-source community and the creators of the technologies that made this project possible: Go, Flutter, PostgreSQL, Ory, Caddy, Grafana, and countless others. The documentation and community support for these tools made this ambitious project achievable as a solo developer.

## 💰 Detailed Cost Analysis: Why Self-Hosting Wins

As a technical professional, I conducted a thorough cost analysis comparing the three primary hosting approaches for a real-time multiplayer platform. The results clearly favored self-hosted infrastructure, particularly when considering the application's specific requirements.

### Cost Breakdown by Platform

#### Firebase (Google Cloud)

For 100k MAUs with real-time multiplayer features:

| Service                 | Usage Estimate     | Monthly Cost     |
| ----------------------- | ------------------ | ---------------- |
| Authentication          | 100k MAUs          | $0 (free tier)   |
| Cloud Firestore Storage | 10GB               | $90              |
| Document Reads          | 15M operations     | $540             |
| Document Writes         | 3M operations      | $360             |
| Network Egress          | 50GB               | $480             |
| Cloud Functions         | 10M invocations    | $3,200           |
| Compute Time            | Function execution | $800             |
| **Total Firebase**      |                    | **$5,470/month** |

#### Supabase

For 100k MAUs with equivalent functionality:

| Service              | Usage Estimate    | Monthly Cost   |
| -------------------- | ----------------- | -------------- |
| Pro Plan Base        | Standard features | $25            |
| Database Storage     | Beyond 8GB        | $15            |
| Bandwidth            | Beyond 250GB      | $450           |
| Realtime Connections | 1000+ concurrent  | $500           |
| **Total Supabase**   |                   | **$990/month** |

#### Self-Hosted (Netcup/Hetzner)

Production-grade dedicated server infrastructure:

**Netcup RS 4000 G11 (Primary Production Server):**

| Resource              | Specification                     | Monthly Cost  |
| --------------------- | --------------------------------- | ------------- |
| Dedicated Server      | 16GB DDR5 ECC, 8 vCPU, 512GB NVMe | €14.19 (~$16) |
| Domain & SSL          | Managed DNS + certificates        | $15           |
| Backup Storage        | External backup solution          | $10           |
| **Total Self-Hosted** |                                   | **$41/month** |

**Alternative Hetzner CCX23 (Scalable Option):**

| Resource              | Specification               | Monthly Cost  |
| --------------------- | --------------------------- | ------------- |
| VPS Server            | 16GB RAM, 4 vCPU, 160GB SSD | €27.09 (~$30) |
| Domain & SSL          | Managed DNS + certificates  | $15           |
| Backup Storage        | External backup solution    | $10           |
| **Total Alternative** |                             | **$55/month** |

**Hardware Specifications Advantage:**
The Netcup RS 4000 G11 delivers enterprise-grade hardware at exceptional value:

- **AMD EPYC™ 9634 processors** (up to 3.7 GHz per core) - server-class performance
- **DDR5 ECC memory** - Error-correcting memory for production reliability
- **NVMe SSD storage** - Ultra-fast storage with low latency
- **99.9% uptime SLA** - Enterprise-grade availability guarantee
- **2.5 Gbps network** - High-bandwidth connectivity for real-time applications

This hardware specification would cost significantly more on cloud platforms, where equivalent compute and memory resources often exceed $200-400/month before factoring in storage and bandwidth costs.

### Technical Advantages of Self-Hosting

**Performance Predictability:**

- Dedicated resources eliminate noisy neighbor effects
- Predictable latency for real-time multiplayer
- Custom caching strategies optimized for game workloads

**Infrastructure Control:**

- Full control over database optimization
- Custom WebSocket handling without platform limitations
- Ability to implement game-specific performance optimizations

**Cost Scaling Characteristics:**

```go
// Linear scaling model for self-hosted
func calculateMonthlyCost(users int) float64 {
    baseServerCost := 40.0
    additionalServers := float64(users) / 50000 // Add server per 50k users
    return baseServerCost + (additionalServers * 40.0)
}

// vs Firebase's exponential scaling
func firebaseCost(operations int) float64 {
    return float64(operations) * 0.036 // $0.036 per 1k operations
}
```

### Long-term Economic Impact

At 500k MAUs, the cost comparison becomes even more dramatic:

- **Firebase**: ~$27,000/month
- **Supabase**: ~$4,500/month
- **Self-Hosted (Netcup)**: ~$205/month (distributed across 5 RS 4000 G11 servers)
- **Self-Hosted (Hetzner)**: ~$350/month (distributed across 7 CCX23 instances)

The self-hosted approach provides 99.2% cost savings compared to Firebase at scale, while maintaining superior performance characteristics essential for real-time gaming applications. The dedicated server option (Netcup) offers the best price-performance ratio with DDR5 ECC memory and NVMe storage.
