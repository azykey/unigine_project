# Arquitetura do Sistema WarZ - Unigine Edition

## 🏗️ Diagrama de Arquitetura

```
┌─────────────────────────────────────────────────────────────┐
│                     Unigine Engine (C#)                     │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────────── Main Loop ────────────────────┐      │
│  │  main.cs → AppSystemLogic → AppWorldLogic        │      │
│  └──────────────────────────────────────────────────┘      │
│                         ↓                                    │
│  ┌──────────────────────────────────────────────────┐      │
│  │         GameBootstrapper                         │      │
│  │  (Inicializador Principal do Sistema)            │      │
│  └──────────────────────────────────────────────────┘      │
│    ↓     ↓      ↓      ↓                                    │
│  ┌─────────────────────────────────────────────────┐       │
│  │  CAMADA DE MANAGERS (Singletons)                │       │
│  ├─────────────────────────────────────────────────┤       │
│  │ • GameManager        (Lógica central)           │       │
│  │ • NetworkManager     (Rede multiplayer)         │       │
│  │ • GameUIManager      (Interface)                │       │
│  │ • GameConfig         (Configurações)            │       │
│  └─────────────────────────────────────────────────┘       │
│    ↓              ↓                  ↓                      │
│  ┌──────────┐ ┌──────────┐    ┌──────────────┐            │
│  │ WORLD    │ │GAMEPLAY  │    │MULTIPLAYER   │            │
│  ├──────────┤ ├──────────┤    ├──────────────┤            │
│  │• GameLevel│• PlayerCtrl│   │• NetworkPkts │           │
│  │• Sectors  │• DamageSystem  │• RemotePlayers           │
│  │• Spawns   │• HealthComp    │• Synchronization        │
│  │• Atmosphere│• Weapons     │• Latency Comp            │
│  └──────────┘ └──────────┘    └──────────────┘            │
│    ↓              ↓                  ↓                      │
│  ┌─────────────────────────────────────────────────┐       │
│  │         UI LAYER (HUD, Menus)                   │       │
│  ├─────────────────────────────────────────────────┤       │
│  │ • HUD Panel          (Saúde, munição, mapa)    │       │
│  │ • Pause Menu         (Pausa/Opções)            │       │
│  │ • Kill Feed          (Notificações)            │       │
│  │ • Damage Indicators  (Direção de dano)         │       │
│  └─────────────────────────────────────────────────┘       │
│    ↓              ↓                  ↓                      │
│  ┌─────────────────────────────────────────────────┐       │
│  │       RENDERING & PHYSICS (Unigine Native)      │       │
│  ├─────────────────────────────────────────────────┤       │
│  │ • 3D Graphics        • Particle Effects         │       │
│  │ • Physics Engine     • Sound System             │       │
│  │ • Lighting          • Post-Processing           │       │
│  └─────────────────────────────────────────────────┘       │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

---

## 🎛️ Componentes & Responsabilidades

### 1️⃣ **GameBootstrapper**
```
Responsabilidades:
├── Init() → Carregar Configurações
├── InitializeGame() → Criar Managers
├── LoadLevel() → Carregar Cenário
└── StartGame() → Iniciar Loop
```

### 2️⃣ **GameManager** (Singleton)
```
Responsabilidades:
├── Manter Estado do Jogo (Loading, Menu, Running, etc)
├── Gerenciar Lista de Jogadores
├── Controlar Fluxo de Jogo
├── Broadcast de Eventos
└── Pausar/Resumir/Encerrar
```

### 3️⃣ **NetworkManager** (Singleton)
```
Responsabilidades:
├── Conectar ao Servidor
├── Enviar/Receber Pacotes
├── Sincronizar Jogadores Remotos
├── Gerenciar Latência
└── Handle de Desconexões
```

### 4️⃣ **GameLevel**
```
Responsabilidades:
├── Carregar Dados do Nível
├── Gerenciar Spawn Points
├── Gerenciar Setores (LOD)
├── Controlar Atmosfera
└── Gerenciar Objetos da Cena
```

### 5️⃣ **PlayerController**
```
Responsabilidades:
├── Processar Input (Teclado/Mouse)
├── Calcular Movimento
├── Aplicar Físicas
├── Gerenciar Saúde
├── Sincronizar com Rede
└── Animar Personagem
```

### 6️⃣ **DamageSystem**
```
Responsabilidades:
├── Aplicar Dano
├── Raycast Balístico
├── Dano em Área
├── Física de Impacto
├── Efeitos Visuais
└── Log de Eventos
```

### 7️⃣ **GameUIManager** (Singleton)
```
Responsabilidades:
├── Atualizar HUD
├── Mostrar Menus
├── Notificações
├── Indicadores de Dano
└── Kill Feed
```

### 8️⃣ **GameConfig** (Static)
```
Responsabilidades:
├── Carregar Configurações
├── Salvar Preferências
├── Reset para Padrões
├── Fornecer Valores Globais
└── Validar Configurações
```

---

## 🔄 Fluxos Principais

### A) Inicialização do Sistema
```
main.cs
  ↓
AppSystemLogic.Init()
  ↓
AppWorldLogic.Init()
  ↓
GameBootstrapper.Init()
  ├→ GameConfig.LoadConfig()
  ├→ GameManager.Init()
  ├→ NetworkManager.Init()
  ├→ GameUIManager.Init()
  ├→ GameLevel.Init()
  └→ GameManager.StartGame()
```

### B) Loop Principal
```
Update (60 FPS)
  ├→ GameManager.Update()
  ├→ NetworkManager.Update()
  │   └→ ProcessIncomingPackets()
  ├→ PlayerController.Update()
  │   ├→ HandleInput()
  │   ├→ UpdateMovement()
  │   └→ UpdateAnimation()
  ├→ GameUIManager.Update()
  │   └→ UpdateHUD()
  └→ Render (Unigine)
