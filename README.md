local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")

local player = Players.LocalPlayer
local correctKey = "key"

local function getChar()
	return player.Character or player.CharacterAdded:Wait()
end

-- ================= BLUR =================

local blur = Instance.new("BlurEffect")
blur.Size = 0
blur.Parent = Lighting

-- ================= GUI =================

local gui = Instance.new("ScreenGui")
gui.Parent = player:WaitForChild("PlayerGui")

-- ================= KEY SYSTEM =================

local keyFrame = Instance.new("Frame", gui)
keyFrame.Size = UDim2.new(0,350,0,180)
keyFrame.Position = UDim2.new(0.5,-175,0.5,-90)
keyFrame.BackgroundColor3 = Color3.fromRGB(25,25,25)
keyFrame.BorderSizePixel = 0
Instance.new("UICorner",keyFrame).CornerRadius = UDim.new(0,16)

local keyTitle = Instance.new("TextLabel",keyFrame)
keyTitle.Size = UDim2.new(1,0,0,40)
keyTitle.BackgroundTransparency = 1
keyTitle.Text = "put key in"
keyTitle.TextScaled = true
keyTitle.Font = Enum.Font.GothamBold
keyTitle.TextColor3 = Color3.new(1,1,1)

local keyBox = Instance.new("TextBox",keyFrame)
keyBox.Size = UDim2.new(0.8,0,0,40)
keyBox.Position = UDim2.new(0.1,0,0.4,0)
keyBox.BackgroundColor3 = Color3.fromRGB(40,40,40)
keyBox.TextScaled = true
keyBox.Font = Enum.Font.Gotham
keyBox.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner",keyBox).CornerRadius = UDim.new(0,12)

local submit = Instance.new("TextButton",keyFrame)
submit.Size = UDim2.new(0.6,0,0,35)
submit.Position = UDim2.new(0.2,0,0.75,0)
submit.Text = "Submit"
submit.TextScaled = true
submit.Font = Enum.Font.GothamBold
submit.BackgroundColor3 = Color3.fromRGB(0,170,255)
submit.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner",submit).CornerRadius = UDim.new(0,10)

-- ================= MAIN PANEL =================

local main = Instance.new("Frame",gui)
main.Size = UDim2.new(0,420,0,350)
main.Position = UDim2.new(0.5,-210,0.5,-175)
main.BackgroundColor3 = Color3.fromRGB(25,25,25)
main.BorderSizePixel = 0
main.Visible = false
Instance.new("UICorner",main).CornerRadius = UDim.new(0,16)

local topBar = Instance.new("Frame",main)
topBar.Size = UDim2.new(1,0,0,35)
topBar.BackgroundColor3 = Color3.fromRGB(35,35,35)
topBar.BorderSizePixel = 0
Instance.new("UICorner",topBar).CornerRadius = UDim.new(0,16)

local minimize = Instance.new("TextButton",topBar)
minimize.Size = UDim2.new(0,30,0,30)
minimize.Position = UDim2.new(1,-35,0,2)
minimize.Text = "-"
minimize.TextScaled = true
minimize.Font = Enum.Font.GothamBold
minimize.BackgroundColor3 = Color3.fromRGB(60,60,60)
minimize.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner",minimize).CornerRadius = UDim.new(0,8)

local content = Instance.new("Frame",main)
content.Size = UDim2.new(1,0,1,-35)
content.Position = UDim2.new(0,0,0,35)
content.BackgroundTransparency = 1

-- ================= DRAG =================

local dragging = false
local dragStart
local startPos

topBar.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1
	or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = main.Position
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement
	or input.UserInputType == Enum.UserInputType.Touch) then
		local delta = input.Position - dragStart
		main.Position = UDim2.new(startPos.X.Scale,startPos.X.Offset + delta.X,startPos.Y.Scale,startPos.Y.Offset + delta.Y)
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1
	or input.UserInputType == Enum.UserInputType.Touch then
		dragging = false
	end
end)

-- ================= MINIMIZE =================

local minimized = false
local originalSize = main.Size

