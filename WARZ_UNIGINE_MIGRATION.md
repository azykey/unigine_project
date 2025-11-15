# Migração WarZ para Unigine - Documentação

## 📋 Overview

O projeto **WarZ** foi convertido de **C++ (Visual Studio 2026)** para **C# (Unigine Engine)**.

Esta é uma migração completa que inclui:
- ✅ Game Manager e lógica de jogo
- ✅ Sistema de levels e world
- ✅ Multiplayer networking
- ✅ Player controller e gameplay
- ✅ Damage system e combate
- ✅ UI e HUD
- ✅ Configurações de jogo

---

## 📁 Estrutura de Pastas

```
source/
├── GameLogic/                    # Código principal do jogo
│   ├── GameManager.cs            # Gerenciador principal do jogo
│   ├── GameBootstrapper.cs       # Inicializador do sistema
│   ├── World/
│   │   └── GameLevel.cs          # Gerenciador de níveis
│   ├── Multiplayer/
│   │   └── NetworkManager.cs     # Sistema de rede
│   ├── Gameplay/
│   │   ├── PlayerController.cs   # Controle do jogador
│   │   └── DamageSystem.cs       # Sistema de dano e combate
│   ├── UI/
│   │   └── GameUIManager.cs      # Interface do usuário
│   └── Common/
│       └── GameConfig.cs         # Configurações globais
├── AppSystemLogic.cs             # Lógica de sistema Unigine
├── AppWorldLogic.cs              # Lógica de mundo Unigine
├── AppEditorLogic.cs             # Lógica de editor
└── main.cs                       # Ponto de entrada

data/                            # Assets e dados do jogo
├── configs/                     # Arquivos de configuração
├── scripts.ung                  # Scripts compilados
└── unigine_project.world        # Cena do mundo
```

---

## 🎮 Componentes Principais

### 1. **GameManager** (`GameLogic/GameManager.cs`)
- Gerenciador central do jogo
- Controla estado do jogo (Loading, Menu, GameRunning, etc.)
- Gerencia lista de jogadores
- Singleton pattern para acesso global

### 2. **GameBootstrapper** (`GameLogic/GameBootstrapper.cs`)
- Inicializa o sistema completo de jogo
- Carrega configurações
- Cria managers core
- Inicia nível
- Sequência de inicialização: Config → Managers → Level → Start

### 3. **GameLevel** (`GameLogic/World/GameLevel.cs`)
- Representa um nível/mapa do jogo
- Gerencia spawn points
- Carrega dados do nível
- Gerencia atmosfera e ambiente

### 4. **NetworkManager** (`GameLogic/Multiplayer/NetworkManager.cs`)
- Comunicação com servidor
- Sincronização de jogadores remotos
- Fila de pacotes de rede
- Suporta múltiplos tipos de pacotes

### 5. **PlayerController** (`GameLogic/Gameplay/PlayerController.cs`)
- Controle do personagem jogável
- Movimento, salto, sprint
- Sistema de saúde
- Input handling (teclado/mouse)

### 6. **DamageSystem** (`GameLogic/Gameplay/DamageSystem.cs`)
- Cálculo de dano
- Impacto de projéteis
- Dano em área (explosões)
- Sistema de armas
- Efeitos visuais de impacto

### 7. **GameUIManager** (`GameLogic/UI/GameUIManager.cs`)
- Gerenciador de interface do usuário
- HUD (vida, munição, radar)
- Menu de pausa
- Notificações de jogo

### 8. **GameConfig** (`GameLogic/Common/GameConfig.cs`)
- Configurações centralizadas
- Parâmetros de servidor, jogador, combate, gráficos, áudio, rede
- Carregamento de arquivos de configuração
- Reset para padrões

---

## 🚀 Como Usar

### Inicializar o Jogo

1. **Adicionar GameBootstrapper à cena**
   ```csharp
   var bootstrapperObj = new GameObject("Bootstrapper");
   var bootstrapper = bootstrapperObj.AddComponent<GameBootstrapper>();
   bootstrapper.AutoInitialize = true;
   ```

2. **Ou usar via código**
   ```csharp
   GameBootstrapper.Instance.InitializeGame();
   ```

### Acessar Managers

```csharp
// Game Manager
var gameManager = GameManager.Instance;
gameManager.StartGame();
gameManager.PauseGame();

// Network Manager
var networkManager = NetworkManager.Instance;
networkManager.ConnectToServer("127.0.0.1", 27015);

// UI Manager
var uiManager = GameUIManager.Instance;
uiManager.UpdatePlayerHealth(80, 100);
uiManager.ShowKillNotification("Player1", "Player2", "Rifle");
```

### Configurar Jogo

```csharp
// Carregar configurações
GameConfig.LoadConfig();

// Acessar configurações
Debug.LogFormat("Max Players: {0}", GameConfig.MaxPlayers);
Debug.LogFormat("Server: {0}:{1}", GameConfig.ServerAddress, GameConfig.ServerPort);

// Modificar configurações
GameConfig.GameMode = "Team Deathmatch";
GameConfig.MaxPlayers = 64;
GameConfig.SaveConfig();
```

### PlayerController em Ação

