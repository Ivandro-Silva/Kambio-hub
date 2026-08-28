# Kâmbio Hub — Plano Unificado de Correção de Bugs e Implementação de Melhorias

## Objetivo

Este documento define a ordem exata de trabalho para corrigir problemas existentes e, posteriormente, implementar as novas melhorias do Kâmbio Business + Grupo Kâmbio Hub.

## Regra de execução

**Uma etapa por vez.**

Fluxo obrigatório:

```text
Auditar → Identificar a causa → Apresentar diagnóstico → Propor correção
→ Implementar somente a etapa autorizada → Testar localmente
→ Relatar resultado → Aguardar aprovação → Próxima etapa
```

**Não implementar várias etapas simultaneamente.**

Antes de alterar código, analisar a estrutura existente, reutilizar a lógica já existente e preservar as regras financeiras atuais.

---

# FASE 1 — CORREÇÃO DOS BUGS CRÍTICOS

## 1. Carteiras duplicadas + eliminação que não persiste

### Problema

Existem carteiras duplicadas. Ao clicar no `X` para eliminar uma carteira, ela desaparece por alguns segundos e depois volta a aparecer.

### Objetivo

Descobrir a causa real e corrigir a persistência.

### Auditoria obrigatória

Investigar:

- coleção Firebase/Firestore responsável pelas carteiras;
- documentos e IDs existentes;
- possíveis documentos duplicados;
- campos usados para identificar uma carteira;
- função de criação;
- função de eliminação;
- listeners `onSnapshot`;
- consultas que carregam carteiras;
- sincronização entre KB e Hub;
- IndexedDB/cache/offline persistence, se existente;
- qualquer função que possa recriar/restaurar uma carteira;
- regras de segurança relevantes.

Verificar se o `delete` realmente executa, se retorna erro, se remove apenas o estado local ou se algum listener/consulta posterior restaura o documento.

**Não esconder duplicadas com CSS ou `filter()`. Corrigir a origem.**

### Resultado esperado

- Eliminar persiste.
- Atualizar a página não restaura.
- Reiniciar a aplicação não restaura.
- Listeners não recriam documentos eliminados.
- Duplicadas são identificadas com segurança.
- Nenhuma carteira legítima é apagada por engano.

### Testes

Testar eliminação, refresh, nova sessão, `firebase serve`, offline se aplicável, nomes iguais, IDs diferentes e carteiras com transações relacionadas.

**Não apagar duplicados automaticamente sem identificar a origem e confirmar quando houver risco de perda de dados.**

---

# 2. Área do Táxi — não é possível adicionar entradas

Investigar e corrigir:

- botão;
- evento;
- formulário/modal;
- validações;
- função de criação;
- coleção Firestore;
- campos obrigatórios;
- `addDoc`/`setDoc` ou equivalente;
- permissões;
- tratamento de erros;
- atualização da interface;
- listeners;
- erros JavaScript no console.

Primeiro descobrir a causa. Não alterar a lógica financeira do Táxi sem necessidade.

### Resultado

```text
Adicionar entrada → preencher → validar → salvar → confirmar → atualizar lista
```

Testar criação válida, campos vazios, valores inválidos, persistência, edição/exclusão existentes e erros do Firebase.

---

# 3. Transferências entre carteiras cripto — AUDITORIA CRÍTICA

### Problema

Os USDT desapareceram de uma carteira durante/após uma transferência entre carteiras cripto e isso impediu a continuidade do registro das vendas do dia.

Esta é uma **prioridade crítica**, pois envolve integridade de saldo.

### Objetivo

Descobrir exatamente:

- de onde os USDT foram retirados;
- para onde deveriam ter sido enviados;
- se foram realmente transferidos;
- se o saldo foi apenas alterado visualmente;
- se o lote foi alterado;
- se houve perda/duplicação de registro;
- se a carteira de destino recebeu;
- se o histórico registra a operação.

Auditar todo o fluxo:

```text
Carteira origem
↓
seleção de USDT/lotes
↓
quantidade
↓
criação da transferência
↓
baixa na origem
↓
crédito no destino
↓
lotes
↓
saldo
↓
histórico
```

Verificar coleções, lotes, transferências, cálculo de saldo, funções, `batch`/`transaction`, listeners, sincronização, offline persistence, concorrência, precisão decimal e quantidade disponível.

### Regra financeira

Transferência entre carteiras **não é venda nem compra** e não deve gerar lucro.

Exemplo:

```text
Carteira A: 100 USDT
Transferência: 40 USDT
Carteira A: 60 USDT
Carteira B: +40 USDT

Total do sistema antes/depois: 100 USDT
```

### Preservação dos lotes

Os lotes devem preservar a taxa de compra original.

Exemplo:

```text
Lote original: 100 USDT @ 1.130 Kz
Transferência: 40 USDT

Origem: 60 USDT @ 1.130 Kz
Destino: 40 USDT @ 1.130 Kz
```

Utilizar a estrutura equivalente já existente. Não criar uma segunda regra de lotes.

### Integridade

A operação deve ser atômica quando necessário:

```text
ou tudo acontece
ou nada acontece
```

Nunca permitir:

```text
Origem -40
Destino +0
```

ou perda do lote após uma transferência bem-sucedida.

### Testes

- transferência parcial;
- transferência total;
- duas carteiras;
- múltiplos lotes;
- quantidade inválida;
- quantidade superior ao disponível;
- falha no destino;
- falha na origem;
- refresh;
- offline, se aplicável;
- histórico;
- preservação da taxa;
- saldo total antes/depois.

**Não apagar ou reconstruir histórico para mascarar o problema. Primeiro diagnosticar. Se houver dados perdidos, apresentar diagnóstico antes de qualquer recuperação.**

---

# 4. Seleção de múltiplos lotes + botão MÁX nas transferências

Depois de corrigir a transferência, melhorar a seleção de lotes.

**Reutilizar a mesma lógica de checkbox já existente na área de Vendas.**

Localizar:

- seleção de lotes;
- checkboxes;
- cálculo de quantidade disponível;
- comportamento de seleção múltipla.

Não duplicar lógica desnecessariamente.

### Interface desejada

```text
☐ Lote #001 — 50 USDT @ 1.130 Kz
☐ Lote #002 — 30 USDT @ 1.135 Kz
☐ Lote #003 — 80 USDT @ 1.140 Kz

[ MÁX ]

Quantidade: [ 160 USDT ]
```

O botão `MÁX` deve respeitar:

- lotes selecionados;
- quantidade disponível;
- saldo;
- precisão decimal;
- regras existentes.

Se a lógica atual exigir seleção explícita de lotes, respeitá-la.

A experiência deve ser consistente com a área de Vendas.

---

# FASE 2 — SEPARAÇÃO KB / HUB

## 5. Vendas pendentes devem permanecer no Kâmbio Business

Vendas pendentes **não devem ser enviadas ao Hub**.

Estrutura conceitual:

```text
Kâmbio Business
└── Vendas
    ├── Pendentes
    ├── Concluídas
    └── Canceladas
```

Somente uma venda concluída/financeiramente efetiva deve alimentar a consolidação do Hub.

Uma pendente não deve entrar em:

- volume vendido do Hub;
- lucro;
- património;
- movimentos financeiros consolidados.

Auditar armazenamento, listeners, queries, sincronização e critérios de status.

---

# FASE 3 — DIVISÃO DE LUCRO

## 6. Configuração central, com personalização por negócio

A configuração fica no **Hub**, mas cada negócio pode ter uma divisão diferente.

### Padrão global

```text
Reinvestimento: 60%
Retirada:       30%
Reserva:        10%
Total:          100%
```

### Exemplos específicos

```text
Kâmbio Business: 60 / 30 / 10
Importação:     50 / 30 / 20
Táxi:            70 / 20 / 10
```

