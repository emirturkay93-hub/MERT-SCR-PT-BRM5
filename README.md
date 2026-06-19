-- [[ MERT SCRIPTS V1.0 ]]
getgenv().BlackhawkESP_Connections = getgenv().BlackhawkESP_Connections or {}
getgenv().RaycastBudget = 10
local Vector3_new = Vector3.new

local Vector2_new = Vector2.new

local CFrame_new = CFrame.new

local Color3_new = Color3.new

local Color3_fromHSV = Color3.fromHSV

local RaycastParams_new = RaycastParams.new

local math_floor = math.floor

local math_deg = math.deg

local math_rad = math.rad

local math_clamp = math.clamp

local math_abs = math.abs

local pairs = pairs

local ipairs = ipairs

local type = type

local tick = tick

local os_clock = os.clock

local string_format = string.format

local rawget = rawget

local pcall = pcall

local tostring = tostring

local getgenv = getgenv

local getgc = getgc

local filtergc = filtergc

local Drawing = Drawing

local Drawing_new = Drawing and Drawing.new

local CFrame_angles = CFrame.Angles



-- Services

local Players = game:GetService("Players")

local RunService = game:GetService("RunService")

local UserInputService = game:GetService("UserInputService")

local Workspace = game:GetService("Workspace")

local Lighting = game:GetService("Lighting")

local CoreGui = game:GetService("CoreGui")

local ReplicatedStorage = game:GetService("ReplicatedStorage")



local Camera = Workspace.CurrentCamera

local LocalPlayer = Players.LocalPlayer

local Mouse = LocalPlayer:GetMouse()



-- SolveLead defined later at line 746



-- Local cache variables for BulkScan

local gcCache = nil

local lastScan = 0





-- Optimized Weapon Management

local WeaponManager = {

    Equipped = nil,

    Logic = {},

    Visuals = {},

    LastScan = 0,

    -- ScanCooldown unused

    Connection = nil

}



local ESPElements = {}

local CurrentFrameId = 0

local ESPEnabled = false

local ActorManager = {

    Enemies = {},

    Teammates = {},

    Vehicles = {},

    _lastMapUpdate = 0,

    _lastListUpdate = 0,

    _lastTargetUpdate = 0,

    Initialized = false,

    SelectedTarget = nil,

    SelectedActor = nil,

    _rageShots = {},

    _rageShotsNeeded = {},

    _rageLast = {},

    _ignoredTargets = {}

}



-- Forward Declarations

local SetupHooks

local EnableSpeedController

local GetSecureContainer

local DetectMapType = function() return 1 end

local SecureESPContainer = nil

local ShouldShowEntity -- Forward declaration

local RefreshFirearmControllers



-- Core Service Globals (Found via GC)

local InventoryService, ReplicatorService, ClientService, WorldService, BulletService, Network, InputService

local CharacterController, ControllerService, FirearmInventoryClass, ActorClass, Recoiler, EffectsService

local CalibersTable, VehicleService, FirearmInventory, GroundController, HelicopterController



-- Configuration (V12 Persistence Fix)

getgenv().Blackhawk_Config = getgenv().Blackhawk_Config or {

    -- Entity Filters

    ShowPlayers = false,

    ShowZombies = false,

    ShowNPCs = false,

    ShowSquadMembers = false,

    ShowVehicles = false,

    VehicleColor = Color3_new(0, 1, 1),

    VehicleHealthColor = Color3_new(0, 1, 0),

    IgnoreLocalVehicle = false,



    -- Visual Options

    UseBoxESP = false,

    SimpleESP = false,

    UseHighlight = false,

    UseTracers = false,

    ShowNames = false,

    ShowDistance = false,

    ShowHealth = false,

    ShowWeaponInfo = false,

    BoxStyle = "Full",



    -- Vehicle Visuals

    ShowVehicleBox = false,

    VehicleBoxStyle = "Full",

    ShowVehicleName = false,

    ShowVehicleHealth = false,

    VehicleHealthBarSide = "Bottom",



    TextScale = 1.20,

    DynamicTextScaling = true,



    -- Distance Settings

    MaxDistance = 2000,



    -- Combat Settings

    NoRecoil = false,

    NoSpread = false,

    UnlockFiremodes = false,

    RPMValue = 800,

    AutoReload = true,

    InfiniteAmmo = false,



    -- Silent Aim

    SilentAim = false,

    SilentAimFOV = 100,

    SilentAimHitChance = 100,

    SilentAimTargetPart = "Head",



    -- Ragebot

    RageMode = false,

    Rage_BurstSize = 1,

    Rage_LookAt = false,

    Rage_FOV = 360,

    Rage_Range = 500,

    Rage_TargetPart = "Head",



    ShowFOV = false,

    Prediction = false,

    BulletDrop = false,

    ShowPrediction = false,

    PredictionColor = Color3.fromRGB(255, 0, 0),

    HitboxExpander = false,

    HitboxSize = 4,

    SmartTargetSwitch = false, -- Auto-switch to next target when current will die



    -- Character Movement

    Character_Fly = false,

    Character_FlySpeed = 50,

    Character_WalkSpeedEnabled = false,

    Character_WalkSpeed = 16,

    Character_SprintSpeed = 25,

    CharacterNoclip_Enabled = false,

    RecoilEffect_Enabled = false,

    VehicleWaypoints = {}, -- Initialize to avoid empty list on scan



    -- Vehicle Turret Settings

    TurretUnlockFiremodes = false,

    TurretNoRecoil = false,

    TurretNoSpread = false,

    AlwaysDay = false,

    NoFog = false,

    ThermalVision = false,

    NightVision = false,



    -- Colors

    BoxColor = nil,       -- Default to Entity Color

    HealthBarColor = nil, -- Default to Green/Gradient

    PlayerColor = Color3_new(1, 0.65, 0),

    TeammateColor = Color3_new(0, 1, 0),

    ZombieColor = Color3_new(1, 0, 0),

    NPCColor = Color3_new(1, 1, 0),

    SquadColor = Color3_new(0, 1, 0),

    TracerColor = Color3_new(1, 1, 1),



    -- Zombie Type Colors

    ZombieColors = {

        [1] = Color3_new(0.58, 0.29, 0),

        [2] = Color3_new(1, 1, 0),

        [3] = Color3_new(1, 0, 0),

        [4] = Color3_new(0.58, 0, 1)

    },



    -- Layout Settings

    BoxScale = 1.0,





    HealthBarSide = "Left",

    UseHealthGradient = true,



    -- Map-specific

    AutoDetectMap = true,

    CurrentMapType = "Unknown",

    CurrentMapName = "Unknown",



    -- FPS Optimization

    FPS_NoRain = false,

    FPS_NoEffects = false,

    FPS_NoShells = false,

    FPS_NoLensFlare = false,

    FPS_LowQuality = false,



    -- Vehicle Fly

    VehicleFly_Enabled = false,

    VehicleFly_Speed = 100,
    VehicleFly_ToggleKey = Enum.KeyCode.X,
    Character_FlyToggleKey = Enum.KeyCode.V,

    VehicleFly_UpKey = Enum.KeyCode.E,

    VehicleFly_DownKey = Enum.KeyCode.Q,



    -- Vehicle Teleport

    VehicleWaypoints = {},

    ScannedVehicles = {},

    TurretCH_Enabled = false,

    TurretCH_Style = "Cross",

    TurretCH_Color = Color3.fromRGB(0, 255, 0),

    TurretCH_Size = 10,

    TurretCH_Gap = 3,

    TurretCH_Thickness = 1.5,

    Turret_ZoomEnabled = false,

    Turret_ZoomValue = 40,

    VehicleSpeedMultiplier = 0,

    WeaponInfoColor = Color3.fromRGB(255, 255, 100)

}

local Config = getgenv().Blackhawk_Config



-- ==========================================

-- VEHICLE FLY SYSTEM (After Config for scope access)

-- ==========================================



local VehicleFly = {

    HooksApplied = false

}



-- Apply Vehicle Fly Movement (hooks into VehicleClass.Update)

local function ApplyVehicleFly(vehicle, dt, originalUpdate)
    local cfg = getgenv().Blackhawk_Config or {}
    if not cfg.VehicleFly_Enabled or not vehicle.Controlling then

        return originalUpdate(vehicle, dt)

    end



    if not Camera then return originalUpdate(vehicle, dt) end



    -- Calculate movement direction based on camera

    local moveDir = Vector3_new()

    if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveDir = moveDir + Camera.CFrame.LookVector end

    if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveDir = moveDir - Camera.CFrame.LookVector end

    if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveDir = moveDir - Camera.CFrame.RightVector end

    if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveDir = moveDir + Camera.CFrame.RightVector end

    if UserInputService:IsKeyDown(Config.VehicleFly_UpKey) then moveDir = moveDir + Vector3_new(0, 1, 0) end

    if UserInputService:IsKeyDown(Config.VehicleFly_DownKey) then moveDir = moveDir - Vector3_new(0, 1, 0) end



    if moveDir.Magnitude > 0 then moveDir = moveDir.Unit end



    -- Calculate new position

    local currentCF = vehicle.CFrame or (vehicle.Hitbox and vehicle.Hitbox.CFrame) or CFrame_new()

    local newPos = currentCF.Position + (moveDir * (cfg.VehicleFly_Speed or 100) * dt)

    local newCF = CFrame.lookAt(newPos, newPos + Camera.CFrame.LookVector)



    -- Update vehicle

    vehicle.CFrame = newCF

    if vehicle.Hitbox then vehicle.Hitbox.CFrame = newCF end



    -- Update controller solver if available

    pcall(function()

        if ControllerService and ControllerService.Controller then

            local ctrl = ControllerService.Controller

            if ctrl._vehicle == vehicle and ctrl._solver and ctrl._solver.SetState then

                ctrl._solver:SetState(newCF, Vector3_new(), Vector3_new(), vehicle.ComponentReplicates)

            end

        end

    end)



    -- Update actor position

    pcall(function()

        if ReplicatorService and ReplicatorService.LocalActor then

            ReplicatorService.LocalActor.SimulatedPosition = newCF.Position

        end

    end)



    return newCF, vehicle.Hitbox, {}

end



-- Hook VehicleClass Update (called during service initialization)

function VehicleFly.HookVehicles()

    if VehicleFly.HooksApplied then return end



    for _, tbl in pairs(getgc(true)) do

        if type(tbl) == "table" and rawget(tbl, "Update") and not tbl._FlyHooked then

            -- Detect VehicleClass by checking for vehicle-specific methods

            if rawget(tbl, "SetRPM") or rawget(tbl, "_updateLightModes") then

                local old = tbl.Update

                tbl.Update = function(self, dt) return ApplyVehicleFly(self, dt, old) end

                tbl._FlyHooked = true

            end

        end

    end



    VehicleFly.HooksApplied = true

end



-- Toggle Vehicle Fly

function VehicleFly.Toggle()

    Config.VehicleFly_Enabled = not Config.VehicleFly_Enabled

    print("[Vehicle Fly]", Config.VehicleFly_Enabled and "ENABLED" or "DISABLED")

end



-- BRM5 Place IDs (Dumped from PlaceService)

local Places = {

    ["Menu"] = { 16173503753, 2916899287 },

    ["CM_Mission1"] = { 83829699029749, 4843465225 },

    ["OW_Ronograd"] = { 95595459346841, 3701546109 },

    ["OW_Blank"] = { 0, 5899968224 },

    ["HQ_Seychelles"] = { 139188553486454, 14014688944 },

    ["PVP_Coast"] = { 99240342190508, 5480112241 },

    ["PVP_Favela"] = { 123576346999506, 5468388011 },

    ["PVP_NYC"] = { 71499109870653, 3826587512 },

    ["PVP_Tokyo"] = { 125537938344868, 4524359706 },

    ["PVP_Office"] = { 113296614204646, 5289429734 },

    ["PVP_Blank"] = { 0, 10938546013 },

    ["ZMP_NYC"] = { 84460047957624, 4747446334 },

    ["ZME_NYC"] = { 84460047957624, 4747446334 }

}



local lastCollections = nil



local function BulkScan(retry) -- High performance scanning engine

    -- Prevent scanning in Menu (optimization), but return proper structure

    if game.PlaceId == 16173503753 or game.PlaceId == 2916899287 then

        return {}, {

            FirearmInventoryClass = {},

            FirearmInventoryReplicator = {},

            FirearmInventory = {},

            TurretController = {}

        }

    end



    local now = tick()

    -- Only scan if we don't have a cache OR it's been a long time (45s cooldown for RAM stability)

    if not retry and gcCache and (now - lastScan) < 45 then

        return {

            ReplicatorService = ReplicatorService,

            BulletService = BulletService,

        }, lastCollections

    end



    local startTime = os_clock()



    -- Optimized Scanner: Use filtergc with 'table' filter for 10x faster iteration

    local success, res = pcall(function() return filtergc("table") end)

    if not success then success, res = pcall(function() return getgc(true) end) end

    if not success or type(res) ~= "table" then return end



    gcCache = res

    lastScan = now



    -- Local results table to avoid frequent global access

    local found = {}

    local collections = {

        FirearmInventoryClass = {},

        FirearmInventoryReplicator = {},

        FirearmInventory = {},

        TurretController = {}

    }



    local count = 0

    for _, obj in pairs(res) do

        count = count + 1

        if count % 10000 == 0 then task.wait() end -- Anti-Lag PROTECTED



        if type(obj) == "table" then

            -- 1. IDENTIFY UNIQUE SERVICES (Fingerprinting via rawget) - ONLY IF MISSING



            -- 1. IDENTIFY UNIQUE SERVICES (Fingerprinting via rawget)

            -- Use independent IFs to ensure one object can be identified as multiple services (e.g. ControllerService vs CharacterController)



            -- ReplicatorService

            if not ReplicatorService and not found.ReplicatorService and rawget(obj, "Actors") and rawget(obj, "LocalActor") then

                found.ReplicatorService = obj

            end



            -- BulletService

            if not BulletService and not found.BulletService and rawget(obj, "_multithreadSend") then

                found.BulletService = obj

            end



            -- WorldService

            if not WorldService and not found.WorldService and rawget(obj, "InactiveWorld") then

                found.WorldService = obj

            end



            -- ClientService

            if not ClientService and not found.ClientService and rawget(obj, "Clients") and rawget(obj, "LocalClient") then

                found.ClientService = obj

            end



            -- VehicleService

            if not VehicleService and not found.VehicleService and rawget(obj, "Vehicles") and rawget(obj, "Changed") then

                found.VehicleService = obj

            end



            -- EnvironmentService

            if not EnvironmentService and not found.EnvironmentService and rawget(obj, "RainDensity") and (rawget(obj, "_lights") or rawget(obj, "_clouds")) then

                found.EnvironmentService = obj

            end



            -- InventoryService

            if not InventoryService and not found.InventoryService and rawget(obj, "Inventories") and (rawget(obj, "_hasRadio") or rawget(obj, "_droppedItems")) then

                found.InventoryService = obj

            end



            -- CalibersTable

            if not CalibersTable and not found.CalibersTable and rawget(obj, "shotgun_12gauge_00buck") then

                found.CalibersTable = obj

            end



            -- EffectsService

            if not EffectsService and not found.EffectsService and rawget(obj, "BulletLand") and rawget(obj, "BulletFired") then

                found.EffectsService = obj

            end



            -- Network

            if not Network and not found.Network then

                if rawget(obj, "FireServer") and rawget(obj, "_events") then

                    found.Network = obj

                elseif rawget(obj, "_key") and type(rawget(obj, "_key")) == "table" and rawget(obj, "_code") then

                    found.Network = obj

                end

            end



            -- InputService

            if not InputService and not found.InputService and rawget(obj, "_inputMovement") and rawget(obj, "_mouseMovement") then

                found.InputService = obj

            end



            -- ControllerService

            if not ControllerService and not found.ControllerService then

                if rawget(obj, "Controller") ~= nil then -- Key exists

                    local success, hasSimulated = pcall(function() return type(obj.Simulated) == "function" end)

                    if success and hasSimulated then

                        found.ControllerService = obj

                    elseif type(obj.Controller) == "table" and rawget(obj.Controller, "Update") then

                        found.ControllerService = obj -- Fallback to active controller check

                    end

                end

            end



            -- 2. IDENTIFY CLASSES AND PROTOTYPES

            if not CharacterController and not found.CharacterController and rawget(obj, "SetCFrame") and rawget(obj, "SetVehicleGoal") then

                if rawget(obj, "__index") == obj or not rawget(obj, "Parent") then

                    found.CharacterController = obj

                end

            end



            if not FirearmInventoryClass and not found.FirearmInventoryClass and rawget(obj, "Equip") and rawget(obj, "Unequip") and rawget(obj, "Discharge") then

                found.FirearmInventoryClass = obj

            end



            if not Recoiler and not found.Recoiler and rawget(obj, "Impulse") and rawget(obj, "TimeSkip") and rawget(obj, "GetCameraAdjustment") then

                found.Recoiler = obj

            end



            if not ActorClass and not found.ActorClass and rawget(obj, "Update") and rawget(obj, "Vault") and rawget(obj, "Slide") then

                found.ActorClass = obj

            end



            if not CharacterCamera and not found.CharacterCamera and rawget(obj, "Update") and rawget(obj, "Render") and rawget(obj, "Watch") then

                found.CharacterCamera = obj

            end



            if not TurretController and not found.TurretController and rawget(obj, "_discharge") and rawget(obj, "Discharge") then

                found.TurretController = obj

            end



            -- 3. Vehicle Controllers

            if not GroundController and not found.GroundController and rawget(obj, "_toggleEngine") and rawget(obj, "_toggleIndicator") then

                found.GroundController = obj

            end

            if not HelicopterController and not found.HelicopterController and rawget(obj, "_toggleEngine") and rawget(obj, "_toggleSteerMode") and not rawget(obj, "_toggleIndicator") then

                found.HelicopterController = obj

            end



            -- 4. COLLECT INSTANCES

            if rawget(obj, "_firearm") and rawget(obj, "_actor") then

                table.insert(collections.FirearmInventoryReplicator, obj)

            elseif rawget(obj, "_firearm") and rawget(obj, "_bulletsShot") then

                table.insert(collections.FirearmInventory, obj)

            elseif rawget(obj, "_turret") and rawget(obj, "_config") then

                table.insert(collections.TurretController, obj)

            end

        end

    end



    -- Update Globals & Log Found Services

    local function logService(name, obj)

        if obj then

            print(string_format("[BulkScan] Found %s", name))

            return obj

        end

    end



    ReplicatorService = logService("ReplicatorService", found.ReplicatorService) or ReplicatorService

    BulletService = logService("BulletService", found.BulletService) or BulletService

    WorldService = logService("WorldService", found.WorldService) or WorldService

    ClientService = logService("ClientService", found.ClientService) or ClientService

    VehicleService = logService("VehicleService", found.VehicleService) or VehicleService

    EnvironmentService = logService("EnvironmentService", found.EnvironmentService) or EnvironmentService

    InventoryService = logService("InventoryService", found.InventoryService) or InventoryService

    CalibersTable = logService("CalibersTable", found.CalibersTable) or CalibersTable

    EffectsService = logService("EffectsService", found.EffectsService) or EffectsService

    CharacterController = logService("CharacterController", found.CharacterController) or CharacterController

    ControllerService = logService("ControllerService", found.ControllerService) or ControllerService

    FirearmInventoryClass = logService("FirearmInventoryClass", found.FirearmInventoryClass) or FirearmInventoryClass

    Recoiler = logService("Recoiler", found.Recoiler) or Recoiler

    ActorClass = logService("ActorClass", found.ActorClass) or ActorClass

    TurretController = logService("TurretController", found.TurretController) or TurretController

    GroundController = logService("GroundController", found.GroundController) or GroundController

    HelicopterController = logService("HelicopterController", found.HelicopterController) or HelicopterController

    CharacterCamera = logService("CharacterCamera", found.CharacterCamera) or

        (shared.Engine and shared.Engine.CharacterCamera)

    Network = logService("Network", found.Network) or Network

    InputService = logService("InputService", found.InputService) or InputService



    FirearmInventory = collections.FirearmInventory

    lastCollections = collections



    print(string_format("[BulkScan] Memory scan took: %.1fms ", (os_clock() - startTime) * 1000))

    if not found.ReplicatorService then warn("[BulkScan] CRITICAL: ReplicatorService NOT FOUND!") end

    if not found.BulletService then warn("[BulkScan] CRITICAL: BulletService NOT FOUND!") end

    if not found.CharacterController then warn("[BulkScan] CRITICAL: CharacterController NOT FOUND!") end



    pcall(SetupVehicleHooks)



    -- RETRY MECHANISM IF CRITICAL SERVICES MISSING

    if not found.ReplicatorService or not found.BulletService or not found.CharacterController then

        -- Clean up local references to big table (but keep found/collections for caller)

        res = nil



        if not retry then

            task.spawn(function()

                local lastRetry = tick()

                while not (ReplicatorService and BulletService and CharacterController) do

                    -- SAFETY: If in Menu, do NOT retry. Wait for spawning.

                    if game.PlaceId == 16173503753 or game.PlaceId == 2916899287 then

                        task.wait(5)

                        continue

                    end



                    -- IMMEDIATE MEMORY CLEANUP

                    gcCache = nil -- Drop the huge table from cache

                    pcall(function()

                        collectgarbage("count"); collectgarbage("collect")

                    end) -- Force Aggressive GC



                    print("[BulkScan] Critical services missing. FORCE GC executed. Retrying in 5s...")

                    task.wait(5) -- Increased wait to be less aggressive



                    -- Force retry scan

                    BulkScan(true)

                end



                print("[BulkScan] All critical services found!")

            end)

        end

    end



    return found, collections

end



-- Kept for compatibility

local function FindService(serviceName, manualCache)

    if not ReplicatorService then BulkScan() end

    if serviceName == "ReplicatorService" then

        return ReplicatorService

    elseif serviceName == "BulletService" then

        return BulletService

    elseif serviceName == "CharacterController" then

        return CharacterController

    elseif serviceName == "InventoryService" then

        return InventoryService

    end

    return nil

end



--[[

    FORWARD DECLARATIONS

]]



-- Moved GetMuzzlePosition here for visibility

function GetMuzzlePosition()

    -- Priority 0: Active Equipped Controller from Cached WeaponManager

    -- This avoids scanning the huge FirearmInventory table every frame

    if WeaponManager.Equipped then

        local handler = WeaponManager.Equipped

        if handler._originalMuzzle then

            local success, cf = pcall(handler._originalMuzzle, handler)

            if success and cf then return cf.Position end

        elseif handler.GetMuzzleCFrame then

            local success, cf = pcall(handler.GetMuzzleCFrame, handler)

            if success and cf then return cf.Position end

        end

    end



    -- Fallback: Start with Camera Position to avoid nil errors if logic fails

    return Camera.CFrame.Position

end



function GetSecureContainer()

    if SecureESPContainer and SecureESPContainer.Parent then

        return SecureESPContainer

    end



    local function TryCreate(parent)

        local success, result = pcall(function()

            local f = parent:FindFirstChild("ESP_Cache")

            if not f then

                f = Instance.new("ScreenGui")

                f.Name = "ESP_Cache"

                f.ResetOnSpawn = false

                -- Safe syn.protect_gui call

                if (parent.Name == "CoreGui" or parent.Name == "RobloxGui") and type(syn) == "table" and syn.protect_gui then

                    pcall(function() syn.protect_gui(f) end)

                end

                f.Parent = parent

            end

            return f

        end)

        if success and result then return result end

        return nil

    end



    -- Prio 1: specific executor gui container (gethui)

    local success, hui = pcall(function() return gethui() end)

    if success and hui then

        local folder = TryCreate(hui)

        if folder then

            SecureESPContainer = folder

            return folder

        end

    end



    -- Prio 2: CoreGui (if accessible)

    local success2, core = pcall(function() return game:GetService("CoreGui") end)

    if success2 and core then

        local folder = TryCreate(core)

        if folder then

            SecureESPContainer = folder

            return folder

        end

    end



    -- Prio 3: PlayerGui (Fallback)

    if LocalPlayer then

        local pGui = LocalPlayer:FindFirstChild("PlayerGui")

        if pGui then

            local folder = TryCreate(pGui)

            if folder then

                SecureESPContainer = folder

                return folder

            end

        end

    end



    return nil

end



local function InitializeServices()

    -- Optimized Bulk Memory Scan

    local found, collections = BulkScan()

    if not found then return false end



    -- Sync with legacy globals

    CalibersTable = found.CalibersTable



    print("[VoidSens] Services initialized. Setting up hooks...")

    SetupHooks()



    -- 4. Hook BulletService (Global Silent Aim) - Initialize ONCE

    if BulletService then

        -- Only save original if it doesn't exist (prevents recursion on re-execution)

        if not BulletService._originalDischarge then

            BulletService._originalDischarge = BulletService.Discharge

        end



        local lastUsedCaliber, lastUsedVelocity = nil, 2000

        BulletService.Discharge = function(self, originCF, p49, p50, p51, p52, p53, p54, p55, p56, p57)

            local muzzlePos



            if (p51) and (Config.SilentAim or Config.RageMode) and (math.random(1, 100) <= Config.SilentAimHitChance) then

                p57 = false



                local targetHead = nil
                local targetActor = nil

                if Config.RageMode then
                    targetHead = ActorManager.SelectedTarget_RB
                    targetActor = ActorManager.SelectedActor_RB
                end

                if not targetHead and Config.SilentAim then
                    targetHead = ActorManager.SelectedTarget_SA
                    targetActor = ActorManager.SelectedActor_SA
                end



                if targetHead then

                    local targetPos = targetActor._visiblePoint or targetHead.Position

                    local muzzle_velocity = 2000



                    -- Caching for high RPM/High Pellet count performance

                    if p49 == lastUsedCaliber then

                        muzzle_velocity = lastUsedVelocity

                    elseif p49 and BulletService.GetInfo then

                        local vel = BulletService:GetInfo(p49, p50)

                        if vel and vel > 0 then

                            muzzle_velocity = vel

                            lastUsedCaliber = p49

                            lastUsedVelocity = vel

                        end

                    end



                    local targetVelocity = getgenv()._lastPredVel or Vector3_new(0, 0, 0)

                    local gravity = 32.2



                    if originCF then muzzlePos = originCF.Position end

                    if not muzzlePos and GetMuzzlePosition then

                        muzzlePos = GetMuzzlePosition()

                    end



                    if muzzlePos and targetPos then

                        local aimPos, _ = SolveLead(muzzlePos, targetPos, targetVelocity, muzzle_velocity, gravity)

                        getgenv().LastSilentAimUpdate = tick()

                        originCF = CFrame_new(muzzlePos, aimPos)



                        -- Target Acquisition Visual

                        if not ActorManager._lastRBLog or (tick() - ActorManager._lastRBLog > 5) then

                            print(string_format("[Ragebot] TARGET ACQUIRED: %s", targetActor.OwnerName or "NPC"))

                            ActorManager._lastRBLog = tick()

                        end

                    end

                end

            end

            -- [INFINITE AMMO] True Memory Synchronization

            if Config.InfiniteAmmo and self._mag then

                local maxCap = self._max or 100

                self._mag.Ammo = maxCap

                self._mag.Capacity = maxCap

                if self._item and self._item.MetaData then

                    self._item.MetaData.Ammo = maxCap

                    self._item.MetaData.Capacity = maxCap

                end

                -- FORCE SERVER SYNC: Send reload packet if server thinks we are empty

                if Network and (not self._lastSync or tick() - self._lastSync > 0.5) then

                    local mags = self:_getMags()

                    if mags and mags[1] and mags[1].UID then

                        self._lastSync = tick()

                        -- Silent Reload Trigger

                        Network:FireServer("InventoryAction", "Reload", mags[1].UID, false)

                    end

                end

            end



            -- ONE HIT KILL: Send multiple hit replications only if hitting an actor

            if Config.OneHitKill and p51 then

                for i = 1, 5 do

                    BulletService._originalDischarge(self, originCF, p49, p50, p51, p52, p53, p54, p55, p56, p57)

                end

            end



            return BulletService._originalDischarge(self, originCF, p49, p50, p51, p52, p53, p54, p55, p56, p57)

        end

        getgenv()._BulletServiceHooked = true

    end



    return true

end







-- Controller Cache

local AllControllersCache = {

    Visuals = {},

    Logic = {}

}



RefreshFirearmControllers = function(silent)

    -- Throttling

    if tick() - WeaponManager.LastScan < 0.2 then return end

    WeaponManager.LastScan = tick()



    -- Optimized: Use BulkScan and its collections

    local _, collections = BulkScan()

    if not collections then return end



    -- Sync Caches

    WeaponManager.Visuals = collections.FirearmInventoryReplicator

    WeaponManager.Logic = collections.FirearmInventory



    -- Include Turrets in Visuals

    for _, t in ipairs(collections.TurretController) do

        table.insert(WeaponManager.Visuals, t)

    end



    -- Include Active Turret from LocalActor

    if ReplicatorService and ReplicatorService.LocalActor then

        local myActor = ReplicatorService.LocalActor

        if myActor.Turret and type(myActor.Turret) == "table" and myActor.Turret._config then

            table.insert(WeaponManager.Visuals, myActor.Turret)

        end

    end



    -- Sync with legacy globals

    AllControllersCache.Visuals = WeaponManager.Visuals

    AllControllersCache.Logic = WeaponManager.Logic

    FirearmInventory = WeaponManager.Logic



    -- Update Equipped reference

    if InventoryService and InventoryService.Equipped then

        WeaponManager.Equipped = InventoryService.Equipped.Handler

    end

end







-- Projectile lead and bullet drop calculation
local function SolveLead(sourcePos, targetPos, targetVelocity, bulletSpeed, gravity)
    if not sourcePos or not targetPos then return targetPos, 0 end
    
    local distance = (targetPos - sourcePos).Magnitude
    local time = distance / (bulletSpeed > 0 and bulletSpeed or 2000)

    local predictedPos = targetPos
    if targetVelocity and type(targetVelocity) == "userdata" and targetVelocity.Magnitude > 0.1 then
        -- HÄ±z sÄ±nÄ±rÄ±: helikopterlerde saÃ§malamamasÄ± iÃ§in
        if targetVelocity.Magnitude > 300 then
            targetVelocity = targetVelocity.Unit * 300
        end
        predictedPos = targetPos + (targetVelocity * time)
    end

    distance = (predictedPos - sourcePos).Magnitude
    time = distance / (bulletSpeed > 0 and bulletSpeed or 2000)
    
    if targetVelocity and type(targetVelocity) == "userdata" and targetVelocity.Magnitude > 0.1 then
        predictedPos = targetPos + (targetVelocity * time)
    end

    local dropCompensation = 0.5 * gravity * (time * time)
    predictedPos = predictedPos + Vector3_new(0, dropCompensation, 0)

    -- NaN veya hatalÄ± deÄŸer kontrolÃ¼ (saÄŸ alta kaymayÄ± Ã¶nler)
    if predictedPos.X ~= predictedPos.X or predictedPos.Y ~= predictedPos.Y or predictedPos.Z ~= predictedPos.Z then
        return targetPos, 0
    end

    return predictedPos, time
