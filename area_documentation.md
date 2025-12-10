# Documentation: Adding a New AREA

## System Architecture

The AREA (Action-REAction) system works in 3 steps:
1. **Action**: Detects an event on a service
2. **Engine**: Listens for triggered actions
3. **Reaction**: Executes a response on another service

## Steps to Add a New AREA

### 1. Define the Action in `/action`

Create a file for your new action (e.g., `githubIssue.go`):

```go
func GetNewAction(userID string) {
    // 1. Retrieve the service token
    // 2. Call the service API
    // 3. If condition is met, trigger the action
    
    _, err = initializers.DB.Actions.FindUnique(
        db.Actions.ID.Equals(action.ID),
    ).Update(
        db.Actions.Triggered.Set(true),
    ).Exec(ctx)
}
```

### 2. Register the Routine in `/routine/routine.go`

Add your action to the polling system:

```go
func LaunchRoutines() {
    go func() {
        ticker := time.NewTicker(5 * time.Second)
        for range ticker.C {
            users, _ := initializers.DB.Users.FindMany().Exec(context.Background())
            
            for _, user := range users {
                action.GetGithubWebHook(user.ID)
                action.GetNewAction(user.ID)  // ← Your new action
            }
        }
    }()
}
```

### 3. Create the Reaction in `/reaction`

Create a file for your reaction (e.g., `slackMsg.go`):

```go
func ReactWithSlackMsg(user db.UsersModel, message string) {
    // 1. Configure the webhook/API
    // 2. Prepare the payload
    // 3. Send the HTTP request
}
```

### 4. Add the Listener in `/engine/listener.go`

Connect action and reaction:

```go
if area.Name == "Github_issue_to_slack" {  // ← Your AREA name
    for _, action := range actions {
        if action.Triggered {
            // Reset the trigger
            initializers.DB.Actions.FindUnique(...).Update(
                db.Actions.Triggered.Set(false),
            ).Exec(ctx)
            
            // Execute the reaction
            for _, user := range users {
                reaction.ReactWithSlackMsg(user, "New issue!")
            }
        }
    }
}
```

### 5. Define Models in `/model`

If necessary, add your configuration structures:

```go
type SlackConfig struct {
    WebhookURL string `json:"webhook_url"`
}
```

## Complete Example: Github PR → Discord

**Action** (`githubPr.go`): Detects closed PRs
↓
**Routine**: Checks every 5 seconds
↓
**Engine**: Listens for `Github_pr_to_discord` with `Triggered = true`
↓
**Reaction** (`discordMsg.go`): Sends a Discord message

## Key Points

- **Polling**: 5 seconds (configurable in `time.NewTicker`)
- **Trigger**: Boolean flag system `Triggered` on the action
- **Reset**: The listener sets `Triggered = false` after execution
- **User tokens**: Retrieved via `UserServiceTokens` for each service

## Database

Use Prisma (`schema.prisma`) to define your tables:
- `Areas`: AREA configurations
- `Actions`: Trigger events
- `Reactions`: Automated responses
- `UserServiceTokens`: User OAuth2 tokens
