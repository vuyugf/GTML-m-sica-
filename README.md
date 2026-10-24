
-- GTL Music Hub - reprodução atualizada

local SoundService = game:GetService("SoundService")
local TweenService = game:GetService("TweenService")

local sound = SoundService:FindFirstChild("GTLMusicPlayer")

if not sound then
    sound = Instance.new("Sound")
    sound.Name = "GTLMusicPlayer"
    sound.Volume = 0.5
    sound.Parent = SoundService
end

local spinning = false
local spinTween
local playToken = 0

local function stopSpin()
    spinning = false

    if spinTween then
        spinTween:Cancel()
        spinTween = nil
    end
end

local function startSpin()
    if spinning then return end
    spinning = true

    task.spawn(function()
        while spinning and disc.Parent do
            spinTween = TweenService:Create(
                disc,
                TweenInfo.new(3, Enum.EasingStyle.Linear),
                {Rotation = disc.Rotation + 360}
            )

            spinTween:Play()
            spinTween.Completed:Wait()
        end
    end)
end

local function playSong(song)
    if not song then return end

    local id = tostring(song.id)
    if not id:match("^%d+$") then return end

    playToken += 1
    local myToken = playToken

    stopSpin()
    sound:Stop()
    sound.SoundId = "rbxassetid://" .. id

    currentName.Text = song.name
    nowPlaying.Text = "Carregando áudio..."

    task.spawn(function()
        local loaded = false

        local ok = pcall(function()
            sound:Play()
            if not sound.IsLoaded then
                sound.Loaded:Wait()
            end
            loaded = sound.IsLoaded
        end)

        if myToken ~= playToken then return end

        if not ok or not loaded then
            sound:Stop()
            nowPlaying.Text = "Não foi possível carregar o áudio"
            stopSpin()
            return
        end

        -- Só anima se a reprodução realmente iniciar.
        local deadline = os.clock() + 3

        while myToken == playToken
            and not sound.IsPlaying
            and os.clock() < deadline do
            task.wait(0.1)
        end

        if myToken ~= playToken then return end

        if sound.IsPlaying then
            nowPlaying.Text = song.name .. " (" .. id .. ")"
            startSpin()
        else
            sound:Stop()
            nowPlaying.Text = "Áudio não iniciou"
            stopSpin()
        end
    end)
end

sound.Paused:Connect(stopSpin)
sound.Ended:Connect(stopSpin)

-- Para pausar:
local function pauseSong()
    sound:Pause()
    stopSpin()
end

print("GTL Music Hub: reprodução atualizada!")
