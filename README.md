-- [[ DASH HUB LOADER - FORCE FIX FINAL ]] --

-- 1. Destrava as variáveis de segurança para o script não fechar
getgenv().DashHub_Auth_Verified = true
getgenv().DashHub_ExecutadoPeloLoader = true
_G.DashHub_Auth_Verified = true
_G.DashHub_ExecutadoPeloLoader = true

-- 2. Função "Trator": Limpa a sujeira e arruma o link
local function CarregarNaForca(url, nomeJogo)
    -- Correção automática de espaços no link (Transforma espaço em %20)
    -- Isso é necessário para o link "Dash Hub BETA" funcionar
    local urlCorrigida = url:gsub(" ", "%%20")
    
    local sucesso, conteudo = pcall(function()
        return game:HttpGet(urlCorrigida)
    end)

    if not sucesso then
        return warn("CRÍTICO: Não foi possível acessar o link: " .. urlCorrigida)
    end

    -- ALGORITMO DE LIMPEZA (Remove lixo de README.md e Markdown)
    local codigoLimpo = conteudo:gsub("^#.-\n", "") -- Remove título #
    codigoLimpo = codigoLimpo:gsub("```lua", "")    -- Remove início de bloco
    codigoLimpo = codigoLimpo:gsub("```", "")       -- Remove fim de bloco
    
    local func, erro = loadstring(codigoLimpo)
    
    if func then
        task.spawn(func)
        print("✅ " .. nomeJogo .. " Carregado com Sucesso.")
    else
        -- Tenta rodar o original se a limpeza falhar
        local func2 = loadstring(conteudo)
        if func2 then
            task.spawn(func2)
        else
            warn("ERRO NO CÓDIGO DO GITHUB: " .. tostring(erro))
        end
    end
end

-- 3. Sistema de Hitbox (Aumentando para Cima conforme solicitado)
task.spawn(function()
    local Players = game:GetService("Players")
    local lp = Players.LocalPlayer
    
    while task.wait(1) do
        pcall(function()
            for _, p in pairs(Players:GetPlayers()) do
                if p ~= lp and p.Team ~= lp.Team and p.Character then
                    local hrp = p.Character:FindFirstChild("HumanoidRootPart")
                    if hrp then
                        -- Aumenta para cima (Y = 20) e pros lados (10)
                        hrp.Size = Vector3.new(10, 20, 10)
                        hrp.Transparency = 0.7
                        hrp.CanCollide = false
                    end
                end
            end
        end)
    end
end)

-- 4. Execução dos Links Solicitados
local id = game.PlaceId

if id == 286090429 then
    -- Link Arsenal solicitado
    CarregarNaForca("https://raw.githubusercontent.com/WB-LITEN/dash-hub-script-arsenal/refs/heads/main/README.md", "Arsenal")

elseif id == 11815767793 then
    -- Link UBG/BETA solicitado (com tratamento de espaço automático)
    CarregarNaForca("https://raw.githubusercontent.com/WB-LITEN/Dash-Hub-BETA/refs/heads/main/Dash%20Hub%20BETA", "UBG/BETA")

else
    -- Se não for nenhum dos dois, tenta rodar o BETA como padrão
    warn("Jogo não identificado (ID: "..id.."), tentando carregar BETA...")
    CarregarNaForca("https://raw.githubusercontent.com/WB-LITEN/Dash-Hub-BETA/refs/heads/main/Dash%20Hub%20BETA", "Fallback")
end