end



-- Calculate expected damage based on caliber dropoff

-- GetBulletDamage was unused and removed



-- Calculate damage based on caliber, distance, and body part

-- Mirrors BulletService.GetDamageGraph logic from game



--[[

    ENTITY CLASSIFICATION

]]

local function GetEntityType(actor)

    -- Check if zombie

    if actor.Zombie == true then

        return "Zombie"

    end



    -- Check if player (has valid Player owner in game)

    if actor.Owner and Players:GetPlayerByUserId(actor.Owner.UserId) then

        return "Player"

    end



    -- Check if NPC (has owner but not a real player, or AI-controlled)

    if actor.Owner or actor.OwnerName then

        return "NPC"

    end



    return "Unknown"

end



local function IsTeammate(actor, cachedLocalSquad)

    -- Rule 0: Zombies are NEVER teammates

    if actor.Zombie == true then return false end



    -- Detect if it's a real player

    local ownerPlayer = actor.Owner and Players:GetPlayerByUserId(actor.Owner.UserId)

    local isRealPlayer = ownerPlayer ~= nil



    -- Rule 1: Friendly Maps (Only real PLAYERS are teammates)

    if isRealPlayer and (Config.CurrentMapName == "OW_Ronograd" or Config.CurrentMapName == "ZME_NYC") then

        return true

    end



    -- Rule 2: Team Check (PVP)

    if isRealPlayer and ownerPlayer.Team and LocalPlayer.Team and ownerPlayer.Team == LocalPlayer.Team then

        return true

    end



    -- Rule 3: ClientService Squad Check

    if isRealPlayer and cachedLocalSquad and cachedLocalSquad ~= "" then

        if ClientService.Clients and ClientService.Clients[ownerPlayer] then

            local targetClient = ClientService.Clients[ownerPlayer]

            return targetClient.Squad == cachedLocalSquad

        end

    end



    return false

end



--[[

    SPATIAL PARTITIONING HELPERS

]]

local function GetSectorKey(position, sectorSize)

    local x = math.floor(position.X / sectorSize)

    local z = math.floor(position.Z / sectorSize)

    return x * 65536 + z -- Numeric key: faster hash than string concat

end



