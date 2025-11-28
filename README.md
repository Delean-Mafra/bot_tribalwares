# 🏰 Tribal Wars Bot - Extensão de Automação

**Copyright © Delean Mafra**  
Todos os direitos reservados

**Licença:** Creative Commons BY-NC 4.0  
https://creativecommons.org/licenses/by-nc/4.0/deed.pt-br

---

## 📋 Descrição

Extensão de navegador para automação de ações no jogo Tribal Wars. O bot executa tarefas repetitivas automaticamente, permitindo que você se concentre na estratégia do jogo.

## ✨ Funcionalidades

### PHASE 1 - Construção Automática
- Constrói edifícios automaticamente seguindo uma ordem estratégica otimizada
- Verifica a cada 10 segundos se há edifícios disponíveis para construção
- Segue uma sequência de construção baseada em guias profissionais do jogo
- Opção para aguardar ordem específica ou construir o que estiver disponível primeiro

### PHASE 2 - Construção + Farm Automático
- Todas as funcionalidades da PHASE 1
- Ataque automático a aldeias (farming) para coleta de recursos
- Evita atacar aldeias que já estão sendo atacadas
- Suporte para diferentes templates de tropas (lanças, espadas, machados, cavalaria leve)
- Intervalo aleatório entre ações (20-40 segundos por padrão) para simular comportamento humano

## 🎮 Instalação

### Chrome/Edge/Brave
1. Baixe ou clone este repositório
2. Abra `chrome://extensions/` (ou `edge://extensions/`)
3. Ative o **"Modo do desenvolvedor"**
4. Clique em **"Carregar sem compactação"**
5. Selecione a pasta `Tribalwars`

### Firefox
1. Baixe ou clone este repositório
2. Abra `about:debugging#/runtime/this-firefox`
3. Clique em **"Carregar extensão temporária"**
4. Selecione o arquivo `manifest.json` na pasta `Tribalwars`

## ⚙️ Configuração

Clique no ícone da extensão na barra de ferramentas para acessar o painel de controle:

### Configurações Disponíveis

#### Bot Ativo
- Liga/desliga o bot
- Quando desativado, nenhuma ação será executada

#### Modo de Operação
- **PHASE 1**: Apenas construção automática de edifícios
- **PHASE 2**: Construção + farm automático de aldeias

#### Tempo de Espera
- **Mínimo**: Tempo mínimo entre ações (padrão: 20 segundos)
- **Máximo**: Tempo máximo entre ações (padrão: 40 segundos)
- O bot escolhe um tempo aleatório entre estes valores para cada ação

#### Aguardar Ordem de Construção
- **Ativado**: Segue rigorosamente a ordem de construção predefinida
- **Desativado**: Constrói qualquer edifício disponível na fila

#### Coordenadas para Farm (PHASE 2)
- Lista de coordenadas de aldeias para atacar
- Uma coordenada por linha no formato: `XXX|YYY`
- Exemplo: `500|500`

#### Template de Tropas (PHASE 2)
- **Set 1**: 10 Lanças + 10 Espadas
- **Set 2**: 15 Lanças + 3 Machados
- **Set 3**: 8 Cavalaria Leve (padrão)

## 🏗️ Arquitetura do Código

### Estrutura de Arquivos

```
Tribalwars/
├── manifest.json       # Configuração da extensão
├── content.js          # Script principal (lógica do bot)
├── popup.html          # Interface do painel de controle
├── popup.js            # Lógica da interface
├── icon16.png          # Ícone 16x16
├── icon48.png          # Ícone 48x48
├── icon128.png         # Ícone 128x128
└── README.md           # Esta documentação
```

### content.js - Script Principal

#### Variáveis Globais
- `MIN_WAIT_TIME`: Tempo mínimo de espera (ms)
- `MAX_WAIT_TIME`: Tempo máximo de espera (ms)
- `PHASE`: Modo de operação atual
- `WAIT_FOR_ORDER_BUILDINGS`: Aguardar ordem de construção
- `FARM_COORDINATES`: Array de coordenadas para farm
- `FARM_TROOP_SET`: Template de tropas selecionado
- `BOT_ENABLED`: Estado de ativação do bot
- `farmTroopSets`: Objeto com templates de tropas predefinidos

#### Funções Principais

##### `loadSettings()`
Carrega configurações salvas do Chrome Storage.

```javascript
return new Promise((resolve) => {
    chrome.storage.sync.get({...}, function(items) {
        // Aplica configurações
    });
});
```

##### `executePhase1()`
Executa a fase 1 (apenas construção):
1. Detecta a tela atual do jogo
2. Se estiver no quartel general, inicia loop de construção
3. Se estiver na visão geral, navega para o quartel general

##### `executePhase2()`
Executa a fase 2 (construção + farm):
1. Calcula delay aleatório entre ações
2. Detecta a tela atual
3. Constrói edifícios quando no quartel general
4. Envia ataques quando no ponto de encontro
5. Confirma ataques quando solicitado

##### `getCurrentView()`
Identifica a tela atual do jogo pela URL:
- `overview` → OVERVIEW_VIEW
- `main` → HEADQUARTERS_VIEW
- `place` → RALLY_POINT_VIEW
- `confirm` → ATTACK_CONFIRM_VIEW