minimize.MouseButton1Click:Connect(function()
	minimized = not minimized
	
	if minimized then
		content.Visible = false
		blur.Size = 0
		TweenService:Create(main,TweenInfo.new(0.25),{Size = UDim2.new(originalSize.X.Scale,originalSize.X.Offset,0,35)}):Play()
		minimize.Text = "+"
	else
		TweenService:Create(main,TweenInfo.new(0.25),{Size = originalSize}):Play()
		task.wait(0.25)
		content.Visible = true
		blur.Size = 20
		minimize.Text = "-"
	end
end)

-- ================= KEY CHECK =================

submit.MouseButton1Click:Connect(function()
	if keyBox.Text == correctKey then
		keyFrame:Destroy()
		main.Visible = true
		blur.Size = 20
	else
		keyTitle.Text = "wrong"
		keyTitle.TextColor3 = Color3.new(1,0,0)
	end
end)

-- ================= BUTTON CREATOR =================

local function makeButton(text,y,callback)
	local btn = Instance.new("TextButton",content)
	btn.Size = UDim2.new(0.8,0,0,40)
	btn.Position = UDim2.new(0.1,0,0,y)
	btn.Text = text
	btn.TextScaled = true
	btn.Font = Enum.Font.GothamBold
	btn.BackgroundColor3 = Color3.fromRGB(50,50,50)
	btn.TextColor3 = Color3.new(1,1,1)
	Instance.new("UICorner",btn).CornerRadius = UDim.new(0,10)
	btn.MouseButton1Click:Connect(callback)
end

-- ================= FLY (REAL MOVEMENT) =================

local flying = false
local flyConnection

makeButton("Toggle Fly",20,function()
	flying = not flying
	local char = getChar()
	local hrp = char:WaitForChild("HumanoidRootPart")
	local humanoid = char:WaitForChild("Humanoid")

	if flying then
		humanoid:ChangeState(Enum.HumanoidStateType.Physics)

		flyConnection = RunService.RenderStepped:Connect(function()
			local moveDir = humanoid.MoveDirection
			hrp.Velocity = moveDir * 60
		end)
	else
		if flyConnection then flyConnection:Disconnect() end
	end
end)

-- ================= SPEED & JUMP =================

makeButton("Speed +10",80,function()
	getChar().Humanoid.WalkSpeed += 10
end)

makeButton("Jump +20",130,function()
	getChar().Humanoid.JumpPower += 20
end)

-- ================= SPIN =================

makeButton("Spin Rapid",180,function()
	local hrp = getChar():WaitForChild("HumanoidRootPart")
	local spin = Instance.new("BodyAngularVelocity",hrp)
	spin.MaxTorque = Vector3.new(math.huge,math.huge,math.huge)
	spin.AngularVelocity = Vector3.new(0,400,0)
	task.wait(2)
	spin:Destroy()
end)

-- ================= ESP =================

local espEnabled = false

local function removeESP()
	for _,plr in pairs(Players:GetPlayers()) do
		if plr.Character and plr.Character:FindFirstChild("Head") then
			local head = plr.Character.Head
			if head:FindFirstChild("NameESP") then
				head.NameESP:Destroy()
			end
		end
	end
end

local function addESP(plr)
	if plr == player then return end
	
	local function apply(char)
		if not espEnabled then return end
		local head = char:WaitForChild("Head")
		if head:FindFirstChild("NameESP") then return end
		
		local bill = Instance.new("BillboardGui",head)
		bill.Name = "NameESP"
		bill.Size = UDim2.new(0,100,0,40)
		bill.StudsOffset = Vector3.new(0,2,0)
		bill.AlwaysOnTop = true
		
		local label = Instance.new("TextLabel",bill)
		label.Size = UDim2.new(1,0,1,0)
		label.BackgroundTransparency = 1
		label.Text = plr.Name
		label.TextScaled = true
		label.Font = Enum.Font.GothamBold
		label.TextColor3 = Color3.new(1,0,0)
	end
	
	if plr.Character then apply(plr.Character) end
	plr.CharacterAdded:Connect(apply)
end

makeButton("Toggle ESP",230,function()
	espEnabled = not espEnabled
	if espEnabled then
		for _,plr in pairs(Players:GetPlayers()) do
			addESP(plr)
		end
	else
		removeESP()
	end
end)