local function DecodeSectorKey(key)

    -- Decode numeric key back to x, z

    local x = math.floor(key / 65536)

    local z = key - x * 65536

    -- Handle negative z (two's complement style)

    if z > 32767 then z = z - 65536 end

    return x, z

end



--[[

    ACTOR MANAGER (Entity Caching System)

    Decouples heavy entity iteration from RenderStepped

]]

local function IsActorActuallyDead(actor)
    if not actor then return true end
    if actor.Alive == false then return true end
    if actor.Dead == true then return true end
    if actor.Downed == true then return true end
    
    if type(actor.Health) == "number" and actor.Health <= 0 then return true end
    if type(actor.Health) == "table" and type(actor.Health.Value) == "number" and actor.Health.Value <= 0 then return true end

    local char = actor.Character
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum and hum.Health <= 0 then return true end
    end
    
    return false
end

function ActorManager:Update(dt)

    if not ReplicatorService or not ReplicatorService.Actors then return end



    local now = tick()



    -- Initialize spatial partitioning fields (once)

    if not self._sectors then

        self._sectors = {}       -- Sector grid

        self._sectorSize = 512   -- Sector size (studs)

        self._playerSector = nil -- Current player sector

        self._targetQueue = {}   -- Top 3 pre-cached targets

        self._lastSectorUpdate = 0

    end



    -- Optimize Map Detection (Every 10s)

    if not self._lastMapUpdate or (now - self._lastMapUpdate > 10) then

        Config.CurrentMapType = DetectMapType()

        self._lastMapUpdate = now

    end



    -- 1. Refresh Entity Lists (Every 0.5s - throttled for FPS)

    if not self._lastListUpdate or (now - self._lastListUpdate > 0.5) then

        if #self.Enemies > 0 then table.clear(self.Enemies) end

        if #self.Teammates > 0 then table.clear(self.Teammates) end

        if #self.Vehicles > 0 then table.clear(self.Vehicles) end



        -- SPATIAL PARTITIONING: Clear and rebuild sectors

        table.clear(self._sectors)



        local localSquad = nil

        if ClientService and type(ClientService.LocalClient) == "table" then

            localSquad = ClientService.LocalClient.Squad

        end



        local localPlayer = LocalPlayer



        for _, actor in pairs(ReplicatorService.Actors) do

            if IsActorActuallyDead(actor) or not actor.Character then continue end

            if actor.Owner == localPlayer then continue end



            actor._entityType = GetEntityType(actor)

            actor._isTeammate = IsTeammate(actor, localSquad)



            -- OPTIMIZATION: Cache visibility check (runs every 0.5s)

            if ShouldShowEntity then

                actor._cachedShow = ShouldShowEntity(actor, actor._entityType, localSquad)

            else

                actor._cachedShow = true

            end



            if actor._isTeammate then

                table.insert(self.Teammates, actor)

            else

                table.insert(self.Enemies, actor)



                -- SPATIAL PARTITIONING: Bucket enemy into sector

                -- OPTIMIZATION: actor.Position is engine-interpolated; actor.RootPart is direct ref (no FindFirstChild)

                local actorPos = actor.Position or (actor.RootPart and actor.RootPart.Position)

                if actorPos then

                    local sectorKey = GetSectorKey(actorPos, self._sectorSize)

                    if not self._sectors[sectorKey] then

                        self._sectors[sectorKey] = {}

                    end

                    table.insert(self._sectors[sectorKey], actor)

                end

            end

        end



        if VehicleService and VehicleService.Vehicles then

            for _, veh in pairs(VehicleService.Vehicles) do

                table.insert(self.Vehicles, veh)

            end

        end



        -- SPATIAL PARTITIONING: Update player sector

        local playerPos = Camera.CFrame.Position

        self._playerSector = GetSectorKey(playerPos, self._sectorSize)



        self._lastListUpdate = now

        self.Initialized = true

    end



    -- 2. Update Combat Targets (OPTIMIZED)

    -- Silent Aim: Balanced refresh (0.05s = 20Hz)

    if self.Initialized and (not self._lastTargetUpdate or (now - self._lastTargetUpdate > 0.05)) then

        if GetCombatTarget then

            if Config.SilentAim or Config.ShowPrediction then

                local target, actor = GetCombatTarget("SilentAim")

                self.SelectedTarget_SA = target

                self.SelectedActor_SA = actor

            else

                self.SelectedTarget_SA = nil

                self.SelectedActor_SA = nil

            end

        end

        self._lastTargetUpdate = now

    end



    -- Ragebot: Ultra-High-Speed Target Acquisition (60Hz = 0.016s)

    -- Ragebot: Direct scan (persistence handled inside GetCombatTarget)

    if self.Initialized and Config.RageMode and GetCombatTarget then

        local updateInterval = 0.016 -- 60Hz update rate (was 30Hz)



        if not self._lastRageUpdate or (now - self._lastRageUpdate > updateInterval) then

            local target, actor = GetCombatTarget("Rage")

            self.SelectedTarget_RB = target

            self.SelectedActor_RB = actor

            self._lastRageUpdate = now

        end

    elseif not Config.RageMode then

        self.SelectedTarget_RB = nil

        self.SelectedActor_RB = nil

    end

end



--[[

    WALLCHECK LOGIC (Optimized with Penetration)

]]

local CollectionService = game:GetService("CollectionService")



local function IsIgnored(part)

    if not part then return false end

    -- CharacterCast collision group handles most filtering (debris, vehicles, particles).

    -- Only check transparency and glass name as edge cases.

    if part.Transparency > 0.9 then return true end

    if part.Name == "Glass" or part.Name == "Window" then return true end

    return false

end



local VisCache = {}

local VisRayParams = RaycastParams.new()

VisRayParams.FilterType = Enum.RaycastFilterType.Exclude

VisRayParams.CollisionGroup = "CharacterCast" -- Match game's own raycast group (ActorClass.lua:1114)

VisRayParams.IgnoreWater = true



-- Cache Cleanup (Prevent Memory Bloat)

local LastVisCacheCleanup = 0



-- Pre-allocate filter table to reduce garbage collection

local VisFilterTable = table.create(16)



-- Pre-calculate hitbox offsets to avoid per-raycast allocations

local HitboxOffsets = {

    -- Axial points (6)

    Vector3_new(1, 0, 0), Vector3_new(-1, 0, 0),

    Vector3_new(0, 1, 0), Vector3_new(0, -1, 0),

    Vector3_new(0, 0, 1), Vector3_new(0, 0, -1),

    -- Diagonal Octants (8)

    Vector3_new(1, 1, 1), Vector3_new(1, 1, -1),

    Vector3_new(1, -1, 1), Vector3_new(1, -1, -1),

    Vector3_new(-1, 1, 1), Vector3_new(-1, 1, -1),

    Vector3_new(-1, -1, 1), Vector3_new(-1, -1, -1)

}



local function IsVisible(targetActor, targetPart, origin, isPriority)

    if not targetPart or not targetPart.Parent then return false end



    local now = getgenv().LastTime or tick()

    local uid = targetActor.UID



    -- CACHE CLEANUP: Prevent memory bloat (every 2 seconds)

    if now - LastVisCacheCleanup > 2 then

        for k, v in pairs(VisCache) do

            if now - v.last > 5 then -- Remove entries older than 5s

                VisCache[k] = nil

            end

        end

        LastVisCacheCleanup = now

    end



    -- PRIORITY MODE: Active target gets fresh raycast, skips cache

    if not isPriority then

        -- Non-priority: Use cache (for search candidates)

        if VisCache[uid] and (now - VisCache[uid].last < 0.05) then

            return VisCache[uid].result

        end

    end



    -- RAYCAST BUDGET CHECK (priority targets don't consume budget)

    if not isPriority then

        if getgenv().RaycastBudget <= 0 then

            return false -- Skip visibility check if budget exhausted

        end

        getgenv().RaycastBudget = getgenv().RaycastBudget - 1

    end



    local originPos = origin or getgenv().LastCamCF.Position

    local direction = (targetPart.Position - originPos)

    local dist = targetActor.LOD_Distance or direction.Magnitude



    -- RESTORED ENGINE OPTIMIZATION

    if not targetActor.ViewportOnScreen and (dist > 100) and not Config.RageMode then

        VisCache[uid] = { result = false, last = now }

        return false

    end



    local character = targetPart.Parent

    

    -- RESTORED FILTER TABLE UPDATE

    table.clear(VisFilterTable)

    table.insert(VisFilterTable, Camera)

    if LocalPlayer.Character then table.insert(VisFilterTable, LocalPlayer.Character) end

    table.insert(VisFilterTable, character) 



    if ReplicatorService and ReplicatorService.LocalActor then

        local myActor = ReplicatorService.LocalActor

        if myActor.Seat and myActor.Seat.Model then

            table.insert(VisFilterTable, myActor.Seat.Model)

        end

    end

    -- OPTIMIZATION: Cache filter result for 5 frames

    VisRayParams.FilterDescendantsInstances = VisFilterTable



    local function CheckPointVisibleInternal(point)

        local dir = (point - originPos)

        local result = Workspace:Raycast(originPos, dir, VisRayParams)

        if not result then return true end

        if result.Instance:IsDescendantOf(character) then return true end



        local penetrations = 0

        local dirUnit = dir.Unit

        

        -- Use persistent secondary params to avoid per-call allocation

        if not getgenv()._VisRayParams2 then

            getgenv()._VisRayParams2 = RaycastParams.new()

            getgenv()._VisRayParams2.FilterType = Enum.RaycastFilterType.Exclude

            getgenv()._VisRayParams2.CollisionGroup = "CharacterCast"

            getgenv()._VisRayParams2.IgnoreWater = true

        end

        local params2 = getgenv()._VisRayParams2

        local filter2 = getgenv()._VisFilterTable2 or table.create(20)

        getgenv()._VisFilterTable2 = filter2

        table.clear(filter2)

        for i=1, #VisFilterTable do filter2[i] = VisFilterTable[i] end

        params2.FilterDescendantsInstances = filter2



        local curRes = result

        while penetrations < 2 do

            local hitPart = curRes.Instance

            if hitPart:IsDescendantOf(character) then return true end

            if IsIgnored(hitPart) then

                table.insert(filter2, hitPart)

                params2.FilterDescendantsInstances = filter2

                penetrations = penetrations + 1

                curRes = Workspace:Raycast(curRes.Position + (dirUnit * 0.1), dir - (curRes.Position - originPos), params2)

                if not curRes then return true end

            else

                return false

            end

        end

        return false

    end



    local isVisible = false

    local visiblePoint = targetPart.Position



    if CheckPointVisibleInternal(visiblePoint) then

        isVisible = true

    elseif Config.HitboxExpander then

        local sz = (Config.HitboxSize or 4) * 0.45

        for i = 1, #HitboxOffsets do

            local p = visiblePoint + (HitboxOffsets[i] * sz)

            if CheckPointVisibleInternal(p) then

                isVisible = true

                visiblePoint = p

                break

            end

        end

    end



    VisCache[uid] = { result = isVisible, last = now, point = visiblePoint }

    if isVisible then targetActor._visiblePoint = visiblePoint end

    return isVisible

end



--[[

    COMBAT TARGETING (OPTIMIZED)

    High-performance target acquisition with pre-filtering and engine-calculated distances

]]

GetCombatTarget = function(mode)

    -- Reset Raycast Budget (Max 20 per frame for ragebot responsiveness)

    getgenv().RaycastBudget = (mode == "Rage") and 20 or 12



    local cfg = Config

    local targetPartName = (mode == "Rage") and cfg.Rage_TargetPart or cfg.SilentAimTargetPart or "Head"



    local useFOV = (mode == "SilentAim")

    local fovLimit = cfg.SilentAimFOV

    local fovLimitSq = fovLimit * fovLimit

    local maxDistance = (mode == "Rage") and cfg.Rage_Range or cfg.MaxDistance or 2000

    local maxDistSq = maxDistance * maxDistance -- Pre-calculate for faster comparison



    local checkOrigin = Camera.CFrame.Position

    local muzzle = GetMuzzlePosition()

    if muzzle then checkOrigin = muzzle end



    if not ActorManager.Enemies or #ActorManager.Enemies == 0 then return nil, nil end



    -- OPTIMIZATION: Validate Active Target First (1 Priority Raycast)

    -- If current target still valid, return immediately (saves 10-19 raycasts!)

    if mode == "Rage" and ActorManager.SelectedActor_RB then

        local activeActor = ActorManager.SelectedActor_RB



        -- Check if this target was marked for ignore (e.g. single-shot switching)

        local ignored = ActorManager._ignoredTargets

        if ignored and ignored[activeActor.UID] and tick() < ignored[activeActor.UID] then

            ActorManager.SelectedActor_RB = nil

            ActorManager.SelectedTarget_RB = nil

            -- Quick validation checks

        elseif activeActor.Alive and activeActor.Character then

            local dist = activeActor.LOD_Distance

            if not dist or dist <= 0 then

                dist = (activeActor.Position - checkOrigin).Magnitude

            end



            -- Still in range?

            if dist <= maxDistance then

                local targetPart = activeActor.Parts[targetPartName]



                -- PRIORITY VISIBILITY CHECK (doesn't consume budget)

                if targetPart and IsVisible(activeActor, targetPart, checkOrigin, true) then

                    -- Active target still valid! Return early

                    return targetPart, activeActor

                end

            end



            -- Active target invalid, clear it and search for new one

            ActorManager.SelectedActor_RB = nil

            ActorManager.SelectedTarget_RB = nil

        else

            -- Active target invalid, clear it and search for new one

            ActorManager.SelectedActor_RB = nil

            ActorManager.SelectedTarget_RB = nil

        end

    end

    -- [MERT FIX] Silent Aim: Target Stickiness System
    -- Locks onto current target while they stay within 1.5x FOV, preventing jittery target switching
    if mode == "SilentAim" and ActorManager.SelectedActor_SA then
        local stickActor = ActorManager.SelectedActor_SA
        local stickTarget = ActorManager.SelectedTarget_SA

        -- Validate: actor must still be alive
        if not IsActorActuallyDead(stickActor) and stickActor.Character and stickTarget and stickTarget.Parent then
            local dist = stickActor.LOD_Distance or (stickActor.Position - checkOrigin).Magnitude

            if dist <= maxDistance then
                -- Use exact FOV for sticky check so we don't shoot outside FOV circle
                local stickyFOV = (cfg.SilentAimFOV or 100)
                local targetPos3D = stickTarget.Position
                local screenPos, onScreen = Camera:WorldToViewportPoint(targetPos3D)

                if onScreen then
                    local mouse = UserInputService:GetMouseLocation()
                    local dx = screenPos.X - mouse.X
                    local dy = screenPos.Y - mouse.Y
                    local distSq = dx * dx + dy * dy

                    -- Still inside sticky FOV zone AND visible? Keep the lock.
                    if distSq <= (stickyFOV * stickyFOV) then
                        if IsVisible(stickActor, stickTarget, checkOrigin, true) then
                            return stickTarget, stickActor -- Keep existing target lock
                        end
                    end
                end
            end
        end

        -- Lock expired — clear it so we can search for a new target
        ActorManager.SelectedActor_SA = nil
        ActorManager.SelectedTarget_SA = nil
    end

    -- PRE-ALLOCATE: Reuse candidates table to reduce GC pressure



    local candidates = getgenv()._targetCandidates or table.create(64)

    table.clear(candidates)

    getgenv()._targetCandidates = candidates



    local now = tick()

    local ignoredTargets = ActorManager._ignoredTargets

    local mouse, mouseX, mouseY



    if useFOV then

        mouse = UserInputService:GetMouseLocation()

        mouseX, mouseY = mouse.X, mouse.Y

    end





    -- SPATIAL PARTITIONING: Get enemies from nearby sectors only



    local enemyList

    if mode == "Rage" and ActorManager._sectors and ActorManager._playerSector then

        -- Reuse cached table to avoid GC

        enemyList = getgenv()._cachedEnemyList or table.create(100)

        table.clear(enemyList)

        getgenv()._cachedEnemyList = enemyList



        local px, pz = DecodeSectorKey(ActorManager._playerSector)

        for dx = -1, 1 do

            for dz = -1, 1 do

                local key = (px + dx) * 65536 + (pz + dz)

                local sectorEnemies = ActorManager._sectors[key]

                if sectorEnemies then

                    for _, actor in ipairs(sectorEnemies) do

                        table.insert(enemyList, actor)

                    end

                end

            end

        end



        -- Fallback: If no enemies in nearby sectors, use full list

        if #enemyList == 0 then

            enemyList = ActorManager.Enemies

        end

    else

        -- Silent Aim or no sectors: use full list

        enemyList = ActorManager.Enemies

    end



    -- FAST PATH: Single enemy in Rage mode = skip candidate table entirely

    if mode == "Rage" and #enemyList == 1 then

        local actor = enemyList[1]

        if actor.Alive and actor.Character and actor._cachedShow ~= false then

            local dist = actor.LOD_Distance or 9999

            if dist <= maxDistance then

                local part = actor.Parts and actor.Parts[targetPartName]

                if not part then part = actor.Parts and (actor.Parts.Head or actor.Parts.UpperTorso) end

                if part and part.Parent and IsVisible(actor, part, checkOrigin) then

                    return part, actor

                end

            end

        end

    end



    -- 1. FAST PRE-FILTER & SCORE (Single pass, minimal allocations)

    for _, actor in pairs(enemyList) do

        -- Quick alive check

        if not actor.Alive then continue end



        -- Ignored targets check (with expiry cleanup)

        if ignoredTargets then

            local ignoreUntil = ignoredTargets[actor.UID]

            if ignoreUntil then

                if now < ignoreUntil then

                    continue

                else

                    ignoredTargets[actor.UID] = nil -- Expired - cleanup

                    -- Also reset damage tracking for this target

                    if ActorManager._rageDmg then ActorManager._rageDmg[actor.UID] = nil end

                    if ActorManager._rageShots then ActorManager._rageShots[actor.UID] = nil end

                    if ActorManager._rageLast then ActorManager._rageLast[actor.UID] = nil end

                end

            end

        end



        local char = actor.Character

        if not char then continue end



        -- OPTIMIZATION: Use engine-calculated LOD_Distance if available (saves magnitude calc!)

        local dist = actor.LOD_Distance

        if not dist or dist <= 0 then

            -- Fallback: Calculate manually only if engine value missing

            -- OPTIMIZATION: Use actor.Position or actor.RootPart (direct engine ref, no FindFirstChild)

            local actorPos = actor.Position

            if not actorPos then

                local root = actor.RootPart or (actor.Parts and actor.Parts.Head)

                if not root then continue end

                actorPos = root.Position

            end

            local delta = actorPos - checkOrigin

            dist = math.sqrt(delta.X * delta.X + delta.Y * delta.Y + delta.Z * delta.Z)

        end



        -- Distance pre-filter (using squared distance when possible)

        if dist > maxDistance then continue end



        -- RAGEBOT: 360Â° TARGET ACQUISITION (No FOV/Viewport restrictions!)

        -- For Ragebot (mode == "Rage"), useFOV = false, so targets in ALL directions

        -- are valid. Only limited by Rage_Range distance, not screen position.

        local score = dist -- Default: closest first (Ragebot)



        -- FOV filtering for Silent Aim

        if useFOV then

            -- Early viewport check using engine data

            if not actor.ViewportOnScreen and dist > 50 then continue end



            -- OPTIMIZATION: Use engine-interpolated Position instead of PrimaryPart lookup

            local rootPos = actor.Position

            if not rootPos then

                local root = char.PrimaryPart or char:FindFirstChild("Head")

                if not root then continue end

                rootPos = root.Position

            end



            local pos, onScreen = Camera:WorldToViewportPoint(rootPos)

            if not onScreen then continue end



            -- Screen distance (squared for performance)

            local dx = pos.X - mouseX

            local dy = pos.Y - mouseY

            local screenDistSq = (dx * dx) + (dy * dy)



            if screenDistSq > fovLimitSq then continue end

            score = screenDistSq -- Silent Aim: closest to crosshair

        end



        -- Store candidate with pre-calculated score

        table.insert(candidates, {

            actor = actor,

            score = score,

            char = char,

            dist = dist -- Cache for later use

        })

    end



    -- Early exit if no candidates

    if #candidates == 0 then return nil, nil end



    -- 2. OPTIMIZED SORT: Use ascending score (best target = lowest score)
    -- Fully sort candidates so if the best target is occluded, it correctly falls back to the 2nd best
    table.sort(candidates, function(a, b)
        return a.score < b.score
    end)

    -- 3. VISIBILITY CHECK (Budget-aware, prioritized order)
    local fallbackParts = {"Head", "UpperTorso", "LowerTorso"}

    for i = 1, #candidates do
        local candle = candidates[i]
        local actor = candle.actor
        local char = candle.char

        -- Dynamic Bone Targeting: If preferred part is hidden, try torso
        local partsToCheck = { targetPartName }
        for _, fb in ipairs(fallbackParts) do
            if fb ~= targetPartName then table.insert(partsToCheck, fb) end
        end

        for _, partName in ipairs(partsToCheck) do
            local part = actor.Parts and actor.Parts[partName]
            if not part then part = char:FindFirstChild(partName) end

            if part and part.Parent then
                -- Perform visibility check (uses raycast budget)
                if IsVisible(actor, part, checkOrigin) then
                    -- [MERT FIX] Cache new target for Sticky Target system
                    if mode == "SilentAim" then
                        ActorManager.SelectedActor_SA = actor
                        ActorManager.SelectedTarget_SA = part
                    end
                    return part, actor -- SUCCESS: Return first visible part
                end

                -- Budget exhausted - stop checking
                if getgenv().RaycastBudget <= 0 then break end
            end
        end
        
        if getgenv().RaycastBudget <= 0 then break end
    end



    return nil, nil -- No valid target found

end



-- Hook Weapon Controller (GetMuzzleCFrame) - Server Leniency Method

local function HookController(controller)

    if type(controller) ~= "table" then return end

    if controller._hookedMuzzle then return end



    -- Safety: Skip turrets (they have their own class hook)

    if rawget(controller, "_turret") or rawget(controller, "_config") then

        return

    end

    if controller.GetMuzzleCFrame then

        controller._originalMuzzle = controller.GetMuzzleCFrame



        controller.GetMuzzleCFrame = function(self, ...)

            local muzzleCF, v2, v3 = controller._originalMuzzle(self, ...)

            if not muzzleCF then return muzzleCF, v2, v3 end

            -- Determine if this controller is a vehicle weapon

            if self._actor and not self._actor.IsLocalPlayer and ReplicatorService.LocalActor.Character then

                local hum = ReplicatorService.LocalActor.Character:FindFirstChild("male")

                if hum and hum.SeatPart and hum.SeatPart.Parent == self._actor then

                    isVehicleWeapon = true

                end

            end



            local isYours = false

            if self._actor and self._actor.IsLocalPlayer then

                isYours = true

            end



            -- 2. Check if it's a vehicle we are driving/gunning

            if not isYours and isVehicleWeapon then

                isYours = true

            end



            -- Combat Targeting (Ragebot priority, then Silent Aim)

            local combatTarget = ActorManager.SelectedTarget_RB or ActorManager.SelectedTarget_SA

            local targetActor = ActorManager.SelectedActor_RB or ActorManager.SelectedActor_SA



            -- Return muzzle CFrame unmodified to prevent "Auto Shoot" visual snapping

            -- Silent Aim Logic moved to BulletService.Discharge

            -- UPDATE: We MUST modify this for the Server to accept the hit (OriginCFrame validation)

            -- [MERT FIX] Silent Aim: FOV + Visibility gate at fire time
            -- Only redirect aim if target is STILL inside FOV AND visible (not behind a wall)
            if Config.SilentAim and not Config.RageMode and combatTarget and targetActor then
                -- 1. FOV check at moment of firing (target may have moved out of FOV since last cache update)
                local targetPos3D = (targetActor.Parts and targetActor.Parts.Head and targetActor.Parts.Head.Position)
                    or (targetActor.Position)
                if targetPos3D then
                    local screenPos, onScreen = Camera:WorldToViewportPoint(targetPos3D)
                    if onScreen then
                        local mouse = UserInputService:GetMouseLocation()
                        local dx = screenPos.X - mouse.X
                        local dy = screenPos.Y - mouse.Y
                        local fovLimit = Config.SilentAimFOV or 100
                        -- If outside FOV circle, cancel silent aim this shot
                        if (dx * dx + dy * dy) > (fovLimit * fovLimit) then
                            combatTarget = nil
                            targetActor = nil
                        end
                    else
                        -- Not on screen = outside FOV, cancel
                        combatTarget = nil
                        targetActor = nil
                    end
                end

                -- 2. Visibility check at moment of firing (target behind wall?)
                if combatTarget and targetActor then
                    local checkPos = muzzleCF and muzzleCF.Position or Camera.CFrame.Position
                    if not IsVisible(targetActor, combatTarget, checkPos, true) then
                        combatTarget = nil
                        targetActor = nil
                    end
                end
            end

            if (Config.SilentAim or Config.RageMode) and combatTarget and targetActor then

                local targetHead = combatTarget



                -- Calculate Aim Position with Prediction

                local targetPos = targetActor._visiblePoint or targetHead.Position

                local gravity = 32.2 -- Hardcoded engine gravity

                local muzzle_velocity = 2000



                if self._firearm and self._firearm.Tune then

                    local caliberName = self._firearm.Tune.Caliber

                    local barrelLen = self._firearm.Tune.Barrel



                    if BulletService and BulletService.GetInfo then

                        local success, vel = pcall(function() return BulletService:GetInfo(caliberName, barrelLen) end)

                        if success and vel then muzzle_velocity = vel end

                    end

                end



                local targetVelocity = Vector3_new(0, 0, 0)
                if targetActor._oldDirection then
                    targetVelocity = targetActor._oldDirection * 60
                elseif targetHead.Parent and targetHead.Parent.PrimaryPart then
                    targetVelocity = targetHead.Parent.PrimaryPart.AssemblyLinearVelocity or Vector3_new(0, 0, 0)
                end
                
                -- HÄ±z limiti ve NaN kontrolÃ¼ (AraÃ§lar/Helikopterler iÃ§in)
                if typeof(targetVelocity) == "Vector3" then
                    if targetVelocity.X ~= targetVelocity.X then targetVelocity = Vector3_new(0, 0, 0) end
                    if targetVelocity.Magnitude > 300 then
                        targetVelocity = targetVelocity.Unit * 300
                    end
                else
                    targetVelocity = Vector3_new(0, 0, 0)
                end



                if not muzzleCF then return end -- Safety check



                local aimPos, travelTime = SolveLead(muzzleCF.Position, targetPos, targetVelocity, muzzle_velocity,
                    gravity)

                -- [MERT FIX] Humanized Hit Chance (Dynamic Spread)
                -- If hit chance fails, apply a realistic offset based on distance so the bullet narrowly misses
                if Config.SilentAim and not Config.RageMode then
                    local hitChance = Config.SilentAimHitChance or 100
                    if math.random(1, 100) > hitChance then
                        local dist = (muzzleCF.Position - aimPos).Magnitude
                        -- Calculate spread: max 1.5 studs offset per 100 studs of distance
                        local missFactor = (100 - hitChance) / 100
                        local maxSpread = (dist / 100) * 1.5 * missFactor
                        
                        -- Avoid spreading too much at point blank
                        if maxSpread > 0.1 then
                            aimPos = aimPos + Vector3_new(
                                (math.random() - 0.5) * maxSpread,
                                (math.random() - 0.5) * maxSpread,
                                (math.random() - 0.5) * maxSpread
                            )
                        end
                    end
                end

                -- Manual compensation removed to avoid double-application.
                -- SolveLead handles Config.BulletDrop internally.





                -- Calculate Zeroing Angle to subtract (ONLY IF ADS IS TRUE)

                local zeroAngle = 0

                local actor = self._actor



                -- Game only applies zeroing when ADS is true

                if actor and actor.ADS and actor.ViewModel and actor.ViewModel.Zero then

                    local zeroData = actor.ViewModel.Zero

                    local zeroDist = zeroData

                    if type(zeroData) == "table" then zeroDist = zeroData[4] end



                    if zeroDist then

                        local v177 = zeroDist * gravity / (muzzle_velocity ^ 2)

                        zeroAngle = math.asin(math.clamp(v177, -1, 1)) * 0.5

                    end

                end



                muzzleCF = CFrame_new(muzzleCF.Position, aimPos) * CFrame.Angles(-zeroAngle, 0, 0)

            end



            return muzzleCF, v2, v3

        end

        controller._hookedMuzzle = true

    elseif controller._discharge then

        -- Hooking _discharge logic

        controller._originalDischarge = controller._discharge



        controller._discharge = function(self, active)

            self._firing = active -- Set firing flag

            local r = controller._originalDischarge(self, active)



            -- Trigger Combat Logic on discharge

            if active and (Config.SilentAim or Config.RageMode) then

                local targetHead = ActorManager.SelectedTarget_RB or ActorManager.SelectedTarget_SA

                local targetActor = ActorManager.SelectedActor_RB or ActorManager.SelectedActor_SA



                -- [MERT FIX] _discharge FOV + Visibility Check
                if Config.SilentAim and not Config.RageMode and targetHead and targetActor then
                    local targetPos3D = (targetActor.Parts and targetActor.Parts.Head and targetActor.Parts.Head.Position) or (targetActor.Position)
                    if targetPos3D then
                        local screenPos, onScreen = Camera:WorldToViewportPoint(targetPos3D)
                        if onScreen then
                            local mouse = UserInputService:GetMouseLocation()
                            local dx = screenPos.X - mouse.X
                            local dy = screenPos.Y - mouse.Y
                            local fovLimit = Config.SilentAimFOV or 100
                            if (dx * dx + dy * dy) > (fovLimit * fovLimit) then
                                targetHead = nil
                                targetActor = nil
                            end
                        else
                            targetHead = nil
                            targetActor = nil
                        end
                    end
                    
                    if targetHead and targetActor then
                        local checkPos = Camera.CFrame.Position
                        if not IsVisible(targetActor, targetHead, checkPos, true) then
                            targetHead = nil
                            targetActor = nil
                        end
                    end
                end

                if targetHead and targetHead.Parent then

                    -- Calculate Direction to Target

                    local muzzlePos = GetMuzzlePosition() -- Use shared function

                    if not muzzlePos then muzzlePos = Camera.CFrame.Position end



                    local targetPos = targetHead.Position

                    local direction = (targetPos - muzzlePos).Unit



                    -- Force Character Orientation (Consistent with ActorClass Yaw calculation)

                    if self._actor and self._actor.IsLocalPlayer and ReplicatorService.LocalActor.Character then

                        local root = ReplicatorService.LocalActor.Character.PrimaryPart

                        local myActor = ReplicatorService.LocalActor

                        if root then

                            local yaw = math.atan2(direction.X, direction.Z)

                            myActor.Orientation = yaw

                            myActor.GoalOrientation = yaw

                            myActor.CameraX = yaw

                        end

                    end

                end

            end



            -- RECOIL EFFECT (Visual Shake)

            if active and Config.RecoilEffect_Enabled then

                task.spawn(function()

                    for i = 1, 3 do

                        Camera.CFrame = Camera.CFrame * CFrame.Angles(math.rad(math.random(-1, 1)*0.1), math.rad(math.random(-1, 1)*0.1), 0)

                        task.wait(0.01)

                    end

                end)

            end



            return r

        end

    end

end



-- Turret Crosshair & Zoom Logic
-- These declarations are moved to the UI section below.



-- Mouse Wheel Scroll for Global Turret Zoom
-- This function is moved to the UI section below.



-- This function is moved to the UI section below.



-- FOV Circle

local FOVCircle = Drawing.new("Circle")

FOVCircle.Thickness = 1

FOVCircle.NumSides = 60

FOVCircle.Radius = Config.SilentAimFOV

FOVCircle.Filled = false

FOVCircle.Visible = false

FOVCircle.Color = Color3.fromRGB(255, 255, 255)



-- Velocity Cache

getgenv().VelocityCache = {}



-- Prediction Visualizer (Global)

getgenv().PredictionDot = Drawing.new("Circle")

getgenv().PredictionDot.Thickness = 1

getgenv().PredictionDot.NumSides = 10

getgenv().PredictionDot.Radius = 2

getgenv().PredictionDot.Filled = true

getgenv().PredictionDot.Visible = false

getgenv().PredictionDot.Color = Config.PredictionColor



-- Weapon Info Overlay

local WeaponInfoText = Drawing.new("Text")

WeaponInfoText.Size = 22

WeaponInfoText.Center = false

WeaponInfoText.Outline = true

WeaponInfoText.Color = Color3.fromRGB(220, 200, 255)

WeaponInfoText.Position = Vector2.new(50, Camera.ViewportSize.Y - 100)

WeaponInfoText.Visible = false

WeaponInfoText.Font = 3



--[[

    ENGINE HOOKS - Character Controller (Fly & Noclip)

    Hooks into the game's update loop instead of running an external loop.

]]



-- Speed Controller (SpeedPenalty Method - More Compatible)

local SpeedControllerConnection = nil



function EnableSpeedController()

    if SpeedControllerConnection then return end



    SpeedControllerConnection = RunService.Heartbeat:Connect(function()

        -- Find local actor

        local actor = nil

        if ReplicatorService and ReplicatorService.LocalActor then

            actor = ReplicatorService.LocalActor

        elseif ControllerService and ControllerService.Controller and ControllerService.Controller._localActor then

            actor = ControllerService.Controller._localActor

        end



        if not actor then return end



        local walkEnabled = Config.Character_WalkSpeedEnabled

        local sprintEnabled = Config.Character_SprintSpeedEnabled



        if not walkEnabled and not sprintEnabled then

            -- Reset to normal

            if actor.SpeedPenalty then

                actor.SpeedPenalty = nil

            end

            return

        end



        -- Detect if sprinting (from CharacterController: IsSprinting property)

        local controller = ControllerService and ControllerService.Controller

        local isSprinting = controller and controller.IsSprinting



        local multiplier = 1



        if sprintEnabled and isSprinting then

            -- Sprint speed hack: User wants X studs/s, base is 16.8

            multiplier = (Config.Character_SprintSpeed or 25) / 16.8

        elseif walkEnabled and not isSprinting then

            -- Walk speed hack: User wants X studs/s, base is 12

            multiplier = (Config.Character_WalkSpeed or 16) / 12

        end



        -- Apply as SpeedPenalty (>1 = faster, <1 = slower)

        actor.SpeedPenalty = multiplier

    end)



    table.insert(getgenv().BlackhawkESP_Connections, SpeedControllerConnection)

    -- print removed

end



--[[

    AGGRESSIVE GUN MODS & HOOKS

    Implementation based on reference script for maximum reliability

]]

local function ApplyGunMods(firearm, force)

    if not firearm or not firearm._firearm or not firearm._firearm.Tune then return end



    local now = getgenv().LastTime or tick()

    -- Fast Response: Removed throttle per user request

    firearm._LastTuneTime = now



    local tune = firearm._firearm.Tune

    local caliber = firearm._caliber

    local override = firearm._firearm.OverrideTune



    -- Initialize Backups (Tune)

    if not tune._Originals then

        tune._Originals = {

            Recoil_X = tune.Recoil_X,

            Recoil_Z = tune.Recoil_Z,

            Recoil_Camera = tune.Recoil_Camera,

            RecoilForce_Tap = tune.RecoilForce_Tap,

            RecoilForce_Impulse = tune.RecoilForce_Impulse,

            Recoil_Range = tune.Recoil_Range,

            Recoil_KickBack = tune.Recoil_KickBack,

            Barrel_Spread = tune.Barrel_Spread,

            Spread = tune.Spread,

            RPM = tune.RPM,

            Firemodes = tune.Firemodes,

            Bolt = tune.Bolt,

            Bolt_Action_Pause = tune.Bolt_Action_Pause,

            -- Add potential movement spread keys if they exist

            Hipfire_Spread = tune.Hipfire_Spread,

            Move_Spread = tune.Move_Spread,

            Jump_Spread = tune.Jump_Spread,

            Damage = tune.Damage,

            Bullets = tune.Bullets

        }

    end



    -- Initialize Backups (Caliber)

    if caliber and not caliber._Originals then

        caliber._Originals = {

            Spread = caliber.Spread,

            RecoilForce = caliber.RecoilForce

        }

    end



    if override and not override._Originals then

        override._Originals = {

            Recoil_X = override.Recoil_X,

            Recoil_Z = override.Recoil_Z,

            Recoil_Camera = override.Recoil_Camera,

            RecoilForce_Tap = override.RecoilForce_Tap,

            RecoilForce_Impulse = override.RecoilForce_Impulse,

            Spread = override.Spread,

            RPM = override.RPM,

            Firemodes = override.Firemodes,

            Damage = override.Damage

        }

    end



    -- Apply Modifications

    if Config.NoRecoil then

        tune.Recoil_X = 0; tune.Recoil_Y = 0; tune.Recoil_Z = 0; tune.Recoil_Camera = 0

        tune.RecoilForce_Tap = 0; tune.RecoilForce_Impulse = 0

        tune.Recoil_Range = Vector2_new(0, 0); tune.Recoil_KickBack = 0

        tune.FocusRecoil_X = 0; tune.FocusRecoil_Z = 0; tune.FocusRecoil_Camera = 0

        tune.Sway = 0; tune.Sway_ADS = 0



        if tune.Bolt then tune.Bolt = 0 end

        if tune.Bolt_Action_Pause then tune.Bolt_Action_Pause = 0 end



        if override then

            override.Recoil_X = 0; override.Recoil_Y = 0; override.Recoil_Z = 0; override.Recoil_Camera = 0

            override.RecoilForce_Tap = 0; override.RecoilForce_Impulse = 0

            override.Sway = 0; override.Sway_ADS = 0

        end



        if caliber then

            caliber.RecoilForce = 0

        end

    else

        -- Restore Recoil

        for k, v in pairs(tune._Originals) do if tune[k] ~= nil then tune[k] = v end end

        if override and override._Originals then

            for k, v in pairs(override._Originals) do if override[k] ~= nil then override[k] = v end end

        end

        if caliber and caliber._Originals then

            caliber.RecoilForce = caliber._Originals.RecoilForce

        end

    end



    if Config.NoSpread then

        tune.Barrel_Spread = 0

        tune.Spread = 0

        tune.MinSpread = 0

        tune.MaxSpread = 0

        tune.Hipfire_Spread = 0

        tune.Move_Spread = 0

        tune.Jump_Spread = 0



        if caliber then

            caliber.Spread = 0

        end

        if override then

            override.Spread = 0; override.MinSpread = 0; override.MaxSpread = 0

        end

    else

        tune.Barrel_Spread = tune._Originals.Barrel_Spread

        tune.Spread = tune._Originals.Spread

        tune.MinSpread = tune._Originals.MinSpread or 0

        tune.MaxSpread = tune._Originals.MaxSpread or 0

        tune.Hipfire_Spread = tune._Originals.Hipfire_Spread

        tune.Move_Spread = tune._Originals.Move_Spread

        tune.Jump_Spread = tune._Originals.Jump_Spread



        if caliber and caliber._Originals then

            caliber.Spread = caliber._Originals.Spread

        end

        if override and override._Originals then

            override.Spread = override._Originals.Spread

        end

    end



    if Config.CustomRPM and Config.RPMValue then

        tune.RPM = Config.RPMValue

        if override then override.RPM = Config.RPMValue end

    else

        tune.RPM = tune._Originals and tune._Originals.RPM or (tune.RPM)

        if override and override._Originals and override._Originals.RPM then

            override.RPM = override._Originals.RPM

        end

    end



    if Config.UnlockFiremodes then

        local hasAuto = false

        if tune.Firemodes then

            for _, m in pairs(tune.Firemodes) do if m == 2 then hasAuto = true end end

            if not hasAuto then table.insert(tune.Firemodes, 2) end

        end



        if override and override.Firemodes then

            local hasAutoOverride = false

            for _, m in pairs(override.Firemodes) do if m == 2 then hasAutoOverride = true end end

            if not hasAutoOverride then table.insert(override.Firemodes, 2) end

        end

    end



    -- EFFECTIVE ONE HIT KILL (Server-Safe Method)

    if Config.OneHitKill then

        tune.Damage = 98 -- High but below common sanity check limits (usually 100+)

        tune.Bullets = tune._Originals and tune._Originals.Bullets or 1

        if override then

            override.Damage = 98

            override.Bullets = override._Originals and override._Originals.Bullets or 1

        end



        -- Override CalibersTable for guaranteed NPC kills

        if CalibersTable and firearm._caliber then

            local cal = firearm._caliber

            if not cal._OriginalDamage then

                cal._OriginalDamage = cal.Damage

                cal._OriginalDropoff = cal.Dropoff

                cal._OriginalBullets = cal.Bullets

            end

            cal.Damage = 460 -- Extremely high damage for OHK

            cal.Dropoff = {0, 0, 0, 0}

            cal.Bullets = cal._OriginalBullets or 1 -- Keep original bullet count per user request

        end

    else

        -- Restore Damage

        if tune._Originals and tune._Originals.Damage then

            tune.Damage = tune._Originals.Damage

            tune.Bullets = tune._Originals.Bullets or 1

        end

        if override and override._Originals and override._Originals.Damage then

            override.Damage = override._Originals.Damage

            override.Bullets = override._Originals.Bullets or 1

        end

        if firearm._caliber and firearm._caliber._OriginalDamage then

            firearm._caliber.Damage = firearm._caliber._OriginalDamage

            firearm._caliber.Dropoff = firearm._caliber._OriginalDropoff

            firearm._caliber.Bullets = firearm._caliber._OriginalBullets or 1

        end

    end

end



local function SetupAggressiveHooks()

    -- 1. Hook FirearmInventoryClass (Permanent Instance Capture)

    local fc = FirearmInventoryClass

    if fc and type(fc) == "table" then

        if not fc._OriginalEquip then fc._OriginalEquip = fc.Equip end

        if fc._OriginalEquip then

            fc.Equip = function(self, ...)

                ApplyGunMods(self, true) -- Sets mods whenever weapon is held

                return fc._OriginalEquip(self, ...)

            end

        end



        -- 1.1 Hook Discharge (Aggressive V12)

        if not fc._OriginalDischarge then fc._OriginalDischarge = fc.Discharge end

        if fc._OriginalDischarge then

            fc.Discharge = function(self, ...)

                if Config.InfiniteAmmo then

                    local mag = self._mag

                    if mag then

                        -- Based on Competitive analysis: Discharge() reduces Capacity. We restore it instantly.

                        local maxCap = self._max or 100

                        mag.Capacity = maxCap

                        if mag.MetaData then mag.MetaData.Capacity = maxCap end

                    else

                        -- Handle non-mag weapons (chamber)

                        if self._item and self._item.MetaData then

                            self._item.MetaData.Chamber = true

                        end

                    end

                end

                return fc._OriginalDischarge(self, ...)

            end

        end

    end



    -- 2. Hook Recoiler (Permanent Visual Recoil Override)

    local rc = Recoiler

    if rc and type(rc) == "table" then

        if not rc._OriginalImpulse then rc._OriginalImpulse = rc.Impulse end

        if rc._OriginalImpulse then

            rc.Impulse = function(self, ...)

                if Config.NoRecoil then return end

                return rc._OriginalImpulse(self, ...)

            end

        end



        if rc.GetViewmodelAdjustment and not rc._OriginalVMAdjust then

            rc._OriginalVMAdjust = rc.GetViewmodelAdjustment

            rc.GetViewmodelAdjustment = function(self, ...)

                if Config.NoRecoil then return CFrame_new() end

                return rc._OriginalVMAdjust(self, ...)

            end

        end



        if rc.GetCameraAdjustment and not rc._OriginalCamAdjust then

            rc._OriginalCamAdjust = rc.GetCameraAdjustment

            rc.GetCameraAdjustment = function(self, ...)

                if Config.NoRecoil then return CFrame_new(), 0 end

                return rc._OriginalCamAdjust(self, ...)

            end

        end

    end



    -- 3. Hook CharacterCamera (Universal View Logic: Zoom, Spinbot, Ragebot)

    local cc = CharacterCamera or (shared.Engine and shared.Engine.CharacterCamera) or getgenv().CharacterCamera

    if not cc then

        for _, obj in pairs(getgc(true)) do

            if type(obj) == "table" and rawget(obj, "Update") and rawget(obj, "Render") and rawget(obj, "Watch") then

                cc = obj; CharacterCamera = obj; break

            end

        end

    end



    if cc and cc.Render and not cc._renderHooked then

        cc._originalRender = cc.Render

        cc.Render = function(self, dt, ...)

            local res = { pcall(cc._originalRender, self, dt, ...) }



            -- GLOBAL OVERRIDE FOR TURRET ZOOM

            if Config.Turret_ZoomEnabled then

                local localActor = ReplicatorService and ReplicatorService.LocalActor

                local ctrl = ControllerService and ControllerService.Controller

                local inTurret = (localActor and localActor.Turret) or (ctrl and ctrl._turret)



                if inTurret then

                    workspace.CurrentCamera.FieldOfView = Config.Turret_ZoomValue or 70

                end

            end



            if res[1] then return unpack(res, 2) end

        end

        cc._renderHooked = true

    end



    if cc and cc.Update and not cc._updateHooked then

        cc._originalUpdate = cc.Update

        cc.Update = function(self, delta, dt, ...)

            local res = cc._originalUpdate(self, delta, dt, ...)



            local localActor = (ReplicatorService and ReplicatorService.LocalActor) or (self and self._localActor)



            -- Spinbot Logic

            if Config.Spinbot and localActor and localActor.Alive then

                local speed = (Config.SpinbotSpeed or 10)

                local yaw = (tick() * speed) % (math.pi * 2)

                localActor.Orientation = yaw

                localActor.GoalOrientation = yaw

                localActor.CameraX = yaw

            -- Ragebot Orientation Lock

            elseif Config.RageMode and Config.Rage_LookAt and ActorManager.SelectedTarget_RB then

                local targetPos = ActorManager.SelectedTarget_RB.Position

                local myPos = localActor.Position or Vector3_new()

                local lookDir = (Vector3_new(targetPos.X, myPos.Y, targetPos.Z) - myPos).Unit



                local yaw = math.atan2(lookDir.X, lookDir.Z)

                local pitch = math.atan2(targetPos.Y - myPos.Y, Vector2_new(lookDir.X, lookDir.Z).Magnitude)



                localActor.Orientation = yaw; localActor.GoalOrientation = yaw; localActor.CameraX = yaw; localActor.CameraY = pitch

            end



            return res

        end

        cc._updateHooked = true

    end



    -- 4. Hook CharacterController (True NoClip)

    local d = CharacterController

    if d and d._processNewPosition then

        if not d._originalProcessNewPosition then

            d._originalProcessNewPosition = d._processNewPosition

            d._processNewPosition = function(self, nextPos, ...)

                if Config.CharacterNoclip_Enabled then

                    -- Return desired position, grounded=true, normal=up

                    return nextPos, true, Vector3.new(0, 1, 0)

                end

                return d._originalProcessNewPosition(self, nextPos, ...)

            end

        end

    end

end



local function SetupActorHooks()

    -- Hook ActorClass.Update (Crash Protection for Climbing Bug)

    local ac = ActorClass



    -- Fallback: If not found, check Replicator

    if not ac and ReplicatorService and ReplicatorService.Actors then

        for _, actor in pairs(ReplicatorService.Actors) do

            local mt = getmetatable(actor)

            if mt and mt.__index then

                ac = mt.__index

                ActorClass = ac -- Set Global

                break

            end

        end

    end



    if ac and ac.Update then

        if not ac._originalUpdate then

            ac._originalUpdate = ac.Update



            ac.Update = function(self, ...)

                local args = { pcall(ac._originalUpdate, self, ...) }

                if args[1] then

                    return unpack(args, 2)

                end



                -- CRASH MITIGATION (V4): Return authoritative state if engine failed

                -- This prevents stationary character bug when the game's update loop crashes (e.g. Climbing/Rappelling bug)

                if self.Alive and self.Character and self.CFrame then

                    -- ReplicatorService:Update expects (CFrame, Model/Part) to perform BulkMoveTo

                    return self.CFrame, self.RootPart or self.Character.PrimaryPart or self.Character:FindFirstChild("LowerTorso")

                end

            end

        end

    end



        -- [TRUE GOD MODE] Hook ActorClass.State and direct properties

    if ac then

        ac._originalState = ac._originalState or ac.State

        ac.State = function(self, key, value, ...)

            local isMe = (ReplicatorService and ReplicatorService.LocalActor == self)

            if isMe then

                if Config.Character_HealthRegen then

                    if key == "Dead" or key == "Downed" then return end

                    if (key == "Health" or key == "Hull") then value = 999999 end

                end

                -- Block fall damage / fly death

                if Config.Character_Fly and (key == "Dead" or key == "Downed") then return end

                if Config.Character_Fly and (key == "Health" or key == "Hull") then

                    value = 999999

                end

            end

            return ac._originalState(self, key, value, ...)

        end



        local function lockProperty(obj, prop, val)

            pcall(function()

                if rawget(obj, prop) ~= val then rawset(obj, prop, val) end

            end)

        end



        -- Hook LocalActor's __newindex to intercept ALL property writes (V12 Frame-level lock)

        local function HookActorMeta(actor)

            if not actor or actor._metaHooked then return end

            local mt = getmetatable(actor)

            if not mt then return end

            local old_index = mt.__index



            -- We hook the metatable directly to ensure health is ALWAYS what we want

            local proxy = {

                Health = 999999,

                Hull = 999999,

                Alive = true,

                Dead = false

            }



            pcall(function()

                if not actor._orig_newindex then

                    actor._orig_newindex = mt.__newindex

                    mt.__newindex = function(t, k, v)

                        if t == actor then

                            if Config.Character_HealthRegen then

                                if (k == "Health" or k == "Hull") then return end -- Ignore writes to health

                                if (k == "Dead" or k == "Downed") and v == true then return end -- Ignore death/knock set

                                if k == "Alive" and v == false then return end

                            end

                        end

                        if actor._orig_newindex then return actor._orig_newindex(t, k, v) end

                        rawset(t, k, v)

                    end

                end

            end)

            actor._metaHooked = true

        end



        -- High-Frequency Engine Property Lock + Meta Hook

        task.spawn(function()

            while task.wait() do -- Faster loop (heartbeat level)

                if ReplicatorService then

                    local me = ReplicatorService.LocalActor

                    if me then

                        HookActorMeta(me)

                        if Config.Character_HealthRegen then

                            -- V14: Safe High-Val Brute Force

                            pcall(function()

                                me.Health = 999999

                                me.Hull = 999999

                                me.Stamina = 1000

                                me.Alive = true

                                me.Dead = false

                                me.Downed = false

                                rawset(me, "Health", 999999)

                                rawset(me, "Hull", 999999)

                                rawset(me, "Alive", true)

                                rawset(me, "Dead", false)

                                rawset(me, "Downed", false)

                            end)

                            if me.Character and me.Character:FindFirstChildOfClass("Humanoid") then

                                me.Character:FindFirstChildOfClass("Humanoid").Health = 100

                            end

                        end

                    end

                end

                if not getgenv().Blackhawk_Running then break end

            end

        end)



        -- Infinite Ammo: High-speed server-side sync backup with 999999 Method

        task.spawn(function()

            while task.wait(0.05) do

                if Config.InfiniteAmmo and ReplicatorService and Network and InventoryService then

                    pcall(function()

                        local inv = InventoryService.Equipped

                        local ctrl = inv and inv.Handler

                        if ctrl then

                            local mag = ctrl._mag

                            if mag then

                                -- V14: Set current weapon ammo to 999999

                                mag.Ammo = 999999

                                mag.Capacity = 999999

                                if mag.MetaData then

                                    mag.MetaData.Ammo = 999999

                                    mag.MetaData.Capacity = 999999

                                end



                                -- V15: Inventory-Wide Deep Sync (Lock ALL items in ALL inventories)

                                if InventoryService.Inventories then

                                    for _, inventory in pairs(InventoryService.Inventories) do

                                        if inventory._items then

                                            for _, item in pairs(inventory._items) do

                                                if item.Ammo ~= nil or item.Capacity ~= nil then

                                                    item.Ammo = 999999

                                                    item.Capacity = 999999

                                                    if item.MetaData then

                                                        item.MetaData.Ammo = 999999

                                                        item.MetaData.Capacity = 999999

                                                    end

                                                end

                                            end

                                        end

                                    end

                                end



                                -- Periodic server sync (every 2 seconds)

                                if not self._lastServerSync or tick() - self._lastServerSync > 2 then

                                    local ms = ctrl:_getMags()

                                    if ms and ms[1] then

                                        self._lastServerSync = tick()

                                        getgenv().LastSilentReload = tick()

                                        Network:FireServer("InventoryAction", "Reload", ms[1].UID, false)

                                    end

                                end

                            end

                        end

                    end)

                end

                if not getgenv().Blackhawk_Running then break end

            end

        end)

    end

end



-- NEW: Network Hook to block server damage packets

local function SetupNetworkHooks()

    if not Network then return end



    local function HookEvents(events)

        if not events then return end



        -- Use global Config for persistence across re-executions

        local cfg = getgenv().Blackhawk_Config



        -- Intercept ReplicateActor for health spoofing

        if events["ReplicateActor"] and not events._isHookedV12 then

            local oldRep = events["ReplicateActor"]

            events["ReplicateActor"] = function(uid, hp, ...)

                if cfg.Character_HealthRegen and ReplicatorService and uid == ReplicatorService._uid then

                    hp = 999999 -- V14 Force 999k HP Override

                end

                return oldRep(uid, hp, ...)

            end

            events._isHookedV12 = true

        end



        -- Intercept Bulk updates

        if events["ReplicateBulkActor"] and not events._isHookedV12_Bulk then

            local oldRep = events["ReplicateBulkActor"]

            events["ReplicateBulkActor"] = function(actors)

                if cfg.Character_HealthRegen and ReplicatorService then

                    for _, data in ipairs(actors) do

                        if data[1] == ReplicatorService._uid then

                            data[2] = 999999 -- Force 999k health

                        end

                    end

                end

                return oldRep(actors)

            end

            events._isHookedV12_Bulk = true

        end



        -- Intercept State updates (Dead/Health)

        if events["StateActor"] and not events._isHookedV12_State then

            local oldRep = events["StateActor"]

            events["StateActor"] = function(uid, key, value, force)

                if cfg.Character_HealthRegen and ReplicatorService and uid == ReplicatorService._uid then

                    if key == "Dead" or key == "Downed" then return end -- Block death/downed packets

                    if key == "Health" or key == "Hull" or key == "Stamina" or key == "Oxygen" then

                        value = 999999

                    end

                end

                return oldRep(uid, key, value, force)

            end

            events._isHookedV12_State = true

        end



        -- Intercept Damage / Kill events (Visual Block)

        if events["ReplicateKill"] and not events._isHookedV12_Kill then

            local oldKill = events["ReplicateKill"]

            events["ReplicateKill"] = function(uid, ...)

                if cfg.Character_HealthRegen and ReplicatorService and uid == ReplicatorService._uid then

                    return -- Block kill notification for self

                end

                return oldKill(uid, ...)

            end

            events._isHookedV12_Kill = true

        end



        -- Intercept Inventory updates (Infinite Ammo Sync)

        if events["InventoryAction"] and not events._isHookedV12_Inv then

            local oldAction = events["InventoryAction"]

            events["InventoryAction"] = function(uid, action, ...)

                if cfg.InfiniteAmmo then

                    -- Block reload animations triggered by our silent sync

                    if (action == "Reload" or action == "Reloaded") and tick() - (getgenv().LastSilentReload or 0) < 1.5 then

                        return

                    end

                end

                return oldAction(uid, action, ...)

            end

            events._isHookedV12_Inv = true

        end

    end



    -- 1. Hook FUTURE connections

    if not Network._isHookedConnect then

        local oldConnect = Network.ConnectEvents

        Network.ConnectEvents = function(self, events)

            HookEvents(events)

            return oldConnect(self, events)

        end

        Network._isHookedConnect = true

    end



    -- 2. Hook ALREADY ACTIVE connections

    if Network._events then

        HookEvents(Network._events)

    end

end



local function SetupEnvironmentHooks()

    -- Hook EnvironmentService:Update for No Fog and Always Day

    local es = EnvironmentService or FindService("EnvironmentService", true)



    if es then

        local mt = getmetatable(es)

        if mt and mt.Update then

            -- Hook the Metatable (Class) function

            if not mt._originalUpdate then

                setreadonly(mt, false)

                mt._originalUpdate = mt.Update

                local lighting = Lighting -- Cache once at hook creation time



                mt.Update = function(self, dt, ...)

                    -- 1. Call Original First (let game calculate effects)

                    local result = mt._originalUpdate(self, dt, ...)



                    -- 2. Force Overrides AFTER game logic



                    if Config.AlwaysDay then

                        lighting.ClockTime = 12

                    end



                    if Config.NoFog then

                        -- Override Lighting Fog

                        lighting.FogEnd = 100000

                        lighting.FogStart = 0



                        -- Override Atmosphere if it exists (EnvironmentService controls this)

                        local atmosphere = self._atmosphere

                        -- fallback to finding in Lighting if self._atmosphere isn't accessible

                        if not atmosphere then atmosphere = lighting:FindFirstChildOfClass("Atmosphere") end



                        if atmosphere then

                            atmosphere.Density = 0

                            atmosphere.Haze = 0

                            atmosphere.Glare = 0

                            atmosphere.Offset = 0

                        end



                        -- Override Clouds

                        local clouds = self._clouds or lighting:FindFirstChildOfClass("Clouds")

                        if clouds then

                            clouds.Cover = 0

                            clouds.Density = 0

                        end

                    end



                    return result

                end

                setreadonly(mt, true)

            end

        else

            -- Fallback if no metatable (unlikely but possible if found table IS the class)

            if es.Update and not es._originalUpdate then

                es._originalUpdate = es.Update

                local lighting = Lighting -- Cache once at hook creation time

                es.Update = function(self, dt, ...)

                    local result = es._originalUpdate(self, dt, ...)



                    -- Direct Force Fallback

                    if Config.AlwaysDay then lighting.ClockTime = 12 end

                    if Config.NoFog then

                        lighting.FogEnd = 100000

                        local atm = lighting:FindFirstChildOfClass("Atmosphere")

                        if atm then

                            atm.Density = 0; atm.Haze = 0

                        end

                        local clouds = lighting:FindFirstChildOfClass("Clouds")

                        if clouds then clouds.Cover = 0 end

                    end

                    return result

                end

            end

        end

    end

end



-- Hook Vehicle Controllers for Speed

local function SetupVehicleHooks()

    local gc = GroundController

    if gc and gc.Update and not gc._speedHooked then

        gc._origUpdate = gc.Update

        gc.Update = function(self, ...)

            if Config.VehicleSpeedMultiplier and Config.VehicleSpeedMultiplier > 0 then

                if self._tune then

                    if not self._tune._origAccel then self._tune._origAccel = self._tune.AccelerationFactor or 1 end

                    if not self._tune._origSpeed then self._tune._origSpeed = self._tune.MaxSpeed or 1 end

                    if not self._tune._origRSpeed then self._tune._origRSpeed = self._tune.ReverseMaxSpeed or 1 end



                    local newAccel = self._tune._origAccel * Config.VehicleSpeedMultiplier

                    local newSpeed = self._tune._origSpeed * Config.VehicleSpeedMultiplier



                    if self._tune.MaxSpeed ~= newSpeed then

                        self._tune.AccelerationFactor = newAccel

                        self._tune.MaxSpeed = newSpeed

                        self._tune.ReverseMaxSpeed = self._tune._origRSpeed * Config.VehicleSpeedMultiplier

                        if self._solver and self._solver.NewTune then

                            pcall(function() self._solver:NewTune() end)

                        end

                    end

                end

            elseif self._tune and self._tune._origAccel then

                self._tune.AccelerationFactor = self._tune._origAccel

                self._tune.MaxSpeed = self._tune._origSpeed

                self._tune.ReverseMaxSpeed = self._tune._origRSpeed

                if self._solver and self._solver.NewTune then

                    pcall(function() self._solver:NewTune() end)

                end

                self._tune._origAccel = nil

            end

            return gc._origUpdate(self, ...)

        end

        gc._speedHooked = true

    end



    local hc = HelicopterController

    if hc and hc.Update and not hc._speedHooked then

        hc._origUpdate = hc.Update

        hc.Update = function(self, ...)

            if Config.VehicleSpeedMultiplier and Config.VehicleSpeedMultiplier > 0 then

                if self._tune then

                    if not self._tune._origFlightAccel then self._tune._origFlightAccel = self._tune.Acceleration or 1 end

                    if not self._tune._origFlightSpeed then self._tune._origFlightSpeed = self._tune.Speed or 1 end

                    if not self._tune._origAccel then self._tune._origAccel = self._tune.AccelerationFactor or 1 end

                    if not self._tune._origSpeed then self._tune._origSpeed = self._tune.MaxSpeed or 1 end



                    local newFlightSpeed = self._tune._origFlightSpeed * Config.VehicleSpeedMultiplier



                    if self._tune.Speed ~= newFlightSpeed then

                        self._tune.Acceleration = self._tune._origFlightAccel * Config.VehicleSpeedMultiplier

                        self._tune.Speed = newFlightSpeed

                        self._tune.AccelerationFactor = self._tune._origAccel * Config.VehicleSpeedMultiplier

                        self._tune.MaxSpeed = self._tune._origSpeed * Config.VehicleSpeedMultiplier

                        if self._solver and self._solver.NewTune then

                            pcall(function() self._solver:NewTune() end)

                        end

                    end

                end

            elseif self._tune and self._tune._origFlightAccel then

                self._tune.Acceleration = self._tune._origFlightAccel

                self._tune.Speed = self._tune._origFlightSpeed

                self._tune.AccelerationFactor = self._tune._origAccel

                self._tune.MaxSpeed = self._tune._origSpeed

                if self._solver and self._solver.NewTune then

                    pcall(function() self._solver:NewTune() end)

                end

                self._tune._origFlightAccel = nil

            end

            return hc._origUpdate(self, ...)

        end

        hc._speedHooked = true

    end

end



-- Removed 'local' to use forward declaration

function SetupHooks()

    pcall(SetupNetworkHooks)

    pcall(SetupAggressiveHooks)

    pcall(SetupActorHooks)

    pcall(SetupEnvironmentHooks)

    pcall(SetupVehicleHooks)



    -- Try to find TargetController more aggressively

    local TargetController = CharacterController

    if not TargetController or not TargetController.Update then

        for _, obj in pairs(getgc(true)) do

            if type(obj) == "table" and rawget(obj, "Update") and rawget(obj, "SetCFrame") and rawget(obj, "_processNewPosition") then

                TargetController = obj

                CharacterController = obj

                break

            end

        end

    end



    if not TargetController or not TargetController.Update then

        warn("[VoidSens] CRITICAL: CharacterController NOT found after deep scan. Noclip/Fly might not work!")

    end



    if TargetController and TargetController.Update then

        EnableSpeedController()

        if not TargetController._originalUpdate then

            TargetController._originalUpdate = TargetController.Update

        end



        local OldUpdate = TargetController._originalUpdate



        if not TargetController._originalProcessNewPosition then

            TargetController._originalProcessNewPosition = TargetController._processNewPosition

        end



        if TargetController._originalProcessNewPosition then

            TargetController._processNewPosition = function(self, nextPos, ...)

                if Config.CharacterNoclip_Enabled or Config.Character_Fly then

                    -- V12: Ultimate bypass, force ground check to true to stop falling through map

                    return nextPos, true, Vector3.new(0, 1, 0), nil

                end

                return TargetController._originalProcessNewPosition(self, nextPos, ...)

            end

        end



        TargetController.Update = function(self, viewInput, dt, ...)

            local localActor = (ReplicatorService and ReplicatorService.LocalActor) or (self and self._localActor)



            if Config.Character_Fly and localActor and localActor.Alive then

                localActor.UseClient = true; localActor._lod = false; localActor._isInactive = false

                self.VelocityGravity = 0; self.HeightState = 0; self.IsGrounded = true

                if localActor.Rappelling then localActor.Rappelling = nil end



                if localActor.Character and localActor.Character.Parent ~= workspace then localActor.Character.Parent = workspace end

                if localActor.Parts then

                    for _, part in pairs(localActor.Parts) do if part:IsA("BasePart") then part.LocalTransparencyModifier = 0 end end

                end



                local camCF = Camera.CFrame

                local dir = Vector3_new(0, 0, 0)

                if UserInputService:IsKeyDown(Enum.KeyCode.W) then dir = dir + camCF.LookVector end

                if UserInputService:IsKeyDown(Enum.KeyCode.S) then dir = dir - camCF.LookVector end

                if UserInputService:IsKeyDown(Enum.KeyCode.D) then dir = dir + camCF.RightVector end

                if UserInputService:IsKeyDown(Enum.KeyCode.A) then dir = dir - camCF.RightVector end

                if UserInputService:IsKeyDown(Enum.KeyCode.Space) then dir = dir + Vector3_new(0, 1, 0) end

                if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then dir = dir - Vector3_new(0, 1, 0) end



                local flySpeedVal = tonumber(Config.Character_FlySpeed) or 50

                local flySprint = UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) and 2.5 or 1

                local deltaTime = type(dt) == "number" and dt or 0.016

                local nextPos = self._position + (dir * flySpeedVal * flySprint * deltaTime)



                self._position = nextPos; self._lastSafePosition = nextPos

                localActor.Position = nextPos; localActor.SimulatedPosition = nextPos

                local _, yRot = camCF:ToOrientation()

                localActor.CFrame = CFrame.new(nextPos) * CFrame.Angles(0, yRot, 0); localActor.Orientation = yRot

                return

            elseif Config.CharacterNoclip_Enabled then

                -- Aggressive Noclip state lock

                self.VelocityGravity = 0; self.IsGrounded = true

                if localActor and localActor.Rappelling then localActor.Rappelling = nil end

            end



            return OldUpdate(self, viewInput, dt, ...)

        end

    end



    -- Persistent Noclip/Fly Bypass

    if getgenv().NoclipConnection then pcall(function() getgenv().NoclipConnection:Disconnect() end) end

    getgenv().NoclipConnection = RunService.Stepped:Connect(function()

        local cfg = getgenv().Blackhawk_Config

        if cfg.CharacterNoclip_Enabled or cfg.Character_Fly then

            local la = ReplicatorService and ReplicatorService.LocalActor

            local char = la and la.Character

            if char then

                for _, p in pairs(char:GetDescendants()) do

                    if p:IsA("BasePart") then

                        p.CanCollide = false; p.CanTouch = false; p.CanQuery = false; p.Massless = true

                        p.AssemblyLinearVelocity = Vector3.new(); p.Velocity = Vector3.new()

                    end

                end

            end

        end

    end)

    table.insert(getgenv().BlackhawkESP_Connections, getgenv().NoclipConnection)

end



--[[

    COMBAT LOGIC

]]

local function UpdateCombat(frameId)

    local now = getgenv().LastTime or tick()

    -- OPTIMIZATION: Cache mouse state once per frame (avoid repeated UserInputService calls)

    local mousePressed = UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton1)



    local cfg = Config

    local activeVelocity = 2000 -- Already local, keep initial value

    local showWeaponInfo = cfg.ShowWeaponInfo

    local showPred = cfg.ShowPrediction or (cfg.SilentAim and cfg.Prediction)

    local activeTune -- Declare activeTune as local here





    -- BACKGROUND LOOPS COMPLETELY REMOVED.

    -- Modifications are handled permanently by class hooks.



    -- Update via signals if possible

    if InventoryService and not WeaponManager.Connection then

        if InventoryService.OnInterfaceStateChanged then

            WeaponManager.Connection = InventoryService.OnInterfaceStateChanged:Connect(function()

                WeaponManager.Equipped = InventoryService.Equipped and InventoryService.Equipped.Handler

            end)

            table.insert(getgenv().BlackhawkESP_Connections, WeaponManager.Connection)

        end

        -- Initial Sync

        WeaponManager.Equipped = InventoryService.Equipped and InventoryService.Equipped.Handler

    end



        -- JIT Cache & Immediate Modification

        local equippedController = WeaponManager.Equipped or

            (InventoryService and InventoryService.Equipped and InventoryService.Equipped.Handler)



        -- OHK Logic transitioned to ApplyGunMods and CalibersTable for 460 DMG reliability



        -- [INFINITE AMMO] Hook Discharge + lock mag every frame

        if Config.InfiniteAmmo and equippedController then

            pcall(function()

                -- 1. Lock current magazine to 999k

                local mag = equippedController._mag

                if mag then

                    mag.Ammo = 999999

                    mag.Capacity = 999999

                    if mag.MetaData then

                        mag.MetaData.Ammo = 999999

                        mag.MetaData.Capacity = 999999

                    end

                end

                -- 2. Hook Discharge to force ammo at moment of firing

                if not equippedController._dischargeHooked then

                    equippedController._dischargeHooked = true

                    local origDischarge = equippedController.Discharge

                    if origDischarge then

                        equippedController.Discharge = function(self, ...)

                            if Config.InfiniteAmmo then

                                local m = self._mag

                                if m then

                                    m.Ammo = 999999

                                    m.Capacity = 999999

                                    if m.MetaData then

                                        m.MetaData.Ammo = 999999

                                        m.MetaData.Capacity = 999999

                                    end

                                end

                            end

                            return origDischarge(self, ...)

                        end

                    end

                end

                -- 3. Lock reserve mags and search for boxes every 5 frames

                if frameId % 5 == 0 then

                    local mags = equippedController:_getMags()

                    if mags then

                        for _, m in pairs(mags) do

                            if m then

                                m.Ammo = 999999

                                m.Capacity = 999999

                                if m.MetaData then

                                    m.MetaData.Ammo = 999999

                                    m.MetaData.Capacity = 999999

                                end

                            end

                        end

                    end

                    -- Caliber search for boxes in Drawing Loop

                    local cal = equippedController._caliber or (equippedController._config and equippedController._config.Caliber)

                    if cal and InventoryService and InventoryService.Inventories then

                        for _, inv in pairs(InventoryService.Inventories) do

                            if inv._items then

                                for _, item in pairs(inv._items) do

                                    if item._caliber == cal then

                                        item.Ammo = 999999

                                        item.Capacity = 999999

                                        if item.MetaData then item.MetaData.Ammo = 999999; item.MetaData.Capacity = 999999 end

                                    end

                                end

                            end

                        end

                    end

                end

            end)

        end



        -- [LOGIC REMOVED]



    -- Cache Combat Targets for this frame

    local targetPart_RB = ActorManager.SelectedTarget_RB

    local targetPart_SA = ActorManager.SelectedTarget_SA



    if equippedController then

        ApplyGunMods(equippedController, false)



        -- IMMEDIATE HOOK: Ensure Silent Aim works instantly

        if (Config.SilentAim or Config.RageMode) and not equippedController._isHookedESP then

            HookController(equippedController)

            equippedController._isHookedESP = true

        end



        -- Ragebot Combat Logic (OPTIMIZED - Cached Stats & Smart Shooting)

        local rageDidShoot = false -- Track if we fired this frame



        if Config.RageMode and ActorManager.SelectedTarget_RB then

            local targetActor = ActorManager.SelectedActor_RB

            if not targetActor or not targetActor.Alive then return end



            local uid = targetActor.UID



            -- CACHE: Weapon stats + damage (cached per weapon change, not per frame)

            local weaponRPM = 600

            local weaponCaliber = equippedController._caliber



            -- Use cached weapon data if same weapon as last frame

            local cachedWeapon = ActorManager._cachedRageWeapon

            if cachedWeapon and cachedWeapon.controller == equippedController then

                weaponRPM = cachedWeapon.rpm

                -- estDamage from cache

            else

                -- Recalculate weapon stats (only on weapon change)

                if equippedController._firearm and equippedController._firearm.Tune then

                    weaponRPM = equippedController._firearm.Tune.RPM or 600

                end



                local estDmg = 80

                local tgtPartName = Config.Rage_TargetPart or "Head"

                local calConfig = CalibersTable and weaponCaliber and CalibersTable[weaponCaliber]

                if calConfig and calConfig.Damage and calConfig.Dropoff and BulletService then

                    local ok, dmg = pcall(function()

                        local _, rangeOff = BulletService:GetInfo(weaponCaliber, 10)

                        return BulletService:GetDamageGraph(0, rangeOff or 0, calConfig.Damage, calConfig.Dropoff,

                            calConfig.Damp or 1, tgtPartName)

                    end)

                    if ok and dmg and dmg > 0 then

                        local bullets = calConfig.Bullets or 1

                        estDmg = dmg * bullets

                    end

                end



                ActorManager._cachedRageWeapon = {

                    controller = equippedController,

                    rpm = weaponRPM,

                    estDamage = estDmg

                }

            end



            local estDamage = ActorManager._cachedRageWeapon and ActorManager._cachedRageWeapon.estDamage or 80



            -- 1 shot per RPM tick, then switch target immediately

            local fireDelay = 60 / (Config.RPMValue or weaponRPM)

            local canFire = (now - (ActorManager._rageLast[uid] or 0) >= fireDelay)



            if canFire then

                local hasAmmo = true -- Infinite ammo handles itself above

                if not Config.InfiniteAmmo then

                    local mag = equippedController._mag

                    local meta = equippedController._item and equippedController._item.MetaData

                    local cur = (mag and (mag.Ammo or 0)) or (meta and meta.Chamber and 1 or 0)

                    hasAmmo = cur > 0

                end



                if hasAmmo then

                    ActorManager._rageLast[uid] = now

                    rageDidShoot = true



                    local dischargeMethod = equippedController.Discharge

                    if dischargeMethod then

                        dischargeMethod(equippedController, Camera.CFrame)

                        -- Switch target after 1 shot

                        ActorManager._ignoredTargets[uid] = now + 0.05

                        ActorManager.SelectedTarget_RB = nil

                        ActorManager.SelectedActor_RB = nil

                    end

                end

            end

        end



        -- SAFETY: Force stop weapon when ragebot is active but NOT shooting this frame

        if Config.RageMode and not rageDidShoot and equippedController._firing

            and not mousePressed then

            -- Optional cleanup

        end



        -- AUTO FIRE / TRIGGERBOT LOGIC (Integrated for Silent Aim accuracy)

        if Config.Triggerbot and not Config.RageMode then

            local targetActor = ActorManager.SelectedActor_SA

            local validTarget = false



            if targetActor and targetActor.Alive then

                -- Silent Aim Target Check (Priority)

                validTarget = true

            else

                -- Fallback: Raycast forward

                local mouse = UserInputService:GetMouseLocation()

                local ray = Camera:ViewportPointToRay(mouse.X, mouse.Y)

                local params = RaycastParams.new()

                params.FilterType = Enum.RaycastFilterType.Exclude

                params.FilterDescendantsInstances = { Camera, LocalPlayer.Character }

                local result = Workspace:Raycast(ray.Origin, ray.Direction * 1000, params)



                if result and result.Instance then

                    local model = result.Instance:FindFirstAncestorOfClass("Model")

                    if model then

                        for _, enemy in ipairs(ActorManager.Enemies) do

                            if enemy.Character == model and enemy.Alive then

                                validTarget = true

                                break

                            end

                        end

                    end

                end

            end



            if validTarget and equippedController then

                local weaponRPM = (equippedController._firearm and equippedController._firearm.Tune and equippedController._firearm.Tune.RPM) or 600

                local fireDelay = 60 / weaponRPM

                local lastShot = equippedController._lastTriggerShot or 0



                if now - lastShot >= (fireDelay * 0.95) then -- Slight buffer for stability

                    -- Enhanced Nil Safety for triggerbot ammo check

                    local mag = equippedController._mag

                    local meta = equippedController._item and equippedController._item.MetaData

                    local hasAmmo = false



                    if Config.InfiniteAmmo then

                        hasAmmo = true

                    elseif mag then

                        hasAmmo = (mag.Ammo or 0) > 0

                    elseif meta then

                        hasAmmo = meta.Chamber == true

                    end



                    if hasAmmo then

                         equippedController._lastTriggerShot = now

                        if equippedController.Discharge then

                            equippedController:Discharge(Camera.CFrame)

                        end

                    end

                end

            end

        end





        -- IMMEDIATE HOOK: Vehicle Turret (Manual Fetch)

        -- This ensures we hook the turret even if it's not in the main cache yet

        if ReplicatorService and ReplicatorService.LocalActor then

            local myActor = ReplicatorService.LocalActor

            if myActor.Turret and type(myActor.Turret) == "table" then

                local turretController = myActor.Turret

                -- Apply Turret Mods (Directly)

                local tConfig = turretController._config

                if tConfig then

                    -- Unlock config table efficiently

                    if setreadonly then pcall(setreadonly, tConfig, false) end

                    if make_writeable then pcall(make_writeable, tConfig) end



                    if cfg.TurretNoRecoil then

                        tConfig.Recoil = 0

                        tConfig.Recoil_Base = Vector2.new(0, 0)

                        tConfig.Recoil_Range = Vector2.new(0, 0)

                        tConfig.Recoil_Camera = 0

                        tConfig.Recoil_Kick = 0

                        tConfig.Recoil_Smooth = 0

                        tConfig.RPM_RecoilScale = 0

                        tConfig.CameraShake = nil -- Disable visual shake

                    end

                    if cfg.TurretNoSpread then

                        tConfig.Spread = 0

                        tConfig.Barrel_Spread = 0

                        tConfig.MinSpread = 0

                        tConfig.MaxSpread = 0

                        tConfig.Recoil_Spread = 0



                        if CalibersTable and tConfig.Caliber then

                            local cal = CalibersTable[tConfig.Caliber]

                            if cal then cal.Spread = 0 end

                        end

                    end

                    if cfg.TurretUnlockFiremodes and tConfig.Firemodes then

                        -- Check for all auto-related modes (0, 2)

                        local hasAuto = false

                        for _, m in pairs(tConfig.Firemodes) do if m == 0 or m == 2 then hasAuto = true end end

                        if not hasAuto then table.insert(tConfig.Firemodes, 0) end

                    end

                end

            end

        end





        -- 1. Gun Mods (Standard Cache Loop - Keep for Turrets ONLY)

        -- THROTTLED: Run every 10 frames (approx 0.16s) to save CPU

        if (frameId % 10 == 0) and (cfg.TurretNoRecoil or cfg.TurretNoSpread or cfg.TurretUnlockFiremodes) then

            -- VISUALS: Turrets

            if AllControllersCache.Visuals then

                for _, controller in ipairs(AllControllersCache.Visuals) do

                    if type(controller) ~= "table" or controller == equippedController then continue end -- Skip active/invalid



                    local isTurret = rawget(controller, "_turret") ~= nil



                    if isTurret then

                        local tConfig = controller._config

                        if tConfig then

                            -- Unlock config table if readonly

                            if setreadonly then pcall(setreadonly, tConfig, false) end



                            if cfg.TurretNoRecoil then

                                tConfig.Recoil = 0

                                tConfig.Recoil_Base = Vector2.new(0, 0)

                                tConfig.Recoil_Camera = 0

                                tConfig.Recoil_Kick = 0

                            end

                            if cfg.TurretNoSpread then

                                tConfig.Spread = 0

                                tConfig.Barrel_Spread = 0

                            end

                            if cfg.TurretUnlockFiremodes and tConfig.Firemodes then

                                local hasAuto = false

                                for _, m in pairs(tConfig.Firemodes) do if m == 0 or m == 2 then hasAuto = true end end

                                if not hasAuto then table.insert(tConfig.Firemodes, 0) end

                            end

                        end

                    end

                end

            end

        end



        -- Weapon Info Overlay (Fixed Visibility Logic)

        if WeaponInfoText then

            local shouldShow = Config.ShowWeaponInfo == true

            -- INSTANT HIDE: If toggled off, hide immediately regardless of weapon state

            if not shouldShow then

                WeaponInfoText.Visible = false

            else

                local hasWeapon = false



                if equippedController and equippedController._firearm then

                    local firearm = equippedController._firearm

                    activeTune = firearm.Tune

                    if activeTune then

                        hasWeapon = true

                        -- Update text only every 5 frames for performance

                        if frameId % 5 == 0 then

                            local velocity = 0

                            if BulletService and activeTune.Caliber then

                                local success, v = pcall(function()

                                    return BulletService:GetInfo(activeTune.Caliber, activeTune.Barrel or 1)

                                end)

                                if success and v and v > 0 then velocity = v end

                            end

                            if velocity == 0 then

                                velocity = activeTune.Velocity or (activeTune.MuzzleVelocity and activeTune.MuzzleVelocity.Value) or 2000

                            end

                            activeVelocity = velocity



                            local weaponName = firearm.Name or activeTune.Name or "Unknown"

                            local caliberName = ""

                            if CalibersTable and activeTune.Caliber and CalibersTable[activeTune.Caliber] then

                                local cal = CalibersTable[activeTune.Caliber]

                                if cal and cal.FamilyName then

                                    caliberName = cal.FamilyName

                                    if cal.VariantName and cal.VariantName ~= "" then

                                        caliberName = caliberName .. " " .. cal.VariantName

                                    end

                                end

                            end



                            local velocityMS = math_floor(velocity * 0.3048 + 0.5)

                            local infoText = string_format("Weapon: %s\nCaliber: %s\nVelocity: %d m/s", weaponName,

                                caliberName ~= "" and caliberName or "N/A", velocityMS)



                            if WeaponInfoText.Text ~= infoText then

                                WeaponInfoText.Text = infoText

                            end

                            WeaponInfoText.Position = Vector2_new(50, Camera.ViewportSize.Y - 120)

                            -- Apply user-set color

                            WeaponInfoText.Color = Config.WeaponInfoColor or Color3.fromRGB(255, 255, 100)

                        end

                    end

                end

                WeaponInfoText.Visible = hasWeapon

            end

        end



        -- 2. Apply Silent Aim Hooks -> uses cached FirearmInventory

        if Config.SilentAim and FirearmInventory and type(FirearmInventory) == "table" then

            for _, controller in pairs(FirearmInventory) do

                if type(controller) == "table" and controller._firearm and not controller._isHookedESP then

                    HookController(controller)

                    controller._isHookedESP = true

                end

            end

        end



        -- Spinbot / Ragebot: High-Priority Orientation & CFrame Lock (Triple-Lock System)

        if (cfg.RageMode or cfg.Spinbot) and ReplicatorService and ReplicatorService.LocalActor then

            local myActor = ReplicatorService.LocalActor

            if myActor.Alive and myActor.Character then

                -- OPTIMIZATION: actor.RootPart is direct engine ref (no FindFirstChild)

                local root = myActor.RootPart or myActor.Character.PrimaryPart

                if root then

                    local muzzlePos = GetMuzzlePosition() or Camera.CFrame.Position

                    local targetPos = targetPart_RB and targetPart_RB.Position



                    local yaw, pitch



                    if cfg.RageMode and cfg.Rage_LookAt and targetPart_RB then

                        -- RAGE TARGET ROTATION

                        local direction = (targetPos - muzzlePos).Unit

                        yaw = math.atan2(direction.X, direction.Z)

                        pitch = math.atan2(targetPos.Y - muzzlePos.Y, Vector2_new(direction.X, direction.Z).Magnitude)



                        root.CFrame = CFrame.new(root.Position,

                            root.Position + Vector3_new(math.sin(yaw), 0, math.cos(yaw)))

                    elseif cfg.Spinbot then

                        -- SPINBOT ROTATION

                        local speed = (Config.SpinbotSpeed or 20)

                        yaw = (tick() * speed) % (math.pi * 2)

                        pitch = 0



                        root.CFrame = CFrame.new(root.Position) * CFrame.Angles(0, yaw, 0)

                    end



                    if yaw then

                        -- Update Actor Properties for engine visibility and server replication

                        myActor.Orientation = yaw

                        myActor.GoalOrientation = yaw

                        myActor.CameraX = yaw

                        myActor.CameraY = pitch



                        local controller = ControllerService and ControllerService.Controller

                        if controller then

                            controller.Orientation = yaw

                        end

                    end

                end

            end

        end



        -- 4. Update Combat Visuals (FOV, Prediction, Turret CH)

        local myActor = ReplicatorService and ReplicatorService.LocalActor



        -- FALL DAMAGE PREVENTION

        if Config.Character_Fly then

            getgenv()._lastFlyTick = tick()

        end

        if not Config.Character_Fly and tick() - (getgenv()._lastFlyTick or 0) < 2 then

            pcall(function()

                local root = myActor and (myActor.RootPart or (myActor.Character and myActor.Character.PrimaryPart))

                if root and root.Velocity.Y < -5 then

                    root.Velocity = Vector3.new(root.Velocity.X, -5, root.Velocity.Z)

                end

            end)

        end



        -- VEHICLE SPEED HACK (Unthrottled for high-speed responsiveness)

        local cfg = getgenv().Blackhawk_Config

        if cfg and cfg.VehicleSpeedHack and VehicleService and VehicleService.Vehicles then

            pcall(function()

                for _, veh in pairs(VehicleService.Vehicles) do

                    if veh and veh.Controlling then

                        local solver = veh.Solver or veh._solver

                        local mult = cfg.VehicleSpeedMult or 5 -- Increased multiplier effect

                        if solver and solver._tune then

                            local tune = solver._tune

                            if not tune._origMaxSpeed then tune._origMaxSpeed = tune.MaxSpeed end

                            if not tune._origTorque then tune._origTorque = tune.Engine_Torque or 1000 end



                            tune.MaxSpeed = tune._origMaxSpeed * mult

                            tune.Engine_Torque = tune._origTorque * (mult * 3) -- Increased torque multiplier

                            tune.AccelerationFactor = (tune.AccelerationFactor or 1) * 3 -- Increased acceleration multiplier

                        end



                        if solver and solver._transmissions then

                            for _, trans in pairs(solver._transmissions) do

                                if trans then

                                    if not trans._origMaxSpeed then

                                        trans._origMaxSpeed = trans.MaxSpeed or 100

                                        trans._origAcc = trans.AccelerationFactor or 1

                                    end

                                    trans.MaxSpeed = (trans._origMaxSpeed or 100) * mult

                                    trans.AccelerationFactor = (trans._origAcc or 1) * 3 -- Increased acceleration multiplier

                                    if trans._gearToSpeed then

                                        for i, v in pairs(trans._gearToSpeed) do

                                            if type(v) == "number" then trans._gearToSpeed[i] = v * mult end

                                        end

                                    end

                                end

                            end

                        end

                    end

                end

            end)

        elseif cfg and not cfg.VehicleSpeedHack and VehicleService and VehicleService.Vehicles then

            -- Reset logic

            pcall(function()

                for _, veh in pairs(VehicleService.Vehicles) do

                    if veh and veh.Controlling then

                        local solver = veh.Solver or veh._solver

                        if solver and solver._tune and solver._tune._origMaxSpeed then

                            solver._tune.MaxSpeed = solver._tune._origMaxSpeed

                            solver._tune.Engine_Torque = solver._tune._origTorque

                            solver._tune.AccelerationFactor = solver._tune._origAccel or 1 -- Reset acceleration

                        end

                        if solver and solver._transmissions then

                            for _, trans in pairs(solver._transmissions) do

                                if trans and trans._origMaxSpeed then

                                    trans.MaxSpeed = trans._origMaxSpeed

                                    trans.AccelerationFactor = trans._origAcc or 1 -- Reset acceleration

                                    if trans._gearToSpeed then

                                        -- Re-calculate original gear speeds if needed, or store them

                                        -- For now, just reset the multiplier effect if it was applied

                                        for i, v in pairs(trans._gearToSpeed) do

                                            if type(v) == "number" and trans._origMaxSpeed ~= 0 then

                                                -- This is a simplified reset. A full reset would require storing original gear ratios.

                                                -- Assuming original gear speeds are proportional to MaxSpeed.

                                                trans._gearToSpeed[i] = (v / (cfg.VehicleSpeedMult or 5))

                                            end

                                        end

                                    end

                                end

                            end

                        end

                    end

                end

            end)

        end



        -- TURRET CROSSHAIR & ZOOM

        if CENTER_CH then

            local inTurret = false

            if cfg and myActor and myActor.Turret and type(myActor.Turret) == "table" then

                inTurret = true



                -- Zoom Logic

                if cfg.Turret_ZoomEnabled and cfg.Turret_ZoomValue then

                    Camera.FieldOfView = cfg.Turret_ZoomValue

                end



                if cfg.TurretCH_Enabled then

                    CENTER_CH.Visible = true

                    for _, part in pairs(CENTER_CH:GetChildren()) do

                        if part:IsA("Frame") and part.Name ~= "Circle" then -- Exclude the special Circle frame

                            part.BackgroundColor3 = cfg.TurretCH_Color or Color3_new(1, 1, 1)



                            -- Style Logic

                            local style = cfg.TurretCH_Style or "Cross"

                            if style == "Dot" then

                                part.Visible = (part.Name == "Center")

                            elseif style == "Circle" then

                                part.Visible = false -- Handled by the specific Circle frame

                            elseif style == "T-Shape" then

                                part.Visible = (part.Name ~= "Top")

                            elseif style == "Cross" then

                                part.Visible = (part.Name ~= "Center")

                            else -- Default to Cross if unknown style

                                part.Visible = (part.Name ~= "Center")

                            end

                        end

                    end

                    -- Special Circle Handling

                    if CENTER_CH:FindFirstChild("Circle") then

                        CENTER_CH.Circle.Visible = (cfg.TurretCH_Style == "Circle")

                        CENTER_CH.Circle.UIStroke.Color = cfg.TurretCH_Color or Color3_new(1, 1, 1)

                    end

                else

                    CENTER_CH.Visible = false

                    if cfg and cfg.Turret_ZoomEnabled then

                        Camera.FieldOfView = 70

                    end

                end

            else

                CENTER_CH.Visible = false

                if cfg and cfg.Turret_ZoomEnabled then

                    Camera.FieldOfView = 70

                end

            end

        end



        -- FOV CIRCLE

        if Config.SilentAim and Config.ShowFOV and FOVCircle then

            local mouse = UserInputService:GetMouseLocation()

            FOVCircle.Visible = true

            FOVCircle.Position = mouse

            FOVCircle.Radius = Config.SilentAimFOV

        elseif FOVCircle then

            FOVCircle.Visible = false

        end



        -- Optimized Prediction Logic (Using Silent Aim target for visualization)

        local targetPart = targetPart_SA

        local targetActor = ActorManager.SelectedActor_SA



        if targetPart then

            local rawVel = Vector3_new(0, 0, 0)

            if targetActor and targetActor._oldDirection then

                rawVel = targetActor._oldDirection * 60

            elseif targetPart.Parent and targetPart.Parent.PrimaryPart then

                rawVel = targetPart.Parent.PrimaryPart.AssemblyLinearVelocity or Vector3_new(0, 0, 0)

            end



            -- Initialize or Lerp

            if not getgenv()._lastPredVel then

                getgenv()._lastPredVel = rawVel

            else

                -- LERP Smoothing: 0.15 seems to be the sweet spot for stability

                getgenv()._lastPredVel = getgenv()._lastPredVel:Lerp(rawVel, 0.15)

            end

        end



        if getgenv().PredictionDot then

            local dot = getgenv().PredictionDot



            if Config.ShowPrediction and targetPart then

                local muzzlePos = GetMuzzlePosition() or Camera.CFrame.Position

                local targetPos = targetPart.Position

                local targetVel = getgenv()._lastPredVel or Vector3_new(0, 0, 0)

                local gravity = 32.2 -- Restored to game value (was 25)



                local aimPos, _ = SolveLead(muzzlePos, targetPos, targetVel, activeVelocity, gravity)

                local screenPos, onScreen = Camera:WorldToViewportPoint(aimPos)



                if onScreen then

                    dot.Position = Vector2_new(screenPos.X, screenPos.Y)

                    dot.Color = cfg.PredictionColor or Color3_fromHSV(0.33, 1, 1)

                    dot.Visible = true

                else

                    dot.Visible = false

                end

            else

                dot.Visible = false

            end

        end

    end

end



local function GetEntityColor(actor, entityType, cachedLocalSquad)

    if entityType == "Player" then

        if IsTeammate(actor, cachedLocalSquad) then

            return Config.SquadColor or Config.TeammateColor or Color3_new(0, 1, 0) -- Green

        end

        return Config.PlayerColor or Color3_new(1, 0.65, 0)                         -- Orange/Red

    elseif entityType == "Zombie" then

        -- Check zombie ability level for color

        if actor.Health and type(actor.Health) == "table" and actor.Health.Ability then

            return Config.ZombieColors[actor.Health.Ability] or Config.ZombieColor or Color3_new(1, 0, 0)

        end

        return Config.ZombieColor or Color3_new(1, 0, 0)

    elseif entityType == "NPC" then

        return Config.NPCColor or Color3_new(1, 1, 0) -- Yellow

    end



    return Color3_new(1, 1, 1) -- White for unknown

end



--[[

    MAP DETECTION

]]

--[[

    VEHICLE TELEPORT LOGIC

]]



local function GetNetwork()

    if Network then return Network end



    -- Robust Logic from VehicleTeleport.lua

    -- 1. Check Executor Global Cache

    if getgenv().network and type(getgenv().network._key) == "table" then

        Network = getgenv().network

        return Network

    end



    -- 2. Try getnilinstances

    for _, obj in pairs(getnilinstances()) do

        if obj:IsA("ModuleScript") and obj.Name:lower():find("network") then

            local s, v = pcall(require, obj)

            if s and type(v) == "table" and v.FireServer then

                Network = v

                return Network

            end

        end

    end



    -- 3. Deep GC Scan

    for _, v in pairs(getgc(true)) do

        if type(v) == "table" then

            if rawget(v, "_key") and type(rawget(v, "_key")) == "table" and rawget(v, "_code") then

                Network = v

                return Network

            end

            -- Fallback fingerprint

            if rawget(v, "FireServer") and rawget(v, "_events") then

                Network = v

                return Network

            end

        end

    end

    return nil

end



local ActiveControllerCache = nil

local function GetActiveVehicleController()

    -- 1. Trust ControllerService

    if ControllerService and ControllerService.Controller and ControllerService.Controller._vehicle then

        return ControllerService.Controller

    end



    -- 2. Check ReplicatorService for LocalActor Seat/Turret UID

    local myVehUID = nil

    local localActor = ReplicatorService and ReplicatorService.LocalActor

    if localActor then

        if localActor.Seat then

            myVehUID = localActor.Seat.UID

            if not myVehUID and localActor.Seat._vehicle then

                myVehUID = localActor.Seat._vehicle.UID

            end

        elseif localActor.Turret then

            if localActor.Turret._vehicle then

                myVehUID = localActor.Turret._vehicle.UID

            elseif localActor.Turret._uid then

                myVehUID = localActor.Turret._uid:split("_")[1]

            end

        end

    end



    -- 3. Fallback: Find Vehicle via VehicleService

    if VehicleService and VehicleService.Vehicles then

        local player = LocalPlayer

        local char = player.Character

        local root = char and (char:FindFirstChild("HumanoidRootPart") or char.PrimaryPart)

        local hum = char and char:FindFirstChild("Humanoid")

        local seat = hum and hum.SeatPart



        for _, veh in pairs(VehicleService.Vehicles) do

            if veh and type(veh) == "table" then

                -- Match via LocalActor UID

                if myVehUID and tostring(veh.UID) == tostring(myVehUID) then

                    return { _vehicle = veh, _solver = veh._solver or {} }

                end



                if veh.Model then

                    -- Match via Roblox Seat

                    if seat and seat:IsDescendantOf(veh.Model) then

                        return { _vehicle = veh, _solver = veh._solver or {} }

                    end



                    -- Match via Distance / Ownership

                    if veh.Owner == player or veh.Owner == player.Name or veh._owner == player or (veh._owner and veh._owner.Name == player.Name) then

                         local targetPart = veh.Model.PrimaryPart or veh.Model:FindFirstChildOfClass("BasePart")

                         local dist = (targetPart and root) and (root.Position - targetPart.Position).Magnitude or 9999



                         if dist < 80 or dist == 9999 then

                             return { _vehicle = veh, _solver = veh._solver or {} }

                         end

                    end

                end

            end

        end

    end



    return nil

end



local function TeleportVehicle(targetPos, targetRotY)

    local controller = GetActiveVehicleController()



    local newCF = CFrame.new(targetPos)

    if targetRotY then

        newCF = newCF * CFrame.Angles(0, math.rad(targetRotY), 0)

    end



    local net = (Network or (ClientService and ClientService.Network) or (shared.Engine and shared.Engine.Network))

    if not net then return end



    local vehicle = controller and controller._vehicle

    if not vehicle then

        -- Character Teleport

        local char = LocalPlayer.Character

        local root = char and (char:FindFirstChild("HumanoidRootPart") or char.PrimaryPart)

        if root then

            local offsetCF = newCF * CFrame.new(0, 5, 0)

            pcall(function()

                if ControllerService and ControllerService.Controller and ControllerService.Controller.SetCFrame then

                    ControllerService.Controller:SetCFrame(offsetCF)

                else

                    root.CFrame = offsetCF

                end

            end)

            root.Velocity = Vector3.new(0, 0, 0)

            if Notifier and notify then

                notify({ Title = "Teleport", Content = "Character Teleported!", Duration = 3 })

            end

        end

        return

    end



    -- Teleport Logic for Vehicles



    -- Teleport Logic for Vehicles

    local x, y, z, r00, r01, r02, r10, r11, r12, r20, r21, r22 = newCF:GetComponents()

    local steering = (vehicle and vehicle.Steering) or 0

    local throttle = (vehicle and vehicle.Throttle) or 0

    local components = (vehicle and vehicle.ComponentReplicates) or {}



    -- Replicate to Server

    pcall(function()

        local n = net or GetNetwork()

        if n then

            n:FireServer("ReplicateVehicle", vehicle.UID, x, y, z, r00, r01, r02, r10, r11, r12, r20, r21, r22, steering, throttle, components)

        end

    end)



    -- Local Physics update

    if vehicle.Hitbox then vehicle.Hitbox.CFrame = newCF end

    vehicle.CFrame = newCF



    if controller._solver and controller._solver.SetState then

        controller._solver:SetState(newCF, Vector3.new(0,0,0), Vector3.new(0,0,0), components)

    end



    if Notifier and notify then

        notify({ Title = "Teleport", Content = "Vehicle Teleported!", Duration = 3 })

    else

        print("[Teleport] Vehicle Teleported!")

    end

end



DetectMapType = function()

    -- 1. Check PlaceId (Most Reliable)

    local currentId = game.PlaceId

    for name, data in pairs(Places) do

        if data[1] == currentId or data[2] == currentId then

            Config.CurrentMapName = name -- Store exact name (e.g. ZMP_NYC)



            if name:find("PVP_") then return "PVP" end

            if name:find("ZM") then return "Zombie" end

            if name:find("OW_") then return "OpenWorld" end

            if name:find("HQ_") or name:find("CM_") then return "Headquarters" end

            if name == "Menu" then return "Menu" end

        end

    end



    -- 2. Fallback: WorldService

    if not WorldService then

        return "Unknown"

    end



    -- WorldService has _world property (string name of current world)

    local placeName = WorldService._world or ""

    Config.CurrentMapName = placeName -- Fallback name



    if placeName:find("PVP_") then

        return "PVP"

    elseif placeName:find("ZM") then

        return "Zombie"

    elseif placeName:find("OW_") or placeName == "Default" then

        return "OpenWorld"

    elseif placeName:find("HQ_") or placeName:find("CM_") then

        return "Headquarters"

    elseif placeName == "Menu" then

        return "Menu"

    end



    return "Unknown"

end



ShouldShowEntity = function(actor, entityType, cachedLocalSquad)

    -- Map-specific filtering

    if Config.AutoDetectMap then

        local mapType = Config.CurrentMapType



        -- In PVP, don't show zombies

        if mapType == "PVP" and entityType == "Zombie" then

            return false

        end

    end



    -- User filter settings

    if entityType == "Player" then

        if IsTeammate(actor, cachedLocalSquad) and not Config.ShowSquadMembers then

            return false

        end

        return Config.ShowPlayers

    elseif entityType == "Zombie" then

        return Config.ShowZombies

    elseif entityType == "NPC" then

        return Config.ShowNPCs

    end



    return false

end



--[[

    ESP RENDERING (HYBRID: Highlight + Drawing API)

]]

local function CreateHighlight(actor, color)

    if not actor.Character then return nil end



    local highlight = Instance.new("Highlight")

    highlight.FillColor = color

    highlight.FillTransparency = 0.9

    highlight.OutlineColor = color

    highlight.OutlineTransparency = 0

    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop

    highlight.Parent = actor.Character



    return highlight

end



local function CreateDrawingESP()

    local esp = {

        BoxOutline = Drawing_new("Square"),

        Box = Drawing_new("Square"),

        HealthBarOutline = Drawing_new("Square"),

        HealthBar = Drawing_new("Square"),

        Name = Drawing_new("Text"),

        Distance = Drawing_new("Text")

    }



    -- Setup Defaults

    esp.BoxOutline.Visible = false

    esp.BoxOutline.Color = Color3.new(0, 0, 0)

    esp.BoxOutline.Thickness = 3

    esp.BoxOutline.Transparency = 1

    esp.BoxOutline.Filled = false

    esp.BoxOutline.ZIndex = 1



    esp.Box.Visible = false

    esp.Box.Color = Color3.new(1, 1, 1)

    esp.Box.Thickness = 1

    esp.Box.Transparency = 1

    esp.Box.Filled = false

    esp.Box.ZIndex = 2



    esp.HealthBarOutline.Visible = false

    esp.HealthBarOutline.Color = Color3.new(0, 0, 0)

    esp.HealthBarOutline.Thickness = 1

    esp.HealthBarOutline.Filled = true

    esp.HealthBarOutline.Transparency = 1

    esp.HealthBarOutline.ZIndex = 1



    esp.HealthBar.Visible = false

    esp.HealthBar.Color = Color3.new(0, 1, 0)

    esp.HealthBar.Thickness = 1

    esp.HealthBar.Filled = true

    esp.HealthBar.ZIndex = 2



    esp.Name.Visible = false

    esp.Name.Color = Color3.new(1, 1, 1)

    esp.Name.Size = 13

    esp.Name.Center = true

    esp.Name.Outline = true

    esp.Name.Font = 2 -- UI

    esp.Name.ZIndex = 3



    esp.Distance.Visible = false

    esp.Distance.Color = Color3.new(1, 1, 1)

    esp.Distance.Size = 12

    esp.Distance.Center = true

    esp.Distance.Outline = true

    esp.Distance.Font = 2

    esp.Distance.ZIndex = 3



    esp.Tracer = Drawing.new("Line")

    esp.Tracer.Visible = false

    esp.Tracer.Color = Color3.new(1, 1, 1)

    esp.Tracer.Thickness = 1

    esp.Tracer.Transparency = 1



    -- Corner Box Lines (8 lines)

    esp.Corners = {}

    for i = 1, 8 do

        local line = Drawing.new("Line")

        line.Visible = false

        line.Thickness = 1

        line.Color = Color3.new(1, 1, 1)

        table.insert(esp.Corners, line)

    end



    return esp

end



local function CreateSimpleESP(espData)

    local box = Instance.new("BoxHandleAdornment")

    box.Name = "ESPBox"

    box.AlwaysOnTop = true

    box.ZIndex = 5

    box.Transparency = 0.6

    box.Color3 = espData.Color



    -- Secure GUI Logic (Global Cached Container)

    local container = GetSecureContainer()

    if not container then return end



    box.Parent = container

    espData.SimpleESP = { Box = box, Container = container }

end







local function CreateESPForActor(actor, cachedLocalSquad)

    local entityType = GetEntityType(actor)



    if not ShouldShowEntity(actor, entityType, cachedLocalSquad) then

        return

    end



    local color = GetEntityColor(actor, entityType, cachedLocalSquad)

    local espData = {

        Actor = actor,

        EntityType = entityType,

        Color = color

    }



    -- Create visual elements

    if Config.UseHighlight then

        espData.Highlight = CreateHighlight(actor, color)

    end



    -- Always create Drawing ESP elements (managed visibility)

    espData.Drawing = CreateDrawingESP()



    -- Create Simple ESP elements (used if Config.SimpleESP is enabled)

    CreateSimpleESP(espData)



    ESPElements[actor.UID] = espData

end



-- Helper function to efficiently toggle all drawing visibility (with Caching)

local function SetAllVisible(drawings, espData, visible)

    if espData._visible == visible then return end

    espData._visible = visible



    local boxOn = visible and Config.UseBoxESP

    drawings.Box.Visible = boxOn

    drawings.BoxOutline.Visible = boxOn



    drawings.Name.Visible = visible and Config.ShowNames

    drawings.Distance.Visible = visible and Config.ShowDistance



    local healthOn = visible and Config.ShowHealth

    drawings.HealthBar.Visible = healthOn

    drawings.HealthBarOutline.Visible = healthOn



    drawings.Tracer.Visible = visible and Config.UseTracers



    -- Highlight Optimization (Roblox 31 Limit Fix)
    if espData.Highlight then
        if not visible or not Config.UseHighlight then
            pcall(function() espData.Highlight:Destroy() end)
            espData.Highlight = nil
        else
            espData.Highlight.Enabled = true
        end
    end



    -- Hide corner lines if not visible

    if not visible then

        for _, line in ipairs(drawings.Corners) do line.Visible = false end

    end

end



-- Hitbox Expander Helper

local OriginalSizes = setmetatable({}, { __mode = "k" })

local function ApplyHitboxExpander(actor, espData, cfg)

    local head = espData.CachedHead

    local enabled = cfg.HitboxExpander



    if not head or not head.Parent then

        local char = actor.Character

        -- OPTIMIZATION: actor.Parts.Head is direct engine ref (no FindFirstChild)

        head = (actor.Parts and actor.Parts.Head) or (char and char:FindFirstChild("Head"))

        espData.CachedHead = head

    end



    if not head then return end



    if enabled and actor.Alive then

        if not OriginalSizes[head] then OriginalSizes[head] = head.Size end

        local sz = cfg.HitboxSize or 4

        head.Size = Vector3_new(sz, sz, sz)

        head.Transparency = 0.5

        head.CanCollide = false

    elseif OriginalSizes[head] then

        head.Size = OriginalSizes[head]

        head.Transparency = 0

        head.CanCollide = true

        OriginalSizes[head] = nil

    end

end



local function RemoveESP(uid)
    local espData = ESPElements[uid]
    if not espData then return end

    -- Clean up Highlight
    if espData.Highlight then pcall(function() espData.Highlight:Destroy() end) end

    -- Clean up Simple ESP
    if espData.SimpleESP then
        pcall(function() espData.SimpleESP.Box:Destroy() end)
    end

    -- Clean up Drawings
    if espData.Drawing then
        local drawings = espData.Drawing
        
        -- Fix drawing ghosting by making them invisible first
        pcall(function() SetAllVisible(drawings, espData, false) end)

        if drawings.Box then pcall(function() drawings.Box.Visible = false; drawings.Box:Remove() end) end
        if drawings.BoxOutline then pcall(function() drawings.BoxOutline.Visible = false; drawings.BoxOutline:Remove() end) end
        if drawings.Name then pcall(function() drawings.Name.Visible = false; drawings.Name:Remove() end) end
        if drawings.Distance then pcall(function() drawings.Distance.Visible = false; drawings.Distance:Remove() end) end
        if drawings.HealthBar then pcall(function() drawings.HealthBar.Visible = false; drawings.HealthBar:Remove() end) end

        if drawings.HealthBarOutline then pcall(function() drawings.HealthBarOutline:Remove() end) end

        if drawings.Tracer then pcall(function() drawings.Tracer:Remove() end) end

        if drawings.Corners then

            for _, line in ipairs(drawings.Corners) do

                pcall(function() line:Remove() end)

            end

        end

    end



    ESPElements[uid] = nil

end



local function UpdateESPForActor(uid, espData, frameId, cachedCfg, localSquad)

    local actor = espData.Actor

    local drawings = espData.Drawing

    if not drawings then return end -- Safety Guard



    local entityType = actor._entityType or "Unknown"



    -- Mark as updated

    espData.LastUpdate = frameId



    -- Check if actor is still valid and Alive (Engine Check)
    if IsActorActuallyDead(actor) then
        if actor then ApplyHitboxExpander(actor, espData, cachedCfg) end -- Reset hitbox on death
        RemoveESP(uid)
        return
    end



    -- Hitbox Expander Integration

    ApplyHitboxExpander(actor, espData, cachedCfg)



    -- Bug Fix: Re-check filters for existing elements (REMOVED: Already checked in loop)

    -- if not ShouldShowEntity(actor, entityType, localSquad) then

    --    SetAllVisible(drawings, espData, false)

    --    return

    -- end



    local distance = actor.LOD_Distance or 9999



    -- PERFORMANCE: Skip inactive, too far, or engine-culled actors

    -- ViewportOnScreen: Fresh engine check for frustum visibility

    -- OnScreen: Priority flag for top 10 closest actors (even if off-frustum/occluded)

    local isVisible = actor.ViewportOnScreen or actor.OnScreen



    if actor._isInactive or (distance > cachedCfg.MaxDistance) or (not isVisible) then

        SetAllVisible(drawings, espData, false)

        if espData.SimpleESP and espData.SimpleESP.Box then

            espData.SimpleESP.Box.Visible = false

        end

        return

    end



    -- DISTANCE THROTTLING: Adaptive refresh rate based on distance

    -- 0-300: 60Hz | 300-800: 30Hz | 800+: 10Hz

    if distance > 1200 and (frameId % 6) ~= 0 then return end

    if distance > 600 and (frameId % 3) ~= 0 then return end

    if distance > 300 and (frameId % 2) ~= 0 then return end



    -- MODE: Simple ESP (BoxHandleAdornment)

    if cachedCfg.SimpleESP then

        SetAllVisible(drawings, espData, false)

        local sESP = espData.SimpleESP

        local isVisible = actor.OnScreen or actor.ViewportOnScreen or actor.LOD_OnScreen

        if distance < 50 then isVisible = true end



        if isVisible then

            local char = actor.Character

            -- OPTIMIZATION: actor.Parts.Head is direct engine ref (no FindFirstChild)

            local target = (actor.Parts and actor.Parts.Head) or (char and char:FindFirstChild("Head"))

            if target then

                local box = sESP.Box

                pcall(function()

                    box.Visible = true; box.Adornee = target;

                    box.Size = target.Size; box.Color3 = espData.Color

                end)

            end

        else

            if sESP and sESP.Box then pcall(function() sESP.Box.Visible = false end) end

        end

        return

    end



    -- OPTIMIZATION: Use engine's ViewportOnScreen + LOD_OnScreen to skip rendering

    -- Engine pre-calculates ViewportOnScreen via WorldToViewportPoint in actor:Update()

    -- LOD_OnScreen is set by ReplicatorService for the nearest 10 actors

    local engineVisible = actor.ViewportOnScreen or actor.LOD_OnScreen



    if not engineVisible then

        SetAllVisible(drawings, espData, false)

        return

    end



    -- Still need WorldToViewportPoint for precise screen coordinates

    local worldPos = actor.Position or (actor.RootPart and actor.RootPart.Position)

    if not worldPos then return end



    local cam = getgenv().LastCamCF or Camera.CFrame -- Use cached Camera CFrame

    local screen_point, onScreen = Camera:WorldToViewportPoint(worldPos)

    if not onScreen then

        SetAllVisible(drawings, espData, false)

        return

    end



    local screenX, screenY = screen_point.X, screen_point.Y

    SetAllVisible(drawings, espData, true)



    -- Update Highlight (Dinamik OluÅŸturma & Silme)
    if Config.UseHighlight then
        if not espData.Highlight then
            espData.Highlight = CreateHighlight(actor, espData.Color)
        end
        if espData.Highlight then
            espData.Highlight.Enabled = true
            espData.Highlight.FillColor = espData.Color
            espData.Highlight.OutlineColor = espData.Color
        end
    else
        if espData.Highlight then
            espData.Highlight.Enabled = false
            -- Roblox'un 31 Highlight limitinden dolayÄ±, kullanÄ±lmadÄ±ÄŸÄ±nda silmek daha gÃ¼venli
            pcall(function() espData.Highlight:Destroy() end)
            espData.Highlight = nil
        end
    end



    -- Calculate Box Dimensions (Using Engine HeightState)

    local height = 6.0                   -- Height multiplier for engine world

    local hState = actor.HeightState

    if hState == 1 then height = 4.0     -- Crouching (Enum 1)

    elseif hState == 2 then height = 2.0 -- Proning (Enum 2)

    elseif hState == 4 then height = 3.0 -- Swimming (Enum 4)

    end



    local top_point = Camera:WorldToViewportPoint(worldPos + Vector3_new(0, height * 0.5, 0))

    local bottom_point = Camera:WorldToViewportPoint(worldPos - Vector3_new(0, height * 0.5, 0))

    local scale = cachedCfg.BoxScale or 1

    local boxHeight = math_abs(bottom_point.Y - top_point.Y) * scale

    local boxWidth = boxHeight * 0.6



    local bpX, bpY = screenX - boxWidth * 0.5, screenY - boxHeight * 0.5

    local boxPosition = Vector2_new(bpX, bpY) -- Fix for Health Bar Error



    -- Update Drawings



    -- Determine box color (override or entity-based)

    local boxColor = cachedCfg.BoxColor or espData.Color



    -- Box

    if cachedCfg.UseBoxESP then

        if cachedCfg.BoxStyle == "Corner" then

            -- Corner box style - hide full box, show corner lines

            drawings.Box.Visible = false

            drawings.BoxOutline.Visible = false



            local cornerLength = (boxWidth < boxHeight and boxWidth or boxHeight) * 0.25

            local corners = drawings.Corners



            -- OPTIMIZATION: Cache intermediate coordinates

            local rightX = bpX + boxWidth

            local bottomY = bpY + boxHeight

            local topLeft = Vector2_new(bpX, bpY)

            local topRight = Vector2_new(rightX, bpY)

            local bottomLeft = Vector2_new(bpX, bottomY)

            local bottomRight = Vector2_new(rightX, bottomY)



            -- Top-left corner

            corners[1].From = topLeft

            corners[1].To = Vector2_new(bpX + cornerLength, bpY)

            corners[1].Color = boxColor

            corners[1].Visible = true



            corners[2].From = topLeft

            corners[2].To = Vector2_new(bpX, bpY + cornerLength)

            corners[2].Color = boxColor

            corners[2].Visible = true



            -- Top-right corner

            corners[3].From = topRight

            corners[3].To = Vector2_new(rightX - cornerLength, bpY)

            corners[3].Color = boxColor

            corners[3].Visible = true



            corners[4].From = topRight

            corners[4].To = Vector2_new(rightX, bpY + cornerLength)

            corners[4].Color = boxColor

            corners[4].Visible = true



            -- Bottom-left corner

            corners[5].From = bottomLeft

            corners[5].To = Vector2_new(bpX + cornerLength, bottomY)

            corners[5].Color = boxColor

            corners[5].Visible = true



            corners[6].From = bottomLeft

            corners[6].To = Vector2_new(bpX, bottomY - cornerLength)

            corners[6].Color = boxColor

            corners[6].Visible = true



            -- Bottom-right corner

            corners[7].From = bottomRight

            corners[7].To = Vector2_new(rightX - cornerLength, bottomY)

            corners[7].Color = boxColor

            corners[7].Visible = true



            corners[8].From = bottomRight

            corners[8].To = Vector2_new(rightX, bottomY - cornerLength)

            corners[8].Color = boxColor

            corners[8].Visible = true

        else

            -- Full box style

            -- Hide corner lines

            local corners = drawings.Corners

            if corners[1].Visible then

                for _, line in ipairs(corners) do

                    line.Visible = false

                end

            end



            drawings.BoxOutline.Size = Vector2_new(boxWidth, boxHeight)

            drawings.BoxOutline.Position = Vector2_new(bpX, bpY)

            drawings.BoxOutline.Visible = true



            drawings.Box.Size = Vector2_new(boxWidth, boxHeight)

            drawings.Box.Position = Vector2_new(bpX, bpY)

            drawings.Box.Color = boxColor

            drawings.Box.Visible = true

        end

    else

        drawings.Box.Visible = false

        drawings.BoxOutline.Visible = false

        local corners = drawings.Corners

        if corners[1].Visible then

            for _, line in ipairs(corners) do

                line.Visible = false

            end

        end

    end



    -- Name

    if cachedCfg.ShowNames then

        local displayName = actor.OwnerName

        if entityType == "Zombie" then

            local h = actor.Health

            if h and type(h) == "table" and h.Ability then

                local abilityNames = { "Crippled", "Slow", "Normal", "Sprinter" }

                displayName = abilityNames[h.Ability] or "Zombie"

            else

                displayName = "Zombie"

            end

        elseif not displayName or displayName == "???" then

            displayName = "AI"

        end



        local dynamicScale = cachedCfg.DynamicTextScaling and math_clamp(boxHeight * 0.016, 0.5, 1.0) or 1.0

        local nameSize = 13 * cachedCfg.TextScale * dynamicScale



        drawings.Name.Text = displayName

        drawings.Name.Position = Vector2_new(screenX, bpY - (nameSize + 3))

        drawings.Name.Color = espData.Color

        drawings.Name.Size = nameSize

        drawings.Name.Visible = true

    else

        drawings.Name.Visible = false

    end



    -- Distance (Meters)

    if cachedCfg.ShowDistance then

        local dynamicScale = cachedCfg.DynamicTextScaling and math_clamp(boxHeight * 0.016, 0.5, 1.0) or 1.0

        local distSize = 12 * cachedCfg.TextScale * dynamicScale



        drawings.Distance.Text = string_format("[%dm]", math_floor(distance * 0.28))

        drawings.Distance.Size = distSize



        -- Adjust position based on Health Bar if at Bottom

        local distY = bpY + boxHeight + 2

        if cachedCfg.HealthBarSide == "Bottom" then

            distY = distY + 6 -- Make room for bar

        end



        drawings.Distance.Position = Vector2_new(screenX, distY)

        drawings.Distance.Visible = true

    else

        drawings.Distance.Visible = false

    end



    -- Health Bar

    if cachedCfg.ShowHealth and actor.Health then

        local healthPercent = 1

        local hp = actor.Health

        if type(hp) == "number" then

            healthPercent = math_clamp(hp * 0.01, 0, 1)

        end



        local barColor = cachedCfg.HealthBarColor or Color3_new(0, 1, 0)

        if cachedCfg.UseHealthGradient then

            barColor = Color3_fromHSV(healthPercent * 0.3, 1, 1) -- Green to Red

        end



        local barSize, barPos, outlineSize, outlinePos

        local hbSide = cachedCfg.HealthBarSide



        if hbSide == "Bottom" then

            local barW = boxWidth * healthPercent

            barSize = Vector2_new(barW, 4)

            barPos = Vector2_new(boxPosition and boxPosition.X or 0, boxPosition and boxPosition.Y + boxHeight + 2 or 0)

            outlineSize = Vector2_new(boxWidth + 2, 6)

            outlinePos = Vector2_new(boxPosition and boxPosition.X - 1 or 0, boxPosition and boxPosition.Y + boxHeight + 1 or 0)

        elseif hbSide == "Right" then

            local barH = boxHeight * healthPercent

            barSize = Vector2_new(2, barH)

            barPos = Vector2_new(boxPosition and boxPosition.X + boxWidth + 3 or 0, boxPosition and boxPosition.Y + (boxHeight - barH) or 0)

            outlineSize = Vector2_new(4, boxHeight + 2)

            outlinePos = Vector2_new(boxPosition and boxPosition.X + boxWidth + 2 or 0, boxPosition and boxPosition.Y - 1 or 0)

        else

            local barH = boxHeight * healthPercent

            barSize = Vector2_new(2, barH)

            barPos = Vector2_new(boxPosition and boxPosition.X - 5 or 0, boxPosition and boxPosition.Y + (boxHeight - barH) or 0)

            outlineSize = Vector2_new(4, boxHeight + 2)

            outlinePos = Vector2_new(boxPosition and boxPosition.X - 6 or 0, boxPosition and boxPosition.Y - 1 or 0)

        end



        drawings.HealthBarOutline.Size = outlineSize

        drawings.HealthBarOutline.Position = outlinePos

        drawings.HealthBarOutline.Visible = true



        drawings.HealthBar.Size = barSize

        drawings.HealthBar.Position = barPos

        drawings.HealthBar.Color = barColor

        drawings.HealthBar.Visible = true

    else

        drawings.HealthBar.Visible = false

        drawings.HealthBarOutline.Visible = false

    end



    -- Tracers

    if cachedCfg.UseTracers then

        local viewport = Camera.ViewportSize

        drawings.Tracer.From = Vector2_new(viewport.X * 0.5, viewport.Y)

        drawings.Tracer.To = Vector2_new(screenX, bpY + boxHeight)

        drawings.Tracer.Color = espData.Color

        drawings.Tracer.Visible = true

    else

        drawings.Tracer.Visible = false

    end

end



--[[

    VEHICLE UPDATE LOOP

    Iterates over VehicleService.Vehicles (Engine Table)

]]

local function HideVehicleESP(espData)

    local drawings = espData.Drawing

    drawings.Box.Visible = false

    drawings.BoxOutline.Visible = false

    drawings.Name.Visible = false

    drawings.Distance.Visible = false

    drawings.HealthBar.Visible = false

    drawings.HealthBarOutline.Visible = false

    if drawings.Corners then for _, l in ipairs(drawings.Corners) do l.Visible = false end end

end



local function UpdateESPForVehicle(uid, espData, vehicle, currentFrameId)

    local pos = nil

    -- OPTIMIZATION: vehicle.CFrame.Position is engine-maintained; avoid PrimaryPart lookup

    pos = (vehicle.Model.PrimaryPart and vehicle.Model.PrimaryPart.Position) or vehicle.CFrame.Position



    -- OPTIMIZATION: Cache camera position for vehicle ESP distance calc

    local camPos = Camera.CFrame.Position

    local dv = camPos - pos

    local dist = math.sqrt(dv.X * dv.X + dv.Y * dv.Y + dv.Z * dv.Z) -- Faster than .Magnitude



    if dist > Config.MaxDistance then

        HideVehicleESP(espData)

        return

    end



    local screenPos, onScreen = Camera:WorldToViewportPoint(pos)



    if not onScreen then

        HideVehicleESP(espData)

        return

    end



    espData.LastUpdate = currentFrameId



    -- OPTIMIZATION: Cache vehicle extents size (expensive call)

    if not espData._cachedSize or (currentFrameId % 60 == 0) then

        espData._cachedSize = vehicle.Model:GetExtentsSize()

    end

    local size = espData._cachedSize

    local height = size.Y

    local width = math.max(size.X, size.Z)

    local boxSize = Vector2.new(1000 / dist * width, 1000 / dist * height) * (Config.BoxScale or 1.0)

    local boxPos = Vector2.new(screenPos.X - boxSize.X / 2, screenPos.Y - boxSize.Y / 2)

    local boxHeight = boxSize.Y

    local boxWidth = boxSize.X



    local drawings = espData.Drawing

    local boxColor = Config.VehicleColor or Color3.new(0, 1, 1)



    -- BOX

    if Config.ShowVehicleBox then

        -- Update Outline First (Background) - Size offset to prevent occlusion

        drawings.BoxOutline.Color = Color3.new(0, 0, 0)

        drawings.BoxOutline.Transparency = 1

        drawings.BoxOutline.Thickness = 1 -- Back to 1px

        drawings.BoxOutline.Size = boxSize + Vector2.new(2, 2)

        drawings.BoxOutline.Position = boxPos - Vector2.new(1, 1)

        drawings.BoxOutline.ZIndex = 0



        -- Update Main Box Second (Foreground)

        drawings.Box.Color = boxColor

        drawings.Box.Transparency = 1

        drawings.Box.Thickness = 1 -- Back to 1px

        drawings.Box.Size = boxSize

        drawings.Box.Position = boxPos

        drawings.Box.ZIndex = 5



        if Config.VehicleBoxStyle == "Corner" then

            drawings.Box.Visible = false

            drawings.BoxOutline.Visible = false



            if drawings.Corners then

                local cornerLength = math.min(boxWidth, boxHeight) * 0.25

                local c = drawings.Corners



                -- TL

                c[1].From = Vector2.new(boxPos.X, boxPos.Y); c[1].To = Vector2.new(boxPos.X + cornerLength, boxPos.Y)

                c[2].From = Vector2.new(boxPos.X, boxPos.Y); c[2].To = Vector2.new(boxPos.X, boxPos.Y + cornerLength)

                -- TR

                c[3].From = Vector2.new(boxPos.X + boxWidth, boxPos.Y); c[3].To = Vector2.new(

                    boxPos.X + boxWidth - cornerLength, boxPos.Y)

                c[4].From = Vector2.new(boxPos.X + boxWidth, boxPos.Y); c[4].To = Vector2.new(boxPos.X + boxWidth,

                    boxPos.Y + cornerLength)

                -- BL

                c[5].From = Vector2.new(boxPos.X, boxPos.Y + boxHeight); c[5].To = Vector2.new(boxPos.X + cornerLength,

                    boxPos.Y + boxHeight)

                c[6].From = Vector2.new(boxPos.X, boxPos.Y + boxHeight); c[6].To = Vector2.new(boxPos.X,

                    boxPos.Y + boxHeight - cornerLength)

                -- BR

                c[7].From = Vector2.new(boxPos.X + boxWidth, boxPos.Y + boxHeight); c[7].To = Vector2.new(

                    boxPos.X + boxWidth - cornerLength, boxPos.Y + boxHeight)

                c[8].From = Vector2.new(boxPos.X + boxWidth, boxPos.Y + boxHeight); c[8].To = Vector2.new(

                    boxPos.X + boxWidth, boxPos.Y + boxHeight - cornerLength)



                for _, l in ipairs(c) do

                    l.Color = boxColor

                    l.Transparency = 1

                    l.Thickness = 1

                    l.ZIndex = 5

                    l.Visible = true

                end

            end

        else

            if drawings.Corners then for _, l in pairs(drawings.Corners) do l.Visible = false end end



            drawings.BoxOutline.Visible = true

            drawings.Box.Visible = true

        end

    else

        drawings.Box.Visible = false

        drawings.BoxOutline.Visible = false

        if drawings.Corners then for _, l in pairs(drawings.Corners) do l.Visible = false end end

    end



    -- NAME

    if Config.ShowVehicleName then

        local dynamicScale = Config.DynamicTextScaling and math.clamp(boxHeight / 60, 0.5, 1.0) or 1.0

        local nameSize = 13 * (Config.TextScale or 1) * dynamicScale



        local name = vehicle._vehicleName or (vehicle.Model and vehicle.Model.Name) or "Vehicle"

        drawings.Name.Text = name

        drawings.Name.Size = nameSize

        drawings.Name.Position = Vector2.new(screenPos.X, boxPos.Y - (nameSize + 3))

        drawings.Name.Color = boxColor

        drawings.Name.ZIndex = 3

        drawings.Name.Outline = true

        drawings.Name.Visible = true

    else

        drawings.Name.Visible = false

    end



    -- DISTANCE

    if Config.ShowDistance then

        local dynamicScale = Config.DynamicTextScaling and math.clamp(boxHeight / 60, 0.5, 1.0) or 1.0

        local distSize = 12 * (Config.TextScale or 1) * dynamicScale



        local meters = math.floor(dist * 0.28) -- Convert studs to meters approx

        drawings.Distance.Text = string.format("[%dm]", meters)

        drawings.Distance.Size = distSize

        drawings.Distance.Color = boxColor

        drawings.Distance.ZIndex = 3

        drawings.Distance.Outline = true



        -- Position at bottom, adjusting for health bar if needed

        local distY = boxPos.Y + boxHeight + 2

        if Config.ShowVehicleHealth and Config.VehicleHealthBarSide == "Bottom" then

            distY = distY + 6

        end



        drawings.Distance.Position = Vector2.new(screenPos.X, distY)

        drawings.Distance.Visible = true

    else

        drawings.Distance.Visible = false

    end



    -- HEALTH

    if Config.ShowVehicleHealth and vehicle.Healths then

        local hpPercent = 1

        local hull = vehicle.Healths.Hull

        local engine = vehicle.Healths.Engine



        if hull then

            -- Fetch actual Max Health from Game Configs

            local maxHull = 100

            local armor = vehicle.VehicleArmor

            local module = vehicle.VehicleModule

            if module and module.Armor and armor and module.Armor[armor] then

                maxHull = module.Armor[armor].Health or 100

            end



            hpPercent = math.clamp(hull / maxHull, 0, 1)



            -- Engine health check (lower of the two)

            if engine then

                local maxEng = 100

                if module and module.Armor and armor and module.Armor[armor] then

                    maxEng = module.Armor[armor].EngineHealth or 100

                elseif BulletService and BulletService.ENGINE_HEALTH then

                    maxEng = BulletService.ENGINE_HEALTH

                end



                local engPercent = math.clamp(engine / maxEng, 0, 1)

                if engPercent < hpPercent then

                    hpPercent = engPercent

                end

            end

        else

            -- Fallback for vehicles without standard hull (rare in BRM5)

            local currentHp = 0

            local maxHp = 0

            for _, v in pairs(vehicle.Healths) do

                if type(v) == "number" then

                    currentHp = currentHp + v; maxHp = maxHp + 100

                end

            end

            if maxHp > 0 then hpPercent = math.clamp(currentHp / maxHp, 0, 1) end

        end



        local barColor = Config.VehicleHealthColor or Color3.new(0, 1, 0)

        if Config.UseHealthGradient then

            barColor = Color3.fromHSV(hpPercent * 0.3, 1, 1) -- Red-Green gradient

        end



        local barSize, barPos, outlineSize, outlinePos



        if Config.VehicleHealthBarSide == "Bottom" then

            local barW = boxWidth * hpPercent

            barSize = Vector2.new(barW, 4)

            barPos = Vector2.new(boxPos.X, boxPos.Y + boxHeight + 2)

            outlineSize = Vector2.new(boxPos.X - 1, boxPos.Y + boxHeight + 1)

        elseif Config.VehicleHealthBarSide == "Right" then

            local barW = 2

            local barH = boxHeight * hpPercent

            barSize = Vector2.new(barW, barH)

            barPos = Vector2.new(boxPos.X + boxWidth + 3, boxPos.Y + (boxHeight - barH))

            outlineSize = Vector2.new(barW + 2, boxHeight + 2)

            outlinePos = Vector2.new(boxPos.X + boxWidth + 2, boxPos.Y - 1)

        else -- Left

            local barW = 2

            local barH = boxHeight * hpPercent

            barSize = Vector2.new(barW, barH)

            barPos = Vector2.new(boxPos.X - 5, boxPos.Y + (boxHeight - barH))

            outlineSize = Vector2.new(barW + 2, boxHeight + 2)

            outlinePos = Vector2.new(boxPos.X - 6, boxPos.Y - 1)

        end



        -- Draw Outline First (Background) - Size offset

        drawings.HealthBarOutline.Size = outlineSize + Vector2.new(2, 2)

        drawings.HealthBarOutline.Position = outlinePos - Vector2.new(1, 1)

        drawings.HealthBarOutline.Color = Color3.new(0, 0, 0)

        drawings.HealthBarOutline.Transparency = 1

        drawings.HealthBarOutline.Thickness = 1

        drawings.HealthBarOutline.ZIndex = 0

        drawings.HealthBarOutline.Visible = true



        -- Draw Bar Second (Foreground)

        drawings.HealthBar.Size = barSize

        drawings.HealthBar.Position = barPos

        drawings.HealthBar.Color = barColor

        drawings.HealthBar.Transparency = 1

        drawings.HealthBar.ZIndex = 5

        drawings.HealthBar.Visible = true

    else

        drawings.HealthBar.Visible = false

        drawings.HealthBarOutline.Visible = false

    end

end



local function UpdateVehicles(frameId, cachedCfg)

    if not Config.ShowVehicles then return end

    if not ActorManager.Initialized then return end



    local localActor = ReplicatorService and ReplicatorService.LocalActor

    local myVehUID = nil

    if localActor then

        -- Correct UID detection from Seat/Turret

        if localActor.Seat then

            myVehUID = localActor.Seat.UID or (localActor.Seat.Model and localActor.Seat.Model.Name)

        end

        if not myVehUID and localActor.Turret and localActor.Turret._uid then

            -- Turret Controller UID is often vehicleUID_TurretIndex, but we check _vehicle ref

            if localActor.Turret._vehicle then

                myVehUID = localActor.Turret._vehicle.UID

            else

                myVehUID = localActor.Turret._uid:split("_")[1]

            end

        end

    end



    for _, actor in ipairs(ActorManager.Vehicles) do

        local uid = "Veh_" .. tostring(actor.UID)

        local espData = ESPElements[uid]



        -- Ignore Local Vehicle Logic

        local isLocal = (myVehUID == actor.UID)

        if Config.IgnoreLocalVehicle and isLocal then

            if espData and espData._visible ~= false then

                HideVehicleESP(espData)

                if espData.Highlight then espData.Highlight.Enabled = false end

                espData._visible = false

            end

            continue

        end



        -- Create if missing

        if not espData then

            espData = {

                Drawing = CreateDrawingESP(),

                Type = "Vehicle",

                LastUpdate = frameId,

                _visible = true

            }

            ESPElements[uid] = espData

        end



        espData._visible = true

        UpdateESPForVehicle(actor.UID, espData, actor, frameId)

    end

end



-- IsEnemy helper for Hitbox Expander

local function IsEnemyForExpander(player)

    if not player or not LocalPlayer then return false end

    -- If using team system

    if player.Team and LocalPlayer.Team and player.Team == LocalPlayer.Team then return false end

    -- Fallback/Safety: consider everyone else enemy if not same team

    return true

end



-- Hitbox expander merged into ESP loop



-- Server Hide Hook (Spoof Size property)

task.spawn(function()

    if not getrawmetatable then return end

    local mt = getrawmetatable(game)

    setreadonly(mt, false)

    local oldIndex = mt.__index



    mt.__index = newcclosure(function(self, key)

        if key == "Size" and Config.HitboxExpander and OriginalSizes[self] then

            return OriginalSizes[self]

        end



        if type(oldIndex) == "function" then

            local success, result = pcall(oldIndex, self, key)

            if success then return result end

        elseif type(oldIndex) == "table" then

            local success, result = pcall(function() return oldIndex[key] end)

            if success then return result end

        end

        return nil

    end)

    setreadonly(mt, true)

end)



-- CRASH MITIGATION V3: Robust Hook for ActorClass.Update

-- Prevents the error: "ActorClass:2080: attempt to perform arithmetic (div) on nil and number"

-- This hook uses a global pcall wrapper to suppress engine crashes during any state (including Climbing).

task.spawn(function()

    while not ActorClass do task.wait(0.5) end

    if BulletService and not BulletService._CrashMitigationAppliedV3 then

        local oldUpdate = ActorClass.Update

        ActorClass.Update = function(self, ...)

            local success, v1, v2 = pcall(oldUpdate, self, ...)

            if not success then

                -- Safely suppress the crash and log once to console for visibility

                if not self._crashReported then

                    warn("ANTICRASH: ActorClass:Update failed safely (suppressed engine crash). Move/ESP may stutter for this actor.")

                    self._crashReported = true

                end

                return nil, nil -- Safe return for ReplicatorService bulk move

            end

            return v1, v2

        end

        BulletService._CrashMitigationAppliedV3 = true

        warn("ANTICRASH: Applied robust ActorClass:Update mitigation hook (V3).")

    end

end)



-- OPTIMIZATION: Module-level cached config (avoid per-frame table allocation)

local _cachedCfg = {}

local _cachedCfgFrame = -30



local function RefreshCachedCfg()

    _cachedCfg.MaxDistance = Config.MaxDistance or 1000

    _cachedCfg.UseBoxESP = Config.UseBoxESP

    _cachedCfg.BoxStyle = Config.BoxStyle

    _cachedCfg.BoxScale = Config.BoxScale or 1

    _cachedCfg.BoxColor = Config.BoxColor

    _cachedCfg.ShowNames = Config.ShowNames

    _cachedCfg.ShowDistance = Config.ShowDistance

    _cachedCfg.ShowHealth = Config.ShowHealth

    _cachedCfg.HealthBarSide = Config.HealthBarSide

    _cachedCfg.HealthBarColor = Config.HealthBarColor

    _cachedCfg.UseHealthGradient = Config.UseHealthGradient

    _cachedCfg.UseTracers = Config.UseTracers

    _cachedCfg.UseHighlight = Config.UseHighlight

    _cachedCfg.SimpleESP = Config.SimpleESP

    _cachedCfg.TextScale = (Config.TextScale or 1.2)

    _cachedCfg.DynamicTextScaling = true

    _cachedCfg.HitboxExpander = Config.HitboxExpander

    _cachedCfg.HitboxSize = Config.HitboxSize

end



local function UpdateESP()

    if not ESPEnabled and not Config.HitboxExpander then return end

    if not ReplicatorService then return end



    CurrentFrameId = CurrentFrameId + 1

    -- Robust Squad Detection (Fixes "index number with Squad" error)

    local lClient = ClientService and ClientService.LocalClient

    local localSquad = nil

    if type(lClient) == "table" and lClient.Squad and type(lClient.Squad) == "table" then

        localSquad = lClient.Squad

    end



    -- OPTIMIZATION: Refresh config cache every 30 frames instead of every frame

    if CurrentFrameId - _cachedCfgFrame >= 30 then

        RefreshCachedCfg()

        _cachedCfgFrame = CurrentFrameId

    end

    local cachedCfg = _cachedCfg



    -- Process Actor Lists

    local function Process(list)

        for _, actor in ipairs(list) do

            -- Early exit if hidden (Pre-calculated in ActorManager)

            if actor._cachedShow == false then

                if ESPElements[actor.UID] then RemoveESP(actor.UID) end

                continue

            end



            local uid = actor.UID

            if not uid then continue end

            if not ESPElements[uid] then CreateESPForActor(actor, localSquad) end

            local espData = ESPElements[uid]

            if espData then

                -- Throttle Color Updates

                if (CurrentFrameId % 30) == 0 then

                    espData.Color = GetEntityColor(actor, actor._entityType, localSquad)

                end

                UpdateESPForActor(uid, espData, CurrentFrameId, cachedCfg, localSquad)

            end

        end

    end



    if ActorManager.Initialized then

        Process(ActorManager.Enemies)

        Process(ActorManager.Teammates)

    end



    -- STABILITY FIX: Proactive cleanup of dead/invalid actors (prevents stuck ESP)

    if CurrentFrameId % 5 == 0 then

        for uid, espData in pairs(ESPElements) do

            local actor = espData.Actor

            -- If actor is gone, dead, or hasn't updated in over 2 frames, remove it

            if not actor or not actor.Alive or not actor.Character or (espData.LastUpdate < CurrentFrameId - 2) then

                RemoveESP(uid)

            end

        end

    end

end





-- [[ UI SECTION ]]

local UIS = game:GetService("UserInputService")

local TOGGLE_KEY = Enum.KeyCode.RightControl



-- Turret Crosshair & Zoom Logic
-- Turret Crosshair & Zoom Logic
local TurretCH_Lines = {Drawing.new("Line"), Drawing.new("Line"), Drawing.new("Line"), Drawing.new("Line"), Drawing.new("Line"), Drawing.new("Line"), Drawing.new("Line"), Drawing.new("Line")}
local TurretCH_Circle = Drawing.new("Circle")
local TurretCH_Triangle = {Drawing.new("Line"), Drawing.new("Line"), Drawing.new("Line")}

-- Mouse Wheel Scroll for Global Turret Zoom
local ZoomConnection = nil
local function SetupZoomScroll()
    if ZoomConnection then pcall(function() ZoomConnection:Disconnect() end) end
    ZoomConnection = UserInputService.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseWheel and Config.Turret_ZoomEnabled then
            local delta = input.Position.Z
            local current = Config.Turret_ZoomValue or 70
            if delta > 0 then
                Config.Turret_ZoomValue = math.clamp(current - 10, 1, 120)
            else
                Config.Turret_ZoomValue = math.clamp(current + 10, 1, 120)
            end
        end
    end)
    table.insert(getgenv().BlackhawkESP_Connections, ZoomConnection)
