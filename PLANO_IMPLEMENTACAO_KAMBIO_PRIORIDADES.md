# PROMPT — IMPLEMENTAÇÃO KÂMBIO BUSINESS / HUB

## Regra principal

Implementar primeiro **todas as funcionalidades prioritárias**, testar e validar. **Somente depois de todas as prioridades concluídas, fazer commit e deploy.** Após confirmar que o deploy das prioridades está funcionando, iniciar as funcionalidades restantes.

Não considerar uma correção concluída apenas porque o código não apresenta erros de sintaxe. Auditar o código existente, reproduzir os problemas, identificar a causa raiz, corrigir, testar e validar persistência.

---

# 1. FASE PRIORITÁRIA

## P1 — Pagamentos pendentes aparecendo em Dívidas

O problema já foi solicitado anteriormente, mas a correção não surtiu efeito.

Auditar a origem dos pagamentos, estados (`pending`, `paid`, `completed`, `cancelled` ou equivalentes), consultas, filtros, agregações, listeners e cálculos.

Reproduzir o problema e descobrir por que pagamentos pendentes estão sendo classificados como dívidas.

Resultado esperado:

```text
Pagamento pendente → continua pendente → não aparece em Dívidas
Pagamento efetivamente devido → aparece em Dívidas
```

Testar criação, alteração de estado, refresh, nova sessão e múltiplos clientes.

---

## P2 — Dinheiro em custódia

A área de dinheiro em custódia deve apresentar **todas as carteiras** disponíveis que participam da operação.

Quando houver devolução parcial ou total, o dinheiro deve sair efetivamente de uma origem válida.

### Cripto

```text
Carteira cripto → devolução → redução do saldo da carteira
```

### KZ

```text
Conta KZ → devolução → redução do saldo da conta
```

Não alterar somente o registro visual da custódia.

Exemplo:

```text
Custódia: 100.000 Kz
Devolução: 40.000 Kz
Custódia restante: 60.000 Kz
Origem reduzida: 40.000 Kz
```

Para devolução total, o saldo da custódia deve chegar a zero.

Validar saldo, origem, histórico, devolução parcial, total e múltiplas carteiras.

---

## P3 — Loading em todos os botões de gravação

Todos os botões que salvam ou executam operações de escrita devem ter loading e ficar temporariamente desabilitados.

Abranger, conforme existente:

- salvar;
- adicionar;
- editar;
- confirmar;
- importar/restaurar;
- transferir;
- registrar venda;
- registrar compra;
- registrar pagamento;
- devolver custódia;
- reembolsar;
- demais operações de escrita.

Fluxo:

```text
Clique → Loading → Desabilitar → Executar → Sucesso/Erro → Reativar
```

O objetivo é impedir múltiplos cliques e operações duplicadas.

---

## P4 — Modal de confirmação para operações importantes

Operações de risco devem solicitar confirmação antes de salvar/executar.

Exemplos:

- apagar;
- restaurar backup;
- transferir valores;
- reembolsar;
- devolver custódia;
- alterar dados financeiros;
- alterar divisão do lucro;
- outras operações críticas encontradas na auditoria.

Exemplo:

```text
Confirmar operação

Tem certeza que deseja realizar esta operação?

Valor: XXXXX
Origem: XXXXX
Destino: XXXXX

[Cancelar] [Confirmar]
```

Não usar confirmação desnecessariamente em ações simples.

---

## P5 — Valor disponível para saque de salário

Rever a lógica atual, pois o valor está fixo.

O sistema deve calcular dinamicamente quanto está disponível para saque com base no lucro e no histórico real de saques.

Exemplo:

```text
Lucro disponível para salário: 500.000 Kz
Já sacado:                      150.000 Kz
Disponível para sacar:          350.000 Kz
```

Após novo saque de 100.000 Kz:

```text
Disponível: 250.000 Kz
```

O utilizador pode sacar diariamente ou semanalmente, mas deve sempre saber:

- quanto já sacou;
- quanto ainda pode sacar;
- de onde esse valor veio;
- qual é o saldo disponível.

Não usar um valor fixo independente do histórico.

---

## P6 — Divisão do lucro com histórico

A divisão do lucro pode ser alterada a qualquer momento.

A alteração deve valer para o futuro e **não modificar os dados passados**.

Exemplo:

```text
Período antigo:
Salário 30% | Reinvestimento 60% | Reserva 10%

Novo período:
Salário 50% | Reinvestimento 40% | Reserva 10%
```

As operações antigas permanecem calculadas conforme a configuração vigente na época.

Registrar histórico/versionamento:

- negócio;
- percentuais;
- data de início;
- data da alteração;
- período de validade.

A configuração deve ser **global por padrão**, mas permitir personalização por negócio.

Exemplo:

```text
Global
↓
Padrão

Kâmbio Business
↓
Configuração própria

Importação
↓
Configuração própria

Táxi
↓
Configuração própria
```

O utilizador deve poder aumentar o salário e reduzir o reinvestimento quando precisar, utilizando apenas o lucro disponível, sem comprometer indevidamente o capital da Kâmbio Business.

As alterações devem aparecer no rastreio do lucro.

---

## P7 — Entradas e saídas / saldo salarial

Atualmente uma saída reduz apenas o lucro pessoal gerado pela KB e ignora outras entradas de salário, podendo gerar saldo negativo incorreto.

Auditar:

- entradas;
- saídas;
- salário;
- lucro pessoal;
- transferências;
- contas pessoais;
- saldos;
- histórico.

O saldo deve refletir o valor realmente disponível segundo a estrutura financeira existente.

Exemplo:

```text
Salário KB:      100.000 Kz
Outras entradas:  50.000 Kz
Saldo:           150.000 Kz

Saída:           120.000 Kz

Saldo correto:    30.000 Kz
```

Não resolver apenas bloqueando saldo negativo. Corrigir a origem do cálculo.

---

## P8 — Responsividade

Melhorar a responsividade de:

- Definições;
- Dashboard;
- Comprar;
- Ranking;
- Transferências;
- Financeiro.

Testar desktop, tablet e telemóvel.

Verificar tabelas, cards, formulários, filtros, modais, gráficos, menus, espaçamento e overflow.

Não fazer redesign geral nesta etapa. Preservar a identidade existente.

---

## P9 — Evolução do lucro

Na área:

`EVOLUÇÃO DO LUCRO — ÚLTIMOS 6 MESES`

ao clicar num mês do gráfico deve abrir um histórico detalhado de vendas desse mês.

Mostrar todos os dias:

```text
01/08 → faturação
02/08 → faturação
03/08 → faturação
...
31/08 → faturação
```

Respeitar o número real de dias de cada mês.

Adicionar opção para consultar todos os meses disponíveis, não apenas os últimos 6.

Possíveis filtros:

```text
Últimos 6 meses
Este ano
Ano anterior
Todos os meses
Período personalizado
```

Usar os dados reais existentes.

---

# 2. TESTES DAS PRIORIDADES

Antes do deploy, testar todas as prioridades em conjunto.

Verificar:

- cálculos financeiros;
- persistência no Firebase;
- refresh;
- nova sessão;
- múltiplos cliques;
- confirmações;
- saldos;
- histórico;
- relações entre dados;
- responsividade;
- ausência de regressões.

Para operações financeiras:

```text
Origem correta
↓
Movimento correto
↓
Destino correto
↓
Saldo correto
↓
Histórico correto
```

---

# 3. DEPLOY

**Somente depois de P1–P9 estarem implementadas e validadas:**

1. Revisar todas as alterações.
2. Executar testes.
3. Corrigir regressões.
4. Verificar integridade dos dados.
5. Fazer commit.
6. Fazer deploy.
7. Testar novamente o ambiente publicado.
8. Confirmar que as funcionalidades prioritárias continuam funcionando após o deploy.

**Não iniciar a Fase 2 antes deste ponto.**

---

# 4. FASE 2 — FUNCIONALIDADES RESTANTES

## F1 — Meta financeira

Adicionar nas Definições uma meta personalizável conforme os objetivos do utilizador.

Campos conforme necessário:

- nome/descrição;
- valor objetivo;
- prazo;
- data de início;
- estado;
- progresso.

Exemplo:

```text
Meta: Comprar equipamento
Valor: 2.000.000 Kz
Prazo: 31/12/2026
Progresso: 65%
```

Usar dados financeiros reais para o progresso.

---

## F2 — Contas de pagamento dos clientes em KZ

Adicionar ao cadastro do cliente uma área para contas de pagamento em KZ.

Considerar a estrutura existente e, quando aplicável:

- banco;
- titular;
- número da conta;
- IBAN;
- outros dados necessários.

Diferenciar conta bancária KZ de carteira cripto.

---

## F3 — Pesquisa por número de telemóvel

Corrigir a pesquisa para tratar números gravados com diferentes espaços/formatações.

Exemplo:

```text
923123456
923 123 456
923-123-456
923123 456
```

Normalizar para comparação sem alterar a forma como o número é apresentado.

Evitar duplicação causada apenas por formatação.

---

## F4 — Alertas de dados incompletos

Criar seção de alertas para clientes com:

- ID em falta;
- carteira em falta;
- IBAN/conta KZ em falta.

Exemplo:

```text
3 clientes sem ID
5 clientes sem carteira
2 clientes sem IBAN KZ
```

Permitir localizar os clientes afetados.

---

