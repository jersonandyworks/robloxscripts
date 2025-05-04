# Explaining the Enemy Code 🧒🎮

Let's break down the enemy code like we're telling a story about a enemy guard!

## 🌍 Part 1: Setting Up Our enemy Guard

```lua
local enemy = script.Parent
local rootPart = enemy:WaitForChild("HumanoidRootPart")
local humanoid = enemy:FindFirstChildOfClass("Humanoid") or Instance.new("Humanoid", enemy)

```

- enemy = Our enemy guard
- rootPart = The enemy's belly button (center point)
- humanoid = Makes the enemy walk like a person

## 👀 Part 2: How Far Our enemy Can See

```lua
local maxAngleDegrees = 60
local maxDistance = 30
```

- The enemy can only see 60 degrees in front (like wearing horse blinders)
- It can only see 30 steps away (studs in Roblox)

## 🛣️ Part 3: enemy's GPS System

```lua
local pathfinding = game:GetService("PathfindingService")
```

\* This is the enemy's map app that helps it find paths around walls

## 🚶 Part 4: Making the enemy Walk Nicely

```lua
humanoid.WalkSpeed = 16
humanoid:SetStateEnabled(Enum.HumanoidStateType.Climbing, false)
humanoid:SetStateEnabled(Enum.HumanoidStateType.Seated, false)
```

- WalkSpeed = 16 = Walking speed (faster than normal)
- The enemy won't climb or sit down (we turned those off)

## 🔦 Part 5: The enemy's Flashlight Vision

```lua
local raycastParams = RaycastParams.new()
raycastParams.FilterDescendantsInstances = {enemy}
raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
```

- raycastParams = Special flashlight settings
- It ignores the enemy's own body when looking (so it doesn't see itself)

## 👁️ Part 6: How the enemy Spies Players

```lua
local function canSeePlayer(targetRoot)
local direction = (targetRoot.Position - rootPart.Position)
local distance = direction.Magnitude
```

- canSeePlayer = The enemy's "spy check"
- It measures how far away you are and which direction

## 📐 Part 7: Checking the Vision Cone

```lua
if distance > maxDistance then return false end

local lookVector = rootPart.CFrame.LookVector
local angle = math.deg(math.acos(lookVector:Dot(direction.Unit)))
if angle > maxAngleDegrees then return false end
```

- If you're too far → "I can't see you!"
- If you're outside the 60° cone → "I can't see you!"

## 🧱 Part 8: Wall Detection

```lua
    local rayResult = workspace:Raycast(
        rootPart.Position + Vector3.new(0,1.5,0),
        direction,
        raycastParams
    )

    if rayResult then
        local hitParent = rayResult.Instance:FindFirstAncestorOfClass("Model")
        if hitParent == targetRoot.Parent then
            return true
        end
        return false
    end
```

- The enemy shoots an invisible laser beam
- If it hits a wall first → "I can't see through walls!"
- If it hits you → "I SEE YOU!"

## 🏃 Part 9: Chasing Players

```lua
    local function chasePlayer(target)
    local targetRoot = target.Character.HumanoidRootPart
    local lastSeenPosition = targetRoot.Position
```

- chasePlayer = The enemy's running mode
- It remembers where it last saw you

🗺️ Part 10: Using the GPS

```lua
    while task.wait(0.5) and target.Character do
        if canSeePlayer(targetRoot) then
            lastSeenPosition = targetRoot.Position

            local path = pathfinding:CreatePath({
                AgentRadius = 2,
                AgentHeight = 5,
                AgentCanJump = false
            })
```

- Checks every 0.5 seconds if it still sees you
- Uses GPS pathfinding to find the best route

## 🚶➡️🏃 Part 11: Following the Path

```lua
            path:ComputeAsync(rootPart.Position, lastSeenPosition)

            if path.Status == Enum.PathStatus.Success then
                for _, waypoint in ipairs(path:GetWaypoints()) do
                    humanoid:MoveTo(waypoint.Position)
                    local reached = humanoid.MoveToFinished:Wait(0.5)
                    if not reached then break end
                end
            end
```

- The enemy follows breadcrumb trail (waypoints)
- If it gets stuck, it gives up on that path

## 👋 Part 12: Main Game Loop

```lua
while task.wait(0.3) do
    for _, player in ipairs(game.Players:GetPlayers()) do
        if player.Character then
            if canSeePlayer(player.Character.HumanoidRootPart) then
                chasePlayer(player)
                break
            end
        end
    end
end

```

- The enemy constantly checks all players
- If it sees one, it chases them and ignores others

### 🌈 Fun Summary

🤖 Enemy has flashlight vision (60° cone, 30 studs)
🧱 It can't see through walls
🗺️ Uses GPS pathfinding to chase
🔄 Constantly checks for players

Now your Enemy guard will perfectly chase players while respecting walls!
