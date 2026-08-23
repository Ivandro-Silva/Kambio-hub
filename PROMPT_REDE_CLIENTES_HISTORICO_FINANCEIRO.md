# Prompt — Rede de Indicações + Histórico Financeiro do Kâmbio Hub

Você é um **Senior Full-Stack Engineer especializado em aplicações financeiras, Firebase/Firestore, JavaScript modular e visualização de dados**.

Você está trabalhando no projeto existente **Kâmbio Business + Grupo Kâmbio Hub**.

## 1. REGRA PRINCIPAL — ANALISAR ANTES DE ALTERAR

Antes de escrever ou modificar qualquer código:

1. Analise completamente a estrutura atual do projeto.
2. Identifique:
   - coleções Firebase/Firestore existentes;
   - estrutura dos documentos;
   - entidades de clientes;
   - entidades de carteiras;
   - transações;
   - vendas;
   - compras;
   - lotes;
   - relações entre clientes e carteiras;
   - funções de renderização;
   - funções de pesquisa/filtro;
   - componentes/modais existentes.
3. Identifique onde a funcionalidade solicitada deve ser integrada.
4. Reutilize funções, componentes, estilos e estruturas existentes sempre que possível.
5. **Não crie uma segunda estrutura de dados para representar informações que já existem.**
6. Não altere funcionalidades existentes que não estejam diretamente relacionadas com esta feature.
7. Não faça um redesign geral da aplicação nesta tarefa.
8. Não substitua o Firebase/Firestore existente por outra tecnologia.

O código atual possui páginas grandes/monolíticas e existe uma evolução planejada para modularização. Portanto, qualquer código novo deve ser criado de forma organizada e compatível com a futura separação em módulos.

---

# 2. FEATURE A — RELAÇÃO "QUEM ADICIONOU QUEM"

Atualmente existe um campo utilizado para identificar **quem adicionou determinado cliente ao grupo**.

Esse campo deve deixar de ser um campo de texto livre.

## Comportamento desejado

Ao adicionar ou editar um cliente, o campo:

**"Adicionado por"**

deve permitir selecionar um cliente que já pertence ao grupo.

Exemplo:

```text
Adicionado por
[ Pesquisar cliente... ▼ ]

João Manuel
Maria Silva
Carlos António
Pedro José
```

A pesquisa deve funcionar pelo nome e, se existirem esses campos, também por telefone, username, ID ou outro identificador disponível.

### Regras

- O administrador não deve precisar digitar manualmente o nome.
- O valor armazenado deve utilizar um identificador estável do cliente, preferencialmente o `clientId`/document ID existente.
- O nome deve ser obtido do cliente relacionado.
- Se o cliente selecionado for eliminado ou estiver indisponível, o sistema não deve quebrar.
- Deve existir uma opção para indicar que o cliente foi adicionado diretamente pelo administrador/sistema, caso isso seja compatível com a estrutura existente.
- Um cliente não pode ser definido como seu próprio indicador/origem.
- Evitar duplicação desnecessária de dados.

Antes de criar novos campos, verifique se a estrutura atual já possui alguma informação equivalente.

---

# 3. FEATURE B — ÁRVORE GENEALÓGICA / REDE DE CLIENTES

A partir da relação:

```text
Cliente A
    ↓ adicionou
Cliente B
    ↓ adicionou
Cliente C
    ↓ adicionou
Cliente D
```

criar uma visualização de **rede/árvore de indicações**.

## Objetivo

Permitir ao administrador entender visualmente:

- quem adicionou quem;
- quantas pessoas cada cliente adicionou;
- profundidade da rede;
- ramificações;
- clientes sem indicação;
- clientes que possuem muitos descendentes.

Exemplo:

```text
                    João
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
        Maria      Carlos      Pedro
          │                     │
       ┌──┴──┐                  ↓
       ↓     ↓                André
      Ana   Paulo
```

## Interface

Criar uma seção:

**Rede de Clientes**

Com:

- pesquisa de cliente;
- seleção de cliente raiz;
- visualização da árvore;
- expandir/recolher níveis;
- contador de filhos diretos;
- contador de descendentes;
- identificação visual de clientes ativos/inativos, caso esse estado já exista;
- possibilidade de clicar em um cliente para visualizar seus dados/resumo.

### Importante

Não usar uma árvore meramente decorativa.

A estrutura visual deve ser construída dinamicamente a partir dos dados reais do Firebase.

Não criar uma biblioteca externa sem antes verificar se o projeto já possui alguma biblioteca de visualização disponível.

Se for possível implementar com HTML/CSS/JavaScript nativo de forma consistente com o projeto, prefira essa abordagem.

---

# 4. FEATURE C — HISTÓRICO FINANCEIRO COMPLETO

Criar uma área de:

**Histórico Financeiro**

que permita consultar as operações financeiras relacionadas às carteiras.

Antes de implementar, identifique quais coleções/registos existentes representam:

- vendas;
- compras;
- transferências;
- depósitos;
- levantamentos;
- movimentações entre carteiras;
- outras operações financeiras existentes.