end
SetupZoomScroll()

local function UpdateTurretCH()
    local actor = ReplicatorService and ReplicatorService.LocalActor
    local turret = actor and actor.Turret
    local isVisible = false
    local drawPos = UserInputService:GetMouseLocation()

    if Config.TurretCH_Enabled and turret and type(turret) == "table" then
        isVisible = true
        pcall(function()
            if turret._turret and turret._turret.Sight then
                local sight = turret._turret.Sight
                local pos, onScreen = Camera:WorldToViewportPoint(sight.WorldPosition + sight.WorldCFrame.LookVector * 1000)
                if onScreen then
                    drawPos = Vector2.new(pos.X, pos.Y)
                end
            end
        end)
    end

    TurretCH_Circle.Visible = false
    for i = 1, 8 do TurretCH_Lines[i].Visible = false end
    for i = 1, 3 do TurretCH_Triangle[i].Visible = false end

    if isVisible then
        local color = Config.TurretCH_Color or Color3.fromRGB(0, 255, 0)
        local style = Config.TurretCH_Style or "Cross"
        local size = Config.TurretCH_Size or 10
        local gap = Config.TurretCH_Gap or 3
        local thick = Config.TurretCH_Thickness or 1.5

        if style == "Cross" then
            TurretCH_Lines[1].Visible = true; TurretCH_Lines[1].From = drawPos - Vector2.new(0, gap); TurretCH_Lines[1].To = drawPos - Vector2.new(0, gap + size)
            TurretCH_Lines[2].Visible = true; TurretCH_Lines[2].From = drawPos + Vector2.new(0, gap); TurretCH_Lines[2].To = drawPos + Vector2.new(0, gap + size)
            TurretCH_Lines[3].Visible = true; TurretCH_Lines[3].From = drawPos - Vector2.new(gap, 0); TurretCH_Lines[3].To = drawPos - Vector2.new(gap + size, 0)
            TurretCH_Lines[4].Visible = true; TurretCH_Lines[4].From = drawPos + Vector2.new(gap, 0); TurretCH_Lines[4].To = drawPos + Vector2.new(gap + size, 0)
        elseif style == "X-Cross" then
            local offset = (gap + size) * 0.707; local goffset = gap * 0.707
            TurretCH_Lines[1].Visible = true; TurretCH_Lines[1].From = drawPos - Vector2.new(goffset, goffset); TurretCH_Lines[1].To = drawPos - Vector2.new(offset, offset)
            TurretCH_Lines[2].Visible = true; TurretCH_Lines[2].From = drawPos + Vector2.new(goffset, goffset); TurretCH_Lines[2].To = drawPos + Vector2.new(offset, offset)
            TurretCH_Lines[3].Visible = true; TurretCH_Lines[3].From = drawPos + Vector2.new(-goffset, goffset); TurretCH_Lines[3].To = drawPos + Vector2.new(-offset, offset)
            TurretCH_Lines[4].Visible = true; TurretCH_Lines[4].From = drawPos + Vector2.new(goffset, -goffset); TurretCH_Lines[4].To = drawPos + Vector2.new(offset, -offset)
        elseif style == "Circle" then
            TurretCH_Circle.Visible = true; TurretCH_Circle.Position = drawPos; TurretCH_Circle.Color = color; TurretCH_Circle.Radius = size; TurretCH_Circle.Thickness = thick; TurretCH_Circle.Filled = false
        elseif style == "Dot" then
            TurretCH_Circle.Visible = true; TurretCH_Circle.Position = drawPos; TurretCH_Circle.Color = color; TurretCH_Circle.Radius = thick * 2; TurretCH_Circle.Filled = true
        elseif style == "T-Shape" then
            TurretCH_Lines[2].Visible = true; TurretCH_Lines[2].From = drawPos + Vector2.new(0, gap); TurretCH_Lines[2].To = drawPos + Vector2.new(0, gap + size)
            TurretCH_Lines[3].Visible = true; TurretCH_Lines[3].From = drawPos - Vector2.new(gap, 0); TurretCH_Lines[3].To = drawPos - Vector2.new(gap + size, 0)
            TurretCH_Lines[4].Visible = true; TurretCH_Lines[4].From = drawPos + Vector2.new(gap, 0); TurretCH_Lines[4].To = drawPos + Vector2.new(gap + size, 0)
        elseif style == "Square" then
            TurretCH_Lines[1].Visible = true; TurretCH_Lines[1].From = drawPos + Vector2.new(-size, -size); TurretCH_Lines[1].To = drawPos + Vector2.new(size, -size)
            TurretCH_Lines[2].Visible = true; TurretCH_Lines[2].From = drawPos + Vector2.new(size, -size); TurretCH_Lines[2].To = drawPos + Vector2.new(size, size)
            TurretCH_Lines[3].Visible = true; TurretCH_Lines[3].From = drawPos + Vector2.new(size, size); TurretCH_Lines[3].To = drawPos + Vector2.new(-size, size)
            TurretCH_Lines[4].Visible = true; TurretCH_Lines[4].From = drawPos + Vector2.new(-size, size); TurretCH_Lines[4].To = drawPos + Vector2.new(-size, -size)
        elseif style == "HUD" then
            local hSize = size * 1.5
            TurretCH_Lines[1].Visible = true; TurretCH_Lines[1].From = drawPos + Vector2.new(-hSize, -size); TurretCH_Lines[1].To = drawPos + Vector2.new(-hSize + 5, -size)
            TurretCH_Lines[2].Visible = true; TurretCH_Lines[2].From = drawPos + Vector2.new(-hSize, -size); TurretCH_Lines[2].To = drawPos + Vector2.new(-hSize, -size + 5)
            TurretCH_Lines[3].Visible = true; TurretCH_Lines[3].From = drawPos + Vector2.new(hSize, -size); TurretCH_Lines[3].To = drawPos + Vector2.new(hSize - 5, -size)
            TurretCH_Lines[4].Visible = true; TurretCH_Lines[4].From = drawPos + Vector2.new(hSize, -size); TurretCH_Lines[4].To = drawPos + Vector2.new(hSize, -size + 5)
            TurretCH_Lines[5].Visible = true; TurretCH_Lines[5].From = drawPos + Vector2.new(-hSize, size); TurretCH_Lines[5].To = drawPos + Vector2.new(-hSize + 5, size)
            TurretCH_Lines[6].Visible = true; TurretCH_Lines[6].From = drawPos + Vector2.new(-hSize, size); TurretCH_Lines[6].To = drawPos + Vector2.new(-hSize, size - 5)
            TurretCH_Lines[7].Visible = true; TurretCH_Lines[7].From = drawPos + Vector2.new(hSize, size); TurretCH_Lines[7].To = drawPos + Vector2.new(hSize - 5, size)
            TurretCH_Lines[8].Visible = true; TurretCH_Lines[8].From = drawPos + Vector2.new(hSize, size); TurretCH_Lines[8].To = drawPos + Vector2.new(hSize, size - 5)
        elseif style == "Triangle" then
            TurretCH_Triangle[1].Visible = true; TurretCH_Triangle[1].From = drawPos - Vector2.new(0, gap + size); TurretCH_Triangle[1].To = drawPos + Vector2.new(-size, gap)
            TurretCH_Triangle[2].Visible = true; TurretCH_Triangle[2].From = drawPos + Vector2.new(-size, gap); TurretCH_Triangle[2].To = drawPos + Vector2.new(size, gap)
            TurretCH_Triangle[3].Visible = true; TurretCH_Triangle[3].From = drawPos + Vector2.new(size, gap); TurretCH_Triangle[3].To = drawPos - Vector2.new(0, gap + size)
        elseif style == "Arrow" then
            TurretCH_Lines[1].Visible = true; TurretCH_Lines[1].From = drawPos - Vector2.new(0, gap); TurretCH_Lines[1].To = drawPos - Vector2.new(0, gap + size)
            TurretCH_Lines[2].Visible = true; TurretCH_Lines[2].From = drawPos - Vector2.new(0, gap + size); TurretCH_Lines[2].To = drawPos - Vector2.new(size/2, gap + size/2)
            TurretCH_Lines[3].Visible = true; TurretCH_Lines[3].From = drawPos - Vector2.new(0, gap + size); TurretCH_Lines[3].To = drawPos - Vector2.new(-size/2, gap + size/2)
        end
        for i = 1, 8 do if TurretCH_Lines[i].Visible then TurretCH_Lines[i].Color = color; TurretCH_Lines[i].Thickness = thick end end
        for i = 1, 3 do if TurretCH_Triangle[i].Visible then TurretCH_Triangle[i].Color = color; TurretCH_Triangle[i].Thickness = thick end end
    end