### Regras

- O Hub é a fonte central.
- Existe uma configuração padrão global.
- Cada negócio pode sobrescrever o padrão.
- Novos negócios herdam o padrão.
- Alterar o padrão não sobrescreve configurações personalizadas.
- A soma deve obrigatoriamente ser 100%.
- Não duplicar a configuração em cada aplicação.
- Reutilizar/consolidar a lógica `SPLIT` existente.

---

# FASE 4 — COMENTÁRIOS SINCRONIZADOS

## 7. Comentários Kâmbio Business ↔ Hub

KB e Hub devem utilizar a mesma fonte de comentários.

```text
Comentários centralizados
        ↓
 ┌──────┴──────┐
 KB            Hub
```

Um comentário criado no KB deve aparecer no Hub e vice-versa.

Preservar, quando aplicável:

- ID;
- texto;
- autor;
- data/hora;
- origem;
- módulo;
- negócio;
- estado.

Evitar duas cópias independentes e listeners duplicados.

---

# FASE 5 — MODELO PATRIMONIAL DO HUB

## 8. Separar negócios, investimentos, aplicações e contas pessoais

O Hub deve funcionar como centro de visão financeira/patrimonial.

```text
PATRIMÓNIO
├── Negócios
├── Investimentos
├── Aplicações financeiras
└── Contas pessoais
```

### Negócios

- Kâmbio Business;
- Táxi;
- Importação;
- futuros negócios.

### Investimentos

- negócio de óleo;
- outros investimentos.

### Aplicações financeiras

- depósito a prazo;
- outras aplicações.

### Contas pessoais

Aparecem apenas no Hub.

O Kâmbio Business permanece focado na operação do negócio.

---

# FASE 6 — NEGÓCIO DO ÓLEO

## 9. Investimentos em negócios com parceiros

Permitir registrar:

```text
Negócio: Óleo para revenda
Investidor: Ivandro
Parceira: [nome]
Minha participação: 50%
Meu capital: 100.000 Kz
Capital da parceira: 100.000 Kz
Capital total: 200.000 Kz
Responsável pela operação: Parceira
```

O capital total do negócio não deve ser confundido com o património pessoal.

Somente a participação pertencente ao usuário entra no património dele.

Futuramente permitir:

- capital investido;
- participação;
- capital dos parceiros;
- compras;
- vendas;
- lucro;
- distribuição;
- retiradas;
- valor atual da participação.

---

# FASE 7 — DEPÓSITOS A PRAZO

## 10. Aplicações financeiras

Permitir registrar:

- instituição;
- capital aplicado;
- data de início;
- vencimento;
- taxa;
- juros;
- valor esperado;
- estado;
- observações.

Exemplo:

```text
Capital: 500.000 Kz
Início: 01/08/2026
Vencimento: 15/12/2026
Taxa: X%
Valor esperado: XXX.XXX Kz
Estado: Em aplicação
```

Objetivo: evitar que aplicações esquecidas deixem de aparecer na visão patrimonial e permitir alertas de vencimento.

---

# FASE 8 — TÁXI COMO NEGÓCIO + CARRO COMO ATIVO

## 11. Investimento inicial

O custo do carro e os custos necessários para colocá-lo operacional devem ser tratados como investimento/ativo do negócio.

Exemplo:

```text
Compra:       5.000.000 Kz
Legalização:    300.000 Kz
Reparação:      500.000 Kz
Pintura:        200.000 Kz
Documentação:   100.000 Kz
Outros:         150.000 Kz
Total:        6.250.000 Kz
```

Armazenar separadamente:

- aquisição;
- preparação;
- custo total;
- valor atual estimado;
- receitas;
- despesas;
- lucro.

O carro entra no património como ativo, não como dinheiro disponível.

Evitar dupla contagem.

---

# FASE 9 — IMPORTAÇÃO

## 12. Capital investido e stock