Não inventar tipos de transação que não existam no sistema.

## A tabela deve apresentar, quando os dados existirem:

| Campo | Descrição |
|---|---|
| Data | Data/hora da operação |
| Tipo | Venda, compra, transferência, etc. |
| Carteira | Carteira envolvida |
| Cliente | Cliente relacionado |
| Ativo | USDT, USD, EUR, etc. |
| Quantidade | Quantidade movimentada |
| Taxa | Taxa utilizada |
| Valor | Valor financeiro |
| Resultado | Lucro/prejuízo quando aplicável |
| Estado | Estado da operação |

Adaptar os campos à estrutura real existente.

---

# 5. FILTRO ENTRE CARTEIRAS

O histórico deve permitir comparar e pesquisar operações por carteira.

Criar filtros como:

```text
Carteira
[ Todas ▼ ]

Tipo
[ Todos ▼ ]

Ativo
[ Todos ▼ ]

Cliente
[ Pesquisar... ]

Período
[ Data inicial ] → [ Data final ]

Pesquisar
[ 🔍 ]

Limpar filtros
```

## Comportamento

O administrador deve conseguir:

- visualizar todas as carteiras;
- selecionar uma carteira específica;
- comparar duas ou mais carteiras, se a estrutura existente permitir;
- filtrar por período;
- filtrar por tipo de operação;
- filtrar por ativo;
- pesquisar cliente;
- combinar vários filtros.

Os filtros devem atualizar apenas os componentes necessários.

**Não executar `render()` completo da aplicação a cada alteração de input.**

Respeite o padrão existente de atualização localizada dos elementos.

---

# 6. ANÁLISE DE DESEMPENHO DAS CARTEIRAS

Além da tabela, criar uma área de análise.

O administrador deve conseguir responder rapidamente:

### Qual carteira vendeu mais?

Apresentar ranking baseado nos dados reais.

Exemplo:

```text
CARTEIRAS — VOLUME DE VENDAS

🥇 Carteira Principal       1.240 USDT
🥈 Carteira João              860 USDT
🥉 Carteira Maria             530 USDT
```

O critério exato deve ser definido a partir da estrutura financeira existente.

Não confundir:

- quantidade de operações;
- quantidade de clientes;
- volume vendido;
- valor financeiro;
- lucro.

Esses indicadores devem permanecer separados.

---

# 7. CARTEIRA COM MAIS CLIENTES

Criar também ranking por quantidade de clientes.

Exemplo:

```text
CLIENTES POR CARTEIRA

Carteira Principal      128 clientes
Carteira João             74 clientes
Carteira Maria            42 clientes
```

Se um cliente puder estar relacionado com mais de uma carteira, determine a regra correta com base na estrutura existente antes de contar.

**Não contar duplicadamente o mesmo cliente sem justificativa.**

---

# 8. DASHBOARD ANALÍTICO

No topo da seção de histórico, criar indicadores resumidos.

Exemplo:

```text
┌────────────────┐ ┌────────────────┐ ┌────────────────┐
│ VOLUME VENDIDO │ │ CLIENTES       │ │ OPERAÇÕES      │
│                │ │                │ │                │
│ 4.820 USDT     │ │ 247            │ │ 1.284          │
└────────────────┘ └────────────────┘ └────────────────┘
```

E uma visualização comparativa:

```text
DESEMPENHO DAS CARTEIRAS

Carteira A   ████████████████████  42%
Carteira B   █████████████         27%
Carteira C   █████████             18%
Carteira D   ██████                13%
```

Utilizar dados reais.

Se já existir sistema de gráficos no projeto, reutilizá-lo.

Se não existir, implementar de acordo com a arquitetura atual e sem introduzir dependências desnecessárias.

---

# 9. RELAÇÃO ENTRE REDE E CARTEIRAS

Sempre que os dados permitirem, conectar as duas funcionalidades.

Ao abrir um cliente na árvore:

```text
João Manuel

Clientes adicionados: 12
Descendentes: 31

Carteira principal:
Kâmbio João

Volume vendido:
480 USDT

Total de operações:
37
```

Isso permitirá entender não apenas **quem indicou quem**, mas também o impacto financeiro dessa rede.

Essa integração deve ser feita somente se os dados existentes permitirem estabelecer a relação de forma confiável.

---

# 10. FIREBASE / FIRESTORE

Antes de criar consultas novas:

1. Identifique a estrutura atual das coleções.
2. Verifique quais campos já existem.
3. Reutilize índices/consultas existentes quando possível.
4. Evite buscar todos os documentos desnecessariamente.
5. Evite listeners duplicados.
6. Evite consultas N+1.

Se a árvore exigir muitas relações, considere uma estratégia eficiente de carregamento.

Exemplo conceitual:

```text
Carregar clientes
       ↓
Criar mapa clientId → cliente
       ↓
Criar relação parentId → filhos
       ↓
Construir árvore
```

