# PROMPT — Correção da Restauração de Backup e Importação de Dados

## Contexto

Foi realizada anteriormente uma correção relacionada à importação/restauração de dados através de ficheiros Excel/CSV.

A explicação anterior foi:

> A causa foi corrigida. O ficheiro Excel continha o CSV inteiro dentro de uma única coluna, por isso as 46 linhas não eram reconhecidas como clientes.

Segundo a implementação anterior, o importador passou a:
- Ler `.xlsx`, `.xls` e `.csv`.
- Detectar CSV encapsulado numa única célula.
- Reconstruir automaticamente as colunas.
- Importar nome, telefone, grupo, carteiras, notas e IDs.
- Bloquear duplicados.
- Validar o JavaScript sem erros.

O procedimento indicado era:

1. Kâmbio → Definições.
2. Dados & Backup.
3. Clicar em `IMPORTAR CLIENTES XLSX / CSV`.
4. Selecionar `kambio_clients_2026-08-28.xlsx`.
5. Confirmar.

Porém, **a correção não resolveu o problema na prática**. A mensagem de erro continua e o backup/restauração não foi concluído com sucesso.

Além disso, o botão não deve funcionar apenas para clientes. Deve funcionar de forma geral para os dados que o sistema efetivamente suporta no mecanismo de backup/restauração.

---

# OBJETIVO

Investigar e corrigir definitivamente o problema de restauração/importação.

Não assumir que a correção anterior funcionou apenas porque o JavaScript foi validado sem erros.

Garantir que:
1. O ficheiro seja realmente processado.
2. O erro atual seja reproduzido e identificado.
3. A causa real seja encontrada.
4. A restauração funcione de ponta a ponta.
5. O botão não fique limitado a clientes.
6. Todos os tipos de dados suportados sejam tratados.
7. Dados existentes não sejam destruídos indevidamente.
8. Duplicados sejam tratados com segurança.
9. O utilizador saiba o que foi restaurado, ignorado ou rejeitado.
10. A operação seja testada antes de ser considerada concluída.

# REGRA MAIS IMPORTANTE

Não considerar concluído porque:
- não há erros de sintaxe;
- o código parece correto;
- a função foi criada;
- o ficheiro foi reconhecido;
- a importação começou;
- apareceu uma mensagem de sucesso.

A tarefa só termina quando o fluxo real funcionar e a persistência for confirmada.

# FASE 1 — AUDITORIA ANTES DE ALTERAR

**Não modificar código imediatamente.**

## Interface

Mapear:
- botão atual;
- texto;
- função chamada;
- modal;
- input de ficheiro;
- mensagens;
- progresso;
- sucesso/erro.

## Firebase / backend

Identificar:
- coleções Firestore;
- documentos;
- subcoleções;
- Storage, se existir;
- Authentication, se necessário;
- IDs;
- campos;
- relações.

## Entidades

Descobrir no projeto quais dados existem, por exemplo:
- clientes;
- grupos;
- carteiras;
- lotes;
- vendas;
- compras;
- transferências;
- transações;
- configurações;
- comentários;
- Hub;
- Táxi;
- investimentos;
- outras entidades.

**Não assumir esta lista como definitiva. Analisar o projeto real.**

# FASE 2 — REPRODUZIR O ERRO

Usar `kambio_clients_2026-08-28.xlsx`, se disponível.

Verificar:
1. se é realmente `.xlsx`;
2. se possui múltiplas colunas;
3. se contém CSV numa única célula;
4. número de linhas;
5. cabeçalhos;
6. quantidade esperada de registros;
7. estrutura;
8. mensagem de erro;
9. função onde ocorre;
10. ponto exato da falha.

**Não corrigir apenas com base na descrição anterior. Reproduzir o problema real.**

# FASE 3 — RASTREAR O FLUXO

Mapear:

```text
Selecionar ficheiro
↓
Detectar formato
↓
Ler ficheiro
↓
Interpretar dados
↓
Identificar tipo de backup/importação
↓
Reconstruir estrutura
↓
Validar
↓
Detectar duplicados
↓
Mapear entidades
↓
Gravar no Firebase
↓
Confirmar persistência
↓
Atualizar interface
↓
Mostrar resultado
```

Descobrir exatamente onde falha.

Não corrigir somente o parsing se a falha estiver na validação, mapeamento, gravação, permissões, IDs, Firestore, backup ou tratamento de erros.

# FASE 4 — SEPARAR IMPORTAÇÃO DE CLIENTES E RESTAURAÇÃO

São operações diferentes.

