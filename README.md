# QA SauceDemo — Exploratório de regressão

> **Preenchido pela IA em:** 01/10/2026 08:09 (BRT, UTC-3)
> **Executado/validado pelo QA em:** ____/____/______ ____:____
> **Atualizado pela IA em:** 01/10/2026 08:31 (BRT, UTC-3) — inclusão de `00-prompt-ia/` e `06-cypress/`
> **Revisão sênior dos scripts:** 01/10/2026 08:59 — ver `06-cypress/REVISAO-SENIOR.md`
> **Status:** Rascunho — tudo marcado como HIPÓTESE A VERIFICAR depende de execução manual.

## Estrutura
- `00-PLANO-COMPLETO.md`
- `README.md`
- `00-prompt-ia/01-prompt-original.md`
- `00-prompt-ia/02-prompt-complemento-cypress.md`
- `00-prompt-ia/03-prompt-preencher-session-sheet.md`
- `00-prompt-ia/04-prompt-execucao-cypress.md`
- `01-mapa-de-riscos/mapa-de-riscos.md`
- `02-charters/charters.md`
- `03-session-sheets/C1-jornadas-e2e-alternativas.md`
- `03-session-sheets/C2-estado-e-navegacao.md`
- `03-session-sheets/C3-persistencia-carrinho-sessao.md`
- `03-session-sheets/C4-entradas-checkout.md`
- `03-session-sheets/C5-comparacao-perfis.md`
- `03-session-sheets/C6-sessao-seguranca-basica.md`
- `03-session-sheets/C7-visual-usabilidade.md`
- `03-session-sheets/C8-catalogo-detalhe-ordenacao.md`
- `03-session-sheets/_MODELO-session-sheet.md`
- `04-bug-reports/C1-BR-01.md`
- `04-bug-reports/C2-BR-01.md`
- `04-bug-reports/C2-BR-02.md`
- `04-bug-reports/C3-BR-01.md`
- `04-bug-reports/C3-BR-02.md`
- `04-bug-reports/C3-BR-03.md`
- `04-bug-reports/C4-BR-01.md`
- `04-bug-reports/C4-BR-02.md`
- `04-bug-reports/C5-BR-01.md`
- `04-bug-reports/C5-BR-02.md`
- `04-bug-reports/C5-BR-03.md`
- `04-bug-reports/C5-BR-04.md`
- `04-bug-reports/C5-BR-05.md`
- `04-bug-reports/C5-BR-06.md`
- `04-bug-reports/C5-BR-07.md`
- `04-bug-reports/C5-BR-08.md`
- `04-bug-reports/C5-BR-09.md`
- `04-bug-reports/C6-BR-01.md`
- `04-bug-reports/C6-BR-02.md`
- `04-bug-reports/C6-BR-03.md`
- `04-bug-reports/C6-BR-04.md`
- `04-bug-reports/C7-BR-01.md`
- `04-bug-reports/C8-BR-01.md`
- `04-bug-reports/C8-BR-02.md`
- `04-bug-reports/_MODELO-bug-report.md`
- `04-bug-reports/evidencias/C1-BR-01-C1.6-carrinho-vazio-checkout.png`
- `05-criterio-automacao/criterio-automacao.md`
- `06-cypress/.gitignore`
- `06-cypress/README.md`
- `06-cypress/REVISAO-SENIOR.md`
- `06-cypress/cypress.config.js`
- `06-cypress/eslint.config.mjs`
- `06-cypress/package.json`
- `06-cypress/cypress/e2e/exploratorio/C1-jornadas-e2e-alternativas.cy.js`
- `06-cypress/cypress/e2e/exploratorio/C2-estado-e-navegacao.cy.js`
- `06-cypress/cypress/e2e/exploratorio/C3-persistencia-carrinho-sessao.cy.js`
- `06-cypress/cypress/e2e/exploratorio/C4-entradas-checkout.cy.js`
- `06-cypress/cypress/e2e/exploratorio/C5-comparacao-perfis.cy.js`
- `06-cypress/cypress/e2e/exploratorio/C6-sessao-seguranca-basica.cy.js`
- `06-cypress/cypress/e2e/exploratorio/C7-visual-usabilidade.cy.js`
- `06-cypress/cypress/e2e/exploratorio/C8-catalogo-detalhe-ordenacao.cy.js`
- `06-cypress/cypress/fixtures/entradas-checkout.json`
- `06-cypress/cypress/support/commands.js`
- `06-cypress/cypress/support/e2e.js`
- `06-cypress/cypress/support/selectors.js`

## Como usar
1. Leia `00-PLANO-COMPLETO.md` (seções 1 a 5).
2. Para cada sessão, abra a ficha em `03-session-sheets/` e preencha data, início, fim, TBS e a coluna de resultado de cada ideia.
3. Quando executar um bug pré-preenchido em `04-bug-reports/`, troque o Status para *Confirmado* ou *Não reproduzido*, preencha *Resultado obtido* e *Evidência*.
4. Bugs novos: copie `_MODELO-bug-report.md` com o nome `C<charter>-BR-<nº>.md` (numeração por charter: o próximo do C2 é `C2-BR-02.md`), assim a pasta fica ordenada de C1 a C8. Evidências em `04-bug-reports/evidencias/` com o mesmo prefixo.
5. Aplique `05-criterio-automacao/` a cada bug confirmado.
6. Scripts: abra `06-cypress/` no VS Code, `npm install` e `npm run cy:open` (detalhes em `06-cypress/README.md`).
7. Depois de cada execução, use `00-prompt-ia/03-prompt-preencher-session-sheet.md` para a IA preencher a ficha com os achados.

## Checklist do critério de aceite
- [x] 8 charters, cada um com risco explícito (coluna "Risco que justifica").
- [x] 44 de 60 ideias (73%) exploram estado, navegação ou dados.
- [x] Itens obrigatórios cobertos: E2E alternativas (C1), estado/navegação (C2, C3), entradas (C4), perfis (C5), sessão/segurança (C6), visual/usabilidade (C7).
- [x] Prioridade de 1 hora no fim do plano.

## Se eu tivesse só 1 hora

1. **C6 — Sessão e segurança (25 min):** fecha o achado em aberto do cookie do locked_out_user; é o único risco com impacto de segurança e já tem indício real.
2. **C2 — Estado e navegação no checkout (25 min):** URL direta e voltar/F5 nas etapas podem gerar pedido vazio ou duplicado, e nada disso está na suíte.
3. **10 min de registro:** transformar o que for confirmado em bug report e marcar o que vira automação — achado sem registro não reduz risco.