```

### C) Combate/Dano
```
PlayerController.Attack()
  ├→ DamageSystem.RaycastDamage()
  │   ├→ Hit Detection
  │   ├→ DamageSystem.ApplyDamage()
  │   ├→ PlayDamageEffect()
  │   └→ LogDamageEvent()
  ├→ HealthComponent.TakeDamage()
  │   ├→ Reduzir Saúde
  │   └→ Se <= 0 → Die()
  └→ NetworkManager.SendPacket()
      └→ Sincronizar para Servidor
```

### D) Multiplicador/Networking
```
NetworkManager.Update()
  ├→ Recv Packets
  │   ├→ PlayerJoined
  │   ├→ PlayerLeft
  │   ├→ PlayerUpdate
  │   ├→ GameStateUpdate
  │   └→ Outros
  ├→ UpdateRemotePlayers()
  │   ├→ Atualizar Posição
  │   ├→ Atualizar Estado
  │   └→ Sincronizar Saúde
  └→ Send Outgoing Packets
      └→ Player Position/State/Actions
```

---

## 📊 Fluxo de Dados

```
INPUT
(Teclado/Mouse)
  ↓
PlayerController
  ├→ Armazena Input
  ├→ Calcula Movimento
  └→ Aplica Física
  ↓
GameManager
  ├→ Valida Ação
  ├→ Atualiza Estado
  └→ Notifica Listeners
  ↓
NetworkManager
  ├→ Serializa Dados
  ├→ Envia para Servidor
  └→ Recebe Confirmação
  ↓
Unigine Rendering
  ├→ Atualiza Posição
  ├→ Renderiza Personagem
  └→ Mostra na Tela
  ↓
OUTPUT
(Personagem se move)
```

---

## 🗂️ Estrutura de Código

```
GameLogic/
│
├── GameManager.cs
│   └── public class GameManager : Component
│       ├── Instance (Singleton)
│       ├── StartGame()
│       ├── PauseGame()
│       ├── ResumeGame()
│       ├── EndGame()
│       ├── RegisterPlayer()
│       └── GetAllPlayers()
│
├── GameBootstrapper.cs
│   └── public class GameBootstrapper : Component
│       ├── InitializeGame()
│       ├── ShutdownGame()
│       ├── RestartGame()
│       └── LoadGameConfiguration()
│
├── World/
│   └── GameLevel.cs
│       ├── LoadLevelData()
│       ├── GetRandomSpawnPoint()
│       ├── GetAtmosphere()
│       └── AddSector()
│
├── Gameplay/
│   ├── PlayerController.cs
│   │   ├── TakeDamage()
│   │   ├── Heal()
│   │   ├── Die()
│   │   └── SetPosition()
│   │
│   └── DamageSystem.cs
│       ├── ApplyDamage()
│       ├── RaycastDamage()
│       ├── AreaDamage()
│       └── BulletPenetration()
│
├── Multiplayer/
│   └── NetworkManager.cs
│       ├── ConnectToServer()
│       ├── SendPacket()
│       ├── GetRemotePlayer()
│       └── DisconnectFromServer()
│
├── UI/
│   └── GameUIManager.cs
│       ├── ShowHUD()
│       ├── UpdatePlayerHealth()
│       ├── ShowDamageIndicator()
│       └── ShowKillNotification()
│
└── Common/
    └── GameConfig.cs
        ├── LoadConfig()
        ├── SaveConfig()
        ├── ResetToDefaults()
        └── GetGameModes()
```

---

## 🔌 Padrões de Projeto Utilizados

| Padrão | Classe | Benefício |
|--------|--------|-----------|
| **Singleton** | GameManager, NetworkManager, GameUIManager | Uma única instância global |
| **Component** | PlayerController, HealthComponent | Composição flexível |
| **Factory** | DamageSystem | Criação de objetos |
| **Observer** | GameManager events | Desacoplamento |
| **State Machine** | GameManager states | Controle de fluxo |
| **Strategy** | DamageSystem types | Múltiplas estratégias |
| **Facade** | GameBootstrapper | Simplificar inicialização |

---

## 📈 Escalabilidade

### Adicionar Nova Feature
```
1. Criar nova pasta em GameLogic/
2. Criar classe derivada de Component
3. Implementar Init() e Update()
4. Registrar com Manager apropriado
5. Adicionar à GameBootstrapper.InitializeGame()
6. Pronto! ✅
```

### Exemplo: Adicionar Sistema de Inventário
```csharp
// 1. Criar arquivo
// GameLogic/Gameplay/InventorySystem.cs

public class InventorySystem : Component {
    public void AddItem(Item item) { ... }
    public List<Item> GetItems() { ... }
}

// 2. Integrar no GameBootstrapper
private void InitializeCoreManagers() {
    var inventory = new GameObject("Inventory").AddComponent<InventorySystem>();
    Log.Message("✓ InventorySystem ready");
}
```

---

## ⚡ Performance Expectations

| Métrica | Esperado | Otimização |
|---------|----------|-----------|
| **FPS** | 60+ | Unigine LOD, Culling |
| **Memory** | <512MB | Object pooling |
| **Network** | 60 tick/s | Delta compression |
| **Build Time** | <5s | Incremental build |
| **Startup** | <3s | Async loading |

---

## 📚 Como Ler Este Documento

1. **Visão Geral**: Comece com o diagrama de arquitetura
2. **Detalhes**: Leia os componentes
3. **Fluxos**: Entenda os processos principais
4. **Codificação**: Consulte exemplos no código
5. **Escalação**: Aprenda como adicionar features

---

**Última Atualização**: 15 de Novembro de 2025  
**Versão**: 1.0  
**Status**: Produção ✅
