# Area - Automation Platform

Area is an automation platform that connects multiple web services to create automated workflows following the **Action-Reaction** pattern. When an event (Action) is detected on one service, it triggers a response (Reaction) on another service.

## Table of Contents

- [Features](#features)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Running the Project](#running-the-project)
- [API Reference](#api-reference)
- [Available Services](#available-services)
- [Database Schema](#database-schema)

## Features

- **Multi-service integration**: Connect GitHub, Discord, Google, Steam, Twitch, and OpenWeather
- **OAuth2 authentication**: Login with Google, GitHub, or Discord
- **Custom automation workflows**: Create areas with configurable actions and reactions
- **Real-time monitoring**: Engine polls every 5 seconds for triggered actions
- **Web and mobile clients**: Next.js web app and React Native mobile app

## Architecture

```
Area/
├── back/                    # Go backend API (port 8080)
├── front/                   # Next.js frontend (port 8081)
└── mobile/                  # React Native mobile app
```

## Prerequisites

- **Docker** and **Docker Compose** (recommended)
- Or for manual setup:
  - Go 1.21+
  - Node.js 18+
  - PostgreSQL 13+

## Installation

### Using Docker (Recommended)

1. Clone the repository:
```bash
git clone <repository-url>
cd Area
```

2. Copy the environment template and configure it:
```bash
cp env_template .env
```

3. Edit `.env` with your API credentials (see [Configuration](#configuration))

4. Start all services:
```bash
docker-compose up --build
```

### Manual Installation

**Backend:**
```bash
cd back
go mod download
go mod tidy
```

**Frontend:**
```bash
cd front
npm install
```

**Mobile:**
```bash
cd mobile
npm install
```

## Configuration

Create a `.env` file in the project root with the following variables:

```env
# Database
DATABASE_URL=postgresql://dev:dev@db:5432/area?schema=public

# Google OAuth
GOOGLE_CLIENT_ID=your-google-client-id.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=your-google-client-secret
GOOGLE_REDIRECT_URL=http://localhost:8080/auth/google/callback

# GitHub OAuth
GH_BASIC_CLIENT_ID=your-github-client-id
GH_BASIC_SECRET_ID=your-github-client-secret

# Discord OAuth
DISCORD_CLIENT_ID=your-discord-client-id
DISCORD_CLIENT_SECRET=your-discord-client-secret
DISCORD_REDIRECT_URL=http://localhost:8080/auth/discord/callback
DISCORD_BOT_TOKEN=your-discord-bot-token

# JWT
JWT_SECRET=your-jwt-secret-key

# Optional: External APIs
STEAM_API_KEY=your-steam-api-key
TWITCH_CLIENT_ID=your-twitch-client-id
TWITCH_CLIENT_SECRET=your-twitch-client-secret
OPENWEATHER_API_KEY=your-openweather-api-key
```

### OAuth Setup

**Google:**
1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project and enable Gmail, Drive, Sheets, and Calendar APIs
3. Create OAuth 2.0 credentials with redirect URI `http://localhost:8080/auth/google/callback`

**GitHub:**
1. Go to GitHub Settings > Developer settings > OAuth Apps
2. Create a new app with callback URL `http://localhost:8080/auth/github/callback`

**Discord:**
1. Go to [Discord Developer Portal](https://discord.com/developers/applications)
2. Create a new application and add a bot
3. Add redirect URI `http://localhost:8080/auth/discord/callback`

## Running the Project

### Using Docker

```bash
docker-compose up
```

Services will be available at:
- **Backend API**: http://localhost:8080
- **Frontend**: http://localhost:8081
- **Database**: localhost:5432

### Manual Setup

**Start PostgreSQL** (if not using Docker)

**Backend:**
```bash
cd back
go run main.go
```

**Frontend:**
```bash
cd front
npm run dev
```

**Mobile:**
```bash
cd mobile
npm start
# Then: npm run android OR npm run ios
```

## API Reference

Base URL: `http://localhost:8080`

### Authentication

All protected endpoints require a JWT token in the `Authorization` header:
```
Authorization: Bearer <token>
```

#### Public Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/auth/register` | Register a new user |
| `POST` | `/auth/login` | Login with email/password |
| `GET` | `/auth/google/login` | Initiate Google OAuth |
| `GET` | `/auth/google/callback` | Google OAuth callback |
| `GET` | `/auth/github/login` | Initiate GitHub OAuth |
| `GET` | `/auth/github/callback` | GitHub OAuth callback |
| `GET` | `/auth/discord/login` | Initiate Discord OAuth |
| `GET` | `/auth/discord/callback` | Discord OAuth callback |
| `GET` | `/about.json` | Get available services and actions/reactions |
| `GET` | `/client.apk` | Download mobile APK |

#### Protected Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/me/userId` | Get current user's ID |
| `GET` | `/areas` | Get all user's areas |
| `POST` | `/areas/create` | Create a new area |
| `PATCH` | `/areas/:areaId/status` | Enable/disable an area |
| `DELETE` | `/areas/:areaId/delete` | Delete an area |
| `POST` | `/areas/:areaId/action/add` | Link an action to an area |
| `POST` | `/areas/:areaId/reaction/add` | Link a reaction to an area |
| `GET` | `/services/:userId` | Get user's connected services |
| `GET` | `/users/:userId` | Get specific user profile |

#### Admin Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/users` | List all users |
| `PATCH` | `/users/:userId/status` | Enable/disable user access |

### Request/Response Examples

**Register User**
```bash
POST /auth/register
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "securepassword"
}
```

Response:
```json
{
  "message": "Login successful",
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "email": "user@example.com"
  }
}
```

**Create Area**
```bash
POST /areas/create
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "My Automation"
}
```

**Link Action to Area**
```bash
POST /areas/:areaId/action/add
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "github_pr_merged",
  "service_name": "Github"
}
```

**Link Reaction to Area**
```bash
POST /areas/:areaId/reaction/add
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "discord_send_message",
  "service_name": "Discord"
}
```

**Update Area Status**
```bash
PATCH /areas/:areaId/status
Authorization: Bearer <token>
Content-Type: application/json

{
  "is_enabled": true
}
```

## Available Services

### GitHub

**Actions:**
| Name | Description | Required Config |
|------|-------------|-----------------|
| `github_pr_merged` | PR closed on repository | `owner`, `repo` |
| `github_new_PR` | New PR created | `owner`, `repo` |

**Reactions:**
| Name | Description |
|------|-------------|
| `github_create_pr` | Create PR in TestArea repository |
| `github_create_team` | Create team in organization |

### Discord

**Actions:**
| Name | Description | Required Config |
|------|-------------|-----------------|
| `discord_message_received` | Message received in channel | `channel_id` |

**Reactions:**
| Name | Description | Required Config |
|------|-------------|-----------------|
| `discord_send_message` | Send message to channel | `channel_id`, `content` |
| `discord_add_emoji` | Add emoji to last message | `channel_id`, `content` |

### Google

**Actions:**
| Name | Description |
|------|-------------|
| `google_new_file_drive` | New file created in Drive |
| `google_new_mail` | New email received |

**Reactions:**
| Name | Description | Required Config |
|------|-------------|-----------------|
| `google_Send_Email` | Send email to self | `content` |
| `google_Create_Sheet` | Create Google Sheet | `content` |
| `google_Create_Event` | Create Calendar event | `content` |

### Steam

**Actions:**
| Name | Description |
|------|-------------|
| `steam_valiance_playing` | User is playing a game |

### Twitch

**Actions:**
| Name | Description | Required Config |
|------|-------------|-----------------|
| `twitch_streamer_live` | Streamer is live | `streamer_name` |

### OpenWeather

**Actions:**
| Name | Description |
|------|-------------|
| `openweather_rain` | It's raining |

## Database Schema

The application uses PostgreSQL with Prisma ORM.

**Core Models:**

- **Users**: User accounts with email/password or OAuth
- **Services**: Available service integrations (Google, GitHub, Discord, etc.)
- **UserServiceTokens**: OAuth tokens for user-service connections
- **Areas**: Automation workflows linking actions to reactions
- **Actions**: Event detectors with configuration and trigger state
- **Reactions**: Response handlers with configuration

**Relationships:**
```
Users 1--* Areas
Users 1--* UserServiceTokens
Services 1--* UserServiceTokens
Services 1--* Actions
Services 1--* Reactions
Areas 1--1 Actions
Areas 1--1 Reactions
```

## How It Works

1. **User creates an Area** with a name
2. **User links an Action** (e.g., `github_new_PR` on a specific repo)
3. **User links a Reaction** (e.g., `discord_send_message` to a channel)
4. **Routine** polls every 5 seconds, checking if action conditions are met
5. When triggered, the action's `triggered` flag is set to `true`
6. **Engine** detects triggered actions and executes corresponding reactions
7. After execution, `triggered` is reset to `false`

## Tech Stack

**Backend:**
- Go 1.21+ with Gin framework
- Prisma (Go client) for database access
- JWT for authentication
- bcrypt for password hashing

**Frontend:**
- Next.js 16
- React 19
- Material-UI
- Tailwind CSS

**Mobile:**
- React Native 0.82
- React Navigation
- React Native Paper

**Infrastructure:**
- Docker & Docker Compose
- PostgreSQL 13
- Alpine Linux (container images)

## Contributing

Want to add new services, actions, or reactions? See [HOWTOCONTRIBUTE.md](HOWTOCONTRIBUTE.md) for detailed instructions on extending the project.
