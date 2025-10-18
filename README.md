-- RedTopGlobal Auto Aim Script
-- By: Sla Hub

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")

-- Configurações do Auto Aim
local CurrentTarget = nil
local TargetLockDistance = 100
local AutoAimEnabled = false

-- Função principal do RedTopGlobal
function RedTopGlobal()
    if not AutoAimEnabled then return end
    if not LocalPlayer.Character then return end
    if not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then return end
    
    -- Verifica se o alvo atual ainda é válido
    if CurrentTarget then
        if CurrentTarget.Character and 
           CurrentTarget.Character:FindFirstChild("HumanoidRootPart") and 
           CurrentTarget.Character.Humanoid.Health > 0 then
            
            local distance = (LocalPlayer.Character.HumanoidRootPart.Position - CurrentTarget.Character.HumanoidRootPart.Position).Magnitude
            
            -- Mantém o lock se o alvo estiver dentro da distância
            if distance <= TargetLockDistance then
                local targetHRP = CurrentTarget.Character.HumanoidRootPart
                LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(
                    LocalPlayer.Character.HumanoidRootPart.Position,
                    Vector3.new(targetHRP.Position.X, LocalPlayer.Character.HumanoidRootPart.Position.Y, targetHRP.Position.Z)
                )
                return
            else
                -- Alvo saiu do range, reseta
                CurrentTarget = nil
            end
        else
            -- Alvo inválido, reseta
            CurrentTarget = nil
        end
    end
    
    -- Procura novo alvo se não há um travado
    if not CurrentTarget then
        local closestPlayer = nil
        local shortestDistance = math.huge
        
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and 
               player.Character and 
               player.Character:FindFirstChild("HumanoidRootPart") and 
               player.Character.Humanoid.Health > 0 then
                
                local distance = (LocalPlayer.Character.HumanoidRootPart.Position - player.Character.HumanoidRootPart.Position).Magnitude
                
                if distance < shortestDistance and distance <= TargetLockDistance then
                    closestPlayer = player
                    shortestDistance = distance
                end
            end
        end
        
        if closestPlayer then
            CurrentTarget = closestPlayer
        end
    end
end

-- Interface gráfica
local OrionLib = loadstring(game:HttpGet(('https://raw.githubusercontent.com/shlexware/Orion/main/source')))()
local Window = OrionLib:MakeWindow({
    Name = "RedTopGlobal - Auto Aim", 
    HidePremium = false, 
    SaveConfig = true, 
    ConfigFolder = "RedTopGlobal"
})

-- Aba principal
local MainTab = Window:MakeTab({
    Name = "Auto Aim",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

-- Status do alvo
local TargetStatus = MainTab:AddLabel("Alvo: Nenhum")

-- Atualiza o status do alvo
function UpdateTargetStatus()
    if CurrentTarget then
        local distance = (LocalPlayer.Character.HumanoidRootPart.Position - CurrentTarget.Character.HumanoidRootPart.Position).Magnitude
        TargetStatus:Set("Alvo: " .. CurrentTarget.Name .. " | Distância: " .. math.floor(distance) .. " studs")
    else
        TargetStatus:Set("Alvo: Nenhum")
    end
end

-- Toggle do Auto Aim
MainTab:AddToggle({
    Name = "Ativar RedTopGlobal",
    Default = false,
    Callback = function(Value)
        AutoAimEnabled = Value
        if not Value then
            CurrentTarget = nil
            UpdateTargetStatus()
        end
    end    
})

-- Controle de distância
MainTab:AddSlider({
    Name = "Distância de Lock",
    Min = 50,
    Max = 500,
    Default = 100,
    Color = Color3.fromRGB(255, 0, 0),
    Increment = 10,
    ValueName = "studs",
    Callback = function(Value)
        TargetLockDistance = Value
    end    
})

-- Botão para forçar reset do alvo
MainTab:AddButton({
    Name = "Resetar Alvo Atual",
    Callback = function()
        CurrentTarget = nil
        UpdateTargetStatus()
    end
})

-- Informações
MainTab:AddParagraph("Como funciona:", "• Encontra o jogador mais próximo\n• Trava no primeiro alvo encontrado\n• Mantém lock até 100 studs de distância\n• Reset automático quando alvo sai do range")

-- Loop principal
RunService.Heartbeat:Connect(function()
    RedTopGlobal()
    UpdateTargetStatus()
end)

-- Inicializar
OrionLib:MakeNotification({
    Name = "RedTopGlobal Carregado!",
    Content = "Sistema de Auto Aim ativado com sucesso",
    Image = "rbxassetid://4483345998",
    Time = 5
})

OrionLib:Init()