## F5 — Clientes que usam conta de terceiros

Dar destaque aos clientes que pagam através de conta de terceiros.

Exemplo:

```text
⚠ Conta de pagamento de terceiro
```

Manter os dados estruturados e não substituir os dados originais do cliente.

---

## F6 — Reembolso de venda de USDT na KB

Implementar reembolso na Kâmbio Business.

O reembolso deve devolver os valores às contas/carteiras de origem.

Fluxo:

```text
Venda
↓
Solicitar reembolso
↓
Confirmar
↓
Identificar origem
↓
Devolver valor
↓
Atualizar saldo
↓
Registrar histórico
```

Não apagar a venda original.

Registrar o reembolso separadamente e impedir reembolsos duplicados.

Testar reembolso total e parcial se permitido, origem KZ, origem cripto, lotes, saldo e histórico.

---

## F7 — Integração com carteiras digitais

Verificar se é possível integrar as carteiras digitais utilizadas ao sistema para facilitar o registro/reconciliação das transações.

Antes de implementar:

1. Identificar as carteiras utilizadas.
2. Verificar APIs oficiais.
3. Verificar autenticação.
4. Verificar leitura de saldo.
5. Verificar leitura de transações.
6. Verificar escrita, se disponível.
7. Verificar limites, custos e segurança.

Não criar integração fictícia.

Se uma carteira não possuir API adequada, documentar a limitação.

---

## F8 — Fornecedores

Criar/aperfeiçoar identificação completa dos fornecedores.

Incluir, conforme a estrutura necessária:

- identificação;
- contacto;
- banco;
- titular;
- IBAN de recebimento em KZ;
- outros dados necessários.

Diferenciar fornecedores de clientes.

---

## F9 — Múltiplas contas / Dashboard Admin

Quando a funcionalidade de múltiplas contas for criada, implementar um dashboard administrativo separado para monitorar o uso do sistema.

Conforme as permissões da arquitetura:

- coletar feedback;
- definir limites de uso;
- restringir funcionalidades;
- monitorar utilização;
- gerenciar permissões;
- acompanhar atividade;
- controlar contas;
- suspender/restringir funcionalidades quando necessário.

Separar claramente:

```text
Administrador da plataforma
```

de:

```text
Utilizador / empresa
```

Não misturar dados entre contas.

---

# 5. REGRAS TÉCNICAS

- Reutilizar funções e componentes existentes.
- Não criar estruturas duplicadas.
- Não migrar de Firebase/framework sem necessidade.
- Não apagar dados.
- Não alterar regras financeiras sem compreender o impacto.
- Evitar listeners duplicados.
- Evitar consultas N+1.
- Manter histórico financeiro.
- Preservar relações entre entidades.
- Validar permissões.
- Proteger operações contra múltiplos cliques.
- Usar transações/batches apropriados para operações financeiras.
- Não considerar validação de sintaxe como teste funcional.

---

# 6. REGRAS FINANCEIRAS

Sempre que uma operação movimentar dinheiro:

```text
Origem
↓
Movimento
↓
Destino
↓
Histórico
↓
Saldo
```

Tudo deve permanecer consistente.

Não criar dinheiro artificial.

Não remover dinheiro apenas alterando um campo visual.

Não recalcular o passado usando configurações atuais quando a operação histórica possuir outra configuração de divisão do lucro.

Evitar dupla contabilização de valores.

---

# 7. PROTOCOLO DE EXECUÇÃO

Para cada item:

```text
AUDITAR
↓
DIAGNOSTICAR
↓
APRESENTAR CAUSA E PLANO
↓
IMPLEMENTAR
↓
TESTAR
↓
VALIDAR
↓
PRÓXIMO ITEM
```

Durante a Fase 1, trabalhar pelas prioridades na ordem P1 → P9.

Depois de todas concluídas:

```text
TESTES FINAIS
↓
COMMIT
↓
DEPLOY
↓
VALIDAÇÃO DO DEPLOY
```

Somente então:

```text
FASE 2
↓
F1 → F9
```

---

# 8. INSTRUÇÃO FINAL

**Comece exclusivamente pela P1 — Pagamentos pendentes aparecendo indevidamente na área de Dívidas.**

Primeiro audite e reproduza o problema.

Não altere outras funcionalidades.

Apresente:

- arquivos envolvidos;
- funções envolvidas;
- fluxo atual;
- causa encontrada;
- impacto;
- solução proposta;
- riscos de regressão.

Depois aguarde autorização para implementar.

Após concluir P1, testar e validar, avance para P2.

**Não implemente todas as funcionalidades de uma vez.**

**Não faça deploy antes de todas as prioridades P1–P9 estarem concluídas e validadas.**

**Não desenvolva a Fase 2 antes do deploy bem-sucedido das prioridades.**
