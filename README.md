local player = game.Players.LocalPlayer
local camera = workspace.CurrentCamera
local vu = game:GetService("VirtualUser")
local runService = game:GetService("RunService")

-- --- チューニング設定 ---
local cooldownTime = 0.5 -- 次のターゲットへ移るまでの待機（0.5秒まで短縮）
local grabRange = 200    -- 判定距離を200に短縮（短くするほど反応速度が上がります）
-- ----------------------

local lastClickTime = 0
local isGrabbingNow = false

-- 公式イベントをより確実に、高速に取得
local function getGrabEvent()
    for _, v in pairs(player:GetDescendants()) do
        if v:IsA("BindableEvent") and v.Name == "GrabNotifyEvent" then
            return v
        end
    end
    return nil
end

local grabEvent = getGrabEvent()
if grabEvent then
    grabEvent.Event:Connect(function(state)
        isGrabbingNow = state
        if not state then lastClickTime = tick() end
    end)
end

-- レイキャストのパラメータを固定して高速化
local rayParams = RaycastParams.new()
rayParams.FilterDescendantsInstances = {player.Character}
rayParams.FilterType = Enum.RaycastFilterType.Blacklist

local function fastCheck()
    local now = tick()

    -- 掴み中、またはクールタイム中なら即終了（最速パス）
    if isGrabbingNow or (now - lastClickTime < cooldownTime) then 
        return 
    end

    local center = camera.ViewportSize / 2
    local ray = camera:ViewportPointToRay(center.X, center.Y)
    
    -- 高速な Raycast 方式に変更
    local result = workspace:Raycast(ray.Origin, ray.Direction * grabRange, rayParams)

    if result and result.Instance then
        local model = result.Instance:FindFirstAncestorOfClass("Model")
        if model and model:FindFirstChild("Humanoid") and model.Name ~= player.Name then
            -- 掴み実行
            vu:CaptureController()
            vu:Button1Down(Vector2.new(center.X, center.Y))
            
            isGrabbingNow = true
            lastClickTime = now
        end
    end
end

-- ハートビート（物理計算直後）に同期させて、ズレを最小限にする
runService.Heartbeat:Connect(fastCheck)
