local player = game.Players.LocalPlayer
local petsFolder = workspace:FindFirstChild("Pets") or workspace -- À adapter selon où sont tes pets
local equippedPetsFolder = player:FindFirstChild("EquippedPets")

-- Création d'un dossier pour les pets équipés si inexistant
if not equippedPetsFolder then
    equippedPetsFolder = Instance.new("Folder")
    equippedPetsFolder.Name = "EquippedPets"
    equippedPetsFolder.Parent = player
end

-- Fonction pour "équiper" un pet (copie visuelle + donne fly + ride)
local function equipPet(pet)
    if not pet or not pet:IsA("Model") then return end

    -- Clone le pet dans le dossier EquippedPets (visuel)
    local petClone = pet:Clone()
    petClone.Parent = equippedPetsFolder

    -- Positionne le pet sur le joueur (exemple simple)
    local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if hrp then
        petClone:SetPrimaryPartCFrame(hrp.CFrame * CFrame.new(2, 0, 0))
    end

    -- Ajoute une "monture" : attache le pet au HumanoidRootPart
    local weld = Instance.new("WeldConstraint")
    weld.Part0 = hrp
    weld.Part1 = petClone.PrimaryPart
    weld.Parent = petClone.PrimaryPart

    -- Ajout d'un script pour voler (fly) - simplifié
    local flyScript = Instance.new("Script")
    flyScript.Source = [[
        local player = game.Players.LocalPlayer
        local hrp = player.Character and player.Character:WaitForChild("HumanoidRootPart")
        local flying = false

        local UserInputService = game:GetService("UserInputService")

        UserInputService.InputBegan:Connect(function(input, gameProcessed)
            if gameProcessed then return end
            if input.KeyCode == Enum.KeyCode.F then
                flying = not flying
                print("Fly mode toggled:", flying)
            end
        end)

        game:GetService("RunService").RenderStepped:Connect(function()
            if flying and hrp then
                hrp.Velocity = Vector3.new(0, 50, 0) -- monte en l'air
            end
        end)
    ]]
    flyScript.Parent = petClone

    -- Ajoute effet visuel de vol (ex: particules)
    local particle = Instance.new("ParticleEmitter")
    particle.Color = ColorSequence.new(Color3.new(0,1,1))
    particle.LightEmission = 1
    particle.Size = NumberSequence.new(1)
    particle.Rate = 20
    particle.Parent = petClone.PrimaryPart
end

-- Équipe tous les pets
for _, pet in pairs(petsFolder:GetChildren()) do
    equipPet(pet)
end
