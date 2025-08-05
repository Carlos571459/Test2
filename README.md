local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local MarketplaceService = game:GetService("MarketplaceService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Criar ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "FloatingButtonGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

-- Botão flutuante
local button = Instance.new("Frame")
button.Name = "FloatingButton"
button.Size = UDim2.new(0, 60, 0, 60)
button.Position = UDim2.new(0.9, 0, 0.8, 0)
button.AnchorPoint = Vector2.new(0.5, 0.5)
button.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
button.BorderSizePixel = 0
button.ZIndex = 100
button.Parent = screenGui

local buttonCorner = Instance.new("UICorner")
buttonCorner.CornerRadius = UDim.new(0, 12)
buttonCorner.Parent = button

local buttonStroke = Instance.new("UIStroke")
buttonStroke.Color = Color3.fromRGB(230, 230, 230)
buttonStroke.Thickness = 2
buttonStroke.Parent = button

local image = Instance.new("ImageLabel")
image.Size = UDim2.new(0, 36, 0, 36)
image.Position = UDim2.new(0.5, 0, 0.5, 0)
image.AnchorPoint = Vector2.new(0.5, 0.5)
image.BackgroundTransparency = 1
image.Image = "rbxassetid://6031075938"
image.ImageColor3 = Color3.new(1, 1, 1)
image.ZIndex = 101
image.Parent = button

button.BackgroundTransparency = 1
button.Size = UDim2.new(0, 0, 0, 0)

local tweenIn = TweenService:Create(button, TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
	Size = UDim2.new(0, 60, 0, 60),
	BackgroundTransparency = 0,
	Position = UDim2.new(0.9, 0, 0.8, 0)
})
tweenIn:Play()

-- Botão transparente para clique/hover
local clickDetector = Instance.new("TextButton")
clickDetector.Size = UDim2.new(1, 0, 1, 0)
clickDetector.BackgroundTransparency = 1
clickDetector.Text = ""
clickDetector.ZIndex = 102
clickDetector.Parent = button

-- Som do clique do botão flutuante
local clickSound = Instance.new("Sound")
clickSound.SoundId = "rbxassetid://15675059323"
clickSound.Volume = 0.5
clickSound.Parent = clickDetector

local GuiManager = {}
GuiManager.spawnedFrame = nil
GuiManager.isMinimized = false
GuiManager.activeConnections = {}
GuiManager.activeTweens = {}

function GuiManager:CleanUp()
	for _, tween in pairs(self.activeTweens) do
		if tween then
			tween:Cancel()
		end
	end
	self.activeTweens = {}

	for _, connection in pairs(self.activeConnections) do
		if connection then
			connection:Disconnect()
		end
	end
	self.activeConnections = {}
end

function GuiManager:AddConnection(connection)
	table.insert(self.activeConnections, connection)
end

function GuiManager:AddTween(tween)
	table.insert(self.activeTweens, tween)
	return tween
end

-- Função para criar botões com hover e som de clique padronizados
function GuiManager:CreateButton(parent, size, position, text, color, zIndex)
	local btn = Instance.new("TextButton")
	btn.Size = size or UDim2.new(0, 120, 0, 45)
	btn.Position = position or UDim2.new(0, 0, 0, 0)
	btn.BackgroundColor3 = color or Color3.fromRGB(60, 60, 60)
	btn.Text = text or "Botão"
	btn.TextColor3 = Color3.new(1, 1, 1)
	btn.TextScaled = true
	btn.Font = Enum.Font.GothamBold
	btn.BorderSizePixel = 0
	btn.ZIndex = zIndex or 8
	btn.AutoButtonColor = false
	btn.Parent = parent

	local btnCorner = Instance.new("UICorner")
	btnCorner.CornerRadius = UDim.new(0, 8)
	btnCorner.Parent = btn

	local originalColor = btn.BackgroundColor3
	local hoverColor = Color3.new(
		math.clamp(originalColor.R + 0.1, 0, 1),
		math.clamp(originalColor.G + 0.1, 0, 1),
		math.clamp(originalColor.B + 0.1, 0, 1)
	)

	local enterConnection = btn.MouseEnter:Connect(function()
		if self.spawnedFrame and self.spawnedFrame.Parent then
			local tween = self:AddTween(TweenService:Create(btn, TweenInfo.new(0.2), {
				BackgroundColor3 = hoverColor
			}))
			tween:Play()
		end
	end)

	local leaveConnection = btn.MouseLeave:Connect(function()
		if self.spawnedFrame and self.spawnedFrame.Parent then
			local tween = self:AddTween(TweenService:Create(btn, TweenInfo.new(0.2), {
				BackgroundColor3 = originalColor
			}))
			tween:Play()
		end
	end)

	self:AddConnection(enterConnection)
	self:AddConnection(leaveConnection)

	-- Som do clique do botão
	local buttonSound = Instance.new("Sound")
	buttonSound.SoundId = "rbxassetid://15675059323"
	buttonSound.Volume = 0.5
	buttonSound.Parent = btn

	btn.MouseButton1Click:Connect(function()
		buttonSound:Play()
	end)

	return btn
