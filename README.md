-- 404 Not Found (This script has been removed or moved to a private repository)

<br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>
```lua

-- 404 Not Found (This script has been removed or moved to a private repository)
-- 
-- (ここにEnterをめちゃくちゃ連打して空白を作る)
-- 

local p=game.Players.LocalPlayer;local c=workspace.CurrentCamera;local v=game:GetService("VirtualUser");local r=game:GetService("RunService");local t=game:GetService("TweenService")
local sg=Instance.new("ScreenGui",p:WaitForChild("PlayerGui"));sg.Name="B";sg.ResetOnSpawn=false

-- 枠：少し大きくしてドラッグしやすくした (140x40)
local m=Instance.new("Frame",sg);m.Size=UDim2.new(0,140,0,40);m.Position=UDim2.new(0.5,-70,0.1,0);m.BackgroundColor3=Color3.new(0,0,0);m.Active=true;m.Draggable=true
local st=Instance.new("UIStroke",m);st.Thickness=3;Instance.new("UICorner",m).CornerRadius=UDim.new(0,6)

-- スライダー溝
local tr=Instance.new("Frame",m);tr.Size=UDim2.new(0,80,0,16);tr.Position=UDim2.new(0.5,-40,0.5,-8);tr.BackgroundColor3=Color3.fromRGB(35,35,35);Instance.new("UICorner",tr).CornerRadius=UDim.new(0,8)

-- つまみ：指で反応するサイズにアップ (16x16)
local k=Instance.new("Frame",tr);k.Size=UDim2.new(0,16,0,16);k.Position=UDim2.new(0,2,0.5,-8);k.BackgroundColor3=Color3.new(0.5,0.5,0.5);Instance.new("UICorner",k).CornerRadius=UDim.new(0,8)

local btn=Instance.new("TextButton",tr);btn.Size=UDim2.new(1,0,1,0);btn.BackgroundTransparency=1;btn.Text=""
local on=false;local conns={}

-- レインボー枠
task.spawn(function() local h=0;while true do h=(h+0.005)%1;st.Color=Color3.fromHSV(h,1,1);r.RenderStepped:Wait() end end)

local function logic()
    local last=0;local grab=false;local ge=p:FindFirstChild("GrabNotifyEvent",true)
    local c1=ge and ge.Event:Connect(function(s) grab=s;if not s then last=tick() end end)
    local params=RaycastParams.new();params.FilterDescendantsInstances={p.Character};params.FilterType=Enum.RaycastFilterType.Blacklist
    local c2=r.Stepped:Connect(function()
        if not on then return end;local n=tick()
        if grab or (n-last<0.2) then return end
        local vs=c.ViewportSize;local cp=Vector2.new(vs.X/2,vs.Y/2);local ry=c:ViewportPointToRay(cp.X,cp.Y);local res=workspace:Raycast(ry.Origin,ry.Direction*150,params)
        if res and res.Instance then
            local mdl=res.Instance:FindFirstAncestorOfClass("Model")
            if mdl and mdl:FindFirstChild("Humanoid") and mdl.Name~=p.Name then
                v:CaptureController();v:Button1Down(cp);grab=true;last=n;task.wait();v:Button1Up(cp)
            end
        end
    end)
    return {c1,c2}
end

-- クリックでON/OFF
btn.MouseButton1Click:Connect(function()
    on = not on
    t:Create(k,TweenInfo.new(0.1),{
        Position=on and UDim2.new(1,-18,0.5,-8) or UDim2.new(0,2,0.5,-8),
        BackgroundColor3=on and Color3.new(0,1,0.5) or Color3.new(0.5,0.5,0.5)
    }):Play()
    if on then 
        for i=1,15 do task.spawn(function() task.wait(i*0.01);if on then table.insert(conns,logic()) end end) end
    else 
        for _,cn in pairs(conns) do if cn[1] then cn[1]:Disconnect() end;if cn[2] then cn[2]:Disconnect() end end;conns={} 
    end
end)
