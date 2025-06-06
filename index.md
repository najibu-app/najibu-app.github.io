---
layout: default
title: "Building Najibu: A Self-Hosted Bible Quiz App with Real-Time Multiplayer"
description: "A comprehensive technical showcase of building a full-stack, self-hosted Bible trivia platform with real-time multiplayer gameplay, complete DevOps pipeline, and scalable architecture."
author: Alfred Wayne Kinyua
featured_image: /assets/images/najibu-hero.png
---

# Building Najibu: From Firebase to Full Self-Hosted Infrastructure

## Who I Am

I'm Alfred Wayne Kinyua, a Software Engineer specializing in scalable application development and DevOps engineering. You can connect with me on [LinkedIn](https://www.linkedin.com/in/alfred-wayne-kinyua/) where I document my technical journey. As the sole architect and developer of **Najibu**, I designed and implemented this comprehensive Bible trivia platform from ground zero.

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
    reverse_proxy oathkeeper:4455
}

auth.najibu.app {
    reverse_proxy kratos:4433
}

najibu.app {
    reverse_proxy najibu-landing:3000
}

grafana.najibu.app {
    reverse_proxy grafana:3000
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
# kratos.yml
version: v0.11.1
dsn: postgres://kratos:password@postgres:5432/kratos
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
# oathkeeper-rules.yml
- id: "api-protected"
  match:
    url: "https://api.najibu.app/api/v1/<{user,games,leaderboards,achievements,friends}/**>"
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
    url: "http://najibu-go:8080"

- id: "api-public"
  upstream:
    preserve_host: true
    url: "http://najibu-go:8080"
  match:
    url: "https://api.najibu.app/api/v1/health"
    methods:
      - "GET"
      - "POST"
      - "PUT"
      - "DELETE"
      - "PATCH"
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