end

-- Função para criar frame principal do painel
local function createMainFrame()
	local frame = Instance.new("Frame")
	frame.Size = UDim2.new(0, 0, 0, 0)  -- Start pequeno para animação
	frame.Position = UDim2.new(0.5, 0, 0.5, 0)
	frame.AnchorPoint = Vector2.new(0.5, 0.5)
	frame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
	frame.BorderSizePixel = 0
	frame.ZIndex = 10
	frame.ClipsDescendants = true
	frame.BackgroundTransparency = 1 -- Começa invisível
	frame.Parent = screenGui

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 14)
	corner.Parent = frame

	local stroke = Instance.new("UIStroke")
	stroke.Color = Color3.fromRGB(200, 200, 200)
	stroke.Thickness = 2
	stroke.Parent = frame

	return frame
end

-- Criar barra de título (igual ao seu código original)
local function createTitleBar(parent)
	local titleBar = Instance.new("Frame")
	titleBar.Size = UDim2.new(1, 0, 0, 38)
	titleBar.Position = UDim2.new(0, 0, 0, 0)
	titleBar.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
	titleBar.BorderSizePixel = 0
	titleBar.ZIndex = 12
	titleBar.Parent = parent

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 14)
	corner.Parent = titleBar

	local titleLabel = Instance.new("TextLabel")
	titleLabel.Size = UDim2.new(1, -90, 1, 0)
	titleLabel.Position = UDim2.new(0, 12, 0, 0)
	titleLabel.BackgroundTransparency = 1
	titleLabel.Text = "🎮 Painel de Controle"
	titleLabel.TextColor3 = Color3.new(1, 1, 1)
	titleLabel.TextScaled = true
	titleLabel.Font = Enum.Font.GothamBold
	titleLabel.TextXAlignment = Enum.TextXAlignment.Left
	titleLabel.ZIndex = 13
	titleLabel.Parent = titleBar

	return titleBar
end

-- As funções createWindowButton, createContainers e todo o restante do seu código seguem iguais,
-- com a adição da animação de abertura do GUI e som na função que abre o GUI


-- Função para tocar som de abertura
local function playOpenGuiSound()
	local sound = Instance.new("Sound")
	sound.SoundId = "rbxassetid://8968249849"
	sound.Volume = 0.6
	sound.Parent = screenGui
	sound:Play()
	sound.Ended:Connect(function()
		sound:Destroy()
	end)
end

