# Plano de Melhorias — Kâmbio Business + Grupo Kâmbio Hub

> Criado em 2026-08-22 · Repo: Kambio-hub/ (git, GitHub: Ivandro-Silva/Kambio-hub)
> Regra de trabalho: uma fase (e dentro dela, um item) de cada vez — teste local
> após cada item, go-ahead do Ivandro antes de avançar.

## Estado actual (auditoria 2026-08-22)
- kambio/index.html — 1985 linhas · hub/index.html — 1754 linhas (monolitos)
- ✓ Nenhuma chamada fantasma buildModal() nas duas apps
- ✓ Regra 60/30/10 correcta nas 5 ocorrências (kambio 373-375, 1458-1460; hub 350, 1338)
- Hosting multisite funcional: kambio-business.web.app + grupo-kambio-hub.web.app
- Pasta externa «Kâmbio Business\» = legado, ignorar (não apagar sem aprovação)

## Fase 1 — Robustez preventiva
1. Constante única SPLIT = {reinvest:0.6, salary:0.3, savings:0.1} usada em
   todo o código (hoje duplicada em 5 pontos)
2. Estender o padrão withFocusPreserved (kambio) ao hub; inputs só actualizam
   containers específicos (nunca render() completo em eventos de digitação)
3. Guarda anti-buildModal fantasma + convenção documentada
4. Validar integridade de lotes nas transferências (sempre via registos reais)

## Fase 2 — Refactor modular (sem mudanças visíveis)
- kambio/ → js/firebase.js, js/state.js, js/utils.js, js/pages/*.js, css/
- hub/ → núcleo partilhado + subpastas por módulo: financas/, custodia/,
  taxi/, importacao/
- Módulos ES nativos; Firebase Hosting não muda (pastas públicas mantêm-se)
- Testar persistência offline (IndexedDB) com firebase serve antes de avançar

## Fase 3 — Melhorias Kâmbio Business (uma de cada vez, com teste)
1. Gráficos de evolução — lucro/volume por mês no Dashboard e Financeiro
   (Canvas próprio, sem bibliotecas externas)
2. Exportação/backup — vendas/lotes/clientes → CSV; backup JSON completo das
   coleções (o botão «APAGAR VENDAS» passa a exigir/sugerir export prévia)
3. Sugestão de câmbio + venda rápida — câmbio mínimo sugerido (preço médio
   ponderado de compra + margem-alvo); venda por valor em Kz com selecção
   automática de lotes (FIFO)
4. PWA — manifest + service worker, instalável no telemóvel
5. Alertas de stock mínimo — por ativo/carteira, mínimo configurável
6. Histórico de taxas/spread — evolução das taxas de compra/venda praticadas

## Fase 4 — Novos módulos no Hub
- Módulos restantes na arquitectura modular da Fase 2
- Detalhes a definir com Ivandro ao chegar a esta fase

## Fase 5 — Auto-deploy GitHub Actions
- Workflow deploy.yml: push em main → deploy dos dois sites
- Conta de serviço Firebase com papel «Firebase Hosting Publisher» +
  segredo FIREBASE_SERVICE_ACCOUNT no GitHub
- Nota: o Actions corre nos runners do GitHub (Node 20 via setup-node) —
  a versão do Node no iMac é irrelevante para o workflow

## Fluxo por item
git pull → implementar → teste local (firebase serve) → git add . &&
git commit && git push → deploy manual (firebase deploy --only hosting)
apenas quando o Ivandro pedir

## Princípios permanentes
- 60/30/10 exclusivamente · lotes preservam taxa de compra original
- Saldos de carteira derivados apenas do stock de lotes
- Cada página constrói o seu próprio HTML de modal (nunca buildModal global)
- Ler valores do DOM no topo de cada função de página antes de reconstruir