end

-- Hook Vehicle Controllers for Speed
local function SetupVehicleHooks()
    local gc = GroundController
    if gc and gc.Update and not gc._speedHooked then
        gc._origUpdate = gc.Update
        gc.Update = function(self, ...)
            if Config.VehicleSpeedMultiplier and Config.VehicleSpeedMultiplier > 0 then
                if self._tune then
                    if not self._tune._origAccel then self._tune._origAccel = self._tune.AccelerationFactor or 1 end
                    if not self._tune._origSpeed then self._tune._origSpeed = self._tune.MaxSpeed or 1 end
                    if not self._tune._origRSpeed then self._tune._origRSpeed = self._tune.ReverseMaxSpeed or 1 end
                    self._tune.AccelerationFactor = self._tune._origAccel * Config.VehicleSpeedMultiplier
                    self._tune.MaxSpeed = self._tune._origSpeed * Config.VehicleSpeedMultiplier
                    self._tune.ReverseMaxSpeed = self._tune._origRSpeed * Config.VehicleSpeedMultiplier
                    if self._solver and self._solver.NewTune then pcall(function() self._solver:NewTune() end) end
                end
            elseif self._tune and self._tune._origAccel then
                self._tune.AccelerationFactor = self._tune._origAccel
                self._tune.MaxSpeed = self._tune._origSpeed
                self._tune.ReverseMaxSpeed = self._tune._origRSpeed
                if self._solver and self._solver.NewTune then pcall(function() self._solver:NewTune() end) end
            end
            return gc._origUpdate(self, ...)
        end
        gc._speedHooked = true
    end

    local hc = HelicopterController
    if hc and hc.Update and not hc._speedHooked then
        hc._origUpdate = hc.Update
        hc.Update = function(self, ...)
            if Config.VehicleSpeedMultiplier and Config.VehicleSpeedMultiplier > 0 then
                if self._tune then
                    if not self._tune._origFlightAccel then self._tune._origFlightAccel = self._tune.Acceleration or 1 end
                    if not self._tune._origFlightSpeed then self._tune._origFlightSpeed = self._tune.Speed or 1 end
                    if not self._tune._origAccel then self._tune._origAccel = self._tune.AccelerationFactor or 1 end
                    if not self._tune._origSpeed then self._tune._origSpeed = self._tune.MaxSpeed or 1 end
                    self._tune.Acceleration = self._tune._origFlightAccel * Config.VehicleSpeedMultiplier
                    self._tune.Speed = self._tune._origFlightSpeed * Config.VehicleSpeedMultiplier
                    self._tune.AccelerationFactor = self._tune._origAccel * Config.VehicleSpeedMultiplier
                    self._tune.MaxSpeed = self._tune._origSpeed * Config.VehicleSpeedMultiplier
                    if self._solver and self._solver.NewTune then pcall(function() self._solver:NewTune() end) end
                end
            elseif self._tune and self._tune._origFlightAccel then
                self._tune.Acceleration = self._tune._origFlightAccel
                self._tune.Speed = self._tune._origFlightSpeed
                self._tune.AccelerationFactor = self._tune._origAccel
                self._tune.MaxSpeed = self._tune._origSpeed
                if self._solver and self._solver.NewTune then pcall(function() self._solver:NewTune() end) end
            end
            return hc._origUpdate(self, ...)
        end
        hc._speedHooked = true
    end
