-- SERVICES
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local plr = Players.LocalPlayer

-- CONFIGURAÇÕES
local ESP_ON, AIM_ON, SPEED_ON, JUMP_ON = true, false, false, false
local highlights = {}

local normalWalkSpeed = 16
local boostedSpeed = normalWalkSpeed * 1.5
local normalJumpPower = 50
local boostedJump = normalJumpPower * 1.5

-- CORES ESP
local ESPColors = {
    ["Roxo Claro"] = Color3.fromRGB(186, 85, 211),
    ["Rosa"] = Color3.fromRGB(255, 105, 180),
    ["Preto"] = Color3.fromRGB(0, 0, 0),
    ["RGB"] = Color3.fromRGB(255, 0, 0)
}
local currentESPColorName = "Roxo Claro"
local ESPColor = ESPColors[currentESPColorName]

-- FUNÇÕES ESP
local function criarOuAtualizarHighlight(player)
    if player == plr then return end
    if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then return end

    local hl = highlights[player]
    if not hl or not hl.Parent then
        hl = Instance.new("Highlight")
        hl.Name = "ESP_Highlight"
        hl.Adornee = player.Character
        hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        hl.OutlineTransparency = 0.2
        hl.FillTransparency = 0.6
        hl.OutlineColor = Color3.new(1,1,1)
        hl.FillColor = ESPColor
        hl.Parent = workspace
        highlights[player] = hl
    else
        hl.Adornee = player.Character
        hl.FillColor = ESPColor
    end
end

local function removerHighlight(player)
    if highlights[player] then
        highlights[player]:Destroy()
        highlights[player] = nil
    end
end

local function aplicarESP(player)
    if ESP_ON then
        criarOuAtualizarHighlight(player)
    end

    player.CharacterAdded:Connect(function()
        task.wait(0.05)
        if ESP_ON then
            criarOuAtualizarHighlight(player)
        end
    end)

    player.AncestryChanged:Connect(function(_, parent)
        if not parent then
            removerHighlight(player)
        end
    end)
end

for _, player in pairs(Players:GetPlayers()) do
    aplicarESP(player)
end

Players.PlayerAdded:Connect(function(player)
    aplicarESP(player)
end)

-- GUI
local gui = Instance.new("ScreenGui", plr:WaitForChild("PlayerGui"))
gui.ResetOnSpawn = false

local panel = Instance.new("Frame", gui)
panel.Size = UDim2.new(0, 250, 0, 250)
panel.Position = UDim2.new(0, 20, 0, 70)
panel.BackgroundColor3 = Color3.fromRGB(50,50,50)
panel.Visible = true

local GUI_VISIBLE = true

local toggleGuiBtn = Instance.new("TextButton", gui)
toggleGuiBtn.Size = UDim2.new(0, 120, 0, 40)
toggleGuiBtn.Position = UDim2.new(0, 20, 0, 20)
toggleGuiBtn.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
toggleGuiBtn.TextColor3 = Color3.new(1, 1, 1)
toggleGuiBtn.Text = "Esconder GUI"

toggleGuiBtn.MouseButton1Click:Connect(function()
    GUI_VISIBLE = not GUI_VISIBLE
    panel.Visible = GUI_VISIBLE
    toggleGuiBtn.Text = GUI_VISIBLE and "Esconder GUI" or "Mostrar GUI"
end)

-- FUNÇÃO BOTÃO
local function criarBotao(texto, posY)
    local btn = Instance.new("TextButton", panel)
    btn.Size = UDim2.new(1,0,0.2,0)
    btn.Position = UDim2.new(0,0,posY,0)
    btn.Text = texto
    btn.BackgroundColor3 = Color3.fromRGB(35,35,35)
    btn.TextColor3 = Color3.new(1,1,1)
    return btn
end

local btnESP = criarBotao("ESP ON", 0)
local btnAIM = criarBotao("AIM OFF", 0.2)
local btnSpeed = criarBotao("SPEED OFF", 0.4)
local btnJump = criarBotao("JUMP OFF", 0.6)
local btnESPColor = criarBotao("Cor: "..currentESPColorName, 0.8)

