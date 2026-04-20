--[[
    🚀 ULTRA DELTAKIDD HUB V2.4 - PC + MOBILE COMPATIBLE
    CREATED BY: DELTAKIDD
]]

-- =============================================
-- CHANGE THIS TO YOUR DEFAULT REMOTE PATH
-- =============================================
local DEFAULT_REMOTE_PATH = "ReplicatedStorage.Remotes.BuyItem"   -- ← CHANGE THIS
-- =============================================

local State = {
    LastRemote = nil,
    Logging = false,
    CustomPayload = "",
    TargetDecal = "107499863497854",
    SpyLogs = {}
}

local function setClipboard(text)
    if setclipboard then
        setclipboard(text)
    elseif syn and syn.write_clipboard then
        syn.write_clipboard(text)
    elseif clipboard then
        clipboard(text)
    else
        print("Clipboard not supported")
    end
end

-- Create GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "DeltaKiddHub"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

local MainFrame = Instance.new("Frame")
MainFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 80)
MainFrame.BorderSizePixel = 4
MainFrame.BorderColor3 = Color3.fromRGB(255, 0, 0)

-- Auto adjust size for PC vs Mobile
local isMobile = game:GetService("UserInputService").TouchEnabled and not game:GetService("UserInputService").KeyboardEnabled

if isMobile then
    MainFrame.Size = UDim2.new(0.95, 0, 0.88, 0)
    MainFrame.Position = UDim2.new(0.025, 0, 0.06, 0)
else
    MainFrame.Size = UDim2.new(0, 720, 0, 600)
    MainFrame.Position = UDim2.new(0.5, -360, 0.5, -300)
end

MainFrame.Parent = ScreenGui

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 55)
Title.BackgroundColor3 = Color3.fromRGB(0, 0, 100)
Title.Text = "🌌 ULTRA DELTAKIDD HUB V2.4"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = isMobile and 18 or 22
Title.Font = Enum.Font.GothamBold
Title.Parent = MainFrame

local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Size = UDim2.new(0.95, 0, 0, 55)
ToggleBtn.Position = UDim2.new(0.025, 0, 0, 65)
ToggleBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
ToggleBtn.Text = "ENABLE LIVE ARG SCANNER"
ToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleBtn.TextSize = isMobile and 16 or 18
ToggleBtn.Font = Enum.Font.GothamBold
ToggleBtn.Parent = MainFrame

local Scrolling = Instance.new("ScrollingFrame")
Scrolling.Size = UDim2.new(0.95, 0, 0, isMobile and 260 or 320)
Scrolling.Position = UDim2.new(0.025, 0, 0, 130)
Scrolling.BackgroundColor3 = Color3.fromRGB(0, 0, 60)
Scrolling.ScrollBarThickness = 8
Scrolling.Parent = MainFrame

local LogLabel = Instance.new("TextLabel")
LogLabel.Size = UDim2.new(1, -16, 0, 0)
LogLabel.BackgroundTransparency = 1
LogLabel.Text = "Enable scanner and play the game...\nArguments will appear here."
LogLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
LogLabel.TextXAlignment = Enum.TextXAlignment.Left
LogLabel.TextYAlignment = Enum.TextYAlignment.Top
LogLabel.TextWrapped = true
LogLabel.TextSize = isMobile and 14 or 15
LogLabel.Font = Enum.Font.Gotham
LogLabel.Parent = Scrolling

local CopyLatestBtn = Instance.new("TextButton")
CopyLatestBtn.Size = UDim2.new(0.46, 0, 0, 50)
CopyLatestBtn.Position = UDim2.new(0.025, 0, 0, isMobile and 400 or 470)
CopyLatestBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
CopyLatestBtn.Text = "COPY LATEST"
CopyLatestBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CopyLatestBtn.TextSize = isMobile and 15 or 16
CopyLatestBtn.Parent = MainFrame

local CopyAllBtn = Instance.new("TextButton")
CopyAllBtn.Size = UDim2.new(0.46, 0, 0, 50)
CopyAllBtn.Position = UDim2.new(0.515, 0, 0, isMobile and 400 or 470)
CopyAllBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
CopyAllBtn.Text = "COPY ALL LOGS"
CopyAllBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CopyAllBtn.TextSize = isMobile and 15 or 16
CopyAllBtn.Parent = MainFrame