end

-- States
local ESPEnabled = false
local ESPElements = ESPElements or {}





-- [[ UI BOOTSTRAPPER ]]

pcall(function()

    for _, v in pairs(game:GetService("CoreGui"):GetChildren()) do

        if v.Name == "MertPremiumUI" or v.Name == "MertSidebarUI" or v.Name:find("VoidSens") or v.Name:find("CompKiller") then

            v:Destroy()

        end

    end

end)



local SG = Instance.new("ScreenGui")

SG.Name = "MertPremiumUI"

SG.ResetOnSpawn = false

SG.DisplayOrder = 999

pcall(function() SG.Parent = game:GetService("CoreGui") end)



-- [ GLOBAL RGB CONTROLLER ]

local rgbValue = Color3.fromHSV(0, 0, 1)

RunService.Heartbeat:Connect(function()

    pcall(function()

        rgbValue = Color3.fromHSV(tick() * 0.4 % 1, 0.7, 1)

    end)

end)



-- [ NOTIFIER SYSTEM ]
local function MertNotify(msg)
    local SG = game:GetService("CoreGui"):FindFirstChild("MertPremiumUI") or Instance.new("ScreenGui", game:GetService("CoreGui"))
    local n = Instance.new("TextLabel", SG)
    n.Size = UDim2.new(0, 260, 0, 42)
    n.Position = UDim2.new(1, 10, 1, -100)
    n.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    n.BackgroundTransparency = 0.1
    n.BorderSizePixel = 0
    n.Text = "  " .. msg; n.TextColor3 = Color3.fromRGB(255, 255, 255); n.Font = Enum.Font.GothamBold; n.TextSize = 12; n.TextXAlignment = 0
    local corner = Instance.new("UICorner", n); corner.CornerRadius = UDim.new(0, 6)
    local stroke = Instance.new("UIStroke", n); stroke.Color = Color3.fromRGB(255, 255, 255); stroke.Transparency = 0.5
    n:TweenPosition(UDim2.new(1, -270, 1, -100), "Out", "Quad", 0.3)
    task.delay(4, function() pcall(function() n:TweenPosition(UDim2.new(1, 10, 1, -110), "In", "Quad", 0.3) task.wait(0.4) n:Destroy() end) end)
