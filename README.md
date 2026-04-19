--[[
    🚀 ULTRA DELTAKIDD HUB V2.3 - NO RAYFIELD (Fixed setclipboard)
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

-- ====================== SETCLIPBOARD FIX ======================
local function setClipboard(text)
    if setclipboard then
        setclipboard(text)
    elseif syn and syn.write_clipboard then
        syn.write_clipboard(text)
    elseif clipboard then
        clipboard(text)
    else
        print("Clipboard not supported on this executor")
    end
end
-- ============================================================

-- Create GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "DeltaKiddHub"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 700, 0, 580)
MainFrame.Position = UDim2.new(0.5, -350, 0.5, -290)
MainFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 80)
MainFrame.BorderSizePixel = 4
MainFrame.BorderColor3 = Color3.fromRGB(255, 0, 0)
MainFrame.Parent = ScreenGui

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 60)
Title.BackgroundColor3 = Color3.fromRGB(0, 0, 100)
Title.Text = "🌌 ULTRA DELTAKIDD HUB V2.3"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 22
Title.Font = Enum.Font.GothamBold
Title.Parent = MainFrame

-- Scanner Toggle
local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Size = UDim2.new(0.9, 0, 0, 50)
ToggleBtn.Position = UDim2.new(0.05, 0, 0, 80)
ToggleBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
ToggleBtn.Text = "ENABLE LIVE ARG SCANNER"
ToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleBtn.TextSize = 18
ToggleBtn.Font = Enum.Font.GothamBold
ToggleBtn.Parent = MainFrame

-- Log Area
local Scrolling = Instance.new("ScrollingFrame")
Scrolling.Size = UDim2.new(0.9, 0, 0, 320)
Scrolling.Position = UDim2.new(0.05, 0, 0, 150)
Scrolling.BackgroundColor3 = Color3.fromRGB(0, 0, 60)
Scrolling.ScrollBarThickness = 10
Scrolling.Parent = MainFrame

local LogLabel = Instance.new("TextLabel")
LogLabel.Size = UDim2.new(1, -20, 0, 0)
LogLabel.BackgroundTransparency = 1
LogLabel.Text = "Enable scanner and play the game...\nArguments will appear here."
LogLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
LogLabel.TextXAlignment = Enum.TextXAlignment.Left
LogLabel.TextYAlignment = Enum.TextYAlignment.Top
LogLabel.TextWrapped = true
LogLabel.TextSize = 15
LogLabel.Font = Enum.Font.Gotham
LogLabel.Parent = Scrolling

-- Copy Buttons
local CopyLatestBtn = Instance.new("TextButton")
CopyLatestBtn.Size = UDim2.new(0.42, 0, 0, 45)
CopyLatestBtn.Position = UDim2.new(0.05, 0, 0, 490)
CopyLatestBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
CopyLatestBtn.Text = "📋 COPY LATEST"
CopyLatestBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CopyLatestBtn.Parent = MainFrame

local CopyAllBtn = Instance.new("TextButton")
CopyAllBtn.Size = UDim2.new(0.42, 0, 0, 45)
CopyAllBtn.Position = UDim2.new(0.53, 0, 0, 490)
CopyAllBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
CopyAllBtn.Text = "📋 COPY ALL LOGS"
CopyAllBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CopyAllBtn.Parent = MainFrame

-- Payload
local PayloadBox = Instance.new("TextBox")
PayloadBox.Size = UDim2.new(0.9, 0, 0, 45)
PayloadBox.Position = UDim2.new(0.05, 0, 0, 550)
PayloadBox.PlaceholderText = "Enter payload (Sword, 100, true...)"
PayloadBox.BackgroundColor3 = Color3.fromRGB(0, 0, 60)
PayloadBox.TextColor3 = Color3.fromRGB(255, 255, 255)
PayloadBox.TextSize = 16
PayloadBox.Parent = MainFrame

local InjectBtn = Instance.new("TextButton")
InjectBtn.Size = UDim2.new(0.9, 0, 0, 45)
InjectBtn.Position = UDim2.new(0.05, 0, 0, 605)
InjectBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
InjectBtn.Text = "🔥 SINGLE INJECT"
InjectBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
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
                text = text .. "🕒 " .. log.Time .. " | " .. log.RemoteName .. "\n📂 " .. log.FullPath .. "\n📊 " .. log.ArgsStr .. "\n\n"
            end
            LogLabel.Text = text
            Scrolling.CanvasSize = UDim2.new(0, 0, 0, LogLabel.TextBounds.Y + 50)
        end)
    end
    return oldNamecall(self, ...)
end)

-- Button functions
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

print("ULTRA DELTAKIDD HUB V2.3 LOADED - setclipboard fixed")
