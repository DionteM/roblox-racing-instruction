# AIDriverScript - Pseudocode

```lua
-- AI Kart Driver Script - Main Controller
-- Place this script inside the Driver model in ServerStorage
-- Detectors (ForwardDetector, LeftDetector, RightDetector, TightForwardDetector) must already exist as children of Kart
wait(3)
local Driver = script.Parent
local Humanoid = Driver:WaitForChild("Humanoid")
local HumanoidRootPart = Driver:WaitForChild("HumanoidRootPart")
local Head = Driver:WaitForChild("Head")

-- Services
local Teams = game:GetService("Teams")
local ChatService = game:GetService("Chat")

-- Get team name from driver's name
local driverName = Driver.Name
local teamName = string.match(driverName, "(.+)Driver")
if not teamName then
	warn("Could not parse team name from driver name: " .. driverName)
	return
end

-- Find the team
local Team = nil
for _, team in Teams:GetTeams() do
	if team.Name == teamName then
		Team = team
		break
	end
end

if not Team then
	warn("Team not found: " .. teamName)
	return
end

-- Find the kart
local kartName = teamName .. "Kart"
local Kart = workspace:FindFirstChild(kartName)
if not Kart then
	warn("Kart not found: " .. kartName)
	return
end

local VehicleSeat = Kart:WaitForChild("VehicleSeat")

-- Get kart components
local RearRightDrivingHinge = Kart:WaitForChild("RearRightDrivingHinge")
local RearLeftDrivingHinge = Kart:WaitForChild("RearLeftDrivingHinge")
local FrontLeftSteeringHinge = Kart:WaitForChild("FrontLeftSteeringHinge")
local FrontRightSteeringHinge = Kart:WaitForChild("FrontRightSteeringHinge")

-- Get existing detectors from kart
local ForwardDetector = Kart:WaitForChild("ForwardDetector")
local LeftDetector = Kart:WaitForChild("LeftDetector")
local RightDetector = Kart:WaitForChild("RightDetector")
local TightForwardDetector = Kart:WaitForChild("TightForwardDetector")

local TireColliders = {}
for _, instance in Kart:GetDescendants() do
	if instance:IsA("Part") and string.match(instance.Name, "TireCollider") then
		table.insert(TireColliders, instance)
	end
end

-- Get all checkpoints and sort them
local Checkpoints = {}
for _, instance in workspace:GetDescendants() do
	if instance:IsA("BasePart") and string.match(instance.Name, "Checkpoint") then
		table.insert(Checkpoints, instance)
	end
end
table.sort(Checkpoints, function(a, b)
	local numA = tonumber(string.match(a.Name, "%d+"))
	local numB = tonumber(string.match(b.Name, "%d+"))
	return numA < numB
end)

-- Configuration
local randNORMAL_SPEED = math.random(300,350)
local randNORMAL_ACCERATION = math.random(550,600)
local Config = {
	-- Base driving
	MAX_STEERING_ANGLE = 40,
	NORMAL_SPEED = randNORMAL_SPEED,
	NORMAL_ACCELERATION = randNORMAL_ACCERATION,

	-- Steering purchase
	STEERING_LIMIT = 40,
	OVER_LIMIT_MULTIPLIER = 15,
	SLOWDOWN_AMOUNT = 400,

	-- Avoid
	AVOID_INITIAL_CORRECTION = 25,
	AVOID_CORRECTION_DECAY = 1,
	AVOID_CORRECTION_REGEN_RATE = 15,
	AVOID_REGEN_DELAY = 0.1,

	-- Pursuit
	PURSUIT_ACTIVATION_TIME = 3,
	PURSUIT_MAX_DISTANCE = 50,

	-- Dive
	DIVE_MAX_DISTANCE = 50,
	DIVE_TURN_ANGLE_THRESHOLD = 20,

	-- Dodge
	DODGE_DISTANCE_SPEED_THRESHOLD = 5,
	DODGE_SPEED_DIFFERENTIAL = 5,
	DODGE_STEERING_BOOST = 15,
	DODGE_SPEED_REDUCTION = 100,

	-- Chaining
	CHAINING_DELAY = 0.2,

	-- Reverse
	REVERSE_SPEED_THRESHOLD = 10,
	REVERSE_TIME_THRESHOLD = 1,
	REVERSE_DURATION = 0.5,
	REVERSE_SPEED = 100,
}

-- State management
local StateValue = Instance.new("StringValue")
StateValue.Name = "State"
StateValue.Value = "Default"
StateValue.Parent = Kart

-- Communication folder
local DataFolder = Instance.new("Folder")
DataFolder.Name = "DriverData"
DataFolder.Parent = Kart

-- Next checkpoint tracker
local NextCheckpointValue = Instance.new("ObjectValue")
NextCheckpointValue.Name = "NextCheckpoint"
NextCheckpointValue.Value = Checkpoints[1]
NextCheckpointValue.Parent = DataFolder

-- State-specific data
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

local currentCheckpointIndex = 1
local lapCounter = 0
local isSeated = false

-- Helper: Get effective checkpoint number with laps
local function getEffectiveCheckpointNumber(checkpointIndex)
	return checkpointIndex + (lapCounter * #Checkpoints)
end

-- Helper: Get checkpoint number from checkpoint part
local function getCheckpointNumber(checkpoint)
	return tonumber(string.match(checkpoint.Name, "%d+"))
end

-- Helper: Chat message
local function chat(message)
	--pcall(function()
		--ChatService:Chat(Head,message, Enum.ChatColor.White)
	--end)
end

-- Chat message tables with random options
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

local function getRandomMessage(messageType)
	local messages = ChatMessages[messageType]
	if messages then
		return messages[math.random(1, #messages)]
	end
	return ""
end

-- Helper: Walk to position
local function walkTo(position)
	Humanoid:MoveTo(position)
	local moveTimeout = tick() + 10
	repeat
		task.wait(0.1)
		local distance = (HumanoidRootPart.Position - position).Magnitude
		if distance < 5 or tick() > moveTimeout then break end
	until not Humanoid or Humanoid.Health <= 0
end

-- Steering purchase function
local function applySteeringPurchase(requestedAngle)
	local overLimit = math.abs(requestedAngle) - Config.STEERING_LIMIT
	if overLimit > 0 then
		local speedReduction = Config.SLOWDOWN_AMOUNT + (overLimit * Config.OVER_LIMIT_MULTIPLIER)
		return requestedAngle, speedReduction
	end
	return requestedAngle, 0
end

-- Speed control
local function setSpeed(speed)
	RearRightDrivingHinge.AngularVelocity = -speed
	RearLeftDrivingHinge.AngularVelocity = -speed
	RearRightDrivingHinge.MotorMaxAcceleration = Config.NORMAL_ACCELERATION
	RearLeftDrivingHinge.MotorMaxAcceleration = Config.NORMAL_ACCELERATION
end

-- Steering control
local function steer(angle)
	angle = math.clamp(angle, -Config.MAX_STEERING_ANGLE, Config.MAX_STEERING_ANGLE)
	FrontLeftSteeringHinge.LowerAngle = angle
	FrontLeftSteeringHinge.UpperAngle = angle
	FrontRightSteeringHinge.LowerAngle = angle
	FrontRightSteeringHinge.UpperAngle = angle
end

-- Calculate steering angle to target
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

-- Get current speed
local function getSpeed()
	if VehicleSeat and VehicleSeat.AssemblyLinearVelocity then
		return VehicleSeat.AssemblyLinearVelocity.Magnitude
	end
	return 0
end

-- State transition
local function setState(newState, reason)
	if StateValue.Value ~= newState then
		StateValue.Value = newState

		-- Chat messages with random selection
		if ChatMessages[newState] then
			chat(getRandomMessage(newState))
		end
	end
end

-- Detector logic: Check if part is rival bumper
local function isRivalBumper(part)
	if not part.Name:match("Bumper") then return false, nil end
	if part.BrickColor == Team.TeamColor then return false, nil end
	return true, part:FindFirstAncestorOfClass("Model")
end

-- Forward detector tracking
local forwardTouchingRivals = {}
local forwardTouchingCheckpoint = nil -- Which checkpoint it's touching

ForwardDetector.Touched:Connect(function(part)
	-- Check checkpoint
	if part.Name:match("Checkpoint") then
		forwardTouchingCheckpoint = part

		-- End drift if touching nextCheckpoint
		if DriftData.isActive and part == NextCheckpointValue.Value then
			DriftData.isActive = false
			for _, tire in pairs(TireColliders) do
				--tire.Material = Enum.Material.Rubber
			end
			chat("Drift ended")
		end
		return
	end

	-- Check rival
	local isRival, rivalKart = isRivalBumper(part)
	if isRival and rivalKart then
		if not forwardTouchingRivals[rivalKart] then
			forwardTouchingRivals[rivalKart] = tick()

			-- Start pursuit timer
			task.delay(Config.PURSUIT_ACTIVATION_TIME, function()
				if forwardTouchingRivals[rivalKart] then
					PursuitData.targetKart = rivalKart
					PursuitData.isActive = true
					setState("Pursuit", "forward detection")
				end
			end)
		end
	end
end)

ForwardDetector.TouchEnded:Connect(function(part)
	if part.Name:match("Checkpoint") then
		if forwardTouchingCheckpoint == part then
			forwardTouchingCheckpoint = nil

			-- Start drift if not touching any checkpoint
			if not DriftData.isActive then
				DriftData.isActive = true
				for _, tire in pairs(TireColliders) do
					--tire.Material = Enum.Material.Rubber
				end
				setState("Drift", "no checkpoint contact")
			end
		end

		-- End pursuit/dive if checkpoint lost
		if PursuitData.isActive or DiveData.isActive then
			PursuitData.isActive = false
			if DiveData.isActive then
				DiveData.isActive = false
				if DiveData.steeringConnection then
					DiveData.steeringConnection:Disconnect()
					DiveData.steeringConnection = nil
				end
			end
		end
		return
	end

	local isRival, rivalKart = isRivalBumper(part)
	if isRival and rivalKart then
		forwardTouchingRivals[rivalKart] = nil
		if PursuitData.targetKart == rivalKart then
			PursuitData.isActive = false
		end
	end
end)

-- Side detectors
local function onSideTouch(part, side)
	local isRival, rivalKart = isRivalBumper(part)
	if not isRival then return end

	-- Activate avoid
	AvoidData.side = side
	AvoidData.lastTriggerTime = tick()
	AvoidData.correction = math.max(0, AvoidData.correction - Config.AVOID_CORRECTION_DECAY)
	setState("Avoid", side .. " detector")

	-- Check dodge conditions
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

LeftDetector.Touched:Connect(function(part) onSideTouch(part, "Left") end)
RightDetector.Touched:Connect(function(part) onSideTouch(part, "Right") end)

LeftDetector.TouchEnded:Connect(function(part)
	local isRival = isRivalBumper(part)
	if isRival and AvoidData.side == "Left" then
		AvoidData.side = nil
	end
end)

RightDetector.TouchEnded:Connect(function(part)
	local isRival = isRivalBumper(part)
	if isRival and AvoidData.side == "Right" then
		AvoidData.side = nil
	end
end)

-- TightForwardDetector for Chaining
TightForwardDetector.Touched:Connect(function(part)
	if part.Name:match("Checkpoint") and part == NextCheckpointValue.Value then
		ChainingData.isActive = true
		setState("Chaining", "tight forward touched checkpoint")
	end
end)

TightForwardDetector.TouchEnded:Connect(function(part)
	if part.Name:match("Checkpoint") and part == NextCheckpointValue.Value then
		ChainingData.lastTouchEndTime = tick()
		-- Delay before ending chaining (prevent flip-flop)
		task.delay(Config.CHAINING_DELAY, function()
			if tick() - ChainingData.lastTouchEndTime >= Config.CHAINING_DELAY then
				ChainingData.isActive = false
			end
		end)
	end
end)

-- Avoid correction regeneration
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

-- Main driving update
local function updateDriving()
	if not isSeated then return false end

	local nextCheckpoint = Checkpoints[currentCheckpointIndex]
	local nextCheckpointNumber = getEffectiveCheckpointNumber(currentCheckpointIndex)

	local steeringAngle = 0
	local speedReduction = 0
	local targetSpeed = Config.NORMAL_SPEED
	local currentSpeed = getSpeed()

	-- Check for reverse conditions (highest priority)
	if currentSpeed < Config.REVERSE_SPEED_THRESHOLD then
		if ReverseData.lowSpeedStartTime == 0 then
			ReverseData.lowSpeedStartTime = tick()
		elseif not ReverseData.isActive and (tick() - ReverseData.lowSpeedStartTime) >= Config.REVERSE_TIME_THRESHOLD then
			-- Activate reverse
			ReverseData.isActive = true
			ReverseData.startTime = tick()
			setState("Reverse", "stuck at low speed")
			chat(getRandomMessage("Stuck"))
		end
	else
		ReverseData.lowSpeedStartTime = 0
	end

	-- REVERSE STATE (highest priority)
	if ReverseData.isActive then
		local reverseElapsed = tick() - ReverseData.startTime

		if reverseElapsed < Config.REVERSE_DURATION then
			-- Still reversing
			setState("Reverse", "backing up")

			-- Invert steering and reverse
			local forwardAngle = calculateSteeringAngle(nextCheckpoint.Position)
			steeringAngle = -forwardAngle -- Invert steering

			-- Set reverse speed
			RearRightDrivingHinge.AngularVelocity = Config.REVERSE_SPEED -- Positive for reverse
			RearLeftDrivingHinge.AngularVelocity = Config.REVERSE_SPEED
			RearRightDrivingHinge.MotorMaxAcceleration = Config.NORMAL_ACCELERATION
			RearLeftDrivingHinge.MotorMaxAcceleration = Config.NORMAL_ACCELERATION

			steer(steeringAngle)
			return true
		else
			-- Done reversing
			ReverseData.isActive = false
			ReverseData.lowSpeedStartTime = 0
			chat("Back in action!")
		end
	end

	-- State priority: Avoid > Drift > Dodge > Dive > Pursuit > Chaining > Default

	if AvoidData.side then
		-- AVOID STATE
		setState("Avoid", "avoiding")
		local baseAngle = calculateSteeringAngle(nextCheckpoint.Position)
		local correction = AvoidData.correction

		if AvoidData.side == "Left" then
			steeringAngle = baseAngle + correction
		elseif AvoidData.side == "Right" then
			steeringAngle = baseAngle - correction
		end

		local speedFactor = currentSpeed / Config.NORMAL_SPEED
		speedReduction = correction * speedFactor * 0.5

	elseif DriftData.isActive then
		-- DRIFT STATE (no checkpoint contact)
		setState("Drift", "drifting")
		steeringAngle = calculateSteeringAngle(nextCheckpoint.Position)
		-- No steering purchase in drift

	elseif DodgeData.isActive then
		-- DODGE STATE
		setState("Dodge", "dodging")
		local baseAngle = calculateSteeringAngle(nextCheckpoint.Position)

		if baseAngle > 0 then
			steeringAngle = baseAngle + Config.DODGE_STEERING_BOOST
		elseif baseAngle < 0 then
			steeringAngle = baseAngle - Config.DODGE_STEERING_BOOST
		else
			steeringAngle = math.random(0, 1) == 1 and Config.DODGE_STEERING_BOOST or -Config.DODGE_STEERING_BOOST
		end

		speedReduction = Config.DODGE_SPEED_REDUCTION
		DodgeData.isActive = false -- One-frame dodge

	elseif DiveData.isActive and DiveData.targetKart then
		-- DIVE STATE (copying rival's steering)
		setState("Dive", "diving")
		local targetSeat = DiveData.targetKart:FindFirstChild("VehicleSeat")

		if targetSeat then
			local diveDistance = (VehicleSeat.Position - targetSeat.Position).Magnitude
			if diveDistance > Config.DIVE_MAX_DISTANCE then
				DiveData.isActive = false
				if DiveData.steeringConnection then
					DiveData.steeringConnection:Disconnect()
					DiveData.steeringConnection = nil
				end
				chat("Dive ended - out of range")
			end
		else
			DiveData.isActive = false
			if DiveData.steeringConnection then
				DiveData.steeringConnection:Disconnect()
				DiveData.steeringConnection = nil
			end
		end

		-- Steering is handled by the PropertyChangedSignal connection
		-- Don't override it here, just maintain current steering
		return true

	elseif PursuitData.isActive and PursuitData.targetKart then
		-- PURSUIT STATE
		setState("Pursuit", "pursuing")
		local targetSeat = PursuitData.targetKart:FindFirstChild("VehicleSeat")

		if targetSeat then
			local pursuitDistance = (VehicleSeat.Position - targetSeat.Position).Magnitude
			if pursuitDistance > Config.PURSUIT_MAX_DISTANCE then
				PursuitData.isActive = false
			else
				steeringAngle = calculateSteeringAngle(targetSeat.Position)

				-- Check for dive transition
				local targetVelocity = targetSeat.AssemblyLinearVelocity
				if targetVelocity.Magnitude > 5 then
					local targetForward = targetSeat.CFrame.LookVector
					local velocityDir = targetVelocity.Unit
					local targetForward2D = Vector3.new(targetForward.X, 0, targetForward.Z).Unit
					local velocityDir2D = Vector3.new(velocityDir.X, 0, velocityDir.Z).Unit
					local dot = targetForward2D:Dot(velocityDir2D)
					local turnAngle = math.deg(math.acos(math.clamp(dot, -1, 1)))

					if turnAngle > Config.DIVE_TURN_ANGLE_THRESHOLD then
						DiveData.isActive = true
						DiveData.targetKart = PursuitData.targetKart
						PursuitData.isActive = false

						-- Connect to rival's steering to copy it
						local rivalSteeringHinge = DiveData.targetKart:FindFirstChild("FrontRightSteeringHinge")
						if rivalSteeringHinge then
							DiveData.steeringConnection = rivalSteeringHinge:GetPropertyChangedSignal("UpperAngle"):Connect(function()
								if DiveData.isActive then
									steer(rivalSteeringHinge.UpperAngle)
								end
							end)
						end

						setState("Dive", "target turning sharply")
					end
				end
			end
		else
			PursuitData.isActive = false
		end

	elseif ChainingData.isActive then
		-- CHAINING STATE (targeting next checkpoint)
		setState("Chaining", "chaining to next")
		local nextIndex = currentCheckpointIndex + 1
		if nextIndex > #Checkpoints then
			nextIndex = 1
		end
		local targetCheckpoint = Checkpoints[nextIndex]
		steeringAngle = calculateSteeringAngle(targetCheckpoint.Position)

	else
		-- DEFAULT STATE
		setState("Default", "default navigation")
		steeringAngle = calculateSteeringAngle(nextCheckpoint.Position)
	end

	-- Apply steering purchase (unless in drift)
	if not DriftData.isActive then
		local purchaseReduction
		steeringAngle, purchaseReduction = applySteeringPurchase(steeringAngle)
		speedReduction = speedReduction + purchaseReduction
	end

	-- Apply steering and speed
	steer(steeringAngle)
	setSpeed(targetSpeed - speedReduction)

	return true
end

-- Start execution
chat("AI Driver starting!")
walkTo(VehicleSeat.Position)
VehicleSeat:Sit(Humanoid)
task.wait(0.5)
isSeated = true
chat("Starting race!")

-- Checkpoint advancement (TightForwardDetector handles this)
TightForwardDetector.Touched:Connect(function(part)
	if part.Name:match("Checkpoint") then
		local touchedCheckpointNum = getCheckpointNumber(part)
		local touchedEffectiveNum = touchedCheckpointNum + (lapCounter * #Checkpoints)
		local nextEffectiveNum = getEffectiveCheckpointNumber(currentCheckpointIndex)

		-- Check if this is the next checkpoint or higher
		if touchedEffectiveNum >= nextEffectiveNum then
			-- Advance to next checkpoint
			currentCheckpointIndex = currentCheckpointIndex + 1

			-- Handle lap completion
			if currentCheckpointIndex > #Checkpoints then
				currentCheckpointIndex = 1
				lapCounter = lapCounter + 1
				chat(getRandomMessage("LapComplete") .. " LAP " .. lapCounter .. "!")
			end

			NextCheckpointValue.Value = Checkpoints[currentCheckpointIndex]

			-- End dive if passed checkpoint while diving
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

-- Main loop
while true do
	if #Checkpoints == 0 then
		warn("No checkpoints found!")
		break
		
	end

	updateDriving()
	task.wait(0.05)
end
```
