# 🎮 GUIA PRÁTICO - Como Usar o WarZ Game em Unigine

## Seu Projeto Está Pronto! 

Agora você tem componentes C# que pode usar diretamente no editor Unigine.

---

## 🚀 PASSO 1: Abrir o Unigine Editor

```powershell
# Seu editor está pronto para usar!
# Ele carrega automaticamente com os componentes WarZ
```

---

## 📦 PASSO 2: Adicionar Componentes aos GameObjects

### Opção A: Criar um PlayerController (Personagem Jogável)

1. **No Editor Unigine:**
   - Clique em "Create → Node" (ou use o menu)
   - Nomeie como "Player"
   - No inspetor, clique em "Add Component"
   - Procure por "PlayerController"
   - Clique em "Add"

2. **Pronto!** Seu player pode agora:
   - ✅ Receber input (WASD, Mouse, Espaço)
   - ✅ Se mover e pular
   - ✅ Tomar dano
   - ✅ Morrer e respawnar

### Opção B: Criar um GameManager (Controlador do Jogo)

1. **No Editor Unigine:**
   - Clique em "Create → Node"
   - Nomeie como "GameManager"
   - Adicione componente "GameManager"

2. **Pronto!** Agora você tem:
   - ✅ Controle do estado do jogo
   - ✅ Gerenciamento de jogadores
   - ✅ Lógica central do jogo

### Opção C: Criar um NetworkManager (Multiplayer)

1. **No Editor Unigine:**
   - Clique em "Create → Node"
   - Nomeie como "NetworkManager"
   - Adicione componente "NetworkManager"
   - Configure `ServerAddress` e `ServerPort`

2. **Pronto!** Agora você tem:
   - ✅ Conexão com servidor
   - ✅ Sincronização de jogadores
   - ✅ Sistema de pacotes

---

## 🎮 PASSO 3: Testar no Editor

```
Seu code no AppWorldLogic.cs agora automaticamente:
1. Carrega o mundo
2. Inicializa os componentes
3. Mostra mensagens de status
```

**Verifique o console para ver:**
```
✅ WarZ Game - Unigine World Initialization
✅ Sistema pronto para usar
```

---

## 💡 EXEMPLO PRÁTICO: Criar um Jogo Simples

### Cenário: Um jogador que pode se mover e sofrer dano

#### Passo 1: Criar a Cena Base

```
Cena (World)
├── Player (Node)
│   └── PlayerController (Component)
├── GameManager (Node)
│   └── GameManager (Component)
└── Light (Existing)
```

#### Passo 2: Configurar o Player

1. Selecione o Node "Player"
2. No "Inspector", acesse "PlayerController"
3. Configure valores:
   - `moveSpeed` = 5
   - `sprintSpeed` = 8
   - `maxHealth` = 100

#### Passo 3: Testear!

Clique em **Play** (ou tecla Play no editor)

Você verá:
- ✅ Mensagens de inicialização
- ✅ Player aparece na cena
- ✅ GameManager criado automaticamente

Controles no jogo:
- **WASD** = Mover
- **Mouse** = Olhar ao redor
- **Shift** = Sprint
- **Espaço** = Pular

---

## 🔧 COMPONENTES DISPONÍVEIS

### 1. **PlayerController** (`GameLogic/Gameplay/PlayerController.cs`)

```csharp
// Propriedades
public float GetHealth() → Saúde atual
public float GetHealthPercent() → Porcentagem 0-1
public void TakeDamage(float damage) → Recebe dano
public void Heal(float amount) → Cura
public void Die() → Morte
public void SetPosition(vec3 pos) → Teleportar
public bool IsAlive() → Verifica se vivo
```

### 2. **DamageSystem** (`GameLogic/Gameplay/DamageSystem.cs`)

```csharp
// Métodos Estáticos
DamageSystem.ApplyDamage(target, damageInfo) → Aplica dano
DamageSystem.RaycastDamage(origin, direction, distance, damageInfo) → Raycast de tiro
DamageSystem.AreaDamage(center, radius, damageInfo) → Explosão/Área
DamageSystem.BulletPenetration(origin, direction, damage, out exitDamage) → Penetração
```

### 3. **GameManager** (`GameLogic/GameManager.cs`)

```csharp
// Singleton - Acesso: GameManager.Instance
public void StartGame() → Inicia
public void PauseGame() → Pausa
public void ResumeGame() → Retoma
public void EndGame() → Finaliza
public void RegisterPlayer(Node player) → Registra jogador
public void UnregisterPlayer(Node player) → Remove jogador
public List<Node> GetAllPlayers() → Lista jogadores
```

### 4. **NetworkManager** (`GameLogic/Multiplayer/NetworkManager.cs`)

```csharp
// Singleton - Acesso: NetworkManager.Instance
public bool ConnectToServer(string address, int port) → Conecta
public void DisconnectFromServer() → Desconecta
public void SendPacket(NetworkPacket packet) → Envia pacote
public RemotePlayer GetRemotePlayer(uint id) → Jogador remoto
```

### 5. **GameLevel** (`GameLogic/World/GameLevel.cs`)

```csharp
// Para adicionar componente a Node
public SpawnPoint GetRandomSpawnPoint() → Ponto de spawn
public List<SpawnPoint> GetAllSpawnPoints() → Todos spawns
public string GetLevelName() → Nome do nível
public int GetMaxPlayers() → Máx jogadores
```