end

getgenv().notify = function(data)
    local msg = type(data) == "table" and (data.Content or data.Title) or tostring(data)
    if msg:find("VoidSens") then msg = msg:gsub("VoidSens", "Mert Scripts") end
    MertNotify(msg)
end
local notify = getgenv().notify




-- [ CUSTOM TURRET CROSSHAIR ]

local CENTER_CH = Instance.new("Frame", SG)

CENTER_CH.Name = "TurretCenterCH"; CENTER_CH.Size = UDim2.new(0, 40, 0, 40); CENTER_CH.Position = UDim2.new(0.5, -20, 0.5, -20); CENTER_CH.BackgroundTransparency = 1; CENTER_CH.Visible = false; CENTER_CH.ZIndex = 5000



local function createCHPart(name, size, pos)

    local f = Instance.new("Frame", CENTER_CH)

    f.Name = name; f.Size = size; f.Position = pos; f.BackgroundColor3 = Color3.new(1, 1, 1); f.BorderSizePixel = 0; f.ZIndex = 5001

    return f

end

createCHPart("Top", UDim2.new(0, 2, 0, 15), UDim2.new(0.5, -1, 0, 0))

createCHPart("Bottom", UDim2.new(0, 2, 0, 15), UDim2.new(0.5, -1, 1, -15))

createCHPart("Left", UDim2.new(0, 15, 0, 2), UDim2.new(0, 0, 0.5, -1))

createCHPart("Right", UDim2.new(0, 15, 0, 2), UDim2.new(1, -15, 0.5, -1))

createCHPart("Center", UDim2.new(0, 4, 0, 4), UDim2.new(0.5, -2, 0.5, -2))

local circle = Instance.new("Frame", CENTER_CH)
circle.Name = "Circle"; circle.Size = UDim2.new(0, 20, 0, 20); circle.Position = UDim2.new(0.5, -10, 0.5, -10); circle.BackgroundTransparency = 1; circle.ZIndex = 102; circle.Visible = false
Instance.new("UICorner", circle).CornerRadius = UDim.new(1, 0)
local circStroke = Instance.new("UIStroke", circle); circStroke.Thickness = 2; circStroke.Color = Color3.new(1, 1, 1)


-- Input Listeners (Toggles & Menus)
UIS.InputBegan:Connect(function(i, gp)
    if gp then return end
    local cfg = getgenv().Blackhawk_Config
    if not cfg then return end

    if cfg.Character_FlyToggleKey and i.KeyCode == cfg.Character_FlyToggleKey then
        cfg.Character_Fly = not cfg.Character_Fly
        notify("Character Fly: " .. (cfg.Character_Fly and "ON" or "OFF"))
    end

    if cfg.VehicleFly_ToggleKey and i.KeyCode == cfg.VehicleFly_ToggleKey then
        cfg.VehicleFly_Enabled = not cfg.VehicleFly_Enabled
        notify("Vehicle Fly: " .. (cfg.VehicleFly_Enabled and "ON" or "OFF"))
    end
end)


local function IsVisible(actor, part, origin, isPriority)
    if not part or not part.Parent then return false end
    
    local checkOrigin = origin or Camera.CFrame.Position
    local targetPos = part.Position
    local dir = (targetPos - checkOrigin)
    
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = {Camera, LocalPlayer.Character, actor.Character}
    
    -- Vehicle seat check for NPCs
    if actor.Seat and actor.Seat.Parent then
        table.insert(params.FilterDescendantsInstances, actor.Seat.Parent)
    end
    
    local result = workspace:Raycast(checkOrigin, dir, params)
    
    if not result then return true end
    if result.Instance:IsDescendantOf(actor.Character) then return true end
    
    return false
end

local frameCount = 0
getgenv()._espLoopActive = true

local function _runESPFrame()
    frameCount = (frameCount + 1) % 60
    getgenv().RaycastBudget = 10 -- Reset per frame
    
    getgenv().LastTime = tick()
    getgenv().LastCamCF = Camera.CFrame
    
    -- SPINBOT FIX
    if Config.Spinbot and ReplicatorService and ReplicatorService.LocalActor then
        local la = ReplicatorService.LocalActor
        if la.Alive then
            local speed = (Config.SpinbotSpeed or 50) / 10
            local yaw = (tick() * speed) % (math.pi * 2)
            la.Orientation = yaw; la.GoalOrientation = yaw; la.CameraX = yaw
        end
    end

    if ActorManager then
        ActorManager:Update()
        if frameCount == 0 then
            local now = tick()
            if ActorManager._rageLast then
                for uid, lastTime in pairs(ActorManager._rageLast) do
                    if now - lastTime > 10 then
                        ActorManager._rageLast[uid] = nil
                        if ActorManager._rageShots then ActorManager._rageShots[uid] = nil end
                        if ActorManager._ignoredTargets then ActorManager._ignoredTargets[uid] = nil end
                    end
                end
            end
        end
    end

    if frameCount % 2 == 0 and (ESPEnabled or Config.HitboxExpander) then
        pcall(UpdateESP)
    end
    
    if frameCount % 3 == 0 then
        pcall(UpdateVehicles, frameCount, Config)
        pcall(SetupVehicleHooks)
        pcall(UpdateTurretCH)
    end

    if Config.SilentAim or Config.RageMode or Config.ShowWeaponInfo then
        pcall(UpdateCombat, frameCount)
    end
end

-- Hook LookAtTarget Logic into Watch/Update
task.spawn(function()
    while task.wait() do
        if Config.RageMode and Config.Rage_LookAt and ActorManager.SelectedTarget_RB then
            local target = ActorManager.SelectedTarget_RB.Position
            local camPos = Camera.CFrame.Position
            
            -- Point Camera at target for Ragebot
             pcall(function()
                if ReplicatorService and ReplicatorService.LocalActor then
                    local la = ReplicatorService.LocalActor
                    local dy = target.Y - camPos.Y
                    local dist = (target - camPos).Magnitude
                    local pitch = math.asin(dy / dist)
                    local look = CFrame.lookAt(camPos, target).LookVector
                    local yaw = math.atan2(-look.X, -look.Z)
                    
                    la.CameraX = yaw
                    la.CameraY = pitch
                end
            end)
        end
    end
end)

-- Main Loop
local espLoop = RunService.Heartbeat:Connect(function()
    if not getgenv()._espLoopActive then return end
    pcall(_runESPFrame)
end)
table.insert(getgenv().BlackhawkESP_Connections, espLoop)

-- AUTO MAP REFRESHA
task.spawn(function()
    while task.wait(5) do
        if Config.AutoDetectMap then Config.CurrentMapType = DetectMapType() end
    end
end)

InitializeServices()
VehicleFly.HookVehicles()
notify("Mert Scripts Loaded Successfully!")



-- ==========================================
-- UI INTEGRATION START (Function scope = local register sıfırlama)
-- ==========================================
local Config = getgenv().Blackhawk_Config or {}
Config.VehicleWaypoints = Config.VehicleWaypoints or {}
-- Safe Zone and Sell Point removed from standard waypoints
Config.VehicleWaypoints["Sell Point"] = nil
Config.VehicleWaypoints["Safe Zone"] = nil

local function TeleportVehicle(pos)
    pcall(function()
        local vehicle = nil
        if ControllerService and ControllerService.Controller then
            vehicle = ControllerService.Controller._vehicle
        end
        if not vehicle then return end
        
        local newCF = CFrame.new(pos)
        vehicle.CFrame = newCF
        if vehicle.Hitbox then vehicle.Hitbox.CFrame = newCF end
        if ControllerService.Controller._solver and ControllerService.Controller._solver.SetState then
            ControllerService.Controller._solver:SetState(newCF, Vector3.new(), Vector3.new(), vehicle.ComponentReplicates)
        end
        if ReplicatorService and ReplicatorService.LocalActor then
            ReplicatorService.LocalActor.SimulatedPosition = newCF.Position
        end
    end)
end

local function InitializeMertUI()
-- Service ref'leri zaten global scope'ta var, sadece alias al
local TweenService = game:GetService("TweenService")
local _Stats = game:GetService("Stats")
local _LocalPlayer = Players.LocalPlayer

if CoreGui:FindFirstChild("MertPremiumUI") then
    CoreGui.MertPremiumUI:Destroy()
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MertPremiumUI"
ScreenGui.Parent = CoreGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.ResetOnSpawn = false

local Colors = {
    Background = Color3.fromRGB(5, 5, 5),
    Accent = Color3.fromRGB(255, 255, 255),
    Border = Color3.fromRGB(255, 255, 255),
    TextPrimary = Color3.fromRGB(255, 255, 255),
    TextSecondary = Color3.fromRGB(130, 130, 130),
    DarkBg = Color3.fromRGB(15, 15, 15)
}
local BG_TRANSPARENCY = 0.40

local Watermark
local MainFrame

do -- WATERMARK SCOPE
Watermark = Instance.new("Frame")

Watermark.Size = UDim2.new(0, 260, 0, 36)
Watermark.Position = UDim2.new(1, -270, 0, 10)
Watermark.BackgroundColor3 = Colors.Background
Watermark.BackgroundTransparency = BG_TRANSPARENCY
Watermark.BorderSizePixel = 0
Watermark.Visible = false
Watermark.Parent = ScreenGui
local WmCorner = Instance.new("UICorner")
WmCorner.CornerRadius = UDim.new(0, 6)
WmCorner.Parent = Watermark
local WmStroke = Instance.new("UIStroke")
WmStroke.Color = Colors.Border
WmStroke.Thickness = 1
WmStroke.Transparency = 0.7
WmStroke.Parent = Watermark
local AvatarImage = Instance.new("ImageLabel")
AvatarImage.Size = UDim2.new(0, 24, 0, 24)
AvatarImage.Position = UDim2.new(0, 6, 0.5, -12)
AvatarImage.BackgroundTransparency = 1
AvatarImage.Parent = Watermark
pcall(function()
    AvatarImage.Image = Players:GetUserThumbnailAsync(_LocalPlayer.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size48x48)
end)
local AvatarCorner = Instance.new("UICorner")
AvatarCorner.CornerRadius = UDim.new(1, 0)
AvatarCorner.Parent = AvatarImage
local WmText = Instance.new("TextLabel")
WmText.Size = UDim2.new(1, -40, 1, 0)
WmText.Position = UDim2.new(0, 38, 0, 0)
WmText.BackgroundTransparency = 1
WmText.Text = _LocalPlayer.Name .. " | FPS: -- | MS: --"
WmText.TextColor3 = Colors.TextPrimary
WmText.Font = Enum.Font.GothamMedium
WmText.TextSize = 12
WmText.TextXAlignment = Enum.TextXAlignment.Left
WmText.RichText = true
WmText.Parent = Watermark
local wFrames, wLastUpdate = 0, tick()
RunService.RenderStepped:Connect(function()
    wFrames = wFrames + 1
    if tick() - wLastUpdate >= 1 then
        local wFps, wMs = wFrames, 0
        pcall(function() wMs = math.round(_Stats.Network.ServerStatsItem["Data Ping"]:GetValue()) end)
        WmText.Text = string.format("<b>%s</b>  |  FPS: %d  |  MS: %d", _LocalPlayer.Name, wFps, wMs)
        wFrames = 0; wLastUpdate = tick()
    end
end)
local wDragging, wDragStart, wStartPos
Watermark.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        wDragging = true; wDragStart = input.Position; wStartPos = Watermark.Position
    end
end)
Watermark.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then wDragging = false end
end)
UserInputService.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement and wDragging then
        local delta = input.Position - wDragStart
        Watermark.Position = UDim2.new(wStartPos.X.Scale, wStartPos.X.Offset + delta.X, wStartPos.Y.Scale, wStartPos.Y.Offset + delta.Y)
    end
end)
end -- WATERMARK SCOPE

-- ==========================================
-- ANA MENÜ PENCERESİ
-- ==========================================
do -- MAINFRAME SCOPE
MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 600, 0, 420)
MainFrame.Position = UDim2.new(0.5, -300, 0.5, -210)
MainFrame.BackgroundColor3 = Colors.Background
MainFrame.BackgroundTransparency = BG_TRANSPARENCY
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Visible = false
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 8)
MainCorner.Parent = MainFrame

local MainStroke = Instance.new("UIStroke")
MainStroke.Color = Colors.Border
MainStroke.Thickness = 1.2
MainStroke.Transparency = 0.7
MainStroke.Parent = MainFrame

local Header = Instance.new("Frame")
Header.Size = UDim2.new(1, 0, 0, 35)
Header.BackgroundTransparency = 1
Header.BorderSizePixel = 0
Header.Parent = MainFrame


local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -20, 1, 0)
Title.Position = UDim2.new(0, 0, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "M E R T   U I"
Title.TextColor3 = Colors.TextPrimary
Title.TextTransparency = 0.5
Title.Font = Enum.Font.GothamBold
Title.TextSize = 13
Title.TextXAlignment = Enum.TextXAlignment.Right
Title.Parent = Header

local HeaderLine = Instance.new("Frame")
HeaderLine.Size = UDim2.new(1, 0, 0, 1)
HeaderLine.Position = UDim2.new(0, 0, 1, 0)
HeaderLine.BackgroundColor3 = Color3.fromRGB(255,255,255)
HeaderLine.BorderSizePixel = 0
HeaderLine.Parent = Header

local HeaderLineGrad = Instance.new("UIGradient")
HeaderLineGrad.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(0,0,0)),
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255,255,255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(0,0,0))
})
HeaderLineGrad.Parent = HeaderLine

local SidebarContainer = Instance.new("Frame")
SidebarContainer.Size = UDim2.new(0, 45, 1, -35)
SidebarContainer.Position = UDim2.new(0, 0, 0, 35)
SidebarContainer.BackgroundColor3 = Colors.DarkBg
SidebarContainer.BackgroundTransparency = 0.4
SidebarContainer.BorderSizePixel = 0
SidebarContainer.ClipsDescendants = true
SidebarContainer.ZIndex = 10
SidebarContainer.Parent = MainFrame

local SidebarCorner = Instance.new("UICorner")
SidebarCorner.CornerRadius = UDim.new(0, 8)
SidebarCorner.Parent = SidebarContainer

local SidebarLine = Instance.new("Frame")
SidebarLine.Size = UDim2.new(0, 1, 1, 0)
SidebarLine.Position = UDim2.new(1, -1, 0, 0)
SidebarLine.BackgroundColor3 = Colors.Border
SidebarLine.BackgroundTransparency = 0.85
SidebarLine.BorderSizePixel = 0
SidebarLine.Parent = SidebarContainer

local TabList = Instance.new("ScrollingFrame")
TabList.Size = UDim2.new(1, 0, 1, -20)
TabList.Position = UDim2.new(0, 0, 0, 10)
TabList.BackgroundTransparency = 1
TabList.ScrollBarThickness = 0
TabList.Parent = SidebarContainer

local UIListLayout = Instance.new("UIListLayout")
UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
UIListLayout.Padding = UDim.new(0, 8)
UIListLayout.Parent = TabList

local ContentArea = Instance.new("Frame")
ContentArea.Size = UDim2.new(1, -65, 1, -55)
ContentArea.Position = UDim2.new(0, 55, 0, 45)
ContentArea.BackgroundTransparency = 1
ContentArea.ZIndex = 1
ContentArea.Parent = MainFrame

local isHoveringSidebar = false
local function UpdateSidebar()
    if isHoveringSidebar then
        TweenService:Create(SidebarContainer, TweenInfo.new(0.4, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Size = UDim2.new(0, 170, 1, -35), BackgroundTransparency = 0.15}):Play()
        TweenService:Create(ContentArea, TweenInfo.new(0.4, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Position = UDim2.new(0, 180, 0, 45), Size = UDim2.new(1, -190, 1, -55)}):Play()
    else
        TweenService:Create(SidebarContainer, TweenInfo.new(0.4, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Size = UDim2.new(0, 45, 1, -35), BackgroundTransparency = 0.6}):Play()
        TweenService:Create(ContentArea, TweenInfo.new(0.4, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Position = UDim2.new(0, 55, 0, 45), Size = UDim2.new(1, -65, 1, -55)}):Play()
    end
end

SidebarContainer.MouseEnter:Connect(function() isHoveringSidebar = true UpdateSidebar() end)
SidebarContainer.MouseLeave:Connect(function() isHoveringSidebar = false UpdateSidebar() end)

local dragging, dragStart, startPos
Header.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true dragStart = input.Position startPos = MainFrame.Position
    end
end)
Header.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)
UserInputService.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement and dragging then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

local activePage = nil
local function CreateTab(name, iconChar)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 35)
    btn.BackgroundTransparency = 1
    btn.Text = ""
    btn.Parent = TabList
    
    local iconLbl = Instance.new("TextLabel")
    iconLbl.Size = UDim2.new(0, 45, 1, 0)
    iconLbl.Position = UDim2.new(0, 0, 0, 0)
    iconLbl.BackgroundTransparency = 1
    iconLbl.Text = iconChar
    iconLbl.TextColor3 = Colors.TextSecondary
    iconLbl.Font = Enum.Font.GothamBold
    iconLbl.TextSize = 15
    iconLbl.Parent = btn
    
    local textLbl = Instance.new("TextLabel")
    textLbl.Size = UDim2.new(1, -45, 1, 0)
    textLbl.Position = UDim2.new(0, 45, 0, 0)
    textLbl.BackgroundTransparency = 1
    textLbl.Text = name
    textLbl.TextColor3 = Colors.TextSecondary
    textLbl.Font = Enum.Font.GothamMedium
    textLbl.TextSize = 13
    textLbl.TextXAlignment = Enum.TextXAlignment.Left
    textLbl.Parent = btn
    
    local page = Instance.new("ScrollingFrame")
    page.Size = UDim2.new(1, 0, 1, 0)
    page.BackgroundTransparency = 1
    page.ScrollBarThickness = 1
    page.ScrollBarImageColor3 = Colors.Accent
    page.AutomaticCanvasSize = Enum.AutomaticSize.Y
    page.CanvasSize = UDim2.new(0, 0, 0, 0)
    page.Visible = false
    page.Parent = ContentArea
    
    local pageLayout = Instance.new("UIListLayout")
    pageLayout.SortOrder = Enum.SortOrder.LayoutOrder
    pageLayout.Padding = UDim.new(0, 12)
    pageLayout.Parent = page
    
    btn.MouseEnter:Connect(function()
        TweenService:Create(iconLbl, TweenInfo.new(0.2), {TextColor3 = Colors.TextPrimary}):Play()
        TweenService:Create(textLbl, TweenInfo.new(0.2), {TextColor3 = Colors.TextPrimary}):Play()
    end)
    btn.MouseLeave:Connect(function()
        if activePage ~= page then
            TweenService:Create(iconLbl, TweenInfo.new(0.2), {TextColor3 = Colors.TextSecondary}):Play()
            TweenService:Create(textLbl, TweenInfo.new(0.2), {TextColor3 = Colors.TextSecondary}):Play()
        end
    end)
    
    btn.MouseButton1Click:Connect(function()
        if activePage then activePage.Visible = false end
        activePage = page
        page.Visible = true
        for _, child in pairs(TabList:GetChildren()) do
            if child:IsA("TextButton") then
                local iL = child:GetChildren()[1]
                local tL = child:GetChildren()[2]
                if iL and iL:IsA("TextLabel") then iL.TextColor3 = Colors.TextSecondary end
                if tL and tL:IsA("TextLabel") then tL.TextColor3 = Colors.TextSecondary tL.Font = Enum.Font.GothamMedium end
            end
        end
        iconLbl.TextColor3 = Colors.Accent
        textLbl.TextColor3 = Colors.Accent
        textLbl.Font = Enum.Font.GothamBold
    end)
    
    if not activePage then
        activePage = page
        page.Visible = true
        iconLbl.TextColor3 = Colors.Accent
        textLbl.TextColor3 = Colors.Accent
        textLbl.Font = Enum.Font.GothamBold
    end

    local function AddToggle(text, callback)
        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(1, 0, 0, 42)
        frame.BackgroundColor3 = Color3.fromRGB(0,0,0)
        frame.BackgroundTransparency = 0.65
        frame.Parent = page
        
        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(0, 6)
        corner.Parent = frame
        
        local stroke = Instance.new("UIStroke")
        stroke.Color = Colors.Border
        stroke.Transparency = 0.85
        stroke.Parent = frame
        
        local lbl = Instance.new("TextLabel")
        lbl.Size = UDim2.new(1, -80, 1, 0)
        lbl.Position = UDim2.new(0, 20, 0, 0)
        lbl.BackgroundTransparency = 1
        lbl.Text = text
        lbl.TextColor3 = Colors.TextPrimary
        lbl.Font = Enum.Font.GothamMedium
        lbl.TextSize = 13
        lbl.TextXAlignment = Enum.TextXAlignment.Left
        lbl.Parent = frame
        
        local switchBg = Instance.new("TextButton")
        switchBg.Size = UDim2.new(0, 40, 0, 18)
        switchBg.Position = UDim2.new(1, -60, 0.5, -9)
        switchBg.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        switchBg.BackgroundTransparency = 0.2
        switchBg.Text = ""
        switchBg.AutoButtonColor = false
        switchBg.Parent = frame
        
        local switchCorner = Instance.new("UICorner")
        switchCorner.CornerRadius = UDim.new(1, 0)
        switchCorner.Parent = switchBg
        
        local switchStroke = Instance.new("UIStroke")
        switchStroke.Color = Colors.Border
        switchStroke.Transparency = 0.6
        switchStroke.Parent = switchBg
        
        local circle = Instance.new("Frame")
        circle.Size = UDim2.new(0, 12, 0, 12)
        circle.Position = UDim2.new(0, 3, 0.5, -6)
        circle.BackgroundColor3 = Color3.fromRGB(150, 150, 150)
        circle.Parent = switchBg
        
        local circleCorner = Instance.new("UICorner")
        circleCorner.CornerRadius = UDim.new(1, 0)
        circleCorner.Parent = circle
        
        local state = false
        local function SetValue(val)
            if state == val then return end
            state = val
            if state then
                TweenService:Create(circle, TweenInfo.new(0.25, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Position = UDim2.new(1, -15, 0.5, -6), BackgroundColor3 = Colors.Accent}):Play()
                TweenService:Create(switchStroke, TweenInfo.new(0.2), {Transparency = 0.1}):Play()
                TweenService:Create(stroke, TweenInfo.new(0.2), {Transparency = 0.4}):Play()
                TweenService:Create(frame, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(25, 25, 25)}):Play()
            else
                TweenService:Create(circle, TweenInfo.new(0.25, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Position = UDim2.new(0, 3, 0.5, -6), BackgroundColor3 = Color3.fromRGB(150, 150, 150)}):Play()
                TweenService:Create(switchStroke, TweenInfo.new(0.2), {Transparency = 0.6}):Play()
                TweenService:Create(stroke, TweenInfo.new(0.2), {Transparency = 0.85}):Play()
                TweenService:Create(frame, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(0, 0, 0)}):Play()
            end
            if callback then callback(state) end
        end
        switchBg.MouseButton1Click:Connect(function() SetValue(not state) end)
        
        getgenv().MertUI_Elements = getgenv().MertUI_Elements or {}
        getgenv().MertUI_Elements[text] = {Type = "Toggle", SetValue = SetValue, GetValue = function() return state end}
    end
    
    local function AddSlider(text, min, max, default, callback)
        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(1, 0, 0, 55)
        frame.BackgroundColor3 = Color3.fromRGB(0,0,0)
        frame.BackgroundTransparency = 0.65
        frame.Parent = page
        
        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(0, 6)
        corner.Parent = frame
        
        local stroke = Instance.new("UIStroke")
        stroke.Color = Colors.Border
        stroke.Transparency = 0.85
        stroke.Parent = frame
        
        local lbl = Instance.new("TextLabel")
        lbl.Size = UDim2.new(1, -20, 0, 25)
        lbl.Position = UDim2.new(0, 20, 0, 5)
        lbl.BackgroundTransparency = 1
        lbl.Text = text
        lbl.TextColor3 = Colors.TextPrimary
        lbl.Font = Enum.Font.GothamMedium
        lbl.TextSize = 13
        lbl.TextXAlignment = Enum.TextXAlignment.Left
        lbl.Parent = frame
        
        local valLbl = Instance.new("TextLabel")
        valLbl.Size = UDim2.new(0, 40, 0, 25)
        valLbl.Position = UDim2.new(1, -60, 0, 5)
        valLbl.BackgroundTransparency = 1
        valLbl.Text = tostring(default)
        valLbl.TextColor3 = Colors.TextSecondary
        valLbl.Font = Enum.Font.Gotham
        valLbl.TextSize = 12
        valLbl.TextXAlignment = Enum.TextXAlignment.Right
        valLbl.Parent = frame
        
        local sliderBg = Instance.new("TextButton")
        sliderBg.Size = UDim2.new(1, -40, 0, 6)
        sliderBg.Position = UDim2.new(0, 20, 0, 36)
        sliderBg.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        sliderBg.Text = ""
        sliderBg.AutoButtonColor = false
        sliderBg.Parent = frame
        
        local bgCorner = Instance.new("UICorner")
        bgCorner.CornerRadius = UDim.new(1, 0)
        bgCorner.Parent = sliderBg
        
        local sliderFill = Instance.new("Frame")
        sliderFill.Size = UDim2.new((default - min)/(max - min), 0, 1, 0)
        sliderFill.BackgroundColor3 = Colors.Accent
        sliderFill.Parent = sliderBg
        
        local fillCorner = Instance.new("UICorner")
        fillCorner.CornerRadius = UDim.new(1, 0)
        fillCorner.Parent = sliderFill
        
        local function SetValue(value)
            value = math.clamp(math.floor(value), min, max)
            valLbl.Text = tostring(value)
            local relativePos = (value - min) / (max - min)
            TweenService:Create(sliderFill, TweenInfo.new(0.1), {Size = UDim2.new(relativePos, 0, 1, 0)}):Play()
            if callback then callback(value) end
        end

        local isDraggingSlider = false
        sliderBg.MouseButton1Down:Connect(function() isDraggingSlider = true end)
        UserInputService.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then isDraggingSlider = false end
        end)
        UserInputService.InputChanged:Connect(function(input)
            if isDraggingSlider and input.UserInputType == Enum.UserInputType.MouseMovement then
                local relativePos = math.clamp((input.Position.X - sliderBg.AbsolutePosition.X) / sliderBg.AbsoluteSize.X, 0, 1)
                local value = math.floor(min + ((max - min) * relativePos))
                SetValue(value)
            end
        end)
        
        getgenv().MertUI_Elements = getgenv().MertUI_Elements or {}
        getgenv().MertUI_Elements[text] = {Type = "Slider", SetValue = SetValue, GetValue = function() return tonumber(valLbl.Text) or default end}
    end
    
    local function AddButton(text, callback)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(1, 0, 0, 42)
        btn.BackgroundColor3 = Color3.fromRGB(0,0,0)
        btn.BackgroundTransparency = 0.65
        btn.Text = text
        btn.TextColor3 = Colors.TextPrimary
        btn.Font = Enum.Font.GothamMedium
        btn.TextSize = 13
        btn.Parent = page
        
        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(0, 6)
        corner.Parent = btn
        
        local stroke = Instance.new("UIStroke")
        stroke.Color = Colors.Border
        stroke.Transparency = 0.85
        stroke.Parent = btn
        
        btn.MouseButton1Click:Connect(function()
            TweenService:Create(btn, TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundColor3 = Color3.fromRGB(40,40,40)}):Play()
            task.wait(0.1)
            TweenService:Create(btn, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundColor3 = Color3.fromRGB(0,0,0)}):Play()
            if callback then callback() end
        end)
    end
    
    local function AddKeybind(text, defaultKey, callback)
        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(1, 0, 0, 36)
        frame.BackgroundColor3 = Color3.fromRGB(0,0,0)
        frame.BackgroundTransparency = 0.65
        frame.Parent = page
        
        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(0, 6)
        corner.Parent = frame
        
        local stroke = Instance.new("UIStroke")
        stroke.Color = Colors.Border
        stroke.Transparency = 0.85
        stroke.Parent = frame
        
        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(0.5, 0, 1, 0)
        label.Position = UDim2.new(0, 12, 0, 0)
        label.BackgroundTransparency = 1
        label.Text = text
        label.TextColor3 = Colors.TextSecondary
        label.Font = Enum.Font.GothamMedium
        label.TextSize = 13
        label.TextXAlignment = Enum.TextXAlignment.Left
        label.Parent = frame
        
        local bindBtn = Instance.new("TextButton")
        bindBtn.Size = UDim2.new(0, 70, 0, 24)
        bindBtn.Position = UDim2.new(1, -82, 0.5, -12)
        bindBtn.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
        bindBtn.Text = defaultKey.Name
        bindBtn.TextColor3 = Colors.TextPrimary
        bindBtn.Font = Enum.Font.GothamBold
        bindBtn.TextSize = 12
        bindBtn.Parent = frame
        
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 4)
        btnCorner.Parent = bindBtn
        
        local btnStroke = Instance.new("UIStroke")
        btnStroke.Color = Colors.Border
        btnStroke.Transparency = 0.7
        btnStroke.Parent = bindBtn
        
        local binding = false
        bindBtn.MouseButton1Click:Connect(function()
            binding = true
            bindBtn.Text = "..."
        end)
        
        local function SetValue(keyEnum)
            if type(keyEnum) == "string" then
                pcall(function() keyEnum = Enum.KeyCode[keyEnum] end)
            end
            if typeof(keyEnum) == "EnumItem" then
                bindBtn.Text = keyEnum.Name
                if callback then callback(keyEnum) end
            end
        end

        UserInputService.InputBegan:Connect(function(input, gpe)
            if binding and input.UserInputType == Enum.UserInputType.Keyboard then
                binding = false
                SetValue(input.KeyCode)
            end
        end)
        
        getgenv().MertUI_Elements = getgenv().MertUI_Elements or {}
        getgenv().MertUI_Elements[text] = {Type = "Keybind", SetValue = SetValue, GetValue = function() return bindBtn.Text end}
    end
    
    local function AddTextBox(text, placeholder, callback)
        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(1, 0, 0, 42)
        frame.BackgroundColor3 = Color3.fromRGB(0,0,0)
        frame.BackgroundTransparency = 0.65
        frame.Parent = page
        
        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(0, 6)
        corner.Parent = frame
        
        local lbl = Instance.new("TextLabel")
        lbl.Size = UDim2.new(0.5, -10, 1, 0)
        lbl.Position = UDim2.new(0, 20, 0, 0)
        lbl.BackgroundTransparency = 1
        lbl.Text = text
        lbl.TextColor3 = Colors.TextPrimary
        lbl.Font = Enum.Font.GothamMedium
        lbl.TextSize = 13
        lbl.TextXAlignment = Enum.TextXAlignment.Left
        lbl.Parent = frame
        
        local box = Instance.new("TextBox")
        box.Size = UDim2.new(0.5, -20, 0, 26)
        box.Position = UDim2.new(0.5, 10, 0.5, -13)
        box.BackgroundColor3 = Color3.fromRGB(20,20,20)
        box.Text = ""
        box.PlaceholderText = placeholder
        box.TextColor3 = Colors.TextPrimary
        box.Font = Enum.Font.Gotham
        box.TextSize = 12
        box.Parent = frame
        
        local bCorner = Instance.new("UICorner")
        bCorner.CornerRadius = UDim.new(0, 4)
        bCorner.Parent = box
        
        box.FocusLost:Connect(function()
            if callback then callback(box.Text) end
        end)
    end

    local function AddDropdown(text, options, callback)
        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(1, 0, 0, 42)
        frame.BackgroundColor3 = Color3.fromRGB(0,0,0)
        frame.BackgroundTransparency = 0.65
        frame.ClipsDescendants = true
        frame.Parent = page
        
        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(0, 6)
        corner.Parent = frame
        
        local topFrame = Instance.new("Frame")
        topFrame.Size = UDim2.new(1, 0, 0, 42)
        topFrame.BackgroundTransparency = 1
        topFrame.Parent = frame

        local lbl = Instance.new("TextLabel")
        lbl.Size = UDim2.new(0.5, -10, 1, 0)
        lbl.Position = UDim2.new(0, 20, 0, 0)
        lbl.BackgroundTransparency = 1
        lbl.Text = text
        lbl.TextColor3 = Colors.TextPrimary
        lbl.Font = Enum.Font.GothamMedium
        lbl.TextSize = 13
        lbl.TextXAlignment = Enum.TextXAlignment.Left
        lbl.Parent = topFrame

        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0.5, -20, 0, 26)
        btn.Position = UDim2.new(0.5, 10, 0.5, -13)
        btn.BackgroundColor3 = Color3.fromRGB(20,20,20)
        btn.Text = options[1] and (options[1] .. " ▼") or "Seçiniz ▼"
        btn.TextColor3 = Colors.TextPrimary
        btn.Font = Enum.Font.Gotham
        btn.TextSize = 12
        btn.Parent = topFrame
        
        local dCorner = Instance.new("UICorner")
        dCorner.CornerRadius = UDim.new(0, 4)
        dCorner.Parent = btn

        local listFrame = Instance.new("ScrollingFrame")
        listFrame.Size = UDim2.new(1, -20, 1, -42)
        listFrame.Position = UDim2.new(0, 10, 0, 42)
        listFrame.BackgroundTransparency = 1
        listFrame.ScrollBarThickness = 2
        listFrame.Parent = frame
        
        local lLayout = Instance.new("UIListLayout")
        lLayout.SortOrder = Enum.SortOrder.LayoutOrder
        lLayout.Parent = listFrame
        
        local isOpen = false
        
        local function PopulateList(opts)
            for _, c in pairs(listFrame:GetChildren()) do
                if c:IsA("TextButton") then c:Destroy() end
            end
            local h = 0
            for _, opt in ipairs(opts) do
                local oBtn = Instance.new("TextButton")
                oBtn.Size = UDim2.new(1, 0, 0, 25)
                oBtn.BackgroundColor3 = Color3.fromRGB(15,15,15)
                oBtn.BackgroundTransparency = 0.5
                oBtn.Text = opt
                oBtn.TextColor3 = Colors.TextPrimary
                oBtn.Font = Enum.Font.Gotham
                oBtn.TextSize = 12
                oBtn.Parent = listFrame
                h = h + 25
                oBtn.MouseButton1Click:Connect(function()
                    btn.Text = opt .. " ▼"
                    isOpen = false
                    TweenService:Create(frame, TweenInfo.new(0.2), {Size = UDim2.new(1, 0, 0, 42)}):Play()
                    if callback then callback(opt) end
                end)
                oBtn.MouseEnter:Connect(function() oBtn.BackgroundTransparency = 0 end)
                oBtn.MouseLeave:Connect(function() oBtn.BackgroundTransparency = 0.5 end)
            end
            listFrame.CanvasSize = UDim2.new(0, 0, 0, h)
        end
        
        PopulateList(options)
        
        btn.MouseButton1Click:Connect(function()
            isOpen = not isOpen
            if isOpen then
                TweenService:Create(frame, TweenInfo.new(0.2), {Size = UDim2.new(1, 0, 0, 140)}):Play()
            else
                TweenService:Create(frame, TweenInfo.new(0.2), {Size = UDim2.new(1, 0, 0, 42)}):Play()
            end
        end)
        
        local function Refresh(newOptions)
            PopulateList(newOptions)
            if #newOptions == 0 then btn.Text = "Yok ▼" return end
            local currentRaw = btn.Text:gsub(" ▼", "")
            local found = false
            for _, o in ipairs(newOptions) do
                if o == currentRaw then found = true break end
            end
            if not found then btn.Text = newOptions[1] .. " ▼" end
        end
        return Refresh
    end

    return {AddToggle = AddToggle, AddSlider = AddSlider, AddButton = AddButton, AddKeybind = AddKeybind, AddTextBox = AddTextBox, AddDropdown = AddDropdown}
