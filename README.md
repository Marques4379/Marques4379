-- Script de Hitbox Expander com Interface Simples
local ScreenGui = Instance.new("ScreenGui")
local MainFrame = Instance.new("Frame")
local ToggleBtn = Instance.new("TextButton")

-- Configurações da GUI
ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
MainFrame.Name = "HitboxHub"
MainFrame.Size = UDim2.new(0, 150, 0, 50)
MainFrame.Position = UDim2.new(0.1, 0, 0.1, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
MainFrame.Active = true
MainFrame.Draggable = true -- Facilita mover na tela

ToggleBtn.Parent = MainFrame
ToggleBtn.Size = UDim2.new(1, 0, 1, 0)
ToggleBtn.Text = "Hitbox: OFF"
ToggleBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
ToggleBtn.TextColor3 = Color3.new(1, 1, 1)

local active = false
local size = 20 -- Tamanho da hitbox

-- Função para expandir hitboxes
function expandHitboxes()
    while active do
        for _, player in pairs(game.Players:GetPlayers()) do
            if player ~= game.Players.LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                local hrp = player.Character.HumanoidRootPart
                hrp.Size = Vector3.new(size, size, size)
                hrp.Transparency = 0.7 -- Fica visível mas sem atrapalhar
                hrp.BrickColor = BrickColor.new("Really blue")
                hrp.CanCollide = false -- Evita bugs de colisão kkk
            end
        end
        task.wait(1)
    end
end

-- Ativar/Desativar
ToggleBtn.MouseButton1Click:Connect(function()
    active = not active
    if active then
        ToggleBtn.Text = "Hitbox: ON"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
        task.spawn(expandHitboxes)
    else
        ToggleBtn.Text = "Hitbox: OFF"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
        -- Volta ao normal
        for _, player in pairs(game.Players:GetPlayers()) do
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                player.Character.HumanoidRootPart.Size = Vector3.new(2, 2, 1)
                player.Character.HumanoidRootPart.Transparency = 1
            end
        end
    end
end)
