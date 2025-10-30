# AIDriverScript - Pseudocode

## Part 1: Init

1. At the very top of your script, add a wait command:
   ```lua
   wait(3)
   ```

2. Create a variable called `Driver` and set it equal to `script.Parent`

3. Create a variable called `Humanoid` and use `Driver:WaitForChild("Humanoid")` to find it

4. Create a variable called `HumanoidRootPart` and use `Driver:WaitForChild("HumanoidRootPart")` to find it

5. Create a variable called `Head` and use `Driver:WaitForChild("Head")` to find it

---

## 🎮 Part 2: Serivces

6. Create a variable called `Teams` and use `game:GetService("Teams")` to get the Teams service

7. Create a variable called `ChatService` and use `game:GetService("Chat")` to get the Chat service

---

## 👥 Part 3: Teams

8. Create a variable called `driverName` and set it to `Driver.Name`

9. Create a variable called `teamName` and use `string.match(driverName, "(.+)Driver")` to get the team name

10. Add an `if not teamName then` statement to check if it worked:
    - Inside, use `warn("Could not parse team name from driver name: " .. driverName)`
    - Then add `return` to stop the script

11. Create a variable called `Team` and set it to `nil`

12. Use a `for` loop: `for _, team in Teams:GetTeams() do`
    - Inside the loop, check `if team.Name == teamName then`
      - Set `Team = team`
      - Use `break` to exit the loop
    - Close with `end`
    - Close the for loop with `end`

13. After the loop, check `if not Team then`
    - Use `warn("Team not found: " .. teamName)`
    - Add `return`
    - Close with `end`

---

## 🏎️ Part 4: Kart

14. Create a variable called `kartName` and set it to `teamName .. "Kart"`

15. Create a variable called `Kart` and use `workspace:FindFirstChild(kartName)` to find it

16. Check `if not Kart then`
    - Use `warn("Kart not found: " .. kartName)`
    - Add `return`
    - Close with `end`

17. Create a variable called `VehicleSeat` and use `Kart:WaitForChild("VehicleSeat")`

---

## ⚙️ Part 5: Kart's Parts

18. Create a variable called `RearRightDrivingHinge` and use `Kart:WaitForChild("RearRightDrivingHinge")`

19. Create a variable called `RearLeftDrivingHinge` and use `Kart:WaitForChild("RearLeftDrivingHinge")`

20. Create a variable called `FrontLeftSteeringHinge` and use `Kart:WaitForChild("FrontLeftSteeringHinge")`

21. Create a variable called `FrontRightSteeringHinge` and use `Kart:WaitForChild("FrontRightSteeringHinge")`

22. Create a variable called `ForwardDetector` and use `Kart:WaitForChild("ForwardDetector")`

23. Create a variable called `LeftDetector` and use `Kart:WaitForChild("LeftDetector")`

24. Create a variable called `RightDetector` and use `Kart:WaitForChild("RightDetector")`

25. Create a variable called `TightForwardDetector` and use `Kart:WaitForChild("TightForwardDetector")`

---

## 🛞 Part 6: Tires

26. Create an empty table called `TireColliders` using `{}`

27. Use a `for` loop: `for _, instance in Kart:GetDescendants() do`
    - Check `if instance:IsA("Part") and string.match(instance.Name, "TireCollider") then`
      - Use `table.insert(TireColliders, instance)` to add the tire
      - Close with `end`
    - Close the loop with `end`

---

## 🚩 Part 7: Checkpoints

28. Create an empty table called `Checkpoints` using `{}`

29. Use a `for` loop: `for _, instance in workspace:GetDescendants() do`
    - Check `if instance:IsA("BasePart") and string.match(instance.Name, "Checkpoint") then`
      - Use `table.insert(Checkpoints, instance)`
      - Close with `end`
    - Close the loop with `end`

30. **Copy and paste this code** to sort the checkpoints:
    ```lua
    table.sort(Checkpoints, function(a, b)
        local numA = tonumber(string.match(a.Name, "%d+"))
        local numB = tonumber(string.match(b.Name, "%d+"))
        return numA < numB
    end)
    ```

---

## ⚙️ Part 8: Configuration Settings

31. Create a variable called `randNORMAL_SPEED` and use `math.random(300, 350)`

32. Create a variable called `randNORMAL_ACCERATION` and use `math.random(550, 600)`