Representar o fluxo:

```text
Dinheiro
↓
Capital de importação
↓
Mercadoria
↓
Stock
↓
Venda
↓
Receita + lucro
```

Evitar contar simultaneamente o mesmo capital como dinheiro e stock quando for o mesmo valor convertido em ativo.

---

# FASE 10 — CONTAS PESSOAIS

## 13. Separar contas pessoais das empresariais

No Hub:

```text
Pessoal
├── Contas
├── Dinheiro
└── Outros ativos
```

Quando dinheiro sai do negócio para uso pessoal:

```text
Negócio
↓
Retirada pessoal
↓
Conta pessoal
```

O dinheiro não deve simplesmente desaparecer do negócio.

---

# FASE 11 — PATRIMÓNIO CONSOLIDADO

## 14. Visão patrimonial

O Hub deve responder:

> Quanto vale o meu património e onde ele está?

```text
PATRIMÓNIO
├── Dinheiro disponível
│   ├── Contas pessoais
│   ├── Carteiras
│   └── Caixa
├── Negócios
│   ├── Kâmbio
│   ├── Táxi
│   └── Importação
├── Investimentos
│   └── Óleo
├── Aplicações
│   └── Depósitos a prazo
└── Ativos
    └── Carro
```

Somente ativos pertencentes ao usuário entram no património.

Não incluir:

- vendas pendentes;
- dinheiro de parceiros;
- operações canceladas;
- valores de terceiros.

---

# FASE 12 — PATRIMÓNIO LÍQUIDO

## 15. Evolução futura

Quando houver suporte a obrigações:

```text
Património Líquido = Ativos - Obrigações
```

Preparar a arquitetura, mas não implementar antes das fases anteriores.

---

# FASE 13 — IA E INTELIGÊNCIA FINANCEIRA

## 16. Financial Intelligence

Somente depois da estrutura financeira estar estável:

- insights;
- alertas;
- vencimentos;
- desempenho;
- concentração de capital;
- evolução patrimonial;
- comparação de negócios;
- anomalias;
- recomendações.

A IA deve utilizar exclusivamente dados reais do sistema.

---

# REGRAS FINANCEIRAS PERMANENTES

Preservar obrigatoriamente:

- saldos de carteira derivados dos lotes;
- lotes preservam a taxa de compra original;
- transferências não geram lucro;
- vendas pendentes não são operações financeiras consolidadas;
- dados de terceiros não entram no património pessoal;
- evitar dupla contagem de ativos;
- preservar histórico financeiro;
- não alterar regras financeiras sem justificativa.

---

# REGRAS TÉCNICAS PERMANENTES

- Não reescrever ficheiros inteiros sem necessidade.
- Não apagar código funcional.
- Não migrar para outro framework.
- Não substituir Firebase.
- Não criar dependências externas sem necessidade.
- Reutilizar componentes e funções existentes.
- Reutilizar a lógica de seleção de lotes da área de vendas.
- Evitar `render()` completo em eventos de digitação.
- Evitar listeners duplicados.
- Evitar consultas N+1.
- Preservar funcionamento offline quando já suportado.
- Testar localmente antes de avançar.
- Fazer alterações pequenas e rastreáveis.
- Não fazer redesign visual geral durante estas correções, salvo necessidade funcional.

---

# ORDEM EXATA DE EXECUÇÃO

## FASE 1 — Bugs críticos

### 1. Carteiras
- [ ] Auditar duplicações.
- [ ] Auditar eliminação.
- [ ] Identificar causa do reaparecimento.
- [ ] Corrigir persistência.
- [ ] Testar.

### 2. Táxi
- [ ] Auditar criação de entradas.
- [ ] Identificar causa.
- [ ] Corrigir.
- [ ] Testar.

