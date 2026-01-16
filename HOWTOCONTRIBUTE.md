# How to Contribute

This guide explains how to extend the Area project with new services, actions, reactions, and OAuth providers.

## Table of Contents

- [Project Architecture Overview](#project-architecture-overview)
- [Adding a New Service](#adding-a-new-service)
- [Adding a New Action](#adding-a-new-action)
- [Adding a New Reaction](#adding-a-new-reaction)
- [Adding a New OAuth Provider](#adding-a-new-oauth-provider)
- [Code Conventions](#code-conventions)
- [Testing Your Changes](#testing-your-changes)

## Project Architecture Overview

Understanding the codebase structure is essential before contributing:

```
back/
├── action/           # Action handlers (event detection logic)
├── reaction/         # Reaction handlers (response execution logic)
├── authentification/ # OAuth2 provider implementations
├── templates/        # Service, action, and reaction definitions
├── routine/          # Polling routine (checks actions every 5 seconds)
├── engine/           # Listener (executes reactions when actions trigger)
├── controller/       # API endpoint handlers
├── router/           # Route definitions
├── middleware/       # Authentication middleware
├── model/            # Data structures
├── seed/             # Database seeding
└── schema.prisma     # Database schema
```

### How the Engine Works

1. **Routine** (`routine/routine.go`) runs every 5 seconds:
   - Fetches all actions from the database
   - Calls each action's handler with config data
   - If condition is met, handler sets `action.triggered = true`

2. **Listener** (`engine/listener.go`) runs every 5 seconds:
   - Fetches all enabled areas with their actions and reactions
   - If `action.triggered == true`, executes the reaction handler
   - Resets `action.triggered = false` after execution

---

## Adding a New Service

A service represents an external platform (e.g., Spotify, Slack, Twitter).

### Step 1: Add to Database Seed

Edit `back/seed/main.go`:

```go
func create_spotify_seed(client *db.PrismaClient) {
    ctx := context.Background()

    _, err := client.Services.UpsertOne(
        db.Services.ID.Equals(7), // Use next available ID
    ).Create(
        db.Services.Name.Set("Spotify"),
    ).Update(
        db.Services.Name.Set("Spotify"),
    ).Exec(ctx)

    if err != nil {
        log.Fatalf("Error while seeding: %s", err)
    }
}

func main() {
    // ... existing seeds
    create_spotify_seed(client)
}
```

### Step 2: Register in Templates

Edit `back/templates/config.go` to add your service:

```go
var Services = map[string]*Service{
    // ... existing services

    "Spotify": {
        Name:      "Spotify",
        Actions:   map[string]*ActionDefinition{},
        Reactions: map[string]*ReactionDefinition{},
    },
}
```

### Step 3: Add Environment Variables

Add required API keys to `.env`:

```env
SPOTIFY_CLIENT_ID=your-client-id
SPOTIFY_CLIENT_SECRET=your-client-secret
```

---

## Adding a New Action

Actions detect events from external services. When the condition is met, set `triggered = true`.

### Step 1: Create Action Handler File

Create a new file in `back/action/`, e.g., `spotifyNewTrack.go`:


### Step 2: Register Action in Templates

Edit `back/templates/config.go`:

```go
"Spotify": {
    Name: "Spotify",
    Actions: map[string]*ActionDefinition{
        "spotify_new_track": {
            Name:        "spotify_new_track",
            Description: "New track saved to library",
            Service:     "Spotify",
            Config: []ActionField{
                {
                    Name:     "playlist_id",
                    Type:     "text",
                    Label:    "Playlist ID (optional)",
                    Required: false,
                },
            },
            Handler: action.ExecSpotifyNewTrack,
        },
    },
    Reactions: map[string]*ReactionDefinition{},
},
```

### Step 3: Import the Action Package

Ensure the import in `back/templates/config.go`:

```go
import (
    "github.com/ValianceTekProject/AreaBack/action"
    "github.com/ValianceTekProject/AreaBack/reaction"
)
```

---

## Adding a New Reaction

Reactions execute responses when an action is triggered.

### Step 1: Create Reaction Handler File

Create a new file in `back/reaction/`, e.g., `spotifyAddToPlaylist.go`:

### Step 2: Register Reaction in Templates

Edit `back/templates/config.go`:

```go
"Spotify": {
    Name: "Spotify",
    Actions: map[string]*ActionDefinition{
        // ... existing actions
    },
    Reactions: map[string]*ReactionDefinition{
        "spotify_add_to_playlist": {
            Name:        "spotify_add_to_playlist",
            Description: "Add track to a playlist",
            Service:     "Spotify",
            Config: []ReactionField{
                {
                    Name:     "playlist_id",
                    Type:     "text",
                    Label:    "Playlist ID",
                    Required: true,
                },
                {
                    Name:     "track_uri",
                    Type:     "text",
                    Label:    "Track URI",
                    Required: true,
                },
            },
            Handler: reaction.AddToSpotifyPlaylist,
        },
    },
},
```

---

## Adding a New OAuth Provider

OAuth providers allow users to connect external services to their Area account.

### Step 1: Add Environment Variables

Add to `.env`:

```env
SPOTIFY_CLIENT_ID=your-client-id
SPOTIFY_CLIENT_SECRET=your-client-secret
SPOTIFY_REDIRECT_URL=http://localhost:8080/auth/spotify/callback
```

### Step 2: Create OAuth Handler

Create `back/authentification/spotifyAuth.go`:

### Step 3: Register Routes

Edit `back/router/router.go`:

```go
func setupAuthRouter(router *gin.Engine) *gin.Engine {
    oauthRoute := router.Group("/")
    oauthRoute.Use(middleware.VerifyOauthUser)
    {
        oauthRoute.GET("/auth/google/login", authentification.GoogleLogin)
        oauthRoute.GET("/auth/github/login", authentification.GithubLogin)
        oauthRoute.GET("/auth/discord/login", authentification.DiscordLogin)
        oauthRoute.GET("/auth/spotify/login", authentification.SpotifyLogin)  // Add this
    }

    // ... other routes

    router.GET("/auth/spotify/callback", authentification.SpotifyCallback)  // Add this

    return router
}
```

---

## Code Conventions

### File Naming

- Action handlers: `back/action/<serviceName><ActionName>.go` (e.g., `spotifyNewTrack.go`)
- Reaction handlers: `back/reaction/<serviceName><ReactionName>.go` (e.g., `spotifyAddPlaylist.go`)
- OAuth handlers: `back/authentification/<serviceName>Auth.go` (e.g., `spotifyAuth.go`)

### Function Naming

- Action handlers: `Exec<ServiceName><ActionName>` (e.g., `ExecSpotifyNewTrack`)
- Reaction handlers: `<ActionVerb><ServiceName><Object>` (e.g., `AddToSpotifyPlaylist`)
- Token getters: `Get<ServiceName>Token` (e.g., `GetSpotifyToken`)

### Error Handling

- Always check for required config values
- Log errors but don't crash the routine
- Return errors for critical failures

---

## Testing Your Changes

### 1. Run Database Migrations

```bash
cd back
go run github.com/steebchen/prisma-client-go db push
```

### 2. Run Database Seed

```bash
cd back
go run seed/main.go
```

### 3. Start the Backend

```bash
cd back
go run main.go
```

### 4. Test via API

**Create an Area:**
```bash
curl -X POST http://localhost:8080/areas/create \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"name": "Test Spotify Area"}'
```

**Link an Action:**
```bash
curl -X POST http://localhost:8080/areas/<areaId>/action/add \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"name": "spotify_new_track", "service_name": "Spotify"}'
```

**Link a Reaction:**
```bash
curl -X POST http://localhost:8080/areas/<areaId>/reaction/add \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"name": "discord_send_message", "service_name": "Discord"}'
```

### 5. Verify in about.json

Check that your new service/actions/reactions appear:

```bash
curl http://localhost:8080/about.json | jq
```

---

## Checklist for New Contributions

### Adding a Service
- [ ] Add seed function in `seed/main.go`
- [ ] Register service in `templates/config.go`
- [ ] Add environment variables to `.env` and `env_template`

### Adding an Action
- [ ] Create handler file in `back/action/`
- [ ] Implement handler following the pattern (get action_id, check condition, set triggered)
- [ ] Register in `templates/config.go` with ActionDefinition
- [ ] Add required Config fields

### Adding a Reaction
- [ ] Create handler file in `back/reaction/`
- [ ] Implement handler following the pattern (get reaction_id, execute action)
- [ ] Register in `templates/config.go` with ReactionDefinition
- [ ] Add required Config fields

### Adding an OAuth Provider
- [ ] Create auth handler in `back/authentification/`
- [ ] Implement Login and Callback functions
- [ ] Register routes in `back/router/router.go`
- [ ] Add environment variables
- [ ] Add seed for the service