33. **Copy and paste this Config table:**
    ```lua
    local Config = {
        MAX_STEERING_ANGLE = 40,
        NORMAL_SPEED = randNORMAL_SPEED,
        NORMAL_ACCELERATION = randNORMAL_ACCERATION,
        STEERING_LIMIT = 40,
        OVER_LIMIT_MULTIPLIER = 15,
        SLOWDOWN_AMOUNT = 400,
        AVOID_INITIAL_CORRECTION = 25,
        AVOID_CORRECTION_DECAY = 1,
        AVOID_CORRECTION_REGEN_RATE = 15,
        AVOID_REGEN_DELAY = 0.1,
        PURSUIT_ACTIVATION_TIME = 3,
        PURSUIT_MAX_DISTANCE = 50,
        DIVE_MAX_DISTANCE = 50,
        DIVE_TURN_ANGLE_THRESHOLD = 20,
        DODGE_DISTANCE_SPEED_THRESHOLD = 5,
        DODGE_SPEED_DIFFERENTIAL = 5,
        DODGE_STEERING_BOOST = 15,
        DODGE_SPEED_REDUCTION = 100,
        CHAINING_DELAY = 0.2,
        REVERSE_SPEED_THRESHOLD = 10,
        REVERSE_TIME_THRESHOLD = 1,
        REVERSE_DURATION = 0.5,
        REVERSE_SPEED = 100,
    }
    ```

---

## 📊 Part 9: Instance Attributes

34. Create a `StringValue` object:
    - Use `Instance.new("StringValue")`
    - Set its `Name` property to `"State"`
    - Set its `Value` property to `"Default"`
    - Set its `Parent` property to `Kart`
    - Store this in a variable called `StateValue`

35. Create a `Folder` object:
    - Use `Instance.new("Folder")`
    - Set its `Name` to `"DriverData"`
    - Set its `Parent` to `Kart`
    - Store this in a variable called `DataFolder`

36. Create an `ObjectValue`:
    - Use `Instance.new("ObjectValue")`
    - Set its `Name` to `"NextCheckpoint"`
    - Set its `Value` to `Checkpoints[1]`
    - Set its `Parent` to `DataFolder`
    - Store this in a variable called `NextCheckpointValue`

---

## 📦 Part 10: Data Tables

37. **Copy and paste these data tables:**
    ```lua
    local AvoidData = {
        correction = Config.AVOID_INITIAL_CORRECTION,
        lastTriggerTime = 0,
        side = nil
    }

    local PursuitData = {
        targetKart = nil,
        isActive = false
    }

    local DiveData = {
        targetKart = nil,
        isActive = false,
        steeringConnection = nil
    }

    local DodgeData = {
        isActive = false,
        side = nil
    }

    local DriftData = {
        isActive = false
    }

    local ChainingData = {
        isActive = false,
        lastTouchEndTime = 0
    }

    local ReverseData = {
        isActive = false,
        startTime = 0,
        lowSpeedStartTime = 0
    }
    ```

---

## 🔢 Part 11: Tracked Variables

38. Create a variable called `currentCheckpointIndex` and set it to `1`

39. Create a variable called `lapCounter` and set it to `0`

40. Create a variable called `isSeated` and set it to `false`

---

## 🛠️ Part 12: Helper Functions

### Function 1: Get Effective Checkpoint Number

41. **Copy and paste this function:**
    ```lua
    local function getEffectiveCheckpointNumber(checkpointIndex)
        return checkpointIndex + (lapCounter * #Checkpoints)
    end
    ```

### Function 2: Get Checkpoint Number

42. **Copy and paste this function:**
    ```lua
    local function getCheckpointNumber(checkpoint)
        return tonumber(string.match(checkpoint.Name, "%d+"))
    end
    ```

### Function 3: Chat Function

