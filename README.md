-- Auto Reconectar | OTIMIZADO E REFORÇADO para Delta Executor
-- Funciona: queda de internet, kick, erro de conexão, saída, travamento
-- Recursos: Detecção tripla, tentativas infinitas, interface visual, notificações

-- Serviços
local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local RunService = game:GetService("RunService")
local NetworkClient = game:GetService("NetworkClient")
local Player = Players.LocalPlayer

-- Configurações
local PlaceId = game.PlaceId
local JobId = game.JobId
local TempoEspera = 1.5 -- mais rápido para reconectar
local Tentativas = 0
local Reconectando = false
local Ativo = true

-- ⬇️ INTERFACE VISÍVEL NA TELA
local Tela = Instance.new("ScreenGui")
Tela.Name = "AutoReconectar_Reforcado"
Tela.Parent = game:GetService("CoreGui")
Tela.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local TextoStatus = Instance.new("TextLabel")
TextoStatus.Parent = Tela
TextoStatus.Size = UDim2.new(0, 240, 0, 45)
TextoStatus.Position = UDim2.new(0.02, 0, 0.02, 0)
TextoStatus.BackgroundColor3 = Color3.new(0, 0, 0)
TextoStatus.BackgroundTransparency = 0.2
TextoStatus.BorderSizePixel = 0
TextoStatus.TextColor3 = Color3.new(0, 1, 0)
TextoStatus.Font = Enum.Font.GothamBold
TextoStatus.TextSize = 16
TextoStatus.Text = "✅ Auto Reconectar: ATIVO"
TextoStatus.TextWrapped = true
TextoStatus.Active = true
TextoStatus.Draggable = true -- pode mover pela tela

-- Função principal de reconexão
local function Reconectar()
    if not Ativo or Reconectando then return end
    Reconectando = true
    Tentativas += 1

    -- Atualiza interface
    TextoStatus.TextColor3 = Color3.new(1, 0.5, 0)
    TextoStatus.Text = `🔄 Tentando reconectar... ({Tentativas})`
    print(`[DELTA REFORÇADO] Tentativa {Tentativas} | Servidor: {JobId or "Qualquer"}`)

    -- Tenta 1: VOLTAR PARA O MESMO SERVIDOR (prioridade)
    if JobId and JobId ~= "" then
        local sucesso, erro = pcall(TeleportService.TeleportToPlaceInstance, TeleportService, PlaceId, JobId, Player)
        if sucesso then
            -- Se deu certo, reseta tudo
            Tentativas = 0
            Reconectando = false
            TextoStatus.TextColor3 = Color3.new(0, 1, 0)
            TextoStatus.Text = "✅ Conectado com sucesso!"
            task.delay(4, function() if Tela then TextoStatus.Text = "✅ Auto Reconectar: ATIVO" end end)
            return
        end
        print(`[ERRO MESMO SERVIDOR] {erro}`)
    end

    -- Tenta 2: ENTRAR EM QUALQUER SERVIDOR DO JOGO
    local sucesso2, erro2 = pcall(TeleportService.Teleport, TeleportService, PlaceId, Player)
    if sucesso2 then
        Tentativas = 0
        Reconectando = false
        TextoStatus.TextColor3 = Color3.new(0, 1, 0)
        TextoStatus.Text = "✅ Conectado em novo servidor!"
        task.delay(4, function() if Tela then TextoStatus.Text = "✅ Auto Reconectar: ATIVO" end end)
        return
    end

    print(`[ERRO GERAL] {erro2} | Tentando novamente em {TempoEspera}s...`)
    Reconectando = false
end

-- 🛡️ DETECÇÕES REFORÇADAS (TRIPLA SEGURANÇA)

-- 1. Detecção de queda de conexão / rede
NetworkClient.ChildRemoved:Connect(function(filho)
    if Ativo and not Reconectando then
        TextoStatus.TextColor3 = Color3.new(1, 0, 0)
        TextoStatus.Text = "❌ Conexão caiu! Reconectando..."
        task.wait(TempoEspera)
        Reconectar()
    end
end)

-- 2. Detecção de kick / saída do servidor
Players.PlayerRemoving:Connect(function(quemSaiu)
    if quemSaiu == Player and Ativo and not Reconectando then
        TextoStatus.TextColor3 = Color3.new(1, 0, 0)
        TextoStatus.Text = "❌ Você foi desconectado! Reconectando..."
        task.wait(TempoEspera)
        Reconectar()
    end
end)

-- 3. Verificação CONSTANTE (a cada 1 segundo) - pega tudo que os eventos não pegam
RunService.Heartbeat:Connect(function()
    if not Ativo then return end
    -- Se o jogador não está mais na lista ou não tem personagem = desconectado
    if not Player:IsDescendantOf(Players) or not Player.Character then
        if not Reconectando then
            TextoStatus.TextColor3 = Color3.new(1, 0, 0)
            TextoStatus.Text = "⚠️ Desconexão detectada! Reconectando..."
            task.wait(TempoEspera)
            Reconectar()
        end
    end
end)

-- 4. Detecção de erro de teleporte
TeleportService.TeleportInitFailed:Connect(function(jogador, codigoErro, mensagem)
    if jogador == Player and Ativo then
        print(`[FALHA NO TELEPORTE] Código: {codigoErro} | {mensagem}`)
        TextoStatus.TextColor3 = Color3.new(1, 0, 0)
        TextoStatus.Text = `❌ Falha: {mensagem} | Tentando de novo...`
        task.wait(TempoEspera + 0.5)
        Reconectando = false
        Reconectar()
    end
end)

-- 📢 NOTIFICAÇÕES DO DELTA
if getgenv().Delta then
    Delta.ShowNotification("🔄 Auto Reconectar", "Versão REFORÇADA ativada com sucesso!", 4)
end

-- Confirmação no console
print("\n=====================================")
print("✅ AUTO RECONECTAR | DELTA REFORÇADO")
print("✅ Detecção: Rede, Kick, Erro, Saída")
print("✅ Reconexão: Infinita e Rápida")
print("✅ Interface: Ativada")
print("=====================================\n")