```csharp
// No seu script de jogo
var playerObj = new GameObject("Player");
var controller = playerObj.AddComponent<PlayerController>();

// Tomar dano
controller.TakeDamage(25);

// Curar
controller.Heal(10);

// Verificar status
if (controller.IsAlive())
{
    Debug.LogFormat("Health: {0}%", controller.GetHealthPercent() * 100);
}
```

### Damage System

```csharp
// Criar info de dano
var damageInfo = new DamageSystem.DamageInfo
{
    DealerId = playerId,
    Damage = 25,
    DamagePosition = hitPosition,
    DamageDirection = shootDirection,
    Type = DamageSystem.DamageType.Bullet
};

// Aplicar dano
DamageSystem.ApplyDamage(targetObject, damageInfo);

// Ou usar raycast
DamageSystem.RaycastDamage(origin, direction, 1000, damageInfo);

// Ou dano em área
DamageSystem.AreaDamage(explosionCenter, 50, damageInfo);
```

---

## 🔄 Fluxo de Inicialização

```
main.cs (Entry Point)
    ↓
AppSystemLogic.Init()
    ↓
AppWorldLogic.Init()
    ↓
GameBootstrapper.Init()
    ↓
LoadGameConfiguration() [GameConfig]
    ↓
InitializeCoreManagers() [GameManager, NetworkManager, GameUIManager]
    ↓
LoadLevel() [GameLevel]
    ↓
StartGame() [GameManager.StartGame()]
    ↓
Main Loop (Update)
```

---

## 📝 Exemplos de Uso

### Exemplo 1: Criar um Jogador

```csharp
public void SpawnPlayer(string playerName)
{
    var spawnPoint = GameLevel.Instance.GetRandomSpawnPoint();
    var playerObj = new GameObject(playerName);
    playerObj.WorldPosition = spawnPoint.GetSpawnPosition();
    playerObj.WorldRotation = spawnPoint.GetSpawnRotation();
    
    var controller = playerObj.AddComponent<PlayerController>();
    GameManager.Instance.RegisterPlayer(playerObj);
    
    Log.Message($"Player {playerName} spawned at {spawnPoint.SpawnName}");
}
```

### Exemplo 2: Conectar ao Servidor

```csharp
public void ConnectToGame()
{
    var connected = NetworkManager.Instance.ConnectToServer(
        GameConfig.ServerAddress,
        GameConfig.ServerPort
    );
    
    if (connected)
    {
        Log.Message("Connected to server!");
        GameManager.Instance.StartGame();
    }
}
```

### Exemplo 3: Sistema de Armas

```csharp
public class WeaponSystem : Component
{
    private DamageSystem.Weapon rifle = new DamageSystem.Weapon(
        "M16 Rifle", 
        damage: 25, 
        fireRate: 0.1f, 
        range: 500, 
        type: DamageSystem.WeaponType.Rifle
    );
    
    public void Fire(vec3 position, vec3 direction)
    {
        if (rifle.CanShoot())
        {
            rifle.Shoot();
            
            var damageInfo = new DamageSystem.DamageInfo
            {
                Damage = rifle.Damage,
                DamageDirection = direction,
                Type = DamageSystem.DamageType.Bullet
            };
            
            DamageSystem.RaycastDamage(position, direction, rifle.Range, damageInfo);
        }
    }
}
```

---

## 🔧 Configuração Recomendada

### Servidor
- **Endereço**: 127.0.0.1 (desenvolvimento) ou IP público (produção)
- **Porta**: 27015 (padrão)
- **Max Players**: 32-64

### Jogo
- **Modo**: Deathmatch, Team Deathmatch, Capture Flag
- **Duração**: 3600s (1 hora)
- **Respawn Time**: 5s

### Gráficos
- **Resolução**: 1920x1080 (recomendado)
- **FPS**: 60 (target)
- **VSync**: Ativado
- **FOV**: 90°

### Áudio
- **Master**: 1.0 (100%)
- **Music**: 0.8 (80%)
- **SFX**: 1.0 (100%)

---

## 🐛 Troubleshooting

### Problema: "GameManager instance not found!"
**Solução**: Verifique se o GameBootstrapper foi adicionado e inicializado à cena.

### Problema: Conexão de rede falhando
**Solução**: Verifique GameConfig.ServerAddress e ServerPort. Teste com 127.0.0.1:27015.

### Problema: Jogador não recebendo entrada
**Solução**: Certifique-se de que o PlayerController está ativado e a janela tem foco.

### Problema: Dano não sendo aplicado
**Solução**: Verifique se o objeto-alvo tem HealthComponent ou PlayerController.

---

## 📚 Referências

- **Código Original**: C++ de `TANAWANT-THONGPING-STUDIO`
- **Engine Target**: Unigine (C#)
- **Namespace**: `WarZGame`
- **Versão**: 1.0

---

## ✅ Checklist de Funcionalidades

- [x] Game Manager
- [x] Level System
- [x] Network Manager
- [x] Player Controller
- [x] Damage System
- [x] Weapon System
- [x] UI Manager
- [x] Configuration System
- [x] Bootstrapper
- [ ] AI Enemies (próxima fase)
- [ ] Inventory System (próxima fase)
- [ ] Missions/Quests (próxima fase)
- [ ] Leaderboard (próxima fase)

---

**Última atualização**: 15 de Novembro de 2025
**Status**: Migração Completa v1.0 ✅