##### `buildNextBuilding()`
Tenta construir o próximo edifício disponível:
1. Busca próximo edifício na fila
2. Verifica se está visível e clicável
3. Executa o clique

##### `getNextBuildingElement()`
Retorna o próximo elemento de edifício a ser construído:
- Itera pela fila de construção (`getBuildingElementsQueue()`)
- Verifica se o edifício está disponível
- Respeita a configuração `WAIT_FOR_ORDER_BUILDINGS`

##### `getBuildingElementsQueue()`
Retorna array com ordem de construção otimizada:
- Baseado em guias profissionais do Tribal Wars
- Sequência de ~140 edifícios
- IDs no formato `main_buildlink_<tipo>_<nivel>`

##### `sendFarmAttacks()`
Envia ataques de farm:
1. Verifica tropas disponíveis
2. Obtém lista de aldeias em ataque
3. Escolhe aleatoriamente uma aldeia não atacada
4. Envia o ataque

##### `getAvailableInputs()`
Verifica se há tropas suficientes para o template selecionado:
- Retorna objeto com inputs e quantidades
- Retorna `undefined` se tropas insuficientes

##### `sendAttackToCoordinate(coordinates, inputAmounts)`
Executa o ataque:
1. Preenche campos de tropas
2. Insere coordenadas
3. Clica no botão de ataque

### popup.html/popup.js - Interface

#### Componentes da Interface
- Toggle switches para ativar/desativar opções
- Select boxes para escolha de modos
- Inputs numéricos para tempos de espera
- Textarea para coordenadas
- Botão de salvar com feedback visual

#### Funcionalidades
- **Carregamento**: Busca configurações salvas
- **Atualização em tempo real**: Mostra/esconde seções conforme modo
- **Salvamento**: Persiste configurações e notifica content script
- **Feedback visual**: Confirmação ao salvar (botão verde)

### manifest.json - Configuração da Extensão

```json
{
  "manifest_version": 3,
  "name": "Tribal Wars Bot",
  "permissions": ["storage"],
  "host_permissions": [
    "https://*.tribalwars.net/*",
    "https://*.tribalwars.com.br/*"
  ],
  "content_scripts": [{
    "matches": [...],
    "js": ["content.js"],
    "run_at": "document_end"
  }]
}
```

## 🔒 Segurança e Limitações

### Detecção Anti-Bot
- Use intervalos de tempo realistas (20-40s recomendado)
- Evite farmar 24/7
- Varie os templates de tropas
- Não use em contas principais sem cautela

### Limitações Técnicas
- Requer que os elementos HTML do jogo mantenham suas IDs
- Funciona apenas nas telas suportadas
- Não detecta mudanças na interface do jogo automaticamente

## 📝 Ordem de Construção

A sequência de construção segue a estratégia:

1. **Recursos Iniciais**: Argila/Madeira/Ferro níveis 1-10
2. **Infraestrutura**: Quartel General, Armazém, Mercado
3. **Militar**: Quartel, Ferraria, Estábulo
4. **Defesa**: Muralha
5. **Expansão**: Fazenda para população
6. **Otimização**: Balanceamento de todos os edifícios até nível 20

Baseado em: https://forum.tribalwars.us/index.php?threads/start-up-guide-by-purple-predator.224/

## 🛠️ Desenvolvimento

### Estrutura de Dados

#### Storage
```javascript
{
  botEnabled: boolean,
  phase: 'PHASE_1' | 'PHASE_2',
  minWait: number,
  maxWait: number,
  waitForOrder: boolean,
  farmCoords: string,
  farmTroopSet: string
}
```

#### Mensagens (Runtime)
```javascript
{
  action: 'reloadSettings',
  settings: { /* objeto storage */ }
}
```

### Adicionar Novos Templates de Tropas

Edite `content.js`:

```javascript
let farmTroopSets = {
    "FARM_TROOP_SET_4": {
        "spear": 20,
        "sword": 15,
        "axe": 5
    }
};
```

Edite `popup.html`:

```html
<option value="FARM_TROOP_SET_4">Set 4: Custom</option>
```

## 🐛 Solução de Problemas

### Bot não está funcionando
1. Verifique se está ativado no popup
2. Abra o console (F12) e procure mensagens de log
3. Recarregue a extensão
4. Recarregue a página do jogo

### Erro: "Cannot read properties of null"
- O jogo mudou a estrutura HTML
- Os IDs dos elementos podem estar diferentes
- Verifique o console para identificar qual elemento está faltando

### Configurações não salvam
- Verifique as permissões de storage no manifest
- Limpe o cache da extensão
- Reinstale a extensão

## 📜 Licença

Este projeto está licenciado sob **Creative Commons Atribuição-NãoComercial 4.0 Internacional (CC BY-NC 4.0)**.

### Você tem o direito de:
✅ **Compartilhar** — copiar e redistribuir o material em qualquer suporte ou formato  
✅ **Adaptar** — remixar, transformar e criar a partir do material

### Sob as seguintes condições:
⚠️ **Atribuição** — Você deve dar o crédito apropriado, prover um link para a licença e indicar se mudanças foram feitas  
⚠️ **NãoComercial** — Você não pode usar o material para fins comerciais

Para mais detalhes: https://creativecommons.org/licenses/by-nc/4.0/deed.pt-br

---

**Copyright © Delean Mafra**  
Todos os direitos reservados
