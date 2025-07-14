# LuaPortfolio

**HERE IS THE SYSTEM (its a mining system)**

1ST SCRIPT(TOOL SCRIPT):

local ores = game.Workspace.Ores
local tool = script.Parent

local miningspeed = 80
local pickaxefortune = 2

local defaultMiningTimes = {
	CoalOre = 20,
	IronOre = 30,
	GoldOre = 60,
	JadeOre = 120
}

tool.Equipped:Connect(function()
	for _, ore in pairs(ores:GetChildren()) do
		local rock = ore:FindFirstChild("Rock")
		local prompt = rock and rock:FindFirstChildWhichIsA("ProximityPrompt")
		local miningTime = ore:GetAttribute("MiningTime") or defaultMiningTimes[ore.Name]
		local fortune = ore:GetAttribute("Fortune")
		fortune = pickaxefortune

		if miningTime and prompt then
			local adjustedTime
			if miningspeed >= miningTime then
				adjustedTime = 0.1
			else
				adjustedTime = miningTime - miningspeed
			end

			prompt.HoldDuration = adjustedTime
		end
	end
end)

tool.Unequipped:Connect(function()
	for _, ore in pairs(ores:GetChildren()) do
		local originalTime = defaultMiningTimes[ore.Name]

		if originalTime then
			ore:SetAttribute("MiningTime", originalTime)

			local rock = ore:FindFirstChild("Rock")
			local prompt = rock and rock:FindFirstChildWhichIsA("ProximityPrompt")
			if prompt then
				prompt.HoldDuration = originalTime
			end
		end
	end
end)

2ND (SERVER SCRIPT):

local proximprompt = script.Parent
local rock = proximprompt.Parent
local coalOre = rock.Parent
local actualore = game.ReplicatedStorage.ores.Coal
local fortuneAttribute = proximprompt:GetAttribute("Fortune")

local respawntime = 10

proximprompt.Triggered:Connect(function(player)
	for _, descendant in pairs(coalOre:GetDescendants()) do
		if descendant:IsA("BasePart") then
			descendant.Transparency = 1
			descendant.CanCollide = false
		end
	end

	for _, child in pairs(rock:GetChildren()) do
		if child.Name == "Texture" then
			if child:IsA("Decal") or child:IsA("Texture") then
				child.Transparency = 1
			elseif child:IsA("BasePart") then
				child.Transparency = 1
				child.CanCollide = false
			elseif child:IsA("SurfaceAppearance") then
				child.Enabled = false
			end
		end
	end
	local fortuneAttribute = coalOre:GetAttribute("Fortune")
	if typeof(fortuneAttribute) ~= "number" then
		fortuneAttribute = 1
	end

	local amount = 1 
	if fortuneAttribute == 1 then
		amount = 2
	elseif fortuneAttribute == 2 then
		amount = 3
	elseif fortuneAttribute > 2 then
		amount = 4
	end

	for i = 1, amount do
		local oreClone = actualore:Clone()
		oreClone.Parent = player.Backpack
	end


	proximprompt.Enabled = false

	task.delay(respawntime, function()
		for _, descendant in pairs(coalOre:GetDescendants()) do
			if descendant:IsA("BasePart") then
				descendant.Transparency = 0
				descendant.CanCollide = true
			end
		end

		for _, child in pairs(rock:GetChildren()) do
			if child.Name == "Texture" then
				if child:IsA("Decal") or child:IsA("Texture") then
					child.Transparency = 0
				elseif child:IsA("BasePart") then
					child.Transparency = 0
					child.CanCollide = true
				elseif child:IsA("SurfaceAppearance") then
					child.Enabled = true
				end
			end
		end

		proximprompt.Enabled = true
	end)
end)

THIRD SCRIPT (SELL SCRIPT):

local button = script.Parent
local player = game.Players.LocalPlayer
local RE = game.ReplicatedStorage.MoneyRE

button.MouseButton1Click:Connect(function()
	local backpack = player.Backpack
	local leaderstats = player:WaitForChild("leaderstats")
	local money = leaderstats:WaitForChild("Cash")
	local total = 0

	for _, tool in ipairs(backpack:GetChildren()) do
		if tool:IsA("Tool") then
			local SellableAttribute = tool:GetAttribute("Sellable")
			local SellValueAttribute = tool:GetAttribute("SellValue")

			if SellableAttribute == true and SellValueAttribute and SellValueAttribute > 0 then
				total += SellValueAttribute
				tool:Destroy()
			end
		end
	end

	RE:FireServer(total)
end)