43. Create a function called `chat` that takes `message` as input:
    - Leave it empty for now (we'll add chat later)

### Function 4: Chat Messages

44. **Copy and paste this ChatMessages table:**
    ```lua
    local ChatMessages = {
        Avoid = {
            "Swerving!",
            "Watch it!",
            "Out of my way!",
            "Move!",
            "Coming through!"
        },
        Drift = {
            "Drifting through!",
            "Sliding!",
            "Sideways baby!",
            "Tokyo drift!",
            "Deja vu!"
        },
        Pursuit = {
            "I'm coming for you!",
            "You're mine!",
            "Target locked!",
            "Get back here!",
            "I see you!"
        },
        Dive = {
            "Going for the dive!",
            "Sending it!",
            "Full commit!",
            "No brakes!",
            "Yolo!"
        },
        Dodge = {
            "Dodging!",
            "Not today!",
            "Nice try!",
            "Too slow!",
            "Can't catch me!"
        },
        Reverse = {
            "RAHH!",
            "GET OUT OF MY WAY!",
            "MOVE IT!",
            "STUCK! BACKING UP!",
            "ARGH!",
            "NOT AGAIN!",
            "REVERSE TIME!"
        },
        LapComplete = {
            "🏁 LAP COMPLETE! 🏁",
            "🏁 ANOTHER ONE! 🏁",
            "🏁 LET'S GO! 🏁",
            "🏁 STILL RACING! 🏁",
            "🏁 CAN'T STOP WON'T STOP! 🏁"
        },
        Stuck = {
            "I'M STUCK!",
            "WHAT THE-",
            "COME ON!",
            "SERIOUSLY?!",
            "RAHH! STUCK!",
            "NO NO NO!"
        }
    }
    ```

### Function 5: Get Random Message

45. **Copy and paste this function:**
    ```lua
    local function getRandomMessage(messageType)
        local messages = ChatMessages[messageType]
        if messages then
            return messages[math.random(1, #messages)]
        end
        return ""
    end
    ```

### Function 6: Walk To Position

46. Create a function called `walkTo` that takes `position` as input:
    - Use `Humanoid:MoveTo(position)`
    - Create a variable `moveTimeout` set to `tick() + 10`
    - Use a `repeat` loop:
      - Use `task.wait(0.1)`
      - Calculate `distance` as `(HumanoidRootPart.Position - position).Magnitude`
      - Check `if distance < 5 or tick() > moveTimeout then break end`
    - Loop `until not Humanoid or Humanoid.Health <= 0`

### Functions 7-9: Complex Math Functions (Just Copy These!)

47. **Copy and paste these three functions:**
    ```lua
    local function applySteeringPurchase(requestedAngle)
        local overLimit = math.abs(requestedAngle) - Config.STEERING_LIMIT
        if overLimit > 0 then
            local speedReduction = Config.SLOWDOWN_AMOUNT + (overLimit * Config.OVER_LIMIT_MULTIPLIER)
            return requestedAngle, speedReduction
        end
        return requestedAngle, 0
    end

    local function setSpeed(speed)
        RearRightDrivingHinge.AngularVelocity = -speed
        RearLeftDrivingHinge.AngularVelocity = -speed
        RearRightDrivingHinge.MotorMaxAcceleration = Config.NORMAL_ACCELERATION
        RearLeftDrivingHinge.MotorMaxAcceleration = Config.NORMAL_ACCELERATION
    end

    local function steer(angle)
        angle = math.clamp(angle, -Config.MAX_STEERING_ANGLE, Config.MAX_STEERING_ANGLE)
        FrontLeftSteeringHinge.LowerAngle = angle
        FrontLeftSteeringHinge.UpperAngle = angle
        FrontRightSteeringHinge.LowerAngle = angle
        FrontRightSteeringHinge.UpperAngle = angle
    end
    ```

### Functions 10-12: More Complex Functions (Copy These Too!)

48. **Copy and paste these functions:**
    ```lua
    local function calculateSteeringAngle(targetPosition)
        if not VehicleSeat or not VehicleSeat.Parent then return 0 end

        local kartCFrame = VehicleSeat.CFrame
        local kartForward = kartCFrame.LookVector
        local toTarget = (targetPosition - VehicleSeat.Position).Unit

        kartForward = Vector3.new(kartForward.X, 0, kartForward.Z).Unit
        toTarget = Vector3.new(toTarget.X, 0, toTarget.Z).Unit

        local dot = kartForward:Dot(toTarget)
        local cross = kartForward:Cross(toTarget)
        local angle = math.acos(math.clamp(dot, -1, 1))
        angle = math.deg(angle)

        if cross.Y < 0 then
            angle = -angle
        end

        return angle
    end

    local function getSpeed()
        if VehicleSeat and VehicleSeat.AssemblyLinearVelocity then
            return VehicleSeat.AssemblyLinearVelocity.Magnitude
        end
        return 0
    end

    local function setState(newState, reason)
        if StateValue.Value ~= newState then
            StateValue.Value = newState
            if ChatMessages[newState] then
                chat(getRandomMessage(newState))
            end
        end
    end

    local function isRivalBumper(part)
        if not part.Name:match("Bumper") then return false, nil end
        if part.BrickColor == Team.TeamColor then return false, nil end
        return true, part:FindFirstAncestorOfClass("Model")
    end
    ```

---

## 🎯 Part 13: Detector Tracking

49. Create an empty table called `forwardTouchingRivals` using `{}`

50. Create a variable called `forwardTouchingCheckpoint` and set it to `nil`

---

## 🔍 Part 14: Forward Detector Events

51. Connect to the `ForwardDetector.Touched` event using `ForwardDetector.Touched:Connect(function(part)`:

    - Check `if part.Name:match("Checkpoint") then`
      - Set `forwardTouchingCheckpoint = part`
      - Check `if DriftData.isActive and part == NextCheckpointValue.Value then`
        - Set `DriftData.isActive = false`
        - Call `chat("Drift ended")`
        - Close with `end`
      - Add `return`
      - Close with `end`
    
    - Create variables `isRival` and `rivalKart` by calling `isRivalBumper(part)`
    - Check `if isRival and rivalKart then`
      - Check `if not forwardTouchingRivals[rivalKart] then`
        - Set `forwardTouchingRivals[rivalKart] = tick()`
        - Use `task.delay(Config.PURSUIT_ACTIVATION_TIME, function()`
          - Check `if forwardTouchingRivals[rivalKart] then`
            - Set `PursuitData.targetKart = rivalKart`
            - Set `PursuitData.isActive = true`
            - Call `setState("Pursuit", "forward detection")`
            - Close with `end`
          - Close function with `end)`
        - Close with `end`
      - Close with `end`
    
    - Close the main function with `end)`

52. Connect to the `ForwardDetector.TouchEnded` event using `ForwardDetector.TouchEnded:Connect(function(part)`:

    - Check `if part.Name:match("Checkpoint") then`
      - Check `if forwardTouchingCheckpoint == part then`
        - Set `forwardTouchingCheckpoint = nil`
        - Check `if not DriftData.isActive then`
          - Set `DriftData.isActive = true`
          - Call `setState("Drift", "no checkpoint contact")`
          - Close with `end`
        - Close with `end`
      - Check `if PursuitData.isActive or DiveData.isActive then`
        - Set `PursuitData.isActive = false`
        - Check `if DiveData.isActive then`
          - Set `DiveData.isActive = false`
          - Check `if DiveData.steeringConnection then`
            - Call `DiveData.steeringConnection:Disconnect()`
            - Set `DiveData.steeringConnection = nil`
            - Close with `end`
          - Close with `end`
        - Close with `end`
      - Add `return`
      - Close with `end`
    
    - Create `isRival` and `rivalKart` by calling `isRivalBumper(part)`
    - Check `if isRival and rivalKart then`
      - Set `forwardTouchingRivals[rivalKart] = nil`
      - Check `if PursuitData.targetKart == rivalKart then`
        - Set `PursuitData.isActive = false`
        - Close with `end`
      - Close with `end`
    
    - Close the function with `end)`

---

## ↔️ Part 15: Side Detectors

53. **Copy and paste this onSideTouch function:**
    ```lua
    local function onSideTouch(part, side)
        local isRival, rivalKart = isRivalBumper(part)
        if not isRival then return end

        AvoidData.side = side
        AvoidData.lastTriggerTime = tick()
        AvoidData.correction = math.max(0, AvoidData.correction - Config.AVOID_CORRECTION_DECAY)
        setState("Avoid", side .. " detector")

        if rivalKart and rivalKart:FindFirstChild("VehicleSeat") then
            local rivalSeat = rivalKart.VehicleSeat
            local distance = (VehicleSeat.Position - rivalSeat.Position).Magnitude
            local mySpeed = getSpeed()
            local rivalSpeed = rivalSeat.AssemblyLinearVelocity.Magnitude
            local speedDiff = math.abs(mySpeed - rivalSpeed)
            local distanceSpeedRatio = distance / (mySpeed + 0.01)

            if distanceSpeedRatio < Config.DODGE_DISTANCE_SPEED_THRESHOLD and speedDiff <= Config.DODGE_SPEED_DIFFERENTIAL then
                DodgeData.isActive = true
                DodgeData.side = side
                setState("Dodge", "dodge conditions met")
            end
        end
    end
    ```

54. Connect the left detector using `LeftDetector.Touched:Connect(function(part) onSideTouch(part, "Left") end)`

55. Connect the right detector using `RightDetector.Touched:Connect(function(part) onSideTouch(part, "Right") end)`

56. Connect `LeftDetector.TouchEnded:Connect(function(part)`:
    - Create `isRival` by calling `isRivalBumper(part)`
    - Check `if isRival and AvoidData.side == "Left" then`
      - Set `AvoidData.side = nil`
      - Close with `end`
    - Close function with `end)`

57. Connect `RightDetector.TouchEnded:Connect(function(part)`:
    - Create `isRival` by calling `isRivalBumper(part)`
    - Check `if isRival and AvoidData.side == "Right" then`
      - Set `AvoidData.side = nil`
      - Close with `end`
    - Close function with `end)`

---

## 🔗 Part 16: Chaining Detector

58. Connect `TightForwardDetector.Touched:Connect(function(part)`:
    - Check `if part.Name:match("Checkpoint") and part == NextCheckpointValue.Value then`
      - Set `ChainingData.isActive = true`
      - Call `setState("Chaining", "tight forward touched checkpoint")`
      - Close with `end`
    - Close function with `end)`

59. Connect `TightForwardDetector.TouchEnded:Connect(function(part)`:
    - Check `if part.Name:match("Checkpoint") and part == NextCheckpointValue.Value then`
      - Set `ChainingData.lastTouchEndTime = tick()`
      - Use `task.delay(Config.CHAINING_DELAY, function()`
        - Check `if tick() - ChainingData.lastTouchEndTime >= Config.CHAINING_DELAY then`
          - Set `ChainingData.isActive = false`
          - Close with `end`
        - Close function with `end)`
      - Close with `end`
    - Close function with `end)`

---

## 🔄 Part 17: Correction Regeneration

60. **Copy and paste this background task:**
    ```lua
    task.spawn(function()
        while true do
            task.wait(0.1)
            if not AvoidData.side then
                local timeSince = tick() - AvoidData.lastTriggerTime
                if timeSince > Config.AVOID_REGEN_DELAY then
                    AvoidData.correction = math.min(
                        AvoidData.correction + Config.AVOID_CORRECTION_REGEN_RATE * 0.1,
                        Config.AVOID_INITIAL_CORRECTION
                    )
                end
            end
        end
    end)
    ```

---

## 🚗 Part 18: Main Function

> **Note:** This function is very complex! Your teacher will help you understand it or provide the code.

61. **Skip this part - your teacher will provide the `updateDriving()` function**

---

## 🏁 Part 19: Start the Race

62. Call `chat("AI Driver starting!")`

63. Call `walkTo(VehicleSeat.Position)`

64. Call `VehicleSeat:Sit(Humanoid)`

65. Use `task.wait(0.5)`

66. Set `isSeated = true`

67. Call `chat("Starting race!")`

---

## 📍 Part 20: Checkpoint System

68. **Copy and paste this checkpoint advancement code:**
    ```lua
    TightForwardDetector.Touched:Connect(function(part)
        if part.Name:match("Checkpoint") then
            local touchedCheckpointNum = getCheckpointNumber(part)
            local touchedEffectiveNum = touchedCheckpointNum + (lapCounter * #Checkpoints)
            local nextEffectiveNum = getEffectiveCheckpointNumber(currentCheckpointIndex)

            if touchedEffectiveNum >= nextEffectiveNum then
                currentCheckpointIndex = currentCheckpointIndex + 1

                if currentCheckpointIndex > #Checkpoints then
                    currentCheckpointIndex = 1
                    lapCounter = lapCounter + 1
                    chat(getRandomMessage("LapComplete") .. " LAP " .. lapCounter .. "!")
                end

                NextCheckpointValue.Value = Checkpoints[currentCheckpointIndex]

                if DiveData.isActive then
                    DiveData.isActive = false
                    if DiveData.steeringConnection then
                        DiveData.steeringConnection:Disconnect()
                        DiveData.steeringConnection = nil
                    end
                    chat("Dive ended - passed checkpoint")
                end
            end
        end
    end)
    ```

---

## ♾️ Part 21: Main Loop

69. Create an infinite loop using `while true do`:
    - Check `if #Checkpoints == 0 then`
      - Use `warn("No checkpoints found!")`
      - Use `break`
      - Close with `end`
    - Call `updateDriving()`
    - Use `task.wait(0.05)`
    - Close the loop with `end`

---

## 🎉 You're Done!