### 6. **GameUIManager** (`GameLogic/UI/GameUIManager.cs`)

```csharp
// Singleton
public void UpdatePlayerHealth(float health, float maxHealth) → Atualiza HUD
public void ShowDamageIndicator(vec3 direction) → Mostra impacto
public void ShowKillNotification(string killer, string victim, string weapon) → Kill feed
```

### 7. **GameConfig** (`GameLogic/Common/GameConfig.cs`)

```csharp
// Static - Configurações Globais
GameConfig.ServerAddress = "127.0.0.1"
GameConfig.ServerPort = 27015
GameConfig.MaxPlayers = 32
GameConfig.GameMode = "Deathmatch"
GameConfig.PlayerHealth = 100f
GameConfig.MasterVolume = 1.0f
// E muito mais...
```

---

## 🎯 EXEMPLOS DE CÓDIGO

### Exemplo 1: Aplicar Dano com Raycast

```csharp
// Em qualquer script C# no Unigine
var origin = Node.WorldPosition;
var direction = (Node.WorldRotation * vec3.Forward).Normalized;

var damageInfo = new DamageSystem.DamageInfo
{
    DealerId = 1,
    Damage = 25,
    Type = DamageSystem.DamageType.Bullet
};

DamageSystem.RaycastDamage(origin, direction, 1000, damageInfo);
```

### Exemplo 2: Pegar Referência de um Jogador

```csharp
// Em um script UI ou manager
var allPlayers = GameManager.Instance.GetAllPlayers();
foreach (var player in allPlayers)
{
    Log.Message($"Player: {player.Name}");
}
```

### Exemplo 3: Conectar ao Servidor Multiplayer

```csharp
// No seu inicializador
if (NetworkManager.Instance != null)
{
    bool connected = NetworkManager.Instance.ConnectToServer("192.168.1.100", 27015);
    if (connected)
    {
        Log.Message("Connected!");
    }
}
```

### Exemplo 4: Criar Explosão

```csharp
// Explosão em uma área
var explosionCenter = Node.WorldPosition;
var damageInfo = new DamageSystem.DamageInfo
{
    Damage = 100,
    Type = DamageSystem.DamageType.Explosion
};

DamageSystem.AreaDamage(explosionCenter, 50, damageInfo);
```

---

## 🐛 SOLUÇÃO DE PROBLEMAS

### Problema: "Component not found"
**Solução**: 
- Certifique-se que o projeto foi compilado (`dotnet build`)
- Recarregue o editor Unigine

### Problema: Player não se move
**Solução**:
- Verifique se Input está habilitado
- Confirme que não está em pausa

### Problema: Componente não aparece no menu "Add Component"
**Solução**:
- Clique em "Refresh" no editor
- Ou reinicie o editor

### Problema: Rede não conecta
**Solução**:
- Verifique `GameConfig.ServerAddress`
- Confirme porta 27015 está aberta
- Servidor está rodando?

---

## 📊 ESTRUTURA DO PROJETO ATUAL

```
unigine_project/
├── source/
│   ├── GameLogic/              ← Seus componentes WarZ!
│   │   ├── GameManager.cs
│   │   ├── GameBootstrapper.cs
│   │   ├── World/GameLevel.cs
│   │   ├── Multiplayer/NetworkManager.cs
│   │   ├── Gameplay/PlayerController.cs
│   │   ├── Gameplay/DamageSystem.cs
│   │   ├── UI/GameUIManager.cs
│   │   └── Common/GameConfig.cs
│   ├── AppWorldLogic.cs        ← Hook para o mundo
│   ├── AppSystemLogic.cs
│   ├── AppEditorLogic.cs
│   └── main.cs
├── data/
│   ├── configs/game.cfg        ← Configurações
│   └── unigine_project.world   ← Seu mundo
└── bin/                        ← Binários compilados
```

---

## 🎮 PRÓXIMOS PASSOS

1. **Abra o Unigine Editor** (ele já está pronto!)
2. **Crie alguns GameObjects** com componentes
3. **Configure as propriedades** no inspector
4. **Clique em Play** para testar
5. **Veja o console** para mensagens de debug

---

## 📞 CHEAT SHEET - Comandos Rápidos

```csharp
// Acessar Singletons
GameManager.Instance              // Controlador do jogo
NetworkManager.Instance           // Rede
GameUIManager.Instance            // Interface

// Configurações
GameConfig.MaxPlayers = 64
GameConfig.ServerAddress = "myserver.com"
GameConfig.SaveConfig()

// Debug
Log.Message("Minha mensagem")
Log.Warning("Aviso")
Log.Error("Erro!")

// Input
Input.IsKeyPressed(Input.KEY_W)
Input.MouseDelta
Input.IsKeyDown(Input.KEY_SPACE)
```

---

## ✅ VALIDAÇÃO

```
✅ Componentes C#:      Funcionando
✅ Build:              Sem erros
✅ Unigine:            2.20.0.0 (rodando)
✅ .NET SDK:           8.0 (pronto)
✅ Projeto:            Pronto para usar
```

---

**Agora você está 100% pronto para começar a desenvolver seu jogo WarZ em Unigine!** 🚀🎮

Abra o editor e divirta-se! ✨