end

local Config = getgenv().Blackhawk_Config or {}

local Tab1 = CreateTab("Savaş (Combat)", "⚔")
Tab1.AddToggle("Silent Aim (Görünmez Aimbot)", function(v) Config.SilentAim = v end)
Tab1.AddToggle("FOV Çemberi Göster", function(v) Config.ShowFOV = v end)
Tab1.AddToggle("Hitbox Expander", function(v) Config.HitboxExpander = v end)
Tab1.AddSlider("Hitbox Size", 1, 20, 4, function(v) Config.HitboxSize = v end)
Tab1.AddToggle("Prediction", function(v) Config.Prediction = v end)
Tab1.AddToggle("Bullet Drop", function(v) Config.BulletDrop = v end)
Tab1.AddSlider("Görüş Açısı (FOV)", 10, 800, 100, function(v) Config.SilentAimFOV = v end)
Tab1.AddSlider("Hit Chance", 1, 100, 100, function(v) Config.SilentAimHitChance = v end)
Tab1.AddToggle("Show Weapon Info", function(v) Config.ShowWeaponInfo = v; pcall(function() if WeaponInfoText then WeaponInfoText.Visible = v end end) end)

local Tab2 = CreateTab("Görseller (ESP)", "👁")
Tab2.AddToggle("Enable ESP", function(v) ESPEnabled = v end)
Tab2.AddToggle("Oyuncu Gösterimi", function(v) Config.ShowPlayers = v end)
Tab2.AddToggle("Zombi Gösterimi", function(v) Config.ShowZombies = v end)
Tab2.AddToggle("Show NPCs", function(v) Config.ShowNPCs = v end)
Tab2.AddToggle("Show Teammates", function(v) Config.ShowSquadMembers = v end)
Tab2.AddSlider("ESP Distance", 100, 10000, 2000, function(v) Config.MaxDistance = v end)
Tab2.AddToggle("Kutu (Box)", function(v) Config.UseBoxESP = v end)
Tab2.AddToggle("Names", function(v) Config.ShowNames = v end)
Tab2.AddToggle("Mesafe Göster", function(v) Config.ShowDistance = v end)
Tab2.AddToggle("Health", function(v) Config.ShowHealth = v end)
Tab2.AddToggle("Tracers", function(v) Config.UseTracers = v end)
Tab2.AddToggle("Highlight", function(v) Config.UseHighlight = v end)

local Tab3 = CreateTab("Silah (Gun)", "⚒")
Tab3.AddToggle("Mermi Sekmez (No Recoil)", function(v) Config.NoRecoil = v end)
Tab3.AddToggle("Sekmeme (No Spread)", function(v) Config.NoSpread = v end)
Tab3.AddToggle("Unlock Firemodes", function(v) Config.UnlockFiremodes = v end)
Tab3.AddToggle("Custom RPM", function(v) Config.CustomRPM = v end)
Tab3.AddSlider("Mermi Hızı (Fire Rate)", 100, 3000, 800, function(v) Config.RPMValue = v end)

local Tab4 = CreateTab("Karakter", "♟")
Tab4.AddToggle("Character Fly", function(v) getgenv().Blackhawk_Config.Character_Fly = v end)
Tab4.AddKeybind("Fly Tuşu Ata", Enum.KeyCode.V, function(k) getgenv().Blackhawk_Config.Character_FlyToggleKey = k end)
Tab4.AddSlider("Fly Speed", 10, 1000, 50, function(v) Config.Character_FlySpeed = v end)
Tab4.AddToggle("Hızlı Koşma (Speedhack)", function(v) Config.Character_WalkSpeedEnabled = v end)
Tab4.AddSlider("Yürüme Hızı", 16, 200, 32, function(v) Config.Character_WalkSpeed = v end)
Tab4.AddToggle("Sprint Speed", function(v) Config.Character_SprintSpeedEnabled = v end)
Tab4.AddSlider("Sprint Value", 16, 200, 50, function(v) Config.Character_SprintSpeed = v end)
Tab4.AddToggle("Spinbot", function(v) Config.Spinbot = v end)
Tab4.AddSlider("Spin Speed", 1, 500, 50, function(v) Config.SpinbotSpeed = v end)

local Tab5 = CreateTab("Araçlar", "🚗")
Tab5.AddToggle("Araç Uçurma (Vehicle Fly)", function(v) getgenv().Blackhawk_Config.VehicleFly_Enabled = v end)
Tab5.AddKeybind("Araç Fly Tuşu Ata", Enum.KeyCode.Space, function(k) getgenv().Blackhawk_Config.VehicleFly_ToggleKey = k end)
Tab5.AddSlider("Araç Uçma Hızı", 10, 1000, 100, function(v) Config.VehicleFly_Speed = v end)
Tab5.AddSlider("Speed Multiplier", 0, 10, 0, function(v) Config.VehicleSpeedMultiplier = v end)
Tab5.AddToggle("No Recoil (Turret)", function(v) Config.TurretNoRecoil = v end)
Tab5.AddToggle("No Spread (Turret)", function(v) Config.TurretNoSpread = v end)
Tab5.AddToggle("Unlock Firemodes", function(v) Config.TurretUnlockFiremodes = v end)
Tab5.AddToggle("Turret Crosshair", function(v) Config.TurretCH_Enabled = v end)
Tab5.AddSlider("Crosshair Size", 2, 50, 10, function(v) Config.TurretCH_Size = v end)
Tab5.AddSlider("Crosshair Gap", 0, 20, 3, function(v) Config.TurretCH_Gap = v end)
Tab5.AddSlider("Crosshair Thickness", 1, 5, 1, function(v) Config.TurretCH_Thickness = v end)
Tab5.AddToggle("Turret Zoom (Scroll)", function(v) Config.Turret_ZoomEnabled = v end)
Tab5.AddToggle("Enable Veh ESP", function(v) Config.ShowVehicles = v end)
Tab5.AddToggle("Show Box", function(v) Config.ShowVehicleBox = v end)
Tab5.AddToggle("Show Name", function(v) Config.ShowVehicleName = v end)
Tab5.AddToggle("Show Health", function(v) Config.ShowVehicleHealth = v end)

-- VEHICLE TELEPORT (ÖZEL KONUMLAR)
Tab5.AddButton("Sell Point'e Işınlan", function()
    TeleportVehicle(Vector3.new(-3780.14, 65.05, 1429.74))
end)

local TargetWaypointName = ""
local waypointNames = {}
for name, _ in pairs(Config.VehicleWaypoints) do table.insert(waypointNames, name) end
if #waypointNames == 0 then table.insert(waypointNames, "Yok") end
local SelectedWaypoint = waypointNames[1]

local RefreshDropdown = Tab5.AddDropdown("Kayıtlı Konumlar", waypointNames, function(val) SelectedWaypoint = val end)

Tab5.AddButton("Seçili Konuma Işınlan", function()
    if SelectedWaypoint and Config.VehicleWaypoints[SelectedWaypoint] then
        TeleportVehicle(Config.VehicleWaypoints[SelectedWaypoint])
    end
end)

Tab5.AddTextBox("Yeni Konum Adı", "Örn: Base", function(txt) TargetWaypointName = txt end)

Tab5.AddButton("Mevcut Konumu Kaydet", function()
    if TargetWaypointName ~= "" and TargetWaypointName ~= "Yok" then
        pcall(function()
            local currentPos = nil
            if ControllerService and ControllerService.Controller and ControllerService.Controller._vehicle then
                local veh = ControllerService.Controller._vehicle
                currentPos = (veh.CFrame or veh.Hitbox.CFrame).Position
            end
            if not currentPos and Workspace.CurrentCamera then
                currentPos = Workspace.CurrentCamera.CFrame.Position
            end
            if currentPos then
                Config.VehicleWaypoints[TargetWaypointName] = currentPos
                local newNames = {}
                for name, _ in pairs(Config.VehicleWaypoints) do table.insert(newNames, name) end
                RefreshDropdown(newNames)
            end
        end)
    end
end)

Tab5.AddButton("Seçili Konumu Sil", function()
    if SelectedWaypoint and Config.VehicleWaypoints[SelectedWaypoint] then
        Config.VehicleWaypoints[SelectedWaypoint] = nil
        local newNames = {}
        for name, _ in pairs(Config.VehicleWaypoints) do table.insert(newNames, name) end
        RefreshDropdown(newNames)
    end
end)

local Tab6 = CreateTab("Dünya", "🌍")
Tab6.AddToggle("Always Day", function(v) Config.AlwaysDay = v end)
Tab6.AddToggle("No Fog", function(v) Config.NoFog = v; if v then Lighting.FogEnd = 100000; for _, obj in pairs(Lighting:GetDescendants()) do if obj:IsA("Atmosphere") or obj:IsA("Fog") then obj.Density = 0 end end else Lighting.FogEnd = 2000 end end)
Tab6.AddToggle("Disable Shadows", function(v) Lighting.GlobalShadows = not v end)
Tab6.AddToggle("Thermal Vision", function(v) Config.ThermalVision = v; if EnvironmentService then EnvironmentService.FLIR = v end end)
Tab6.AddToggle("Night Vision", function(v) Config.NightVision = v; if v then pcall(function() game:GetService("Lighting").Brightness = 5; game:GetService("Lighting").Ambient = Color3.fromRGB(80, 150, 80); game:GetService("Lighting").OutdoorAmbient = Color3.fromRGB(80, 150, 80) end) else pcall(function() game:GetService("Lighting").Brightness = 2; game:GetService("Lighting").Ambient = Color3.fromRGB(70, 70, 70); game:GetService("Lighting").OutdoorAmbient = Color3.fromRGB(70, 70, 70) end) end end)
Tab6.AddToggle("Disable Particles", function(v) Config.FPS_NoEffects = v; pcall(function() for _, obj in ipairs(workspace:GetDescendants()) do if obj:IsA("ParticleEmitter") or obj:IsA("Smoke") or obj:IsA("Fire") or obj:IsA("Sparkles") then obj.Enabled = not v end end end) end)
Tab6.AddToggle("Clean Lighting", function(v) Config.FPS_CleanLighting = v; Lighting.GlobalShadows = not v; for _, obj in ipairs(Lighting:GetDescendants()) do if obj:IsA("Atmosphere") or obj:IsA("BloomEffect") or obj:IsA("SunRaysEffect") then obj.Enabled = not v end end end)
Tab6.AddToggle("Texture Remover", function(v) Config.FPS_NoTextures = v; pcall(function() for _, obj in ipairs(workspace:GetDescendants()) do if obj:IsA("Texture") or obj:IsA("Decal") then obj.Transparency = v and 1 or 0 end end end) end)

local Tab7 = CreateTab("Ayarlar", "⚙")
local HttpService = game:GetService("HttpService")
local configFolder = "MertScripts_BRM5"
if isfolder and not isfolder(configFolder) then
    makefolder(configFolder)
end

local function GetConfigs()
    local configs = {}
    if listfiles and isfolder(configFolder) then
        for _, file in ipairs(listfiles(configFolder)) do
            if file:match("%.json$") then
                local name = file:match("([^/\\]+)%.json$")
                if name then table.insert(configs, name) end
            end
        end
    end
    if #configs == 0 then table.insert(configs, "Yok") end
    return configs
end

local configName = ""
Tab7.AddTextBox("Config Adı", "Örn: mert", function(txt) configName = txt end)

local selectedConfig = GetConfigs()[1]
local configDropdown = Tab7.AddDropdown("Kayıtlı Configler", GetConfigs(), function(val) selectedConfig = val end)

Tab7.AddButton("Config Kaydet", function()
    if configName ~= "" and configName ~= "Yok" and writefile then
        local saveTable = {}
        if getgenv().MertUI_Elements then
            for k, v in pairs(getgenv().MertUI_Elements) do
                saveTable[k] = v.GetValue()
            end
        end
        local success, json = pcall(function() return HttpService:JSONEncode(saveTable) end)
        if success then
            writefile(configFolder .. "/" .. configName .. ".json", json)
            configDropdown(GetConfigs())
        end
    end
end)

Tab7.AddButton("Config Yükle", function()
    if selectedConfig and selectedConfig ~= "Yok" and readfile and isfile and isfile(configFolder .. "/" .. selectedConfig .. ".json") then
        local success, json = pcall(function() return readfile(configFolder .. "/" .. selectedConfig .. ".json") end)
        if success then
            local s2, decoded = pcall(function() return HttpService:JSONDecode(json) end)
            if s2 and decoded then
                for k,v in pairs(decoded) do
                    if getgenv().MertUI_Elements and getgenv().MertUI_Elements[k] then
                        pcall(function() getgenv().MertUI_Elements[k].SetValue(v) end)
                    end
                end
            end
        end
    end
end)

Tab7.AddButton("Seçili Config'i Sil", function()
    if selectedConfig and selectedConfig ~= "Yok" and isfile and isfile(configFolder .. "/" .. selectedConfig .. ".json") then
        if delfile then delfile(configFolder .. "/" .. selectedConfig .. ".json") end
        configDropdown(GetConfigs())
    end
end)

Tab7.AddButton("Listeyi Yenile", function()
    configDropdown(GetConfigs())
end)

local TR_to_EN = {
    ["Savaş (Combat)"] = "Combat",
    ["Görseller (ESP)"] = "Visuals (ESP)",
    ["Silah (Gun)"] = "Gun Mods",
    ["Karakter"] = "Player",
    ["Araçlar"] = "Vehicles",
    ["Dünya"] = "World",
    ["Ayarlar"] = "Settings",
    ["Silent Aim (Görünmez Aimbot)"] = "Silent Aim",
    ["FOV Çemberi Göster"] = "Show FOV Circle",
    ["Görüş Açısı (FOV)"] = "FOV Size",
    ["Oyuncu Gösterimi"] = "Show Players",
    ["Zombi Gösterimi"] = "Show Zombies",
    ["Kutu (Box)"] = "Box ESP",
    ["Mesafe Göster"] = "Show Distance",
    ["Mermi Sekmez (No Recoil)"] = "No Recoil",
    ["Sekmeme (No Spread)"] = "No Spread",
    ["Mermi Hızı (Fire Rate)"] = "Fire Rate (RPM)",
    ["Hızlı Koşma (Speedhack)"] = "Speedhack",
    ["Yürüme Hızı"] = "Walk Speed",
    ["Araç Uçurma (Vehicle Fly)"] = "Vehicle Fly",
    ["Araç Uçma Hızı"] = "Vehicle Fly Speed",
    ["Sell Point'e Işınlan"] = "Teleport to Sell Point",
    ["Kayıtlı Konumlar"] = "Saved Waypoints",
    ["Seçili Konuma Işınlan"] = "Teleport to Waypoint",
    ["Yeni Konum Adı"] = "New Waypoint Name",
    ["Mevcut Konumu Kaydet"] = "Save Waypoint",
    ["Seçili Konumu Sil"] = "Delete Waypoint",
    ["Config Adı"] = "Config Name",
    ["Config Kaydet"] = "Save Config",
    ["Kayıtlı Configler"] = "Saved Configs",
    ["Config Yükle"] = "Load Config",
    ["Seçili Config'i Sil"] = "Delete Config",
    ["Listeyi Yenile"] = "Refresh List",
    ["Seçiniz"] = "Select",
    ["Seçiniz ▼"] = "Select ▼",
    ["Yok"] = "None",
    ["Yok ▼"] = "None ▼",
    ["Fly Tuşu Ata"] = "Bind Fly Key",
    ["Araç Fly Tuşu Ata"] = "Bind Vehicle Fly",
    ["Dil (Language)"] = "Language",
    ["Dil: Türkçe"] = "Language: Turkish",
    ["Language: English"] = "Language: English"
}

local EN_to_TR = {}
for k,v in pairs(TR_to_EN) do EN_to_TR[v] = k end

local function SetLanguage(lang)
    Config.Language = lang
    local map = lang == "EN" and TR_to_EN or EN_to_TR
    if MainFrame then
        for _, obj in pairs(MainFrame:GetDescendants()) do
            if obj:IsA("TextLabel") or obj:IsA("TextButton") or obj:IsA("TextBox") then
                if map[obj.Text] then
                    obj.Text = map[obj.Text]
                else
                    local rawText = obj.Text:gsub(" ▼", "")
                    if map[rawText] then
                        obj.Text = map[rawText] .. " ▼"
                    end
                end
                if obj:IsA("TextBox") and obj.PlaceholderText and map[obj.PlaceholderText] then
                    obj.PlaceholderText = map[obj.PlaceholderText]
                end
            end
        end
    end
end

local Tab8 = CreateTab("Dil (Language)", "🌐")
Tab8.AddButton("Dil: Türkçe", function() SetLanguage("TR") end)
Tab8.AddButton("Language: English", function() SetLanguage("EN") end)

-- K TUŞU İLE MENÜYÜ GİZLEME/AÇMA
UserInputService.InputBegan:Connect(function(input, gpe)
    if not gpe and input.KeyCode == Enum.KeyCode.K then
        if MainFrame then
            MainFrame.Visible = not MainFrame.Visible
        end
    end
end)
end -- MAINFRAME SCOPE

-- ==========================================
-- PREMIUM YÜKLEME EKRANI (FULL SCREEN OVERLAY)
-- ==========================================
do -- LOADOVERLAY SCOPE
local LoadOverlay = Instance.new("Frame")
LoadOverlay.Size = UDim2.new(1, 0, 1, 0)
LoadOverlay.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
LoadOverlay.BackgroundTransparency = 0.3
LoadOverlay.BorderSizePixel = 0
LoadOverlay.ZIndex = 100
LoadOverlay.Parent = ScreenGui

local CenterBox = Instance.new("Frame")
CenterBox.Size = UDim2.new(0, 320, 0, 160)
CenterBox.Position = UDim2.new(0.5, -160, 0.5, -80)
CenterBox.BackgroundColor3 = Colors.Background
CenterBox.BackgroundTransparency = 1
CenterBox.ZIndex = 101
CenterBox.Parent = LoadOverlay

local Spinner = Instance.new("Frame")
Spinner.Size = UDim2.new(0, 44, 0, 44)
Spinner.Position = UDim2.new(0.5, -22, 0.5, -45)
Spinner.BackgroundTransparency = 1
Spinner.ZIndex = 101
Spinner.Parent = CenterBox

local SpinCorner = Instance.new("UICorner")
SpinCorner.CornerRadius = UDim.new(1, 0)
SpinCorner.Parent = Spinner

local SpinStroke = Instance.new("UIStroke")
SpinStroke.Color = Color3.fromRGB(255, 255, 255)
SpinStroke.Thickness = 2.5
SpinStroke.Parent = Spinner

local SpinGradient = Instance.new("UIGradient")
SpinGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(50, 50, 50))
})
SpinGradient.Transparency = NumberSequence.new({
    NumberSequenceKeypoint.new(0, 0),
    NumberSequenceKeypoint.new(0.5, 0),
    NumberSequenceKeypoint.new(1, 1)
})
SpinGradient.Parent = SpinStroke

local LoadTitle = Instance.new("TextLabel")
LoadTitle.Size = UDim2.new(1, 0, 0, 30)
LoadTitle.Position = UDim2.new(0, 0, 0.5, 10)
LoadTitle.BackgroundTransparency = 1
LoadTitle.Text = "M E R T   U I"
LoadTitle.TextColor3 = Colors.TextPrimary
LoadTitle.Font = Enum.Font.GothamBold
LoadTitle.TextSize = 22
LoadTitle.ZIndex = 101
LoadTitle.Parent = CenterBox

local LoadSub = Instance.new("TextLabel")
LoadSub.Size = UDim2.new(1, 0, 0, 20)
LoadSub.Position = UDim2.new(0, 0, 0.5, 45)
LoadSub.BackgroundTransparency = 1
LoadSub.Text = "Initializing..."
LoadSub.TextColor3 = Colors.TextSecondary
LoadSub.Font = Enum.Font.Gotham
LoadSub.TextSize = 13
LoadSub.ZIndex = 101
LoadSub.Parent = CenterBox

task.spawn(function()
    -- Spinner Döndürme Animasyonu
    local spinConn
    local rot = 0
    spinConn = RunService.RenderStepped:Connect(function()
        rot = (rot + 6) % 360
        Spinner.Rotation = rot
    end)
    
    local phases = {
        "Bypassing Anti-Cheat...",
        "Fetching Offsets...",
        "Loading Modules...",
        "Welcome to MERT UI"
    }
    
    for i = 1, #phases do
        TweenService:Create(LoadSub, TweenInfo.new(0.2), {TextTransparency = 1}):Play()
        task.wait(0.2)
        LoadSub.Text = phases[i]
        TweenService:Create(LoadSub, TweenInfo.new(0.2), {TextTransparency = 0}):Play()
        task.wait(0.7)
    end
    
    -- Kapanış Animasyonları
    spinConn:Disconnect()
    TweenService:Create(LoadTitle, TweenInfo.new(0.4), {TextTransparency = 1}):Play()
    TweenService:Create(LoadSub, TweenInfo.new(0.4), {TextTransparency = 1}):Play()
    TweenService:Create(SpinStroke, TweenInfo.new(0.4), {Transparency = 1}):Play()
    task.wait(0.4)
    
    TweenService:Create(LoadOverlay, TweenInfo.new(0.8, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {BackgroundTransparency = 1}):Play()
    
    -- Ana Menüyü Zıplatarak Aç (Popup Effect)
    MainFrame.Size = UDim2.new(0, 500, 0, 350)
    MainFrame.Position = UDim2.new(0.5, -250, 0.5, -175)
    MainFrame.Visible = true
    TweenService:Create(MainFrame, TweenInfo.new(0.6, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = UDim2.new(0, 600, 0, 420), Position = UDim2.new(0.5, -300, 0.5, -210)}):Play()
    
    Watermark.Position = UDim2.new(1, 50, 0, 10)
    Watermark.Visible = true
    TweenService:Create(Watermark, TweenInfo.new(0.7, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Position = UDim2.new(1, -270, 0, 10)}):Play()
    
    task.wait(0.8)
    LoadOverlay:Destroy()
end)
end -- LOADOVERLAY SCOPE

end -- UI INTEGRATION Function end
InitializeMertUI()