-- FUNÇÕES DOS BOTÕES
btnESP.MouseButton1Click:Connect(function()
    ESP_ON = not ESP_ON
    btnESP.Text = ESP_ON and "ESP ON" or "ESP OFF"

    if not ESP_ON then
        for _, hl in pairs(highlights) do hl:Destroy() end
        highlights = {}
    else
        for _, player in pairs(Players:GetPlayers()) do aplicarESP(player) end
    end
end)

btnESPColor.MouseButton1Click:Connect(function()
    local names = {"Roxo Claro", "Rosa", "Preto", "RGB"}
    local index = table.find(names, currentESPColorName)
    index = index % #names + 1
    currentESPColorName = names[index]
    ESPColor = ESPColors[currentESPColorName]
    btnESPColor.Text = "Cor: "..currentESPColorName
end)

btnSpeed.MouseButton1Click:Connect(function()
    SPEED_ON = not SPEED_ON
    btnSpeed.Text = SPEED_ON and "SPEED ON" or "SPEED OFF"
    local char = plr.Character
    if char and char:FindFirstChild("Humanoid") then
        char.Humanoid.WalkSpeed = SPEED_ON and boostedSpeed or normalWalkSpeed
    end
end)

btnJump.MouseButton1Click:Connect(function()
    JUMP_ON = not JUMP_ON
    btnJump.Text = JUMP_ON and "JUMP ON" or "JUMP OFF"
    local char = plr.Character
    if char and char:FindFirstChild("Humanoid") then
        char.Humanoid.JumpPower = JUMP_ON and boostedJump or normalJumpPower
    end
end)

-- AIMBOT
local targetPlayer = nil
local MaxDistance = 50
local cam = workspace.CurrentCamera
local lerpAmount = 0.15

local function getPlayerAimingAt()
    local origin = cam.CFrame.Position
    local direction = cam.CFrame.LookVector * MaxDistance
    local raycastParams = RaycastParams.new()
    raycastParams.FilterDescendantsInstances = {plr.Character}
    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist

    local raycastResult = workspace:Raycast(origin, direction, raycastParams)
    if raycastResult and raycastResult.Instance then
        local part = raycastResult.Instance
        local character = part:FindFirstAncestorOfClass("Model")
        if character and character ~= plr.Character then
            local player = Players:GetPlayerFromCharacter(character)
            if player and player.Character and player.Character:FindFirstChild("Head") then
                local headPos = player.Character.Head.Position
                local dist = (headPos - origin).Magnitude
                if dist <= MaxDistance then
                    return player
                end
            end
        end
    end
    return nil
end

plr.CharacterAdded:Connect(function(char)
    targetPlayer = nil
end)

RunService.RenderStepped:Connect(function()
    if AIM_ON then
        local playerLookingAt = getPlayerAimingAt()
        if playerLookingAt then targetPlayer = playerLookingAt end

        if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("Head") then
            local headPos = targetPlayer.Character.Head.Position
            local camPos = cam.CFrame.Position
            local currentLook = cam.CFrame.LookVector
            local desiredDir = (headPos - camPos).Unit
            local newDir = currentLook:Lerp(desiredDir, lerpAmount)
            cam.CFrame = CFrame.new(camPos, camPos + newDir)
        else
            targetPlayer = nil
        end
    end
end)

btnAIM.MouseButton1Click:Connect(function()
    AIM_ON = not AIM_ON
    btnAIM.Text = AIM_ON and "AIM ON" or "AIM OFF"
end)

-- Atualiza a cor do ESP dinamicamente (RGB)
RunService.RenderStepped:Connect(function()
    if ESP_ON then
        for player, hl in pairs(highlights) do
            if hl and hl.Parent then
                if currentESPColorName == "RGB" then
                    hl.FillColor = Color3.fromHSV(tick() % 1,1,1)
                else
                    hl.FillColor = ESPColor
                end
            end
        end
    end
end)