### 3. Transferências cripto
- [ ] Auditar desaparecimento dos USDT.
- [ ] Verificar origem/destino.
- [ ] Verificar lotes.
- [ ] Verificar saldos.
- [ ] Verificar histórico.
- [ ] Corrigir integridade/atomicidade.
- [ ] Testar transferência parcial.
- [ ] Testar transferência total.
- [ ] Testar múltiplos lotes.

### 4. Seleção de lotes
- [ ] Reutilizar lógica dos checkboxes das vendas.
- [ ] Implementar seleção de múltiplos lotes.
- [ ] Implementar botão MÁX.
- [ ] Validar quantidade máxima.
- [ ] Testar.

## FASE 2 — Separação KB/Hub

### 5. Vendas pendentes
- [ ] Auditar sincronização.
- [ ] Impedir envio de pendentes ao Hub.
- [ ] Garantir consolidação das concluídas.
- [ ] Testar.

### 6. Comentários
- [ ] Criar fonte central.
- [ ] Sincronizar KB ↔ Hub.
- [ ] Preservar comentários.
- [ ] Testar.

## FASE 3 — Configuração financeira

### 7. Divisão do lucro
- [ ] Criar padrão global.
- [ ] Criar configuração específica por negócio.
- [ ] Validar total = 100%.
- [ ] Consolidar `SPLIT`.
- [ ] Testar Kâmbio.
- [ ] Testar Importação.
- [ ] Testar Táxi.

## FASE 4 — Modelo patrimonial

### 8. Estrutura
- [ ] Negócios.
- [ ] Investimentos.
- [ ] Aplicações.
- [ ] Ativos.
- [ ] Contas pessoais.

### 9. Táxi
- [ ] Negócio.
- [ ] Carro.
- [ ] Custos de preparação.
- [ ] Valor atual.
- [ ] Receitas/despesas.

### 10. Importação
- [ ] Capital.
- [ ] Stock.
- [ ] Vendas.
- [ ] Lucro.
- [ ] Evitar dupla contagem.

### 11. Óleo
- [ ] Investimento.
- [ ] Participação.
- [ ] Parceira.
- [ ] Capital.
- [ ] Lucro.
- [ ] Valor da participação.

### 12. Depósito a prazo
- [ ] Capital.
- [ ] Vencimento.
- [ ] Taxa.
- [ ] Valor esperado.
- [ ] Alertas.

### 13. Pessoal
- [ ] Contas pessoais.
- [ ] Retiradas.
- [ ] Transferências negócio → pessoal.

## FASE 5 — Consolidação

### 14. Património
- [ ] Património total.
- [ ] Dinheiro disponível.
- [ ] Negócios.
- [ ] Investimentos.
- [ ] Aplicações.
- [ ] Ativos.

### 15. Património líquido
- [ ] Ativos.
- [ ] Obrigações.
- [ ] Cálculo líquido.

## FASE 6 — IA

### 16. Financial Intelligence
- [ ] Insights.
- [ ] Alertas.
- [ ] Análises.
- [ ] Evolução patrimonial.
- [ ] Recomendações baseadas em dados reais.

---

# PROTOCOLO OBRIGATÓRIO PARA A IA

Para cada item:

```text
1. AUDITAR
2. EXPLICAR O QUE FOI ENCONTRADO
3. IDENTIFICAR A CAUSA
4. PROPOR A CORREÇÃO
5. AGUARDAR APROVAÇÃO
6. IMPLEMENTAR
7. TESTAR
8. RELATAR RESULTADO
9. AGUARDAR APROVAÇÃO
10. AVANÇAR PARA O PRÓXIMO ITEM
```

**Nunca pule a auditoria.**

**Nunca implemente todos os itens de uma vez.**

**Nunca invente estruturas de dados sem primeiro analisar as existentes.**

**Nunca corrija um bug financeiro escondendo o sintoma. Corrija a causa.**

O objetivo final é transformar o Kâmbio Business + Grupo Kâmbio Hub em uma plataforma financeira centralizada, mantendo a operação diária do Kâmbio Business separada da visão consolidada patrimonial do Hub.