Em vez de executar uma consulta Firebase individual para cada cliente.

---

# 11. SEGURANÇA E VALIDAÇÃO

Validar:

- cliente inexistente;
- cliente removido;
- referência inválida;
- cliente como próprio parent;
- ciclos na árvore.

Exemplo de ciclo proibido:

```text
A → B → C → A
```

O sistema deve detectar e impedir relações circulares.

---

# 12. UX

A nova funcionalidade deve ser intuitiva.

Não sobrecarregar a interface.

Preferir:

- pesquisa;
- filtros;
- indicadores;
- tabelas;
- árvore expansível;
- rankings;
- gráficos;
- estados vazios;
- loading states.

Não fazer redesign geral do sistema nesta tarefa.

A identidade visual deve ser compatível com o futuro conceito **Kâmbio Financial Intelligence**, mas a prioridade desta implementação é a funcionalidade.

---

# 13. COMPATIBILIDADE COM A ESTRUTURA ATUAL

O projeto está sendo desenvolvido de forma incremental.

Portanto:

- não reescrever os ficheiros inteiros;
- não apagar código funcional;
- não migrar todo o sistema para framework;
- não alterar Firebase Hosting;
- não alterar a arquitetura atual sem necessidade;
- não criar dependências externas desnecessárias;
- não modificar funcionalidades fora do escopo.

Se encontrar código que precise ser refatorado para implementar corretamente esta feature, faça apenas a refatoração mínima necessária.

---

# 14. PROCESSO OBRIGATÓRIO DE IMPLEMENTAÇÃO

Execute em etapas.

### ETAPA 1 — Auditoria

Antes de programar, apresente:

```text
ESTRUTURA ENCONTRADA

Clientes:
→ coleção:
→ campos relevantes:

Carteiras:
→ coleção:
→ campos relevantes:

Transações:
→ coleção:
→ campos relevantes:

Relação cliente/carteira:
→ estrutura:

Histórico existente:
→ estrutura:

Componentes reutilizáveis:
→ ...

Páginas afetadas:
→ ...
```

Não altere código nesta etapa.

### ETAPA 2 — Modelo da relação

Defina como o campo:

`adicionadoPor`

será armazenado utilizando a estrutura existente.

Explique quais documentos/campos serão utilizados.

### ETAPA 3 — Implementação da relação

Implementar:

- seleção de cliente;
- pesquisa;
- validação;
- persistência;
- edição.

Testar.

### ETAPA 4 — Árvore genealógica

Implementar a árvore usando os dados reais.

Testar:

- 0 níveis;
- 1 nível;
- vários níveis;
- múltiplas ramificações;
- clientes sem parent;
- relações inválidas;
- ciclos.

### ETAPA 5 — Histórico financeiro

Implementar a consulta e tabela.

Testar os dados.

### ETAPA 6 — Filtros

Implementar os filtros individualmente e combinados.

### ETAPA 7 — Ranking

Implementar:

- maior volume vendido;
- maior número de clientes;
- maior número de operações;
- outros indicadores que os dados existentes permitirem.

### ETAPA 8 — Integração

Conectar cliente → rede → carteira → histórico.

### ETAPA 9 — Teste final

Testar:

- criação;
- edição;
- pesquisa;
- filtros;
- árvore;
- rankings;
- dados vazios;
- erros;
- permissões;
- persistência Firebase;
- comportamento offline, se aplicável.

---

# 15. REGRA DE TRABALHO

**NÃO implemente todas as etapas de uma vez.**

Trabalhe:

```text
Analisar
↓
Propor solução
↓
Implementar uma etapa
↓
Testar
↓
Explicar o resultado
↓
Aguardar aprovação
↓
Próxima etapa
```

Não avance automaticamente para a próxima etapa.

---

# 16. RESULTADO ESPERADO

Ao finalizar, o sistema deverá permitir ao administrador:

1. Escolher um cliente existente no campo **"Adicionado por"**.
2. Visualizar a rede de clientes em uma **árvore genealógica dinâmica**.
3. Saber quem adicionou quem.
4. Ver quantos clientes cada pessoa adicionou.
5. Consultar o **histórico financeiro completo**.
6. Filtrar o histórico por carteira.
7. Filtrar por cliente, período, ativo e tipo de operação quando suportado pelos dados existentes.
8. Descobrir qual carteira vendeu mais.
9. Descobrir qual carteira possui mais clientes.
10. Comparar o desempenho das carteiras.
11. Navegar do cliente para sua carteira e histórico quando houver relação confiável.
12. Fazer tudo isso utilizando os dados reais do Firebase/Firestore existente.

## REGRA FINAL

**Primeiro compreenda o sistema existente. Depois implemente.**

Não presuma nomes de coleções, campos, IDs ou estruturas.

Não invente dados.

Não altere regras financeiras existentes.

Não faça redesign geral.

Não reescreva arquivos inteiros sem necessidade.

**Preserve o funcionamento atual e adicione a feature de forma incremental, segura e compatível com a arquitetura existente.**