## Importação de clientes

```text
Excel/CSV
↓
Clientes
↓
Criar/atualizar clientes
```

## Restauração de backup

```text
Backup
↓
Clientes
Grupos
Carteiras
Lotes
Vendas
Transferências
Configurações
Comentários
...
↓
Restaurar estrutura
```

Não misturar os dois fluxos.

# FASE 5 — GENERALIZAR O BOTÃO

O texto atual `IMPORTAR CLIENTES XLSX / CSV` é demasiado específico se o objetivo é restauração de dados.

Se for uma função de backup geral, usar um nome como:

`IMPORTAR / RESTAURAR BACKUP`

ou outro nome coerente com o projeto.

Se forem necessárias duas operações, separar:

```text
Importar Clientes
```

e

```text
Restaurar Backup
```

Não usar uma função ambígua para operações diferentes.

# FASE 6 — SUPORTE A TODOS OS DADOS

Basear o mecanismo nas entidades reais existentes.

Exemplo conceitual:

```text
Backup
├── metadata
├── clientes
├── grupos
├── carteiras
├── lotes
├── vendas
├── compras
├── transferências
├── configurações
├── comentários
├── Hub
├── Táxi
├── investimentos
└── outras entidades existentes
```

A lista é apenas exemplo. Descobrir primeiro a estrutura real.

Não criar suporte artificial para entidades inexistentes.

# FASE 7 — FORMATO DO BACKUP

Determinar qual formato o sistema já utiliza e preservá-lo quando possível.

Se já houver backup estruturado, não substituí-lo sem necessidade.

Um formato estruturado poderia ser:

```json
{
  "version": 1,
  "createdAt": "...",
  "source": "Kambio",
  "collections": {
    "clients": [],
    "wallets": [],
    "sales": [],
    "transfers": []
  }
}
```

Este JSON é apenas exemplo. Não substituir o formato existente sem auditoria.

Se Excel/CSV não conseguir representar corretamente relações, IDs, lotes e transações, separar claramente:

```text
Exportação para análise
```

de:

```text
Backup completo/restaurável
```

# FASE 8 — XLSX / XLS / CSV

Continuar suportando:
- `.xlsx`;
- `.xls`;
- `.csv`.

Detectar corretamente:
- CSV normal;
- CSV inteiro armazenado numa única célula.

Não interpretar qualquer ficheiro como CSV automaticamente.

# FASE 9 — PRÉVIA ANTES DA GRAVAÇÃO

Mostrar uma prévia real:

```text
Ficheiro reconhecido

Formato: XLSX
Folhas: 4

Clientes:       46
Carteiras:       8
Lotes:           23
Vendas:          91
Transferências:  12

[Cancelar] [Continuar restauração]
```

Os números são apenas exemplo e devem vir dos dados reais.

# FASE 10 — DUPLICADOS

Preservar a proteção contra duplicados.

Definir comportamento conforme a entidade:

```text
Novo registro → Criar

Existente → Atualizar / Ignorar / Perguntar
```

Preferência para identificação:
1. ID original;
2. identificador único;
3. combinação confiável de campos;
4. nome somente como último recurso.

Não usar apenas nome para decidir duplicidade.

# FASE 11 — RELAÇÕES ENTRE ENTIDADES

Preservar referências como:

```text
clientId
walletId
lotId
```

Exemplo:

```text
Cliente A
↓
Carteira X
↓
Lote 001
↓
Venda 052
```

Não restaurar registros isoladamente e perder relações.

Verificar conflitos de IDs existentes.

# FASE 12 — RESTAURAÇÃO PARCIAL

Evitar:

```text
Clientes ✓
Carteiras ✓
Lotes ✗
Vendas ✗
```

sem informar o utilizador.

Usar batches/transações quando apropriado.

Se uma restauração não puder ser totalmente atómica:
- dividir em lotes;
- registrar progresso;
- registrar falhas;
- informar claramente;
- nunca afirmar sucesso se parte falhou.

# FASE 13 — ERROS CLAROS

Substituir mensagens genéricas.

Exemplo:

```text
Restauração interrompida.

Etapa: Gravação de carteiras
Processados: 8
Concluídos: 5
Com erro: 3

Causa:
[mensagem técnica segura]
```

Outros exemplos:

```text
Sem permissão para gravar na coleção X.
```

```text
O ficheiro não possui uma estrutura de backup reconhecida.
```

```text
O registro X possui um ID inválido.
```

Não expor dados sensíveis desnecessariamente.

# FASE 14 — RESUMO DA RESTAURAÇÃO