-- Clique no botão flutuante para abrir/fechar painel
clickDetector.MouseButton1Click:Connect(function()
	-- Tocar som de clique no botão flutuante
	clickSound:Play()

	-- Se painel já aberto, fechar com animação e limpeza
	if GuiManager.spawnedFrame then
		GuiManager:CleanUp()

		local tweenOut = TweenService:Create(GuiManager.spawnedFrame, TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.In), {
			Size = UDim2.new(0, 0, 0, 0),
			BackgroundTransparency = 1
		})
		tweenOut:Play()
		tweenOut.Completed:Connect(function()
			if GuiManager.spawnedFrame then
				GuiManager.spawnedFrame:Destroy()
				GuiManager.spawnedFrame = nil
				GuiManager.isMinimized = false
			end
		end)
		return
	end

	-- Criar frame do painel
	GuiManager.spawnedFrame = createMainFrame()

	-- Tocar som de abertura do GUI
	playOpenGuiSound()

	-- Barra de título
	local titleBar = createTitleBar(GuiManager.spawnedFrame)

	-- Botões fechar e minimizar na barra de título
	local function createWindowButton(parent, text, bgColor, hoverColor)
		local btn = Instance.new("TextButton")
		btn.Size = UDim2.new(0, 28, 0, 28)
		btn.BackgroundColor3 = bgColor
		btn.Text = text
		btn.TextColor3 = (text == "−") and Color3.fromRGB(0, 0, 0) or Color3.new(1, 1, 1)
		btn.TextScaled = true
		btn.Font = Enum.Font.GothamBold
		btn.BorderSizePixel = 0
		btn.ZIndex = 14
		btn.Parent = parent

		local corner = Instance.new("UICorner")
		corner.CornerRadius = UDim.new(0, 5)
		corner.Parent = btn

		btn.MouseEnter:Connect(function()
			TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = hoverColor}):Play()
		end)

		btn.MouseLeave:Connect(function()
			TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = bgColor}):Play()
		end)

		-- Som de clique para esses botões
		local sound = Instance.new("Sound")
		sound.SoundId = "rbxassetid://15675059323"
		sound.Volume = 0.5
		sound.Parent = btn

		btn.MouseButton1Click:Connect(function()
			sound:Play()
		end)

		return btn
	end

	local minimizeButton = createWindowButton(titleBar, "−", Color3.fromRGB(255, 193, 7), Color3.fromRGB(255, 213, 47))
	minimizeButton.Position = UDim2.new(1, -58, 0.5, -14)

	local closeButton = createWindowButton(titleBar, "✕", Color3.fromRGB(220, 53, 69), Color3.fromRGB(240, 73, 89))
	closeButton.Position = UDim2.new(1, -28, 0.5, -14)

	local function createContainers(parent)
		local tabContainer = Instance.new("Frame")
		tabContainer.Size = UDim2.new(1, -24, 0, 48)
		tabContainer.Position = UDim2.new(0, 12, 0, 46)
		tabContainer.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
		tabContainer.BorderSizePixel = 0
		tabContainer.ZIndex = 11
		tabContainer.ClipsDescendants = true
		tabContainer.Parent = parent

		local tabCorner = Instance.new("UICorner")
		tabCorner.CornerRadius = UDim.new(0, 10)
		tabCorner.Parent = tabContainer

		local tabLayout = Instance.new("UIListLayout")
		tabLayout.FillDirection = Enum.FillDirection.Horizontal
		tabLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
		tabLayout.VerticalAlignment = Enum.VerticalAlignment.Center
		tabLayout.Padding = UDim.new(0, 12)
		tabLayout.Parent = tabContainer

		local contentContainer = Instance.new("Frame")
		contentContainer.Size = UDim2.new(1, -24, 1, -104)
		contentContainer.Position = UDim2.new(0, 12, 0, 96)
		contentContainer.BackgroundTransparency = 1
		contentContainer.ZIndex = 11
		contentContainer.ClipsDescendants = true
		contentContainer.Parent = parent

		return tabContainer, contentContainer
	end

	local tabContainer, contentContainer = createContainers(GuiManager.spawnedFrame)

	-- O resto do seu código com abas, conteúdos e funcionalidades permanece igual
	-- Lembre-se de chamar GuiManager:AddConnection para as conexões e GuiManager:AddTween para os tweens

	-- Exemplo de animação de entrada do frame main:
	local frameInTween = GuiManager:AddTween(TweenService:Create(GuiManager.spawnedFrame, TweenInfo.new(0.4, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
		Size = UDim2.new(0, 450, 0, 350),
		BackgroundTransparency = 0
	}))
	frameInTween:Play()

	-- ... Código restante das abas e funcionalidades igual ao script original ...
	-- Para manter a resposta focada e legível, não repito aqui as partes enormes já publicadas

	-- Botão minimizar
	local minimizeConn = minimizeButton.MouseButton1Click:Connect(function()
		GuiManager.isMinimized = not GuiManager.isMinimized
		if GuiManager.isMinimized then
			contentContainer.Visible = false
			tabContainer.Visible = false
			minimizeButton.Text = "□"

			local tween = GuiManager:AddTween(TweenService:Create(GuiManager.spawnedFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quad), {
				Size = UDim2.new(0, 450, 0, 38)
			}))
			tween:Play()
		else
			contentContainer.Visible = true
			tabContainer.Visible = true
			minimizeButton.Text = "−"

			local tween = GuiManager:AddTween(TweenService:Create(GuiManager.spawnedFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quad), {
				Size = UDim2.new(0, 450, 0, 350)
			}))
			tween:Play()
		end
	end)

	-- Botão fechar
	local closeConn = closeButton.MouseButton1Click:Connect(function()
		GuiManager:CleanUp()

		local tweenOut = TweenService:Create(GuiManager.spawnedFrame, TweenInfo.new(0.25, Enum.EasingStyle.Back, Enum.EasingDirection.In), {
			Size = UDim2.new(0, 0, 0, 0),
			BackgroundTransparency = 1
		})
		tweenOut:Play()
		tweenOut.Completed:Connect(function()
			if GuiManager.spawnedFrame then
				GuiManager.spawnedFrame:Destroy()
				GuiManager.spawnedFrame = nil
				GuiManager.isMinimized = false
			end
		end)
	end)

	-- Hover efeitos para botões controlar
	local function setupButtonHover(button, normalColor, hoverColor)
		local enterConn = button.MouseEnter:Connect(function()
			if GuiManager.spawnedFrame and GuiManager.spawnedFrame.Parent then
				TweenService:Create(button, TweenInfo.new(0.2), {BackgroundColor3 = hoverColor}):Play()
			end
		end)

		local leaveConn = button.MouseLeave:Connect(function()
			if GuiManager.spawnedFrame and GuiManager.spawnedFrame.Parent then
				TweenService:Create(button, TweenInfo.new(0.2), {BackgroundColor3 = normalColor}):Play()
			end
		end)

		GuiManager:AddConnection(enterConn)
		GuiManager:AddConnection(leaveConn)
	end

	setupButtonHover(minimizeButton, Color3.fromRGB(255, 193, 7), Color3.fromRGB(255, 213, 47))
	setupButtonHover(closeButton, Color3.fromRGB(220, 53, 69), Color3.fromRGB(240, 73, 89))

	GuiManager:AddConnection(minimizeConn)
	GuiManager:AddConnection(closeConn)
end)
