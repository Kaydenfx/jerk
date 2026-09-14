local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

local plr = Players.LocalPlayer
local char = plr.Character or plr.CharacterAdded:Wait()
local hum = char:WaitForChild("Humanoid")
local anim = hum:WaitForChild("Animator")

-- Eski UI ve Animasyon Temizliği
if workspace:FindFirstChild("aaa") then workspace.aaa:Destroy() end
local parentTarget = plr:WaitForChild("PlayerGui") -- Güvenli UI hedefi (PlayerGui)
if parentTarget:FindFirstChild("MiniAnimUI") then parentTarget.MiniAnimUI:Destroy() end

-- Animasyon Kurulumu
local isR15 = hum.RigType == Enum.HumanoidRigType.R15
local animId = isR15 and "rbxassetid://698251653" or "rbxassetid://72042024"

local animation = Instance.new("Animation")
animation.Name = "aaa"
animation.AnimationId = animId
animation.Parent = workspace

local animTrack = anim:LoadAnimation(animation)
animTrack.Priority = Enum.AnimationPriority.Action

local isPlaying = false
local isMinimized = false

-- --- ÇOK KÜÇÜK VE TAŞINABİLİR MODERN UI ---
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "MiniAnimUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = parentTarget

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 100, 0, 32) -- Çok küçük boyut
mainFrame.Position = UDim2.new(0.85, 0, 0.5, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(18, 18, 26)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.ClipsDescendants = true
mainFrame.Parent = screenGui

local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(0, 8)
uiCorner.Parent = mainFrame

local uiStroke = Instance.new("UIStroke")
uiStroke.Color = Color3.fromRGB(0, 170, 255)
uiStroke.Thickness = 1.2
uiStroke.Parent = mainFrame

-- UI SÜRÜKLENME (DRAGGABLE) MANTIĞI
local dragging, dragInput, dragStart, startPos

local function update(input)
	local delta = input.Position - dragStart
	mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

mainFrame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = mainFrame.Position
		
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)

mainFrame.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
		dragInput = input
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if input == dragInput and dragging then
		update(input)
	end
end)

-- OYNAT / DURDUR BUTONU
local toggleBtn = Instance.new("TextButton")
toggleBtn.Name = "ToggleBtn"
toggleBtn.Size = UDim2.new(1, -26, 1, -6)
toggleBtn.Position = UDim2.new(0, 3, 0, 3)
toggleBtn.BackgroundColor3 = Color3.fromRGB(28, 28, 40)
toggleBtn.Text = "▶ OYNAT"
toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleBtn.TextSize = 10
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.AutoButtonColor = false
toggleBtn.Parent = mainFrame

local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(0, 6)
btnCorner.Parent = toggleBtn

-- KÜÇÜLTME (MINIMIZE) BUTONU (-)
local minBtn = Instance.new("TextButton")
minBtn.Name = "MinBtn"
minBtn.Size = UDim2.new(0, 18, 0, 18)
minBtn.Position = UDim2.new(1, -21, 0.5, -9)
minBtn.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
minBtn.Text = "-"
minBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
minBtn.TextSize = 12
minBtn.Font = Enum.Font.GothamBold
minBtn.AutoButtonColor = false
minBtn.Parent = mainFrame

local minCorner = Instance.new("UICorner")
minCorner.CornerRadius = UDim.new(0, 5)
minCorner.Parent = minBtn

-- ANIMASYON DÖNGÜSÜ
local function startLoopingAnimation()
	task.spawn(function()
		while isPlaying and animTrack do
			animTrack:Play()
			animTrack:AdjustSpeed(0.8)
			animTrack.TimePosition = 0.6
			
			local startTime = tick()
			while isPlaying and (tick() - startTime < 0.15) do
				task.wait()
			end
		end
		if animTrack then
			animTrack:Stop()
		end
	end)
end

local function toggleAnimation()
	isPlaying = not isPlaying

	if isPlaying then
		toggleBtn.Text = "⏹ DURDUR"
		TweenService:Create(toggleBtn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(200, 35, 55)}):Play()
		TweenService:Create(uiStroke, TweenInfo.new(0.2), {Color = Color3.fromRGB(255, 60, 80)}):Play()
		startLoopingAnimation()
	else
		toggleBtn.Text = "▶ OYNAT"
		TweenService:Create(toggleBtn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(28, 28, 40)}):Play()
		TweenService:Create(uiStroke, TweenInfo.new(0.2), {Color = Color3.fromRGB(0, 170, 255)}):Play()
		if animTrack then
			animTrack:Stop()
		end
	end
end

toggleBtn.MouseButton1Click:Connect(toggleAnimation)

-- KÜÇÜLT / BÜYÜT MANTIĞI
minBtn.MouseButton1Click:Connect(function()
	isMinimized = not isMinimized
	if isMinimized then
		toggleBtn.Visible = false
		minBtn.Text = "+"
		minBtn.Position = UDim2.new(0.5, -9, 0.5, -9)
		TweenService:Create(mainFrame, TweenInfo.new(0.2), {Size = UDim2.new(0, 26, 0, 26)}):Play()
	else
		minBtn.Text = "-"
		minBtn.Position = UDim2.new(1, -21, 0.5, -9)
		TweenService:Create(mainFrame, TweenInfo.new(0.2), {Size = UDim2.new(0, 100, 0, 32)}):Play()
		task.wait(0.15)
		toggleBtn.Visible = true
	end
end)

-- Karakter Yenilendiğinde Bağlantıları Güncelleme
plr.CharacterAdded:Connect(function(newChar)
	isPlaying = false
	char = newChar
	hum = char:WaitForChild("Humanoid")
	anim = hum:WaitForChild("Animator")
	animTrack = anim:LoadAnimation(animation)
	animTrack.Priority = Enum.AnimationPriority.Action
	
	toggleBtn.Text = "▶ OYNAT"
	toggleBtn.BackgroundColor3 = Color3.fromRGB(28, 28, 40)
	uiStroke.Color = Color3.fromRGB(0, 170, 255)
end)
