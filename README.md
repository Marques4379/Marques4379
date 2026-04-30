``lua
local player = game.Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- Interface Principal
local gui = Instance.new("ScreenGui", player.PlayerGui)
gui.Name = "MarquesHubUltimate"
gui.ResetOnSpawn = false

local hub = Instance.new("Frame", gui)
hub.Size = UDim2.new(0, 350, 0, 250)
hub.Position = UDim2.new(0.5, -175, 0.5, -125)
hub.BackgroundColor3 = Color3.fromRGB(255, 255, 255) -- Branco pra o degradê aparecer
hub.Active = true
hub.Draggable = true 
Instance.new("UICorner", hub).CornerRadius = Radius.new(0, 10)

-- [NOVO] Degradê Vermelho
local gradient = Instance.new("UIGradient", hub)
gradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(150, 0, 0)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(40, 0, 0))
})
gradient.Rotation = 45

-- [NOVO] Botão de Fechar Total
local closeAll = Instance.new("TextButton", hub)
closeAll.Size = UDim2.new(0, 30, 0, 30)
closeAll.Position = UDim2.new(1, -35, 0, 5)
closeAll.Text = "X"
closeAll.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
closeAll.TextColor3 = Color3.new(1, 1, 1)
Instance.new("UICorner", closeAll)
closeAll.MouseButton1Click:Connect(function() gui:Destroy() end)

-- [NOVO] Botão de Ajustar Tamanho (Resize)
local resizer = Instance.new("TextButton", hub)
resizer.Size = UDim2.new(0, 20, 0, 20)
resizer.Position = UDim2.new(1, -20, 1, -20)
resizer.Text = "↘"
resizer.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
resizer.TextColor3 = Color3.new(1, 1, 1)
Instance.new("UICorner", resizer)

local draggingSize = false
resizer.MouseButton1Down:Connect(function() draggingSize = true end)
UIS.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then draggingSize = false end end)

RunService.RenderStepped:Connect(function()
    if draggingSize then
        local mousePos = UIS:GetMouseLocation()
        local newSizeX = mousePos.X - hub.AbsolutePosition.X
        local newSizeY = (mousePos.Y - 36) - hub.AbsolutePosition.Y -- Offset da barra do Roblox
        hub.Size = UDim2.new(0, math.max(150, newSizeX), 0, math.max(100, newSizeY))
    end
end)

-- Sistema de Abas e Funções (Mantido e Melhorado)
local pages = {}
local tabsFrame = Instance.new("Frame", hub)
tabsFrame.Size = UDim2.new(1, 0, 0, 40)
tabsFrame.BackgroundTransparency = 1

local function createTab(name, pos)
    local btn = Instance.new("TextButton", tabsFrame)
    btn.Size = UDim2.new(0.33, 0, 1, 0)
    btn.Position = UDim2.new(pos, 0, 0, 0)
    btn.Text = name
    btn.BackgroundTransparency = 0.5
    btn.BackgroundColor3 = Color3.fromRGB(0,0,0)
    btn.TextColor3 = Color3.new(1, 1, 1)
    
    local page = Instance.new("ScrollingFrame", hub)
    page.Size = UDim2.new(1, -20, 1, -60)
    page.Position = UDim2.new(0, 10, 0, 45)
    page.BackgroundTransparency = 1
    page.Visible = (pos == 0)
    page.CanvasSize = UDim2.new(0, 0, 2, 0)
    pages[name] = page
    
    btn.MouseButton1Click:Connect(function()
        for _, p in pairs(pages) do p.Visible = false end
        page.Visible = true
    end)
end

createTab("Main", 0)
createTab("Player", 0.33)
createTab("Visual", 0.66)

local function makeBtn(parent, text, func)
    local b = Instance.new("TextButton", parent)
    b.Size = UDim2.new(1, 0, 0, 40)
    b.Text = text
    b.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    b.TextColor3 = Color3.new(1, 1, 1)
    b.BackgroundTransparency = 0.3
    Instance.new("UICorner", b)
    b.MouseButton1Click:Connect(func)
    local list = parent:FindFirstChild("UIListLayout") or Instance.new("UIListLayout", parent)
    list.Padding = UDim.new(0, 5)
end

-- VARIÁVEIS E LOGICA
local flying, infJump, hitboxActive = false, false, false
local hbSize = 15
local bv = Instance.new("BodyVelocity")

makeBtn(pages["Main"], "VOAR", function()
    flying = not flying
    bv.Parent = flying and player.Character.HumanoidRootPart or nil
end)

makeBtn(pages["Main"], "PULO INFINITO", function() infJump = not infJump end)

makeBtn(pages["Player"], "VELOCIDADE", function()
    local hum = player.Character:FindFirstChild("Humanoid")
    if hum then hum.WalkSpeed = (hum.WalkSpeed == 16) and 60 or 16 end
end)

-- [MELHORADO] Hitbox com Team Check
makeBtn(pages["Visual"], "HITBOX + TEAM CHECK", function()
    hitboxActive = not hitboxActive
    if not hitboxActive then
        for _, p in pairs(game.Players:GetPlayers()) do
            if p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                p.Character.HumanoidRootPart.Size = Vector3.new(2, 2, 1)
                p.Character.HumanoidRootPart.Transparency = 1
            end
        end
    end
end)

RunService.RenderStepped:Connect(function()
    local char = player.Character
    if flying and char and char:FindFirstChild("HumanoidRootPart") then
        bv.Parent = char.HumanoidRootPart
        bv.MaxForce = Vector3.new(1, 1, 1) * 100000
        bv.Velocity = char.Humanoid.MoveDirection * 60 + Vector3.new(0, 2, 0)
    end
    
    if hitboxActive then
        for _, p in pairs(game.Players:GetPlayers()) do
            -- [TEAM CHECK AQUI] [2]
            if p ~= player and p.Team ~= player.Team and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                local hrp = p.Character.HumanoidRootPart
                hrp.Size = Vector3.new(hbSize, hbSize, hbSize)
                hrp.Transparency = 0.7
                hrp.CanCollide = false
            end
        end
    end
end)

UIS.JumpRequest:Connect(function()
    if infJump then player.Character.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping) end
end)