Quando possível, mostrar:

```text
RESTAURAÇÃO CONCLUÍDA

Clientes:        46
Carteiras:        8
Lotes:            23
Vendas:           91
Transferências:   12

Criados:          150
Atualizados:       20
Ignorados:          5
Erros:              0
```

Com falhas:

```text
RESTAURAÇÃO PARCIAL

Processados: 150
Sucesso:     143
Ignorados:     2
Erros:         5
```

Nunca mostrar sucesso quando existirem falhas relevantes.

# FASE 15 — PRESERVAR DADOS EXISTENTES

Antes de restaurar:
- verificar se já existem dados;
- informar o utilizador;
- não apagar automaticamente;
- não sobrescrever indiscriminadamente;
- respeitar a estratégia de duplicados.

Qualquer restauração destrutiva deve exigir confirmação explícita.

# FASE 16 — TESTES OBRIGATÓRIOS

Testar:
1. `.xlsx` verdadeiro com múltiplas colunas;
2. `.xlsx` contendo CSV numa única célula;
3. `.csv` normal;
4. `.xls`;
5. backup com múltiplas entidades;
6. duplicados;
7. IDs existentes;
8. IDs conflitantes;
9. dados relacionados;
10. ficheiro inválido;
11. ficheiro vazio;
12. colunas inesperadas;
13. erro de permissão Firebase;
14. falha durante gravação;
15. refresh após restauração;
16. nova sessão;
17. persistência no Firebase após reiniciar a aplicação.

# FASE 17 — CRITÉRIO DE SUCESSO

Só considerar concluído quando for demonstrado:

```text
Selecionar backup
↓
Reconhecer
↓
Prévia correta
↓
Confirmar
↓
Processar
↓
Firebase atualizado
↓
Relações preservadas
↓
Interface atualizada
↓
Refresh
↓
Dados continuam presentes
```

E o sistema deve funcionar para **todo o conjunto de dados suportado**, não apenas clientes.

# FASE 18 — PUBLICAÇÃO

A correção anterior ainda não havia sido commitada nem publicada.

Depois de a nova implementação funcionar:

1. verificar alterações;
2. testar localmente;
3. confirmar ausência de erros;
4. confirmar restauração real;
5. fazer commit;
6. publicar/deploy;
7. testar novamente no ambiente publicado.

Não afirmar que foi publicado sem confirmar.

# NÃO FAZER

- Não assumir que a correção anterior funcionou.
- Não limitar a solução a clientes.
- Não apagar dados automaticamente.
- Não mascarar erros.
- Não considerar ausência de erros de sintaxe como teste suficiente.
- Não criar outro importador paralelo sem necessidade.
- Não duplicar lógica.
- Não mudar o banco sem necessidade.
- Não alterar dados financeiros para facilitar importação.
- Não afirmar sucesso antes de confirmar persistência.

# FAZER

- Auditar.
- Reproduzir.
- Diagnosticar.
- Corrigir a causa.
- Testar.
- Validar persistência.
- Validar relações.
- Validar múltiplas entidades.
- Mostrar resultados claros.
- Preservar dados existentes.

# PROTOCOLO DE EXECUÇÃO

## ETAPA 1 — AUDITORIA

**Não alterar código.**

Entregar:
- estrutura encontrada;
- função atual do botão;
- fluxo atual;
- formato de backup;
- entidades suportadas;
- mensagem de erro reproduzida;
- ponto exato da falha;
- causa provável;
- causa confirmada, se possível.

Depois aguardar autorização.

## ETAPA 2 — CORREÇÃO

Implementar somente após aprovação do diagnóstico.

## ETAPA 3 — TESTES

Executar os testes definidos.

## ETAPA 4 — VALIDAÇÃO

Confirmar:
- dados;
- relações;
- Firebase;
- refresh;
- nova sessão;
- duplicados;
- múltiplas entidades.

## ETAPA 5 — PUBLICAÇÃO

Somente depois de tudo aprovado.

# RESULTADO FINAL ESPERADO

O sistema deve possuir uma solução coerente de dados e backup:

```text
Importação de dados
        ≠
Importação exclusiva de clientes

Backup completo
        ↓
Todos os dados suportados
        ↓
Relações preservadas
        ↓
Restauração confiável
```

O botão e o mecanismo devem refletir corretamente essa finalidade.

**Comece exclusivamente pela ETAPA 1 — AUDITORIA.**

Não altere código ainda.

Depois da auditoria, apresente o diagnóstico e aguarde autorização para implementar.
