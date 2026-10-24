local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer

-- Configuração de Cores
local COLOR_MURDER = Color3.fromRGB(255, 0, 0)     -- Vermelho (Assassino)
local COLOR_SHERIFF = Color3.fromRGB(0, 100, 255)   -- Azul (Xerife)
local COLOR_INNOCENT = Color3.fromRGB(0, 255, 100)  -- Verde (Inocente)
local COLOR_DROPPED_GUN = Color3.fromRGB(170, 0, 255) -- Roxo (Arma no Chão)

-- Nomes das armas para identificação de papéis
local KNIFE_NAMES = {"Knife", "Faca"}
local GUN_NAMES = {"Revolver", "Gun", "Arma", "Sheriff"}

-- Função para verificar se o jogador possui algum item específico
local function hasItem(player, itemNames)
	local backpack = player:FindFirstChild("Backpack")
	local character = player.Character

	for _, name in ipairs(itemNames) do
		if backpack and backpack:FindFirstChild(name) then
			return true
		end
		if character and character:FindFirstChild(name) then
			return true
		end
	end
	return false
end

-- Função para identificar a função/cargo do jogador
local function getPlayerRole(player)
	if hasItem(player, KNIFE_NAMES) then
		return "Murderer"
	elseif hasItem(player, GUN_NAMES) then
		return "Sheriff"
	else
		return "Innocent"
	end
end

-- Função para aplicar Highlight genérico (através da parede)
local function applyHighlight(object, color, name)
	local highlight = object:FindFirstChild(name)
	if not highlight then
		highlight = Instance.new("Highlight")
		highlight.Name = name
		highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop -- Ver através da parede
		highlight.FillTransparency = 0.4
		highlight.OutlineTransparency = 0
		highlight.Parent = object
	end
	
	highlight.FillColor = color
	highlight.OutlineColor = color
end

-- Função para atualizar o ESP nos Jogadores
local function applyPlayerESP(player)
	if player == LocalPlayer then return end

	local character = player.Character
	if not character then return end

	local role = getPlayerRole(player)
	local color = COLOR_INNOCENT

	if role == "Murderer" then
		color = COLOR_MURDER
	elseif role == "Sheriff" then
		color = COLOR_SHERIFF
	end

	applyHighlight(character, color, "RoleESP")
end

-- Função para procurar e aplicar ESP em Armas caídas na Workspace
local function scanDroppedGuns()
	for _, item in ipairs(Workspace:GetChildren()) do
		-- Verifica se o objeto na Workspace é uma arma (Tool ou Part com nome de arma)
		if item:IsA("Tool") or item:IsA("Model") then
			for _, gunName in ipairs(GUN_NAMES) do
				if item.Name == gunName or item.Name:find("Gun") or item.Name:find("Revolver") then
					applyHighlight(item, COLOR_DROPPED_GUN, "GunESP")
				end
			end
		elseif item:IsA("BasePart") and (item.Name == "GunDrop" or item.Name == "Gun") then
			-- Alguns mapas usam uma Part chamada 'GunDrop'
			applyHighlight(item, COLOR_DROPPED_GUN, "GunESP")
		end
	end
end

-- Loop contínuo para atualizar os jogadores e as armas no chão em tempo real
RunService.Heartbeat:Connect(function()
	-- ESP de Jogadores
	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
			applyPlayerESP(player)
		end
	end
	
	-- ESP da Arma Dropada no Chão
	scanDroppedGuns()
end)

-- Limpeza ao fechar/reiniciar
Players.PlayerRemoving:Connect(function(player)
	if player.Character and player.Character:FindFirstChild("RoleESP") then
		player.Character.RoleESP:Destroy()
	end
end)