local PayloadBox = Instance.new("TextBox")
PayloadBox.Size = UDim2.new(0.95, 0, 0, 50)
PayloadBox.Position = UDim2.new(0.025, 0, 0, isMobile and 460 or 530)
PayloadBox.PlaceholderText = "Enter payload (Sword, 100, true...)"
PayloadBox.BackgroundColor3 = Color3.fromRGB(0, 0, 60)
PayloadBox.TextColor3 = Color3.fromRGB(255, 255, 255)
PayloadBox.TextSize = isMobile and 15 or 16
PayloadBox.Parent = MainFrame

local InjectBtn = Instance.new("TextButton")
InjectBtn.Size = UDim2.new(0.95, 0, 0, 50)
InjectBtn.Position = UDim2.new(0.025, 0, 0, isMobile and 520 or 590)
InjectBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
InjectBtn.Text = "SINGLE INJECT"
InjectBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
InjectBtn.TextSize = isMobile and 16 or 18
InjectBtn.Parent = MainFrame

-- Auto set remote
local function AutoSetRemote()
    local remote = game
    for segment in DEFAULT_REMOTE_PATH:gmatch("[^%.]+") do
        if remote then remote = remote[segment] end
    end
    if remote and (remote:IsA("RemoteEvent") or remote:IsA("RemoteFunction")) then
        State.LastRemote = remote
    end
end
AutoSetRemote()

-- Hook
local oldNamecall = hookmetamethod(game, "__namecall", function(self, ...)
    local method = getnamecallmethod()
    local args = {...}
    if State.Logging and (method == "FireServer" or method == "InvokeServer") then
        pcall(function()
            local entry = {
                Time = os.date("%X"),
                RemoteName = self.Name,
                FullPath = self:GetFullName(),
                ArgsStr = tostring(args),
                FireLine = self:GetFullName() .. ":" .. method .. "(" .. tostring(args) .. ")"
            }
            table.insert(State.SpyLogs, 1, entry)
            if #State.SpyLogs > 60 then table.remove(State.SpyLogs) end

            local text = ""
            for _, log in ipairs(State.SpyLogs) do
                text = text .. log.Time .. " | " .. log.RemoteName .. "\n" .. log.ArgsStr .. "\n\n"
            end
            LogLabel.Text = text
            Scrolling.CanvasSize = UDim2.new(0, 0, 0, LogLabel.TextBounds.Y + 40)
        end)
    end
    return oldNamecall(self, ...)
end)

-- Button Connections
ToggleBtn.MouseButton1Click:Connect(function()
    State.Logging = not State.Logging
    ToggleBtn.Text = State.Logging and "SCANNER ENABLED ✓" or "ENABLE LIVE ARG SCANNER"
    ToggleBtn.BackgroundColor3 = State.Logging and Color3.fromRGB(0, 200, 0) or Color3.fromRGB(255, 0, 0)
end)

CopyLatestBtn.MouseButton1Click:Connect(function()
    if #State.SpyLogs > 0 then
        setClipboard(State.SpyLogs[1].FireLine)
    end
end)

CopyAllBtn.MouseButton1Click:Connect(function()
    local all = ""
    for _, log in ipairs(State.SpyLogs) do
        all = all .. log.FireLine .. "\n\n"
    end
    if all ~= "" then
        setClipboard(all)
    end
end)

InjectBtn.MouseButton1Click:Connect(function()
    if not State.LastRemote then return end
    local args = {}
    for arg in string.gmatch(PayloadBox.Text .. ",", '([^,]+)') do
        local trimmed = arg:match("^%s*(.-)%s*$")
        if trimmed ~= "" then
            if trimmed:lower() == "true" then table.insert(args, true)
            elseif trimmed:lower() == "false" then table.insert(args, false)
            elseif tonumber(trimmed) then table.insert(args, tonumber(trimmed))
            else table.insert(args, trimmed) end
        end
    end
    pcall(function() State.LastRemote:FireServer(unpack(args)) end)
end)

print("ULTRA DELTAKIDD HUB V2.4 LOADED - PC + Mobile Compatible")
