local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")

local plr = Players.LocalPlayer
local char = plr.Character or plr.CharacterAdded:Wait()
local hum = char:WaitForChild("Humanoid")
local anim = hum:WaitForChild("Animator")

-- Eski UI ve Animasyonları Temizle
if workspace:FindFirstChild("aaa") then workspace.aaa:Destroy() end
if CoreGui:FindFirstChild("ModernAnimUI") then CoreGui.ModernAnimUI:Destroy() end

-- Animasyon ID Seçimi
local isR15 = hum.RigType == Enum.HumanoidRigType.R15
local animId = isR15 and "rbxassetid://698251653" or "rbxassetid://72042024"

local animation = Instance.new("Animation")
animation.Name = "aaa"
animation.AnimationId = animId
animation.Parent = workspace

local animTrack = anim:LoadAnimation(animation)
animTrack.Priority = Enum.AnimationPriority.Action

local isPlaying = false

-- SAĞ TARAFA HİZALANMIŞ MODERN UI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ModernAnimUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = CoreGui

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 140, 0, 48)
-- Ekranda Sağ Taraf (X: %90, Y: %50)
mainFrame.Position = UDim2.new(0.9, -150, 0.5, -24)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(0, 10)
uiCorner.Parent = mainFrame

local uiStroke = Instance.new("UIStroke")
uiStroke.Color = Color3.fromRGB(0, 170, 255)
uiStroke.Thickness = 1.5
uiStroke.Parent = mainFrame

local toggleBtn = Instance.new("TextButton")
toggleBtn.Name = "ToggleBtn"
toggleBtn.Size = UDim2.new(1, -8, 1, -8)
toggleBtn.Position = UDim2.new(0, 4, 0, 4)
toggleBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
toggleBtn.Text = "OYNAT"
toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleBtn.TextSize = 13
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.AutoButtonColor = false
toggleBtn.Parent = mainFrame

local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(0, 8)
btnCorner.Parent = toggleBtn

-- LOOP DÖNGÜSÜ VE ANİMASYON KONTROLÜ
local function startLoopingAnimation()
    task.spawn(function()
        while isPlaying and animTrack do
            animTrack:Play()
            animTrack:AdjustSpeed(0.8)
            animTrack.TimePosition = 0.6
            
            -- Belirtilen kare aralığında döngü yap
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
        toggleBtn.Text = "DURDUR"
        TweenService:Create(toggleBtn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(220, 40, 60)}):Play()
        TweenService:Create(uiStroke, TweenInfo.new(0.2), {Color = Color3.fromRGB(255, 60, 80)}):Play()
        startLoopingAnimation()
    else
        toggleBtn.Text = "OYNAT"
        TweenService:Create(toggleBtn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(30, 30, 45)}):Play()
        TweenService:Create(uiStroke, TweenInfo.new(0.2), {Color = Color3.fromRGB(0, 170, 255)}):Play()
        if animTrack then
            animTrack:Stop()
        end
    end
end

toggleBtn.MouseButton1Click:Connect(toggleAnimation)

-- Karakter Yenilendiğinde Bağlantıları Yenile
plr.CharacterAdded:Connect(function(newChar)
    isPlaying = false
    char = newChar
    hum = char:WaitForChild("Humanoid")
    anim = hum:WaitForChild("Animator")
    animTrack = anim:LoadAnimation(animation)
    animTrack.Priority = Enum.AnimationPriority.Action
    
    toggleBtn.Text = "OYNAT"
    toggleBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
    uiStroke.Color = Color3.fromRGB(0, 170, 255)
end)
