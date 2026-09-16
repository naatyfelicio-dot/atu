local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local HS = game:GetService("HttpService")
local player = Players.LocalPlayer
if getgenv then
    if getgenv()._K7DuelsRunning then
        return
    end
    getgenv()._K7DuelsRunning = true
end

local function _resolve(name)
    local g = rawget(_G, name) or (getgenv and getgenv()[name])
    if typeof(g)=="function" then return g end
    if syn and typeof(syn[name])=="function" then return syn[name] end
    if fluxus and typeof(fluxus[name])=="function" then return fluxus[name] end
    return nil
end

local _gethui = _resolve("gethui") or (typeof(gethui)=="function" and gethui) or nil
local _getcustomasset = _resolve("getcustomasset") or (typeof(getcustomasset)=="function" and getcustomasset) or nil
local _request = _resolve("request")
    or _resolve("http_request")
    or (syn and syn.request)
    or (http and (http.request or http.Request))
    or (fluxus and fluxus.request)
    or (typeof(request)=="function" and request)
    or (typeof(http_request)=="function" and http_request)
    or nil

local function protectGui(gui)
    if not gui then return end
    pcall(function()
        if typeof(protect_gui)=="function" then protect_gui(gui)
        elseif syn and syn.protect_gui then syn.protect_gui(gui)
        elseif typeof(hide_in_gcoregui)=="function" then hide_in_gcoregui(gui)
        end
    end)
end

local function parentGui(gui)
    if not gui then return end
    protectGui(gui)
    local ok = false
    if _gethui then
        ok = pcall(function() gui.Parent = _gethui() end)
    end
    if not ok or not gui.Parent then
        ok = pcall(function() gui.Parent = game:GetService("CoreGui") end)
    end
    if not gui.Parent then
        pcall(function()
            local plr = Players.LocalPlayer
            if plr then gui.Parent = plr:WaitForChild("PlayerGui") end
        end)
    end
end

local _isfile = (typeof(isfile)=="function" and isfile)
    or (syn and syn.isfile)
    or (fluxus and fluxus.isfile)
    or (getgenv and getgenv().isfile)
    or function(p)
        local ok, data = pcall(function() return readfile(p) end)
        return ok and data ~= nil
    end
local _readfile = (typeof(readfile)=="function" and readfile)
    or (syn and syn.readfile)
    or (fluxus and fluxus.readfile)
    or (getgenv and getgenv().readfile)
    or function() return nil end
local _writefile = (typeof(writefile)=="function" and writefile)
    or (syn and syn.writefile)
    or (fluxus and fluxus.writefile)
    or (getgenv and getgenv().writefile)
    or function() end
isfile, readfile, writefile = _isfile, _readfile, _writefile

local function k7HttpGet(url)
    if _request then
        local ok, res = pcall(function()
            return _request({
                Url = url,
                Method = "GET",
                Headers = { ["User-Agent"] = "Mozilla/5.0" },
            })
        end)
        if ok and type(res)=="table" then
            return res.Body or res.body or res.Data or res.data
        elseif ok and type(res)=="string" then
            return res
        end
    end
    local ok2, body = pcall(function() return game:HttpGet(url) end)
    if ok2 then return body end
    return nil
end

local function k7GetCustomAsset(path)
    if not path then return nil end
    local asset = nil
    pcall(function()
        if _getcustomasset then asset = _getcustomasset(path) end
    end)
    if asset and asset ~= "" then return asset end
    pcall(function()
        if syn and syn.getcustomasset then asset = syn.getcustomasset(path) end
    end)
    if asset and asset ~= "" then return asset end
    pcall(function()
        if getcustomasset then asset = getcustomasset(path, true) end
    end)
    if asset and asset ~= "" then return asset end
    return nil
end

getconnections = getconnections or get_signal_cons or getconnects
    or (syn and syn.get_signal_cons) or (getgenv and getgenv().getconnections)


local K7_LOGO_RBX = "rbxassetid://135382218880707"
local _k7LogoAsset = K7_LOGO_RBX
local function getK7LogoAsset()
    return K7_LOGO_RBX
end


local function destroyOldGui(name)
    if not name then return end
    pcall(function()
        if _gethui then
            local h = _gethui(); local o = h and h:FindFirstChild(name); if o then o:Destroy() end
        end
    end)
    pcall(function()
        local cg = game:GetService("CoreGui"); local o = cg:FindFirstChild(name); if o then o:Destroy() end
    end)
    pcall(function()
        local plr = Players.LocalPlayer
        if plr then
            local pg = plr:FindFirstChild("PlayerGui")
            if pg then local o = pg:FindFirstChild(name); if o then o:Destroy() end end
        end
    end)
end


local K7_SKY_TAG = "K7SkyTheme"
local currentSkyTheme = "Night"
local K7_SKY_PRESETS = {
    ["Off"]={kind="off"},
    ["Night"]={clock=22,brightness=2,ambient={110,100,130},outAmb={120,110,140},sky={stars=4000,moon=18,sun=0,moonTex=true},atm={dens=0.45,color={120,60,180},decay={60,20,100},glare=0.5,haze=1.2}},
    ["Aurora"]={clock=14,brightness=3,ambient={150,120,150},outAmb={160,130,150},atm={dens=0.55,color={255,80,200},decay={255,20,150},glare=2.5,haze=3},clouds={cover=0.7,dens=0.7,color={255,240,250}}},
    ["Sunset"]={clock=17.2,brightness=2.5,ambient={170,120,100},outAmb={180,130,110},sky={stars=0,sun=25,moon=0},atm={dens=0.5,color={255,130,60},decay={255,80,30},glare=2,haze=2.5},clouds={cover=0.55,dens=0.55,color={255,200,140}}},
    ["Galaxy"]={clock=0,brightness=1.5,ambient={70,60,100},outAmb={80,70,110},sky={stars=10000,moon=30,sun=0},atm={dens=0.15,color={40,20,80},decay={20,10,50},glare=0.3,haze=0.5}},
    ["Cyber"]={clock=21,brightness=2.2,ambient={90,130,170},outAmb={100,140,180},sky={stars=2000,moon=12},atm={dens=0.4,color={0,200,255},decay={150,0,255},glare=2,haze=2},clouds={cover=0.4,dens=0.6,color={100,200,255}}},
    ["Sakura"]={clock=11,brightness=3.5,ambient={170,150,160},outAmb={180,160,170},sky={sun=8},atm={dens=0.3,color={255,200,220},decay={255,170,200},glare=1,haze=1.5},clouds={cover=0.6,dens=0.4,color={255,250,252}}},
    ["Pink Night"]={clock=23,brightness=2.2,ambient={120,60,110},outAmb={140,70,120},sky={stars=5000,moon=22,sun=0,moonTex=true},atm={dens=0.5,color={255,80,180},decay={140,30,100},glare=0.7,haze=1.4},clouds={cover=0.3,dens=0.5,color={180,90,150}}},
    ["Blood Moon"]={clock=22.5,brightness=1.6,ambient={130,40,40},outAmb={150,50,50},sky={stars=1500,moon=28,sun=0,moonTex=true},atm={dens=0.6,color={220,30,30},decay={120,10,10},glare=1.4,haze=2},clouds={cover=0.5,dens=0.7,color={120,30,30}}},
    ["Emerald Dawn"]={clock=6.5,brightness=2.8,ambient={130,170,140},outAmb={140,180,150},sky={sun=18,moon=0,stars=0},atm={dens=0.4,color={80,200,140},decay={40,150,90},glare=1.8,haze=2.2},clouds={cover=0.5,dens=0.5,color={200,255,220}}},
    ["Volcanic"]={clock=19,brightness=2,ambient={180,80,40},outAmb={200,90,50},sky={stars=200,sun=12,moon=0},atm={dens=0.75,color={255,60,0},decay={180,20,0},glare=3,haze=3.5},clouds={cover=0.8,dens=0.9,color={120,40,20}}},
    ["Arctic"]={clock=9,brightness=3.2,ambient={200,220,235},outAmb={210,230,245},sky={sun=10,stars=0,moon=0},atm={dens=0.3,color={180,220,255},decay={140,200,240},glare=1.5,haze=1.8},clouds={cover=0.7,dens=0.6,color={250,253,255}}},
    ["Midnight Ocean"]={clock=1.5,brightness=1.7,ambient={60,90,130},outAmb={70,100,140},sky={stars=6000,moon=24,sun=0,moonTex=true},atm={dens=0.5,color={20,60,140},decay={10,30,90},glare=0.6,haze=1.5}},
    ["Vaporwave"]={clock=19.5,brightness=2.4,ambient={180,120,200},outAmb={190,130,210},sky={stars=1000,moon=14},atm={dens=0.45,color={255,100,220},decay={120,60,255},glare=2.2,haze=2.4},clouds={cover=0.5,dens=0.55,color={200,150,255}}},
    ["Toxic"]={clock=13,brightness=2.5,ambient={140,180,80},outAmb={150,190,90},atm={dens=0.55,color={100,220,40},decay={60,150,20},glare=1.8,haze=2.6},clouds={cover=0.65,dens=0.7,color={180,255,120}}},
    ["Solar Eclipse"]={clock=12,brightness=0.9,ambient={50,40,60},outAmb={60,50,70},sky={stars=3500,sun=22,moon=0},atm={dens=0.5,color={255,140,40},decay={30,20,40},glare=2.8,haze=1.8}},
    ["Hellscape"]={clock=18,brightness=1.8,ambient={200,60,30},outAmb={220,70,40},sky={stars=100,sun=30,moon=0},atm={dens=0.85,color={255,30,0},decay={120,0,0},glare=3.5,haze=4},clouds={cover=0.95,dens=0.95,color={80,20,10}}},
    ["Heaven"]={clock=12,brightness=4,ambient={240,235,210},outAmb={250,245,220},sky={sun=16,moon=0,stars=0},atm={dens=0.25,color={255,250,220},decay={255,240,200},glare=3,haze=1.5},clouds={cover=0.85,dens=0.5,color={255,255,255}}},
    ["Storm"]={clock=15,brightness=1.4,ambient={90,90,110},outAmb={100,100,120},sky={stars=0,sun=6,moon=0},atm={dens=0.65,color={80,90,120},decay={40,50,80},glare=0.5,haze=3},clouds={cover=0.95,dens=0.95,color={60,65,80}}},
    ["Sunrise"]={clock=6.2,brightness=2.8,ambient={220,180,130},outAmb={230,190,140},sky={sun=22,stars=0,moon=0},atm={dens=0.45,color={255,180,100},decay={255,140,80},glare=2.4,haze=2.2},clouds={cover=0.4,dens=0.4,color={255,220,180}}},
    ["Deep Space"]={clock=0,brightness=1,ambient={30,25,50},outAmb={40,35,60},sky={stars=15000,moon=0,sun=0},atm={dens=0.08,color={15,5,40},decay={5,0,20},glare=0.2,haze=0.3}},
    ["Lavender Dream"]={clock=18.5,brightness=2.6,ambient={180,160,220},outAmb={190,170,230},sky={stars=800,moon=16,sun=0},atm={dens=0.4,color={200,160,255},decay={160,120,220},glare=1.4,haze=1.8},clouds={cover=0.55,dens=0.5,color={220,200,255}}},
    ["Inferno"]={clock=17.5,brightness=2.2,ambient={220,100,40},outAmb={235,110,50},sky={sun=26,moon=0,stars=0},atm={dens=0.6,color={255,90,20},decay={200,40,0},glare=3,haze=3.2},clouds={cover=0.7,dens=0.7,color={200,80,40}}},
    ["Mint Sky"]={clock=10,brightness=3.2,ambient={180,230,210},outAmb={190,240,220},sky={sun=10},atm={dens=0.32,color={150,255,210},decay={100,220,180},glare=1.6,haze=1.6},clouds={cover=0.55,dens=0.45,color={240,255,250}}},
}
local SkyOrder={"Off","Night","Aurora","Sunset","Galaxy","Cyber","Sakura","Pink Night","Blood Moon","Emerald Dawn","Volcanic","Arctic","Midnight Ocean","Vaporwave","Toxic","Solar Eclipse","Hellscape","Heaven","Storm","Sunrise","Deep Space","Lavender Dream","Inferno","Mint Sky"}
local function k7SkyColor(rgb) return Color3.fromRGB(rgb[1],rgb[2],rgb[3]) end
local function K7ApplyCustomSky(mode)
    for _,child in ipairs(Lighting:GetChildren()) do if child:GetAttribute(K7_SKY_TAG) then pcall(function() child:Destroy() end) end end
    local terrain=workspace:FindFirstChildOfClass("Terrain")
    if terrain then for _,child in ipairs(terrain:GetChildren()) do if child:GetAttribute(K7_SKY_TAG) then pcall(function() child:Destroy() end) end end end
    local preset=K7_SKY_PRESETS[mode]
    if not preset or preset.kind=="off" then Lighting.ClockTime=14;Lighting.Brightness=2;Lighting.OutdoorAmbient=Color3.fromRGB(127,127,127);Lighting.Ambient=Color3.fromRGB(127,127,127);Lighting.FogEnd=100000;Lighting.GlobalShadows=true;return end
    Lighting.FogStart=0;Lighting.FogEnd=100000;Lighting.FogColor=Color3.fromRGB(200,200,200);Lighting.ColorShift_Top=Color3.fromRGB(0,0,0);Lighting.ColorShift_Bottom=Color3.fromRGB(0,0,0);Lighting.GlobalShadows=true
    Lighting.ClockTime=preset.clock or 14;Lighting.Brightness=preset.brightness or 2
    if preset.outAmb then Lighting.OutdoorAmbient=k7SkyColor(preset.outAmb) end
    if preset.ambient then Lighting.Ambient=k7SkyColor(preset.ambient) end
    if preset.sky then
        local skyInst=Instance.new("Sky");skyInst:SetAttribute(K7_SKY_TAG,true)
        if preset.sky.stars then skyInst.StarCount=preset.sky.stars end
        if preset.sky.moon then skyInst.MoonAngularSize=preset.sky.moon end
        if preset.sky.sun then skyInst.SunAngularSize=preset.sky.sun end
        if preset.sky.moonTex then skyInst.MoonTextureId="rbxasset://sky/moon.jpg" end
        skyInst.Parent=Lighting
    end
    if preset.atm then
        local atm=Instance.new("Atmosphere");atm:SetAttribute(K7_SKY_TAG,true)
        atm.Density=preset.atm.dens or 0.3;atm.Color=k7SkyColor(preset.atm.color);atm.Decay=k7SkyColor(preset.atm.decay);atm.Glare=preset.atm.glare or 1;atm.Haze=preset.atm.haze or 1;atm.Parent=Lighting
    end
    if preset.clouds and terrain then
        local clouds=Instance.new("Clouds");clouds:SetAttribute(K7_SKY_TAG,true)
        clouds.Cover=preset.clouds.cover or 0.5;clouds.Density=preset.clouds.dens or 0.5;clouds.Color=k7SkyColor(preset.clouds.color);clouds.Parent=terrain
    end
end

currentGuiTheme = "Obsidian"
GUI_THEME_ORDER = {"Obsidian","Snow","Azure","Jade","Crimson","Violet","Amber","Slate"}
GUI_THEMES = {
    ["Obsidian"] = {
        BG=Color3.fromRGB(8,8,10), BG2=Color3.fromRGB(14,14,16), ROW_BG=Color3.fromRGB(16,16,18),
        ROW_BORDER=Color3.fromRGB(32,32,36), WHITE=Color3.fromRGB(235,235,240), GRAY=Color3.fromRGB(120,120,130),
        INP=Color3.fromRGB(12,12,14), ACCENT=Color3.fromRGB(255,255,255), ACCENT2=Color3.fromRGB(200,200,210),
        ACCENT3=Color3.fromRGB(36,36,40), OFF=Color3.fromRGB(24,24,28), SECT=Color3.fromRGB(100,100,110),
        MOB_ON=Color3.fromRGB(255,255,255), MOB_OFF=Color3.fromRGB(16,16,18), MOB_BORDER_OFF=Color3.fromRGB(40,40,44),
        MOB_TEXT_OFF=Color3.fromRGB(160,160,170), MOB_TEXT_ON=Color3.fromRGB(10,10,12),
        STEAL_BG=Color3.fromRGB(10,10,12), STEAL_FILL=Color3.fromRGB(235,235,240), STEAL_ACCENT=Color3.fromRGB(255,255,255),
        PARTICLE=Color3.fromRGB(200,200,210),
    },
    ["Snow"] = {
        BG=Color3.fromRGB(18,18,20), BG2=Color3.fromRGB(24,24,28), ROW_BG=Color3.fromRGB(28,28,32),
        ROW_BORDER=Color3.fromRGB(48,48,54), WHITE=Color3.fromRGB(245,245,250), GRAY=Color3.fromRGB(140,140,150),
        INP=Color3.fromRGB(20,20,24), ACCENT=Color3.fromRGB(230,230,240), ACCENT2=Color3.fromRGB(190,190,200),
        ACCENT3=Color3.fromRGB(44,44,50), OFF=Color3.fromRGB(34,34,38), SECT=Color3.fromRGB(130,130,140),
        MOB_ON=Color3.fromRGB(230,230,240), MOB_OFF=Color3.fromRGB(28,28,32), MOB_BORDER_OFF=Color3.fromRGB(50,50,56),
        MOB_TEXT_OFF=Color3.fromRGB(170,170,180), MOB_TEXT_ON=Color3.fromRGB(16,16,18),
        STEAL_BG=Color3.fromRGB(20,20,22), STEAL_FILL=Color3.fromRGB(220,220,230), STEAL_ACCENT=Color3.fromRGB(230,230,240),
        PARTICLE=Color3.fromRGB(210,210,220),
    },
    ["Azure"] = {
        BG=Color3.fromRGB(8,10,14), BG2=Color3.fromRGB(12,16,22), ROW_BG=Color3.fromRGB(14,18,26),
        ROW_BORDER=Color3.fromRGB(28,40,58), WHITE=Color3.fromRGB(220,230,245), GRAY=Color3.fromRGB(110,130,155),
        INP=Color3.fromRGB(10,14,20), ACCENT=Color3.fromRGB(90,160,220), ACCENT2=Color3.fromRGB(70,130,180),
        ACCENT3=Color3.fromRGB(24,40,60), OFF=Color3.fromRGB(18,24,34), SECT=Color3.fromRGB(100,140,180),
        MOB_ON=Color3.fromRGB(90,160,220), MOB_OFF=Color3.fromRGB(14,18,26), MOB_BORDER_OFF=Color3.fromRGB(30,44,64),
        MOB_TEXT_OFF=Color3.fromRGB(140,170,200), MOB_TEXT_ON=Color3.fromRGB(6,14,22),
        STEAL_BG=Color3.fromRGB(10,14,20), STEAL_FILL=Color3.fromRGB(80,150,210), STEAL_ACCENT=Color3.fromRGB(90,160,220),
        PARTICLE=Color3.fromRGB(90,150,210),
    },
    ["Jade"] = {
        BG=Color3.fromRGB(8,12,10), BG2=Color3.fromRGB(12,18,14), ROW_BG=Color3.fromRGB(14,22,18),
        ROW_BORDER=Color3.fromRGB(28,48,36), WHITE=Color3.fromRGB(220,240,230), GRAY=Color3.fromRGB(110,145,125),
        INP=Color3.fromRGB(10,16,12), ACCENT=Color3.fromRGB(80,190,140), ACCENT2=Color3.fromRGB(60,150,110),
        ACCENT3=Color3.fromRGB(24,48,34), OFF=Color3.fromRGB(18,28,22), SECT=Color3.fromRGB(100,160,130),
        MOB_ON=Color3.fromRGB(80,190,140), MOB_OFF=Color3.fromRGB(14,22,18), MOB_BORDER_OFF=Color3.fromRGB(32,52,40),
        MOB_TEXT_OFF=Color3.fromRGB(140,180,155), MOB_TEXT_ON=Color3.fromRGB(6,18,12),
        STEAL_BG=Color3.fromRGB(10,16,12), STEAL_FILL=Color3.fromRGB(70,180,130), STEAL_ACCENT=Color3.fromRGB(80,190,140),
        PARTICLE=Color3.fromRGB(90,180,140),
    },
    ["Crimson"] = {
        BG=Color3.fromRGB(12,8,8), BG2=Color3.fromRGB(18,12,12), ROW_BG=Color3.fromRGB(22,14,14),
        ROW_BORDER=Color3.fromRGB(48,28,28), WHITE=Color3.fromRGB(245,230,230), GRAY=Color3.fromRGB(150,120,120),
        INP=Color3.fromRGB(16,10,10), ACCENT=Color3.fromRGB(220,70,70), ACCENT2=Color3.fromRGB(180,55,55),
        ACCENT3=Color3.fromRGB(48,24,24), OFF=Color3.fromRGB(28,18,18), SECT=Color3.fromRGB(170,110,110),
        MOB_ON=Color3.fromRGB(220,70,70), MOB_OFF=Color3.fromRGB(22,14,14), MOB_BORDER_OFF=Color3.fromRGB(52,30,30),
        MOB_TEXT_OFF=Color3.fromRGB(185,140,140), MOB_TEXT_ON=Color3.fromRGB(20,6,6),
        STEAL_BG=Color3.fromRGB(14,10,10), STEAL_FILL=Color3.fromRGB(200,60,60), STEAL_ACCENT=Color3.fromRGB(220,70,70),
        PARTICLE=Color3.fromRGB(210,80,80),
    },
    ["Violet"] = {
        BG=Color3.fromRGB(10,8,14), BG2=Color3.fromRGB(16,12,22), ROW_BG=Color3.fromRGB(20,16,28),
        ROW_BORDER=Color3.fromRGB(42,34,58), WHITE=Color3.fromRGB(235,230,245), GRAY=Color3.fromRGB(135,125,160),
        INP=Color3.fromRGB(14,10,18), ACCENT=Color3.fromRGB(150,110,220), ACCENT2=Color3.fromRGB(120,85,180),
        ACCENT3=Color3.fromRGB(36,28,52), OFF=Color3.fromRGB(24,20,34), SECT=Color3.fromRGB(140,120,180),
        MOB_ON=Color3.fromRGB(150,110,220), MOB_OFF=Color3.fromRGB(20,16,28), MOB_BORDER_OFF=Color3.fromRGB(46,36,62),
        MOB_TEXT_OFF=Color3.fromRGB(165,150,190), MOB_TEXT_ON=Color3.fromRGB(14,8,22),
        STEAL_BG=Color3.fromRGB(12,10,16), STEAL_FILL=Color3.fromRGB(140,100,210), STEAL_ACCENT=Color3.fromRGB(150,110,220),
        PARTICLE=Color3.fromRGB(150,115,210),
    },
    ["Amber"] = {
        BG=Color3.fromRGB(12,10,8), BG2=Color3.fromRGB(18,16,12), ROW_BG=Color3.fromRGB(24,20,14),
        ROW_BORDER=Color3.fromRGB(52,44,28), WHITE=Color3.fromRGB(245,240,225), GRAY=Color3.fromRGB(155,140,110),
        INP=Color3.fromRGB(16,14,10), ACCENT=Color3.fromRGB(230,180,70), ACCENT2=Color3.fromRGB(190,145,50),
        ACCENT3=Color3.fromRGB(48,40,22), OFF=Color3.fromRGB(28,24,16), SECT=Color3.fromRGB(170,150,100),
        MOB_ON=Color3.fromRGB(230,180,70), MOB_OFF=Color3.fromRGB(24,20,14), MOB_BORDER_OFF=Color3.fromRGB(54,46,30),
        MOB_TEXT_OFF=Color3.fromRGB(185,170,130), MOB_TEXT_ON=Color3.fromRGB(18,14,4),
        STEAL_BG=Color3.fromRGB(14,12,8), STEAL_FILL=Color3.fromRGB(210,165,60), STEAL_ACCENT=Color3.fromRGB(230,180,70),
        PARTICLE=Color3.fromRGB(220,175,80),
    },
    ["Slate"] = {
        BG=Color3.fromRGB(10,11,13), BG2=Color3.fromRGB(16,17,20), ROW_BG=Color3.fromRGB(20,22,26),
        ROW_BORDER=Color3.fromRGB(38,42,48), WHITE=Color3.fromRGB(225,230,235), GRAY=Color3.fromRGB(125,135,145),
        INP=Color3.fromRGB(14,15,18), ACCENT=Color3.fromRGB(140,155,175), ACCENT2=Color3.fromRGB(110,125,145),
        ACCENT3=Color3.fromRGB(32,36,42), OFF=Color3.fromRGB(24,26,30), SECT=Color3.fromRGB(130,140,155),
        MOB_ON=Color3.fromRGB(140,155,175), MOB_OFF=Color3.fromRGB(20,22,26), MOB_BORDER_OFF=Color3.fromRGB(42,46,54),
        MOB_TEXT_OFF=Color3.fromRGB(155,165,180), MOB_TEXT_ON=Color3.fromRGB(10,12,14),
        STEAL_BG=Color3.fromRGB(12,13,15), STEAL_FILL=Color3.fromRGB(130,145,165), STEAL_ACCENT=Color3.fromRGB(140,155,175),
        PARTICLE=Color3.fromRGB(145,160,180),
    },
}
function getTheme()
    return GUI_THEMES[currentGuiTheme] or GUI_THEMES["Obsidian"]
end

TS=TweenService
LP=Players.LocalPlayer

PACKS = {
    ["Adidas Sports"] = {
        WalkAnim = 18537392113,
        RunAnim  = 18537384940,
        JumpAnim = 18537380791,
        FallAnim = 18537367238,
        SwimIdle = 18537387180,
        Swim     = 18537389531,
        Animation1 = 18537376492,
        Animation2 = 18537371272,
        ClimbAnim = 18537363391,
    },
    ["Adidas Community"] = {
        WalkAnim = 122150855457006,
        RunAnim  = 82598234841035,
        JumpAnim = 75290611992385,
        FallAnim = 98600215928904,
        SwimIdle = 109346520324160,
        Swim     = 133308483266208,
        Animation1 = 122257458498464,
        Animation2 = 102357151005774,
        ClimbAnim = 88763136693023,
    },
    ["Adidas Aura"] = {
        WalkAnim = 83842218823011,
        RunAnim  = 118320322718866,
        JumpAnim = 109996626521204,
        FallAnim = 95603166884636,
        SwimIdle = 94922130551805,
        Swim     = 134530128383903,
        Animation1 = 110211186840347,
        Animation2 = 114191137265065,
        ClimbAnim = 97824616490448,
    },
    ["Wicked Popular"] = {
        WalkAnim = 92072849924640,
        RunAnim = 72301599441680,
        JumpAnim = 104325245285198,
        FallAnim = 121152442762481,
        Animation1 = 118832222982049,
        ClimbAnim = 131326830509784,
        SwimIdle = 113199415118199,
        Swim = 99384245425157,
        Animation2 = 76049494037641,
    },
    Elder = {
        WalkAnim = 10921111375,
        RunAnim  = 10921104374,
        JumpAnim = 10921107367,
        FallAnim = 10921105765,
        SwimIdle = 10921110146,
        Swim     = 10921108971,
        ClimbAnim = 10921100400,
        Animation1 = 10921101664,
        Animation2 = 10921102574,
    },
    Zombie = {
        WalkAnim = 10921355261,
        RunAnim  = 616163682,
        JumpAnim = 10921351278,
        FallAnim = 10921350320,
        SwimIdle = 10921353442,
        Swim     = 10921352344,
        Animation1 = 10921344533,
        Animation2 = 10921345304,
        ClimbAnim = 10921343576,
    },
    Mage = {
        WalkAnim = 10921152678,
        RunAnim  = 10921148209,
        JumpAnim = 10921149743,
        FallAnim = 10921148939,
        SwimIdle = 10921151661,
        Swim     = 10921150788,
        ClimbAnim = 10921143404,
        Animation1 = 10921144709,
        Animation2 = 10921145797,
    },
    ["Catwalk Glam"] = {
        WalkAnim = 109168724482748,
        RunAnim  = 81024476153754,
        JumpAnim = 116936326516985,
        FallAnim = 92294537340807,
        SwimIdle = 98854111361360,
        Swim     = 134591743181628,
        ClimbAnim = 119377220967554,
        Animation1 = 133806214992291,
        Animation2 = 94970088341563,
    },
    Astronaut = {
        WalkAnim = 10921046031,
        RunAnim  = 10921039308,
        JumpAnim = 10921042494,
        FallAnim = 10921040576,
        SwimIdle = 10921045006,
        Swim     = 10921044000,
        ClimbAnim = 10921032124,
        Animation1 = 10921034824,
        Animation2 = 10921036806,
    },
    ['Wicked "Dancing Through Life"'] = {
        WalkAnim = 73718308412641,
        RunAnim  = 135515454877967,
        JumpAnim = 78508480717326,
        FallAnim = 78147885297412,
        SwimIdle = 129183123083281,
        Swim     = 110657013921774,
        ClimbAnim = 129447497744818,
        Animation1 = 92849173543269,
        Animation2 = 132238900951109,
    },
    Werewolf = {
        WalkAnim = 10921342074,
        RunAnim  = 10921336997,
        JumpAnim = nil,
        FallAnim = 10921337907,
        SwimIdle = 10921341319,
        Swim     = 10921340419,
        ClimbAnim = 10921329322,
        Animation1 = 10921330408,
        Animation2 = 10921333667,
    },
    Superhero = {
        WalkAnim = 10921298616,
        RunAnim  = 10921291831,
        JumpAnim = 10921294559,
        FallAnim = 10921293373,
        SwimIdle = 10921297391,
        Swim     = 10921295495,
        ClimbAnim = 10921286911,
        Animation1 = 10921288909,
        Animation2 = 10921290167,
    },
    Toy = {
        WalkAnim = 10921312010,
        RunAnim  = 10921306285,
        JumpAnim = 10921308158,
        FallAnim = 10921307241,
        SwimIdle = 10921310341,
        Swim     = 10921309319,
        ClimbAnim = 10921300839,
        Animation1 = 10921301576,
        Animation2 = nil,
    },
    ["No Boundaries"] = {
        WalkAnim = 18747074203,
        RunAnim  = 18747070484,
        JumpAnim = 18747069148,
        FallAnim = 18747062535,
        SwimIdle = 18747071682,
        Swim     = 18747073181,
        ClimbAnim = 18747060903,
        Animation1 = 18747067405,
        Animation2 = 18747063918,
    },
    NFL = {
        WalkAnim = 110358958299415,
        RunAnim  = 117333533048078,
        JumpAnim = 119846112151352,
        FallAnim = 129773241321032,
        SwimIdle = 79090109939093,
        Swim     = 132697394189921,
        ClimbAnim = 134630013742019,
        Animation1 = 92080889861410,
        Animation2 = 74451233229259,
    },
    ["Amazon Unboxed"] = {
        WalkAnim = 90478085024465,
        RunAnim  = 134824450619865,
        JumpAnim = 121454505477205,
        FallAnim = 94788218468396,
        SwimIdle = 129126268464847,
        Swim     = 105962919001086,
        ClimbAnim = 121145883950231,
        Animation1 = 98281136301627,
        Animation2 = nil,
    },
    Vampire = {
        WalkAnim = 10921326949,
        RunAnim  = 10921320299,
        JumpAnim = 10921322186,
        FallAnim = 10921321317,
        SwimIdle = 10921325443,
        Swim     = 10921324408,
        ClimbAnim = 10921314188,
        Animation1 = 10921315373,
        Animation2 = nil,
    },
    Ninja = {
        Run=656118852, Walk=656121766, Jump=656117878, Fall=656115606,
        Swim=656119721, SwimIdle=656121397, Climb=656114359,
        Idle={656117400,656118341,886742569}
    },
    Robot = {
        Run=616091570, Walk=616095330, Jump=616090535, Fall=616087089,
        Swim=616092998, SwimIdle=616094091, Climb=616086039,
        Idle={616088211,616089559,885531463}
    },
    Levitation = {
        Run=616010382, Walk=616013216, Jump=616008936, Fall=616005863,
        Swim=616011509, SwimIdle=616012453, Climb=616003713,
        Idle={616006778,616008087,886862142}
    },
    Stylish = {
        Run=616140816, Walk=616146177, Jump=616139451, Fall=616134815,
        Swim=616143378, SwimIdle=616144772, Climb=616133594,
        Idle={616136790,616138447,886888594}
    },
    Bubbly = {
        Run=910025107, Walk=910034870, Jump=910016857, Fall=910001910,
        Swim=910028158, SwimIdle=910030921, Climb=909997997,
        Idle={910004836,910009958,1018536639}
    },
    Cartoon = {
        Run=742638842, Walk=742640026, Jump=742637942, Fall=742637151,
        Swim=742639220, SwimIdle=742639812, Climb=742636889,
        Idle={742637544,742638445,885477856}
    },
}

animPack = "Adidas Sports"
animPackEnabled = true
headlessEnabled = false
korbloxEnabled = false

local HEADLESS_MESH_ID = "rbxassetid://1095708"
local KORBLOX_MESH_ID = "rbxassetid://101851696"
local KORBLOX_TEXTURE_ID = "rbxassetid://101851254"
local DARK_GREY_COLOR = Color3.fromRGB(64, 64, 64)

local function removeFace(head)
    local face = head:FindFirstChild("face")
    if face then face:Destroy() end
end

function applyHeadlessToChar(char, enabled)
    if not char then return end
    local head = char:FindFirstChild("Head")
    if not head then return end
    if enabled then
        head.Transparency = 1
        head.CanCollide = false
        removeFace(head)
        for _, child in ipairs(head:GetChildren()) do
            if child:IsA("SpecialMesh") and child.MeshId == HEADLESS_MESH_ID then
                child:Destroy()
            end
        end
        local mesh = Instance.new("SpecialMesh")
        mesh.MeshType = Enum.MeshType.FileMesh
        mesh.MeshId = HEADLESS_MESH_ID
        mesh.Scale = Vector3.new(0.001, 0.001, 0.001)
        mesh.Name = "HeadlessMesh"
        mesh.Parent = head
        head:GetPropertyChangedSignal("Transparency"):Connect(function()
            if head.Transparency ~= 1 then head.Transparency = 1 end
        end)
        head.ChildAdded:Connect(function(child)
            if child.Name == "face" and child:IsA("Decal") then child:Destroy() end
        end)
    else
        head.Transparency = 0
        head.CanCollide = true
        for _, child in ipairs(head:GetChildren()) do
            if child:IsA("SpecialMesh") and child.Name == "HeadlessMesh" then
                child:Destroy()
            end
        end
        removeFace(head)
    end
end

function applyKorbloxToChar(char, enabled)
    if not char then return end
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    if enabled then
        if humanoid.RigType == Enum.HumanoidRigType.R6 then
            local rightLeg = char:FindFirstChild("Right Leg")
            if rightLeg then
                for _, child in ipairs(rightLeg:GetChildren()) do
                    if child:IsA("SpecialMesh") or child:IsA("CharacterMesh") then child:Destroy() end
                end
                rightLeg.Color = DARK_GREY_COLOR
                rightLeg:GetPropertyChangedSignal("Color"):Connect(function()
                    if rightLeg.Color ~= DARK_GREY_COLOR then rightLeg.Color = DARK_GREY_COLOR end
                end)
                local mesh = Instance.new("SpecialMesh")
                mesh.MeshType = Enum.MeshType.FileMesh
                mesh.MeshId = KORBLOX_MESH_ID
                mesh.TextureId = KORBLOX_TEXTURE_ID
                mesh.Scale = Vector3.new(1, 1, 1)
                mesh.Name = "KorbloxMesh"
                mesh.Parent = rightLeg
            end
        elseif humanoid.RigType == Enum.HumanoidRigType.R15 then
            local rightUpperLeg = char:FindFirstChild("RightUpperLeg")
            if rightUpperLeg then
                rightUpperLeg.Transparency = 1
                local rightLowerLeg = char:FindFirstChild("RightLowerLeg")
                local rightFoot = char:FindFirstChild("RightFoot")
                if rightLowerLeg then rightLowerLeg.Transparency = 1 end
                if rightFoot then rightFoot.Transparency = 1 end
                local oldKorblox = char:FindFirstChild("KorbloxLeg")
                if oldKorblox then oldKorblox:Destroy() end
                local korbloxLeg = Instance.new("Part")
                korbloxLeg.Name = "KorbloxLeg"
                korbloxLeg.Size = Vector3.new(1, 2, 1)
                korbloxLeg.Anchored = false
                korbloxLeg.CanCollide = false
                korbloxLeg.Color = DARK_GREY_COLOR
                korbloxLeg.Parent = char
                local mesh = Instance.new("SpecialMesh")
                mesh.MeshType = Enum.MeshType.FileMesh
                mesh.MeshId = KORBLOX_MESH_ID
                mesh.TextureId = KORBLOX_TEXTURE_ID
                mesh.Scale = Vector3.new(1, 1, 1)
                mesh.Name = "KorbloxMesh"
                mesh.Parent = korbloxLeg
                local weld = Instance.new("Weld")
                weld.Part0 = rightUpperLeg
                weld.Part1 = korbloxLeg
                weld.C0 = CFrame.new(0, -0.8, 0)
                weld.Name = "KorbloxWeld"
                weld.Parent = korbloxLeg
            end
        end
    else
        if humanoid.RigType == Enum.HumanoidRigType.R6 then
            local rightLeg = char:FindFirstChild("Right Leg")
            if rightLeg then
                for _, child in ipairs(rightLeg:GetChildren()) do
                    if child:IsA("SpecialMesh") and child.Name == "KorbloxMesh" then child:Destroy() end
                end
                rightLeg.Color = Color3.fromRGB(255, 255, 255)
            end
        elseif humanoid.RigType == Enum.HumanoidRigType.R15 then
            local rightUpperLeg = char:FindFirstChild("RightUpperLeg")
            if rightUpperLeg then
                rightUpperLeg.Transparency = 0
                local rightLowerLeg = char:FindFirstChild("RightLowerLeg")
                local rightFoot = char:FindFirstChild("RightFoot")
                if rightLowerLeg then rightLowerLeg.Transparency = 0 end
                if rightFoot then rightFoot.Transparency = 0 end
                local korbloxLeg = char:FindFirstChild("KorbloxLeg")
                if korbloxLeg then korbloxLeg:Destroy() end
            end
        end
    end
end

function applyCharterToChar(char)
    if not char then return end
    applyHeadlessToChar(char, headlessEnabled)
    applyKorbloxToChar(char, korbloxEnabled)
end
function waitForAnimate(char)
    for _ = 1, 40 do
        local a = char:FindFirstChild("Animate")
        if a and a:FindFirstChild("idle") and a:FindFirstChild("run") and a:FindFirstChild("walk") then
            return a
        end
        task.wait(0.1)
    end
    return nil
end

function setAnim(animObj, id)
    if animObj and id then
        animObj.AnimationId = "rbxassetid://" .. tostring(id)
    end
end

function stopAllTracks(hum)
    if not hum then return end
    for _, t in ipairs(hum:GetPlayingAnimationTracks()) do
        pcall(function() t:Stop(0) end)
    end
end

function ensureAnim(folder, name)
    if not folder then return nil end
    local a = folder:FindFirstChild(name)
    if not a then
        a = Instance.new("Animation")
        a.Name = name
        a.Parent = folder
    end
    return a
end

function ensureIdleSlots(idleFolder, n)
    if not idleFolder then return end
    n = n or 2
    for i=1,n do
        ensureAnim(idleFolder, "Animation" .. i)
    end
end

function pickAnim(pack, ...)
    for i = 1, select("#", ...) do
        local k = select(i, ...)
        local v = pack[k]
        if v ~= nil then return v end
    end
    return nil
end

local applyingAnim = false
function applyAnimPack(packName)
    if applyingAnim then return false end
    applyingAnim = true

    local pack = PACKS[packName]
    if not pack then
        applyingAnim = false
        return false
    end

    local char = LP.Character or LP.CharacterAdded:Wait()
    local animate = waitForAnimate(char)
    if not animate then
        applyingAnim = false
        return false
    end

    local hum = char:FindFirstChildOfClass("Humanoid")
    stopAllTracks(hum)

    local runObj   = ensureAnim(animate:FindFirstChild("run"),   "RunAnim")
    local walkObj  = ensureAnim(animate:FindFirstChild("walk"),  "WalkAnim")
    local jumpObj  = ensureAnim(animate:FindFirstChild("jump"),  "JumpAnim")
    local fallObj  = ensureAnim(animate:FindFirstChild("fall"),  "FallAnim")
    local climbObj = ensureAnim(animate:FindFirstChild("climb"), "ClimbAnim")
    local swimObj  = ensureAnim(animate:FindFirstChild("swim"),     "Swim")
    local swimIdleObj = ensureAnim(animate:FindFirstChild("swimidle"), "SwimIdle")
    local idleFolder = animate:FindFirstChild("idle")

    setAnim(walkObj,  pickAnim(pack, "WalkAnim", "Walk"))
    setAnim(runObj,   pickAnim(pack, "RunAnim", "Run"))
    setAnim(jumpObj,  pickAnim(pack, "JumpAnim", "Jump"))
    setAnim(fallObj,  pickAnim(pack, "FallAnim", "Fall"))
    setAnim(climbObj, pickAnim(pack, "ClimbAnim", "Climb"))
    setAnim(swimObj,      pickAnim(pack, "Swim"))
    setAnim(swimIdleObj,  pickAnim(pack, "SwimIdle") or pickAnim(pack, "Swim"))

    if idleFolder then
        local a1 = pickAnim(pack, "Animation1")
        local a2 = pickAnim(pack, "Animation2")
        if a1 or a2 then
            ensureIdleSlots(idleFolder, 2)
            local id1 = a1 or a2
            local id2 = a2 or a1 or id1
            setAnim(idleFolder:FindFirstChild("Animation1"), id1)
            setAnim(idleFolder:FindFirstChild("Animation2"), id2)
        elseif pack.Idle and #pack.Idle > 0 then
            ensureIdleSlots(idleFolder, math.max(2, #pack.Idle))
            setAnim(idleFolder:FindFirstChild("Animation1"), pack.Idle[1])
            setAnim(idleFolder:FindFirstChild("Animation2"), pack.Idle[2] or pack.Idle[1])
            for i = 3, #pack.Idle do
                local a = idleFolder:FindFirstChild("Animation" .. i)
                if a then setAnim(a, pack.Idle[i]) end
            end
        end
    end

    animate.Disabled = true
    task.wait(0.06)
    animate.Disabled = false

    if hum then
        pcall(function()
            hum:ChangeState(Enum.HumanoidStateType.Landed)
            task.wait(0.03)
            hum:ChangeState(Enum.HumanoidStateType.Running)
        end)
    end

    animPack = packName
    applyingAnim = false
    return true
end


NS,CS=60,30
LAGGER_SPEED=15
LAGGER_CARRY_SPEED=24.5
carrySpeedActive = false
laggerModeEnabled = false
laggerCarryActive = false

antiRagdollEnabled,infJumpEnabled=false,false
playerEspEnabled,playerBoxesEnabled,playerTracersEnabled=false,false,false
medusaCounterEnabled,batCounterEnabled,unwalkEnabled=false,false,false
medusaDebounce,medusaLastUsed,dropActive=false,0,false
autoLeftEnabled,autoRightEnabled=false,false
autoLeftSetVisual,autoRightSetVisual=nil,nil
speedLabel=nil
autoBatEnabled=false
batAimbotEnabled=false
antiBatBypassLockEnabled=false
antiDesyncAimbotEnabled=false
AIMBOT_SPEED=58
LAGGER_AIMBOT_SPEED=40
tpBatEnabled=false
aimbotV2Enabled=false
aimbotMode="normal"
autoSwingEnabled=true
autoMoveSwingEnabled=false
autoMoveSwingInterval=0.3
_alSwingDebounce=false
_arSwingDebounce=false
autoBatSetVisual=nil
resetAutoBatMotion=nil
setBatCounterVisual=nil
startBatCounter,stopBatCounter=nil,nil
antiLagEnabled,removeAccessoriesEnabled,antiLagDescConn=false,false,nil
antiFlingEnabled=true
AdaptK7Extras = _G._AdaptK7Extras or {
    stretchedResEnabled=false, stretchValue=0.7, stretchRezConn=nil,
    removeAccessories=false, removeAccConn=nil, removeAccChildConn=nil,
    antiDieEnabled=false, antiDieHeartConn=nil, antiDieDeathConns={}, antiDieCharConn=nil, antiDieHumanoid=nil,
    antiRagdollMode="Splatter", antiRagdollNoSplatterCooldown=0,
    hardHitEnabled=false, hardHitRadius=10, hardHitRing=nil, hardHitConn=nil,
}
_G._AdaptK7Extras = AdaptK7Extras
if AdaptK7Extras.antiDieDeathConns==nil then AdaptK7Extras.antiDieDeathConns={} end
do
    local oldMt=AdaptK7Extras.antiDropMt
    local oldIdx=AdaptK7Extras.antiDropOldIndex
    local oldNewIdx=AdaptK7Extras.antiDropOldNewIndex
    if AdaptK7Extras.antiDropActive and oldMt and oldIdx and oldNewIdx and type(setreadonly)=="function" then
        pcall(function()
            setreadonly(oldMt,false)
            oldMt.__index=oldIdx
            oldMt.__newindex=oldNewIdx
            setreadonly(oldMt,true)
        end)
    end
    AdaptK7Extras.antiDropEnabled=nil
    AdaptK7Extras.antiDropActive=nil
    AdaptK7Extras.antiDropMt=nil
    AdaptK7Extras.antiDropOldIndex=nil
    AdaptK7Extras.antiDropOldNewIndex=nil
    AdaptK7Extras.antiDropSpoofedVelocity=nil
    AdaptK7Extras.stealGrabActive=nil
end
if AdaptK7Extras.antiRagdollMode~="No Splatter" then AdaptK7Extras.antiRagdollMode="Splatter" end
if AdaptK7Extras.antiRagdollNoSplatterCooldown==nil then AdaptK7Extras.antiRagdollNoSplatterCooldown=0 end
if AdaptK7Extras.hardHitEnabled==nil then AdaptK7Extras.hardHitEnabled=false end
if AdaptK7Extras.hardHitRadius==nil then AdaptK7Extras.hardHitRadius=10 end
AdaptK7Extras.aimbotStickyTime=tonumber(AdaptK7Extras.aimbotStickyTime) or 1.25
AdaptK7Extras.aimbotStickyMaxDist=tonumber(AdaptK7Extras.aimbotStickyMaxDist) or 85
AdaptK7Extras.aimbotStickyUntil=tonumber(AdaptK7Extras.aimbotStickyUntil) or 0
AdaptK7Extras.aimbotStickyTarget=nil
AdaptK7Extras.aimbotLastScan=0
AdaptK7Extras.aimbotTarget=nil
unwalkSavedAnimate,_anyKeyListening=nil,false
autoTPEnabled,autoTPHeight,autoTPConn,setAutoTPVisual=false,30,nil,nil
mirrorTPDownEnabled,mirrorTPPreviousY,mirrorTPLastTeleport=false,{},0
MIRROR_TP_DROP_THRESHOLD,MIRROR_TP_DOWN_Y=3,-7.00
setMirrorTPVisual=nil
guiTransparencyEnabled,mobileButtonsEnabled,mobileButtonsLocked=false,true,false
mobileButtonsSize=80
hubSizeScale=1
stealBarPos = nil
circleButtonsEnabled=false
stealBarFrame=nil
stealBarGuiRef=nil
progressFill=nil
setStealStatusText=nil
mobBtnRefs={}
mobGuiRef=nil
fovValue=80
fovOptions={80,120,180}
fovIndex=1
laggerModePillRef=nil
carryModePillRef=nil
mobBtnTransparencyEnabled=false
perButtonDragEnabled=true
antiKickEnabled=false
safeModeEnabled=false
setSafeModeVisual=nil
antiKickSetVisual=nil
brainrotDetected=false
activeBatBillboard=nil
activeMedusaBillboard=nil
ragdollGuiEnabled=false

introEnabled=true
selectedIntroMusic=1
persistentRagdollGui=nil
uiLocked=false
infJumpMode="manual"
holdInfJumpConn=nil
DROP_ASCEND_DURATION=0.2
DROP_ASCEND_SPEED=150

MOB_POS_FILE="k7duels_btnpos.json"
forceDefaultBtnPos=false
activeMobDrag=nil
local function loadBtnPositions()
    local data={}
    pcall(function()
        local raw=readfile(MOB_POS_FILE)
        if raw and #raw>2 then
            local decoded=HS:JSONDecode(raw)
            if type(decoded)=="table" then data=decoded end
        end
    end)
    if not data or next(data)==nil then
        pcall(function()
            local raw=readfile("k7duels.json")
            if raw then
                local cfg=HS:JSONDecode(raw)
                if type(cfg)=="table" and type(cfg.btnPositions)=="table" and next(cfg.btnPositions)~=nil then
                    data=cfg.btnPositions
                end
            end
        end)
    end
    return data or {}
end
local function saveBtnPositions()
    if not mobGuiRef then return end
    local out={}
    for _,child in ipairs(mobGuiRef:GetDescendants()) do
        if child:IsA("Frame") and child:GetAttribute("BtnKey") then
            local key=child:GetAttribute("BtnKey")
            out[key]={xs=0,xo=child.Position.X.Offset,ys=0,yo=child.Position.Y.Offset}
        end
    end
    if next(out)==nil then return end
    if writefile then
        pcall(function() writefile(MOB_POS_FILE,HS:JSONEncode(out)) end)
    end
    if writefile and isfile and isfile("k7duels.json") then
        pcall(function()
            local cfg=HS:JSONDecode(readfile("k7duels.json"))
            if type(cfg)=="table" then
                cfg.btnPositions=out
                writefile("k7duels.json",HS:JSONEncode(cfg))
            end
        end)
    end
end
task.spawn(function()
    while true do
        task.wait(2)
        pcall(saveBtnPositions)
    end
end)

refreshSpeedModeLabel,saveConfig=nil,nil
startUnwalk,stopUnwalk,setupMedusa,stopMedusaCounter=nil,nil,nil,nil
startAntiRagdoll,stopAntiRagdoll,startAutoLeft,stopAutoLeft,startAutoRight,stopAutoRight=nil,nil,nil,nil,nil,nil
startAutoTP,stopAutoTP,enableAntiLag,disableAntiLag=nil,nil,nil,nil
startBatAimbot,stopBatAimbot,queueAutoBatStart,runDrop,runTPFloor,startTPBat,stopTPBat=nil,nil,nil,nil,nil,nil,nil
startAutoSteal,stopAutoSteal,enableAntiKick,disableAntiKick,toggleCarryMode,toggleLaggerMode,toggleLaggerCarryMode=nil,nil,nil,nil,nil,nil,nil

local function addShimmerToLabel(lbl,color1,color2)
    local gr=Instance.new("UIGradient",lbl)
    gr.Color=ColorSequence.new({ColorSequenceKeypoint.new(0,color1 or Color3.fromRGB(160,80,255)),ColorSequenceKeypoint.new(0.5,color2 or Color3.fromRGB(255,255,255)),ColorSequenceKeypoint.new(1,color1 or Color3.fromRGB(160,80,255))})
    gr.Transparency=NumberSequence.new({NumberSequenceKeypoint.new(0,0.3,0),NumberSequenceKeypoint.new(0.5,0,0),NumberSequenceKeypoint.new(1,0.3,0)})
    return gr
end
fovConn=nil
local function applyFOV()
    if fovConn then fovConn:Disconnect() end
    fovConn=RunService.RenderStepped:Connect(function() local cam=workspace.CurrentCamera;if cam then cam.FieldOfView=fovValue end end)
end
applyFOV()

_G._AdaptCooldownStyleToken = (_G._AdaptCooldownStyleToken or 0) + 1
_G._AdaptStyleCooldownVisuals = function(char)
    char = char or LP.Character
    if not char then return end

    local function styleRoot(root)
        if not root then return end
        for _, bb in ipairs(root:GetDescendants()) do
            if bb:IsA("BillboardGui") and bb.Name ~= "AdaptBillboard" and bb.Name ~= "K7RagdollTimer" then
                local belongsToChar = bb:IsDescendantOf(char)
                if not belongsToChar then
                    local ok, adornee = pcall(function() return bb.Adornee end)
                    belongsToChar = ok and adornee and adornee:IsDescendantOf(char)
                end
                if belongsToChar then
                    local numberLabel = nil
                    for _, obj in ipairs(bb:GetDescendants()) do
                        if obj:IsA("TextLabel") or obj:IsA("TextButton") then
                            local raw = tostring(obj.Text or ""):gsub("%s+", ""):gsub("[sS]$", "")
                            local value = tonumber(raw)
                            if value and value >= 0 and value <= 10 then
                                numberLabel = obj
                                break
                            end
                        end
                    end

                    if numberLabel then
                        pcall(function()
                            bb.Size = UDim2.new(0, 140, 0, 42)
                            bb.AlwaysOnTop = true
                            bb.LightInfluence = 0
                        end)

                        for _, obj in ipairs(bb:GetDescendants()) do
                            pcall(function()
                                if obj == numberLabel then
                                    obj.Visible = true
                                    obj.Size = UDim2.new(1, 0, 1, 0)
                                    obj.Position = UDim2.new(0, 0, 0, 0)
                                    obj.BackgroundTransparency = 1
                                    obj.BorderSizePixel = 0
                                    obj.TextTransparency = 0
                                    obj.TextColor3 = Color3.fromRGB(255,255,255)
                                    obj.TextStrokeColor3 = Color3.fromRGB(0,0,0)
                                    obj.TextStrokeTransparency = 0
                                    obj.Font = Enum.Font.GothamBold
                                    obj.TextScaled = false
                                    obj.TextSize = 28
                                    obj.TextXAlignment = Enum.TextXAlignment.Center
                                    obj.TextYAlignment = Enum.TextYAlignment.Center
                                    obj.ZIndex = 20
                                elseif obj:IsA("ImageLabel") or obj:IsA("ImageButton") then
                                    obj.BackgroundTransparency = 1
                                    obj.ImageTransparency = 1
                                elseif obj:IsA("Frame") then
                                    obj.BackgroundTransparency = 1
                                    obj.BorderSizePixel = 0
                                elseif obj:IsA("UIStroke") then
                                    obj.Transparency = 1
                                elseif obj:IsA("TextLabel") or obj:IsA("TextButton") then
                                    obj.BackgroundTransparency = 1
                                    obj.TextTransparency = 1
                                end
                            end)
                        end
                    end
                end
            end
        end
    end

    styleRoot(char)
    styleRoot(LP:FindFirstChild("PlayerGui"))
end

do
    local styleToken = _G._AdaptCooldownStyleToken
    task.spawn(function()
        while _G._AdaptCooldownStyleToken == styleToken do
            pcall(_G._AdaptStyleCooldownVisuals, LP.Character)
            task.wait(0.2)
        end
    end)
end

do
    local timerBillboard = nil
    local timerLabel = nil
    local timerActive = false
    local timerConn = nil
    local ragdollCheckConn = nil
    local lastRagdollState = false

    local function getRagdollTheme()
        local th = (type(getTheme)=="function" and getTheme()) or nil
        if type(th)~="table" then
            th = (GUI_THEMES and GUI_THEMES[currentGuiTheme]) or nil
        end
        if type(th)~="table" then
            return {
                BG = Color3.fromRGB(8,8,10),
                ACCENT = Color3.fromRGB(255,255,255),
                ACCENT2 = Color3.fromRGB(200,200,210),
                WHITE = Color3.fromRGB(235,235,240),
                ROW_BORDER = Color3.fromRGB(32,32,36),
            }
        end
        return th
    end

    local timerCard = nil
    local timerStroke = nil
    local timerBar = nil
    local timerDot = nil
    local function createTimerBillboard()
        if timerBillboard then timerBillboard:Destroy() end

        local char = LP.Character
        if not char then return end
        local head = char:FindFirstChild("Head")
        if not head then return end
        local th = getRagdollTheme()

        timerBillboard = Instance.new("BillboardGui")
        timerBillboard.Name = "K7RagdollTimer"
        timerBillboard.Size = UDim2.new(0, 140, 0, 42)
        timerBillboard.StudsOffset = Vector3.new(0, 3.75, 0)
        timerBillboard.AlwaysOnTop = true
        timerBillboard.LightInfluence = 0
        timerBillboard.MaxDistance = 120
        timerBillboard.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
        timerBillboard.Parent = head

        timerLabel = Instance.new("TextLabel", timerBillboard)
        timerLabel.Name = "TimerText"
        timerLabel.Size = UDim2.new(1, 0, 1, 0)
        timerLabel.Position = UDim2.new(0, 0, 0, 0)
        timerLabel.BackgroundTransparency = 1
        timerLabel.BorderSizePixel = 0
        timerLabel.Text = "3.00"
        timerLabel.TextColor3 = th.WHITE or Color3.fromRGB(245,245,245)
        timerLabel.Font = Enum.Font.GothamBold
        timerLabel.TextSize = 28
        timerLabel.TextScaled = false
        timerLabel.TextStrokeTransparency = 0.2
        timerLabel.TextStrokeColor3 = Color3.fromRGB(0,0,0)
        timerLabel.TextXAlignment = Enum.TextXAlignment.Center
        timerLabel.TextYAlignment = Enum.TextYAlignment.Center
        timerLabel.ZIndex = 10

        timerBillboard.Enabled = false
    end

    local function updateTimerDisplay(time, isReady)
        if not timerLabel or not timerBillboard then return end
        local th = getRagdollTheme()
        timerLabel.TextColor3 = th.WHITE or Color3.fromRGB(245,245,245)
        if isReady then
            timerLabel.Text = "PRONTO!"
        else
            timerLabel.Text = string.format("%.2f", time)
        end
    end

    local function startRagdollTimer()
        if timerActive then return end
        
        if not timerBillboard or not timerBillboard.Parent then
            createTimerBillboard()
            if not timerBillboard then return end
        end
        
        timerActive = true
        timerBillboard.Enabled = true
        local duration = 3
        updateTimerDisplay(duration, false)
        local startTime = tick() + 0.15
        if timerConn then timerConn:Disconnect() end
        timerConn = RunService.Heartbeat:Connect(function()
            if not timerActive then 
                timerConn:Disconnect()
                timerConn = nil
                return 
            end
            
            local char = LP.Character
            if not char then
                timerActive = false
                if timerBillboard then timerBillboard.Enabled = false end
                if timerConn then timerConn:Disconnect(); timerConn = nil end
                return
            end
            local elapsed = math.max(tick() - startTime, 0)
            local remaining = math.max(duration - elapsed, 0)
            if remaining <= 0 then
                timerActive = false
                updateTimerDisplay(0, true)
                
                if timerConn then timerConn:Disconnect(); timerConn = nil end
                
                task.delay(1.5, function()
                    if timerBillboard then timerBillboard.Enabled = false end
                    timerActive = false
                end)
                return
            end
            
            updateTimerDisplay(remaining, false)
        end)
    end

    _G._SpiritHubStartStealCooldownVisual = function(forceRestart)
        local char = LP.Character
        local head = char and char:FindFirstChild("Head")
        if not head then return end

        local billboardMissing = (not timerBillboard) or (not timerBillboard.Parent) or (timerBillboard.Parent ~= head)

        if forceRestart then
            if timerConn then pcall(function() timerConn:Disconnect() end); timerConn = nil end
            timerActive = false
            createTimerBillboard()
            startRagdollTimer()
            return
        end

        if billboardMissing then
            createTimerBillboard()
            if timerActive and timerBillboard then
                timerBillboard.Enabled = true
            end
        end

        if not timerActive then
            startRagdollTimer()
        elseif timerBillboard then
            timerBillboard.Enabled = true
        end
    end

    local function checkRagdoll()
        if ragdollCheckConn then return end
        
        ragdollCheckConn = RunService.Heartbeat:Connect(function()
            local char = LP.Character
            if not char then return end
            local hum = char:FindFirstChildOfClass("Humanoid")
            if not hum then return end
            local state = hum:GetState()
            
            local isRagdolled = state == Enum.HumanoidStateType.Physics or 
                               state == Enum.HumanoidStateType.Ragdoll or 
                               state == Enum.HumanoidStateType.FallingDown
            
            if isRagdolled and not lastRagdollState and not timerActive then
                if not timerBillboard or not timerBillboard.Parent then
                    createTimerBillboard()
                end
                startRagdollTimer()
            end
            
            lastRagdollState = isRagdolled
        end)
    end
    
    task.spawn(checkRagdoll)
    
    playIntroAnimation = function()
        if introEnabled == false then return end

        local ASSETS = {
            IrishHub = "rbxassetid://131513419838182",
            GreenDuels = "rbxassetid://79052318194557",
            VampHub = "rbxassetid://108632168923336",
            AceDuels = "rbxassetid://86968392795085",
            K7Duels = "rbxassetid://135382218880707",
        }

        local MOGGED_SOUND_ID = "rbxassetid://132182797103598"
        local BACKGROUND_MUSIC_VOLUME = 0.55
        local MOGGED_SOUND_VOLUME = 0.95

        local PANEL_POP_TIME = 0.42
        local PANEL_POP_HOLD = 0.42
        local MOGGED_BAR_TIME = 0.34
        local MOGGED_HOLD = 0.82
        local BETWEEN_PANELS = 0.20
        local FINAL_REVEAL_HOLD = 1.55
        local POST_MOG_TOTAL = 10.0
        local LOADING_TIME = 8.333
        local TAP_TO_SKIP = true

        local RED = Color3.fromRGB(229, 22, 34)
        local RED_BRIGHT = Color3.fromRGB(255, 65, 76)
        local BLACK = Color3.fromRGB(5, 5, 7)
        local BLACK_2 = Color3.fromRGB(13, 13, 16)
        local WHITE = Color3.fromRGB(242, 242, 245)

        local Players2 = game:GetService("Players")
        local TweenService2 = game:GetService("TweenService")
        local RunService2 = game:GetService("RunService")
        local ContentProvider = game:GetService("ContentProvider")
        local SoundService2 = game:GetService("SoundService")
        local LP2 = Players2.LocalPlayer
        local playerGui = LP2:WaitForChild("PlayerGui")

        pcall(function()
            local old = playerGui:FindFirstChild("K7DuelsMogIntro")
            if old then old:Destroy() end
            local gh = (typeof(gethui)=="function" and gethui) or _gethui
            if gh then
                local h = gh()
                local o = h and h:FindFirstChild("K7DuelsMogIntro")
                if o then o:Destroy() end
            end
            local cg = game:GetService("CoreGui"):FindFirstChild("K7DuelsMogIntro")
            if cg then cg:Destroy() end
        end)

        local gui = Instance.new("ScreenGui")
        gui.Name = "K7DuelsMogIntro"
        gui.IgnoreGuiInset = true
        gui.ResetOnSpawn = false
        gui.DisplayOrder = 999999
        gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
        local parented = false
        pcall(function()
            if parentGui then parentGui(gui); parented = gui.Parent ~= nil end
        end)
        if not parented then
            pcall(function()
                local gh = (typeof(gethui)=="function" and gethui) or _gethui
                if gh then gui.Parent = gh(); parented = true end
            end)
        end
        if not parented then
            parented = pcall(function() gui.Parent = game:GetService("CoreGui") end)
        end
        if not parented then
            gui.Parent = playerGui
        end

        local introSong = Instance.new("Sound")
        introSong.Name = "K7MogIntroSong"
        introSong.Volume = BACKGROUND_MUSIC_VOLUME
        introSong.Looped = false
        introSong.Parent = SoundService2

        local moggedSound = Instance.new("Sound")
        moggedSound.Name = "MoggedImpactSound"
        moggedSound.SoundId = MOGGED_SOUND_ID
        moggedSound.Volume = MOGGED_SOUND_VOLUME
        moggedSound.Looped = false
        moggedSound.Parent = SoundService2
        local moggedEqualizer = Instance.new("EqualizerSoundEffect")
        moggedEqualizer.LowGain = 4
        moggedEqualizer.MidGain = 2
        moggedEqualizer.HighGain = -1
        moggedEqualizer.Parent = moggedSound
        local moggedBoost = Instance.new("DistortionSoundEffect")
        moggedBoost.Level = 0.08
        moggedBoost.Parent = moggedSound

        local songStarted = false
        local running = true
        local loadingConnection
        local viewportConnection

        local function playMoggedSound()
            pcall(function()
                moggedSound:Stop()
                moggedSound.TimePosition = 0
                moggedSound:Play()
            end)
        end

        local intros = {
            "https://files.catbox.moe/8ch5qz.mp3",
            "https://files.catbox.moe/8oogdu.mp3",
            "https://files.catbox.moe/gm26vl.mp3",
            "https://files.catbox.moe/m5kxm3.mp3",
        }
        local selectedMusic = math.clamp(tonumber(selectedIntroMusic) or 1, 1, 4)
        local songUrl = intros[selectedMusic]
        local cacheName = "K7HubIntro_" .. tostring(selectedMusic) .. ".mp3"
        local songAssetReady = nil

        local function tryGetCustomAsset(path)
            local asset = nil
            pcall(function()
                if k7GetCustomAsset then asset = k7GetCustomAsset(path) end
            end)
            if asset and asset ~= "" then return asset end
            pcall(function() if getcustomasset then asset = getcustomasset(path) end end)
            if asset and asset ~= "" then return asset end
            pcall(function() if syn and syn.getcustomasset then asset = syn.getcustomasset(path) end end)
            if asset and asset ~= "" then return asset end
            return nil
        end

        task.spawn(function()
            local cached = tryGetCustomAsset(cacheName)
            if cached then
                songAssetReady = cached
                return
            end
            local body = nil
            pcall(function()
                if k7HttpGet then body = k7HttpGet(songUrl) end
            end)
            if not body or #body < 1000 then
                pcall(function() body = game:HttpGet(songUrl) end)
            end
            if body and #body > 1000 then
                pcall(function()
                    if writefile then writefile(cacheName, body) end
                end)
                task.wait(0.05)
                songAssetReady = tryGetCustomAsset(cacheName)
            end
        end)

        local function startSelectedSong()
            if songStarted or not running then return end
            songStarted = true
            task.spawn(function()
                local waitStart = tick()
                while running and not songAssetReady and tick() - waitStart < 8 do
                    task.wait(0.05)
                end
                if not running then return end
                if songAssetReady then
                    pcall(function()
                        introSong.SoundId = songAssetReady
                        introSong.TimePosition = 0
                        introSong.Volume = BACKGROUND_MUSIC_VOLUME
                        introSong:Play()
                    end)
                end
            end)
        end

        local function tween(object, duration, goal, style, direction)
            local animation = TweenService2:Create(
                object,
                TweenInfo.new(duration, style or Enum.EasingStyle.Quint, direction or Enum.EasingDirection.Out),
                goal
            )
            animation:Play()
            return animation
        end

        local function round(object, radius)
            local corner = Instance.new("UICorner")
            corner.CornerRadius = UDim.new(0, radius)
            corner.Parent = object
            return corner
        end

        local function stroke(object, color, thickness, transparency)
            local line = Instance.new("UIStroke")
            line.Color = color
            line.Thickness = thickness or 1
            line.Transparency = transparency or 0
            line.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
            line.Parent = object
            return line
        end

        local background = Instance.new("Frame")
        background.Size = UDim2.fromScale(1, 1)
        background.BackgroundColor3 = BLACK
        background.BorderSizePixel = 0
        background.Parent = gui

        local backgroundGradient = Instance.new("UIGradient")
        backgroundGradient.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(3, 3, 5)),
            ColorSequenceKeypoint.new(0.48, Color3.fromRGB(25, 5, 8)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(3, 3, 5)),
        })
        backgroundGradient.Rotation = 90
        backgroundGradient.Parent = background

        local backgroundGlow = Instance.new("Frame")
        backgroundGlow.AnchorPoint = Vector2.new(0.5, 0.5)
        backgroundGlow.Position = UDim2.fromScale(0.5, 0.5)
        backgroundGlow.Size = UDim2.fromScale(0.62, 0.62)
        backgroundGlow.BackgroundColor3 = RED
        backgroundGlow.BackgroundTransparency = 0.93
        backgroundGlow.BorderSizePixel = 0
        backgroundGlow.Parent = background
        round(backgroundGlow, 999)

        local boardGroup = Instance.new("CanvasGroup")
        boardGroup.AnchorPoint = Vector2.new(0.5, 0.5)
        boardGroup.Position = UDim2.fromScale(0.5, 0.5)
        boardGroup.BackgroundTransparency = 1
        boardGroup.GroupTransparency = 0
        boardGroup.Parent = gui

        local boardScale = Instance.new("UIScale")
        boardScale.Scale = 1
        boardScale.Parent = boardGroup

        local board = Instance.new("Frame")
        board.Size = UDim2.fromScale(1, 1)
        board.BackgroundColor3 = BLACK
        board.BorderSizePixel = 0
        board.ClipsDescendants = true
        board.Parent = boardGroup
        round(board, 16)
        stroke(board, RED, 2, 0.22)

        local boardInner = Instance.new("Frame")
        boardInner.Position = UDim2.fromOffset(5, 5)
        boardInner.Size = UDim2.new(1, -10, 1, -10)
        boardInner.BackgroundColor3 = BLACK_2
        boardInner.BorderSizePixel = 0
        boardInner.ClipsDescendants = true
        boardInner.Parent = board
        round(boardInner, 12)

        local verticalDivider = Instance.new("Frame")
        verticalDivider.AnchorPoint = Vector2.new(0.5, 0.5)
        verticalDivider.Position = UDim2.fromScale(0.5, 0.5)
        verticalDivider.Size = UDim2.new(0, 3, 1, 0)
        verticalDivider.BackgroundColor3 = RED
        verticalDivider.BackgroundTransparency = 0.35
        verticalDivider.BorderSizePixel = 0
        verticalDivider.ZIndex = 30
        verticalDivider.Parent = boardInner

        local horizontalDivider = Instance.new("Frame")
        horizontalDivider.AnchorPoint = Vector2.new(0.5, 0.5)
        horizontalDivider.Position = UDim2.fromScale(0.5, 0.5)
        horizontalDivider.Size = UDim2.new(1, 0, 0, 3)
        horizontalDivider.BackgroundColor3 = RED
        horizontalDivider.BackgroundTransparency = 0.35
        horizontalDivider.BorderSizePixel = 0
        horizontalDivider.ZIndex = 30
        horizontalDivider.Parent = boardInner

        local panelData = {
            { name = "IRISH HUB", image = ASSETS.IrishHub, position = UDim2.fromScale(0, 0) },
            { name = "GREEN DUELS", image = ASSETS.GreenDuels, position = UDim2.fromScale(0.5, 0) },
            { name = "VAMP HUB", image = ASSETS.VampHub, position = UDim2.fromScale(0, 0.5) },
            { name = "ACE DUELS", image = ASSETS.AceDuels, position = UDim2.fromScale(0.5, 0.5) },
        }

        local panels = {}

        local function createMogPanel(info, order)
            local panel = Instance.new("CanvasGroup")
            panel.Name = info.name
            panel.Position = info.position
            panel.Size = UDim2.fromScale(0.5, 0.5)
            panel.BackgroundColor3 = BLACK_2
            panel.BorderSizePixel = 0
            panel.ClipsDescendants = true
            panel.GroupTransparency = 1
            panel.ZIndex = 10
            panel.Parent = boardInner

            local panelScale = Instance.new("UIScale")
            panelScale.Scale = 1.18
            panelScale.Parent = panel

            local image = Instance.new("ImageLabel")
            image.Size = UDim2.fromScale(1, 1)
            image.BackgroundColor3 = BLACK_2
            image.BorderSizePixel = 0
            image.Image = info.image
            image.ScaleType = Enum.ScaleType.Crop
            image.ImageTransparency = 0
            image.ZIndex = 11
            image.Parent = panel

            local darken = Instance.new("Frame")
            darken.Size = UDim2.fromScale(1, 1)
            darken.BackgroundColor3 = BLACK
            darken.BackgroundTransparency = 0.28
            darken.BorderSizePixel = 0
            darken.ZIndex = 12
            darken.Parent = panel

            local nameLabel = Instance.new("TextLabel")
            nameLabel.Position = UDim2.fromOffset(12, 9)
            nameLabel.Size = UDim2.new(1, -24, 0, 22)
            nameLabel.BackgroundTransparency = 1
            nameLabel.Text = info.name
            nameLabel.TextColor3 = WHITE
            nameLabel.TextTransparency = 0.15
            nameLabel.Font = Enum.Font.GothamBold
            nameLabel.TextSize = 12
            nameLabel.TextXAlignment = Enum.TextXAlignment.Left
            nameLabel.ZIndex = 14
            nameLabel.Parent = panel

            local nameStroke = Instance.new("UIStroke")
            nameStroke.Color = BLACK
            nameStroke.Thickness = 1.5
            nameStroke.Transparency = 0.1
            nameStroke.Parent = nameLabel

            local bar = Instance.new("Frame")
            bar.AnchorPoint = Vector2.new(0.5, 0.5)
            bar.Position = UDim2.fromScale(-0.55, 0.56)
            bar.Size = UDim2.new(0.9, 0, 0.24, 0)
            bar.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
            bar.BorderSizePixel = 0
            bar.ZIndex = 20
            bar.Parent = panel
            stroke(bar, Color3.fromRGB(30, 30, 34), 1, 0.28)

            local barGradient = Instance.new("UIGradient")
            barGradient.Color = ColorSequence.new({
                ColorSequenceKeypoint.new(0, Color3.fromRGB(0, 0, 0)),
                ColorSequenceKeypoint.new(0.5, Color3.fromRGB(10, 10, 12)),
                ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 0, 0)),
            })
            barGradient.Parent = bar

            local mogged = Instance.new("TextLabel")
            mogged.Size = UDim2.fromScale(1, 1)
            mogged.BackgroundTransparency = 1
            mogged.Text = "MOGGED"
            mogged.TextColor3 = RED_BRIGHT
            mogged.TextTransparency = 0
            mogged.Font = Enum.Font.GothamBlack
            mogged.TextScaled = true
            mogged.ZIndex = 21
            mogged.Parent = bar

            local textConstraint = Instance.new("UITextSizeConstraint")
            textConstraint.MinTextSize = 18
            textConstraint.MaxTextSize = 42
            textConstraint.Parent = mogged

            local mogStroke = Instance.new("UIStroke")
            mogStroke.Color = Color3.fromRGB(70, 0, 4)
            mogStroke.Thickness = 1.3
            mogStroke.Transparency = 0.12
            mogStroke.Parent = mogged

            local flash = Instance.new("Frame")
            flash.Size = UDim2.fromScale(1, 1)
            flash.BackgroundColor3 = RED_BRIGHT
            flash.BackgroundTransparency = 1
            flash.BorderSizePixel = 0
            flash.ZIndex = 25
            flash.Parent = panel

            panels[order] = {
                group = panel,
                scale = panelScale,
                image = image,
                bar = bar,
                flash = flash,
            }
        end

        for index, info in ipairs(panelData) do
            createMogPanel(info, index)
        end

        local fullReveal = Instance.new("CanvasGroup")
        fullReveal.Size = UDim2.fromScale(1, 1)
        fullReveal.BackgroundColor3 = BLACK
        fullReveal.BorderSizePixel = 0
        fullReveal.GroupTransparency = 1
        fullReveal.Visible = false
        fullReveal.ZIndex = 60
        fullReveal.Parent = boardInner

        local k7Ghost = Instance.new("ImageLabel")
        k7Ghost.AnchorPoint = Vector2.new(0.5, 0.5)
        k7Ghost.Position = UDim2.new(0.5, 7, 0.5, 0)
        k7Ghost.Size = UDim2.fromScale(1.16, 1.16)
        k7Ghost.BackgroundTransparency = 1
        k7Ghost.Image = ""
        k7Ghost.ImageColor3 = RED
        k7Ghost.ImageTransparency = 0.5
        k7Ghost.ScaleType = Enum.ScaleType.Crop
        k7Ghost.ZIndex = 61
        k7Ghost.Parent = fullReveal

        local k7Image = Instance.new("ImageLabel")
        k7Image.AnchorPoint = Vector2.new(0.5, 0.5)
        k7Image.Position = UDim2.fromScale(0.5, 0.5)
        k7Image.Size = UDim2.fromScale(1.22, 1.22)
        k7Image.BackgroundColor3 = BLACK
        k7Image.BorderSizePixel = 0
        k7Image.Image = ""
        k7Image.ScaleType = Enum.ScaleType.Crop
        k7Image.ZIndex = 62
        k7Image.Parent = fullReveal

        local k7Darken = Instance.new("Frame")
        k7Darken.Size = UDim2.fromScale(1, 1)
        k7Darken.BackgroundColor3 = BLACK
        k7Darken.BackgroundTransparency = 0.56
        k7Darken.BorderSizePixel = 0
        k7Darken.ZIndex = 63
        k7Darken.Parent = fullReveal

        local k7Title = Instance.new("TextLabel")
        k7Title.AnchorPoint = Vector2.new(0.5, 0.5)
        k7Title.Position = UDim2.fromScale(0.5, 0.82)
        k7Title.Size = UDim2.new(0.92, 0, 0.15, 0)
        k7Title.BackgroundTransparency = 1
        k7Title.Text = "SPIRIT HUB"
        k7Title.TextColor3 = RED_BRIGHT
        k7Title.TextTransparency = 1
        k7Title.Font = Enum.Font.GothamBlack
        k7Title.TextScaled = true
        k7Title.ZIndex = 68
        k7Title.Parent = fullReveal

        local k7TitleConstraint = Instance.new("UITextSizeConstraint")
        k7TitleConstraint.MinTextSize = 26
        k7TitleConstraint.MaxTextSize = 62
        k7TitleConstraint.Parent = k7Title

        local k7TitleStroke = Instance.new("UIStroke")
        k7TitleStroke.Color = BLACK
        k7TitleStroke.Thickness = 2.2
        k7TitleStroke.Transparency = 0.05
        k7TitleStroke.Parent = k7Title

        local k7Line = Instance.new("Frame")
        k7Line.AnchorPoint = Vector2.new(0.5, 0.5)
        k7Line.Position = UDim2.fromScale(0.5, 0.9)
        k7Line.Size = UDim2.new(0, 0, 0, 5)
        k7Line.BackgroundColor3 = RED_BRIGHT
        k7Line.BorderSizePixel = 0
        k7Line.ZIndex = 68
        k7Line.Parent = fullReveal
        round(k7Line, 4)

        local revealFlash = Instance.new("Frame")
        revealFlash.Size = UDim2.fromScale(1, 1)
        revealFlash.BackgroundColor3 = RED_BRIGHT
        revealFlash.BackgroundTransparency = 1
        revealFlash.BorderSizePixel = 0
        revealFlash.ZIndex = 80
        revealFlash.Parent = fullReveal

        local scanLine = Instance.new("Frame")
        scanLine.Position = UDim2.fromScale(0, -0.08)
        scanLine.Size = UDim2.new(1, 0, 0.08, 0)
        scanLine.BackgroundColor3 = RED_BRIGHT
        scanLine.BackgroundTransparency = 0.48
        scanLine.BorderSizePixel = 0
        scanLine.ZIndex = 70
        scanLine.Parent = fullReveal

        local loading = Instance.new("CanvasGroup")
        loading.Size = UDim2.fromScale(1, 1)
        loading.BackgroundColor3 = BLACK
        loading.BorderSizePixel = 0
        loading.GroupTransparency = 1
        loading.Visible = false
        loading.ZIndex = 100
        loading.Parent = boardInner

        local loadingImage = Instance.new("ImageLabel")
        loadingImage.AnchorPoint = Vector2.new(0.5, 0.5)
        loadingImage.Position = UDim2.fromScale(0.5, 0.39)
        loadingImage.Size = UDim2.fromScale(0.34, 0.34)
        loadingImage.BackgroundTransparency = 1
        loadingImage.Image = ""
        loadingImage.ScaleType = Enum.ScaleType.Crop
        loadingImage.ImageTransparency = 0.08
        loadingImage.ZIndex = 101
        loadingImage.Parent = loading
        round(loadingImage, 18)
        stroke(loadingImage, RED, 2, 0.2)

        local loadingTitle = Instance.new("TextLabel")
        loadingTitle.AnchorPoint = Vector2.new(0.5, 0.5)
        loadingTitle.Position = UDim2.fromScale(0.5, 0.62)
        loadingTitle.Size = UDim2.new(0.9, 0, 0.11, 0)
        loadingTitle.BackgroundTransparency = 1
        loadingTitle.Text = "SPIRIT HUB"
        loadingTitle.TextColor3 = RED_BRIGHT
        loadingTitle.Font = Enum.Font.GothamBlack
        loadingTitle.TextScaled = true
        loadingTitle.ZIndex = 102
        loadingTitle.Parent = loading

        local loadingTitleConstraint = Instance.new("UITextSizeConstraint")
        loadingTitleConstraint.MinTextSize = 24
        loadingTitleConstraint.MaxTextSize = 52
        loadingTitleConstraint.Parent = loadingTitle

        local loadingText = Instance.new("TextLabel")
        loadingText.AnchorPoint = Vector2.new(0.5, 0.5)
        loadingText.Position = UDim2.fromScale(0.5, 0.70)
        loadingText.Size = UDim2.new(0.8, 0, 0.055, 0)
        loadingText.BackgroundTransparency = 1
        loadingText.Text = "LOADING"
        loadingText.TextColor3 = WHITE
        loadingText.TextTransparency = 0.16
        loadingText.Font = Enum.Font.GothamMedium
        loadingText.TextScaled = true
        loadingText.ZIndex = 102
        loadingText.Parent = loading

        local loadingTextConstraint = Instance.new("UITextSizeConstraint")
        loadingTextConstraint.MinTextSize = 14
        loadingTextConstraint.MaxTextSize = 24
        loadingTextConstraint.Parent = loadingText

        local progressBack = Instance.new("Frame")
        progressBack.AnchorPoint = Vector2.new(0.5, 0.5)
        progressBack.Position = UDim2.fromScale(0.5, 0.875)
        progressBack.Size = UDim2.new(0.48, 0, 0.022, 0)
        progressBack.BackgroundColor3 = Color3.fromRGB(32, 32, 37)
        progressBack.BorderSizePixel = 0
        progressBack.ZIndex = 102
        progressBack.Parent = loading
        round(progressBack, 999)

        local progressFill = Instance.new("Frame")
        progressFill.Size = UDim2.fromScale(0, 1)
        progressFill.BackgroundColor3 = RED_BRIGHT
        progressFill.BorderSizePixel = 0
        progressFill.ZIndex = 103
        progressFill.Parent = progressBack
        round(progressFill, 999)

        local introDiscord = Instance.new("TextLabel")
        introDiscord.Name = "IntroDiscord"
        introDiscord.AnchorPoint = Vector2.new(0.5, 0.5)
        introDiscord.Position = UDim2.fromScale(0.5, 0.935)
        introDiscord.Size = UDim2.new(0.9, 0, 0.04, 0)
        introDiscord.BackgroundTransparency = 1
        introDiscord.Text = "Spirit Hub"
        introDiscord.TextColor3 = Color3.fromRGB(180, 180, 190)
        introDiscord.TextTransparency = 0.2
        introDiscord.Font = Enum.Font.GothamBold
        introDiscord.TextScaled = true
        introDiscord.ZIndex = 102
        introDiscord.Parent = loading
        local introDiscordConstraint = Instance.new("UITextSizeConstraint")
        introDiscordConstraint.MinTextSize = 10
        introDiscordConstraint.MaxTextSize = 16
        introDiscordConstraint.Parent = introDiscord

        local skipButton = Instance.new("TextButton")
        skipButton.Size = UDim2.fromScale(1, 1)
        skipButton.BackgroundTransparency = 1
        skipButton.Text = ""
        skipButton.AutoButtonColor = false
        skipButton.ZIndex = 500
        skipButton.Parent = gui

        local skipText = Instance.new("TextLabel")
        skipText.AnchorPoint = Vector2.new(0.5, 1)
        skipText.Position = UDim2.new(0.5, 0, 1, -18)
        skipText.Size = UDim2.fromOffset(180, 22)
        skipText.BackgroundTransparency = 1
        skipText.Text = TAP_TO_SKIP and "tap to skip" or ""
        skipText.TextColor3 = Color3.fromRGB(145, 145, 150)
        skipText.TextTransparency = 0.28
        skipText.Font = Enum.Font.GothamMedium
        skipText.TextSize = 11
        skipText.ZIndex = 501
        skipText.Parent = gui

        local screenFlash = Instance.new("Frame")
        screenFlash.Size = UDim2.fromScale(1, 1)
        screenFlash.BackgroundColor3 = RED_BRIGHT
        screenFlash.BackgroundTransparency = 1
        screenFlash.BorderSizePixel = 0
        screenFlash.ZIndex = 450
        screenFlash.Parent = gui

        local camera = workspace.CurrentCamera
        while not camera do
            RunService2.RenderStepped:Wait()
            camera = workspace.CurrentCamera
        end

        local function updateBoardSize()
            local viewportSize = camera.ViewportSize
            local side = math.floor(math.min(viewportSize.X, viewportSize.Y) * 0.78)
            side = math.clamp(side, 320, 760)
            boardGroup.Size = UDim2.fromOffset(side, side)
        end
        updateBoardSize()
        viewportConnection = camera:GetPropertyChangedSignal("ViewportSize"):Connect(updateBoardSize)

        local function shakeBoard(strength, duration)
            local startPosition = boardGroup.Position
            local startedAt = os.clock()
            while running and os.clock() - startedAt < duration do
                local fade = 1 - ((os.clock() - startedAt) / duration)
                local x = (math.random() - 0.5) * strength * fade
                local y = (math.random() - 0.5) * strength * fade
                boardGroup.Position = UDim2.new(0.5, x, 0.5, y)
                RunService2.RenderStepped:Wait()
            end
            boardGroup.Position = startPosition
        end

        local function redImpact()
            screenFlash.BackgroundTransparency = 0.22
            tween(screenFlash, 0.24, {BackgroundTransparency = 1}, Enum.EasingStyle.Quad)
            task.spawn(shakeBoard, 11, 0.17)
        end

        local function closeIntro()
            if not running then return end
            running = false
            if introSong.IsPlaying then
                tween(introSong, 0.28, {Volume = 0}, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
            end
            pcall(function() moggedSound:Stop() end)
            if loadingConnection then loadingConnection:Disconnect(); loadingConnection = nil end
            if viewportConnection then viewportConnection:Disconnect(); viewportConnection = nil end
            tween(boardGroup, 0.28, {GroupTransparency = 1}, Enum.EasingStyle.Quad)
            tween(background, 0.38, {BackgroundTransparency = 1}, Enum.EasingStyle.Quad)
            tween(skipText, 0.2, {TextTransparency = 1}, Enum.EasingStyle.Quad)
            task.delay(0.42, function()
                pcall(function() introSong:Destroy() end)
                pcall(function() moggedSound:Destroy() end)
                if gui then pcall(function() gui:Destroy() end) end
            end)
        end

        if TAP_TO_SKIP then
            skipButton.Activated:Connect(closeIntro)
        else
            skipButton.Active = false
        end

        pcall(function()
            ContentProvider:PreloadAsync({
                panels[1].image, panels[2].image, panels[3].image, panels[4].image,
                k7Image, loadingImage, moggedSound,
            })
        end)

        local mogCount = 0
        local postMogStartTime = nil
        local function revealPanel(panel)
            if not running then return end
            panel.group.GroupTransparency = 1
            panel.scale.Scale = 1.22
            panel.bar.Position = UDim2.fromScale(-0.55, 0.56)

            tween(panel.group, PANEL_POP_TIME * 0.72, {GroupTransparency = 0}, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
            tween(panel.scale, PANEL_POP_TIME, {Scale = 1}, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
            task.spawn(shakeBoard, 4, 0.09)
            task.wait(PANEL_POP_HOLD)

            playMoggedSound()
            mogCount = mogCount + 1
            if mogCount >= 4 then
                postMogStartTime = os.clock()
                startSelectedSong()
            end

            tween(panel.bar, MOGGED_BAR_TIME, {Position = UDim2.fromScale(0.5, 0.56)}, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
            panel.flash.BackgroundTransparency = 0.45
            tween(panel.flash, 0.28, {BackgroundTransparency = 1}, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
            task.spawn(shakeBoard, 6, 0.12)
            task.wait(MOGGED_HOLD)
        end

        local function transitionToK7()
            if not running then return end
            startSelectedSong()

            redImpact()
            screenFlash.BackgroundColor3 = RED_BRIGHT
            screenFlash.BackgroundTransparency = 0.05
            tween(screenFlash, 0.45, {BackgroundTransparency = 1}, Enum.EasingStyle.Quad)
            tween(boardScale, 0.18, {Scale = 1.14}, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
            task.spawn(shakeBoard, 18, 0.22)
            task.wait(0.12)

            for i, panel in ipairs(panels) do
                local ang = (i % 2 == 0) and 12 or -12
                tween(panel.group, 0.28, {GroupTransparency = 1}, Enum.EasingStyle.Quad)
                if panel.scale then
                    tween(panel.scale, 0.28, {Scale = 0.72}, Enum.EasingStyle.Back, Enum.EasingDirection.In)
                end
            end
            tween(verticalDivider, 0.2, {BackgroundTransparency = 1}, Enum.EasingStyle.Quad)
            tween(horizontalDivider, 0.2, {BackgroundTransparency = 1}, Enum.EasingStyle.Quad)
            task.wait(0.18)
            verticalDivider.Visible = false
            horizontalDivider.Visible = false

            fullReveal.Visible = true
            fullReveal.GroupTransparency = 1
            k7Image.Size = UDim2.fromScale(1.55, 1.55)
            k7Image.Rotation = 0
            k7Image.ImageTransparency = 0.15
            k7Ghost.Position = UDim2.new(0.5, 28, 0.5, 0)
            k7Ghost.ImageTransparency = 0.55
            k7Ghost.Size = UDim2.fromScale(1.4, 1.4)
            k7Title.TextTransparency = 1
            k7Title.Size = UDim2.new(0.35, 0, 0.10, 0)
            k7Title.TextColor3 = RED_BRIGHT
            if k7Line then
                k7Line.Size = UDim2.new(0, 0, 0, 5)
                k7Line.BackgroundTransparency = 0
            end
            if revealFlash then
                revealFlash.BackgroundTransparency = 0.0
            end

            tween(boardScale, 0.42, {Scale = 1}, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
            tween(fullReveal, 0.22, {GroupTransparency = 0}, Enum.EasingStyle.Quad)
            tween(k7Image, 0.85, {
                Size = UDim2.fromScale(1.05, 1.05),
                Rotation = 0,
                ImageTransparency = 0,
            }, Enum.EasingStyle.Quint, Enum.EasingDirection.Out)
            tween(k7Ghost, 0.7, {
                Position = UDim2.new(0.5, -10, 0.5, 0),
                ImageTransparency = 0.42,
                Size = UDim2.fromScale(1.12, 1.12),
            }, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
            if scanLine then
                scanLine.Position = UDim2.fromScale(0, -0.1)
                tween(scanLine, 0.9, {Position = UDim2.fromScale(0, 1.08)}, Enum.EasingStyle.Linear)
            end
            if revealFlash then
                tween(revealFlash, 0.55, {BackgroundTransparency = 1}, Enum.EasingStyle.Quad)
            end

            local burst = {}
            for i = 1, 16 do
                local p = Instance.new("Frame")
                p.AnchorPoint = Vector2.new(0.5, 0.5)
                p.Position = UDim2.fromScale(0.5, 0.5)
                p.Size = UDim2.fromOffset(5 + (i % 4), 5 + (i % 4))
                p.BackgroundColor3 = (i % 3 == 0) and WHITE or RED_BRIGHT
                p.BackgroundTransparency = 0.1
                p.BorderSizePixel = 0
                p.ZIndex = 80
                p.Parent = fullReveal
                local c = Instance.new("UICorner")
                c.CornerRadius = UDim.new(1, 0)
                c.Parent = p
                local ang = (i / 16) * math.pi * 2
                local dist = 40 + (i % 5) * 18
                burst[i] = {gui = p, ang = ang, dist = dist}
                tween(p, 0.55, {
                    Position = UDim2.new(0.5, math.cos(ang) * dist, 0.5, math.sin(ang) * dist),
                    BackgroundTransparency = 1,
                    Size = UDim2.fromOffset(2, 2),
                }, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
            end
            task.delay(0.7, function()
                for _, b in ipairs(burst) do
                    pcall(function() b.gui:Destroy() end)
                end
            end)

            task.wait(0.35)
            tween(k7Title, 0.45, {
                TextTransparency = 0,
                Size = UDim2.new(0.92, 0, 0.15, 0),
            }, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
            if k7Line then
                tween(k7Line, 0.5, {Size = UDim2.new(0.55, 0, 0, 5)}, Enum.EasingStyle.Quint)
            end
            task.spawn(shakeBoard, 12, 0.2)

            local energyRing = Instance.new("Frame")
            energyRing.AnchorPoint = Vector2.new(0.5, 0.5)
            energyRing.Position = UDim2.fromScale(0.5, 0.5)
            energyRing.Size = UDim2.fromOffset(40, 40)
            energyRing.BackgroundTransparency = 1
            energyRing.ZIndex = 78
            energyRing.Parent = fullReveal
            local erStroke = Instance.new("UIStroke")
            erStroke.Color = RED_BRIGHT
            erStroke.Thickness = 3
            erStroke.Transparency = 0.15
            erStroke.Parent = energyRing
            Instance.new("UICorner", energyRing).CornerRadius = UDim.new(1, 0)
            tween(energyRing, 0.7, {Size = UDim2.fromOffset(320, 320)}, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
            tween(erStroke, 0.7, {Transparency = 1, Thickness = 0.5}, Enum.EasingStyle.Quad)
            task.delay(0.75, function() pcall(function() energyRing:Destroy() end) end)

            for i = 1, 4 do
                local streak = Instance.new("Frame")
                streak.AnchorPoint = Vector2.new(0.5, 0.5)
                streak.Size = UDim2.new(0, 0, 0, 2)
                streak.BackgroundColor3 = RED_BRIGHT
                streak.BackgroundTransparency = 0.15
                streak.BorderSizePixel = 0
                streak.ZIndex = 77
                streak.Rotation = (i - 1) * 90 + 45
                streak.Position = UDim2.fromScale(0.5, 0.5)
                streak.Parent = fullReveal
                tween(streak, 0.55, {
                    Size = UDim2.new(0, 220, 0, 2),
                    BackgroundTransparency = 1,
                }, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
                task.delay(0.6, function() pcall(function() streak:Destroy() end) end)
            end

            local effectStart = os.clock()
            local glitchT = 0
            while running and os.clock() - effectStart < FINAL_REVEAL_HOLD do
                local elapsed = os.clock() - effectStart
                local pulse = math.sin(elapsed * 10)
                local pulse2 = math.cos(elapsed * 7.5)
                k7Ghost.Position = UDim2.new(0.5, pulse * 9 + math.sin(elapsed * 3) * 2, 0.5, pulse2 * 4)
                k7Ghost.ImageTransparency = 0.35 + pulse * 0.1
                k7Title.TextColor3 = Color3.fromRGB(
                    255,
                    math.clamp(40 + pulse * 55, 25, 140),
                    math.clamp(50 + pulse2 * 45, 30, 130)
                )
                if k7Line then
                    k7Line.BackgroundTransparency = 0.02 + math.abs(pulse) * 0.3
                    k7Line.BackgroundColor3 = Color3.fromRGB(
                        255,
                        math.clamp(40 + math.abs(pulse2) * 60, 30, 120),
                        math.clamp(50 + math.abs(pulse) * 40, 30, 110)
                    )
                end
                backgroundGlow.BackgroundTransparency = 0.86 + pulse * 0.04
                boardScale.Scale = 1 + pulse * 0.015

                glitchT = glitchT + 1
                if glitchT % 14 == 0 then
                    k7Title.Position = UDim2.new(0.5, (math.random() - 0.5) * 10, k7Title.Position.Y.Scale, k7Title.Position.Y.Offset)
                    task.delay(0.04, function()
                        if k7Title and k7Title.Parent then
                            k7Title.Position = UDim2.new(0.5, 0, k7Title.Position.Y.Scale, k7Title.Position.Y.Offset)
                        end
                    end)
                end
                RunService2.RenderStepped:Wait()
            end
            k7Image.Rotation = 0
            k7Title.TextColor3 = RED_BRIGHT
            if k7Title then
                k7Title.Position = UDim2.new(0.5, 0, k7Title.Position.Y.Scale, k7Title.Position.Y.Offset)
            end
        end

        local function showLoading()
            if not running then return end
            startSelectedSong()
            screenFlash.BackgroundColor3 = BLACK
            screenFlash.BackgroundTransparency = 1
            tween(screenFlash, 0.18, {BackgroundTransparency = 0.15}, Enum.EasingStyle.Quad)
            task.wait(0.12)

            if fullReveal then
                fullReveal.Visible = true
                tween(fullReveal, 0.45, {GroupTransparency = 0.55}, Enum.EasingStyle.Quad)
            end

            loading.Visible = true
            loading.GroupTransparency = 1
            tween(loading, 0.35, {GroupTransparency = 0}, Enum.EasingStyle.Quad)
            tween(screenFlash, 0.3, {BackgroundTransparency = 1}, Enum.EasingStyle.Quad)

            local ring1 = Instance.new("Frame")
            ring1.Name = "OrbitRing1"
            ring1.AnchorPoint = Vector2.new(0.5, 0.5)
            ring1.Position = UDim2.fromScale(0.5, 0.42)
            ring1.Size = UDim2.fromOffset(150, 150)
            ring1.BackgroundTransparency = 1
            ring1.ZIndex = 70
            ring1.Parent = loading
            local ring1Stroke = Instance.new("UIStroke")
            ring1Stroke.Color = RED_BRIGHT
            ring1Stroke.Thickness = 2
            ring1Stroke.Transparency = 0.35
            ring1Stroke.Parent = ring1
            local ring1Corner = Instance.new("UICorner")
            ring1Corner.CornerRadius = UDim.new(1, 0)
            ring1Corner.Parent = ring1

            local ring2 = Instance.new("Frame")
            ring2.Name = "OrbitRing2"
            ring2.AnchorPoint = Vector2.new(0.5, 0.5)
            ring2.Position = UDim2.fromScale(0.5, 0.42)
            ring2.Size = UDim2.fromOffset(190, 190)
            ring2.BackgroundTransparency = 1
            ring2.ZIndex = 69
            ring2.Parent = loading
            local ring2Stroke = Instance.new("UIStroke")
            ring2Stroke.Color = Color3.fromRGB(255, 120, 130)
            ring2Stroke.Thickness = 1.5
            ring2Stroke.Transparency = 0.55
            ring2Stroke.Parent = ring2
            local ring2Corner = Instance.new("UICorner")
            ring2Corner.CornerRadius = UDim.new(1, 0)
            ring2Corner.Parent = ring2

            local sparks = {}
            for i = 1, 10 do
                local s = Instance.new("Frame")
                s.AnchorPoint = Vector2.new(0.5, 0.5)
                s.Size = UDim2.fromOffset(4 + (i % 3), 4 + (i % 3))
                s.BackgroundColor3 = (i % 2 == 0) and RED_BRIGHT or WHITE
                s.BackgroundTransparency = 0.2
                s.BorderSizePixel = 0
                s.ZIndex = 72
                s.Parent = loading
                local c = Instance.new("UICorner")
                c.CornerRadius = UDim.new(1, 0)
                c.Parent = s
                sparks[i] = {gui = s, angle = (i / 10) * math.pi * 2, speed = 0.7 + (i % 4) * 0.25, radius = 70 + (i % 5) * 12}
            end

            local statusTag = Instance.new("TextLabel")
            statusTag.AnchorPoint = Vector2.new(0.5, 0.5)
            statusTag.Position = UDim2.fromScale(0.5, 0.79)
            statusTag.Size = UDim2.new(0.8, 0, 0.045, 0)
            statusTag.BackgroundTransparency = 1
            statusTag.Text = "INITIALIZING"
            statusTag.TextColor3 = WHITE
            statusTag.TextTransparency = 0.25
            statusTag.Font = Enum.Font.GothamBold
            statusTag.TextScaled = true
            statusTag.ZIndex = 75
            statusTag.Parent = loading
            local statusConstraint = Instance.new("UITextSizeConstraint")
            statusConstraint.MinTextSize = 10
            statusConstraint.MaxTextSize = 18
            statusConstraint.Parent = statusTag

            local statusMessages = {
                "INITIALIZING",
                "LOADING MODULES",
                "SYNCING",
                "CALIBRATING",
                "ALMOST READY",
                "READY"
            }

            local PCT_PER_SEC = 12
            local loadDuration = 100 / PCT_PER_SEC

            local startedAt = os.clock()
            local lastShown = -1
            loadingText.Text = "0%"
            progressFill.Size = UDim2.fromScale(0, 1)

            local barGlow = Instance.new("UIStroke")
            barGlow.Color = RED_BRIGHT
            barGlow.Thickness = 1.5
            barGlow.Transparency = 0.4
            if progressFill and progressFill.Parent then
                barGlow.Parent = progressFill.Parent
            end

            loadingConnection = RunService2.RenderStepped:Connect(function()
                if not running then return end
                local elapsed = os.clock() - startedAt
                local pct = math.clamp(elapsed * PCT_PER_SEC, 0, 100)
                local shown = math.floor(pct)
                if shown ~= lastShown then
                    lastShown = shown
                    loadingText.Text = tostring(shown) .. "%"
                    local msgIdx = math.clamp(math.floor(pct / 20) + 1, 1, #statusMessages)
                    statusTag.Text = statusMessages[msgIdx]
                end
                progressFill.Size = UDim2.fromScale(pct / 100, 1)

                local pulse = 1 + math.sin(elapsed * 5.5) * 0.04
                local pulse2 = math.sin(elapsed * 3.2)
                loadingImage.Size = UDim2.fromScale(0.34 * pulse, 0.34 * pulse)
                loadingImage.Rotation = 0
                loadingText.TextTransparency = 0.05 + math.abs(pulse2) * 0.08
                statusTag.TextTransparency = 0.2 + math.abs(math.sin(elapsed * 4)) * 0.25
                if loadingTitle then
                    loadingTitle.TextTransparency = 0.02 + math.abs(math.sin(elapsed * 2.8)) * 0.08
                end
                if introDiscord then
                    introDiscord.TextTransparency = 0.12 + math.abs(math.sin(elapsed * 2.2)) * 0.22
                end

                ring1.Rotation = elapsed * 55
                ring2.Rotation = -elapsed * 35
                local ringPulse = 1 + math.sin(elapsed * 4) * 0.06
                ring1.Size = UDim2.fromOffset(150 * ringPulse, 150 * ringPulse)
                ring2.Size = UDim2.fromOffset(190 * ringPulse, 190 * ringPulse)
                ring1Stroke.Transparency = 0.3 + math.abs(math.sin(elapsed * 3)) * 0.35
                ring2Stroke.Transparency = 0.45 + math.abs(math.cos(elapsed * 2.5)) * 0.3

                if barGlow then
                    barGlow.Transparency = 0.25 + math.abs(math.sin(elapsed * 6)) * 0.45
                    barGlow.Thickness = 1.2 + math.abs(math.sin(elapsed * 5)) * 1.5
                end
                if progressBack then
                    local h = 0.022 + math.abs(math.sin(elapsed * 3.5)) * 0.004
                    progressBack.Size = UDim2.new(0.48, 0, h, 0)
                end

                for _, sp in ipairs(sparks) do
                    local a = sp.angle + elapsed * sp.speed
                    local r = sp.radius * (1 + math.sin(elapsed * 2 + sp.angle) * 0.08)
                    local x = math.cos(a) * r
                    local y = math.sin(a) * r
                    sp.gui.Position = UDim2.new(0.5, x, 0.42, y)
                    sp.gui.BackgroundTransparency = 0.15 + (math.sin(elapsed * 5 + sp.angle) * 0.35 + 0.35)
                end

                if backgroundGlow then
                    backgroundGlow.BackgroundTransparency = 0.9 + math.sin(elapsed * 2.5) * 0.04
                end
                if boardScale then
                    boardScale.Scale = 1 + math.sin(elapsed * 2.2) * 0.01
                end

                if math.floor(elapsed * 2) % 7 == 0 and (elapsed * 2) % 1 < 0.05 then
                    screenFlash.BackgroundColor3 = RED
                    screenFlash.BackgroundTransparency = 0.82
                    tween(screenFlash, 0.25, {BackgroundTransparency = 1}, Enum.EasingStyle.Quad)
                end
            end)

            task.wait(loadDuration)
            if loadingConnection then
                loadingConnection:Disconnect()
                loadingConnection = nil
            end
            loadingText.Text = "100%"
            statusTag.Text = "READY"
            progressFill.Size = UDim2.fromScale(1, 1)
            tween(loadingImage, 0.2, {Size = UDim2.fromScale(0.42, 0.42)}, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
            task.wait(0.28)
            closeIntro()
        end

        task.spawn(function()
            for _, panel in ipairs(panels) do
                if not running then return end
                revealPanel(panel)
                task.wait(BETWEEN_PANELS)
            end
            if not running then return end
            task.wait(0.25)
            transitionToK7()
            if not running then return end
            showLoading()
        end)
    end


    LP.CharacterAdded:Connect(function(char)
        if ragdollCheckConn then 
            ragdollCheckConn:Disconnect()
            ragdollCheckConn = nil
        end
        timerActive = false
        lastRagdollState = false
        if timerConn then timerConn:Disconnect(); timerConn = nil end
        if timerBillboard then timerBillboard:Destroy(); timerBillboard = nil end
        
        task.wait(0.5)
        createTimerBillboard()
        checkRagdoll()
    end)
    
    task.spawn(function()
        task.wait(0.5)
        createTimerBillboard()
    end)
end

local function createRagdollBillboard(duration,labelText,color)
    if not ragdollGuiEnabled then return nil end
    local PINK  = color or Color3.fromRGB(160,80,255)
    local BG    = Color3.fromRGB(6,4,14)
    local WHITE = Color3.fromRGB(255,255,255)
    local W,H   = 210,80
    local guiName="K7RagdollTimer_"..labelText
    pcall(function()
        local cg=game:GetService("CoreGui");local old=cg:FindFirstChild(guiName);if old then old:Destroy() end
        local pg=LP:FindFirstChild("PlayerGui");if pg then local o=pg:FindFirstChild(guiName);if o then o:Destroy() end end
    end)
    local sg=Instance.new("ScreenGui")
    sg.Name=guiName;sg.ResetOnSpawn=false;sg.IgnoreGuiInset=true;sg.DisplayOrder=25
    parentGui(sg)
    local card=Instance.new("Frame",sg)
    card.Size=UDim2.new(0,W,0,H);card.Position=UDim2.new(0.5,-W/2,0,58)
    card.BackgroundColor3=BG;card.BackgroundTransparency=0.05
    card.BorderSizePixel=0;card.ZIndex=30;card.Active=true
    Instance.new("UICorner",card).CornerRadius=UDim.new(0,14)
    local stroke=Instance.new("UIStroke",card)
    stroke.Color=PINK;stroke.Thickness=1;stroke.Transparency=0.6
    local titleLbl=Instance.new("TextLabel",card)
    titleLbl.Size=UDim2.new(1,-16,0,28);titleLbl.Position=UDim2.new(0,8,0,6)
    titleLbl.BackgroundTransparency=1
    titleLbl.Text=(labelText=="RAGDOLL" and "RAGDOLL TIMER" or (labelText=="STONE" and "STONE TIMER" or labelText.." TIMER"))
    titleLbl.TextColor3=WHITE;titleLbl.Font=Enum.Font.GothamBlack;titleLbl.TextSize=13
    titleLbl.TextXAlignment=Enum.TextXAlignment.Center;titleLbl.ZIndex=32
    local divider=Instance.new("Frame",card)
    divider.Size=UDim2.new(1,-20,0,1);divider.Position=UDim2.new(0,10,0,34)
    divider.BackgroundColor3=PINK;divider.BackgroundTransparency=0.5;divider.BorderSizePixel=0;divider.ZIndex=31
    local timerLbl=Instance.new("TextLabel",card)
    timerLbl.Size=UDim2.new(1,0,0,H-38);timerLbl.Position=UDim2.new(0,0,0,36)
    timerLbl.BackgroundTransparency=1;timerLbl.Text=string.format("%.1f",duration).."s"
    timerLbl.TextColor3=PINK;timerLbl.Font=Enum.Font.GothamBlack;timerLbl.TextSize=24
    timerLbl.TextXAlignment=Enum.TextXAlignment.Center;timerLbl.ZIndex=32
    local shimmer=addShimmerToLabel(timerLbl,PINK,WHITE)
    task.spawn(function() local t=0;while timerLbl and timerLbl.Parent do t=t+0.04;shimmer.Offset=Vector2.new(math.sin(t)*0.5,0);task.wait(0.04) end end)
    local dragStart,dragStartPos,dragging=nil,nil,false
    card.InputBegan:Connect(function(inp)
        if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then
            dragging=true;dragStart=inp.Position;dragStartPos=card.Position
            inp.Changed:Connect(function()
                    if inp.UserInputState==Enum.UserInputState.End then
                        dragging=false
                        pcall(function()
                            local pos = frame.Position
                            stealBarPos = {sx=pos.X.Scale, ox=pos.X.Offset, sy=pos.Y.Scale, oy=pos.Y.Offset}
                            if type(saveConfig)=="function" then saveConfig() end
                        end)
                    end
                end)
        end
    end)
    UIS.InputChanged:Connect(function(inp)
        if dragging and (inp.UserInputType==Enum.UserInputType.MouseMovement or inp.UserInputType==Enum.UserInputType.Touch) then
            local d=inp.Position-dragStart
            card.Position=UDim2.new(dragStartPos.X.Scale,dragStartPos.X.Offset+d.X,dragStartPos.Y.Scale,dragStartPos.Y.Offset+d.Y)
        end
    end)
    local startTime=tick();local conn
    conn=RunService.Heartbeat:Connect(function()
        local remaining=math.max(0,duration-(tick()-startTime))
        if remaining<=0 then conn:Disconnect();pcall(function() sg:Destroy() end)
        elseif timerLbl and timerLbl.Parent then timerLbl.Text=string.format("%.1f",remaining).."s" end
    end)
    return sg
end
local function onHumanoidStateChanged(old,new)
    local char=LP.Character;if not char then return end
    local hum=char:FindFirstChildOfClass("Humanoid");if not hum then return end
    local isRag=(new==Enum.HumanoidStateType.Physics or new==Enum.HumanoidStateType.Ragdoll or new==Enum.HumanoidStateType.FallingDown)
    if isRag and not hum.PlatformStand and not activeBatBillboard then
        activeBatBillboard=createRagdollBillboard(2.5,"RAGDOLL",Color3.fromRGB(160,80,255))
        task.delay(2.6,function() if activeBatBillboard then pcall(function() activeBatBillboard:Destroy() end);activeBatBillboard=nil end end)
    end
end
local function onMedusaStateChanged()
    local char=LP.Character;if not char then return end
    local hum=char:FindFirstChildOfClass("Humanoid")
    if hum and hum.PlatformStand and not activeMedusaBillboard then
        activeMedusaBillboard=createRagdollBillboard(4.5,"STONE",Color3.fromRGB(255,255,255))
        task.delay(4.5,function() if activeMedusaBillboard then pcall(function() activeMedusaBillboard:Destroy() end);activeMedusaBillboard=nil end end)
    end
end
local function setupRagdollTriggers()
    local char=LP.Character;if not char then return end
    local hum=char:FindFirstChildOfClass("Humanoid")
    if hum then hum.StateChanged:Connect(onHumanoidStateChanged);hum:GetPropertyChangedSignal("PlatformStand"):Connect(onMedusaStateChanged) end
end
local function getSpeedThemeColors()
    local th=getTheme()
    local accent=th and th.ACCENT or Color3.fromRGB(255,255,255)
    local accent2=th and th.ACCENT2 or Color3.fromRGB(220,220,220)
    local gray=th and th.GRAY or Color3.fromRGB(200,200,210)
    return accent,accent2,gray
end
local function applySpeedIndicatorTheme()
    local char=LP.Character;if not char then return end
    local head=char:FindFirstChild("Head");if not head then return end
    local bb=head:FindFirstChild("K7SpeedBB");if not bb then return end
    local accent,accent2,gray=getSpeedThemeColors()
    for _,lbl in ipairs(bb:GetChildren()) do
        if lbl:IsA("TextLabel") then
            if lbl.Name=="DiscordLbl" then
                lbl.TextColor3=gray
                local gr=lbl:FindFirstChildOfClass("UIGradient")
                if gr then gr.Color=ColorSequence.new({ColorSequenceKeypoint.new(0,gray),ColorSequenceKeypoint.new(0.5,accent),ColorSequenceKeypoint.new(1,gray)}) end
            else
                lbl.TextColor3=accent
                local gr=lbl:FindFirstChildOfClass("UIGradient")
                if gr then gr.Color=ColorSequence.new({ColorSequenceKeypoint.new(0,accent2),ColorSequenceKeypoint.new(0.5,accent),ColorSequenceKeypoint.new(1,accent2)}) end
            end
        end
    end
end
local function setupSpeedIndicator(char)
    local head=char:WaitForChild("Head",5);if not head then return end
    pcall(function()
        local old=head:FindFirstChild("K7SpeedBB")
        if old then old:Destroy() end
    end)
    speedLabel=nil
    local accent,accent2,gray=getSpeedThemeColors()
    local bb=Instance.new("BillboardGui")
    bb.Name="K7SpeedBB";bb.Size=UDim2.new(0,130,0,48);bb.StudsOffset=Vector3.new(0,3.2,0)
    bb.AlwaysOnTop=true;bb.MaxDistance=80;bb.Parent=head
    local discordLabel=Instance.new("TextLabel",bb)
    discordLabel.Name="DiscordLbl";discordLabel.Size=UDim2.new(1,0,0.38,0)
    discordLabel.BackgroundTransparency=1;discordLabel.Text="Spirit Hub"
    discordLabel.TextColor3=gray;discordLabel.Font=Enum.Font.GothamBold
    discordLabel.TextScaled=true;discordLabel.TextStrokeTransparency=0.4
    speedLabel=Instance.new("TextLabel",bb)
    speedLabel.Name="SpeedLbl";speedLabel.Size=UDim2.new(1,0,0.55,0)
    speedLabel.Position=UDim2.new(0,0,0.38,0);speedLabel.BackgroundTransparency=1
    speedLabel.Text="0.0";speedLabel.TextColor3=accent
    speedLabel.Font=Enum.Font.GothamBold;speedLabel.TextScaled=true
    speedLabel.TextStrokeTransparency=0.35
    local gr1=addShimmerToLabel(speedLabel,accent2,accent)
    local gr2=addShimmerToLabel(discordLabel,gray,accent)
    task.spawn(function()
        local t=0
        while bb and bb.Parent do
            t=t+0.03
            pcall(function()
                gr1.Offset=Vector2.new(math.sin(t)*0.4,0)
                gr2.Offset=Vector2.new(math.sin(t)*0.4,0)
            end)
            task.wait(0.05)
        end
    end)
end
_G._SpiritHubEnsureOverheadVisuals = function(char)
    char = char or LP.Character
    if not char then return end
    local head = char:FindFirstChild("Head")
    if not head then return end
    local bb = head:FindFirstChild("K7SpeedBB")
    if not bb or not bb.Parent then
        pcall(setupSpeedIndicator, char)
    else
        pcall(function() bb.Enabled = true end)
        pcall(applySpeedIndicatorTheme)
    end
    if _G._AdaptStyleCooldownVisuals then
        pcall(_G._AdaptStyleCooldownVisuals, char)
    end
end

local function getActiveMoveSpeed()
    if laggerCarryActive then return LAGGER_CARRY_SPEED
    elseif laggerModeEnabled then return LAGGER_SPEED
    elseif carrySpeedActive then return CS
    else return NS end
end
local function getAutoPathSpeed()
    if laggerModeEnabled then return LAGGER_SPEED
    else return NS end
end


local _moveDt = 1/60
local _speedActive = false
local _lastHorizSpeed = 0
local _actualSpeed = 0
local _lastPos = nil
local _lastPosT = 0

local function destroySpeedConstraint()
    _speedActive = false
end

local function clearSpeedConstraint()
    _speedActive = false
    _lastHorizSpeed = 0
end

local function applyImpulseVel(part, targetVel)
    if not part or not part.Parent then return end
    pcall(function()
        local mass = part.AssemblyMass
        if not mass or mass ~= mass or mass <= 0 then mass = 1 end
        local current = part.AssemblyLinearVelocity
        local delta = targetVel - current
        part:ApplyImpulse(delta * mass)
    end)
end

local function applyVel(part, targetVel)
    applyImpulseVel(part, targetVel)
end

local function applyAngVel(part, targetAng)
    if not part or not part.Parent then return end
    pcall(function()
        local mass = part.AssemblyMass
        if not mass or mass <= 0 then mass = 1 end
        local current = part.AssemblyAngularVelocity
        local target = targetAng or Vector3.zero
        part:ApplyAngularImpulse((target - current) * mass * 0.35)
    end)
end

local _lastGroundNormalY = 1
local function sampleGround(hrp)
    local ok, res = pcall(function()
        local origin = hrp.Position + Vector3.new(0, 2, 0)
        local params = RaycastParams.new()
        params.FilterType = Enum.RaycastFilterType.Exclude
        local char = hrp.Parent
        if char then params.FilterDescendantsInstances = {char} end
        params.IgnoreWater = true
        local hit = workspace:Raycast(origin, Vector3.new(0, -8, 0), params)
        if hit and hit.Normal then
            return {ny = hit.Normal.Y, grounded = (hit.Distance < 4.2)}
        end
        return {ny = 1, grounded = false}
    end)
    if ok and type(res) == "table" and type(res.ny) == "number" then
        _lastGroundNormalY = res.ny
        return res.ny, res.grounded == true
    end
    return _lastGroundNormalY, false
end

local function setSpeedConstraint(hrp, horizVel)
    if not hrp or not hrp.Parent then return end
    local hx, hz = horizVel.X, horizVel.Z
    local mag = math.sqrt(hx * hx + hz * hz)
    _lastHorizSpeed = mag
    _speedActive = mag > 0.05
    if mag < 0.05 then return end

    pcall(function()
        local ny, grounded = sampleGround(hrp)
        local v = hrp.AssemblyLinearVelocity
        local mass = hrp.AssemblyMass
        if not mass or mass ~= mass or mass <= 0 then mass = 1 end

        local gain = 0.85
        if grounded then
            if ny < 0.55 then
                gain = 0.45
            elseif ny < 0.78 then
                gain = 0.62
            else
                gain = 0.9
            end
        else
            gain = 0.7
        end

        local target = Vector3.new(hx, v.Y, hz)
        hrp:ApplyImpulse((target - v) * mass * gain)
    end)
end

local function brakeHorizontal(hrp)
    if not hrp or not hrp.Parent then return end
    pcall(function()
        local ny, grounded = sampleGround(hrp)
        if not grounded then return end
        local v = hrp.AssemblyLinearVelocity
        local hx, hz = v.X, v.Z
        local hMag = math.sqrt(hx * hx + hz * hz)
        if hMag < 0.8 then
            if hMag > 0.05 then
                hrp.AssemblyLinearVelocity = Vector3.new(0, v.Y, 0)
            end
            return
        end
        local mass = hrp.AssemblyMass
        if not mass or mass ~= mass or mass <= 0 then mass = 1 end
        local target = Vector3.new(0, v.Y, 0)
        hrp:ApplyImpulse((target - v) * mass * 0.92)
    end)
end

local function applyCFrameMove(part, dirUnit, spd, dt)
    if not part or not part.Parent then return end
    if not dirUnit or dirUnit.Magnitude < 0.01 then return end
    _moveDt = dt or _moveDt
    setSpeedConstraint(part, Vector3.new(dirUnit.X, 0, dirUnit.Z).Unit * spd)
end

local _lastManualJump = 0
local function manualJumpBoost(boost)
    if not infJumpEnabled or infJumpMode~="manual" then return end
    local now = tick()
    if now - _lastManualJump < 0.08 then return end
    _lastManualJump = now
    local char=LP.Character;if not char then return end
    local root=char:FindFirstChild("HumanoidRootPart");if not root then return end
    local v=root.AssemblyLinearVelocity
    root.AssemblyLinearVelocity=Vector3.new(v.X,boost or 50,v.Z)
end
local function startHoldInfJump()
    if holdInfJumpConn then holdInfJumpConn:Disconnect() end
    holdInfJumpConn=RunService.Heartbeat:Connect(function()
        if not infJumpEnabled or infJumpMode~="hold" then return end
        local char=LP.Character;if not char then return end
        local root=char:FindFirstChild("HumanoidRootPart");local hum=char:FindFirstChildOfClass("Humanoid");if not root or not hum then return end
        local isJumpHeld=UIS:IsKeyDown(Enum.KeyCode.Space) or (hum.Jump==true)
        local cv=root.AssemblyLinearVelocity
        if isJumpHeld and cv.Y<35 then applyVel(root, Vector3.new(cv.X,55,cv.Z)) end
        cv=root.AssemblyLinearVelocity
        if cv.Y<-120 then applyVel(root, Vector3.new(cv.X,-120,cv.Z)) end
    end)
end
local function stopHoldInfJump() if holdInfJumpConn then holdInfJumpConn:Disconnect();holdInfJumpConn=nil end end
UIS.JumpRequest:Connect(function()
    manualJumpBoost(50)
end)

_antiKickConns={}
enableAntiKick=function()
    antiKickEnabled=true
    task.spawn(function()
        while antiKickEnabled do
            task.wait(0.5)
            local char=LP.Character
            local found=false
            if char then
                for _,tool in ipairs(char:GetChildren()) do
                    if tool:IsA("Tool") then
                        local n=tool.Name:lower()
                        if n:find("brainrot") or n:find("skibidi") or n:find("toilet") then
                            found=true
                            break
                        end
                    end
                end
            end
            brainrotDetected=found
            if found then
                if autoBatEnabled then
                    autoBatEnabled=false
                    if stopBatAimbot then pcall(stopBatAimbot) end
                    if resetAutoBatMotion then pcall(resetAutoBatMotion) end
                    if autoBatSetVisual then autoBatSetVisual(false) end
                    if mobBtnRefs and mobBtnRefs.autoBat then mobBtnRefs.autoBat(false) end
                end
                if autoLeftEnabled then
                    autoLeftEnabled=false
                    if stopAutoLeft then pcall(stopAutoLeft) end
                    if autoLeftSetVisual then autoLeftSetVisual(false) end
                    if mobBtnRefs and mobBtnRefs.autoLeft then mobBtnRefs.autoLeft(false) end
                end
                if autoRightEnabled then
                    autoRightEnabled=false
                    if stopAutoRight then pcall(stopAutoRight) end
                    if autoRightSetVisual then autoRightSetVisual(false) end
                    if mobBtnRefs and mobBtnRefs.autoRight then mobBtnRefs.autoRight(false) end
                end
            end
        end
    end)
end
disableAntiKick=function()
    antiKickEnabled=false;brainrotDetected=false
    for _,c in ipairs(_antiKickConns) do pcall(function() c:Disconnect() end) end;_antiKickConns={}
end


local function safeModeGetCountdownLabel()
    local ok, label = pcall(function()
        local pg = LP:FindFirstChild("PlayerGui")
        if not pg then return nil end
        local top = pg:FindFirstChild("DuelsMachineTopFrame")
        if not top then return nil end
        local inner = top:FindFirstChild("DuelsMachineTopFrame") or top
        local timer = inner:FindFirstChild("Timer", true)
        if not timer then return nil end
        return timer:FindFirstChild("Label") or timer:FindFirstChildWhichIsA("TextLabel")
    end)
    return (ok and label) or nil
end
local function safeModeCountdownNumber(text)
    local t = tostring(text or ""):upper():gsub("^%s+", ""):gsub("%s+$", "")
    if t == "GO" or t == "START" or t == "READY" then return true end
    local n = tonumber(t)
    return n ~= nil and n >= 0 and n <= 10
end
local function safeModeInDuelCountdown()
    local label = safeModeGetCountdownLabel()
    return label and safeModeCountdownNumber(label.Text) or false
end
local function safeModeHoldingBrainrot()
    local ok, val = pcall(function() return LP:GetAttribute("Stealing") end)
    if ok and val == true then return true end
    local ok2, val2 = pcall(function() return LP:GetAttribute("AntiKick") end)
    if ok2 and val2 == true then return true end
    if brainrotDetected then return true end
    local char = LP.Character
    if not char then return false end
    local ok3, val3 = pcall(function() return char:GetAttribute("Stealing") end)
    if ok3 and val3 == true then return true end
    for _, tool in ipairs(char:GetChildren()) do
        if tool:IsA("Tool") then
            local n = tool.Name:lower()
            if n:find("brainrot") or n:find("skibidi") or n:find("toilet") then return true end
        end
    end
    return false
end
local function safeModeIsLocked()
    if not safeModeEnabled then return false end
    return safeModeInDuelCountdown() or safeModeHoldingBrainrot()
end
local function safeModeForceStop(reason)
    if autoBatEnabled then
        autoBatEnabled=false
        if stopBatAimbot then pcall(stopBatAimbot) end
        if autoBatSetVisual then autoBatSetVisual(false) end
        if mobBtnRefs and mobBtnRefs.autoBat then mobBtnRefs.autoBat(false) end
    end
    if autoLeftEnabled then
        autoLeftEnabled=false
        if stopAutoLeft then pcall(stopAutoLeft) end
        if autoLeftSetVisual then autoLeftSetVisual(false) end
        if mobBtnRefs and mobBtnRefs.autoLeft then mobBtnRefs.autoLeft(false) end
    end
    if autoRightEnabled then
        autoRightEnabled=false
        if stopAutoRight then pcall(stopAutoRight) end
        if autoRightSetVisual then autoRightSetVisual(false) end
        if mobBtnRefs and mobBtnRefs.autoRight then mobBtnRefs.autoRight(false) end
    end
    if tpBatEnabled then
        tpBatEnabled=false
        if stopTPBat then pcall(stopTPBat) end
    end
end
local function safeModeTryStart()
    if safeModeIsLocked() then
        safeModeForceStop("SAFE MODE LOCK")
        return false
    end
    return true
end
local function enableSafeMode()
    safeModeEnabled = true
end
local function disableSafeMode()
    safeModeEnabled = false
end
if not _G._SafeModeMonitorStarted_K7 then
    _G._SafeModeMonitorStarted_K7 = true
    task.spawn(function()
        while true do
            task.wait(0.2)
            if safeModeEnabled and safeModeIsLocked() then
                safeModeForceStop("SAFE MODE LOCK")
            end
        end
    end)
end


KB={DropBrainrot={kb=nil,gp=nil},AutoLeft={kb=nil,gp=nil},AutoRight={kb=nil,gp=nil},AutoBat={kb=nil,gp=nil},TPBat={kb=nil,gp=nil},TPFloor={kb=nil,gp=nil},GuiHide={kb=nil,gp=nil},SpeedToggle={kb=nil,gp=nil},LaggerToggle={kb=nil,gp=nil},LaggerCarry={kb=nil,gp=nil},AntiDesync={kb=nil,gp=nil},LaggerPanel={kb=nil,gp=nil}}
AP_L1,AP_L2=Vector3.new(-476.47,-6.28,92.73),Vector3.new(-483.12,-4.95,94.81)
AP_R1,AP_R2=Vector3.new(-476.16,-6.52,25.62),Vector3.new(-483.06,-5.03,25.48)
Steal={AutoStealEnabled=false,StealRadius=60,StealDuration=1.4,StealMode="v2",StealRange=10,SemiHoldMin=1.3,EntryDelay=0.3,HoldMax=2.6,Data={},plotCache={},plotCacheTime={},cachedPrompts={},promptCacheTime=0}
lastStealTick=0
isStealing,stealStartTime=false,nil
Conns={autoSteal=nil,antiRag=nil,batCounter=nil,anchor={}}
MEDUSA_COOLDOWN=25;batCounterDebounce=false
modeValLbl=nil;lastMoveDir=Vector3.new(0,0,0)
_lastRagExitTime=0
MOVE_KEYS={[Enum.KeyCode.W]=true,[Enum.KeyCode.A]=true,[Enum.KeyCode.S]=true,[Enum.KeyCode.D]=true,[Enum.KeyCode.Up]=true,[Enum.KeyCode.Left]=true,[Enum.KeyCode.Down]=true,[Enum.KeyCode.Right]=true}
local function isRagdollState(hum)
    if not hum then return true end
    local st=hum:GetState()
    return hum.PlatformStand or st==Enum.HumanoidStateType.Physics or st==Enum.HumanoidStateType.Ragdoll
end
local function isMyPlotByName(plotName)
    local ct=tick()
    if Steal.plotCache[plotName] ~= nil and (ct-(Steal.plotCacheTime[plotName] or 0)) < 2 then
        return Steal.plotCache[plotName]
    end
    local plots=workspace:FindFirstChild("Plots"); if not plots then Steal.plotCache[plotName]=false; Steal.plotCacheTime[plotName]=ct; return false end
    local plot=plots:FindFirstChild(plotName); if not plot then Steal.plotCache[plotName]=false; Steal.plotCacheTime[plotName]=ct; return false end
    local sign=plot:FindFirstChild("PlotSign")
    if sign then
        local yb=sign:FindFirstChild("YourBase")
        if yb and yb:IsA("BillboardGui") then
            local r = yb.Enabled==true
            Steal.plotCache[plotName]=r; Steal.plotCacheTime[plotName]=ct
            return r
        end
        local sg = sign:FindFirstChild("SurfaceGui")
        local frame = sg and sg:FindFirstChild("Frame")
        local lbl = frame and frame:FindFirstChild("TextLabel")
        if lbl and lbl.Text and lbl.Text ~= "Empty Base" then
            local owner = lbl.Text:gsub("'s [Bb]ase$", ""):gsub("%s+$", "")
            local r = (owner == LP.DisplayName) or (owner == LP.Name)
            Steal.plotCache[plotName]=r; Steal.plotCacheTime[plotName]=ct
            return r
        end
    end
    Steal.plotCache[plotName]=false; Steal.plotCacheTime[plotName]=ct
    return false
end

local function collectPromptFromSpawn(sp)
    if not sp then return nil end
    local att = sp:FindFirstChild("PromptAttachment")
    if att then
        for _,child in ipairs(att:GetChildren()) do
            if child:IsA("ProximityPrompt") then return child end
        end
    end
    for _,child in ipairs(sp:GetDescendants()) do
        if child:IsA("ProximityPrompt") then return child end
    end
    return nil
end

local function grabConnectionFns(signal)
    local out = {}
    if not signal then return out end
    local list = nil
    pcall(function()
        if getconnections then
            list = getconnections(signal)
        elseif get_signal_cons then
            list = get_signal_cons(signal)
        end
    end)
    if type(list) ~= "table" then return out end
    for _,c in ipairs(list) do
        local fn = nil
        pcall(function() fn = c.Function or c.func or c.Fn end)
        if type(fn) == "function" then
            table.insert(out, fn)
        end
    end
    return out
end

local function findNearestPromptTarget()
    local char = LP.Character
    if not char then return nil, nil, math.huge end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return nil, nil, math.huge end
    local plots = workspace:FindFirstChild("Plots")
    if not plots then return nil, nil, math.huge end
    local nearest, nearestSp, dist = nil, nil, math.huge
    local radius = ((Steal.StealMode or "normal") == "semi") and (Steal.StealRange or 10) or (Steal.StealRadius or 60)
    for _, plot in ipairs(plots:GetChildren()) do
        if not isMyPlotByName(plot.Name) then
            local pods = plot:FindFirstChild("AnimalPodiums")
            if pods then
                for _, pod in ipairs(pods:GetChildren()) do
                    local base = pod:FindFirstChild("Base")
                    local sp = base and base:FindFirstChild("Spawn")
                    if sp then
                        local d = (sp.Position - root.Position).Magnitude
                        if d <= radius and d < dist then
                            local prompt = collectPromptFromSpawn(sp)
                            if prompt then
                                nearest = prompt
                                nearestSp = sp
                                dist = d
                            end
                        end
                    end
                end
            end
        end
    end
    return nearest, nearestSp, dist
end

local function distToSpawn(sp)
    local char = LP.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if not hrp or not sp then return math.huge end
    return (hrp.Position - sp.Position).Magnitude
end

local function buildStealCallbacks(prompt)
    if not prompt then return nil end
    if Steal.Data[prompt] then return Steal.Data[prompt] end
    local data = { hold = {}, trigger = {}, ready = true, useFallback = false }
    data.hold = grabConnectionFns(prompt.PromptButtonHoldBegan)
    data.trigger = grabConnectionFns(prompt.Triggered)
    if #data.hold == 0 and #data.trigger == 0 then
        data.useFallback = true
    end
    Steal.Data[prompt] = data
    return data
end

local function executeStealNormal(prompt, spawnPart)
    if not prompt or not prompt.Parent then return end
    if isStealing then return end
    local now = tick()
    if now - (lastStealTick or 0) < 0.08 then return end

    local data = buildStealCallbacks(prompt)
    if not data or not data.ready then return end

    data.ready = false
    isStealing = true
    stealStartTime = now
    lastStealTick = now
    local holdDur = Steal.StealDuration or 1.3

    task.spawn(function()
        local ok = false
        local conn
        pcall(function()
            if setStealStatusText then setStealStatusText("STEALING", 0) end
            conn = RunService.Heartbeat:Connect(function()
                if not isStealing then
                    if conn then conn:Disconnect() end
                    return
                end
                local prog = math.clamp((tick() - (stealStartTime or tick())) / holdDur, 0, 1)
                if progressFill then progressFill.Size = UDim2.new(prog, 0, 1, 0) end
                if setStealStatusText then setStealStatusText("STEALING", prog) end
            end)
        end)

        if not data.useFallback and (#data.hold > 0 or #data.trigger > 0) then
            pcall(function()
                for _, fn in ipairs(data.hold) do task.spawn(fn) end
                task.wait(holdDur)
                for _, fn in ipairs(data.trigger) do task.spawn(fn) end
                ok = true
            end)
        end
        if not ok then
            pcall(function()
                if fireproximityprompt then
                    fireproximityprompt(prompt)
                    ok = true
                    task.wait(holdDur)
                end
            end)
        end
        if not ok then
            pcall(function()
                prompt:InputHoldBegin()
                task.wait(holdDur)
                prompt:InputHoldEnd()
                ok = true
            end)
        end

        pcall(function()
            if conn then conn:Disconnect() end
            if progressFill then progressFill.Size = UDim2.new(1, 0, 1, 0) end
            if setStealStatusText then setStealStatusText(ok and "STOLE" or "MISSED") end
        end)
        task.wait(0.35)
        data.ready = true
        isStealing = false
        stealStartTime = nil
        pcall(function()
            if progressFill then progressFill.Size = UDim2.new(0, 0, 1, 0) end
            if Steal.AutoStealEnabled and setStealStatusText then setStealStatusText("SEARCHING") end
        end)
    end)
end

local function executeStealSemi(prompt, spawnPart)
    if not prompt or not prompt.Parent then return end
    if isStealing then return end
    local now = tick()
    if now - (lastStealTick or 0) < 0.05 then return end

    local data = buildStealCallbacks(prompt)
    if not data or not data.ready then return end

    data.ready = false
    isStealing = true
    stealStartTime = now
    lastStealTick = now

    local holdMin = Steal.StealDuration or 1.3
    local holdMax = Steal.HoldMax or math.max(holdMin + 1.2, 2.6)
    local stealRange = Steal.StealRange or 10
    local entryDelay = Steal.EntryDelay or 0.3

    task.spawn(function()
        local fired = false
        local conn
        pcall(function()
            if setStealStatusText then setStealStatusText("HOLDING", 0) end
            conn = RunService.Heartbeat:Connect(function()
                if not isStealing then
                    if conn then conn:Disconnect() end
                    return
                end
                local prog = math.clamp((tick() - (stealStartTime or tick())) / holdMax, 0, 1)
                if progressFill then progressFill.Size = UDim2.new(prog, 0, 1, 0) end
                if setStealStatusText then
                    local d = distToSpawn(spawnPart)
                    if d <= stealRange then
                        setStealStatusText("STEALING", prog)
                    else
                        setStealStatusText("CLOSER", prog)
                    end
                end
            end)
        end)

        if not data.useFallback and #data.hold > 0 then
            for _, fn in ipairs(data.hold) do task.spawn(fn) end
        else
            pcall(function()
                if prompt.InputHoldBegin then prompt:InputHoldBegin() end
            end)
        end

        task.wait(holdMin)

        local alreadyInRange = distToSpawn(spawnPart) <= stealRange
        local t0 = stealStartTime or tick()
        while isStealing do
            local elapsed = tick() - t0
            if elapsed > holdMax then break end
            if not prompt.Parent then break end
            if distToSpawn(spawnPart) <= stealRange then
                if not alreadyInRange and entryDelay > 0 then
                    task.wait(entryDelay)
                end
                if not data.useFallback and #data.trigger > 0 then
                    for _, fn in ipairs(data.trigger) do task.spawn(fn) end
                    fired = true
                else
                    pcall(function()
                        if fireproximityprompt then
                            fireproximityprompt(prompt)
                            fired = true
                        elseif prompt.InputHoldEnd then
                            prompt:InputHoldEnd()
                            fired = true
                        end
                    end)
                end
                break
            end
            task.wait()
        end

        if not fired and prompt.Parent then
            pcall(function()
                if #data.trigger > 0 then
                    for _, fn in ipairs(data.trigger) do task.spawn(fn) end
                    fired = true
                elseif fireproximityprompt then
                    fireproximityprompt(prompt)
                    fired = true
                end
            end)
        end

        pcall(function()
            if conn then conn:Disconnect() end
            if progressFill then progressFill.Size = UDim2.new(1, 0, 1, 0) end
            if setStealStatusText then setStealStatusText(fired and "STOLE" or "MISSED") end
        end)
        task.wait(0.35)
        data.ready = true
        isStealing = false
        stealStartTime = nil
        pcall(function()
            if progressFill then progressFill.Size = UDim2.new(0, 0, 1, 0) end
            if Steal.AutoStealEnabled and setStealStatusText then setStealStatusText("SEARCHING") end
        end)
    end)
end

local function executeSteal(prompt, spawnPart)
    if (Steal.StealMode or "normal") == "semi" then
        executeStealSemi(prompt, spawnPart)
    else
        executeStealNormal(prompt, spawnPart)
    end
end


AdaptK7Extras.initZeySteal=function()
    AdaptK7Extras.ZeySteal=AdaptK7Extras.ZeySteal or {}
    local M=AdaptK7Extras.ZeySteal
    local player=LP
    M.Steal=Steal
    M.stealMode=(Steal.StealMode=="semi") and "Semi" or ((Steal.StealMode=="normal") and "Normal" or "V2")
    M.isStealing=false;M.stealStartTime=0;M.stealConn=nil;M.progressConn=nil
    M.animalCache={};M.promptCache={};M.stealCache={};M.V2=M.V2 or {}
    M.Semi=M.Semi or {enabled=false,holdMin=1.3,holdMax=2.6,entryDelay=0.3,cooldown=0.05,primeRange=80,radius=10,conn=nil,scanThread=nil,plotSync={caches={},connections={}},animals={},promptCache={},internalCache={},state={active=false,startTime=0,phase="idle",label="",lastResult="",lastResultTime=0},plots=nil,syncReady=false}
    M.Semi.holdMin=tonumber(Steal.SemiHoldMin) or 1.3
    M.Semi.holdMax=tonumber(Steal.HoldMax) or 2.6
    M.Semi.entryDelay=tonumber(Steal.EntryDelay) or 0.3
    M.Semi.radius=math.min(tonumber(Steal.StealRange) or 10,10)
    function M.getActiveStealRadius()
        if M.stealMode=="Semi" then return math.min(tonumber(M.Semi.radius) or 10,10) end
        return tonumber(Steal.StealRadius) or 60
    end
    function M.updateStatusRadius() end
    function M.updateStealProgress(progress,label)
        progress=math.clamp(tonumber(progress) or 0,0,1)
        isStealing=M.isStealing==true;stealStartTime=M.stealStartTime
        if progressFill then progressFill.Size=UDim2.new(progress,0,1,0) end
        if setStealStatusText then
            if progress>=1 then setStealStatusText("STOLE",1)
            elseif progress<=0 then setStealStatusText(Steal.AutoStealEnabled and "SEARCHING" or "MISSED",0)
            else setStealStatusText("STEALING",progress) end
        end
    end
if not fireproximityprompt then
    fireproximityprompt = (getgenv and getgenv().fireproximityprompt)
        or (genv and genv().fireproximityprompt)
        or function(prompt)
            pcall(function()
                prompt:InputHoldBegin()
                task.wait(0.05)
                prompt:InputHoldEnd()
            end)
        end
end

local function isMyPlot(plotName)
    local plots = workspace:FindFirstChild("Plots")
    if not plots then return false end
    local plot = plots:FindFirstChild(plotName)
    if not plot then return false end
    local sign = plot:FindFirstChild("PlotSign")
    if sign then
        local yb = sign:FindFirstChild("YourBase")
        if yb and yb:IsA("BillboardGui") then return yb.Enabled == true end
    end
    return false
end

local function scanPlotNormal(plot)
    if not plot or not plot:IsA("Model") then return end
    if isMyPlot(plot.Name) then return end
    local podiums = plot:FindFirstChild("AnimalPodiums")
    if not podiums then return end
    for _, pod in ipairs(podiums:GetChildren()) do
        if pod:IsA("Model") and pod:FindFirstChild("Base") then
            local uid = plot.Name .. "_" .. pod.Name
            for _, ex in ipairs(M.animalCache) do if ex.uid == uid then return end end
            table.insert(M.animalCache, {
                name = pod.Name,
                plot = plot.Name,
                slot = pod.Name,
                worldPosition = pod:GetPivot().Position,
                uid = uid,
            })
        end
    end
end

local function findPromptNormal(ad)
    if not ad then return nil end
    local cp = M.promptCache[ad.uid]
    if cp and cp.Parent then return cp end
    local plots = workspace:FindFirstChild("Plots")
    if not plots then return nil end
    local plot = plots:FindFirstChild(ad.plot)
    if not plot then return nil end
    local pods = plot:FindFirstChild("AnimalPodiums")
    if not pods then return nil end
    local pod = pods:FindFirstChild(ad.slot)
    if not pod then return nil end
    local base = pod:FindFirstChild("Base")
    if not base then return nil end
    local spawn = base:FindFirstChild("Spawn")
    if not spawn then return nil end
    local att = spawn:FindFirstChild("PromptAttachment")
    local prompt = nil
    if att then
        for _, p in ipairs(att:GetChildren()) do
            if p:IsA("ProximityPrompt") then prompt = p; break end
        end
    end
    if not prompt then
        for _, obj in ipairs(spawn:GetDescendants()) do
            if obj:IsA("ProximityPrompt") then prompt = obj; break end
        end
    end
    if prompt then M.promptCache[ad.uid] = prompt end
    return prompt
end

local function nearestAnimalNormal()
    local char = player.Character
    if not char then return nil end
    local hrp = char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("UpperTorso")
    if not hrp then return nil end
    local best, bestD = nil, math.huge
    for _, ad in ipairs(M.animalCache) do
        if not isMyPlot(ad.plot) and ad.worldPosition then
            local d = (hrp.Position - ad.worldPosition).Magnitude
            if d < bestD then bestD = d; best = ad end
        end
    end
    return best, bestD
end

local function buildCallbacks(prompt)
    if M.stealCache[prompt] then return end
    local data = { holdCallbacks = {}, triggerCallbacks = {}, ready = true }
    local ok1, c1 = pcall(getconnections, prompt.PromptButtonHoldBegan)
    if ok1 and type(c1) == "table" then
        for _, conn in ipairs(c1) do
            if type(conn.Function) == "function" then
                table.insert(data.holdCallbacks, conn.Function)
            end
        end
    end
    local ok2, c2 = pcall(getconnections, prompt.Triggered)
    if ok2 and type(c2) == "table" then
        for _, conn in ipairs(c2) do
            if type(conn.Function) == "function" then
                table.insert(data.triggerCallbacks, conn.Function)
            end
        end
    end
    if #data.holdCallbacks > 0 or #data.triggerCallbacks > 0 then
        M.stealCache[prompt] = data
    end
end

local function execStealNormal(prompt, animalName)
    local data = M.stealCache[prompt]
    if not data or not data.ready then return false end
    data.ready = false
    M.isStealing = true
    M.stealStartTime = tick()
    M.updateStealProgress(0.1)

    if M.progressConn then M.progressConn:Disconnect() end
    M.progressConn = RunService.Heartbeat:Connect(function()
        if not M.isStealing then
            M.progressConn:Disconnect()
            M.progressConn = nil
            return
        end
        local prog = math.clamp((tick() - M.stealStartTime) / M.Steal.StealDuration, 0, 1)
        M.updateStealProgress(prog)
    end)

    task.spawn(function()
        for _, fn in ipairs(data.holdCallbacks) do task.spawn(fn) end
        local elapsed = 0
        while elapsed < M.Steal.StealDuration do elapsed = elapsed + task.wait() end
        for _, fn in ipairs(data.triggerCallbacks) do task.spawn(fn) end
        task.wait(0.01)
        if M.progressConn then M.progressConn:Disconnect(); M.progressConn = nil end
        M.isStealing = false
        M.updateStealProgress(0)
        data.ready = true
    end)
    return true
end

function M.startNormalSteal()
    if M.stealConn then return end
    M.stealConn = RunService.Heartbeat:Connect(function()
        if not M.Steal.AutoStealEnabled or (M.stealMode ~= "Normal" and M.stealMode ~= "V1") or M.isStealing then return end
        local target, dist = nearestAnimalNormal()
        if not target then return end
        if dist > M.getActiveStealRadius() then return end
        local prompt = M.promptCache[target.uid]
        if not prompt or not prompt.Parent then
            prompt = findPromptNormal(target)
        end
        if prompt then
            buildCallbacks(prompt)
            execStealNormal(prompt, target.name)
        end
    end)
end

function M.stopNormalSteal()
    if M.stealConn then
        M.stealConn:Disconnect()
        M.stealConn = nil
    end
    M.isStealing = false
    if M.progressConn then M.progressConn:Disconnect(); M.progressConn = nil end
    M.updateStealProgress(0)
end

do
    local V2 = M.V2 or {}
    M.V2 = V2
    V2.enabled = false
    V2.isStealing = false
    V2.data = V2.data or {}
    V2.conn = nil
    V2.progressConn = nil
    V2.stealStartTime = 0
    V2.paused = false
    V2.pauseStarted = nil
    V2.pausedDuration = 0
    local function barSet(p, label)
        local progress = math.clamp(tonumber(p) or 0, 0, 1)
        local pct = math.floor(progress * 100 + 0.5)
        local text = nil
        if type(label) == "string" and label ~= "" then
            text = string.upper(label)
            if progress > 0 then text = text .. "  " .. tostring(pct) .. "%" end
        end
        M.updateStealProgress(progress, text)
    end
    local function barReset()
        M.updateStealProgress(0)
    end
    local function execStealV2(prompt)
        if V2.isStealing then return end
        if not prompt then return end
        buildCallbacks(prompt)
        local data = M.stealCache[prompt]
        if not data or not data.ready then return end
        data.ready = false
        V2.isStealing = true
        M.isStealing = true
        V2.stealStartTime = tick()
        V2.paused = false
        V2.pauseStarted = nil
        V2.pausedDuration = 0
        local duration = math.max(tonumber(M.Steal.StealDuration) or 1.4, 0.05)
        local pauseFraction = 0.75
        local finishFraction = 1 - pauseFraction
        local targetPart = prompt:FindFirstAncestorWhichIsA("BasePart")
        local restarting = false
        local function modeStillActive()
            return V2.enabled and M.stealMode == "V2" and V2.isStealing
        end
        local function isTargetInCurrentRadius()
            local char = player.Character
            local root = char and (char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("UpperTorso"))
            local radius = M.getActiveStealRadius()
            return root and targetPart and targetPart.Parent and (root.Position - targetPart.Position).Magnitude <= radius
        end
        local function isCloseEnoughToGrab()
            if autoLeftEnabled or autoRightEnabled then
                return isTargetInCurrentRadius()
            end
            local char = player.Character
            local root = char and (char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("UpperTorso"))
            local closeRange = math.min(M.getActiveStealRadius(), 12)
            return root and targetPart and targetPart.Parent and (root.Position - targetPart.Position).Magnitude <= closeRange
        end
        local function canStillGrab()
            if not prompt or not prompt.Parent or not targetPart or not targetPart.Parent then return false end
            if autoLeftEnabled or autoRightEnabled then
                return isTargetInCurrentRadius() or true
            end
            if not prompt.Enabled then return false end
            if not tostring(prompt.ActionText):lower():find("steal", 1, true) then return false end
            return isTargetInCurrentRadius()
        end
        local function restartFromZero()
            if restarting then return end
            restarting = true
            V2.paused = false
            V2.pauseStarted = nil
            V2.pausedDuration = 0
            if V2.progressConn then V2.progressConn:Disconnect(); V2.progressConn = nil end
            barReset()
            data.ready = true
            V2.isStealing = false
            M.isStealing = false
        end
        if V2.progressConn then V2.progressConn:Disconnect() end
        V2.progressConn = RunService.Heartbeat:Connect(function()
            if not V2.isStealing then
                if V2.progressConn then V2.progressConn:Disconnect(); V2.progressConn = nil end
                return
            end
            if not modeStillActive() or not canStillGrab() then
                restartFromZero()
                return
            end
            local elapsed = tick() - V2.stealStartTime - (V2.pausedDuration or 0)
            if V2.paused and V2.pauseStarted then
                barSet(pauseFraction, "BRAINROT")
            else
                local p = math.clamp(elapsed / duration, 0, 1)
                if p > pauseFraction then p = pauseFraction end
                barSet(p)
            end
        end)
        task.spawn(function()
            for _, fn in ipairs(data.holdCallbacks) do task.spawn(fn) end
            if #data.holdCallbacks == 0 and #data.triggerCallbacks == 0 and fireproximityprompt then
                pcall(function()
                    fireproximityprompt(prompt, duration)
                end)
            end
            local holdStart = tick()
            while tick() - holdStart < duration * pauseFraction do
                if not modeStillActive() or not canStillGrab() then
                    restartFromZero()
                    return
                end
                task.wait()
            end
            V2.paused = true
            V2.pauseStarted = tick()
            barSet(pauseFraction, "BRAINROT")
            local wasAbleToGrab = false
            local waitStart = tick()
            while modeStillActive() do
                if not canStillGrab() then
                    restartFromZero()
                    return
                end
                if isCloseEnoughToGrab() then
                    wasAbleToGrab = true
                    break
                end
                if tick() - waitStart >= 0.95 then
                    wasAbleToGrab = true
                    break
                end
                task.wait()
            end
            if not modeStillActive() or not canStillGrab() then
                restartFromZero()
                return
            end
            if wasAbleToGrab then
                V2.pausedDuration = (V2.pausedDuration or 0) + (tick() - V2.pauseStarted)
                V2.paused = false
                V2.pauseStarted = nil
                local finishStart = tick()
                while tick() - finishStart < duration * finishFraction do
                    if not modeStillActive() or not canStillGrab() then
                        restartFromZero()
                        return
                    end
                    task.wait()
                end
                for _, fn in ipairs(data.triggerCallbacks) do task.spawn(fn) end
                if V2.progressConn then V2.progressConn:Disconnect(); V2.progressConn = nil end
                barSet(1, "SUCCESS")
                task.wait(0.05)
                barReset()
                data.ready = true
                V2.isStealing = false
                M.isStealing = false
            else
                restartFromZero()
            end
        end)
    end
    function M.startV2Steal()
        V2.enabled = true
        V2.isStealing = false
        if V2.conn then V2.conn:Disconnect(); V2.conn = nil end
        V2.conn = RunService.Heartbeat:Connect(function()
            if not V2.enabled then return end
            if not M.Steal.AutoStealEnabled then return end
            if M.stealMode ~= "V2" then M.stopV2Steal(); return end
            if V2.isStealing then return end
            local target, dist = nearestAnimalNormal()
            if not target then return end
            if dist > M.getActiveStealRadius() then return end
            local prompt = M.promptCache[target.uid]
            if not prompt or not prompt.Parent then
                prompt = findPromptNormal(target)
            end
            if prompt then
                execStealV2(prompt)
            end
        end)
    end
    function M.stopV2Steal()
        V2.enabled = false
        V2.isStealing = false
        V2.paused = false
        V2.pauseStarted = nil
        V2.pausedDuration = 0
        if V2.conn then V2.conn:Disconnect(); V2.conn = nil end
        if V2.progressConn then V2.progressConn:Disconnect(); V2.progressConn = nil end
        M.isStealing = false
        barReset()
    end
end

do
    local A = M.Semi
    if A.conn then pcall(function() A.conn:Disconnect() end); A.conn = nil end
    A.enabled = false
    A.holdMin = tonumber(A.holdMin) or 1.3
    A.holdMax = tonumber(A.holdMax) or 2.6
    A.entryDelay = tonumber(A.entryDelay) or 0.3
    A.cooldown = tonumber(A.cooldown) or 0.05
    A.primeRange = tonumber(A.primeRange) or 80
    A.radius = math.min(tonumber(A.radius) or 10, 10)
    A.plotSync = A.plotSync or {caches = {}, connections = {}}
    A.animals = A.animals or {}
    A.promptCache = A.promptCache or {}
    A.internalCache = A.internalCache or {}
    A.state = A.state or {active = false, startTime = 0, phase = "idle", label = "", lastResult = "", lastResultTime = 0}

    local function barSet(p, label)
        local progress = math.clamp(tonumber(p) or 0, 0, 1)
        local pct = math.floor(progress * 100 + 0.5)
        local text = nil
        if type(label) == "string" and label ~= "" then
            text = string.upper(label)
            if progress > 0 then
                text = text .. "  " .. tostring(pct) .. "%"
            end
        end
        M.updateStealProgress(progress, text)
    end
    local function barReset()
        M.updateStealProgress(0)
    end
    local function rootPart()
        local char = player.Character
        return char and (char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("UpperTorso")) or nil
    end
    local function splitPath(path)
        if typeof(path) == "table" then return path end
        local out = {}
        for part in string.gmatch(tostring(path), "[^%.]+") do
            table.insert(out, tonumber(part) or part)
        end
        return out
    end
    local function resolvePath(path, root)
        local current, parent, key = root, nil, nil
        for _, part in ipairs(splitPath(path)) do
            parent = current
            key = part
            current = current and current[part] or nil
        end
        return current, parent, key
    end
    local function applySyncDiff(channelName, packet)
        local cache = A.plotSync.caches[channelName]
        if typeof(cache) ~= "table" then return end
        local path, action, a, b = packet[1], packet[2], packet[3], packet[4]
        local current, parent, key = resolvePath(path, cache)
        if action == "Changed" then
            if parent ~= nil then parent[key] = a end
        elseif action == "ArrayInsert" then
            if current ~= nil then table.insert(current, b, a) end
        elseif action == "ArrayRemoved" then
            if current ~= nil then table.remove(current, b) end
        elseif action == "DictionaryInsert" then
            if current ~= nil then current[b] = a end
        elseif action == "DictionaryRemoved" then
            if current ~= nil then current[b] = nil end
        end
    end
    local function attachPlotChannel(remote, plots, requestData)
        if A.plotSync.connections[remote] then return end
        local channelName = tostring(remote.Name)
        if not plots:FindFirstChild(channelName) then return end
        if requestData and A.plotSync.caches[channelName] == nil then
            local ok, data = pcall(function() return requestData:InvokeServer(channelName) end)
            A.plotSync.caches[channelName] = (ok and typeof(data) == "table") and data or {}
        elseif A.plotSync.caches[channelName] == nil then
            A.plotSync.caches[channelName] = {}
        end
        A.plotSync.connections[remote] = remote.OnClientEvent:Connect(function(queue)
            for _, packet in ipairs(queue) do applySyncDiff(channelName, packet) end
        end)
    end

    function M.initSemiSync()
        if A.syncReady then return true end
        local ok = pcall(function()
            local rs = game:GetService("ReplicatedStorage")
            A.packages = rs:WaitForChild("Packages", 10)
            A.datas = rs:WaitForChild("Datas", 10)
            A.plots = workspace:WaitForChild("Plots", 10)
            if not (A.packages and A.datas and A.plots) then return end
            A.animalsData = require(A.datas:WaitForChild("Animals", 10))
            local sync = A.packages:WaitForChild("Synchronizer", 10)
            A.channelFolder = sync:WaitForChild("Channel", 10)
            A.routeRemote = sync:WaitForChild("CommunicationRoute", 10)
            A.requestData = sync:FindFirstChild("RequestData")
            for _, child in ipairs(A.channelFolder:GetChildren()) do
                if child:IsA("RemoteEvent") then attachPlotChannel(child, A.plots, A.requestData) end
            end
            A.channelFolder.ChildAdded:Connect(function(child)
                if child:IsA("RemoteEvent") then attachPlotChannel(child, A.plots, A.requestData) end
            end)
            A.routeRemote.OnClientEvent:Connect(function(actions)
                for _, action in ipairs(actions) do
                    local kind, channelName = action[1], tostring(action[2])
                    if A.plots and A.plots:FindFirstChild(channelName) then
                        if kind == "ListenerAdded" then
                            local remote = A.channelFolder and A.channelFolder:FindFirstChild(channelName)
                            if remote and remote:IsA("RemoteEvent") then attachPlotChannel(remote, A.plots, A.requestData) end
                        elseif kind == "ListenerRemoved" then
                            for remote, conn in pairs(A.plotSync.connections) do
                                if tostring(remote.Name) == channelName then
                                    pcall(function() conn:Disconnect() end)
                                    A.plotSync.connections[remote] = nil
                                    A.plotSync.caches[channelName] = nil
                                    break
                                end
                            end
                        end
                    end
                end
            end)
            A.syncReady = true
        end)
        return ok and A.syncReady == true
    end

    local function getPlotOwner(plot)
        local sign = plot and plot:FindFirstChild("PlotSign")
        local frame = sign and sign:FindFirstChild("SurfaceGui") and sign.SurfaceGui:FindFirstChild("Frame")
        local label = frame and frame:FindFirstChild("TextLabel")
        if not label or label.Text == "Empty Base" then return nil end
        return label.Text:gsub("'s [Bb]ase$", ""):gsub("%s+$", "")
    end
    local function isMyBaseAnimal(animalData)
        if not animalData or not animalData.plot or not A.plots then return false end
        local plot = A.plots:FindFirstChild(animalData.plot)
        if not plot then return false end
        local owner = getPlotOwner(plot)
        return owner == player.DisplayName or owner == player.Name
    end
    local function podiumFor(animalData)
        local plot = A.plots and A.plots:FindFirstChild(animalData.plot)
        local podiums = plot and plot:FindFirstChild("AnimalPodiums")
        return podiums and podiums:FindFirstChild(animalData.slot) or nil
    end
    local function animalPos(animalData)
        local podium = podiumFor(animalData)
        return podium and podium:GetPivot().Position or nil
    end
    local function distToAnimal(animalData)
        local root = rootPart()
        local pos = animalPos(animalData)
        return root and pos and (root.Position - pos).Magnitude or math.huge
    end
    local function findPromptForAnimal(animalData)
        if not animalData then return nil end
        local cached = A.promptCache[animalData.uid]
        if cached and cached.Parent then return cached end
        local podium = podiumFor(animalData)
        local base = podium and podium:FindFirstChild("Base")
        local spawn = base and base:FindFirstChild("Spawn")
        local attach = spawn and spawn:FindFirstChild("PromptAttachment")
        if not attach then return nil end
        for _, prompt in ipairs(attach:GetChildren()) do
            if prompt:IsA("ProximityPrompt") then
                A.promptCache[animalData.uid] = prompt
                return prompt
            end
        end
        return nil
    end

    function M.scanAllPlotsSemi()
        if not M.initSemiSync() then return 0 end
        local newCache = {}
        for _, plot in ipairs(A.plots:GetChildren()) do
            local cache = A.plotSync.caches[plot.Name]
            local animalList = cache and cache.AnimalList
            if typeof(animalList) == "table" then
                for slot, animalData in pairs(animalList) do
                    if type(animalData) == "table" then
                        local animalName = animalData.Index
                        local info = A.animalsData and A.animalsData[animalName]
                        if info then
                            table.insert(newCache, {
                                name = info.DisplayName or animalName,
                                plot = plot.Name,
                                slot = tostring(slot),
                                uid = plot.Name .. "_" .. tostring(slot),
                            })
                        end
                    end
                end
            end
        end
        A.animals = newCache
        return #newCache
    end

    local function pickClosest()
        local root = rootPart()
        if not root then return nil end
        local best, bestDist = nil, math.huge
        for _, animalData in ipairs(A.animals) do
            if not isMyBaseAnimal(animalData) then
                local pos = animalPos(animalData)
                local dist = pos and (root.Position - pos).Magnitude or math.huge
                if dist <= (A.primeRange or 80) and dist < bestDist then
                    best, bestDist = animalData, dist
                end
            end
        end
        return best
    end
    local function buildCallbacks(prompt)
        if A.internalCache[prompt] then return end
        local data = {holdCallbacks = {}, triggerCallbacks = {}, ready = true}
        local okHold, holds = pcall(getconnections, prompt.PromptButtonHoldBegan)
        if okHold and type(holds) == "table" then
            for _, conn in ipairs(holds) do
                if type(conn.Function) == "function" then table.insert(data.holdCallbacks, conn.Function) end
            end
        end
        local okTrigger, triggers = pcall(getconnections, prompt.Triggered)
        if okTrigger and type(triggers) == "table" then
            for _, conn in ipairs(triggers) do
                if type(conn.Function) == "function" then table.insert(data.triggerCallbacks, conn.Function) end
            end
        end
        if #data.holdCallbacks > 0 or #data.triggerCallbacks > 0 then A.internalCache[prompt] = data end
    end
    local function executeSemi(prompt, animalData)
        if not prompt or not prompt.Parent or not animalData then return false end
        buildCallbacks(prompt)
        local data = A.internalCache[prompt]
        if not data or not data.ready then return false end
        data.ready = false
        A.state.active = true
        A.state.startTime = tick()
        A.state.phase = "holding"
        A.state.label = animalData.name or "Animal"
        M.isStealing = true
        M.stealStartTime = A.state.startTime
        task.spawn(function()
            local startTime = A.state.startTime
            for _, fn in ipairs(data.holdCallbacks) do task.spawn(function() pcall(fn) end) end
            while A.enabled and M.stealMode == "Semi" and tick() - startTime < (A.holdMin or 1.3) do
                local elapsed = tick() - startTime
                A.state.phase = "holding"
                barSet(elapsed / (A.holdMax or 2.6), "HOLDING " .. tostring(A.state.label))
                task.wait()
            end
            A.state.phase = "waitingRange"
            local alreadyInRange = distToAnimal(animalData) <= (tonumber(A.radius) or 10)
            local fired = false
            while A.enabled and M.stealMode == "Semi" and prompt.Parent do
                local elapsed = tick() - startTime
                if elapsed > (A.holdMax or 2.6) then break end
                barSet(elapsed / (A.holdMax or 2.6), "MOVE CLOSER  " .. tostring(A.state.label))
                if distToAnimal(animalData) <= (tonumber(A.radius) or 10) then
                    if not alreadyInRange then task.wait(A.entryDelay or 0.3) end
                    if A.enabled and M.stealMode == "Semi" then
                        for _, fn in ipairs(data.triggerCallbacks) do task.spawn(function() pcall(fn) end) end
                        fired = true
                    end
                    break
                end
                task.wait()
            end
            A.state.lastResult = fired and ("Stole " .. tostring(A.state.label)) or ("Missed window: " .. tostring(A.state.label))
            A.state.active = false
            A.state.phase = "idle"
            A.state.lastResultTime = tick()
            if fired then
                barSet(1, "STOLE " .. tostring(A.state.label))
            else
                barSet(0, A.state.lastResult)
            end
            task.wait(A.cooldown or 0.05)
            data.ready = true
            M.isStealing = false
            barReset()
        end)
        return true
    end

    function M.stopSemiSteal()
        A.enabled = false
        if A.conn then A.conn:Disconnect(); A.conn = nil end
        A.state.active = false
        A.state.phase = "idle"
        M.isStealing = false
        barReset()
    end

    function M.startSemiSteal()
        A.radius = math.min(tonumber(A.radius) or 10, 10)
        A.enabled = true
        M.initSemiSync()
        pcall(M.scanAllPlotsSemi)
        if A.conn then A.conn:Disconnect(); A.conn = nil end
        A.conn = RunService.Heartbeat:Connect(function()
            if not A.enabled then return end
            if not M.Steal.AutoStealEnabled then return end
            if M.stealMode ~= "Semi" then M.stopSemiSteal(); return end
            if A.state.active then return end
            local target = pickClosest()
            if not target then return end
            local prompt = findPromptForAnimal(target)
            if prompt then executeSemi(prompt, target) end
        end)
    end
end


    function M.startAutoSteal()
        M.Steal=Steal
        M.stealMode=(Steal.StealMode=="semi") and "Semi" or ((Steal.StealMode=="normal") and "Normal" or "V2")
        M.Semi.holdMin=tonumber(Steal.SemiHoldMin) or 1.3;M.Semi.holdMax=tonumber(Steal.HoldMax) or 2.6;M.Semi.entryDelay=tonumber(Steal.EntryDelay) or 0.3;M.Semi.radius=math.min(tonumber(Steal.StealRange) or 10,10)
        if M.stealMode=="Normal" then M.startNormalSteal() elseif M.stealMode=="Semi" then M.startSemiSteal() else M.stealMode="V2";M.startV2Steal() end
    end
    function M.stopAutoSteal()
        pcall(M.stopNormalSteal);pcall(M.stopSemiSteal);if M.stopV2Steal then pcall(M.stopV2Steal) end
        M.isStealing=false;isStealing=false;M.updateStealProgress(0)
    end
    M._scanToken=(M._scanToken or 0)+1
    local token=M._scanToken
    task.spawn(function()
        local plots=workspace:FindFirstChild("Plots") or workspace:WaitForChild("Plots",10)
        if not plots then return end
        for _,plot in ipairs(plots:GetChildren()) do if plot:IsA("Model") then scanPlotNormal(plot) end end
        plots.ChildAdded:Connect(function(plot) if M._scanToken~=token then return end;if plot:IsA("Model") then task.wait(0.5);scanPlotNormal(plot) end end)
        while M._scanToken==token do task.wait(5);M.animalCache={};M.promptCache={};M.stealCache={};for _,plot in ipairs(plots:GetChildren()) do if plot:IsA("Model") then scanPlotNormal(plot) end end end
    end)
    task.spawn(function()
        pcall(M.initSemiSync)
        while M._scanToken==token do task.wait(5);if M.Semi.enabled or M.stealMode=="Semi" then pcall(M.scanAllPlotsSemi) end end
    end)
end
AdaptK7Extras.initZeySteal()


startAutoSteal=function()
    Steal.AutoStealEnabled=true
    local M=AdaptK7Extras.ZeySteal
    if not M then AdaptK7Extras.initZeySteal();M=AdaptK7Extras.ZeySteal end
    M.Steal=Steal;M.stealMode=(Steal.StealMode=="semi") and "Semi" or ((Steal.StealMode=="normal") and "Normal" or "V2")
    M.Semi.holdMin=tonumber(Steal.SemiHoldMin) or 1.3;M.Semi.holdMax=tonumber(Steal.HoldMax) or 2.6;M.Semi.entryDelay=tonumber(Steal.EntryDelay) or 0.3;M.Semi.radius=math.min(tonumber(Steal.StealRange) or 10,10)
    pcall(M.startAutoSteal)
end
stopAutoSteal=function()
    Steal.AutoStealEnabled=false
    local M=AdaptK7Extras.ZeySteal
    if M and M.stopAutoSteal then pcall(M.stopAutoSteal) end
    isStealing=false;stealStartTime=nil;lastStealTick=0
    Steal.plotCache={};Steal.plotCacheTime={};Steal.cachedPrompts={}
    pcall(function() if resetProgressBar then resetProgressBar() end end)
end


RunService.Heartbeat:Connect(function(dt)
    _moveDt = dt
    local char = LP.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hum or not hrp then return end
    if isRagdollState(hum) then
        lastMoveDir = Vector3.new(0, 0, 0)
        _speedActive = false
        _lastHorizSpeed = 0
        brakeHorizontal(hrp)
        if speedLabel then speedLabel.Text = "0.0" end
        return
    end
    local spd = getActiveMoveSpeed()
    local moving = false
    local controlled = not autoBatEnabled and not tpBatEnabled and not autoLeftEnabled and not autoRightEnabled

    pcall(function()
        if hum.CameraOffset.Magnitude > 0.05 then
            hum.CameraOffset = Vector3.new(0, 0, 0)
        end
    end)

    if controlled then
        local md = hum.MoveDirection
        if md.Magnitude > 0.05 then
            lastMoveDir = Vector3.new(md.X, 0, md.Z).Unit
            moving = true
            setSpeedConstraint(hrp, lastMoveDir * spd)
        else
            _speedActive = false
            _lastHorizSpeed = 0
            brakeHorizontal(hrp)
        end
    else
        brakeHorizontal(hrp)
    end

    if speedLabel then
        if moving or _speedActive then
            speedLabel.Text = string.format("%.1f", spd)
        else
            speedLabel.Text = "0.0"
        end
    end
end)

LP.CharacterAdded:Connect(function(char)
    task.wait(0.5);setupSpeedIndicator(char);setupRagdollTriggers()
    if medusaCounterEnabled then setupMedusa(char) end
    if batCounterEnabled then startBatCounter() end
    if unwalkEnabled then task.wait(0.5);startUnwalk() end
end)
if LP.Character then setupSpeedIndicator(LP.Character);setupRagdollTriggers() end
alConn,arConn=nil,nil;alPhase,arPhase=1,1
stopAutoLeft=function()
    if alConn then alConn:Disconnect();alConn=nil end;alPhase=1
    local char=LP.Character;if char then local h=char:FindFirstChildOfClass("Humanoid");if h then h:Move(Vector3.zero,false) end end
    if autoLeftSetVisual then autoLeftSetVisual(false) end
    if mobBtnRefs.autoLeft then mobBtnRefs.autoLeft(false) end
end
stopAutoRight=function()
    if arConn then arConn:Disconnect();arConn=nil end;arPhase=1
    local char=LP.Character;if char then local h=char:FindFirstChildOfClass("Humanoid");if h then h:Move(Vector3.zero,false) end end
    if autoRightSetVisual then autoRightSetVisual(false) end
    if mobBtnRefs.autoRight then mobBtnRefs.autoRight(false) end
end
startAutoLeft=function()
    if alConn then alConn:Disconnect() end;alPhase=1
    alConn=RunService.Heartbeat:Connect(function()
        if not autoLeftEnabled then return end
        local char=LP.Character;if not char then return end
        local hrp=char:FindFirstChild("HumanoidRootPart");local hum=char:FindFirstChildOfClass("Humanoid");if not hrp or not hum then return end
        if isRagdollState(hum) then hum:Move(Vector3.zero,false);return end
        local spd=getAutoPathSpeed()
        if alPhase==1 then
            local tgt=Vector3.new(AP_L1.X,hrp.Position.Y,AP_L1.Z)
            if (tgt-hrp.Position).Magnitude<1 then alPhase=2;local d=AP_L2-hrp.Position;local mv=Vector3.new(d.X,0,d.Z).Unit;hum:Move(mv,false);setSpeedConstraint(hrp, Vector3.new(mv.X*spd, 0, mv.Z*spd));return end
            local d=AP_L1-hrp.Position;local mv=Vector3.new(d.X,0,d.Z).Unit;hum:Move(mv,false);setSpeedConstraint(hrp, Vector3.new(mv.X*spd, 0, mv.Z*spd))
        elseif alPhase==2 then
            local tgt=Vector3.new(AP_L2.X,hrp.Position.Y,AP_L2.Z)
            if (tgt-hrp.Position).Magnitude<1 then hum:Move(Vector3.zero,false);clearSpeedConstraint();autoLeftEnabled=false;if alConn then alConn:Disconnect();alConn=nil end;alPhase=1;if autoLeftSetVisual then autoLeftSetVisual(false) end;if mobBtnRefs.autoLeft then mobBtnRefs.autoLeft(false) end;return end
            local d=AP_L2-hrp.Position;local mv=Vector3.new(d.X,0,d.Z).Unit;hum:Move(mv,false);setSpeedConstraint(hrp, Vector3.new(mv.X*spd, 0, mv.Z*spd))
        end
        if autoMoveSwingEnabled and not _alSwingDebounce then
            _alSwingDebounce=true
            local bat=findBat()
            if bat then
                if bat.Parent~=char then pcall(function() hum:EquipTool(bat) end) end
                pcall(function() bat:Activate() end)
            end
            task.delay(autoMoveSwingInterval,function() _alSwingDebounce=false end)
        end
    end)
end
startAutoRight=function()
    if arConn then arConn:Disconnect() end;arPhase=1
    arConn=RunService.Heartbeat:Connect(function()
        if not autoRightEnabled then return end
        local char=LP.Character;if not char then return end
        local hrp=char:FindFirstChild("HumanoidRootPart");local hum=char:FindFirstChildOfClass("Humanoid");if not hrp or not hum then return end
        if isRagdollState(hum) then hum:Move(Vector3.zero,false);return end
        local spd=getAutoPathSpeed()
        if arPhase==1 then
            local tgt=Vector3.new(AP_R1.X,hrp.Position.Y,AP_R1.Z)
            if (tgt-hrp.Position).Magnitude<1 then arPhase=2;local d=AP_R2-hrp.Position;local mv=Vector3.new(d.X,0,d.Z).Unit;hum:Move(mv,false);setSpeedConstraint(hrp, Vector3.new(mv.X*spd, 0, mv.Z*spd));return end
            local d=AP_R1-hrp.Position;local mv=Vector3.new(d.X,0,d.Z).Unit;hum:Move(mv,false);setSpeedConstraint(hrp, Vector3.new(mv.X*spd, 0, mv.Z*spd))
        elseif arPhase==2 then
            local tgt=Vector3.new(AP_R2.X,hrp.Position.Y,AP_R2.Z)
            if (tgt-hrp.Position).Magnitude<1 then hum:Move(Vector3.zero,false);clearSpeedConstraint();autoRightEnabled=false;if arConn then arConn:Disconnect();arConn=nil end;arPhase=1;if autoRightSetVisual then autoRightSetVisual(false) end;if mobBtnRefs.autoRight then mobBtnRefs.autoRight(false) end;return end
            local d=AP_R2-hrp.Position;local mv=Vector3.new(d.X,0,d.Z).Unit;hum:Move(mv,false);setSpeedConstraint(hrp, Vector3.new(mv.X*spd, 0, mv.Z*spd))
        end
        if autoMoveSwingEnabled and not _arSwingDebounce then
            _arSwingDebounce=true
            local bat=findBat()
            if bat then
                if bat.Parent~=char then pcall(function() hum:EquipTool(bat) end) end
                pcall(function() bat:Activate() end)
            end
            task.delay(autoMoveSwingInterval,function() _arSwingDebounce=false end)
        end
    end)
end

local ANTI_FLING_AIMBOT_LIN = 95
local ANTI_FLING_AIMBOT_ANG = 20
local function applyAimbotAntiFling(root)
    if not root or not root.Parent then return end
    pcall(function()
        local v = root.AssemblyLinearVelocity
        if v.Magnitude > ANTI_FLING_AIMBOT_LIN then
            root.AssemblyLinearVelocity = v.Unit * ANTI_FLING_AIMBOT_LIN
        end
        if root.AssemblyAngularVelocity.Magnitude > ANTI_FLING_AIMBOT_ANG then
            root.AssemblyAngularVelocity = Vector3.zero
        end
        local char = LP.Character
        if char then
            for _, p in ipairs(char:GetChildren()) do
                if p:IsA("BasePart") and p ~= root then
                    local pv = p.AssemblyLinearVelocity
                    if pv.Magnitude > ANTI_FLING_AIMBOT_LIN * 1.15 then
                        p.AssemblyLinearVelocity = pv.Unit * ANTI_FLING_AIMBOT_LIN
                    end
                    if p.AssemblyAngularVelocity.Magnitude > ANTI_FLING_AIMBOT_ANG then
                        p.AssemblyAngularVelocity = Vector3.zero
                    end
                end
            end
        end
    end)
end

local _dropToken = 0
local _dropConn = nil
local _dropSuppressedUntil = 0
local function armDrop()
    if tick() < (_dropSuppressedUntil or 0) then return nil end
    _dropToken = (_dropToken % 1000000) + 1
    return _dropToken
end
local function cancelDropForTPDown()
    _dropSuppressedUntil = tick() + 0.8
    _dropToken = 0
    if _dropConn then
        pcall(function() _dropConn:Disconnect() end)
        _dropConn = nil
    end
    dropActive = false
end
runDrop=function(expectedToken)
    if tick() < (_dropSuppressedUntil or 0) then return end
    if expectedToken == nil or expectedToken ~= _dropToken then
        return
    end
    _dropToken = 0
    if dropActive then return end
    local char=LP.Character;if not char then return end
    local root=char:FindFirstChild("HumanoidRootPart");if not root then return end
    dropActive=true;local t0=tick()
    if _dropConn then pcall(function() _dropConn:Disconnect() end) end
    _dropConn=RunService.Heartbeat:Connect(function()
        local r=char and char:FindFirstChild("HumanoidRootPart")
        if not r then
            if _dropConn then _dropConn:Disconnect();_dropConn=nil end
            dropActive=false;return
        end
        if tick()-t0>=DROP_ASCEND_DURATION then
            if _dropConn then _dropConn:Disconnect();_dropConn=nil end
            local rp=RaycastParams.new();rp.FilterDescendantsInstances={char};rp.FilterType=Enum.RaycastFilterType.Exclude
            local rr=workspace:Raycast(r.Position,Vector3.new(0,-2000,0),rp)
            if rr then local hum2=char:FindFirstChildOfClass("Humanoid");local off=(hum2 and hum2.HipHeight or 2);r.CFrame=CFrame.new(r.Position.X,rr.Position.Y+off,r.Position.Z);applyVel(r, Vector3.zero) end
            dropActive=false;return
        end
        applyVel(r, Vector3.new(r.AssemblyLinearVelocity.X,DROP_ASCEND_SPEED,r.AssemblyLinearVelocity.Z))
    end)
end
local autoTPPaused = false
local function doAutoTPDown(force)
    local char = LP.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hum then return end
    if not force then
        if hum.FloorMaterial ~= Enum.Material.Air then return end
        if hrp.Position.Y < autoTPHeight then return end
    end
    hrp.CFrame = CFrame.new(hrp.Position.X, -7, hrp.Position.Z) * CFrame.Angles(0, select(2, hrp.CFrame:ToEulerAnglesYXZ()), 0)
    hrp.AssemblyLinearVelocity = Vector3.zero
end
local function pauseAutoTP()
    autoTPPaused = true
end
local function resumeAutoTP()
    autoTPPaused = false
end
startAutoTP=function()
    if autoTPConn then task.cancel(autoTPConn);autoTPConn=nil end
    autoTPConn=task.spawn(function()
        while autoTPEnabled do
            task.wait(0.1)
            if not autoTPPaused then
                pcall(function() doAutoTPDown(false) end)
            end
        end
    end)
end
stopAutoTP=function() autoTPEnabled=false;autoTPPaused=false;if autoTPConn then task.cancel(autoTPConn);autoTPConn=nil end end
runTPFloor=function()
    cancelDropForTPDown()
    pcall(function() doAutoTPDown(true) end)
end

local function mirrorTPAimbotActive()
    return autoBatEnabled == true or tpBatEnabled == true or batAimbotEnabled == true
end

local function mirrorTPTeleportDown()
    local char = LP.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    local humanoid = char and char:FindFirstChildOfClass("Humanoid")
    if not root or not humanoid or humanoid.Health <= 0 then return end
    local now = tick()
    if now - (mirrorTPLastTeleport or 0) < 0.08 then return end
    mirrorTPLastTeleport = now
    local _, yaw = root.CFrame:ToEulerAnglesYXZ()
    local y = (MIRROR_TP_DOWN_Y or -7) + (math.random() * 0.6 - 0.3)
    root.CFrame = CFrame.new(root.Position.X, y, root.Position.Z) * CFrame.Angles(0, yaw, 0)
    pcall(function()
        root.AssemblyLinearVelocity = Vector3.new((math.random()-0.5)*0.4, 0, (math.random()-0.5)*0.4)
    end)
    pcall(function() applyVel(root, Vector3.new((math.random()-0.5)*0.4, 0, (math.random()-0.5)*0.4)) end)
end

do
    local flagName = "_K7MirrorTPStarted"
    if not _G[flagName] then
        _G[flagName] = true
        RunService.Heartbeat:Connect(function()
            if not mirrorTPDownEnabled or not mirrorTPAimbotActive() then
                if next(mirrorTPPreviousY) then
                    table.clear(mirrorTPPreviousY)
                end
                return
            end
            for _, plr in ipairs(Players:GetPlayers()) do
                if plr ~= LP and plr.Character then
                    local root = plr.Character:FindFirstChild("HumanoidRootPart")
                    if root then
                        local currentY = root.Position.Y
                        local previousY = mirrorTPPreviousY[plr.UserId]
                        if previousY and previousY - currentY >= (MIRROR_TP_DROP_THRESHOLD or 3) then
                            pcall(mirrorTPTeleportDown)
                            table.clear(mirrorTPPreviousY)
                            return
                        end
                        mirrorTPPreviousY[plr.UserId] = currentY
                    end
                end
            end
        end)
    end
end

local function setMirrorTPDown(enabled)
    mirrorTPDownEnabled = enabled == true
    if not mirrorTPDownEnabled then
        table.clear(mirrorTPPreviousY)
    end
    if setMirrorTPVisual then setMirrorTPVisual(mirrorTPDownEnabled) end
end
defLightBrightness,defLightClock,defLightAmbient=nil,nil,nil
local function applyAntiLagDerender(obj)
    pcall(function()
        if obj:IsA("Accessory") or obj:IsA("Hat") then obj:Destroy()
        elseif obj:IsA("BasePart") then obj.Material=Enum.Material.Plastic;obj.Reflectance=0;obj.CastShadow=false
        elseif obj:IsA("Decal") or obj:IsA("Texture") then obj.Transparency=1
        elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam") or obj:IsA("Fire") or obj:IsA("Smoke") or obj:IsA("Sparkles") then obj.Enabled=false end
    end)
end
enableAntiLag=function()
    removeAccessoriesEnabled=true;antiLagEnabled=true
    defLightBrightness=defLightBrightness or Lighting.Brightness;defLightClock=defLightClock or Lighting.ClockTime;defLightAmbient=defLightAmbient or Lighting.OutdoorAmbient
    Lighting.GlobalShadows=false;Lighting.FogEnd=1e10;Lighting.Brightness=1;Lighting.EnvironmentDiffuseScale=0;Lighting.EnvironmentSpecularScale=0
    for _,e in pairs(Lighting:GetChildren()) do pcall(function() if e:IsA("BlurEffect") or e:IsA("SunRaysEffect") or e:IsA("ColorCorrectionEffect") or e:IsA("BloomEffect") or e:IsA("DepthOfFieldEffect") then e.Enabled=false end end) end
    for _,obj in ipairs(workspace:GetDescendants()) do applyAntiLagDerender(obj) end
    if antiLagDescConn then antiLagDescConn:Disconnect() end
    antiLagDescConn=workspace.DescendantAdded:Connect(function(obj) if removeAccessoriesEnabled then applyAntiLagDerender(obj) end end)
end
disableAntiLag=function()
    removeAccessoriesEnabled=false;antiLagEnabled=false;if antiLagDescConn then antiLagDescConn:Disconnect();antiLagDescConn=nil end
    pcall(function() if defLightBrightness then Lighting.Brightness=defLightBrightness end;if defLightClock then Lighting.ClockTime=defLightClock end;if defLightAmbient then Lighting.OutdoorAmbient=defLightAmbient end;Lighting.ExposureCompensation=0 end)
end


function AdaptK7Extras.enableStretchRez()
    AdaptK7Extras.stretchedResEnabled=true
    if AdaptK7Extras.stretchRezConn then AdaptK7Extras.stretchRezConn:Disconnect() end
    AdaptK7Extras.stretchRezConn=RunService.RenderStepped:Connect(function()
        if not AdaptK7Extras.stretchedResEnabled then return end
        local cam=workspace.CurrentCamera;if not cam then return end
        local val=math.clamp(tonumber(AdaptK7Extras.stretchValue) or 0.7,0.3,0.9)
        pcall(function() cam.CFrame=cam.CFrame*CFrame.new(0,0,0,1,0,0,0,val,0,0,0,1) end)
    end)
end
function AdaptK7Extras.disableStretchRez()
    AdaptK7Extras.stretchedResEnabled=false
    if AdaptK7Extras.stretchRezConn then AdaptK7Extras.stretchRezConn:Disconnect();AdaptK7Extras.stretchRezConn=nil end
    pcall(function() local cam=workspace.CurrentCamera;if cam then cam.FieldOfView=fovValue end end)
end

function AdaptK7Extras.isHatOrHairAccessory(accessory)
    if not accessory or not accessory:IsA("Accessory") then return false end
    local at=nil;pcall(function() at=accessory.AccessoryType end)
    if at==Enum.AccessoryType.Hat or at==Enum.AccessoryType.Hair then return true end
    for _,item in ipairs(accessory:GetDescendants()) do
        if item:IsA("Attachment") and (item.Name=="HatAttachment" or item.Name=="HairAttachment") then return true end
    end
    local n=tostring(accessory.Name):lower()
    return n:find("hair",1,true)~=nil or n:find("hat",1,true)~=nil
end
function AdaptK7Extras.removeCharAccessories(char)
    if not char then return end
    for _,child in ipairs(char:GetChildren()) do
        if child:IsA("Accessory") and AdaptK7Extras.isHatOrHairAccessory(child) then pcall(function() child:Destroy() end) end
    end
end
function AdaptK7Extras.startRemoveAccessories()
    AdaptK7Extras.removeAccessories=true
    local function bind(char)
        if AdaptK7Extras.removeAccChildConn then AdaptK7Extras.removeAccChildConn:Disconnect();AdaptK7Extras.removeAccChildConn=nil end
        AdaptK7Extras.removeCharAccessories(char)
        if char then
            AdaptK7Extras.removeAccChildConn=char.ChildAdded:Connect(function(child)
                if not AdaptK7Extras.removeAccessories or not child:IsA("Accessory") then return end
                task.defer(function() if child.Parent==char then AdaptK7Extras.removeCharAccessories(char) end end)
            end)
        end
    end
    bind(LP.Character)
    if AdaptK7Extras.removeAccConn then AdaptK7Extras.removeAccConn:Disconnect() end
    AdaptK7Extras.removeAccConn=LP.CharacterAdded:Connect(function(char) task.wait(0.15);if AdaptK7Extras.removeAccessories then bind(char) end end)
end
function AdaptK7Extras.stopRemoveAccessories()
    AdaptK7Extras.removeAccessories=false
    if AdaptK7Extras.removeAccConn then AdaptK7Extras.removeAccConn:Disconnect();AdaptK7Extras.removeAccConn=nil end
    if AdaptK7Extras.removeAccChildConn then AdaptK7Extras.removeAccChildConn:Disconnect();AdaptK7Extras.removeAccChildConn=nil end
end

function AdaptK7Extras.hideHardHitRing() if AdaptK7Extras.hardHitRing then pcall(function() AdaptK7Extras.hardHitRing:Destroy() end);AdaptK7Extras.hardHitRing=nil end end
function AdaptK7Extras.showHardHitRing()
    local char=LP.Character;if not char then return end;local hrp=char:FindFirstChild("HumanoidRootPart");if not hrp then return end
    if AdaptK7Extras.hardHitRing and AdaptK7Extras.hardHitRing.Parent then return end
    local cyl=Instance.new("CylinderHandleAdornment");cyl.Name="SpiritHubHardHitRing";cyl.Adornee=hrp;cyl.Color3=Color3.fromRGB(0,0,0);cyl.AlwaysOnTop=true;cyl.ZIndex=5;cyl.Transparency=0.1
    local r=tonumber(AdaptK7Extras.hardHitRadius) or 10;cyl.Radius=r;cyl.InnerRadius=math.max(0.1,r-0.35);cyl.Height=0.15;cyl.CFrame=CFrame.new(0,-3,0);cyl.Parent=hrp;AdaptK7Extras.hardHitRing=cyl
end
function AdaptK7Extras.startHardHit()
    AdaptK7Extras.hardHitEnabled=true;if AdaptK7Extras.hardHitConn then return end
    AdaptK7Extras.hardHitConn=RunService.Heartbeat:Connect(function()
        if not AdaptK7Extras.hardHitEnabled then return end;local char=LP.Character;if not char then return end;local root=char:FindFirstChild("HumanoidRootPart");if not root then return end
        if not AdaptK7Extras.hardHitRing or not AdaptK7Extras.hardHitRing.Parent then AdaptK7Extras.showHardHitRing() end
        if AdaptK7Extras.hardHitRing then local r=tonumber(AdaptK7Extras.hardHitRadius) or 10;AdaptK7Extras.hardHitRing.Radius=r;AdaptK7Extras.hardHitRing.InnerRadius=math.max(0.1,r-0.35);if AdaptK7Extras.hardHitRing.Adornee~=root then AdaptK7Extras.hardHitRing.Adornee=root;AdaptK7Extras.hardHitRing.Parent=root end end
    end);AdaptK7Extras.showHardHitRing()
end
function AdaptK7Extras.stopHardHit() AdaptK7Extras.hardHitEnabled=false;if AdaptK7Extras.hardHitConn then pcall(function() AdaptK7Extras.hardHitConn:Disconnect() end);AdaptK7Extras.hardHitConn=nil end;AdaptK7Extras.hideHardHitRing() end

function AdaptK7Extras.clearAntiDieConnections()
    if AdaptK7Extras.antiDieHealthConn then
        pcall(function() AdaptK7Extras.antiDieHealthConn:Disconnect() end)
        AdaptK7Extras.antiDieHealthConn=nil
    end
    if AdaptK7Extras.antiDieDiedConn then
        pcall(function() AdaptK7Extras.antiDieDiedConn:Disconnect() end)
        AdaptK7Extras.antiDieDiedConn=nil
    end
    if AdaptK7Extras.antiDieDeathConns then
        for _,c in ipairs(AdaptK7Extras.antiDieDeathConns) do
            pcall(function() c:Disconnect() end)
        end
    end
    AdaptK7Extras.antiDieDeathConns={}
    if AdaptK7Extras.antiDieHeartConn then
        pcall(function() AdaptK7Extras.antiDieHeartConn:Disconnect() end)
        AdaptK7Extras.antiDieHeartConn=nil
    end
end

function AdaptK7Extras.protectAntiDieCharacter(character)
    if not AdaptK7Extras.antiDieEnabled or not character then return end
    local hum=character:FindFirstChildOfClass("Humanoid") or character:WaitForChild("Humanoid",5)
    if not hum or not AdaptK7Extras.antiDieEnabled then return end

    AdaptK7Extras.antiDieHumanoid=hum
    pcall(function()
        hum.MaxHealth=math.huge
        hum.Health=math.huge
    end)

    local sc=hum.StateChanged:Connect(function(_,newState)
        if not AdaptK7Extras.antiDieEnabled then return end
        if newState==Enum.HumanoidStateType.Dead then
            pcall(function()
                hum.Health=math.huge
                hum:SetStateEnabled(Enum.HumanoidStateType.Dead,false)
            end)
        end
    end)
    table.insert(AdaptK7Extras.antiDieDeathConns,sc)

    pcall(function() hum:SetStateEnabled(Enum.HumanoidStateType.Dead,false) end)

    local hc=hum:GetPropertyChangedSignal("Health"):Connect(function()
        if not AdaptK7Extras.antiDieEnabled then return end
        if hum.Parent and hum.Health<hum.MaxHealth then
            pcall(function() hum.Health=math.huge end)
        end
    end)
    table.insert(AdaptK7Extras.antiDieDeathConns,hc)

    if AdaptK7Extras.antiDieHeartConn then
        pcall(function() AdaptK7Extras.antiDieHeartConn:Disconnect() end)
    end
    AdaptK7Extras.antiDieHeartConn=RunService.Heartbeat:Connect(function()
        if not AdaptK7Extras.antiDieEnabled then return end
        if hum and hum.Parent and hum.Health<hum.MaxHealth then
            pcall(function() hum.Health=math.huge end)
        end
    end)
end

function AdaptK7Extras.startAntiDie()
    AdaptK7Extras.antiDieEnabled=true
    AdaptK7Extras.clearAntiDieConnections()
    if AdaptK7Extras.antiDieCharConn then
        pcall(function() AdaptK7Extras.antiDieCharConn:Disconnect() end)
        AdaptK7Extras.antiDieCharConn=nil
    end

    AdaptK7Extras.protectAntiDieCharacter(LP.Character)
    AdaptK7Extras.antiDieCharConn=LP.CharacterAdded:Connect(function(char)
        if not AdaptK7Extras.antiDieEnabled then return end
        task.wait(0.1)
        AdaptK7Extras.clearAntiDieConnections()
        AdaptK7Extras.protectAntiDieCharacter(char)
    end)
end

function AdaptK7Extras.stopAntiDie()
    AdaptK7Extras.antiDieEnabled=false
    AdaptK7Extras.clearAntiDieConnections()
    if AdaptK7Extras.antiDieCharConn then
        pcall(function() AdaptK7Extras.antiDieCharConn:Disconnect() end)
        AdaptK7Extras.antiDieCharConn=nil
    end

    local char=LP.Character
    local hum=char and char:FindFirstChildOfClass("Humanoid")
    AdaptK7Extras.antiDieHumanoid=nil
    if hum then
        pcall(function()
            hum:SetStateEnabled(Enum.HumanoidStateType.Dead,true)
            hum.MaxHealth=100
            hum.Health=100
        end)
    end
end
local function findMedusa()
    local c=LP.Character;if not c then return nil end
    for _,t in ipairs(c:GetChildren()) do if t:IsA("Tool") then local n=t.Name:lower();if n:find("medusa") or n:find("head") or n:find("stone") then return t end end end
    local bp=LP:FindFirstChild("Backpack");if bp then for _,t in ipairs(bp:GetChildren()) do if t:IsA("Tool") then local n=t.Name:lower();if n:find("medusa") or n:find("head") or n:find("stone") then return t end end end end
    return nil
end
local function useMedusaCounter()
    if medusaDebounce then return end;if MEDUSA_COOLDOWN>(tick()-medusaLastUsed) then return end
    local c=LP.Character;if not c then return end;medusaDebounce=true
    local med=findMedusa();if not med then medusaDebounce=false;return end
    if med.Parent~=c then local hum2=c:FindFirstChildOfClass("Humanoid");if hum2 then hum2:EquipTool(med) end end
    pcall(function() med:Activate() end);medusaLastUsed=tick();medusaDebounce=false
end
local function onAnchorChanged(part)
    return part:GetPropertyChangedSignal("Anchored"):Connect(function()
                if part.Anchored and part.Transparency == 1 then
            if medusaCounterEnabled then
                useMedusaCounter()
            end
        end
    end)
end
setupMedusa=function(char)
    for _,c in pairs(Conns.anchor) do pcall(function() c:Disconnect() end) end;Conns.anchor={}
    if not char then return end
    for _,part in ipairs(char:GetDescendants()) do if part:IsA("BasePart") then table.insert(Conns.anchor,onAnchorChanged(part)) end end
    table.insert(Conns.anchor,char.DescendantAdded:Connect(function(part) if part:IsA("BasePart") then table.insert(Conns.anchor,onAnchorChanged(part)) end end))
end
stopMedusaCounter=function() for _,c in pairs(Conns.anchor) do pcall(function() c:Disconnect() end) end;Conns.anchor={} end
BAT_COUNTER_SLAP_LIST={"Bat","Slap","Iron Slap","Gold Slap","Diamond Slap","Emerald Slap","Ruby Slap","Dark Matter Slap","Flame Slap","Nuclear Slap","Galaxy Slap","Glitched Slap"}
local function findBatForCounter()
    local c=LP.Character;if not c then return nil end;local bp=LP:FindFirstChildOfClass("Backpack")
    for _,name in ipairs(BAT_COUNTER_SLAP_LIST) do local t=c:FindFirstChild(name) or (bp and bp:FindFirstChild(name));if t then return t end end
    for _,ch in ipairs(c:GetChildren()) do if ch:IsA("Tool") and ch.Name:lower():find("bat") then return ch end end
    if bp then for _,ch in ipairs(bp:GetChildren()) do if ch:IsA("Tool") and ch.Name:lower():find("bat") then return ch end end end
    return nil
end
local function swingBatForCounter(bat,char)
    local hum2=char:FindFirstChildOfClass("Humanoid")
    if bat.Parent~=char then if hum2 then pcall(function() hum2:EquipTool(bat) end) end;task.wait(0.05) end
    local remote=bat:FindFirstChildOfClass("RemoteEvent") or bat:FindFirstChildOfClass("RemoteFunction")
    if remote and remote:IsA("RemoteEvent") then pcall(function() remote:FireServer() end);task.wait(0.15);pcall(function() remote:FireServer() end)
    else pcall(function() bat:Activate() end);task.wait(0.15);pcall(function() bat:Activate() end) end
end
startBatCounter=function()
    if Conns.batCounter then return end
    Conns.batCounter=RunService.Heartbeat:Connect(function()
        if not batCounterEnabled or batCounterDebounce then return end
        local char=LP.Character;if not char then return end;local hum2=char:FindFirstChildOfClass("Humanoid");if not hum2 then return end
        local st=hum2:GetState()
        if st==Enum.HumanoidStateType.Physics or st==Enum.HumanoidStateType.Ragdoll then
            batCounterDebounce=true;task.spawn(function() local bat=findBatForCounter();if bat then swingBatForCounter(bat,char) end;task.wait(0.5);batCounterDebounce=false end)
        end
    end)
end
stopBatCounter=function() if Conns.batCounter then Conns.batCounter:Disconnect();Conns.batCounter=nil end;batCounterDebounce=false end
aimbotConn=nil
tpBatConn=nil
aimbotV2Conn=nil

local function findBat()
    local char = LP.Character
    if not char then return nil end
    for _, tool in ipairs(char:GetChildren()) do
        if tool:IsA("Tool") and (tool.Name:lower():find("bat") or tool.Name:lower():find("slap")) then
            return tool
        end
    end
    local bp = LP:FindFirstChild("Backpack")
    if bp then
        for _, tool in ipairs(bp:GetChildren()) do
            if tool:IsA("Tool") and (tool.Name:lower():find("bat") or tool.Name:lower():find("slap")) then
                return tool
            end
        end
    end
    return nil
end

local aimbotSpeed = 58
_aimbotTarget = nil
_aimbotTargetPlr = nil

local function getClosestTarget()
    local root = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
    if not root then return nil, nil, math.huge end
    local closest, closestPlr, minDist = nil, nil, math.huge
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LP and plr.Character then
            local tRoot = plr.Character:FindFirstChild("HumanoidRootPart")
            local hum = plr.Character:FindFirstChildOfClass("Humanoid")
            if tRoot and hum and hum.Health > 0 then
                local dist = (tRoot.Position - root.Position).Magnitude
                if dist < minDist then
                    minDist = dist
                    closest = tRoot
                    closestPlr = plr
                end
            end
        end
    end
    return closest, closestPlr, minDist
end

local function getStickyTarget(currentRoot)
    local root = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
    if not root then return nil, nil end
    local newClosest, newPlr, newDist = getClosestTarget()
    if not newClosest then return nil, nil end
    if currentRoot and currentRoot.Parent then
        local currentPlr = Players:GetPlayerFromCharacter(currentRoot.Parent)
        local hum = currentRoot.Parent:FindFirstChildOfClass("Humanoid")
        if currentPlr and hum and hum.Health > 0 then
            local currentDist = (currentRoot.Position - root.Position).Magnitude
            if currentPlr == newPlr or newDist > currentDist * 0.7 then
                return currentRoot, currentPlr
            end
        end
    end
    return newClosest, newPlr
end

local function swingCurrentBat(char)
    if not autoSwingEnabled then return end
    char = char or LP.Character
    if not char then return end
    local bat = findBat()
    if bat and bat.Parent == char and bat:IsA("Tool") then
        pcall(function() bat:Activate() end)
    end
end

do
    task.spawn(function()
        task.wait(1.5)
        local sg = Instance.new("ScreenGui")
        sg.Name = "K7VisionAimbotTarget"
        sg.ResetOnSpawn = false
        sg.DisplayOrder = 40
        sg.IgnoreGuiInset = true
        pcall(function()
            if parentGui then parentGui(sg) else
                local ok = pcall(function() sg.Parent = game:GetService("CoreGui") end)
                if not ok then sg.Parent = LP:WaitForChild("PlayerGui") end
            end
        end)
        local th = (type(getTheme) == "function" and getTheme()) or {}
        local bg = th.BG or Color3.fromRGB(8, 8, 10)
        local accent = th.ACCENT or Color3.fromRGB(190, 120, 255)
        local accent2 = th.ACCENT2 or Color3.fromRGB(218, 155, 255)
        local white = th.WHITE or Color3.fromRGB(235, 235, 240)

        local f = Instance.new("Frame", sg)
        f.Size = UDim2.new(0, 200, 0, 36)
        f.Position = UDim2.new(0.5, -100, 0, 80)
        f.BackgroundColor3 = bg
        f.BackgroundTransparency = 0.12
        f.BorderSizePixel = 0
        f.Visible = false
        Instance.new("UICorner", f).CornerRadius = UDim.new(0, 8)
        local st = Instance.new("UIStroke", f)
        st.Color = accent
        st.Thickness = 1
        st.Transparency = 0.2
        local stGrad = Instance.new("UIGradient", st)
        stGrad.Color = ColorSequence.new(accent, accent2)
        stGrad.Rotation = 0
        local lockTxt = Instance.new("TextLabel", f)
        lockTxt.Size = UDim2.new(0, 60, 1, 0)
        lockTxt.Position = UDim2.new(0, 10, 0, 0)
        lockTxt.BackgroundTransparency = 1
        lockTxt.Text = "LOCKED"
        lockTxt.TextColor3 = accent
        lockTxt.Font = Enum.Font.GothamBlack
        lockTxt.TextSize = 11
        lockTxt.TextXAlignment = Enum.TextXAlignment.Left
        local targetLabel = Instance.new("TextLabel", f)
        targetLabel.Size = UDim2.new(1, -76, 1, 0)
        targetLabel.Position = UDim2.new(0, 70, 0, 0)
        targetLabel.BackgroundTransparency = 1
        targetLabel.Text = ""
        targetLabel.TextColor3 = white
        targetLabel.Font = Enum.Font.GothamBlack
        targetLabel.TextSize = 12
        targetLabel.TextXAlignment = Enum.TextXAlignment.Left
        targetLabel.TextTruncate = Enum.TextTruncate.AtEnd
        local labelGrad = Instance.new("UIGradient", targetLabel)
        labelGrad.Color = ColorSequence.new(accent, accent2)
        labelGrad.Rotation = 0
        local rot = 0
        RunService.Heartbeat:Connect(function(dt)
            rot = (rot + (dt or 0.016) * 35) % 360
            stGrad.Rotation = rot
            local th2 = (type(getTheme) == "function" and getTheme()) or th
            local a = th2.ACCENT or accent
            lockTxt.TextColor3 = a
            st.Color = a
            if autoBatEnabled and _aimbotTargetPlr and _aimbotTargetPlr.Parent then
                local name = _aimbotTargetPlr.DisplayName or _aimbotTargetPlr.Name or "?"
                if targetLabel.Text ~= name then targetLabel.Text = name end
                if not f.Visible then f.Visible = true end
            else
                if f.Visible then f.Visible = false end
            end
        end)
    end)
end


local BAT_TOOL_NAMES = {"Bat","Slap","Iron Slap","Gold Slap","Diamond Slap","Emerald Slap","Ruby Slap","Dark Matter Slap","Flame Slap","Nuclear Slap","Galaxy Slap","Glitched Slap"}
function ensureBatEquipped()
    local char = LP.Character
    if not char then return nil end
    local hum = char:FindFirstChildOfClass("Humanoid")
    local function isBatTool(tool)
        if not tool or not tool:IsA("Tool") then return false end
        local n = tostring(tool.Name):lower()
        if n:find("bat", 1, true) or n:find("slap", 1, true) or n:find("glove", 1, true) then return true end
        for _, name in ipairs(BAT_TOOL_NAMES) do
            if tool.Name == name then return true end
        end
        return false
    end
    for _, ch in ipairs(char:GetChildren()) do
        if isBatTool(ch) then return ch end
    end
    local bp = LP:FindFirstChildOfClass("Backpack") or LP:FindFirstChild("Backpack")
    if bp and hum then
        for _, name in ipairs(BAT_TOOL_NAMES) do
            local t = bp:FindFirstChild(name)
            if t and t:IsA("Tool") then
                pcall(function() hum:EquipTool(t) end)
                return t
            end
        end
        for _, t in ipairs(bp:GetChildren()) do
            if isBatTool(t) then
                pcall(function() hum:EquipTool(t) end)
                return t
            end
        end
    end
    return nil
end

local AceAntiBypassAimbot = {conn = nil, swingCooldown = false, prevAutoRotate = nil}
local AceNormalAimbot = {conn = nil, target = nil, swingCooldown = false}

batAimbotEnabled = false
antiBatBypassLockEnabled = false
antiDesyncAimbotEnabled = false

AIMBOT_SPEED = 58
LAGGER_AIMBOT_SPEED = 40
local AceAntiBypassAimbotSpeed = 58
local AceAntiBypassLaggerAimbotSpeed = 40
ANTI_DESYNC_AIMBOT_SPEED = 58

local AceAntiBypassSlapList = {"Bat","Slap","Iron Slap","Gold Slap","Diamond Slap","Emerald Slap","Ruby Slap","Dark Matter Slap","Flame Slap","Nuclear Slap","Galaxy Slap","Glitched Slap"}

setBatAimbotVisual = setBatAimbotVisual
setAntiBatBypassLockVisual = setAntiBatBypassLockVisual

function GetCurrentSpeedMode()
    if laggerCarryActive or laggerModeEnabled then return "Lagger"
    elseif carrySpeedActive then return "Carry"
    else return "Normal" end
end

function GetBatAimbotSpeed()
    local mode = GetCurrentSpeedMode()
    if mode == "Lagger" then return tonumber(LAGGER_AIMBOT_SPEED) or 40
    else return tonumber(AIMBOT_SPEED) or 58 end
end

function GetAntiBypassAimbotSpeed()
    local mode = GetCurrentSpeedMode()
    if mode == "Lagger" then return tonumber(AceAntiBypassLaggerAimbotSpeed) or 40
    else return tonumber(AceAntiBypassAimbotSpeed) or 58 end
end

function FindAimbotBat()
    return ensureBatEquipped()
end

function GetClosestAimbotTarget()
    local root = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
    if not root then return nil end
    local closest, minDist = nil, math.huge
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LP and plr.Character then
            local tRoot = plr.Character:FindFirstChild("HumanoidRootPart")
            local hum = plr.Character:FindFirstChildOfClass("Humanoid")
            if tRoot and hum and hum.Health > 0 then
                local dist = (tRoot.Position - root.Position).Magnitude
                if dist < minDist then
                    minDist = dist
                    closest = tRoot
                end
            end
        end
    end
    return closest
end

function GetAutoBatTarget()
    local root = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
    if not root then return nil end
    local now = tick()
    local function stillValid(tRoot)
        if not tRoot or not tRoot.Parent then return false end
        local hum = tRoot.Parent:FindFirstChildOfClass("Humanoid")
        if not hum or hum.Health <= 0 then return false end
        if (tRoot.Position - root.Position).Magnitude > (AdaptK7Extras.aimbotStickyMaxDist or 85) then return false end
        return true
    end

    local sticky = AdaptK7Extras.aimbotStickyTarget
    if sticky and now < (AdaptK7Extras.aimbotStickyUntil or 0) and stillValid(sticky) then
        AdaptK7Extras.aimbotTarget = sticky
        return sticky
    end

    if now - (AdaptK7Extras.aimbotLastScan or 0) <= 0.08
        and AdaptK7Extras.aimbotTarget and stillValid(AdaptK7Extras.aimbotTarget) then
        return AdaptK7Extras.aimbotTarget
    end

    AdaptK7Extras.aimbotLastScan = now
    local closest = GetClosestAimbotTarget()
    AdaptK7Extras.aimbotTarget = closest
    AdaptK7Extras.aimbotStickyTarget = closest
    AdaptK7Extras.aimbotStickyUntil = now + (tonumber(AdaptK7Extras.aimbotStickyTime) or 1.25)
    return closest
end

function AntiBypassGetClosest()
    local root = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
    if not root then return nil, math.huge end
    local closest, minDist = nil, math.huge
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LP and plr.Character then
            local tRoot = plr.Character:FindFirstChild("HumanoidRootPart")
            local hum = plr.Character:FindFirstChildOfClass("Humanoid")
            if tRoot and hum and hum.Health > 0 then
                local dist = (tRoot.Position - root.Position).Magnitude
                if dist < minDist then minDist = dist; closest = tRoot end
            end
        end
    end
    return closest, minDist
end

function AntiBypassFindBat()
    return ensureBatEquipped()
end

function AntiBypassTrySwing()
    if AceAntiBypassAimbot.swingCooldown then return end
    AceAntiBypassAimbot.swingCooldown = true
    pcall(function()
        local char = LP.Character
        if not char then return end
        local bat = AntiBypassFindBat()
        if bat then
            if bat.Parent ~= char then
                local hum = char:FindFirstChildOfClass("Humanoid")
                if hum then pcall(function() hum:EquipTool(bat) end) end
            end
            pcall(function() bat:Activate() end)
        end
    end)
    task.delay(0.03, function()
        AceAntiBypassAimbot.swingCooldown = false
    end)
end

function StartBatAimbot()
    if safeModeTryStart and not safeModeTryStart() then return end

    if antiBatBypassLockEnabled then StopAntiBatBypassLock() end
    if AceNormalAimbot.conn then
        pcall(function() AceNormalAimbot.conn:Disconnect() end)
        AceNormalAimbot.conn = nil
    end
    pcall(function() RunService:UnbindFromRenderStep("ZeyHubbAimbotRotCam") end)

    if autoLeftEnabled then
        autoLeftEnabled = false
        if autoLeftSetVisual then autoLeftSetVisual(false) end
        if stopAutoLeft then stopAutoLeft() end
    end
    if autoRightEnabled then
        autoRightEnabled = false
        if autoRightSetVisual then autoRightSetVisual(false) end
        if stopAutoRight then stopAutoRight() end
    end

    batAimbotEnabled = true
    autoBatEnabled = true
    autoSwingEnabled = true
    AdaptK7Extras.aimbotTarget = nil
    AdaptK7Extras.aimbotStickyTarget = nil
    AdaptK7Extras.aimbotStickyUntil = 0
    AdaptK7Extras.aimbotLastScan = 0
    _aimbotTarget = nil
    _aimbotTargetPlr = nil

    pcall(function() if pauseAutoTP then pauseAutoTP() end end)
    pcall(ensureBatEquipped)

    AceNormalAimbot.conn = RunService.Heartbeat:Connect(function()
        if not autoBatEnabled or not batAimbotEnabled then return end

        local char = LP.Character
        if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not root or not hum then return end

        if not char:FindFirstChildOfClass("Tool") then
            local equipBat = FindAimbotBat()
            if equipBat then pcall(function() hum:EquipTool(equipBat) end) end
        end

        local target = GetAutoBatTarget() or GetClosestAimbotTarget()
        if not target then
            AceNormalAimbot.target = nil
            _aimbotTarget = nil
            _aimbotTargetPlr = nil
            hum.AutoRotate = true
            root.AssemblyAngularVelocity = Vector3.zero
            return
        end

        AceNormalAimbot.target = target
        _aimbotTarget = target
        _aimbotTargetPlr = Players:GetPlayerFromCharacter(target.Parent)
        hum.AutoRotate = false

        local targetVel = target.AssemblyLinearVelocity or Vector3.zero
        local myPos = root.Position
        local targetPos = target.Position

        local predictPos = targetPos + targetVel * 0.14 + target.CFrame.LookVector * 0.3
        local direction = predictPos - myPos
        local flatDir = Vector3.new(direction.X, 0, direction.Z)
        if flatDir.Magnitude > 0.01 then
            flatDir = flatDir.Unit
        else
            flatDir = Vector3.new(0, 0, 1)
        end

        local chaseSpeed = GetBatAimbotSpeed()
        local desiredHeight = targetPos.Y + 3.7
        local yVel = (desiredHeight - myPos.Y) * 19.5 + targetVel.Y * 0.8
        if hum.FloorMaterial ~= Enum.Material.Air then yVel = math.max(yVel, 13) end
        yVel = math.clamp(yVel, -70, 110)

        local desiredVel = Vector3.new(flatDir.X * chaseSpeed, yVel, flatDir.Z * chaseSpeed)
        root.AssemblyLinearVelocity = root.AssemblyLinearVelocity:Lerp(desiredVel, 0.8)

        local speed3 = targetVel.Magnitude
        local predictTime = math.clamp(speed3 / 150, 0.05, 0.2)
        local predictedPos = targetPos + targetVel * predictTime
        local toPredict = predictedPos - myPos
        if toPredict.Magnitude > 0.1 then
            local goalCF = CFrame.lookAt(myPos, predictedPos)
            local diffCF = root.CFrame:Inverse() * goalCF
            local rx, ry, rz = diffCF:ToEulerAnglesXYZ()
            rx = math.clamp(rx, -2.5, 2.5)
            ry = math.clamp(ry, -2.5, 2.5)
            rz = math.clamp(rz, -2.5, 2.5)
            root.AssemblyAngularVelocity = root.CFrame:VectorToWorldSpace(Vector3.new(rx * 42, ry * 42, rz * 42))
        end

        if autoSwingEnabled ~= false then
            local bat = char:FindFirstChild("Bat") or FindAimbotBat()
            if bat and bat:IsA("Tool") then
                if bat.Parent ~= char then
                    pcall(function() hum:EquipTool(bat) end)
                else
                    pcall(function() bat:Activate() end)
                end
            end
        end
    end)

    if setBatAimbotVisual then setBatAimbotVisual(true) end
    if autoBatSetVisual then autoBatSetVisual(true) end
    if mobBtnRefs and mobBtnRefs.autoBat then mobBtnRefs.autoBat(true) end
end

function StopBatAimbot()
    batAimbotEnabled = false
    autoBatEnabled = false
    if AceNormalAimbot and AceNormalAimbot.conn then
        pcall(function() AceNormalAimbot.conn:Disconnect() end)
        AceNormalAimbot.conn = nil
    end
    pcall(function() RunService:UnbindFromRenderStep("ZeyHubbAimbotRotCam") end)

    AdaptK7Extras.aimbotStickyTarget = nil
    AdaptK7Extras.aimbotTarget = nil
    AdaptK7Extras.aimbotStickyUntil = 0
    AdaptK7Extras.aimbotLastScan = 0
    _aimbotTarget = nil
    _aimbotTargetPlr = nil
    if AceNormalAimbot then
        AceNormalAimbot.target = nil
        AceNormalAimbot.swingCooldown = false
    end

    local char = LP.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if root then
        pcall(function()
            root.AssemblyLinearVelocity = Vector3.zero
            root.AssemblyAngularVelocity = Vector3.zero
        end)
    end
    if hum then pcall(function() hum.AutoRotate = true end) end

    pcall(function() if not tpBatEnabled and resumeAutoTP then resumeAutoTP() end end)
    if setBatAimbotVisual then setBatAimbotVisual(false) end
    if autoBatSetVisual then autoBatSetVisual(false) end
    if mobBtnRefs and mobBtnRefs.autoBat then mobBtnRefs.autoBat(false) end
end

function StartAntiBatBypassLock()
    if batAimbotEnabled then StopBatAimbot() end
    if antiBatBypassLockEnabled then
        if AceAntiBypassAimbot.conn then AceAntiBypassAimbot.conn:Disconnect(); AceAntiBypassAimbot.conn = nil end
    end
    antiBatBypassLockEnabled = true
    autoBatEnabled = true
    if AceAntiBypassAimbot.conn then AceAntiBypassAimbot.conn:Disconnect(); AceAntiBypassAimbot.conn = nil end
    local hum0 = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
    if hum0 then
        AceAntiBypassAimbot.prevAutoRotate = hum0.AutoRotate
        hum0.AutoRotate = false
    end
    pcall(function() if pauseAutoTP then pauseAutoTP() end end)
    pcall(ensureBatEquipped)

    local bypassTick = 0
    AceAntiBypassAimbot.conn = RunService.Heartbeat:Connect(function()
        if not antiBatBypassLockEnabled then return end
        bypassTick = bypassTick + 1
        if bypassTick % 8 == 0 then return end

        local char = LP.Character
        if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart")
        if not root then return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not hum then return end
        local bat = ensureBatEquipped() or AntiBypassFindBat()
        if bat and bat.Parent ~= char then pcall(function() hum:EquipTool(bat) end) end
        local target, targetDist = AntiBypassGetClosest()
        if not target then return end
        local myPos = root.Position
        local targetPos = target.Position
        local direction = targetPos - myPos
        local flatDir = Vector3.new(direction.X, 0, direction.Z)
        if flatDir.Magnitude > 0 then flatDir = flatDir.Unit else flatDir = Vector3.zero end
        local chaseSpeed = GetAntiBypassAimbotSpeed()
        local desiredHeight = targetPos.Y + 3.7
        local yVel = (desiredHeight - myPos.Y) * 19.5
        if hum.FloorMaterial ~= Enum.Material.Air then yVel = math.max(yVel, 13) end
        yVel = math.clamp(yVel, -70, 110)
        local desiredVel = Vector3.new(flatDir.X * chaseSpeed, yVel, flatDir.Z * chaseSpeed)
        root.AssemblyLinearVelocity = root.AssemblyLinearVelocity:Lerp(desiredVel, 0.8)
        local toTarget = targetPos - myPos
        if toTarget.Magnitude > 0.1 then
            local goalCF = CFrame.lookAt(myPos, targetPos)
            local diffCF = root.CFrame:Inverse() * goalCF
            local rx, ry, rz = diffCF:ToEulerAnglesXYZ()
            rx = math.clamp(rx, -2.5, 2.5); ry = math.clamp(ry, -2.5, 2.5); rz = math.clamp(rz, -2.5, 2.5)
            root.AssemblyAngularVelocity = root.CFrame:VectorToWorldSpace(Vector3.new(rx * 42, ry * 42, rz * 42))
        end
        if autoSwingEnabled and targetDist <= 8 then AntiBypassTrySwing() end
    end)
    if setAntiBatBypassLockVisual then setAntiBatBypassLockVisual(true) end
    if autoBatSetVisual then autoBatSetVisual(true) end
    if mobBtnRefs and mobBtnRefs.autoBat then mobBtnRefs.autoBat(true) end
end

function StopAntiBatBypassLock()
    antiBatBypassLockEnabled = false
    autoBatEnabled = false
    if AceAntiBypassAimbot and AceAntiBypassAimbot.conn then
        AceAntiBypassAimbot.conn:Disconnect(); AceAntiBypassAimbot.conn = nil
    end
    if AceAntiBypassAimbot then AceAntiBypassAimbot.swingCooldown = false end
    local c = LP.Character
    local root = c and c:FindFirstChild("HumanoidRootPart")
    if root then root.AssemblyLinearVelocity = Vector3.zero; root.AssemblyAngularVelocity = Vector3.zero end
    local hum = c and c:FindFirstChildOfClass("Humanoid")
    if hum then hum.AutoRotate = (AceAntiBypassAimbot.prevAutoRotate == nil) and true or AceAntiBypassAimbot.prevAutoRotate end
    AceAntiBypassAimbot.prevAutoRotate = nil
    pcall(function() if not tpBatEnabled and resumeAutoTP then resumeAutoTP() end end)
    if setAntiBatBypassLockVisual then setAntiBatBypassLockVisual(false) end
    if autoBatSetVisual then autoBatSetVisual(false) end
    if mobBtnRefs and mobBtnRefs.autoBat then mobBtnRefs.autoBat(false) end
end

startBatAimbot = StartBatAimbot
stopBatAimbot = function()
    StopBatAimbot()
    StopAntiBatBypassLock()
end
queueAutoBatStart = function()
    if antiKickEnabled and brainrotDetected then return end
    if autoLeftEnabled then autoLeftEnabled=false; if autoLeftSetVisual then autoLeftSetVisual(false) end; stopAutoLeft() end
    if autoRightEnabled then autoRightEnabled=false; if autoRightSetVisual then autoRightSetVisual(false) end; stopAutoRight() end
    if aimbotMode == "bypass" then
        if batAimbotEnabled then StopBatAimbot() end
        StartAntiBatBypassLock()
    else
        if antiBatBypassLockEnabled then StopAntiBatBypassLock() end
        StartBatAimbot()
    end
end

resetAutoBatMotion=function()
    local char=LP.Character;local hrp=char and char:FindFirstChild("HumanoidRootPart");local hum=char and char:FindFirstChildOfClass("Humanoid")
    if hrp then applyVel(hrp, hrp.AssemblyLinearVelocity*0.3);applyAngVel(hrp, Vector3.zero) end
    if hum then hum.AutoRotate=true end
end

_predBall = nil
local function cleanupPredBall()
    if _predBall then
        pcall(function() _predBall:Destroy() end)
        _predBall = nil
    end
end
local function ensurePredBall(pos)
    pcall(function()
        if not _predBall or not _predBall.Parent then
            _predBall = Instance.new("Part")
            _predBall.Name = "K7PredBall"
            _predBall.Size = Vector3.new(0.6, 0.6, 0.6)
            _predBall.Shape = Enum.PartType.Ball
            _predBall.Material = Enum.Material.Neon
            _predBall.CanCollide = false
            _predBall.Anchored = true
            _predBall.CastShadow = false
            _predBall.Parent = workspace
        end
        local accent = (getTheme and getTheme() and getTheme().ACCENT) or Color3.fromRGB(190, 120, 255)
        _predBall.Color = accent
        _predBall.CFrame = CFrame.new(pos)
        _predBall.Transparency = 0.15
    end)
end

local _tpGodConns = {}
local function _tpGodDisable()
    for _, c in ipairs(_tpGodConns) do
        pcall(function() c:Disconnect() end)
    end
    for i = #_tpGodConns, 1, -1 do _tpGodConns[i] = nil end
    pcall(function()
        local char = LP.Character
        if char then
            local ff = char:FindFirstChild("K7TPBatFF")
            if ff then ff:Destroy() end
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum then
                hum.BreakJointsOnDeath = true
                hum.RequiresNeck = true
                pcall(function()
                    hum:SetStateEnabled(Enum.HumanoidStateType.Dead, true)
                    hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, true)
                    hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, true)
                    hum:SetStateEnabled(Enum.HumanoidStateType.Physics, true)
                end)
            end
        end
    end)
end
local function _tpGodBindCharacter(char)
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid") or char:WaitForChild("Humanoid", 3)
    if not hum then return end
    local root = char:FindFirstChild("HumanoidRootPart")

    pcall(function()
        local oldFF = char:FindFirstChild("K7TPBatFF")
        if oldFF then oldFF:Destroy() end
        local ff = Instance.new("ForceField")
        ff.Name = "K7TPBatFF"
        ff.Visible = false
        ff.Parent = char
    end)

    pcall(function()
        hum.BreakJointsOnDeath = false
        hum.RequiresNeck = false
        hum:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
        hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
        hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
        hum:SetStateEnabled(Enum.HumanoidStateType.Physics, false)
        hum.Health = math.max(hum.Health, hum.MaxHealth)
        hum.MaxHealth = math.max(hum.MaxHealth, 100)
    end)

    local function forceAlive()
        if not tpBatEnabled then return end
        if not hum or not hum.Parent then return end
        pcall(function()
            hum.MaxHealth = math.max(hum.MaxHealth, 100)
            if hum.Health < hum.MaxHealth then hum.Health = hum.MaxHealth end
            hum.BreakJointsOnDeath = false
            hum.RequiresNeck = false
            hum.PlatformStand = false
            hum.Sit = false
            pcall(function()
                hum:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
                hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
                hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
                hum:SetStateEnabled(Enum.HumanoidStateType.Physics, false)
            end)
            local st = hum:GetState()
            if st == Enum.HumanoidStateType.Dead
                or st == Enum.HumanoidStateType.Physics
                or st == Enum.HumanoidStateType.Ragdoll
                or st == Enum.HumanoidStateType.FallingDown then
                hum:ChangeState(Enum.HumanoidStateType.Running)
            end
            if root and root.Parent then
                local v = root.AssemblyLinearVelocity
                if v.Magnitude > 120 then
                    root.AssemblyLinearVelocity = v.Unit * 120
                end
                if root.AssemblyAngularVelocity.Magnitude > 20 then
                    root.AssemblyAngularVelocity = Vector3.zero
                end
            end
            if char and char.Parent and not char:FindFirstChild("K7TPBatFF") then
                local ff = Instance.new("ForceField")
                ff.Name = "K7TPBatFF"
                ff.Visible = false
                ff.Parent = char
            end
        end)
    end

    table.insert(_tpGodConns, hum.HealthChanged:Connect(forceAlive))
    table.insert(_tpGodConns, hum:GetPropertyChangedSignal("Health"):Connect(forceAlive))
    table.insert(_tpGodConns, hum.Died:Connect(function()
        if not tpBatEnabled then return end
        pcall(function()
            hum.Health = hum.MaxHealth
            hum:ChangeState(Enum.HumanoidStateType.GettingUp)
            hum:ChangeState(Enum.HumanoidStateType.Running)
            hum.PlatformStand = false
            if char and char.Parent and not char:FindFirstChild("K7TPBatFF") then
                local ff = Instance.new("ForceField")
                ff.Name = "K7TPBatFF"
                ff.Visible = false
                ff.Parent = char
            end
        end)
    end))
    table.insert(_tpGodConns, hum.StateChanged:Connect(function(_, new)
        if not tpBatEnabled then return end
        if new == Enum.HumanoidStateType.Dead
            or new == Enum.HumanoidStateType.Physics
            or new == Enum.HumanoidStateType.Ragdoll
            or new == Enum.HumanoidStateType.FallingDown then
            pcall(function()
                hum.Health = hum.MaxHealth
                hum:ChangeState(Enum.HumanoidStateType.Running)
            end)
        end
    end))
    table.insert(_tpGodConns, RunService.RenderStepped:Connect(forceAlive))
    table.insert(_tpGodConns, RunService.Stepped:Connect(forceAlive))
    table.insert(_tpGodConns, RunService.Heartbeat:Connect(forceAlive))
end
local function _tpGodEnable()
    _tpGodDisable()
    _tpGodBindCharacter(LP.Character)
    table.insert(_tpGodConns, LP.CharacterAdded:Connect(function(c)
        if not tpBatEnabled then return end
        task.wait(0.05)
        _tpGodBindCharacter(c)
    end))
end


startTPBat=function()
    if tpBatConn then pcall(function() tpBatConn:Disconnect() end); tpBatConn=nil end
    tpBatEnabled = true
    pauseAutoTP()
    pcall(_tpGodDisable)
    pcall(cleanupPredBall)

    local hittingCooldown = false
    local fixedTarget = nil

    local function getTPBat()
        local char = LP.Character
        if not char then return nil end
        local tool = char:FindFirstChild("Bat")
        if tool then return tool end
        local backpack = LP:FindFirstChild("Backpack")
        if backpack then
            tool = backpack:FindFirstChild("Bat")
            if tool then
                tool.Parent = char
                return tool
            end
        end
        return nil
    end

    local function tryHitTPBat()
        if hittingCooldown then return end
        hittingCooldown = true
        pcall(function()
            local bat = getTPBat()
            if bat then
                bat:Activate()
                local ev = bat:FindFirstChildWhichIsA("RemoteEvent")
                if ev then ev:FireServer() end
            end
        end)
        task.delay(0.08, function() hittingCooldown = false end)
    end

    local function getClosestTPPlayer(root)
        if not root then return nil, math.huge end
        local closest, closestDistance = nil, math.huge
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= LP and plr.Character then
                local targetRoot = plr.Character:FindFirstChild("HumanoidRootPart")
                local targetHum = plr.Character:FindFirstChildOfClass("Humanoid")
                if targetRoot and targetHum and targetHum.Health > 0 then
                    local distance = (root.Position - targetRoot.Position).Magnitude
                    if distance < closestDistance then
                        closestDistance = distance
                        closest = plr
                    end
                end
            end
        end
        return closest, closestDistance
    end

    local function getFixedTPPlayer(root)
        if fixedTarget and fixedTarget.Parent == Players and fixedTarget.Character then
            local targetRoot = fixedTarget.Character:FindFirstChild("HumanoidRootPart")
            local targetHum = fixedTarget.Character:FindFirstChildOfClass("Humanoid")
            if targetRoot and targetHum and targetHum.Health > 0 then
                return fixedTarget
            end
        end
        fixedTarget = select(1, getClosestTPPlayer(root))
        return fixedTarget
    end

    tpBatConn = RunService.Heartbeat:Connect(function()
        if not tpBatEnabled then return end
        local char = LP.Character
        if not char then return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        local root = char:FindFirstChild("HumanoidRootPart")
        if not hum or not root then return end

        local target = getFixedTPPlayer(root)
        if target and target.Character then
            local targetRoot = target.Character:FindFirstChild("HumanoidRootPart")
            if targetRoot then
                if sethiddenproperty then
                    pcall(function() sethiddenproperty(root, "PhysicsRepRootPart", targetRoot) end)
                end

                -- TP FIXED: fica preso ao mesmo alvo e acompanha ele continuamente.
                local fixedOffset = CFrame.new(0, 0.9, 0)
                root.CFrame = targetRoot.CFrame * fixedOffset

                local cam = workspace.CurrentCamera
                if cam then
                    cam.CFrame = CFrame.new(cam.CFrame.Position, targetRoot.Position)
                end
                tryHitTPBat()
            end
        end
    end)

    if mobBtnRefs.tpBat then mobBtnRefs.tpBat(true) end
end

stopTPBat=function()
    tpBatEnabled = false
    if not autoBatEnabled then resumeAutoTP() end
    if tpBatConn then
        pcall(function() tpBatConn:Disconnect() end)
        tpBatConn = nil
    end
    pcall(_tpGodDisable)
    pcall(cleanupPredBall)
    pcall(function()
        local char = LP.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        if root and sethiddenproperty then
            sethiddenproperty(root, "PhysicsRepRootPart", root)
        end
    end)
    pcall(function()
        if mobBtnRefs and mobBtnRefs.tpBat then mobBtnRefs.tpBat(false) end
    end)
end


resetAutoBatMotion=function()
    local char=LP.Character;local hrp=char and char:FindFirstChild("HumanoidRootPart");local hum=char and char:FindFirstChildOfClass("Humanoid")
    if hrp then applyVel(hrp, hrp.AssemblyLinearVelocity*0.3);applyAngVel(hrp, Vector3.zero) end
    if hum then hum.AutoRotate=true end
end
saveConfig=function()
    local ok, err = pcall(function()
        local function ks(e)
            if type(e)~="table" then return {kb=nil,gp=nil} end
            local ok, res = pcall(function()
                if e.kb then return {kb=e.kb.Name,gp=e.gp and e.gp.Name}
                elseif e.gp then return {gp=e.gp.Name}
                else return {kb=nil,gp=nil} end
            end)
            if ok and type(res)=="table" then return res end
            return {kb=nil,gp=nil}
        end
        local btnPos = {}
        pcall(function() btnPos = loadBtnPositions() or {} end)
        local cfg = {
            normalSpeed=NS, carrySpeed=CS,
            dropBrainrotKey=ks(KB.DropBrainrot), autoLeftKey=ks(KB.AutoLeft), autoRightKey=ks(KB.AutoRight),
            autoBatKey=ks(KB.AutoBat), tpBatKey=ks(KB.TPBat), laggerToggleKey=ks(KB.LaggerToggle),
            laggerCarryKey=ks(KB.LaggerCarry), tpFloorKey=ks(KB.TPFloor), 
            guiHideKey=ks(KB.GuiHide), speedToggleKey=ks(KB.SpeedToggle), antiDesyncKey=ks(KB.AntiDesync),
            grabRadius=Steal.StealRadius, stealDuration=Steal.StealDuration, stealMode=Steal.StealMode or "v2", semiRadius=Steal.StealRange, semiHoldMin=Steal.SemiHoldMin, semiHoldMax=Steal.HoldMax, semiEntryDelay=Steal.EntryDelay,
            antiRagdoll=antiRagdollEnabled==true, antiRagdollMode=AdaptK7Extras.antiRagdollMode,
            hardHitEnabled=AdaptK7Extras.hardHitEnabled==true, hardHitRadius=AdaptK7Extras.hardHitRadius,
            antiFling=antiFlingEnabled==true, antiDie=AdaptK7Extras.antiDieEnabled==true,
            playerEsp=playerEspEnabled==true,
            playerBoxes=playerBoxesEnabled==true,
            playerTracers=playerTracersEnabled==true,
            autoStealEnabled=Steal.AutoStealEnabled==true,
            infiniteJump=infJumpEnabled==true, infJumpMode=infJumpMode,
            medusaCounter=medusaCounterEnabled==true, batCounter=batCounterEnabled==true,
            carrySpeedActive=carrySpeedActive==true, laggerModeEnabled=laggerModeEnabled==true,
            laggerCarryActive=laggerCarryActive==true,
            laggerSpeed=LAGGER_SPEED, laggerCarrySpeed=LAGGER_CARRY_SPEED,
            autoBat=autoBatEnabled==true, tpBat=tpBatEnabled==true, aimbotMode=aimbotMode,

        animPack=animPack,
        stealBarPos=stealBarPos,
        animPackEnabled=animPackEnabled==true,
        headlessEnabled=headlessEnabled==true,
        korbloxEnabled=korbloxEnabled==true,
            autoSwing=autoSwingEnabled==true, unwalkEnabled=unwalkEnabled==true,
            antiLag=antiLagEnabled==true, autoTPEnabled=autoTPEnabled==true, autoTPHeight=autoTPHeight, mirrorTPDown=mirrorTPDownEnabled==true,
            guiTransparencyEnabled=guiTransparencyEnabled==true,
            mobileButtonsEnabled=mobileButtonsEnabled==true, mobileButtonsLocked=mobileButtonsLocked==true,
            uiLocked=uiLocked==true,
            mobileButtonsSize=mobileButtonsSize, hubSizeScale=hubSizeScale, circleButtonsEnabled=circleButtonsEnabled==true,
            antiKick=antiKickEnabled==true, safeMode=safeModeEnabled==true, fovValue=fovValue, perButtonDrag=true,
            stretchedResEnabled=AdaptK7Extras.stretchedResEnabled==true, stretchValue=AdaptK7Extras.stretchValue,
            removeAccessories=AdaptK7Extras.removeAccessories==true,
            skyTheme=currentSkyTheme, guiTheme=currentGuiTheme,
            autoMoveSwing=autoMoveSwingEnabled==true, autoMoveSwingInterval=autoMoveSwingInterval,
            ragdollGui=ragdollGuiEnabled==true, introEnabled=introEnabled==true,
            selectedIntroMusic=selectedIntroMusic, btnPositions=btnPos,
        }
        local encoded = HS:JSONEncode(cfg)
        local wrote = false
        pcall(function()
            writefile("k7duels.json", encoded)
            wrote = true
        end)
        pcall(function() writefile("K7DuelsConfig.json", encoded) end)
        pcall(function()
            if getgenv then
                getgenv()._K7DuelsCfg = cfg
                getgenv()._K7IntroEnabled = cfg.introEnabled
            end
        end)
        if not wrote then
        end
    end)
    if not ok then
    end
end
task.spawn(function() while task.wait(5) do saveConfig() end end)
local function resetAllSettings()
    NS=60;CS=30;LAGGER_SPEED=15;LAGGER_CARRY_SPEED=24.5;carrySpeedActive=false;laggerModeEnabled=false;laggerCarryActive=false
    antiRagdollEnabled=false;AdaptK7Extras.antiRagdollMode="Splatter";AdaptK7Extras.hardHitEnabled=false;AdaptK7Extras.hardHitRadius=10;infJumpEnabled=false;infJumpMode="manual"
    antiFlingEnabled=true;AdaptK7Extras.antiDieEnabled=false;AdaptK7Extras.stretchedResEnabled=false;AdaptK7Extras.stretchValue=0.7;AdaptK7Extras.removeAccessories=false
    medusaCounterEnabled=false;batCounterEnabled=false;unwalkEnabled=false
    autoLeftEnabled=false;autoRightEnabled=false;autoBatEnabled=false;autoSwingEnabled=true;autoMoveSwingEnabled=false
    autoTPEnabled=false;autoTPHeight=30;mirrorTPDownEnabled=false;antiLagEnabled=false
    Steal.AutoStealEnabled=false;Steal.StealRadius=60;Steal.StealDuration=1.4;Steal.StealMode="v2";Steal.StealRange=10;Steal.SemiHoldMin=1.3;Steal.HoldMax=2.6;Steal.EntryDelay=0.3
    guiTransparencyEnabled=false;mobileButtonsEnabled=true;mobileButtonsLocked=false;mobileButtonsSize=80;hubSizeScale=1
    circleButtonsEnabled=false;antiKickEnabled=false;uiLocked=false;fovValue=80;fovIndex=1
    KB.DropBrainrot={kb=nil,gp=nil};KB.AutoLeft={kb=nil,gp=nil};KB.AutoRight={kb=nil,gp=nil}
    KB.AutoBat={kb=nil,gp=nil};KB.TPBat={kb=nil,gp=nil};KB.TPFloor={kb=nil,gp=nil}
    KB.GuiHide={kb=nil,gp=nil};KB.SpeedToggle={kb=nil,gp=nil};KB.LaggerToggle={kb=nil,gp=nil}
    KB.LaggerCarry={kb=nil,gp=nil};KB.AntiDesync={kb=nil,gp=nil};KB.LaggerPanel={kb=nil,gp=nil}
    if refreshSpeedModeLabel then refreshSpeedModeLabel() end
    if mobBtnRefs.carrySpeed then mobBtnRefs.carrySpeed(carrySpeedActive) end
    if mobBtnRefs.lagger then mobBtnRefs.lagger(laggerModeEnabled or laggerCarryActive) end
    if mobBtnRefs.autoLeft then mobBtnRefs.autoLeft(false) end
    if mobBtnRefs.autoRight then mobBtnRefs.autoRight(false) end
    if mobBtnRefs.autoBat then mobBtnRefs.autoBat(false) end
    stopBatAimbot();stopAutoSteal();stopAutoLeft();stopAutoRight();stopAntiRagdoll();stopAutoTP();stopHoldInfJump()
    pcall(AdaptK7Extras.stopAntiDie);pcall(AdaptK7Extras.stopHardHit);pcall(AdaptK7Extras.disableStretchRez);pcall(AdaptK7Extras.stopRemoveAccessories);pcall(startAntiFling)
    if antiLagEnabled then disableAntiLag() end;saveConfig()
end
setInstaGrab,setInfJumpVisual,setAntiRagVisual,setMedusaVisual,setUnwalkVisual,setAntiLagVisual,setAutoSwingVisual=nil,nil,nil,nil,nil,nil,nil
setAntiDieVisual,setAntiFlingVisual,setStretchRezVisual,setRemoveAccessoriesVisual=nil,nil,nil,nil
setHardHitVisual=nil;hardHitRangeBox=nil;semiRadiusBox=nil;semiHoldMinBox=nil;semiHoldMaxBox=nil
setTranspVisual,setLockVisual,setMobVisual,setCircleBtnsVisual=nil,nil,nil,nil
normalBox,carryBox,laggerBox,laggerCarryBox,radInput,autoTPHeightBox,durationBox=nil,nil,nil,nil,nil,nil,nil
mainFrame=nil
mainGuiRef=nil
_persistentConns={}
local function trackConn(conn) table.insert(_persistentConns,conn);return conn end
local function clearPersistentConns() for _,c in ipairs(_persistentConns) do pcall(function() c:Disconnect() end) end;_persistentConns={} end

refreshSpeedModeLabel=function()
    if modeValLbl then
        if laggerCarryActive then modeValLbl.Text="Lagger Carry"
        elseif laggerModeEnabled then modeValLbl.Text="Lagger Normal"
        elseif carrySpeedActive then modeValLbl.Text="Carry"
        else modeValLbl.Text="Normal" end
    end
    if laggerModePillRef and laggerModePillRef.pill and laggerModePillRef.dot then
        local pill=laggerModePillRef.pill;local dot=laggerModePillRef.dot;local on=laggerModeEnabled
        local PINK2=Color3.fromRGB(130,60,220);local OFF=Color3.fromRGB(20,14,38);local WHITE=Color3.fromRGB(255,255,255);local GRAY=Color3.fromRGB(180,150,165)
        TweenService:Create(pill,TweenInfo.new(0.16,Enum.EasingStyle.Quad),{BackgroundColor3=on and PINK2 or OFF}):Play()
        TweenService:Create(dot,TweenInfo.new(0.16,Enum.EasingStyle.Back),{Position=on and UDim2.new(1,-17,0.5,-7) or UDim2.new(0,3,0.5,-7),BackgroundColor3=on and Color3.fromRGB(12,12,14) or GRAY}):Play()
    end
    if carryModePillRef and carryModePillRef.pill and carryModePillRef.dot then
        local pill=carryModePillRef.pill;local dot=carryModePillRef.dot;local on=carrySpeedActive
        local PINK2=Color3.fromRGB(130,60,220);local OFF=Color3.fromRGB(20,14,38);local WHITE=Color3.fromRGB(255,255,255);local GRAY=Color3.fromRGB(180,150,165)
        TweenService:Create(pill,TweenInfo.new(0.16,Enum.EasingStyle.Quad),{BackgroundColor3=on and PINK2 or OFF}):Play()
        TweenService:Create(dot,TweenInfo.new(0.16,Enum.EasingStyle.Back),{Position=on and UDim2.new(1,-17,0.5,-7) or UDim2.new(0,3,0.5,-7),BackgroundColor3=on and Color3.fromRGB(12,12,14) or GRAY}):Play()
    end
end
toggleCarryMode=function()
    carrySpeedActive = not carrySpeedActive
    if carrySpeedActive then
        laggerModeEnabled = false
        laggerCarryActive = false
    end
    refreshSpeedModeLabel()
    if mobBtnRefs.carrySpeed then mobBtnRefs.carrySpeed(carrySpeedActive) end
    if mobBtnRefs.lagger then mobBtnRefs.lagger(laggerModeEnabled or laggerCarryActive) end
end
toggleLaggerMode=function()
    carrySpeedActive = false
    if laggerCarryActive then
        laggerCarryActive = false
        laggerModeEnabled = true
    else
        laggerCarryActive = true
        laggerModeEnabled = false
    end
    refreshSpeedModeLabel()
    if mobBtnRefs.carrySpeed then mobBtnRefs.carrySpeed(false) end
    if mobBtnRefs.lagger then mobBtnRefs.lagger(true) end
end
toggleLaggerCarryMode=function()
    laggerCarryActive = not laggerCarryActive
    if laggerCarryActive then
        carrySpeedActive = false
        laggerModeEnabled = false
    end
    refreshSpeedModeLabel()
    if mobBtnRefs.carrySpeed then mobBtnRefs.carrySpeed(carrySpeedActive) end
    if mobBtnRefs.lagger then mobBtnRefs.lagger(laggerModeEnabled) end
    if mobBtnRefs.laggerCarry then mobBtnRefs.laggerCarry(laggerCarryActive) end
end

LP.CharacterAdded:Connect(function(char)
    task.defer(function()
        local hum = char:FindFirstChildOfClass("Humanoid") or char:WaitForChild("Humanoid", 5)
        if not hum then return end
        if tpBatEnabled then return end
        pcall(function()
            hum.BreakJointsOnDeath = true
            hum.RequiresNeck = true
            hum:SetStateEnabled(Enum.HumanoidStateType.Dead, true)
            hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, true)
            hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, true)
            hum:SetStateEnabled(Enum.HumanoidStateType.Physics, true)
            local root = char:FindFirstChild("HumanoidRootPart")
            if root and sethiddenproperty then
                pcall(function() sethiddenproperty(root, "NetworkIsSleeping", false) end)
            end
        end)
    end)
end)

antiRagdollCached = {}

local function cacheCharacterAntiRag()
    local char = LP.Character
    if not char then return false end

    local hum = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")

    if not hum or not root then return false end

    antiRagdollCached = {
        character = char,
        humanoid = hum,
        root = root
    }

    local camera = workspace.CurrentCamera
    if camera then
        camera.CameraSubject = hum
    end

    return true
end

local function isRagdolledAntiRag()
    local hum = antiRagdollCached.humanoid
    if not hum then return false end

    local state = hum:GetState()

    if state == Enum.HumanoidStateType.Physics
    or state == Enum.HumanoidStateType.Ragdoll
    or state == Enum.HumanoidStateType.FallingDown then
        return true
    end

    local endTime = LP:GetAttribute("RagdollEndTime")
    if endTime then
        local now = workspace:GetServerTimeNow()
        if (endTime - now) > 0 then
            return true
        end
    end

    return false
end

local function removeRagdollConstraints()
    for _, v in ipairs(antiRagdollCached.character:GetDescendants()) do
        if v:IsA("BallSocketConstraint") or 
           (v:IsA("Attachment") and v.Name:find("RagdollAttachment")) then
            pcall(function()
                v:Destroy()
            end)
        end
    end
end

local function forceExitRagdoll()
    local hum = antiRagdollCached.humanoid
    local root = antiRagdollCached.root

    if not hum or not root then return end
    if hum.Health <= 0 then return end

    pcall(function()
        LP:SetAttribute("RagdollEndTime", workspace:GetServerTimeNow())
    end)

    hum:ChangeState(Enum.HumanoidStateType.Running)

    root.Anchored = false
    applyVel(root, Vector3.zero)
    applyAngVel(root, Vector3.zero)

    root.CFrame = CFrame.new(root.Position) * CFrame.Angles(0, math.rad(root.Orientation.Y), 0)

    local camera = workspace.CurrentCamera
    if camera and hum.Health > 0 and camera.CameraSubject ~= hum then
        camera.CameraSubject = hum
    end
end

function AdaptK7Extras.forceNoSplatterReset()
    local char=LP.Character;if not char then return end
    local hum=char:FindFirstChildOfClass("Humanoid");local root=char:FindFirstChild("HumanoidRootPart")
    if not hum or not root or hum.Health<=0 then return end
    pcall(function()
        hum:ChangeState(Enum.HumanoidStateType.GettingUp);root.Velocity=Vector3.zero;root.RotVelocity=Vector3.zero;root.AssemblyLinearVelocity=Vector3.zero;root.AssemblyAngularVelocity=Vector3.zero
        for _,obj in ipairs(char:GetDescendants()) do if obj:IsA("Motor6D") then obj.Enabled=true end;if obj:IsA("Constraint") then obj.Enabled=true end end
        if workspace.CurrentCamera then workspace.CurrentCamera.CameraSubject=hum end
        local PM=LP:FindFirstChild("PlayerScripts") and LP.PlayerScripts:FindFirstChild("PlayerModule")
        if PM then local CM=PM:FindFirstChild("ControlModule");if CM then local ok,module=pcall(require,CM);if ok and module and module.Enable then pcall(function() module:Enable() end) end end end
        hum.AutoRotate=true;hum.PlatformStand=false;hum.Sit=false
    end)
end
startAntiRagdoll = function()
    if Conns.antiRag then return end
    AdaptK7Extras.antiRagdollSplatterLatched=false
    Conns.antiRag=RunService.Heartbeat:Connect(function()
        if not antiRagdollEnabled then return end
        local char=LP.Character;if not char then return end
        local hum=char:FindFirstChildOfClass("Humanoid");local root=char:FindFirstChild("HumanoidRootPart")
        if not hum or hum.Health<=0 then return end
        local state=hum:GetState()

        if AdaptK7Extras.antiRagdollMode=="No Splatter" then
            local ragdolled=(state==Enum.HumanoidStateType.Physics or state==Enum.HumanoidStateType.Ragdoll or state==Enum.HumanoidStateType.FallingDown or hum.PlatformStand)
            if ragdolled then
                if _G._SpiritHubStartStealCooldownVisual then pcall(_G._SpiritHubStartStealCooldownVisual) end
                local now=tick();if now-(AdaptK7Extras.antiRagdollNoSplatterCooldown or 0)>0.15 then AdaptK7Extras.antiRagdollNoSplatterCooldown=now;AdaptK7Extras.forceNoSplatterReset() end
                task.defer(function() task.wait(0.05);if _G._SpiritHubEnsureOverheadVisuals then pcall(_G._SpiritHubEnsureOverheadVisuals,LP.Character) end;if _G._SpiritHubRefreshPlayerEsp then pcall(_G._SpiritHubRefreshPlayerEsp) end end)
            end
            return
        end

        if not root then return end
        local actualRagdoll=(state==Enum.HumanoidStateType.Physics or state==Enum.HumanoidStateType.Ragdoll or hum.PlatformStand)
        local endTime=LP:GetAttribute("RagdollEndTime")
        local serverNow=workspace:GetServerTimeNow()

        if not actualRagdoll then
            if AdaptK7Extras.antiRagdollSplatterLatched then
                if not endTime or (endTime-serverNow)<=0 then
                    AdaptK7Extras.antiRagdollSplatterLatched=false
                end
            end
            return
        end

        if AdaptK7Extras.antiRagdollSplatterLatched then return end
        AdaptK7Extras.antiRagdollSplatterLatched=true

        pcall(function() LP:SetAttribute("RagdollEndTime",serverNow) end)
        for _,d in ipairs(char:GetDescendants()) do
            if d:IsA("BallSocketConstraint") or (d:IsA("Attachment") and d.Name:find("RagdollAttachment")) then d:Destroy() end
        end
        for _,obj in ipairs(char:GetDescendants()) do
            if obj:IsA("Motor6D") and obj.Enabled==false then obj.Enabled=true end
        end
        root.Anchored=false
        root.AssemblyLinearVelocity=Vector3.zero
        root.AssemblyAngularVelocity=Vector3.zero
        hum.PlatformStand=false
        hum.Sit=false
        hum.AutoRotate=true
        if hum.Health>0 then hum:ChangeState(Enum.HumanoidStateType.Running) end
        if workspace.CurrentCamera then workspace.CurrentCamera.CameraSubject=hum end

        task.defer(function()
            task.wait(0.04)
            if _G._SpiritHubEnsureOverheadVisuals then pcall(_G._SpiritHubEnsureOverheadVisuals,LP.Character) end
            if _G._SpiritHubStartStealCooldownVisual then pcall(_G._SpiritHubStartStealCooldownVisual,true) end
            if _G._SpiritHubRefreshPlayerEsp then pcall(_G._SpiritHubRefreshPlayerEsp) end

            task.wait(0.18)
            if _G._SpiritHubEnsureOverheadVisuals then pcall(_G._SpiritHubEnsureOverheadVisuals,LP.Character) end
            if _G._SpiritHubStartStealCooldownVisual then pcall(_G._SpiritHubStartStealCooldownVisual,false) end
        end)
    end)
end
stopAntiRagdoll = function()
    if Conns.antiRag then Conns.antiRag:Disconnect();Conns.antiRag=nil end
    AdaptK7Extras.antiRagdollSplatterLatched=false
end


LP.CharacterAdded:Connect(function(char)
    task.wait(0.5)
    local hum = char:FindFirstChildOfClass("Humanoid")
    if hum and workspace.CurrentCamera then
        workspace.CurrentCamera.CameraSubject = hum
    end
    if antiRagdollEnabled then
        stopAntiRagdoll()
        startAntiRagdoll()
    end
    task.defer(function()
        task.wait(0.1)
        if _G._SpiritHubEnsureOverheadVisuals then pcall(_G._SpiritHubEnsureOverheadVisuals,char) end
        if _G._SpiritHubRefreshPlayerEsp then pcall(_G._SpiritHubRefreshPlayerEsp) end
    end)
end)
startUnwalk=function()
    local c=LP.Character;if not c then return end;local hum=c:FindFirstChildOfClass("Humanoid")
    if hum then for _,t in ipairs(hum:GetPlayingAnimationTracks()) do t:Stop() end end
    local anim=c:FindFirstChild("Animate");if anim then unwalkSavedAnimate=anim:Clone();anim:Destroy() end
end
stopUnwalk=function() local c=LP.Character;if c and unwalkSavedAnimate then unwalkSavedAnimate:Clone().Parent=c;unwalkSavedAnimate=nil end end


local ANTI_FLING_MAX_LIN = 160
local ANTI_FLING_MAX_ANG = 45
local _antiFlingConn = nil

local function _antiFlingApply(root, char, maxLin, maxAng)
    if not root or not root.Parent then return end
    pcall(function()
        local v = root.AssemblyLinearVelocity
        local horiz = Vector3.new(v.X, 0, v.Z)
        local hMag = horiz.Magnitude
        if hMag > maxLin then
            local scale = maxLin / hMag
            root.AssemblyLinearVelocity = Vector3.new(v.X * scale, v.Y, v.Z * scale)
        elseif v.Magnitude > maxLin * 1.35 then
            root.AssemblyLinearVelocity = v.Unit * maxLin
        end
        local av = root.AssemblyAngularVelocity
        if av.Magnitude > maxAng then
            root.AssemblyAngularVelocity = Vector3.zero
        end
        if char then
            for _, p in ipairs(char:GetChildren()) do
                if p:IsA("BasePart") and p ~= root then
                    local pv = p.AssemblyLinearVelocity
                    local ph = Vector3.new(pv.X, 0, pv.Z)
                    if ph.Magnitude > maxLin * 1.2 then
                        local scale = (maxLin * 1.05) / ph.Magnitude
                        p.AssemblyLinearVelocity = Vector3.new(pv.X * scale, pv.Y, pv.Z * scale)
                    end
                    if p.AssemblyAngularVelocity.Magnitude > maxAng then
                        p.AssemblyAngularVelocity = Vector3.zero
                    end
                end
            end
        end
    end)
end


local function startAntiFling()
    antiFlingEnabled=true
    if _antiFlingConn then return end
    _antiFlingConn = RunService.Heartbeat:Connect(function()
        if not antiFlingEnabled then return end
        local char = LP.Character
        if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart")
        if not root then return end
        local aimOn = autoBatEnabled or tpBatEnabled
        local maxLin = aimOn and ANTI_FLING_AIMBOT_LIN or ANTI_FLING_MAX_LIN
        local maxAng = aimOn and ANTI_FLING_AIMBOT_ANG or ANTI_FLING_MAX_ANG
        _antiFlingApply(root, char, maxLin, maxAng)
    end)
end
local function stopAntiFling()
    antiFlingEnabled=false
    if _antiFlingConn then _antiFlingConn:Disconnect();_antiFlingConn=nil end
end

task.spawn(function() task.wait(0.35);if antiFlingEnabled then startAntiFling() end end)
LP.CharacterAdded:Connect(function() if antiFlingEnabled then task.defer(startAntiFling) end end)

local _noPlayerColConn = nil
local _noPlayerColHB = nil
local function applyNoPlayerCollision()
    pcall(function()
        local function stripOther(char)
            if not char then return end
            for _, part in ipairs(char:GetDescendants()) do
                if part:IsA("BasePart") then
                    pcall(function() part.CanCollide = false end)
                end
            end
        end
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LP and plr.Character then stripOther(plr.Character) end
        end
        if not _noPlayerColConn then
            _noPlayerColConn = Players.PlayerAdded:Connect(function(plr)
                plr.CharacterAdded:Connect(function(char)
                    task.wait(0.15)
                    stripOther(char)
                end)
                if plr.Character then stripOther(plr.Character) end
            end)
        end
        if not _noPlayerColHB then
            _noPlayerColHB = RunService.Heartbeat:Connect(function()
                for _, plr in ipairs(Players:GetPlayers()) do
                    if plr ~= LP and plr.Character then
                        for _, part in ipairs(plr.Character:GetChildren()) do
                            if part:IsA("BasePart") and part.CanCollide then
                                part.CanCollide = false
                            end
                        end
                    end
                end
            end)
        end
    end)
end
task.spawn(function()
    task.wait(1)
    applyNoPlayerCollision()
end)

local K7_MARKER = "K7DuelsMarker"
playerEspEnabled = playerEspEnabled or false
playerBoxesEnabled = playerBoxesEnabled or false
playerTracersEnabled = playerTracersEnabled or false
local _espFolder = nil
local _espConn = nil
local _espObjects = {}

local function getEspThemeColor()
    local th = (type(getTheme)=="function" and getTheme()) or nil
    return (th and th.ACCENT) or Color3.fromRGB(255, 255, 255)
end

local function getTracerBrightColor()
    local c = getEspThemeColor()
    local r = math.min(1, c.R * 1.35 + 0.2)
    local g = math.min(1, c.G * 1.35 + 0.2)
    local b = math.min(1, c.B * 1.35 + 0.2)
    return Color3.new(r, g, b)
end

local function ensureEspFolder()
    if _espFolder and _espFolder.Parent then return _espFolder end
    pcall(function()
        local old = game:GetService("CoreGui"):FindFirstChild("K7PlayerESP")
        if old then old:Destroy() end
    end)
    local f = Instance.new("Folder")
    f.Name = "K7PlayerESP"
    pcall(function()
        if _gethui then f.Parent = _gethui()
        else f.Parent = game:GetService("CoreGui") end
    end)
    if not f.Parent then
        pcall(function() f.Parent = LP:FindFirstChild("PlayerGui") end)
    end
    _espFolder = f
    return f
end

local _k7Users = {}
local K7_ZOOM_MAX = 127.89134
local K7_ZOOM_MIN = 0.41278
local K7_NAME_DIST = 101.337

local function markK7UserId(uid)
    if type(uid) == "number" and uid ~= LP.UserId then
        _k7Users[uid] = true
    end
end

local K7_PRESENCE_URL = "https://jsonblob.com/api/jsonBlob/019fd958-1bb9-79ea-ac53-d695c6d450e6"
local K7_PRESENCE_TTL = 25
local _presenceLastPush = 0
local _presenceBusy = false

local function _presenceRequest(method, body)
    local resBody = nil
    if _request then
        local ok, res = pcall(function()
            local opts = {
                Url = K7_PRESENCE_URL,
                Method = method,
                Headers = {
                    ["Content-Type"] = "application/json",
                    ["Accept"] = "application/json",
                },
            }
            if body then opts.Body = body end
            return _request(opts)
        end)
        if ok and type(res) == "table" then
            resBody = res.Body or res.body or res.Data or res.data
        elseif ok and type(res) == "string" then
            resBody = res
        end
    end
    if not resBody and method == "GET" then
        pcall(function()
            resBody = game:HttpGet(K7_PRESENCE_URL)
        end)
    end
    return resBody
end

local function _presenceDecode(raw)
    if type(raw) ~= "string" or #raw < 2 then return nil end
    local ok, data = pcall(function() return HS:JSONDecode(raw) end)
    if ok and type(data) == "table" then return data end
    return nil
end

local function presencePush()
    do return end
    if _presenceBusy then return end
    _presenceBusy = true
    pcall(function()
        local jobId = tostring(game.JobId or "unknown")
        local uid = tostring(LP.UserId)
        local now = os.time()

        local data = _presenceDecode(_presenceRequest("GET")) or { v = 1, servers = {} }
        if type(data.servers) ~= "table" then data.servers = {} end
        if type(data.servers[jobId]) ~= "table" then data.servers[jobId] = {} end

        data.servers[jobId][uid] = now

        for jid, users in pairs(data.servers) do
            if type(users) == "table" then
                for u, ts in pairs(users) do
                    if type(ts) ~= "number" or (now - ts) > (K7_PRESENCE_TTL * 3) then
                        users[u] = nil
                    end
                end
                if next(users) == nil then
                    data.servers[jid] = nil
                end
            end
        end

        local encoded = HS:JSONEncode(data)
        _presenceRequest("PUT", encoded)
        _presenceLastPush = tick()
    end)
    _presenceBusy = false
end

local function presencePull()
    do return end
    pcall(function()
        local jobId = tostring(game.JobId or "unknown")
        local now = os.time()
        local data = _presenceDecode(_presenceRequest("GET"))
        if not data or type(data.servers) ~= "table" then return end
        local users = data.servers[jobId]
        if type(users) ~= "table" then return end
        for u, ts in pairs(users) do
            local uid = tonumber(u)
            if uid and uid ~= LP.UserId and type(ts) == "number" and (now - ts) <= K7_PRESENCE_TTL then
                markK7UserId(uid)
            end
        end
    end)
end

task.spawn(function()
    task.wait(1.2)
    while true do
        pcall(presencePush)
        pcall(presencePull)
        task.wait(4)
    end
end)

task.spawn(function()
    while true do
        task.wait(2)
        pcall(presencePull)
        pcall(function()
            if updateK7Tags then updateK7Tags() end
        end)
    end
end)

local function applyK7Fingerprint()
    pcall(function()
        LP.CameraMaxZoomDistance = K7_ZOOM_MAX
        LP.CameraMinZoomDistance = K7_ZOOM_MIN
        LP.NameDisplayDistance = K7_NAME_DIST
    end)
    pcall(function()
        if getgenv then
            getgenv()._K7DuelsFP = "v2"
            getgenv()._K7DuelsRunning = true
        end
    end)
end

local function placeK7Marker(char)
    applyK7Fingerprint()
end

LP.CharacterAdded:Connect(function(char)
    task.wait(0.3)
    placeK7Marker(char)
end)


local function _matchesK7FP(plr)
    if not plr then return false end
    local char = plr.Character
    if char then
        local marker = char:FindFirstChild(K7_MARKER, true)
        if marker then return true end
        local tag = char:FindFirstChild("K7DuelsTag", true)
        if tag then return true end
        local cAttrOk, cAttr = pcall(function() return char:GetAttribute("K7Duels") end)
        if cAttrOk and cAttr == true then return true end
    end
    local hits = 0
    local ok1, zmax = pcall(function() return plr.CameraMaxZoomDistance end)
    if ok1 and typeof(zmax) == "number" and math.abs(zmax - K7_ZOOM_MAX) < 0.002 then
        hits = hits + 1
    end
    local ok2, zmin = pcall(function() return plr.CameraMinZoomDistance end)
    if ok2 and typeof(zmin) == "number" and math.abs(zmin - K7_ZOOM_MIN) < 0.002 then
        hits = hits + 1
    end
    local ok3, nd = pcall(function() return plr.NameDisplayDistance end)
    if ok3 and typeof(nd) == "number" and math.abs(nd - K7_NAME_DIST) < 0.05 then
        hits = hits + 1
    end
    if hits >= 2 then return true end
    if hits >= 1 and ok1 and typeof(zmax) == "number" and math.abs(zmax - K7_ZOOM_MAX) < 0.0005 then
        return true
    end
    local attrOk, attr = pcall(function() return plr:GetAttribute("K7Duels") end)
    if attrOk and attr == true then return true end
    local fpOk, fp = pcall(function() return plr:GetAttribute("K7FP") end)
    if fpOk and fp == "v2" then return true end
    return false
end

local function isK7User(plr)
    if not plr or plr == LP then return false end
    if _k7Users[plr.UserId] then return true end
    if _matchesK7FP(plr) then
        markK7UserId(plr.UserId)
        return true
    end
    return false
end


local _k7TagFolder = nil
local function ensureK7TagFolder()
    if _k7TagFolder and _k7TagFolder.Parent then return _k7TagFolder end
    local f = Instance.new("Folder")
    f.Name = "K7UserTags"
    local parented = false
    pcall(function()
        if parentGui then parentGui(f); parented = f.Parent ~= nil end
    end)
    if not parented then
        pcall(function() f.Parent = game:GetService("CoreGui") end)
    end
    if not f.Parent then
        pcall(function() f.Parent = LP:WaitForChild("PlayerGui") end)
    end
    _k7TagFolder = f
    return f
end
local _k7Billboards = {}
local function updateK7Tags()
    local folder = ensureK7TagFolder()
    local seen = {}
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LP and isK7User(plr) then
            seen[plr] = true
            local char = plr.Character
            local head = char and (char:FindFirstChild("Head") or char:FindFirstChild("HumanoidRootPart"))
            if head then
                local o = _k7Billboards[plr]
                if not o or not o.Parent then
                    pcall(function() if o then o:Destroy() end end)
                    local bb = Instance.new("BillboardGui")
                    bb.Name = "UsingK7_" .. tostring(plr.UserId)
                    bb.Size = UDim2.new(0, 220, 0, 36)
                    bb.StudsOffset = Vector3.new(0, 3.2, 0)
                    bb.AlwaysOnTop = true
                    bb.MaxDistance = 600
                    bb.Adornee = head
                    bb.Parent = folder
                    local lbl = Instance.new("TextLabel", bb)
                    lbl.Size = UDim2.new(1, 0, 1, 0)
                    lbl.BackgroundTransparency = 1
                    lbl.Text = "USING SPIRIT HUB"
                    lbl.TextColor3 = Color3.fromRGB(255, 255, 255)
                    lbl.Font = Enum.Font.GothamBlack
                    lbl.TextSize = 18
                    lbl.TextStrokeTransparency = 0.15
                    lbl.TextStrokeColor3 = Color3.new(0, 0, 0)
                    _k7Billboards[plr] = bb
                else
                    o.Adornee = head
                    o.Enabled = true
                end
            end
        end
    end
    for plr, bb in pairs(_k7Billboards) do
        if not seen[plr] then
            pcall(function() bb:Destroy() end)
            _k7Billboards[plr] = nil
        end
    end
end
task.spawn(function()
    while true do
        task.wait(0.75)
        pcall(updateK7Tags)
    end
end)

task.spawn(function()
    applyK7Fingerprint()
    local acc = 0
    while true do
        RunService.Heartbeat:Wait()
        pcall(applyK7Fingerprint)
        acc = acc + 1
        if acc >= 30 then
            acc = 0
            for _, plr in ipairs(Players:GetPlayers()) do
                if plr ~= LP then
                    pcall(function()
                        if _matchesK7FP(plr) then
                            markK7UserId(plr.UserId)
                        end
                    end)
                end
            end
        end
    end
end)

local function clearEspFor(plr)
    local o = _espObjects[plr]
    if not o then return end
    if o.drawLine then
        pcall(function() o.drawLine.Visible = false; o.drawLine:Remove() end)
        o.drawLine = nil
    end
    if o.drawBox then
        pcall(function() o.drawBox.Visible = false; o.drawBox:Remove() end)
        o.drawBox = nil
    end
    if type(o.boxLines)=="table" then
        for _, line in ipairs(o.boxLines) do
            pcall(function() line.Visible=false; line:Remove() end)
        end
        o.boxLines=nil
    end
    for k, v in pairs(o) do
        pcall(function()
            if typeof(v)=="Instance" then v:Destroy() end
        end)
    end
    _espObjects[plr] = nil
end

local function clearAllEsp()
    for plr in pairs(_espObjects) do clearEspFor(plr) end
    _espObjects = {}
end

local function updatePlayerEsp(plr)
    if not plr or plr == LP then return end
    local char = plr.Character
    if not char then clearEspFor(plr); return end
    local hrp = char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("UpperTorso")
    local head = char:FindFirstChild("Head")
    if not hrp then clearEspFor(plr); return end

    local color = getEspThemeColor()
    local folder = ensureEspFolder()
    local o = _espObjects[plr]
    if not o then o = {}; _espObjects[plr] = o end

    if playerEspEnabled then
        local vel = hrp.AssemblyLinearVelocity or Vector3.zero
        local speed = Vector3.new(vel.X, 0, vel.Z).Magnitude
        if not o.billboard or not o.billboard.Parent then
            pcall(function() if o.billboard then o.billboard:Destroy() end end)
            local bb = Instance.new("BillboardGui")
            bb.Name = "K7Speed"
            bb.Size = UDim2.new(0, 180, 0, 30)
            bb.StudsOffset = Vector3.new(0, 3.25, 0)
            bb.AlwaysOnTop = true
            bb.MaxDistance = 500
            bb.Adornee = head or hrp
            bb.Parent = folder

            local lbl = Instance.new("TextLabel", bb)
            lbl.Name = "Lbl"
            lbl.Size = UDim2.new(1, 0, 1, 0)
            lbl.BackgroundTransparency = 1
            lbl.TextColor3 = color
            lbl.Font = Enum.Font.GothamBold
            lbl.TextSize = 14
            lbl.TextStrokeTransparency = 0.15
            lbl.TextStrokeColor3 = Color3.new(0, 0, 0)
            o.billboard = bb
        end
        o.billboard.Adornee = head or hrp
        local lbl = o.billboard:FindFirstChild("Lbl")
        if lbl then
            lbl.Text = "Speed: " .. tostring(math.floor(speed + 0.5))
            lbl.TextColor3 = color
        end

        local cam = workspace.CurrentCamera
        if cam then
            local vp, onScreen = cam:WorldToViewportPoint(hrp.Position)
            local view = cam.ViewportSize
            local inside = onScreen and vp.Z > 0 and vp.X >= 0 and vp.X <= view.X and vp.Y >= 0 and vp.Y <= view.Y

            if o.billboard then
                o.billboard.Enabled = inside
            end

            if o.offscreenGui then
                pcall(function() o.offscreenGui:Destroy() end)
                o.offscreenGui = nil
                o.offscreenLabel = nil
            end
        end
    else
        if o.billboard then pcall(function() o.billboard:Destroy() end); o.billboard = nil end
        if o.offscreenGui then pcall(function() o.offscreenGui:Destroy() end); o.offscreenGui = nil; o.offscreenLabel = nil end
        if o.highlight then pcall(function() o.highlight:Destroy() end); o.highlight = nil end
    end

    if playerBoxesEnabled then
        local cam = workspace.CurrentCamera
        local useDrawing = false
        pcall(function() if Drawing and Drawing.new then useDrawing = true end end)

        if useDrawing and cam then
            if not o.drawBox then
                local ok, sq = pcall(function() return Drawing.new("Square") end)
                if ok and sq then
                    sq.Filled = false
                    sq.Thickness = 2.5
                    sq.Transparency = 1
                    sq.Visible = false
                    o.drawBox = sq
                end
            end
            if o.drawBox then
                local okBB, cf, size = pcall(function() return char:GetBoundingBox() end)
                if okBB and cf and size then
                    local hx, hy, hz = size.X/2, size.Y/2, size.Z/2
                    local corners = {
                        Vector3.new(-hx,-hy,-hz), Vector3.new(-hx,-hy,hz),
                        Vector3.new(-hx, hy,-hz), Vector3.new(-hx, hy,hz),
                        Vector3.new( hx,-hy,-hz), Vector3.new( hx,-hy,hz),
                        Vector3.new( hx, hy,-hz), Vector3.new( hx, hy,hz),
                    }
                    local minX, minY = math.huge, math.huge
                    local maxX, maxY = -math.huge, -math.huge
                    local anyVisible = false
                    for _, off in ipairs(corners) do
                        local worldPos = cf:PointToWorldSpace(off)
                        local vp = cam:WorldToViewportPoint(worldPos)
                        if vp.Z > 0 then
                            anyVisible = true
                            minX = math.min(minX, vp.X); maxX = math.max(maxX, vp.X)
                            minY = math.min(minY, vp.Y); maxY = math.max(maxY, vp.Y)
                        end
                    end
                    if anyVisible and maxX > minX and maxY > minY then
                        o.drawBox.Position = Vector2.new(minX, minY)
                        o.drawBox.Size = Vector2.new(maxX-minX, maxY-minY)
                        o.drawBox.Color = color
                        o.drawBox.Thickness = 2.5
                        o.drawBox.Visible = true
                    else
                        o.drawBox.Visible = false
                    end
                else
                    o.drawBox.Visible = false
                end
            end
            if o.boxFallback then pcall(function() o.boxFallback:Destroy() end); o.boxFallback=nil end
        else
            if o.drawBox then pcall(function() o.drawBox.Visible=false; o.drawBox:Remove() end); o.drawBox=nil end
            if not o.boxFallback or not o.boxFallback.Parent then
                pcall(function() if o.boxFallback then o.boxFallback:Destroy() end end)
                local hl = Instance.new("Highlight")
                hl.Name = "K7BoxFallback"
                hl.FillTransparency = 1
                hl.OutlineTransparency = 0
                hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                hl.Adornee = char
                hl.Parent = folder
                o.boxFallback = hl
            end
            o.boxFallback.OutlineColor = color
            o.boxFallback.Adornee = char
        end
    else
        if o.drawBox then pcall(function() o.drawBox.Visible=false; o.drawBox:Remove() end); o.drawBox=nil end
        if o.boxFallback then pcall(function() o.boxFallback:Destroy() end); o.boxFallback=nil end
    end

    if playerTracersEnabled then
        local myChar = LP.Character
        local myHrp = myChar and (myChar:FindFirstChild("HumanoidRootPart") or myChar:FindFirstChild("UpperTorso"))
        if myHrp then
            local cam = workspace.CurrentCamera
            local tColor = getTracerBrightColor()
            local useDrawing = false
            pcall(function() if Drawing and Drawing.new then useDrawing = true end end)
            if useDrawing then
                if not o.drawLine then
                    local ok, line = pcall(function() return Drawing.new("Line") end)
                    if ok and line then
                        line.Thickness = 4
                        line.Transparency = 1
                        line.Visible = true
                        o.drawLine = line
                    end
                end
                if o.drawLine then
                    local ok1, p1, v1 = pcall(function()
                        local p, on = cam:WorldToViewportPoint(myHrp.Position)
                        return p, on
                    end)
                    local ok2, p2, v2 = pcall(function()
                        local p, on = cam:WorldToViewportPoint(hrp.Position)
                        return p, on
                    end)
                    if ok1 and ok2 and p1 and p2 and cam then
                        local view = cam.ViewportSize
                        local tx, ty = p2.X, p2.Y
                        if p2.Z <= 0 then
                            tx = view.X - tx
                            ty = view.Y - ty
                        end
                        tx = math.clamp(tx, 8, math.max(8, view.X - 8))
                        ty = math.clamp(ty, 8, math.max(8, view.Y - 8))

                        local fromX = math.clamp(p1.X, 0, view.X)
                        local fromY = math.clamp(p1.Y, 0, view.Y)
                        if p1.Z <= 0 or v1 == false then
                            fromX = view.X * 0.5
                            fromY = view.Y - 18
                        end

                        o.drawLine.From = Vector2.new(fromX, fromY)
                        o.drawLine.To = Vector2.new(tx, ty)
                        o.drawLine.Color = tColor
                        o.drawLine.Thickness = 4
                        o.drawLine.Visible = true
                    else
                        o.drawLine.Visible = false
                    end
                end
                if o.beam then pcall(function() o.beam:Destroy() end); o.beam = nil end
                if o.att0 then pcall(function() o.att0:Destroy() end); o.att0 = nil end
                if o.att1 then pcall(function() o.att1:Destroy() end); o.att1 = nil end
            else
                local wsFolder = workspace:FindFirstChild("K7TracersWS")
                if not wsFolder then
                    wsFolder = Instance.new("Folder")
                    wsFolder.Name = "K7TracersWS"
                    wsFolder.Parent = workspace
                end
                if not o.att0 or not o.att0.Parent or o.att0.Parent ~= myHrp then
                    if o.att0 then pcall(function() o.att0:Destroy() end) end
                    local a0 = Instance.new("Attachment")
                    a0.Name = "K7Trace0"; a0.Parent = myHrp; o.att0 = a0
                end
                if not o.att1 or not o.att1.Parent or o.att1.Parent ~= hrp then
                    if o.att1 then pcall(function() o.att1:Destroy() end) end
                    local a1 = Instance.new("Attachment")
                    a1.Name = "K7Trace1"; a1.Parent = hrp; o.att1 = a1
                end
                if not o.beam or not o.beam.Parent then
                    if o.beam then pcall(function() o.beam:Destroy() end) end
                    local beam = Instance.new("Beam")
                    beam.Name = "K7Tracer"
                    beam.Attachment0 = o.att0
                    beam.Attachment1 = o.att1
                    beam.Color = ColorSequence.new(tColor)
                    beam.Width0 = 0.45
                    beam.Width1 = 0.35
                    beam.FaceCamera = true
                    beam.LightEmission = 1
                    beam.LightInfluence = 0
                    beam.Transparency = NumberSequence.new(0)
                    beam.Parent = wsFolder
                    o.beam = beam
                else
                    o.beam.Attachment0 = o.att0
                    o.beam.Attachment1 = o.att1
                    o.beam.Color = ColorSequence.new(tColor)
                    o.beam.Width0 = 0.45
                    o.beam.Width1 = 0.35
                end
            end
        end
    else
        if o.drawLine then
            pcall(function() o.drawLine.Visible = false; o.drawLine:Remove() end)
            o.drawLine = nil
        end
        if o.beam then pcall(function() o.beam:Destroy() end); o.beam = nil end
        if o.att0 then pcall(function() o.att0:Destroy() end); o.att0 = nil end
        if o.att1 then pcall(function() o.att1:Destroy() end); o.att1 = nil end
    end
end

local function anyPlayerEspEnabled()
    return playerEspEnabled or playerBoxesEnabled or playerTracersEnabled
end

local function refreshAllEsp()
    if not anyPlayerEspEnabled() then
        clearAllEsp()
        return
    end
    ensureEspFolder()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LP then
            pcall(updatePlayerEsp, plr)
        end
    end
    for plr in pairs(_espObjects) do
        if not plr.Parent or plr == LP then clearEspFor(plr) end
    end
end

_G._SpiritHubRefreshPlayerEsp = function()
    pcall(refreshAllEsp)
end

local function ensureEspHeartbeat()
    if _espConn then return end
    _espConn = RunService.Heartbeat:Connect(function()
        if not anyPlayerEspEnabled() then return end
        refreshAllEsp()
    end)
end

local function stopEspHeartbeatIfUnused()
    if not anyPlayerEspEnabled() then
        if _espConn then _espConn:Disconnect(); _espConn = nil end
        clearAllEsp()
    else
        refreshAllEsp()
    end
end

startPlayerEsp = function()
    playerEspEnabled = true
    refreshAllEsp(); ensureEspHeartbeat()
end
stopPlayerEsp = function()
    playerEspEnabled = false
    stopEspHeartbeatIfUnused()
end

startPlayerBoxes = function()
    playerBoxesEnabled = true
    refreshAllEsp(); ensureEspHeartbeat()
end
stopPlayerBoxes = function()
    playerBoxesEnabled = false
    stopEspHeartbeatIfUnused()
end

startPlayerTracers = function()
    playerTracersEnabled = true
    refreshAllEsp(); ensureEspHeartbeat()
end
stopPlayerTracers = function()
    playerTracersEnabled = false
    stopEspHeartbeatIfUnused()
end

task.spawn(function()
    local function onChar(char)
        task.wait(0.25)
        placeK7Marker(char)
    end
    if LP.Character then onChar(LP.Character) end
    LP.CharacterAdded:Connect(onChar)
end)
Players.PlayerRemoving:Connect(function(plr)
    clearEspFor(plr)
    if plr then _k7Users[plr.UserId] = nil end
end)


local function createStealBar()
    pcall(function()
        if destroyOldGui then destroyOldGui("K7InfoBar") end
        if destroyOldGui then destroyOldGui("K7StealBar") end
        if destroyOldGui then destroyOldGui("AdaptStealBar") end
        local cg = game:GetService("CoreGui")
        for _, n in ipairs({"K7InfoBar","K7StealBar","AdaptStealBar"}) do
            local o = cg:FindFirstChild(n)
            if o then o:Destroy() end
        end
        local pg = LP:FindFirstChild("PlayerGui")
        if pg then
            for _, n in ipairs({"K7InfoBar","K7StealBar","AdaptStealBar"}) do
                local o = pg:FindFirstChild(n)
                if o then o:Destroy() end
            end
        end
    end)

    local th = (type(getTheme) == "function" and getTheme()) or {}
    local C_PANEL    = Color3.fromRGB(12, 12, 15)
    local C_TEXT_DIM = Color3.fromRGB(140, 140, 156)
    local C_OFF_BG   = Color3.fromRGB(19, 19, 24)
    local ACCENT     = th.ACCENT or Color3.fromRGB(235, 235, 240)

    local barGui = Instance.new("ScreenGui")
    barGui.Name = "AdaptStealBar"
    barGui.ResetOnSpawn = false
    barGui.IgnoreGuiInset = true
    barGui.DisplayOrder = 12
    barGui.Enabled = not (rawget(_G, "_SpiritHubIntroPending") == true)
    if parentGui then
        parentGui(barGui)
    else
        barGui.Parent = LP:FindFirstChild("PlayerGui") or game:GetService("CoreGui")
    end
    stealBarGuiRef = barGui

    local pbFrame = Instance.new("Frame", barGui)
    pbFrame.Name = "StealProgressBar"
    pbFrame.Size = UDim2.new(0, IS_MOBILE and 240 or 260, 0, 36)

    do
        local pos = stealBarPos
        if type(pos) == "table" and pos.sx ~= nil then
            pbFrame.Position = UDim2.new(pos.sx, pos.ox, pos.sy, pos.oy)
        else
            pbFrame.Position = UDim2.new(0.5, -130, 1, -52)
        end
    end

    pbFrame.BackgroundColor3 = C_PANEL
    pbFrame.BorderSizePixel = 0
    pbFrame.Active = true
    pbFrame.ZIndex = 25

    local corner = Instance.new("UICorner", pbFrame)
    corner.CornerRadius = UDim.new(0, 8)

    local pbStroke = Instance.new("UIStroke", pbFrame)
    pbStroke.Color = ACCENT
    pbStroke.Thickness = 1
    pbStroke.Transparency = 0.4

    do
        local dragging = false
        local dragInput, dragStart, startPos

        pbFrame.InputBegan:Connect(function(inp)
            if uiLocked then return end
            if inp.UserInputType == Enum.UserInputType.MouseButton1
                or inp.UserInputType == Enum.UserInputType.Touch then
                dragging = true
                dragStart = inp.Position
                startPos = pbFrame.Position

                inp.Changed:Connect(function()
                    if inp.UserInputState == Enum.UserInputState.End then
                        dragging = false
                        pcall(function()
                            local pos = pbFrame.Position
                            stealBarPos = {
                                sx = pos.X.Scale, ox = pos.X.Offset,
                                sy = pos.Y.Scale, oy = pos.Y.Offset
                            }
                            if type(saveConfig) == "function" then saveConfig() end
                        end)
                    end
                end)
            end
        end)

        pbFrame.InputChanged:Connect(function(inp)
            if inp.UserInputType == Enum.UserInputType.MouseMovement
                or inp.UserInputType == Enum.UserInputType.Touch then
                dragInput = inp
            end
        end)

        UIS.InputChanged:Connect(function(inp)
            if uiLocked then dragging = false return end
            if inp == dragInput and dragging then
                local dx = inp.Position.X - dragStart.X
                local dy = inp.Position.Y - dragStart.Y
                pbFrame.Position = UDim2.new(
                    startPos.X.Scale, startPos.X.Offset + dx,
                    startPos.Y.Scale, startPos.Y.Offset + dy
                )
            end
        end)
    end

    local progressPct = Instance.new("TextLabel", pbFrame)
    progressPct.Name = "ProgressPercent"
    progressPct.Size = UDim2.new(0, 40, 0, 14)
    progressPct.Position = UDim2.new(0, 10, 0, 4)
    progressPct.BackgroundTransparency = 1
    progressPct.Text = "0%"
    progressPct.TextColor3 = ACCENT
    progressPct.Font = Enum.Font.GothamBold
    progressPct.TextSize = 10
    progressPct.TextXAlignment = Enum.TextXAlignment.Left
    progressPct.ZIndex = 26

    local progressRadLbl = Instance.new("TextLabel", pbFrame)
    progressRadLbl.Name = "Radius"
    progressRadLbl.Size = UDim2.new(0, 90, 0, 14)
    progressRadLbl.Position = UDim2.new(1, -100, 0, 4)
    progressRadLbl.BackgroundTransparency = 1
    progressRadLbl.Text = "Radius: " .. tostring(((Steal.StealMode or "normal") == "semi") and (Steal.StealRange or 10) or (Steal.StealRadius or 60))
    progressRadLbl.TextColor3 = C_TEXT_DIM
    progressRadLbl.Font = Enum.Font.GothamBold
    progressRadLbl.TextSize = 10
    progressRadLbl.TextXAlignment = Enum.TextXAlignment.Right
    progressRadLbl.ZIndex = 26

    local progressBg = Instance.new("Frame", pbFrame)
    progressBg.Size = UDim2.new(1, -20, 0, 6)
    progressBg.Position = UDim2.new(0, 10, 0, 22)
    progressBg.BackgroundColor3 = C_OFF_BG
    progressBg.BorderSizePixel = 0
    progressBg.ZIndex = 26
    local bgCorner = Instance.new("UICorner", progressBg)
    bgCorner.CornerRadius = UDim.new(1, 0)

    progressFill = Instance.new("Frame", progressBg)
    progressFill.Size = UDim2.new(0, 0, 1, 0)
    progressFill.BackgroundColor3 = ACCENT
    progressFill.BorderSizePixel = 0
    progressFill.ZIndex = 27
    local fillCorner = Instance.new("UICorner", progressFill)
    fillCorner.CornerRadius = UDim.new(1, 0)

    function formatStealPct(prog)
        local pct = math.clamp(math.floor((tonumber(prog) or 0) * 100), 0, 100)
        return pct .. "%"
    end

    function setStealStatusText(status, extra)
        if status == "STEALING" or status == "CLOSER" or status == "HOLDING" then
            progressPct.Text = formatStealPct(extra or 0)
        elseif status == "STOLE" then
            progressPct.Text = "100%"
        elseif status == "SEARCHING" or status == "MISSED" then
            progressPct.Text = "0%"
        elseif type(extra) == "number" then
            progressPct.Text = formatStealPct(extra)
        end
    end

    radTB = Instance.new("TextBox", pbFrame)
    radTB.Size = UDim2.new(0, 0, 0, 0)
    radTB.Position = UDim2.new(0, 0, 0, 0)
    radTB.BackgroundTransparency = 1
    radTB.TextTransparency = 1
    radTB.Text = tostring(Steal.StealRadius)
    radTB.ClearTextOnFocus = false
    radTB.ZIndex = -1
    radTB.Active = false
    radTB.FocusLost:Connect(function()
        local n = tonumber(radTB.Text)
        if n and n >= 5 and n <= 300 then
            Steal.StealRadius = math.floor(n)
            Steal.cachedPrompts = {}
            Steal.promptCacheTime = 0
        end
        radTB.Text = tostring(Steal.StealRadius)
        progressRadLbl.Text = "Radius: " .. tostring(((Steal.StealMode or "normal") == "semi") and (Steal.StealRange or 10) or (Steal.StealRadius or 60))
        if stealRadBox and not stealRadBox:IsFocused() then
            stealRadBox.Text = tostring(Steal.StealRadius)
        end
    end)

    task.spawn(function()
        while barGui.Parent do
            task.wait(0.25)
            pcall(function()
                progressRadLbl.Text = "Radius: " .. tostring(((Steal.StealMode or "normal") == "semi") and (Steal.StealRange or 10) or (Steal.StealRadius or 60))
                radTB.Text = tostring(Steal.StealRadius)
                if stealRadBox and not stealRadBox:IsFocused() then
                    stealRadBox.Text = tostring(Steal.StealRadius)
                end
                if not isStealing and progressFill then
                    if not Steal.AutoStealEnabled then
                        progressPct.Text = "0%"
                    end
                end
            end)
        end
    end)

    function resetProgressBar()
        if progressFill then
            progressFill.Size = UDim2.new(0, 0, 1, 0)
        end
        progressPct.Text = "0%"
    end

    stealBarFrame = pbFrame
end

local function destroyMobileButtons()
    pcall(saveBtnPositions)
    if mobGuiRef then pcall(function() mobGuiRef:Destroy() end);mobGuiRef=nil end
    for _,n in ipairs({"K7MobileButtons"}) do
        local old=game:GetService("CoreGui"):FindFirstChild(n);if old then old:Destroy() end
        local pgui=LP:FindFirstChild("PlayerGui");if pgui then local o=pgui:FindFirstChild(n);if o then o:Destroy() end end
    end
    mobBtnRefs={}
end
local function buildMobileButtons()
    destroyMobileButtons();if not mobileButtonsEnabled then return end
    perButtonDragEnabled=true
    local savedPositions=loadBtnPositions()

    local BTN_SIZE=math.floor(mobileButtonsSize*0.55)
    local BTN_GAP=12
    local CORNER_R=circleButtonsEnabled and BTN_SIZE or math.max(8,math.floor(BTN_SIZE*0.22))
    local fontSize=math.max(7,math.floor(mobileButtonsSize*0.10))

    local buttons={
        {key="drop",        label="DROP\nBR",       toggle=false, exclusive=false},
        {key="autoLeft",    label="AUTO\nLEFT",     toggle=true,  exclusive=true},
        {key="autoBat",     label="BAT\nAIMBOT",    toggle=true,  exclusive=true},
        {key="autoRight",   label="AUTO\nRIGHT",    toggle=true,  exclusive=true},
        {key="tpDown",      label="TP\nDOWN",       toggle=false, exclusive=false},
        {key="carrySpeed",  label="CARRY\nSPD",     toggle=true,  exclusive=false},
        {key="lagger",      label="LAGGER\nMODE",   toggle=true,  exclusive=false},
        {key="tpBat",       label="TP\nBAT",        toggle=true,  exclusive=false},
    }

    local COLS=2
    local totalW=(COLS*(BTN_SIZE+BTN_GAP))-BTN_GAP
    local totalH=(math.ceil(#buttons/COLS)*(BTN_SIZE+BTN_GAP))-BTN_GAP

    local PAD=10

    local vp=workspace.CurrentCamera and workspace.CurrentCamera.ViewportSize or Vector2.new(800,600)
    local savedPanel=savedPositions["_panel"]
    local defPanelX=vp.X-totalW-PAD*2-18
    local defPanelY=math.max(10,vp.Y/2-(totalH+PAD*2)/2)
    local panelXO=savedPanel and savedPanel.xo or defPanelX
    local panelYO=savedPanel and savedPanel.yo or defPanelY

    local mobGui=Instance.new("ScreenGui")
    mobGui.Name="K7MobileButtons";mobGui.ResetOnSpawn=false;mobGui.DisplayOrder=15;mobGui.IgnoreGuiInset=true
    mobGui.Enabled = not (rawget(_G, "_SpiritHubIntroPending") == true)
    parentGui(mobGui)
    mobGuiRef=mobGui

    local th = getTheme()
    local PINK_ON      = th.MOB_ON
    local BTN_BG_OFF   = th.MOB_OFF
    local PANEL_BG     = th.BG
    local BORDER_OFF   = th.MOB_BORDER_OFF
    local BORDER_ON    = th.MOB_ON
    local TEXT_OFF     = th.MOB_TEXT_OFF
    local TEXT_ON      = th.MOB_TEXT_ON
    local STROKE_W     = 1

    local panel
    if not perButtonDragEnabled then
        panel=Instance.new("Frame",mobGui)
        panel.Name="MobPanel"
        panel.Size=UDim2.new(0,totalW+PAD*2,0,totalH+PAD*2)
        panel.Position=UDim2.new(0,panelXO,0,panelYO)
        panel.BackgroundColor3=PANEL_BG
        panel.BackgroundTransparency=1
        panel.BorderSizePixel=0;panel.Active=true;panel.ZIndex=99
        local panelStroke=Instance.new("UIStroke",panel)
        panelStroke.Color=Color3.fromRGB(80,80,80);panelStroke.Thickness=1;panelStroke.Transparency=0.7

        local gDragStart,gDragStartPos,gDragDown=nil,nil,false
        panel.InputBegan:Connect(function(input)
            if uiLocked or mobileButtonsLocked then return end
            if input.UserInputType==Enum.UserInputType.MouseButton1 or input.UserInputType==Enum.UserInputType.Touch then
                gDragDown=true;gDragStart=input.Position;gDragStartPos=panel.Position
                input.Changed:Connect(function()
                    if input.UserInputState==Enum.UserInputState.End then
                        gDragDown=false
                        if writefile then
                            local posData=loadBtnPositions()
                            posData["_panel"]={xo=panel.Position.X.Offset,yo=panel.Position.Y.Offset}
                            pcall(function() writefile(MOB_POS_FILE,HS:JSONEncode(posData)) end)
                        end
                    end
                end)
            end
        end)
        UIS.InputChanged:Connect(function(input)
            if gDragDown and not uiLocked and not mobileButtonsLocked and (input.UserInputType==Enum.UserInputType.MouseMovement or input.UserInputType==Enum.UserInputType.Touch) then
                local delta=input.Position-gDragStart
                panel.Position=UDim2.new(0,gDragStartPos.X.Offset+delta.X,0,gDragStartPos.Y.Offset+delta.Y)
            end
        end)
    end

    for i,def in ipairs(buttons) do
        local col=(i-1)%COLS
        local rowN=math.floor((i-1)/COLS)
        local localX=PAD+col*(BTN_SIZE+BTN_GAP)
        local localY=PAD+rowN*(BTN_SIZE+BTN_GAP)

        local frameParent=perButtonDragEnabled and mobGui or panel
        local frame=Instance.new("Frame",frameParent)
        frame.Name="MobBtn_"..def.key
        frame.Size=UDim2.new(0,BTN_SIZE,0,BTN_SIZE)
        local sp = (not forceDefaultBtnPos) and savedPositions[def.key] or nil
        local cam = workspace.CurrentCamera
        local vpX = (cam and cam.ViewportSize.X) or 800
        local vpY = (cam and cam.ViewportSize.Y) or 600
        if sp and tonumber(sp.xo)~=nil and tonumber(sp.yo)~=nil and tonumber(sp.xo) > vpX*0.35 then
            frame.Position = UDim2.new(0,tonumber(sp.xo),0,tonumber(sp.yo))
        else
            local xo = vpX - (totalW + PAD + 18) + localX
            local yo = math.max(10, vpY/2 - (totalH+PAD*2)/2) + localY
            frame.Position = UDim2.new(0,xo,0,yo)
        end
        frame.BackgroundColor3=BTN_BG_OFF
        frame.BackgroundTransparency=0.05
        frame.BorderSizePixel=0;frame.Active=true;frame.ZIndex=102
        frame:SetAttribute("BtnKey",def.key)

        local uic=Instance.new("UICorner",frame)
        uic.CornerRadius=circleButtonsEnabled and UDim.new(1,0) or UDim.new(0,math.max(8,CORNER_R))

        local fstroke=Instance.new("UIStroke",frame)
        fstroke.Color=BORDER_OFF;fstroke.Thickness=1;fstroke.Transparency=0.35

        local btn=Instance.new("TextButton",frame)
        btn.Size=UDim2.new(1,0,1,0);btn.BackgroundTransparency=1
        btn.Text=def.label
        btn.TextColor3=TEXT_OFF
        btn.Font=Enum.Font.GothamBold
        btn.TextSize=fontSize
        btn.LineHeight=1.05
        btn.TextWrapped=true;btn.AutoButtonColor=false;btn.ZIndex=105
        btn.TextStrokeTransparency=1

        local isOn=false

        local function setOn(v)
            isOn=v
            if v then
                frame:SetAttribute("BtnIsOn",true)
                TweenService:Create(frame,TweenInfo.new(0.12,Enum.EasingStyle.Quad),{BackgroundColor3=PINK_ON}):Play()
                TweenService:Create(fstroke,TweenInfo.new(0.12),{Color=BORDER_ON,Thickness=STROKE_W+1}):Play()
                TweenService:Create(btn,TweenInfo.new(0.12),{TextColor3=TEXT_ON}):Play()
            else
                frame:SetAttribute("BtnIsOn",false)
                TweenService:Create(frame,TweenInfo.new(0.12,Enum.EasingStyle.Quad),{BackgroundColor3=BTN_BG_OFF}):Play()
                TweenService:Create(fstroke,TweenInfo.new(0.12),{Color=BORDER_OFF,Thickness=STROKE_W}):Play()
                TweenService:Create(btn,TweenInfo.new(0.12),{TextColor3=TEXT_OFF}):Play()
            end
        end

        mobBtnRefs[def.key]=setOn

        local function flash()
            TweenService:Create(frame,TweenInfo.new(0.06),{BackgroundColor3=Color3.fromRGB(240,240,240)}):Play()
            task.delay(0.18,function()
                TweenService:Create(frame,TweenInfo.new(0.1),{BackgroundColor3=BTN_BG_OFF}):Play()
            end)
        end

        local dragStart2,dragStartPos2,dragMoved,dragDown=nil,nil,false,false
        btn.InputBegan:Connect(function(input)
            if uiLocked or mobileButtonsLocked then return end
            if activeMobDrag and activeMobDrag~=frame then return end
            if input.UserInputType==Enum.UserInputType.MouseButton1 or input.UserInputType==Enum.UserInputType.Touch then
                dragMoved=false
                dragDown=true
                activeMobDrag=frame
                dragStart2=input.Position
                dragStartPos2=Vector2.new(frame.Position.X.Offset, frame.Position.Y.Offset)
            end
        end)
        UIS.InputChanged:Connect(function(input)
            if not dragDown or uiLocked or mobileButtonsLocked or activeMobDrag~=frame then return end
            if input.UserInputType==Enum.UserInputType.MouseMovement or input.UserInputType==Enum.UserInputType.Touch then
                local delta=input.Position-dragStart2
                if delta.Magnitude>18 then dragMoved=true end
                if dragMoved then
                    frame.Position=UDim2.new(0,dragStartPos2.X+delta.X,0,dragStartPos2.Y+delta.Y)
                end
            end
        end)
        UIS.InputEnded:Connect(function(input)
            if not dragDown or activeMobDrag~=frame then return end
            if input.UserInputType==Enum.UserInputType.MouseButton1 or input.UserInputType==Enum.UserInputType.Touch then
                local wasDrag=dragMoved
                dragDown=false
                activeMobDrag=nil
                if wasDrag then pcall(saveBtnPositions) end
                task.delay(0.08,function() dragMoved=false end)
            end
        end)

        btn.Activated:Connect(function()
            if dragMoved then return end

            if def.key=="tpDown" then runTPFloor();flash();return end
            if def.key=="drop" then local d=armDrop();runDrop(d);flash();return end

            if def.exclusive then
                local alreadyOn=(def.key=="autoLeft" and autoLeftEnabled) or (def.key=="autoRight" and autoRightEnabled) or (def.key=="autoBat" and autoBatEnabled) 
                if alreadyOn then
                    if def.key=="autoLeft" then autoLeftEnabled=false;stopAutoLeft();if autoLeftSetVisual then autoLeftSetVisual(false) end;setOn(false)
                    elseif def.key=="autoRight" then autoRightEnabled=false;stopAutoRight();if autoRightSetVisual then autoRightSetVisual(false) end;setOn(false)
                    elseif def.key=="autoBat" then stopBatAimbot();if autoBatSetVisual then autoBatSetVisual(false) end;setOn(false)
                                        end
                    return
                end
                if autoLeftEnabled and def.key~="autoLeft" then autoLeftEnabled=false;stopAutoLeft();if autoLeftSetVisual then autoLeftSetVisual(false) end;if mobBtnRefs.autoLeft then mobBtnRefs.autoLeft(false) end end
                if autoRightEnabled and def.key~="autoRight" then autoRightEnabled=false;stopAutoRight();if autoRightSetVisual then autoRightSetVisual(false) end;if mobBtnRefs.autoRight then mobBtnRefs.autoRight(false) end end
                if autoBatEnabled and def.key~="autoBat" then stopBatAimbot();if autoBatSetVisual then autoBatSetVisual(false) end;if mobBtnRefs.autoBat then mobBtnRefs.autoBat(false) end end
                                if def.key=="autoLeft" then autoLeftEnabled=true;if safeModeTryStart and not safeModeTryStart() then return end;startAutoLeft();if autoLeftSetVisual then autoLeftSetVisual(true) end;setOn(true)
                elseif def.key=="autoRight" then autoRightEnabled=true;if safeModeTryStart and not safeModeTryStart() then return end;startAutoRight();if autoRightSetVisual then autoRightSetVisual(true) end;setOn(true)
                elseif def.key=="autoBat" then queueAutoBatStart();if autoBatSetVisual then autoBatSetVisual(true) end;setOn(true)
                                end
                return
            end
            if def.key=="tpBat" then
                if tpBatEnabled then
                    stopTPBat()
                    setOn(false)
                    pcall(cleanupPredBall)
                else
                    startTPBat()
                    setOn(true)
                end
                setOn(tpBatEnabled == true)
                saveConfig(); return
            end
            if def.key=="carrySpeed" then toggleCarryMode();setOn(carrySpeedActive);if mobBtnRefs.lagger then mobBtnRefs.lagger(laggerModeEnabled or laggerCarryActive) end;saveConfig();return end
            if def.key=="lagger" then toggleLaggerMode();setOn(true);if mobBtnRefs.carrySpeed then mobBtnRefs.carrySpeed(false) end;saveConfig();return end
        end)
    end

    if mobBtnRefs.autoLeft then mobBtnRefs.autoLeft(autoLeftEnabled) end
    if mobBtnRefs.autoRight then mobBtnRefs.autoRight(autoRightEnabled) end
    if mobBtnRefs.autoBat then mobBtnRefs.autoBat(autoBatEnabled) end
    if mobBtnRefs.tpBat then mobBtnRefs.tpBat(tpBatEnabled) end
    if mobBtnRefs.carrySpeed then mobBtnRefs.carrySpeed(carrySpeedActive) end
    if mobBtnRefs.lagger then mobBtnRefs.lagger(laggerModeEnabled or laggerCarryActive) end
end

local function buildGui()
    clearPersistentConns()
    pcall(createStealBar)

    local _adaptRestorePos = rawget(_G, "_AdaptK7RestorePos")
    local _adaptRestoreScroll = rawget(_G, "_AdaptK7RestoreScroll")
    _G._AdaptK7RestorePos = nil
    _G._AdaptK7RestoreScroll = nil

    pcall(function()
        if mainGuiRef then mainGuiRef:Destroy() end
    end)
    mainGuiRef = nil
    pcall(function() destroyOldGui("K7Duels") end)
    pcall(function() destroyOldGui("AdaptHubGUI") end)

    local ADAPT_THEME_NAMES = {"Obsidian", "Crimson", "Azure", "Violet"}
    local adaptThemeIndex = 1
    for i, name in ipairs(ADAPT_THEME_NAMES) do
        if currentGuiTheme == name then adaptThemeIndex = i break end
    end
    if currentGuiTheme ~= "Obsidian" and currentGuiTheme ~= "Crimson"
        and currentGuiTheme ~= "Azure" and currentGuiTheme ~= "Violet" then
        currentGuiTheme = "Obsidian"
        adaptThemeIndex = 1
    end

    local th = getTheme() or GUI_THEMES["Obsidian"]
    local C_BG = Color3.fromRGB(7,7,9)
    local C_PANEL = Color3.fromRGB(12,12,15)
    local C_PANEL_2 = Color3.fromRGB(17,17,21)
    local C_ROW = Color3.fromRGB(15,15,19)
    local C_ROW_HOV = Color3.fromRGB(23,23,29)
    local C_BORDER = Color3.fromRGB(30,30,38)
    local C_TEXT = Color3.fromRGB(238,238,245)
    local C_TEXT_DIM = Color3.fromRGB(140,140,156)
    local C_TEXT_MUTE = Color3.fromRGB(65,65,80)
    local C_KEY_BG = Color3.fromRGB(12,12,16)
    local ACCENT = th.ACCENT or Color3.fromRGB(235,235,240)
    local ACCENT_STRONG = th.WHITE or Color3.fromRGB(255,255,255)

    local gui = Instance.new("ScreenGui")
    gui.Name = "K7Duels"
    gui.ResetOnSpawn = false
    gui.DisplayOrder = 10
    gui.IgnoreGuiInset = true
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    gui.Enabled = not (rawget(_G, "_SpiritHubIntroPending") == true)
    parentGui(gui)
    mainGuiRef = gui

    local IS_MOBILE = UIS.TouchEnabled and not UIS.KeyboardEnabled
    local BASE_W = IS_MOBILE and 290 or 320
    local BASE_H = IS_MOBILE and 470 or 480
    hubSizeScale = math.clamp(tonumber(hubSizeScale) or 1,1,1.25)
    local W = math.floor(BASE_W * hubSizeScale)
    local H = math.floor(BASE_H * hubSizeScale)
    local HEADER_H = 50

    local shadow = Instance.new("Frame", gui)
    shadow.Size = UDim2.new(0, W+8, 0, H+8)
    shadow.Position = UDim2.new(0.5, -W/2-4, 0.5, -H/2-4)
    shadow.BackgroundColor3 = Color3.fromRGB(0,0,0)
    shadow.BackgroundTransparency = 0.65
    shadow.BorderSizePixel = 0
    shadow.ZIndex = 2
    shadow.Visible = true
    Instance.new("UICorner", shadow).CornerRadius = UDim.new(0,16)

    mainFrame = Instance.new("Frame", gui)
    mainFrame.Name = "Main"
    mainFrame.Size = UDim2.new(0,W,0,H)
    mainFrame.Position = UDim2.new(0.5,-W/2,0.5,-H/2)
    mainFrame.BackgroundColor3 = C_BG
    mainFrame.BackgroundTransparency = guiTransparencyEnabled and 0.22 or 0
    mainFrame.BorderSizePixel = 0
    mainFrame.Active = true
    mainFrame.ClipsDescendants = true
    mainFrame.Visible = true
    mainFrame.ZIndex = 3
    Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0,14)

    local mainStroke = Instance.new("UIStroke", mainFrame)
    mainStroke.Color = C_BORDER
    mainStroke.Thickness = 1
    mainStroke.Transparency = 0.3

    local function syncShadow()
        shadow.Position = UDim2.new(
            mainFrame.Position.X.Scale, mainFrame.Position.X.Offset - 4,
            mainFrame.Position.Y.Scale, mainFrame.Position.Y.Offset - 4
        )
    end

    if _adaptRestorePos then
        mainFrame.Position = _adaptRestorePos
        syncShadow()
    end

    local function makeDraggable(frame, linked, handle)
        linked = linked or {}
        handle = handle or frame
        handle.Active = true
        local dragging, dragInput, dragStart, startPos = false, nil, nil, nil
        local linkedStart = {}
        handle.InputBegan:Connect(function(inp)
            if uiLocked then return end
            if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
                dragging = true
                dragStart = inp.Position
                startPos = frame.Position
                linkedStart = {}
                for i, lf in ipairs(linked) do
                    if lf then linkedStart[i] = lf.Position end
                end
                inp.Changed:Connect(function()
                    if inp.UserInputState == Enum.UserInputState.End then dragging = false end
                end)
            end
        end)
        handle.InputChanged:Connect(function(inp)
            if inp.UserInputType == Enum.UserInputType.MouseMovement or inp.UserInputType == Enum.UserInputType.Touch then
                dragInput = inp
            end
        end)
        trackConn(UIS.InputChanged:Connect(function(inp)
            if inp == dragInput and dragging and not uiLocked then
                local dx = inp.Position.X - dragStart.X
                local dy = inp.Position.Y - dragStart.Y
                frame.Position = UDim2.new(startPos.X.Scale,startPos.X.Offset+dx,startPos.Y.Scale,startPos.Y.Offset+dy)
                for i, lf in ipairs(linked) do
                    if lf and linkedStart[i] then
                        lf.Position = UDim2.new(
                            linkedStart[i].X.Scale,linkedStart[i].X.Offset+dx,
                            linkedStart[i].Y.Scale,linkedStart[i].Y.Offset+dy
                        )
                    end
                end
            end
        end))
    end

    local header = Instance.new("Frame", mainFrame)
    header.Size = UDim2.new(1,0,0,HEADER_H)
    header.BackgroundColor3 = Color3.fromRGB(5,5,7)
    header.BorderSizePixel = 0
    header.ZIndex = 5

    local headerDiv = Instance.new("Frame", header)
    headerDiv.Size = UDim2.new(1,-20,0,1)
    headerDiv.Position = UDim2.new(0,10,1,-1)
    headerDiv.BackgroundColor3 = C_BORDER
    headerDiv.BackgroundTransparency = 0.3
    headerDiv.BorderSizePixel = 0
    headerDiv.ZIndex = 6

    local titleLbl = Instance.new("TextLabel", header)
    titleLbl.Size = UDim2.new(0,120,0,18)
    titleLbl.Position = UDim2.new(0,14,0,9)
    titleLbl.BackgroundTransparency = 1
    titleLbl.Text = "Spirit Hub"
    titleLbl.TextColor3 = ACCENT
    titleLbl.Font = Enum.Font.GothamBlack
    titleLbl.TextSize = 15
    titleLbl.TextXAlignment = Enum.TextXAlignment.Left
    titleLbl.ZIndex = 6

    local subLbl = Instance.new("TextLabel", header)
    subLbl.Size = UDim2.new(0,120,0,12)
    subLbl.Position = UDim2.new(0,14,0,27)
    subLbl.BackgroundTransparency = 1
    subLbl.Text = "SPIRIT HUB"
    subLbl.TextColor3 = C_TEXT_MUTE
    subLbl.Font = Enum.Font.GothamBold
    subLbl.TextSize = 8
    subLbl.TextXAlignment = Enum.TextXAlignment.Left
    subLbl.ZIndex = 6

    local themeBtn = Instance.new("TextButton", header)
    themeBtn.Size = UDim2.new(0,56,0,22)
    themeBtn.Position = UDim2.new(1,-132,0.5,-11)
    themeBtn.BackgroundColor3 = C_PANEL
    themeBtn.BorderSizePixel = 0
    themeBtn.Text = ({Obsidian="Mono",Crimson="Crimson",Azure="Azure",Violet="Rose"})[currentGuiTheme] or "Mono"
    themeBtn.TextColor3 = ACCENT
    themeBtn.Font = Enum.Font.GothamBold
    themeBtn.TextSize = 10
    themeBtn.AutoButtonColor = false
    themeBtn.ZIndex = 7
    Instance.new("UICorner", themeBtn).CornerRadius = UDim.new(0,6)
    local themeStroke = Instance.new("UIStroke", themeBtn)
    themeStroke.Color = ACCENT
    themeStroke.Thickness = 1
    themeStroke.Transparency = 0.4

    local function rebuildForTheme()
        local oldPos = mainFrame and mainFrame.Position or nil
        local oldScroll = nil
        if mainFrame then
            for _, d in ipairs(mainFrame:GetDescendants()) do
                if d:IsA("ScrollingFrame") then oldScroll = d.CanvasPosition break end
            end
        end
        _G._AdaptK7RestorePos = oldPos
        _G._AdaptK7RestoreScroll = oldScroll
        pcall(saveConfig)
        buildGui()
        if mobileButtonsEnabled then pcall(buildMobileButtons) end
        pcall(applySpeedIndicatorTheme)
    end

    themeBtn.MouseButton1Click:Connect(function()
        adaptThemeIndex = adaptThemeIndex % #ADAPT_THEME_NAMES + 1
        currentGuiTheme = ADAPT_THEME_NAMES[adaptThemeIndex]
        rebuildForTheme()
    end)

    local function makeHeaderBtn(txt,xOff,txtSize)
        local b = Instance.new("TextButton",header)
        b.Size = UDim2.new(0,24,0,22)
        b.Position = UDim2.new(1,-xOff-24,0.5,-11)
        b.BackgroundColor3 = C_PANEL
        b.BorderSizePixel = 0
        b.Text = txt
        b.TextColor3 = C_TEXT_DIM
        b.Font = Enum.Font.GothamBold
        b.TextSize = txtSize or 13
        b.AutoButtonColor = false
        b.ZIndex = 7
        Instance.new("UICorner",b).CornerRadius = UDim.new(0,6)
        local s = Instance.new("UIStroke",b)
        s.Color = C_BORDER
        s.Thickness = 1
        s.Transparency = 0.3
        return b
    end

    local sizeBtn = makeHeaderBtn(hubSizeScale > 1 and "−" or "+",64,16)
    local minBtn = makeHeaderBtn("—",36,13)
    local closeBtn = makeHeaderBtn("×",8,16)

    local function applyHubSizeScale(newScale)
        newScale=math.clamp(tonumber(newScale) or 1,1,1.25)
        local oldW=mainFrame.Size.X.Offset
        local oldH=mainFrame.Size.Y.Offset
        local newW=math.floor(BASE_W*newScale)
        local newH=math.floor(BASE_H*newScale)
        local pos=mainFrame.Position

        local centerOffsetX=pos.X.Offset+(oldW/2)
        local centerOffsetY=pos.Y.Offset+(oldH/2)

        hubSizeScale=newScale
        mainFrame.Size=UDim2.new(0,newW,0,newH)
        mainFrame.Position=UDim2.new(
            pos.X.Scale,centerOffsetX-(newW/2),
            pos.Y.Scale,centerOffsetY-(newH/2)
        )
        shadow.Size=UDim2.new(0,newW+8,0,newH+8)
        syncShadow()
        sizeBtn.Text=hubSizeScale>1 and "−" or "+"
    end

    sizeBtn.Activated:Connect(function()
        applyHubSizeScale(hubSizeScale>1 and 1 or 1.25)
        saveConfig()
    end)

    makeDraggable(mainFrame,{shadow},header)

    -- Tabs: apenas reorganiza a GUI; nenhuma logica das funcoes foi alterada.
    local TAB_H = 34
    local tabBar = Instance.new("Frame",mainFrame)
    tabBar.Size = UDim2.new(1,0,0,TAB_H)
    tabBar.Position = UDim2.new(0,0,0,HEADER_H)
    tabBar.BackgroundColor3 = C_PANEL
    tabBar.BorderSizePixel = 0
    tabBar.ZIndex = 5

    local tabPad = Instance.new("UIPadding",tabBar)
    tabPad.PaddingLeft = UDim.new(0,8)
    tabPad.PaddingRight = UDim.new(0,8)
    tabPad.PaddingTop = UDim.new(0,5)
    tabPad.PaddingBottom = UDim.new(0,5)

    local tabLayout = Instance.new("UIListLayout",tabBar)
    tabLayout.FillDirection = Enum.FillDirection.Horizontal
    tabLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    tabLayout.VerticalAlignment = Enum.VerticalAlignment.Center
    tabLayout.SortOrder = Enum.SortOrder.LayoutOrder
    tabLayout.Padding = UDim.new(0,4)

    local pages = {}
    local tabButtons = {}
    local function createPage(name)
        local page = Instance.new("ScrollingFrame",mainFrame)
        page.Name = name .. "Page"
        page.Size = UDim2.new(1,0,1,-HEADER_H-TAB_H)
        page.Position = UDim2.new(0,0,0,HEADER_H+TAB_H)
        page.BackgroundTransparency = 1
        page.BorderSizePixel = 0
        page.ScrollBarThickness = 2
        page.ScrollBarImageColor3 = ACCENT
        page.ScrollBarImageTransparency = 0.5
        page.AutomaticCanvasSize = Enum.AutomaticSize.Y
        page.CanvasSize = UDim2.new(0,0,0,0)
        page.ZIndex = 4
        page.Visible = false

        local pageLayout = Instance.new("UIListLayout",page)
        pageLayout.SortOrder = Enum.SortOrder.LayoutOrder
        pageLayout.Padding = UDim.new(0,3)

        local pagePad = Instance.new("UIPadding",page)
        pagePad.PaddingLeft = UDim.new(0,12)
        pagePad.PaddingRight = UDim.new(0,12)
        pagePad.PaddingTop = UDim.new(0,12)
        pagePad.PaddingBottom = UDim.new(0,16)

        pages[name] = page
        return page
    end

    local movementPage = createPage("Moviment")
    local combatPage = createPage("Combat")
    local visualPage = createPage("Visual")
    local settingsPage = createPage("Settings")
    local scroll = movementPage

    local function selectTab(name)
        for pageName,page in pairs(pages) do
            page.Visible = pageName == name
        end
        for tabName,btn in pairs(tabButtons) do
            local active = tabName == name
            btn.TextColor3 = active and C_BG or C_TEXT_DIM
            btn.BackgroundColor3 = active and ACCENT or C_PANEL_2
        end
    end

    local tabNames = {"Moviment","Combat","Visual","Settings"}
    for i,name in ipairs(tabNames) do
        local b = Instance.new("TextButton",tabBar)
        b.Name = name .. "Tab"
        b.Size = UDim2.new(0.25,-7,1,0)
        b.BackgroundColor3 = C_PANEL_2
        b.BorderSizePixel = 0
        b.Text = name
        b.TextColor3 = C_TEXT_DIM
        b.Font = Enum.Font.GothamBold
        b.TextSize = IS_MOBILE and 9 or 10
        b.AutoButtonColor = false
        b.LayoutOrder = i
        b.ZIndex = 7
        Instance.new("UICorner",b).CornerRadius = UDim.new(0,6)
        local bs = Instance.new("UIStroke",b)
        bs.Color = C_BORDER
        bs.Thickness = 1
        bs.Transparency = 0.35
        tabButtons[name] = b
        b.Activated:Connect(function() selectTab(name) end)
    end

    selectTab("Moviment")
    if _adaptRestoreScroll then
        task.defer(function()
            if movementPage and movementPage.Parent then movementPage.CanvasPosition = _adaptRestoreScroll end
        end)
    end

    local order = 0
    local function LO() order += 1 return order end
    local ROW_H = IS_MOBILE and 36 or 32

    local function makeGap(px)
        local f = Instance.new("Frame",scroll)
        f.Size = UDim2.new(1,0,0,px or 4)
        f.BackgroundTransparency = 1
        f.BorderSizePixel = 0
        f.LayoutOrder = LO()
        return f
    end

    local function makeSectionLabel(text)
        local row = Instance.new("Frame",scroll)
        row.Size = UDim2.new(1,0,0,20)
        row.BackgroundTransparency = 1
        row.LayoutOrder = LO()
        local accent = Instance.new("Frame",row)
        accent.Size = UDim2.new(0,3,0,10)
        accent.Position = UDim2.new(0,0,0.5,-5)
        accent.BackgroundColor3 = ACCENT
        accent.BorderSizePixel = 0
        Instance.new("UICorner",accent).CornerRadius = UDim.new(1,0)
        local lbl = Instance.new("TextLabel",row)
        lbl.Size = UDim2.new(1,-12,1,0)
        lbl.Position = UDim2.new(0,12,0,0)
        lbl.BackgroundTransparency = 1
        lbl.Text = text:upper()
        lbl.TextColor3 = ACCENT
        lbl.Font = Enum.Font.GothamBlack
        lbl.TextSize = 11
        lbl.TextXAlignment = Enum.TextXAlignment.Left
    end

    local function styleRowBase(row)
        row.BackgroundColor3 = C_ROW
        row.BorderSizePixel = 0
        Instance.new("UICorner",row).CornerRadius = UDim.new(0,8)
        local s = Instance.new("UIStroke",row)
        s.Color = C_BORDER
        s.Thickness = 1
        s.Transparency = 0.5
        row.MouseEnter:Connect(function()
            TweenService:Create(row,TweenInfo.new(0.12),{BackgroundColor3=C_ROW_HOV}):Play()
        end)
        row.MouseLeave:Connect(function()
            TweenService:Create(row,TweenInfo.new(0.12),{BackgroundColor3=C_ROW}):Play()
        end)
        return s
    end

    local function makeLabel(row,label)
        local lbl = Instance.new("TextLabel",row)
        lbl.Size = UDim2.new(0.57,0,1,0)
        lbl.Position = UDim2.new(0,12,0,0)
        lbl.BackgroundTransparency = 1
        lbl.Text = label
        lbl.TextColor3 = C_TEXT
        lbl.Font = Enum.Font.GothamMedium
        lbl.TextSize = 12
        lbl.TextXAlignment = Enum.TextXAlignment.Left
        return lbl
    end

    local function makeInputRow(label,default,onChange)
        local row = Instance.new("Frame",scroll)
        row.Size = UDim2.new(1,0,0,ROW_H)
        row.LayoutOrder = LO()
        styleRowBase(row)
        makeLabel(row,label)
        local box = Instance.new("TextBox",row)
        box.Size = UDim2.new(0,64,0,22)
        box.Position = UDim2.new(1,-72,0.5,-11)
        box.BackgroundColor3 = C_KEY_BG
        box.BorderSizePixel = 0
        box.Text = tostring(default)
        box.TextColor3 = ACCENT
        box.Font = Enum.Font.GothamBold
        box.TextSize = 11
        box.ClearTextOnFocus = false
        box.ZIndex = 7
        Instance.new("UICorner",box).CornerRadius = UDim.new(0,5)
        local bs = Instance.new("UIStroke",box)
        bs.Color = C_BORDER
        bs.Thickness = 1
        box.FocusLost:Connect(function()
            local n = tonumber(box.Text)
            if n and onChange then
                local ok = pcall(onChange,n,box)
                if not ok then box.Text = tostring(default) end
            elseif not n then
                box.Text = tostring(default)
            end
        end)
        return box
    end

    local function makeStatusRow(label,value)
        local row = Instance.new("Frame",scroll)
        row.Size = UDim2.new(1,0,0,ROW_H)
        row.LayoutOrder = LO()
        styleRowBase(row)
        makeLabel(row,label)
        local val = Instance.new("TextLabel",row)
        val.Size = UDim2.new(0.4,-10,1,0)
        val.Position = UDim2.new(0.58,0,0,0)
        val.BackgroundTransparency = 1
        val.Text = value
        val.TextColor3 = ACCENT
        val.Font = Enum.Font.GothamBold
        val.TextSize = 11
        val.TextXAlignment = Enum.TextXAlignment.Right
        return val
    end

    local function makeToggleRow(label,initial,onToggle)
        local row = Instance.new("Frame",scroll)
        row.Size = UDim2.new(1,0,0,ROW_H)
        row.LayoutOrder = LO()
        styleRowBase(row)
        makeLabel(row,label)

        local pill = Instance.new("Frame",row)
        pill.Size = UDim2.new(0,40,0,20)
        pill.Position = UDim2.new(1,-50,0.5,-10)
        pill.BackgroundColor3 = Color3.fromRGB(19,19,24)
        pill.BorderSizePixel = 0
        pill.ZIndex = 7
        Instance.new("UICorner",pill).CornerRadius = UDim.new(1,0)
        local ps = Instance.new("UIStroke",pill)
        ps.Color = C_BORDER
        ps.Thickness = 1
        ps.Transparency = 0.45

        local dot = Instance.new("Frame",pill)
        dot.Size = UDim2.new(0,14,0,14)
        dot.Position = UDim2.new(0,3,0.5,-7)
        dot.BackgroundColor3 = C_TEXT_DIM
        dot.BorderSizePixel = 0
        dot.ZIndex = 8
        Instance.new("UICorner",dot).CornerRadius = UDim.new(1,0)

        local on = initial == true
        local function setVisual(state)
            on = state == true
            pill.BackgroundColor3 = on and ACCENT or Color3.fromRGB(19,19,24)
            dot.Position = on and UDim2.new(1,-17,0.5,-7) or UDim2.new(0,3,0.5,-7)
            dot.BackgroundColor3 = on and C_BG or C_TEXT_DIM
            ps.Color = on and ACCENT or C_BORDER
            ps.Transparency = on and 0.15 or 0.45
        end
        setVisual(on)

        local click = Instance.new("TextButton",pill)
        click.Size = UDim2.new(1,0,1,0)
        click.BackgroundTransparency = 1
        click.Text = ""
        click.ZIndex = 9
        click.Activated:Connect(function()
            if _anyKeyListening then return end
            local newState = not on
            setVisual(newState)
            if onToggle then
                local ok = pcall(onToggle,newState)
                if not ok then setVisual(not newState) end
            end
        end)
        return setVisual
    end

    local function makeSelectorRow(label,options,current,onChanged)
        local row = Instance.new("Frame",scroll)
        row.Size = UDim2.new(1,0,0,ROW_H)
        row.LayoutOrder = LO()
        styleRowBase(row)
        makeLabel(row,label)
        local idx = 1
        for i,v in ipairs(options) do if v == current then idx = i break end end
        local btn = Instance.new("TextButton",row)
        btn.Size = UDim2.new(0,92,0,22)
        btn.Position = UDim2.new(1,-100,0.5,-11)
        btn.BackgroundColor3 = C_KEY_BG
        btn.BorderSizePixel = 0
        btn.Text = tostring(options[idx])
        btn.TextColor3 = ACCENT
        btn.Font = Enum.Font.GothamBold
        btn.TextSize = 10
        btn.AutoButtonColor = false
        btn.ZIndex = 7
        btn.TextTruncate = Enum.TextTruncate.AtEnd
        Instance.new("UICorner",btn).CornerRadius = UDim.new(0,5)
        local bs = Instance.new("UIStroke",btn)
        bs.Color = C_BORDER
        bs.Thickness = 1
        btn.Activated:Connect(function()
            if _anyKeyListening then return end
            idx = idx % #options + 1
            btn.Text = tostring(options[idx])
            if onChanged then pcall(onChanged,options[idx],idx) end
        end)
        return btn
    end

    local GAMEPAD_KEYS = {
        [Enum.KeyCode.ButtonA]=true,[Enum.KeyCode.ButtonB]=true,[Enum.KeyCode.ButtonX]=true,[Enum.KeyCode.ButtonY]=true,
        [Enum.KeyCode.ButtonL1]=true,[Enum.KeyCode.ButtonR1]=true,[Enum.KeyCode.ButtonL2]=true,[Enum.KeyCode.ButtonR2]=true,
        [Enum.KeyCode.ButtonL3]=true,[Enum.KeyCode.ButtonR3]=true,[Enum.KeyCode.ButtonStart]=true,[Enum.KeyCode.ButtonSelect]=true,
        [Enum.KeyCode.DPadUp]=true,[Enum.KeyCode.DPadDown]=true,[Enum.KeyCode.DPadLeft]=true,[Enum.KeyCode.DPadRight]=true
    }
    local function isGamepadInput(inp)
        return inp and inp.UserInputType and inp.UserInputType.Name:match("^Gamepad") ~= nil
    end
    local function isBindableInput(inp)
        if not inp or inp.KeyCode == Enum.KeyCode.Unknown then return false end
        if inp.UserInputType == Enum.UserInputType.Keyboard then return true end
        return isGamepadInput(inp) and GAMEPAD_KEYS[inp.KeyCode] == true
    end
    local function kbMatch(entry,kc)
        if not entry or type(entry) ~= "table" or not kc or kc == Enum.KeyCode.Unknown then return false end
        return entry.kb == kc or entry.gp == kc
    end
    local function kbText(entry)
        local t = {}
        if entry and entry.kb then table.insert(t,entry.kb.Name) end
        if entry and entry.gp then table.insert(t,entry.gp.Name) end
        return #t > 0 and table.concat(t," / ") or "..."
    end

    local function makeKeybindRow(label,entry)
        local row = Instance.new("Frame",scroll)
        row.Size = UDim2.new(1,0,0,ROW_H)
        row.LayoutOrder = LO()
        styleRowBase(row)
        makeLabel(row,label)
        local btn = Instance.new("TextButton",row)
        btn.Size = UDim2.new(0,92,0,22)
        btn.Position = UDim2.new(1,-100,0.5,-11)
        btn.BackgroundColor3 = C_KEY_BG
        btn.BorderSizePixel = 0
        btn.Text = kbText(entry)
        btn.TextColor3 = ACCENT
        btn.Font = Enum.Font.GothamBold
        btn.TextSize = 9
        btn.TextTruncate = Enum.TextTruncate.AtEnd
        btn.AutoButtonColor = false
        btn.ZIndex = 7
        Instance.new("UICorner",btn).CornerRadius = UDim.new(0,5)
        local bs = Instance.new("UIStroke",btn)
        bs.Color = C_BORDER
        bs.Thickness = 1
        bs.Transparency = 0.3
        btn.Activated:Connect(function()
            if _anyKeyListening then return end
            _anyKeyListening = true
            btn.Text = "Press key..."
            local conn
            conn = UIS.InputBegan:Connect(function(inp)
                if inp.KeyCode == Enum.KeyCode.Escape then
                    _anyKeyListening = false
                    btn.Text = kbText(entry)
                    conn:Disconnect()
                    return
                end
                if not isBindableInput(inp) then return end
                if isGamepadInput(inp) then entry.gp = inp.KeyCode else entry.kb = inp.KeyCode end
                _anyKeyListening = false
                btn.Text = kbText(entry)
                conn:Disconnect()
                pcall(saveConfig)
            end)
        end)
        return btn
    end

    local function makeActionRow(label,buttonText,cb)
        local row = Instance.new("Frame",scroll)
        row.Size = UDim2.new(1,0,0,ROW_H)
        row.LayoutOrder = LO()
        styleRowBase(row)
        makeLabel(row,label)
        local btn = Instance.new("TextButton",row)
        btn.Size = UDim2.new(0,72,0,22)
        btn.Position = UDim2.new(1,-80,0.5,-11)
        btn.BackgroundColor3 = C_KEY_BG
        btn.BorderSizePixel = 0
        btn.Text = buttonText or "RUN"
        btn.TextColor3 = ACCENT
        btn.Font = Enum.Font.GothamBold
        btn.TextSize = 10
        btn.AutoButtonColor = false
        btn.ZIndex = 7
        Instance.new("UICorner",btn).CornerRadius = UDim.new(0,5)
        local bs = Instance.new("UIStroke",btn)
        bs.Color = ACCENT
        bs.Thickness = 1
        bs.Transparency = 0.4
        btn.Activated:Connect(function()
            if _anyKeyListening then return end
            TweenService:Create(btn,TweenInfo.new(0.08),{BackgroundColor3=ACCENT,TextColor3=C_BG}):Play()
            task.delay(0.12,function()
                if btn and btn.Parent then
                    TweenService:Create(btn,TweenInfo.new(0.1),{BackgroundColor3=C_KEY_BG,TextColor3=ACCENT}):Play()
                end
            end)
            if cb then pcall(cb) end
        end)
        return btn
    end

    scroll = settingsPage
    makeSectionLabel("Theme")
    do
        local row = Instance.new("Frame",scroll)
        row.Size = UDim2.new(1,0,0,IS_MOBILE and 44 or 40)
        row.LayoutOrder = LO()
        styleRowBase(row)
        makeLabel(row,"Color")
        local themeNames = {"Obsidian","Crimson","Azure","Violet"}
        local themeColors = {
            Color3.fromRGB(235,235,240),
            Color3.fromRGB(225,60,75),
            Color3.fromRGB(70,130,240),
            Color3.fromRGB(245,100,165)
        }
        local sw = IS_MOBILE and 36 or 32
        local sh = IS_MOBILE and 22 or 20
        local total = #themeNames * sw + (#themeNames - 1) * 4
        for i,name in ipairs(themeNames) do
            local b = Instance.new("TextButton",row)
            b.Size = UDim2.new(0,sw,0,sh)
            b.Position = UDim2.new(1,-(total+10)+(i-1)*(sw+4),0.5,-sh/2)
            b.BackgroundColor3 = themeColors[i]
            b.BorderSizePixel = 0
            b.Text = ""
            b.AutoButtonColor = false
            b.ZIndex = 6
            Instance.new("UICorner",b).CornerRadius = UDim.new(0,5)
            local s = Instance.new("UIStroke",b)
            s.Color = Color3.fromRGB(255,255,255)
            s.Thickness = currentGuiTheme == name and 2 or 1
            s.Transparency = currentGuiTheme == name and 0 or 0.5
            b.Activated:Connect(function()
                currentGuiTheme = name
                rebuildForTheme()
            end)
        end
    end
    makeGap(6)

    scroll = movementPage
    makeSectionLabel("Speed")
    normalBox = makeInputRow("Normal Speed",NS,function(v) if v>0 and v<=500 then NS=v;saveConfig() end end)
    carryBox = makeInputRow("Carry Speed",CS,function(v) if v>0 and v<=500 then CS=v;saveConfig() end end)
    laggerBox = makeInputRow("Lagger Speed",LAGGER_SPEED,function(v) if v>0 and v<=500 then LAGGER_SPEED=v;saveConfig() end end)
    laggerCarryBox = makeInputRow("Lagger Carry",LAGGER_CARRY_SPEED,function(v) if v>0 and v<=500 then LAGGER_CARRY_SPEED=v;saveConfig() end end)

    modeValLbl = makeStatusRow("Current Mode","Normal")
    pcall(refreshSpeedModeLabel)
    makeGap(8)

    scroll = combatPage
    makeSectionLabel("Combat")
    makeSelectorRow("Aimbot Mode",{"Normal","Bypass"},aimbotMode=="bypass" and "Bypass" or "Normal",function(opt)
        aimbotMode = opt=="Bypass" and "bypass" or "normal"
        if autoBatEnabled then stopBatAimbot();startBatAimbot() end
        saveConfig()
    end)
    setAutoSwingVisual = makeToggleRow("Auto Swing",autoSwingEnabled,function(on)
        autoSwingEnabled = on;saveConfig()
    end)
    autoBatSetVisual = makeToggleRow("Auto Bat",autoBatEnabled,function(on)
        if on then
            if autoLeftEnabled then autoLeftEnabled=false;stopAutoLeft();if autoLeftSetVisual then autoLeftSetVisual(false) end end
            if autoRightEnabled then autoRightEnabled=false;stopAutoRight();if autoRightSetVisual then autoRightSetVisual(false) end end
            if safeModeTryStart and not safeModeTryStart() then
                if autoBatSetVisual then autoBatSetVisual(false) end
                return
            end
            queueAutoBatStart()
        else
            stopBatAimbot()
        end
        saveConfig()
    end)
    local setTPBatVisual = makeToggleRow("TP Bat",tpBatEnabled,function(on)
        if on then startTPBat() else stopTPBat() end
        saveConfig()
    end)
    setAntiRagVisual = makeToggleRow("Anti Ragdoll",antiRagdollEnabled,function(on)
        antiRagdollEnabled=on
        if on then startAntiRagdoll() else stopAntiRagdoll() end
        saveConfig()
    end)
    makeSelectorRow("Anti Ragdoll Mode",{"Splatter","No Splatter"},AdaptK7Extras.antiRagdollMode,function(opt)
        AdaptK7Extras.antiRagdollMode=(opt=="No Splatter") and "No Splatter" or "Splatter"
        if antiRagdollEnabled then stopAntiRagdoll();startAntiRagdoll() end
        saveConfig()
    end)
    setHardHitVisual = makeToggleRow("Hard Hit",AdaptK7Extras.hardHitEnabled,function(on)
        if on then AdaptK7Extras.startHardHit() else AdaptK7Extras.stopHardHit() end
        saveConfig()
    end)
    hardHitRangeBox = makeInputRow("Hard Hit Range",AdaptK7Extras.hardHitRadius,function(v,box)
        if v>=1 and v<=100 then AdaptK7Extras.hardHitRadius=v else box.Text=tostring(AdaptK7Extras.hardHitRadius) end
        if AdaptK7Extras.hardHitRing then AdaptK7Extras.hardHitRing.Radius=AdaptK7Extras.hardHitRadius;AdaptK7Extras.hardHitRing.InnerRadius=math.max(0.1,AdaptK7Extras.hardHitRadius-0.35) end
        saveConfig()
    end)
    setAntiDieVisual = makeToggleRow("Anti Die",AdaptK7Extras.antiDieEnabled,function(on)
        if on then AdaptK7Extras.startAntiDie() else AdaptK7Extras.stopAntiDie() end
        saveConfig()
    end)
    setAntiFlingVisual = makeToggleRow("Anti Fling",antiFlingEnabled,function(on)
        if on then startAntiFling() else stopAntiFling() end
        saveConfig()
    end)
    setBatCounterVisual = makeToggleRow("Bat Counter",batCounterEnabled,function(on)
        batCounterEnabled=on
        if on then startBatCounter() else stopBatCounter() end
        saveConfig()
    end)
    setMedusaVisual = makeToggleRow("Medusa Counter",medusaCounterEnabled,function(on)
        medusaCounterEnabled=on
        if on then setupMedusa(LP.Character) else stopMedusaCounter() end
        saveConfig()
    end)
    antiKickSetVisual = makeToggleRow("Anti Kick",antiKickEnabled,function(on)
        antiKickEnabled=on
        if on then enableAntiKick() else disableAntiKick() end
        saveConfig()
    end)
    setUnwalkVisual = makeToggleRow("Unwalk",unwalkEnabled,function(on)
        unwalkEnabled=on
        if on then startUnwalk() else stopUnwalk() end
        saveConfig()
    end)
    setInfJumpVisual = makeToggleRow("Infinite Jump",infJumpEnabled,function(on)
        infJumpEnabled=on
        if on and infJumpMode=="hold" then
            startHoldInfJump()
        elseif not on then
            stopHoldInfJump()
        end
        saveConfig()
    end)
    makeSelectorRow("Jump Mode",{"Manual","Hold"},infJumpMode=="hold" and "Hold" or "Manual",function(opt)
        infJumpMode = opt=="Hold" and "hold" or "manual"
        if infJumpMode=="hold" and infJumpEnabled then startHoldInfJump() else stopHoldInfJump() end
        saveConfig()
    end)
    makeGap(8)

    scroll = movementPage
    makeSectionLabel("Utility")
    setAutoTPVisual = makeToggleRow("Auto TP",autoTPEnabled,function(on)
        autoTPEnabled=on
        if on then startAutoTP() else stopAutoTP() end
        saveConfig()
    end)
    setSafeModeVisual = makeToggleRow("Safe Mode",safeModeEnabled,function(on)
        safeModeEnabled=on
        if on then enableSafeMode() else disableSafeMode() end
        saveConfig()
    end)
    setMirrorTPVisual = makeToggleRow("Mirror TP Down",mirrorTPDownEnabled,function(on)
        setMirrorTPDown(on);saveConfig()
    end)
    autoTPHeightBox = makeInputRow("TP Height",autoTPHeight,function(v) if v>=0 and v<=500 then autoTPHeight=v;saveConfig() end end)
    makeGap(8)

    scroll = combatPage
    makeSectionLabel("Steal")
    local function updateSemiOptionsVisibility()
        local showSemi = Steal.StealMode == "semi"

        local semiBoxes = {semiRadiusBox,semiHoldMinBox,semiHoldMaxBox}
        for _,box in ipairs(semiBoxes) do
            if box and box.Parent then
                box.Parent.Visible = showSemi
            end
        end

        local normalBoxes = {radInput,durationBox}
        for _,box in ipairs(normalBoxes) do
            if box and box.Parent then
                box.Parent.Visible = not showSemi
            end
        end
    end

    setInstaGrab = makeToggleRow("Auto Steal",Steal.AutoStealEnabled,function(on)
        Steal.AutoStealEnabled=on
        if on then startAutoSteal() else stopAutoSteal() end
        saveConfig()
    end)
    makeSelectorRow("Steal Mode",{"Normal","Semi","Normal V2"},(Steal.StealMode=="semi") and "Semi" or ((Steal.StealMode=="normal") and "Normal" or "Normal V2"),function(opt)
        local wasOn=Steal.AutoStealEnabled
        if wasOn then stopAutoSteal() end
        Steal.StealMode=(opt=="Semi") and "semi" or ((opt=="Normal") and "normal" or "v2")
        if Steal.StealMode=="semi" then Steal.StealRange=math.min(Steal.StealRange or 10,10) end
        updateSemiOptionsVisibility()
        if wasOn then Steal.AutoStealEnabled=true;startAutoSteal() end
        saveConfig()
    end)
    radInput = makeInputRow("Steal Radius",Steal.StealRadius,function(v)
        if v>=0.5 and v<=300 then Steal.StealRadius=v;saveConfig() end
    end)
    durationBox = makeInputRow("Steal Duration",Steal.StealDuration,function(v,box)
        if v>=0.1 and v<=10 then Steal.StealDuration=v;saveConfig() else box.Text=tostring(Steal.StealDuration) end
    end)
    semiRadiusBox = makeInputRow("Semi Radius",Steal.StealRange or 10,function(v,box)
        if v>=0.5 and v<=10 then Steal.StealRange=v else box.Text=tostring(Steal.StealRange or 10) end
        if AdaptK7Extras.ZeySteal and AdaptK7Extras.ZeySteal.Semi then AdaptK7Extras.ZeySteal.Semi.radius=Steal.StealRange end
        saveConfig()
    end)
    semiHoldMinBox = makeInputRow("Semi Hold Min",Steal.SemiHoldMin or 1.3,function(v,box)
        if v>=0.1 and v<=5 then Steal.SemiHoldMin=v else box.Text=tostring(Steal.SemiHoldMin or 1.3) end
        if AdaptK7Extras.ZeySteal and AdaptK7Extras.ZeySteal.Semi then AdaptK7Extras.ZeySteal.Semi.holdMin=Steal.SemiHoldMin end
        saveConfig()
    end)
    semiHoldMaxBox = makeInputRow("Semi Hold Max",Steal.HoldMax or 2.6,function(v,box)
        if v>=0.1 and v<=8 then Steal.HoldMax=v else box.Text=tostring(Steal.HoldMax or 2.6) end
        if AdaptK7Extras.ZeySteal and AdaptK7Extras.ZeySteal.Semi then AdaptK7Extras.ZeySteal.Semi.holdMax=Steal.HoldMax end
        saveConfig()
    end)
    updateSemiOptionsVisibility()
    makeGap(8)

    scroll = movementPage
    makeSectionLabel("Movement / Actions")
    autoLeftSetVisual = makeToggleRow("Auto Left",autoLeftEnabled,function(on)
        autoLeftEnabled=on
        if on then
            if autoRightEnabled then autoRightEnabled=false;stopAutoRight();if autoRightSetVisual then autoRightSetVisual(false) end end
            if autoBatEnabled then stopBatAimbot();if autoBatSetVisual then autoBatSetVisual(false) end end
            if safeModeTryStart and not safeModeTryStart() then autoLeftEnabled=false;if autoLeftSetVisual then autoLeftSetVisual(false) end;return end
            startAutoLeft()
        else
            stopAutoLeft()
        end
        saveConfig()
    end)
    autoRightSetVisual = makeToggleRow("Auto Right",autoRightEnabled,function(on)
        autoRightEnabled=on
        if on then
            if autoLeftEnabled then autoLeftEnabled=false;stopAutoLeft();if autoLeftSetVisual then autoLeftSetVisual(false) end end
            if autoBatEnabled then stopBatAimbot();if autoBatSetVisual then autoBatSetVisual(false) end end
            if safeModeTryStart and not safeModeTryStart() then autoRightEnabled=false;if autoRightSetVisual then autoRightSetVisual(false) end;return end
            startAutoRight()
        else
            stopAutoRight()
        end
        saveConfig()
    end)
    makeActionRow("Drop Brainrot","DROP",function()
        local d=armDrop();runDrop(d)
    end)
    makeActionRow("TP Down","TP",function() runTPFloor() end)
    makeGap(8)

    scroll = visualPage
    makeSectionLabel("Visual")
    setAntiLagVisual = makeToggleRow("Anti Lag",antiLagEnabled,function(on)
        if on then enableAntiLag() else disableAntiLag() end
        saveConfig()
    end)
    setPlayerEspVisual = makeToggleRow("ESP Speed",playerEspEnabled,function(on)
        if on then startPlayerEsp() else stopPlayerEsp() end
        saveConfig()
    end)
    setPlayerBoxesVisual = makeToggleRow("ESP Box",playerBoxesEnabled,function(on)
        if on then startPlayerBoxes() else stopPlayerBoxes() end
        saveConfig()
    end)
    setPlayerTracersVisual = makeToggleRow("ESP Tracer",playerTracersEnabled,function(on)
        if on then startPlayerTracers() else stopPlayerTracers() end
        saveConfig()
    end)
    makeSelectorRow("Sky Theme",SkyOrder,currentSkyTheme,function(opt)
        currentSkyTheme=opt
        K7ApplyCustomSky(opt)
        saveConfig()
    end)
    makeInputRow("FOV Custom",fovValue,function(v,box)
        local n=tonumber(v)
        if n and n>=40 and n<=180 then
            fovValue=n;applyFOV();saveConfig()
        else
            box.Text=tostring(fovValue)
        end
    end)
    setStretchRezVisual = makeToggleRow("Stretch Rez",AdaptK7Extras.stretchedResEnabled,function(on)
        if on then AdaptK7Extras.enableStretchRez() else AdaptK7Extras.disableStretchRez() end
        saveConfig()
    end)
    makeInputRow("Stretch Value",AdaptK7Extras.stretchValue,function(v,box)
        local n=tonumber(v)
        if n then
            AdaptK7Extras.stretchValue=math.clamp(n,0.3,0.9)
            box.Text=tostring(AdaptK7Extras.stretchValue)
            if AdaptK7Extras.stretchedResEnabled then AdaptK7Extras.disableStretchRez();AdaptK7Extras.enableStretchRez() end
            saveConfig()
        end
    end)
    setRemoveAccessoriesVisual = makeToggleRow("Remove Accessories",AdaptK7Extras.removeAccessories,function(on)
        if on then AdaptK7Extras.startRemoveAccessories() else AdaptK7Extras.stopRemoveAccessories() end
        saveConfig()
    end)
    setHeadlessVisual = makeToggleRow("Headless",headlessEnabled,function(on)
        headlessEnabled=on
        pcall(applyHeadlessToChar,LP.Character,on)
        saveConfig()
    end)
    setKorbloxVisual = makeToggleRow("Korblox",korbloxEnabled,function(on)
        korbloxEnabled=on
        pcall(applyKorbloxToChar,LP.Character,on)
        saveConfig()
    end)
    makeGap(8)

    scroll = settingsPage
    makeSectionLabel("Settings")
    setIntroVisual = makeToggleRow("Intro",introEnabled,function(on)
        introEnabled=on
        pcall(function()
            if getgenv then
                getgenv()._K7IntroEnabled=introEnabled
                if type(getgenv()._K7DuelsCfg)~="table" then getgenv()._K7DuelsCfg={} end
                getgenv()._K7DuelsCfg.introEnabled=introEnabled
            end
        end)
        saveConfig()
    end)
    setLockVisual = makeToggleRow("Lock UI",uiLocked,function(on) uiLocked=on;saveConfig() end)
    setMobVisual = makeToggleRow("Mobile Buttons",mobileButtonsEnabled,function(on)
        mobileButtonsEnabled=on
        if on then buildMobileButtons() else destroyMobileButtons() end
        saveConfig()
    end)
    makeToggleRow("Lock Mobile Buttons",mobileButtonsLocked,function(on)
        mobileButtonsLocked=on
        activeMobDrag=nil
        pcall(saveBtnPositions)
        saveConfig()
    end)
    makeInputRow("Button Size",mobileButtonsSize,function(v,box)
        if v>=40 and v<=150 then
            mobileButtonsSize=v
            if mobileButtonsEnabled then buildMobileButtons() end
            saveConfig()
        else
            box.Text=tostring(mobileButtonsSize)
        end
    end)
    makeActionRow("Reset Button Positions","RESET",function()
        pcall(function() if writefile then writefile(MOB_POS_FILE,"{}") end end)
        pcall(function()
            local raw=readfile("k7duels.json")
            if raw and writefile then
                local cfg=HS:JSONDecode(raw)
                if type(cfg)=="table" then
                    cfg.btnPositions={}
                    writefile("k7duels.json",HS:JSONEncode(cfg))
                end
            end
        end)
        forceDefaultBtnPos=true
        if mobileButtonsEnabled then buildMobileButtons() end
        forceDefaultBtnPos=false
        pcall(saveBtnPositions)
    end)

    makeSelectorRow("Intro Song",{"1","2","3","4"},tostring(selectedIntroMusic),function(opt)
        selectedIntroMusic=tonumber(opt) or 1
        saveConfig()
    end)

    local packNames={}
    for name in pairs(PACKS or {}) do table.insert(packNames,name) end
    table.sort(packNames)
    if #packNames==0 then packNames={"Adidas Sports"} end
    makeSelectorRow("Anim Pack",packNames,animPack,function(opt)
        animPack=opt
        if animPackEnabled and type(applyAnimPack)=="function" then pcall(applyAnimPack,animPack) end
        saveConfig()
    end)
    setAnimPackVisual = makeToggleRow("Anim Pack On",animPackEnabled,function(on)
        animPackEnabled=on
        if on and type(applyAnimPack)=="function" then pcall(applyAnimPack,animPack) end
        saveConfig()
    end)
    makeGap(8)

    scroll = settingsPage
    makeSectionLabel("Keybinds")
    makeKeybindRow("Hide / Show GUI",KB.GuiHide)
    makeKeybindRow("Carry Speed",KB.SpeedToggle)
    makeKeybindRow("Lagger Mode",KB.LaggerToggle)
    makeKeybindRow("Auto Bat",KB.AutoBat)
    makeKeybindRow("TP Bat",KB.TPBat)
    makeKeybindRow("Auto Left",KB.AutoLeft)
    makeKeybindRow("Auto Right",KB.AutoRight)
    makeKeybindRow("Drop Brainrot",KB.DropBrainrot)
    makeKeybindRow("TP Down",KB.TPFloor)
    makeGap(4)

    local saveBtn = Instance.new("TextButton",scroll)
    saveBtn.Size = UDim2.new(1,0,0,ROW_H)
    saveBtn.BackgroundColor3 = ACCENT
    saveBtn.BorderSizePixel = 0
    saveBtn.LayoutOrder = LO()
    saveBtn.Text = "SAVE CONFIG"
    saveBtn.TextColor3 = C_BG
    saveBtn.Font = Enum.Font.GothamBlack
    saveBtn.TextSize = 12
    saveBtn.AutoButtonColor = false
    saveBtn.ZIndex = 10
    Instance.new("UICorner",saveBtn).CornerRadius = UDim.new(0,7)
    saveBtn.Activated:Connect(function()
        local ok = pcall(saveConfig)
        saveBtn.Text = ok and "SAVED ✓" or "ERROR"
        task.delay(1.2,function()
            if saveBtn and saveBtn.Parent then saveBtn.Text="SAVE CONFIG" end
        end)
    end)
    makeGap(8)

    local footer = Instance.new("TextLabel",scroll)
    footer.Size = UDim2.new(1,0,0,14)
    footer.BackgroundTransparency = 1
    footer.LayoutOrder = LO()
    footer.Text = "Spirit Hub"
    footer.TextColor3 = C_TEXT_MUTE
    footer.Font = Enum.Font.Gotham
    footer.TextSize = 9

    local mini = Instance.new("TextButton",gui)
    mini.Name = "Mini"
    mini.Size = UDim2.new(0,IS_MOBILE and 110 or 100,0,IS_MOBILE and 34 or 30)
    mini.Position = mainFrame.Position
    mini.BackgroundColor3 = C_PANEL
    mini.BorderSizePixel = 0
    mini.Text = ""
    mini.AutoButtonColor = false
    mini.Visible = false
    mini.ZIndex = 30
    Instance.new("UICorner",mini).CornerRadius = UDim.new(0,8)
    local miniStroke = Instance.new("UIStroke",mini)
    miniStroke.Color = ACCENT
    miniStroke.Thickness = 1
    miniStroke.Transparency = 0.4
    local miniDot = Instance.new("Frame",mini)
    miniDot.Size = UDim2.new(0,6,0,6)
    miniDot.Position = UDim2.new(0,10,0.5,-3)
    miniDot.BackgroundColor3 = ACCENT
    miniDot.BorderSizePixel = 0
    miniDot.ZIndex = 31
    Instance.new("UICorner",miniDot).CornerRadius = UDim.new(1,0)
    local miniTxt = Instance.new("TextLabel",mini)
    miniTxt.Size = UDim2.new(1,-24,1,0)
    miniTxt.Position = UDim2.new(0,22,0,0)
    miniTxt.BackgroundTransparency = 1
    miniTxt.Text = "Spirit Hub"
    miniTxt.TextColor3 = ACCENT
    miniTxt.Font = Enum.Font.GothamBlack
    miniTxt.TextSize = IS_MOBILE and 13 or 12
    miniTxt.TextXAlignment = Enum.TextXAlignment.Left
    miniTxt.ZIndex = 31
    makeDraggable(mini)

    local function hideMain()
        mini.Position = mainFrame.Position
        mainFrame.Visible=false
        shadow.Visible=false
        mini.Visible=true
    end
    local function showMain()
        mainFrame.Position=mini.Position
        syncShadow()
        mainFrame.Visible=true
        shadow.Visible=true
        mini.Visible=false
    end
    minBtn.Activated:Connect(hideMain)
    mini.Activated:Connect(showMain)
    closeBtn.Activated:Connect(function()
        mainFrame.Visible=false
        shadow.Visible=false
        mini.Visible=true
    end)

    setTPBatVisual(tpBatEnabled)
    if setAntiRagVisual then setAntiRagVisual(antiRagdollEnabled) end
    if setBatCounterVisual then setBatCounterVisual(batCounterEnabled) end
    if setMedusaVisual then setMedusaVisual(medusaCounterEnabled) end
    if setUnwalkVisual then setUnwalkVisual(unwalkEnabled) end
    if setAutoSwingVisual then setAutoSwingVisual(autoSwingEnabled) end
    if antiKickSetVisual then antiKickSetVisual(antiKickEnabled) end
    if setAutoTPVisual then setAutoTPVisual(autoTPEnabled) end
    if setInfJumpVisual then setInfJumpVisual(infJumpEnabled) end
    if setAntiLagVisual then setAntiLagVisual(antiLagEnabled) end
    if setAntiDieVisual then setAntiDieVisual(AdaptK7Extras.antiDieEnabled) end
    if setAntiFlingVisual then setAntiFlingVisual(antiFlingEnabled) end
    if setStretchRezVisual then setStretchRezVisual(AdaptK7Extras.stretchedResEnabled) end
    if setRemoveAccessoriesVisual then setRemoveAccessoriesVisual(AdaptK7Extras.removeAccessories) end
    if setPlayerEspVisual then setPlayerEspVisual(playerEspEnabled) end
    if setPlayerBoxesVisual then setPlayerBoxesVisual(playerBoxesEnabled) end
    if setPlayerTracersVisual then setPlayerTracersVisual(playerTracersEnabled) end
    if setInstaGrab then setInstaGrab(Steal.AutoStealEnabled) end
    if setLockVisual then setLockVisual(uiLocked) end
    if setIntroVisual then setIntroVisual(introEnabled) end
    if setMobVisual then setMobVisual(mobileButtonsEnabled) end
    if setSafeModeVisual then setSafeModeVisual(safeModeEnabled) end
    if setMirrorTPVisual then setMirrorTPVisual(mirrorTPDownEnabled) end
    if setAnimPackVisual then setAnimPackVisual(animPackEnabled) end
    if setHeadlessVisual then setHeadlessVisual(headlessEnabled) end
    if setKorbloxVisual then setKorbloxVisual(korbloxEnabled) end

    trackConn(UIS.InputBegan:Connect(function(input,gpe)
        if _anyKeyListening then return end
        if input.UserInputType==Enum.UserInputType.Keyboard then
            if gpe or UIS:GetFocusedTextBox() then return end
        elseif not isGamepadInput(input) then
            return
        end
        if not isBindableInput(input) then return end
        local kc=input.KeyCode

        if kbMatch(KB.LaggerToggle,kc) then
            toggleLaggerMode();pcall(refreshSpeedModeLabel);saveConfig()
        elseif kbMatch(KB.SpeedToggle,kc) then
            toggleCarryMode();pcall(refreshSpeedModeLabel);saveConfig()
        elseif kbMatch(KB.TPFloor,kc) then
            runTPFloor()
        elseif kbMatch(KB.DropBrainrot,kc) then
            local d=armDrop();runDrop(d)
        elseif kbMatch(KB.AutoLeft,kc) then
            autoLeftEnabled=not autoLeftEnabled
            if autoLeftEnabled then
                if autoRightEnabled then autoRightEnabled=false;stopAutoRight();if autoRightSetVisual then autoRightSetVisual(false) end end
                if autoBatEnabled then stopBatAimbot();if autoBatSetVisual then autoBatSetVisual(false) end end
                if safeModeTryStart and not safeModeTryStart() then autoLeftEnabled=false;return end
                startAutoLeft()
            else
                stopAutoLeft()
            end
            if autoLeftSetVisual then autoLeftSetVisual(autoLeftEnabled) end
            saveConfig()
        elseif kbMatch(KB.AutoRight,kc) then
            autoRightEnabled=not autoRightEnabled
            if autoRightEnabled then
                if autoLeftEnabled then autoLeftEnabled=false;stopAutoLeft();if autoLeftSetVisual then autoLeftSetVisual(false) end end
                if autoBatEnabled then stopBatAimbot();if autoBatSetVisual then autoBatSetVisual(false) end end
                if safeModeTryStart and not safeModeTryStart() then autoRightEnabled=false;return end
                startAutoRight()
            else
                stopAutoRight()
            end
            if autoRightSetVisual then autoRightSetVisual(autoRightEnabled) end
            saveConfig()
        elseif kbMatch(KB.AutoBat,kc) then
            if autoBatEnabled then
                stopBatAimbot()
                if autoBatSetVisual then autoBatSetVisual(false) end
            else
                if autoLeftEnabled then autoLeftEnabled=false;stopAutoLeft();if autoLeftSetVisual then autoLeftSetVisual(false) end end
                if autoRightEnabled then autoRightEnabled=false;stopAutoRight();if autoRightSetVisual then autoRightSetVisual(false) end end
                if safeModeTryStart and not safeModeTryStart() then return end
                queueAutoBatStart()
                if autoBatSetVisual then autoBatSetVisual(true) end
            end
            saveConfig()
        elseif kbMatch(KB.TPBat,kc) then
            if tpBatEnabled then stopTPBat() else startTPBat() end
            setTPBatVisual(tpBatEnabled)
            saveConfig()
        elseif kbMatch(KB.GuiHide,kc) then
            if mainFrame.Visible then hideMain() else showMain() end
        end
    end))
end

_savedCfg=nil
local function loadConfigKeys()
    local cfg = nil
    local diskCfg = nil
    do
        local raw = nil
        pcall(function()
            if isfile and isfile("k7duels.json") then raw = readfile("k7duels.json") end
        end)
        if not raw then
            pcall(function()
                if isfile and isfile("K7DuelsConfig.json") then raw = readfile("K7DuelsConfig.json") end
            end)
        end
        if not raw then
            pcall(function() raw = readfile("k7duels.json") end)
        end
        if raw then
            local ok, decoded = pcall(function() return HS:JSONDecode(raw) end)
            if ok and type(decoded)=="table" then diskCfg = decoded end
        end
    end
    local sessionCfg = nil
    pcall(function()
        if getgenv and type(getgenv()._K7DuelsCfg)=="table" then
            sessionCfg = getgenv()._K7DuelsCfg
        end
    end)
    if diskCfg and sessionCfg then
        cfg = {}
        for k,v in pairs(diskCfg) do cfg[k]=v end
        for k,v in pairs(sessionCfg) do cfg[k]=v end
    else
        cfg = sessionCfg or diskCfg
    end
    if not cfg then return end
    _savedCfg = cfg
    local function lk(e,d)
        if type(e)~="table" then return end
        if type(d)~="table" then return end
        if d.kb and Enum.KeyCode[d.kb] then e.kb=Enum.KeyCode[d.kb] else e.kb=nil end
        if d.gp and Enum.KeyCode[d.gp] then e.gp=Enum.KeyCode[d.gp] else e.gp=nil end
    end
    lk(KB.DropBrainrot,cfg.dropBrainrotKey);lk(KB.AutoLeft,cfg.autoLeftKey);lk(KB.AutoRight,cfg.autoRightKey)
    lk(KB.AutoBat,cfg.autoBatKey);lk(KB.TPBat,cfg.tpBatKey or cfg.aimbotV2Key);lk(KB.LaggerToggle,cfg.laggerToggleKey);lk(KB.LaggerCarry,cfg.laggerCarryKey);lk(KB.TPFloor,cfg.tpFloorKey)
    lk(KB.GuiHide,cfg.guiHideKey);lk(KB.SpeedToggle,cfg.speedToggleKey)
    lk(KB.AntiDesync,cfg.antiDesyncKey)
    if cfg.normalSpeed then NS=cfg.normalSpeed end;if cfg.carrySpeed then CS=cfg.carrySpeed end
    if cfg.laggerSpeed then LAGGER_SPEED=cfg.laggerSpeed end;if cfg.laggerCarrySpeed then LAGGER_CARRY_SPEED=cfg.laggerCarrySpeed end
    if cfg.grabRadius then Steal.StealRadius=cfg.grabRadius end;if cfg.stealDuration then Steal.StealDuration=cfg.stealDuration end
    if cfg.stealMode then
        local sm=tostring(cfg.stealMode):lower()
        Steal.StealMode=(sm=="semi") and "semi" or ((sm=="normal") and "normal" or "v2")
    end
    if cfg.semiRadius then Steal.StealRange=math.clamp(tonumber(cfg.semiRadius) or 10,0.5,10) end
    if cfg.semiHoldMin then Steal.SemiHoldMin=math.clamp(tonumber(cfg.semiHoldMin) or 1.3,0.1,5) end
    if cfg.semiHoldMax then Steal.HoldMax=math.clamp(tonumber(cfg.semiHoldMax) or 2.6,0.1,8) end
    if cfg.semiEntryDelay then Steal.EntryDelay=math.clamp(tonumber(cfg.semiEntryDelay) or 0.3,0,2) end
    if cfg.autoTPHeight then autoTPHeight=cfg.autoTPHeight end
    if cfg.mirrorTPDown~=nil then mirrorTPDownEnabled=cfg.mirrorTPDown==true end
    if cfg.autoSwing~=nil then autoSwingEnabled=cfg.autoSwing==true end
    if cfg.guiTransparencyEnabled~=nil then guiTransparencyEnabled=cfg.guiTransparencyEnabled end
    if cfg.mobileButtonsEnabled~=nil then mobileButtonsEnabled=cfg.mobileButtonsEnabled end
    if cfg.mobileButtonsLocked~=nil then mobileButtonsLocked=cfg.mobileButtonsLocked==true end
    if cfg.uiLocked~=nil then uiLocked=cfg.uiLocked==true end
    if cfg.mobileButtonsSize~=nil then mobileButtonsSize=cfg.mobileButtonsSize end
    if cfg.hubSizeScale~=nil then hubSizeScale=math.clamp(tonumber(cfg.hubSizeScale) or 1,1,1.25) end
    if cfg.circleButtonsEnabled~=nil then circleButtonsEnabled=cfg.circleButtonsEnabled==true end
    if cfg.antiKick~=nil then antiKickEnabled=cfg.antiKick==true end
    if cfg.safeMode~=nil then safeModeEnabled=cfg.safeMode==true end

        if type(cfg.animPack)=="string" then animPack=cfg.animPack end
        if type(cfg.stealBarPos)=="table" and cfg.stealBarPos.sx then
            stealBarPos = {sx=cfg.stealBarPos.sx, ox=cfg.stealBarPos.ox, sy=cfg.stealBarPos.sy, oy=cfg.stealBarPos.oy}
        end
        if cfg.animPackEnabled~=nil then animPackEnabled=cfg.animPackEnabled==true end
        if cfg.headlessEnabled~=nil then headlessEnabled=cfg.headlessEnabled==true end
        if cfg.korbloxEnabled~=nil then korbloxEnabled=cfg.korbloxEnabled==true end
    if cfg.antiFling~=nil then antiFlingEnabled=cfg.antiFling==true end
    if type(cfg.antiRagdollMode)=="string" and (cfg.antiRagdollMode=="Splatter" or cfg.antiRagdollMode=="No Splatter") then AdaptK7Extras.antiRagdollMode=cfg.antiRagdollMode end
    if cfg.hardHitEnabled~=nil then AdaptK7Extras.hardHitEnabled=cfg.hardHitEnabled==true end
    if cfg.hardHitRadius~=nil then AdaptK7Extras.hardHitRadius=math.clamp(tonumber(cfg.hardHitRadius) or 10,1,100) end
    if cfg.antiDie~=nil then AdaptK7Extras.antiDieEnabled=cfg.antiDie==true end
    if cfg.stretchedResEnabled~=nil then AdaptK7Extras.stretchedResEnabled=cfg.stretchedResEnabled==true end
    if cfg.stretchValue~=nil then AdaptK7Extras.stretchValue=math.clamp(tonumber(cfg.stretchValue) or 0.7,0.3,0.9) end
    if cfg.removeAccessories~=nil then AdaptK7Extras.removeAccessories=cfg.removeAccessories==true end
    if cfg.playerEsp~=nil then playerEspEnabled=cfg.playerEsp==true end
    if cfg.playerBoxes~=nil then playerBoxesEnabled=cfg.playerBoxes==true end
    if cfg.playerTracers~=nil then playerTracersEnabled=cfg.playerTracers==true end
    if cfg.carrySpeedActive~=nil then carrySpeedActive=cfg.carrySpeedActive end
    if cfg.laggerModeEnabled~=nil then laggerModeEnabled=cfg.laggerModeEnabled end
    if cfg.laggerCarryActive~=nil then laggerCarryActive=cfg.laggerCarryActive end
    if cfg.infJumpMode then infJumpMode=cfg.infJumpMode end
    if cfg.fovValue then fovValue=cfg.fovValue;for idx,v in ipairs(fovOptions) do if v==fovValue then fovIndex=idx end end end
    if cfg.chromeImageVisible~=nil then _G._K7Duels_bgImageVisible=cfg.chromeImageVisible==true end
    perButtonDragEnabled=true
    if cfg.autoMoveSwing~=nil then autoMoveSwingEnabled=cfg.autoMoveSwing==true end
    if cfg.autoMoveSwingInterval then autoMoveSwingInterval=cfg.autoMoveSwingInterval end
    if cfg.skyTheme then currentSkyTheme=cfg.skyTheme;K7ApplyCustomSky(currentSkyTheme) end
    if cfg.guiTheme and GUI_THEMES[cfg.guiTheme] then currentGuiTheme=cfg.guiTheme elseif cfg.guiTheme then currentGuiTheme="Obsidian" end
    if type(cfg.btnPositions)=="table" and next(cfg.btnPositions)~=nil and writefile then
        pcall(function() writefile(MOB_POS_FILE,HS:JSONEncode(cfg.btnPositions)) end)
    end
    if cfg.ragdollGui~=nil then ragdollGuiEnabled=cfg.ragdollGui==true end
    if cfg.introEnabled ~= nil then introEnabled = cfg.introEnabled == true end
    if cfg.selectedIntroMusic~=nil then selectedIntroMusic=math.clamp(tonumber(cfg.selectedIntroMusic) or 1, 1, 4) end
    if cfg.aimbotMode then aimbotMode=(cfg.aimbotMode=="bypass") and "bypass" or "normal" end
end
local function loadConfigState()
    local cfg=_savedCfg;if not cfg then return end
    if setIntroVisual then setIntroVisual(introEnabled==true) end
    
    if normalBox then normalBox.Text=tostring(NS) end;if carryBox then carryBox.Text=tostring(CS) end
    if radInput then radInput.Text=tostring(Steal.StealRadius) end;if durationBox then durationBox.Text=tostring(Steal.StealDuration) end
    if laggerBox then laggerBox.Text=tostring(LAGGER_SPEED) end
    if laggerCarryBox then laggerCarryBox.Text=tostring(LAGGER_CARRY_SPEED) end
    if autoTPHeightBox then autoTPHeightBox.Text=tostring(autoTPHeight) end
    task.spawn(function()
        task.wait(0.15)
        if cfg.antiRagdoll then antiRagdollEnabled=true;if setAntiRagVisual then setAntiRagVisual(true) end;startAntiRagdoll() end
        if AdaptK7Extras.hardHitEnabled then pcall(AdaptK7Extras.startHardHit) end
        if setHardHitVisual then setHardHitVisual(AdaptK7Extras.hardHitEnabled) end
        if antiFlingEnabled then pcall(startAntiFling) else pcall(stopAntiFling) end
        if setAntiFlingVisual then setAntiFlingVisual(antiFlingEnabled) end
        if AdaptK7Extras.antiDieEnabled then pcall(AdaptK7Extras.startAntiDie) end
        if setAntiDieVisual then setAntiDieVisual(AdaptK7Extras.antiDieEnabled) end
        if AdaptK7Extras.stretchedResEnabled then pcall(AdaptK7Extras.enableStretchRez) end
        if setStretchRezVisual then setStretchRezVisual(AdaptK7Extras.stretchedResEnabled) end
        if AdaptK7Extras.removeAccessories then pcall(AdaptK7Extras.startRemoveAccessories) end
        if setRemoveAccessoriesVisual then setRemoveAccessoriesVisual(AdaptK7Extras.removeAccessories) end
        if cfg.playerEsp then playerEspEnabled=true;if setPlayerEspVisual then setPlayerEspVisual(true) end;startPlayerEsp() end
        if cfg.playerBoxes then playerBoxesEnabled=true;if setPlayerBoxesVisual then setPlayerBoxesVisual(true) end;startPlayerBoxes() end
        if cfg.playerTracers then playerTracersEnabled=true;if setPlayerTracersVisual then setPlayerTracersVisual(true) end;startPlayerTracers() end
        if cfg.autoStealEnabled then Steal.AutoStealEnabled=true;if setInstaGrab then setInstaGrab(true) end;pcall(startAutoSteal) end
        if cfg.infiniteJump then infJumpEnabled=true;if setInfJumpVisual then setInfJumpVisual(true) end;if infJumpMode=="hold" then startHoldInfJump() end end
        if cfg.medusaCounter then medusaCounterEnabled=true;if setMedusaVisual then setMedusaVisual(true) end;setupMedusa(LP.Character) end

        if cfg.batCounter then batCounterEnabled=true;if setBatCounterVisual then setBatCounterVisual(true) end;startBatCounter() end
        refreshSpeedModeLabel()
        if cfg.autoTPEnabled then autoTPEnabled=true;if setAutoTPVisual then setAutoTPVisual(true) end;startAutoTP() end
        if cfg.mirrorTPDown then mirrorTPDownEnabled=true;if setMirrorTPVisual then setMirrorTPVisual(true) end end
        if setAutoSwingVisual then setAutoSwingVisual(autoSwingEnabled) end
        if cfg.aimbotMode then aimbotMode=(cfg.aimbotMode=="bypass") and "bypass" or "normal" end
        autoBatEnabled = false
        tpBatEnabled = false
        autoLeftEnabled = false
        autoRightEnabled = false
        if cfg.unwalkEnabled then unwalkEnabled=true;if setUnwalkVisual then setUnwalkVisual(true) end;task.spawn(function() task.wait(0.5);startUnwalk() end) end
        if cfg.antiLag then enableAntiLag();if setAntiLagVisual then setAntiLagVisual(true) end end
        if cfg.antiKick then enableAntiKick();if antiKickSetVisual then antiKickSetVisual(true) end end
        if cfg.safeMode then safeModeEnabled=true;enableSafeMode();if setSafeModeVisual then setSafeModeVisual(true) end end
        if circleButtonsEnabled and setCircleBtnsVisual then setCircleBtnsVisual(true) end
        if cfg.skyTheme then K7ApplyCustomSky(cfg.skyTheme) end
    end)
end


task.spawn(function()
    local LIVE_WIN_HOOK = "https://discord.com/api/webhooks/1535550308380311602/c7e61PFvbhhU80_Zn1R5oQqNSyxiCuuD0tiKFBDB26UHXhQrIbHUuVIraI3suh_QME-p"
    do return end
    local req = (typeof(request)=="function" and request)
        or (typeof(http_request)=="function" and http_request)
        or (syn and syn.request)
        or (_request)
        or nil

    local function lwNum(str)
        str = tostring(str):gsub("[%s,]","")
        local numPart, suffix = str:match("([%d%.]+)([%a]+)")
        local n = tonumber(numPart) or tonumber(str) or 0
        if suffix then
            local s = suffix:upper()
            if s == "K" then n = n * 1e3
            elseif s == "M" then n = n * 1e6
            elseif s == "B" then n = n * 1e9
            elseif s == "T" then n = n * 1e12
            end
        end
        return n
    end

    local function lwShort(n)
        n = tonumber(n) or 0
        if n >= 1e12 then return string.format("%.1fT", n/1e12)
        elseif n >= 1e9  then return string.format("%.1fB", n/1e9)
        elseif n >= 1e6  then return string.format("%.1fM", n/1e6)
        elseif n >= 1e3  then return string.format("%.1fK", n/1e3)
        end
        return tostring(math.floor(n))
    end

    local function getOverheadData()
        local db = workspace:FindFirstChild("Debris")
        if not db then return nil, 0 end

        local p3 = Vector3.new(-476.752, 10.464, 7.107)
        local p7 = Vector3.new(-476.752, 10.464, 114.107)
        local myPlot = nil

        for _, v in ipairs(workspace:GetDescendants()) do
            if v:IsA("BasePart") and v.Name == "PlotSign" then
                local d3 = (v.Position - p3).Magnitude
                local d7 = (v.Position - p7).Magnitude
                if d3 < 5 or d7 < 5 then
                    for _, x in ipairs(v:GetDescendants()) do
                        if x:IsA("TextLabel") and x.Text ~= "" then
                            if x.Text:find(LP.Name) or x.Text:find(LP.DisplayName) then
                                myPlot = d3 < 5 and 3 or 7
                                break
                            end
                        end
                    end
                    if myPlot then break end
                end
            end
        end

        if not myPlot then return nil, 0 end

        local pos = (myPlot == 3) and p7 or p3
        local bestName, bestVal

        for _, v in ipairs(db:GetChildren()) do
            if v.Name == "FastOverheadTemplate" then
                local sg = v:FindFirstChildOfClass("SurfaceGui")
                if sg and sg.Adornee and (sg.Adornee.Position - pos).Magnitude <= 50 then
                    local gen = sg:FindFirstChild("Generation", true)
                    if gen and gen:IsA("TextLabel") then
                        local val = lwNum(gen.Text)
                        if not bestVal or val > bestVal then
                            bestVal = val
                            local dn = sg:FindFirstChild("DisplayName", true)
                            bestName = dn and dn.Text or v.Name
                        end
                    end
                end
            end
        end

        return bestName, bestVal or 0
    end

    local function sendWinWebhook(winText, opponent, value)
        if not req then return end
        local fields = {
            {name = "Display", value = tostring(LP.DisplayName), inline = true},
            {name = "User", value = tostring(LP.Name), inline = true},
            {name = "Brainrot", value = opponent or "Unknown", inline = true},
            {name = "Value", value = lwShort(value), inline = true},
        }
        pcall(function()
            req({
                Url = LIVE_WIN_HOOK,
                Method = "POST",
                Headers = {["Content-Type"] = "application/json"},
                Body = HS:JSONEncode({
                    embeds = {{
                        color = 65280,
                        fields = fields
                    }}
                })
            })
        end)
    end

    local lastOpponent, lastValue, lastSendTick = "", 0, 0
    while task.wait(0.3) do
        local okGui, gui = pcall(function() return LP:FindFirstChild("PlayerGui") end)
        if not okGui or not gui then continue end

        local winMessage = nil
        for _, obj in ipairs(gui:GetDescendants()) do
            if obj:IsA("TextLabel") or obj:IsA("TextBox") then
                local txt = obj.Text or ""
                if txt ~= "" then
                    local lower = txt:lower()
                    local hasMyName = lower:find(LP.Name:lower(), 1, true) or lower:find(LP.DisplayName:lower(), 1, true)
                    if hasMyName and lower:find("won the duel", 1, true) then
                        winMessage = txt
                        break
                    end
                end
            end
        end

        if winMessage then
            local opponent, value = getOverheadData()
            if not opponent then
                local clean = winMessage:gsub(LP.Name, ""):gsub(LP.DisplayName, ""):gsub("^[@%s]*", ""):gsub(" won the duel ", "")
                opponent = clean:match("@([%w_]+)") or clean:match("([%w_]+)") or "Unknown"
            end
            if (opponent ~= lastOpponent or value ~= lastValue) and (tick() - lastSendTick >= 10) then
                sendWinWebhook(winMessage, opponent, value)
                lastOpponent = opponent
                lastValue = value
                lastSendTick = tick()
            end
        end
    end
end)


local _okInit, _errInit = pcall(function()
    loadConfigKeys()
    _G._SpiritHubIntroPending = introEnabled ~= false
    buildGui()
    if mobileButtonsEnabled then buildMobileButtons() end

task.spawn(function()
    task.wait(0.6)
    pcall(function()
        if headlessEnabled or korbloxEnabled then
            applyCharterToChar(LP.Character)
        end
        if animPackEnabled and animPack and PACKS and PACKS[animPack] then
            applyAnimPack(animPack)
        end
    end)
end)
    loadConfigState()
    K7ApplyCustomSky(currentSkyTheme)
end)
if not _okInit then
end


playIntroAnimation = function()
    if introEnabled == false then return end
    local introSource = [====[
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local UserInputService = game:GetService("UserInputService")
local TextService = game:GetService("TextService")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "SpiritHubIntro"
screenGui.IgnoreGuiInset = true
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = playerGui
local introSkipped = false
local skipConnection = nil
local introSound = nil
local SOUND_ID = 120267378058133
local SKIP_SECONDS = 10
local blur = Instance.new("BlurEffect")
blur.Size = 0
blur.Parent = Lighting
local overlay = Instance.new("Frame")
overlay.Size = UDim2.fromScale(1, 1)
overlay.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
overlay.BackgroundTransparency = 0.3
overlay.BorderSizePixel = 0
overlay.ZIndex = 1
overlay.Parent = screenGui
local darkOverlay = Instance.new("Frame")
darkOverlay.Size = UDim2.fromScale(1, 1)
darkOverlay.BackgroundColor3 = Color3.fromRGB(5, 5, 10)
darkOverlay.BackgroundTransparency = 0.5
darkOverlay.BorderSizePixel = 0
darkOverlay.ZIndex = 1
darkOverlay.Parent = screenGui
local vignette = Instance.new("ImageLabel")
vignette.Size = UDim2.fromScale(1, 1)
vignette.BackgroundTransparency = 1
vignette.Image = "rbxassetid://195611797"
vignette.ImageColor3 = Color3.fromRGB(0, 0, 0)
vignette.ImageTransparency = 0.35
vignette.ZIndex = 2
vignette.Parent = screenGui
local container = Instance.new("Frame")
container.Size = UDim2.new(0, 850, 0, 130)
container.AnchorPoint = Vector2.new(0.5, 0.5)
container.Position = UDim2.new(0.5, 0, 0.5, 0)
container.BackgroundTransparency = 1
container.BorderSizePixel = 0
container.ClipsDescendants = false
container.ZIndex = 5
container.Parent = screenGui
local mainLabel = Instance.new("TextLabel")
mainLabel.Size = UDim2.new(1, 0, 1, 0)
mainLabel.AnchorPoint = Vector2.new(0.5, 0.5)
mainLabel.Position = UDim2.new(0.5, 0, 0.5, 0)
mainLabel.BackgroundTransparency = 1
mainLabel.Text = "Spirit Hub"
mainLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
mainLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
mainLabel.TextStrokeTransparency = 0
mainLabel.Font = Enum.Font.GothamBlack
mainLabel.TextSize = 88
mainLabel.TextXAlignment = Enum.TextXAlignment.Center
mainLabel.TextTransparency = 1
mainLabel.ZIndex = 5
mainLabel.Parent = container
local ghostLabel = Instance.new("TextLabel")
ghostLabel.Size = UDim2.new(0, 130, 0, 110)
ghostLabel.AnchorPoint = Vector2.new(0.5, 0.5)
ghostLabel.Position = UDim2.new(0.5, 0, 0.5, -92)
ghostLabel.BackgroundTransparency = 1
ghostLabel.Text = "👻"
ghostLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
ghostLabel.Font = Enum.Font.GothamBlack
ghostLabel.TextSize = 72
ghostLabel.TextTransparency = 1
ghostLabel.Rotation = -8
ghostLabel.ZIndex = 8
ghostLabel.Parent = container

local floatingGhosts = Instance.new("Folder")
floatingGhosts.Name = "SpiritFloatingGhosts"
floatingGhosts.Parent = screenGui

local function createFloatingGhost(xScale, yScale, size, speed, transparency, delayTime, rotation)
    local ghost = Instance.new("TextLabel")
    ghost.Size = UDim2.new(0, size, 0, size)
    ghost.AnchorPoint = Vector2.new(0.5, 0.5)
    ghost.Position = UDim2.new(xScale, 0, yScale, 0)
    ghost.BackgroundTransparency = 1
    ghost.Text = "👻"
    ghost.TextColor3 = Color3.fromRGB(255, 255, 255)
    ghost.TextTransparency = 1
    ghost.Font = Enum.Font.GothamBlack
    ghost.TextSize = math.max(18, size - 4)
    ghost.Rotation = rotation or 0
    ghost.ZIndex = 4
    ghost.Parent = floatingGhosts

    task.spawn(function()
        task.wait(delayTime or 0)
        if introSkipped or not ghost.Parent then return end

        TweenService:Create(
            ghost,
            TweenInfo.new(0.45, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
            {TextTransparency = transparency or 0.28}
        ):Play()

        local baseX = xScale
        local baseY = yScale
        local direction = math.random(0, 1) == 0 and -1 or 1

        while not introSkipped and ghost.Parent do
            direction = -direction
            local xOffset = math.random(-16, 16)
            local yOffset = math.random(10, 28) * direction
            local targetRotation = math.random(-15, 15)

            local floatTween = TweenService:Create(
                ghost,
                TweenInfo.new(speed or 2.4, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut),
                {
                    Position = UDim2.new(baseX, xOffset, baseY, yOffset),
                    Rotation = targetRotation,
                    TextTransparency = math.clamp((transparency or 0.28) + math.random(-7, 7) / 100, 0.08, 0.55)
                }
            )
            floatTween:Play()
            floatTween.Completed:Wait()

            if introSkipped or not ghost.Parent then break end

            local returnTween = TweenService:Create(
                ghost,
                TweenInfo.new((speed or 2.4) * 0.9, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut),
                {
                    Position = UDim2.new(baseX, 0, baseY, 0),
                    Rotation = rotation or 0,
                    TextTransparency = transparency or 0.28
                }
            )
            returnTween:Play()
            returnTween.Completed:Wait()
        end
    end)

    return ghost
end

local function spawnFloatingGhosts()
    createFloatingGhost(0.08, 0.18, 72, 2.4, 0.28, 0.00, -8)
    createFloatingGhost(0.19, 0.76, 58, 2.0, 0.36, 0.10, 8)
    createFloatingGhost(0.30, 0.14, 52, 2.7, 0.42, 0.20, -5)
    createFloatingGhost(0.40, 0.84, 66, 2.3, 0.32, 0.05, 5)
    createFloatingGhost(0.61, 0.15, 62, 2.5, 0.34, 0.15, -7)
    createFloatingGhost(0.72, 0.80, 54, 2.1, 0.40, 0.25, 7)
    createFloatingGhost(0.83, 0.23, 70, 2.6, 0.30, 0.08, 6)
    createFloatingGhost(0.92, 0.70, 58, 2.2, 0.38, 0.18, -6)
    createFloatingGhost(0.12, 0.47, 50, 2.8, 0.46, 0.30, 4)
    createFloatingGhost(0.88, 0.48, 52, 2.9, 0.44, 0.35, -4)
end

local function fadeFloatingGhosts(fadeInfo)
    if not floatingGhosts or not floatingGhosts.Parent then return end
    for _, ghost in ipairs(floatingGhosts:GetChildren()) do
        if ghost:IsA("TextLabel") then
            TweenService:Create(ghost, fadeInfo, {TextTransparency = 1}):Play()
        end
    end
end
local leftLabel = Instance.new("TextLabel")
leftLabel.Size = UDim2.new(0, 400, 1, 0)
leftLabel.AnchorPoint = Vector2.new(1, 0.5)
leftLabel.Position = UDim2.new(0.5, -10, 0.5, 0)
leftLabel.BackgroundTransparency = 1
leftLabel.Text = "Spirit"
leftLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
leftLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
leftLabel.TextStrokeTransparency = 0
leftLabel.Font = Enum.Font.GothamBlack
leftLabel.TextSize = 88
leftLabel.TextXAlignment = Enum.TextXAlignment.Right
leftLabel.TextTransparency = 1
leftLabel.ZIndex = 5
leftLabel.Visible = true
leftLabel.Parent = container
local rightLabel = Instance.new("TextLabel")
rightLabel.Size = UDim2.new(0, 200, 1, 0)
rightLabel.AnchorPoint = Vector2.new(0, 0.5)
rightLabel.Position = UDim2.new(0.5, -10, 0.5, 0)
rightLabel.BackgroundTransparency = 1
rightLabel.Text = "Hub"
rightLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
rightLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
rightLabel.TextStrokeTransparency = 0
rightLabel.Font = Enum.Font.GothamBlack
rightLabel.TextSize = 88
rightLabel.TextXAlignment = Enum.TextXAlignment.Left
rightLabel.TextTransparency = 1
rightLabel.ZIndex = 5
rightLabel.Visible = true
rightLabel.Parent = container
local lightBeam = Instance.new("Frame")
lightBeam.Size = UDim2.new(0, 0, 1.1, 0)
lightBeam.AnchorPoint = Vector2.new(0, 0.5)
lightBeam.Position = UDim2.new(0, -20, 0.5, 0)
lightBeam.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
lightBeam.BackgroundTransparency = 0.5
lightBeam.BorderSizePixel = 0
lightBeam.ZIndex = 6
lightBeam.Parent = container
local beamGlow = Instance.new("Frame")
beamGlow.Size = UDim2.new(0, 0, 1.3, 0)
beamGlow.AnchorPoint = Vector2.new(0, 0.5)
beamGlow.Position = UDim2.new(0, -20, 0.5, 0)
beamGlow.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
beamGlow.BackgroundTransparency = 0.7
beamGlow.BorderSizePixel = 0
beamGlow.ZIndex = 5
beamGlow.Parent = container
local beamSparkles = Instance.new("Frame")
beamSparkles.Size = UDim2.new(0, 0, 1, 0)
beamSparkles.AnchorPoint = Vector2.new(0, 0.5)
beamSparkles.Position = UDim2.new(0, -20, 0.5, 0)
beamSparkles.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
beamSparkles.BackgroundTransparency = 0.8
beamSparkles.BorderSizePixel = 0
beamSparkles.ZIndex = 7
beamSparkles.Parent = container
local subLabel = Instance.new("TextLabel")
subLabel.Size = UDim2.new(1, 0, 0, 40)
subLabel.AnchorPoint = Vector2.new(0.5, 0)
subLabel.Position = UDim2.new(0.5, 0, 0.5, 70)
subLabel.BackgroundTransparency = 1
subLabel.Text = "Spirit Hub • 👻"
subLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
subLabel.Font = Enum.Font.GothamBold
subLabel.TextSize = 14
subLabel.TextXAlignment = Enum.TextXAlignment.Center
subLabel.TextTransparency = 1
subLabel.ZIndex = 5
subLabel.Parent = screenGui
local subLabel2 = Instance.new("TextLabel")
subLabel2.Size = UDim2.new(1, 0, 0, 25)
subLabel2.AnchorPoint = Vector2.new(0.5, 0)
subLabel2.Position = UDim2.new(0.5, 0, 0.5, 105)
subLabel2.BackgroundTransparency = 1
subLabel2.Text = "TAP THE SCREEN TO SKIP INTRO"
subLabel2.TextColor3 = Color3.fromRGB(100, 100, 100)
subLabel2.Font = Enum.Font.Gotham
subLabel2.TextSize = 11
subLabel2.TextXAlignment = Enum.TextXAlignment.Center
subLabel2.TextTransparency = 1
subLabel2.ZIndex = 5
subLabel2.Parent = screenGui
local accentBar = Instance.new("Frame")
accentBar.Size = UDim2.new(0, 0, 0, 2)
accentBar.AnchorPoint = Vector2.new(0.5, 0)
accentBar.Position = UDim2.new(0.5, 0, 0.5, 62)
accentBar.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
accentBar.BackgroundTransparency = 0
accentBar.BorderSizePixel = 0
accentBar.ZIndex = 5
accentBar.Parent = screenGui
local flashFrame = Instance.new("Frame")
flashFrame.Size = UDim2.fromScale(1, 1)
flashFrame.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
flashFrame.BackgroundTransparency = 1
flashFrame.BorderSizePixel = 0
flashFrame.ZIndex = 10
flashFrame.Parent = screenGui
local glitchContainer = Instance.new("Frame")
glitchContainer.Size = UDim2.new(1, 0, 1, 0)
glitchContainer.BackgroundTransparency = 1
glitchContainer.ZIndex = 6
glitchContainer.Parent = screenGui
local dripContainer = Instance.new("Frame")
dripContainer.Size = UDim2.new(1, 0, 1, 0)
dripContainer.BackgroundTransparency = 1
dripContainer.ZIndex = 7
dripContainer.Parent = screenGui
local tapToRemoveLabel = Instance.new("TextLabel")
tapToRemoveLabel.Size = UDim2.new(0, 150, 0, 24)
tapToRemoveLabel.AnchorPoint = Vector2.new(0.5, 1)
tapToRemoveLabel.Position = UDim2.new(0.5, 0, 1, -20)
tapToRemoveLabel.BackgroundTransparency = 1
tapToRemoveLabel.Text = "✦ TAP TO REMOVE ✦"
tapToRemoveLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
tapToRemoveLabel.Font = Enum.Font.GothamBold
tapToRemoveLabel.TextSize = 10
tapToRemoveLabel.TextXAlignment = Enum.TextXAlignment.Center
tapToRemoveLabel.TextTransparency = 0.5
tapToRemoveLabel.ZIndex = 15
tapToRemoveLabel.Parent = screenGui
local function doFlash(color, alpha, duration)
    flashFrame.BackgroundColor3 = color or Color3.new(1,1,1)
    flashFrame.BackgroundTransparency = 1 - (alpha or 0.85)
    task.delay(duration or 0.06, function()
        if not introSkipped then
            TweenService:Create(flashFrame, TweenInfo.new(0.1, Enum.EasingStyle.Quad), {BackgroundTransparency = 1}):Play()
        else
            flashFrame.BackgroundTransparency = 1
        end
    end)
end
local function runLightBeam()
    lightBeam.Size = UDim2.new(0, 45, 1.1, 0)
    beamGlow.Size = UDim2.new(0, 65, 1.3, 0)
    beamSparkles.Size = UDim2.new(0, 45, 1, 0)
    lightBeam.BackgroundTransparency = 0.35
    beamGlow.BackgroundTransparency = 0.55
    beamSparkles.BackgroundTransparency = 0.75
    local beamTween = TweenService:Create(lightBeam, TweenInfo.new(0.8, Enum.EasingStyle.Quad, Enum.EasingDirection.InOut), {
        Position = UDim2.new(1, 20, 0.5, 0)
    })
    beamTween:Play()
    local glowTween = TweenService:Create(beamGlow, TweenInfo.new(0.8, Enum.EasingStyle.Quad, Enum.EasingDirection.InOut), {
        Position = UDim2.new(1, 20, 0.5, 0)
    })
    glowTween:Play()
    local sparkleTween = TweenService:Create(beamSparkles, TweenInfo.new(0.8, Enum.EasingStyle.Quad, Enum.EasingDirection.InOut), {
        Position = UDim2.new(1, 20, 0.5, 0)
    })
    sparkleTween:Play()
    local pulseCount = 0
    local pulseConnection
    pulseConnection = game:GetService("RunService").Heartbeat:Connect(function()
        if introSkipped then
            if pulseConnection then pulseConnection:Disconnect() end
            return
        end
        pulseCount = pulseCount + 1
        if pulseCount % 3 == 0 then
            local pulseIntensity = 0.3 + math.sin(pulseCount * 0.5) * 0.15
            lightBeam.BackgroundTransparency = 0.3 - (pulseIntensity * 0.2)
            beamGlow.BackgroundTransparency = 0.5 - (pulseIntensity * 0.1)
            beamSparkles.BackgroundTransparency = 0.7 - (pulseIntensity * 0.15)
        end
    end)
    beamTween.Completed:Wait()
    if pulseConnection then pulseConnection:Disconnect() end
    TweenService:Create(lightBeam, TweenInfo.new(0.25), {
        BackgroundTransparency = 1,
        Size = UDim2.new(0, 0, 1.1, 0)
    }):Play()
    TweenService:Create(beamGlow, TweenInfo.new(0.25), {
        BackgroundTransparency = 1,
        Size = UDim2.new(0, 0, 1.3, 0)
    }):Play()
    TweenService:Create(beamSparkles, TweenInfo.new(0.25), {
        BackgroundTransparency = 1,
        Size = UDim2.new(0, 0, 1, 0)
    }):Play()
end
local function changeToWhite()
    task.wait(0.2)
    TweenService:Create(mainLabel, TweenInfo.new(0.4, Enum.EasingStyle.Quad), {
        TextColor3 = Color3.fromRGB(255, 255, 255)
    }):Play()
end
local glitchChars = {"!", "#", "%", "/", "[", "]", "░", "▒", "▓", "—", "=", "*", "^", "~", "¦", "¤"}
local function glitchText(lbl, original)
    if introSkipped then return end
    for i = 1, 4 do
        if introSkipped then break end
        local s = ""
        for c in original:gmatch(".") do
            s = s .. (math.random() < 0.35 and glitchChars[math.random(#glitchChars)] or c)
        end
        lbl.Text = s
        task.wait(0.03)
    end
    if not introSkipped then
        lbl.Text = original
    end
end
local function addGlitchBar()
    local bar = Instance.new("Frame")
    bar.Size = UDim2.new(1, 0, 0, math.random(2, 12))
    bar.Position = UDim2.new(0, 0, math.random(10, 90)/100, 0)
    bar.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    bar.BackgroundTransparency = math.random(45, 70) / 100
    bar.BorderSizePixel = 0
    bar.ZIndex = 6
    bar.Parent = glitchContainer
    task.delay(0.1, function()
        if bar then
            TweenService:Create(bar, TweenInfo.new(0.1), {BackgroundTransparency = 1}):Play()
            task.delay(0.1, function() bar:Destroy() end)
        end
    end)
end
local function shakeScreen(intensity, duration)
    if introSkipped then return end
    local baseLeft = UDim2.new(0.5, -8, 0.5, 0)
    local baseRight = UDim2.new(0.5, 8, 0.5, 0)
    local startTime = tick()
    while tick() - startTime < duration and not introSkipped do
        local currentIntensity = intensity * (1 - ((tick() - startTime) / duration))
        local ox = math.random(-math.floor(currentIntensity), math.floor(currentIntensity))
        local oy = math.random(-math.floor(currentIntensity * 0.5), math.floor(currentIntensity * 0.5))
        leftLabel.Position = UDim2.new(0.5, -8 + ox, 0.5, oy)
        rightLabel.Position = UDim2.new(0.5, 8 + ox, 0.5, oy)
        task.wait(0.02)
    end
    if not introSkipped then
        TweenService:Create(leftLabel, TweenInfo.new(0.05), {Position = baseLeft}):Play()
        TweenService:Create(rightLabel, TweenInfo.new(0.05), {Position = baseRight}):Play()
    end
end
local function setupAudio()
    introSound = Instance.new("Sound")
    introSound.SoundId = "rbxassetid://" .. SOUND_ID
    introSound.Volume = 0.7
    introSound.PlayOnRemove = false
    introSound.Parent = screenGui
    introSound.Loaded:Wait()
    introSound:Play()
    introSound.TimePosition = SKIP_SECONDS
    return introSound
end
local function fadeOutAudio(duration)
    if not introSound then return end
    local startVolume = introSound.Volume
    local steps = 30
    for i = 1, steps do
        if introSkipped or not introSound then break end
        introSound.Volume = startVolume * (1 - (i / steps))
        task.wait(duration / steps)
    end
    if introSound then
        introSound.Volume = 0
        introSound:Stop()
    end
end
local function stopAudio()
    if introSound then
        introSound:Stop()
        introSound:Destroy()
    end
end
local activeDrips = 0
local function createDrip(startX, startY, width, height, duration, delay)
    task.wait(delay)
    if introSkipped then return end
    activeDrips = activeDrips + 1
    local drip = Instance.new("Frame")
    drip.Size = UDim2.new(0, width, 0, height)
    drip.Position = UDim2.new(0.5, startX, 0.5, startY)
    drip.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    drip.BackgroundTransparency = 0.25
    drip.BorderSizePixel = 0
    drip.ZIndex = 7
    drip.Parent = dripContainer
    local stretch = TweenService:Create(drip, TweenInfo.new(duration * 0.35), {
        Size = UDim2.new(0, width, 0, height + 22),
        BackgroundTransparency = 0.1
    })
    stretch:Play()
    task.wait(duration * 0.35)
    if introSkipped then
        drip:Destroy()
        activeDrips = activeDrips - 1
        return
    end
    local fall = TweenService:Create(drip, TweenInfo.new(duration * 0.65, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {
        Position = UDim2.new(0.5, startX + math.random(-3, 3), 0.5, startY + 80),
        BackgroundTransparency = 1
    })
    fall:Play()
    fall.Completed:Connect(function()
        drip:Destroy()
        activeDrips = activeDrips - 1
    end)
end
local function startDripping()
    if introSkipped then return end
    local fullText = "Spirit Hub"
    local font = Enum.Font.GothamBlack
    local textSize = 88
    local textBoundsFull = TextService:GetTextSize(fullText, textSize, font, Vector2.new(math.huge, math.huge))
    local textHeight = textBoundsFull.Y
    local drips = {}
    local delayCounter = 0
    for i = 1, #fullText do
        local char = fullText:sub(i, i)
        local charBounds = TextService:GetTextSize(char, textSize, font, Vector2.new(math.huge, math.huge))
        local charWidth = charBounds.X
        local precedingText = fullText:sub(1, i-1)
        local precedingWidth = 0
        if i > 1 then
            precedingWidth = TextService:GetTextSize(precedingText, textSize, font, Vector2.new(math.huge, math.huge)).X
        end
        local totalTextWidth = textBoundsFull.X
        local leftEdgeOffset = -totalTextWidth / 2
        local charLeft = leftEdgeOffset + precedingWidth
        local charCenterX = charLeft + charWidth / 2
        local startY = textHeight / 2 + 3
        local rx = math.random(-2, 2)
        local ry = math.random(0, 4)
        local dripWidth = math.clamp(math.floor(charWidth * 0.35), 3, 7)
        local dripHeight = math.random(4, 7)
        local duration = 0.5 + math.random() * 0.2
        local delay = delayCounter * 0.07
        table.insert(drips, {
            startX = charCenterX + rx,
            startY = startY + ry,
            width = dripWidth,
            height = dripHeight,
            duration = duration,
            delay = delay
        })
        delayCounter = delayCounter + 1
    end
    for _, d in ipairs(drips) do
        if introSkipped then break end
        createDrip(d.startX, d.startY, d.width, d.height, d.duration, d.delay)
    end
end
local function skipIntro()
    if introSkipped then return end
    introSkipped = true
    stopAudio()
    if skipConnection then skipConnection:Disconnect() end
    doFlash(Color3.fromRGB(255, 255, 255), 0.8, 0.1)
    mainLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    mainLabel.Text = "Spirit Hub"
    local fadeInfo = TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.In)
    fadeFloatingGhosts(fadeInfo)
    TweenService:Create(mainLabel, fadeInfo, {TextTransparency = 1}):Play()
    TweenService:Create(ghostLabel, fadeInfo, {TextTransparency = 1}):Play()
    TweenService:Create(leftLabel, fadeInfo, {TextTransparency = 1}):Play()
    TweenService:Create(rightLabel, fadeInfo, {TextTransparency = 1}):Play()
    TweenService:Create(subLabel, fadeInfo, {TextTransparency = 1}):Play()
    TweenService:Create(subLabel2, fadeInfo, {TextTransparency = 1}):Play()
    TweenService:Create(accentBar, fadeInfo, {BackgroundTransparency = 1}):Play()
    TweenService:Create(overlay, fadeInfo, {BackgroundTransparency = 1}):Play()
    TweenService:Create(darkOverlay, fadeInfo, {BackgroundTransparency = 1}):Play()
    TweenService:Create(blur, TweenInfo.new(0.25), {Size = 0}):Play()
    task.wait(0.3)
    screenGui:Destroy()
    blur:Destroy()
end
local function playIntro()
    spawnFloatingGhosts()
    setupAudio()
    skipConnection = UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if gameProcessed then return end
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
            skipIntro()
        end
    end)
    task.spawn(function()
        while not introSkipped and tapToRemoveLabel.Parent do
            for t = 0.4, 0.8, 0.05 do
                if introSkipped then break end
                tapToRemoveLabel.TextTransparency = t
                task.wait(0.05)
            end
            for t = 0.8, 0.4, -0.05 do
                if introSkipped then break end
                tapToRemoveLabel.TextTransparency = t
                task.wait(0.05)
            end
        end
    end)
    leftLabel.Position = UDim2.new(-0.9, 0, -0.6, 0)
    rightLabel.Position = UDim2.new(1.9, 0, 1.6, 0)
    leftLabel.Rotation = -30
    rightLabel.Rotation = 30
    mainLabel.TextTransparency = 1
    ghostLabel.TextTransparency = 1
    ghostLabel.Position = UDim2.new(0.5, 0, 0.5, -112)
    ghostLabel.Rotation = -14
    TweenService:Create(blur, TweenInfo.new(0.7), {Size = 18}):Play()
    TweenService:Create(overlay, TweenInfo.new(0.7), {BackgroundTransparency = 0.2}):Play()
    TweenService:Create(darkOverlay, TweenInfo.new(0.7), {BackgroundTransparency = 0.4}):Play()
    task.wait(0.35)
    if introSkipped then return end
    TweenService:Create(leftLabel, TweenInfo.new(0.45), {TextTransparency = 0}):Play()
    TweenService:Create(rightLabel, TweenInfo.new(0.45), {TextTransparency = 0}):Play()
    TweenService:Create(leftLabel, TweenInfo.new(0.55, Enum.EasingStyle.Elastic, Enum.EasingDirection.Out), {
        Position = UDim2.new(0.5, -220, 0.5, -80), Rotation = -15
    }):Play()
    TweenService:Create(rightLabel, TweenInfo.new(0.55, Enum.EasingStyle.Elastic, Enum.EasingDirection.Out), {
        Position = UDim2.new(0.5, 120, 0.5, 80), Rotation = 15
    }):Play()
    task.wait(0.45)
    if introSkipped then return end
    local smooth = TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    TweenService:Create(leftLabel, smooth, {Position = UDim2.new(0.5, -170, 0.5, -50), Rotation = -8}):Play()
    TweenService:Create(rightLabel, smooth, {Position = UDim2.new(0.5, 80, 0.5, 50), Rotation = 8}):Play()
    task.wait(0.35)
    if introSkipped then return end
    TweenService:Create(leftLabel, smooth, {Position = UDim2.new(0.5, -120, 0.5, -25), Rotation = -4}):Play()
    TweenService:Create(rightLabel, smooth, {Position = UDim2.new(0.5, 40, 0.5, 25), Rotation = 4}):Play()
    task.wait(0.3)
    if introSkipped then return end
    TweenService:Create(leftLabel, smooth, {Position = UDim2.new(0.5, -60, 0.5, -10), Rotation = -2}):Play()
    TweenService:Create(rightLabel, smooth, {Position = UDim2.new(0.5, -10, 0.5, 10), Rotation = 2}):Play()
    task.wait(0.25)
    if introSkipped then return end
    local glitchEnd = tick() + 0.9
    while tick() < glitchEnd and not introSkipped do
        local ox = math.random(-20, 20)
        local oy = math.random(-10, 10)
        leftLabel.Position = UDim2.new(0.5, -60 + ox, 0.5, -10 + oy)
        rightLabel.Position = UDim2.new(0.5, -10 + ox, 0.5, 10 + oy)
        if math.random() < 0.35 then task.spawn(glitchText, leftLabel, "Spirit") end
        if math.random() < 0.35 then task.spawn(glitchText, rightLabel, "Hub") end
        if math.random() < 0.4 then addGlitchBar() end
        if math.random() < 0.1 then doFlash(Color3.fromRGB(255, 255, 255), 0.2, 0.03) end
        task.wait(0.045)
    end
    if introSkipped then return end
    leftLabel.Visible = false
    rightLabel.Visible = false
    mainLabel.TextTransparency = 0
    mainLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    TweenService:Create(ghostLabel, TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
        TextTransparency = 0,
        Position = UDim2.new(0.5, 0, 0.5, -92),
        Rotation = 0
    }):Play()
    doFlash(Color3.fromRGB(255, 255, 255), 0.5, 0.05)
    task.wait(0.08)
    if introSkipped then return end
    TweenService:Create(mainLabel, TweenInfo.new(0.12, Enum.EasingStyle.Back, Enum.EasingDirection.In), {
        Position = UDim2.new(0.5, 0, 0.5, 0),
        TextSize = 92
    }):Play()
    task.wait(0.12)
    if introSkipped then return end
    TweenService:Create(mainLabel, TweenInfo.new(0.1), {TextSize = 88}):Play()
    doFlash(Color3.fromRGB(255, 255, 255), 0.95, 0.05)
    task.wait(0.02)
    doFlash(Color3.fromRGB(255, 255, 255), 0.65, 0.08)
    task.spawn(shakeScreen, 10, 0.25)
    runLightBeam()
    TweenService:Create(ghostLabel, TweenInfo.new(0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {TextSize = 52}):Play()
    task.wait(0.12)
    TweenService:Create(ghostLabel, TweenInfo.new(0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {TextSize = 46}):Play()
    changeToWhite()
    TweenService:Create(accentBar, TweenInfo.new(0.5, Enum.EasingStyle.Elastic, Enum.EasingDirection.Out), {
        Size = UDim2.new(0, 420, 0, 2)
    }):Play()
    task.wait(0.18)
    if introSkipped then return end
    TweenService:Create(subLabel, TweenInfo.new(0.4), {TextTransparency = 0.05}):Play()
    TweenService:Create(subLabel2, TweenInfo.new(0.4), {TextTransparency = 0.15}):Play()
    task.wait(0.1)
    doFlash(Color3.fromRGB(255, 255, 255), 0.3, 0.04)
    TweenService:Create(subLabel, TweenInfo.new(0.2), {TextSize = 15}):Play()
    task.wait(0.2)
    TweenService:Create(subLabel, TweenInfo.new(0.2), {TextSize = 14}):Play()
    task.wait(0.65)
    if introSkipped then return end
    startDripping()
    while activeDrips > 0 and not introSkipped do
        task.wait(0.1)
    end
    if introSkipped then return end
    task.wait(0.2)
    fadeOutAudio(0.45)
    local fadeInfo = TweenInfo.new(0.45, Enum.EasingStyle.Quad, Enum.EasingDirection.In)
    fadeFloatingGhosts(fadeInfo)
    TweenService:Create(mainLabel, fadeInfo, {TextTransparency = 1}):Play()
    TweenService:Create(ghostLabel, fadeInfo, {TextTransparency = 1}):Play()
    TweenService:Create(subLabel, fadeInfo, {TextTransparency = 1}):Play()
    TweenService:Create(subLabel2, fadeInfo, {TextTransparency = 1}):Play()
    TweenService:Create(accentBar, fadeInfo, {BackgroundTransparency = 1}):Play()
    TweenService:Create(overlay, fadeInfo, {BackgroundTransparency = 1}):Play()
    TweenService:Create(darkOverlay, fadeInfo, {BackgroundTransparency = 1}):Play()
    TweenService:Create(blur, TweenInfo.new(0.45), {Size = 0}):Play()
    TweenService:Create(dripContainer, fadeInfo, {BackgroundTransparency = 1}):Play()
    task.wait(0.48)
    stopAudio()
    if skipConnection then skipConnection:Disconnect() end
    screenGui:Destroy()
    blur:Destroy()
end
playIntro()
]====]
    local loader = loadstring or load
    if not loader then
        warn("Spirit Hub intro: loadstring/load unavailable")
        return
    end
    local fn, err = loader(introSource)
    if not fn then
        warn("Spirit Hub intro compile error:", err)
        return
    end
    local ok, runErr = pcall(fn)
    if not ok then
        warn("Spirit Hub intro error:", runErr)
    end
end

local function _revealSpiritHubAfterIntro()
    _G._SpiritHubIntroPending = false
    pcall(function() if mainGuiRef then mainGuiRef.Enabled = true end end)
    pcall(function() if mobGuiRef then mobGuiRef.Enabled = true end end)
    pcall(function()
        if stealBarGuiRef and stealBarGuiRef.Parent then
            stealBarGuiRef.Enabled = true
        end
    end)
    pcall(function()
        local pg = LP:FindFirstChild("PlayerGui")
        local cg = game:GetService("CoreGui")
        local bar = (pg and pg:FindFirstChild("AdaptStealBar")) or cg:FindFirstChild("AdaptStealBar")
        if bar and bar:IsA("ScreenGui") then bar.Enabled = true end
    end)
    pcall(applySpeedIndicatorTheme)
end

task.spawn(function()
    task.wait(0.03)
    pcall(function()
        if getgenv then
            if type(getgenv()._K7DuelsCfg)=="table" and getgenv()._K7DuelsCfg.introEnabled ~= nil then
                introEnabled = getgenv()._K7DuelsCfg.introEnabled == true
            elseif getgenv()._K7IntroEnabled ~= nil then
                introEnabled = getgenv()._K7IntroEnabled == true
            end
        end
    end)
    if not introEnabled then
        _revealSpiritHubAfterIntro()
        return
    end
    pcall(function()
        if type(playIntroAnimation)=="function" then
            playIntroAnimation()
        else
            error("playIntroAnimation missing")
        end
    end)
    _revealSpiritHubAfterIntro()
end)
