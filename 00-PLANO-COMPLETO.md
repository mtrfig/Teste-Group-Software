# Plano de testes exploratórios de regressão — SauceDemo

> **Preenchido pela IA em:** 01/10/2026 08:09 (BRT, UTC-3)
> **Executado/validado pelo QA em:** ____/____/______ ____:____
> **Status:** Rascunho — tudo marcado como HIPÓTESE A VERIFICAR depende de execução manual.

Complementar à suíte Cypress existente (login válido/inválido, bloqueado, acesso sem login, compra E2E com 2 itens, obrigatórios do checkout, 4 ordenações). Ambiente: **Chrome desktop** (responsividade via DevTools).

**Regra do documento:** nenhum comportamento do SauceDemo é afirmado como fato. Toda ideia, resultado esperado e bug pré-preenchido é **HIPÓTESE A VERIFICAR**.

---

# 1. Mapa de riscos — SauceDemo (regressão exploratória)



Escala: Alta/Alto = 3, Média/Médio = 2, Baixa/Baixo = 1. Risco = Probabilidade × Impacto. Ordenado do maior para o menor; empate decidido pelo impacto.

| # | Área | Probabilidade | Impacto | Risco (P×I) | Justificativa | Charter |
|---|---|---|---|---|---|---|
| R1 | Sessão e controle de acesso (cookie, URL direta, pós-logout) | Alta | Alto | 9 | Já existe um indício concreto (cookie `session-username` emitido para o locked_out_user). Se o controle de acesso depende só desse cookie no cliente, qualquer usuário bloqueado ou deslogado pode entrar nas páginas internas. A suíte só testa a tela de login, não o acesso por URL/cookie. | C6 |
| R2 | Estado do checkout (F5, voltar, URL direta nas etapas, duas abas) | Alta | Alto | 9 | Checkout em 3 etapas com estado no cliente é o ponto clássico de quebra: pular etapa, finalizar com carrinho vazio, dados do formulário perdidos. A suíte só cobre o fluxo linear. | C2 |
| R3 | Integridade do carrinho (contador x itens x total em fluxos não lineares) | Média | Alto | 6 | A suíte valida total = soma num fluxo feliz com 2 itens. Remover no meio, 1 item, todos os itens, e alternar entre listagem/detalhe/carrinho podem dessincronizar contador e total — erro de valor é de alto impacto. | C1 |
| R4 | Comportamento por perfil de usuário (problem/error/performance/visual) | Alta | Médio | 6 | Os perfis existem justamente para simular falhas. Probabilidade alta de diferenças; impacto médio porque o perfil é de teste, mas cada diferença representa uma classe real de defeito (imagem errada, botão que não responde, lentidão). | C5 |
| R5 | Persistência do carrinho entre logout/login e entre usuários | Média | Alto | 6 | Se o carrinho ficar no navegador e não for limpo no logout, um usuário pode ver/comprar itens de outro no mesmo computador (vazamento de dados entre contas). | C3 |
| R6 | Validação de dados no checkout (espaços, longos, especiais, emoji, CEP) | Alta | Médio | 6 | A suíte só testa campo vazio. Campo só com espaços, CEP com letras e textos enormes provavelmente passam ou quebram o layout. Impacto médio: gera pedido com dados inválidos. | C4 |
| R7 | Visual e responsividade (375 / 768 / 1366) | Média | Médio | 4 | Sem cobertura automatizada visual. Menu, botões e carrinho podem ficar inacessíveis em telas pequenas — bloqueia compra em mobile. | C7 |
| R8 | Catálogo, detalhe do produto e ordenação com estado | Baixa | Baixo | 1 | Ordenação já é coberta; o risco novo é ela se perder após F5/voltar ou o estado do botão Add/Remove divergir entre listagem e detalhe. | C8 |


**Observação:** as probabilidades são estimativas de QA, não fatos sobre o SauceDemo. Cada uma deve ser revista após as sessões (subir ou baixar conforme o que for confirmado).

---

# 2. Charters de sessão



Formato: *Explorar &lt;área&gt; com &lt;recursos/técnica&gt; para descobrir &lt;tipo de risco&gt;*.

| ID | Charter | Risco que justifica | Timebox | Usuário(s) | Heurísticas |
|---|---|---|---|---|---|
| C1 | Explorar o fluxo de compra com remoção de itens no meio do caminho, cancelamento e volta, e compras de 1 item e de todos os itens, para descobrir divergências entre contador, itens e total fora do caminho linear | R3 — Erro de valor/quantidade quando o usuário não segue o fluxo linear (a suíte só cobre 2 itens em linha reta). | 30 min | standard_user | SFDIPOT (Função, Dados), CRUD no carrinho, Interrupções, Zero-Um-Muitos |
| C2 | Explorar as etapas do checkout com botão voltar, F5, URL digitada direto e duas abas, para descobrir etapas puláveis, perda de dados e pedidos finalizados em estado inválido | R2 — Estado do checkout no cliente permite pular etapas, finalizar sem itens ou perder dados. | 30 min | standard_user | SFDIPOT (Tempo, Operações), Interrupções, Pular etapas, Recarregar/Voltar |
| C3 | Explorar logout, login de novo e troca de usuário no mesmo navegador para descobrir carrinho que persiste indevidamente ou vaza entre contas | R5 — Itens de um usuário aparecem para outro no mesmo navegador (vazamento entre contas) ou carrinho some sem aviso. | 20 min | standard_user, problem_user, visual_user | SFDIPOT (Dados, Plataforma), Persistência, Troca de contexto |
| C4 | Explorar os campos First Name, Last Name e Zip/Postal Code com espaços, textos longos, caracteres especiais, acentos, emojis, números e letras para descobrir validações ausentes e quebras de layout | R6 — Pedido criado com dados inválidos ou layout quebrado; a suíte só testa campo vazio. | 25 min | standard_user (e problem_user para comparar o campo Last Name) | Dados: limites, tipos inválidos, Unicode, espaços; Test Heuristics Cheat Sheet (strings) |
| C5 | Explorar o mesmo roteiro curto de compra com cada perfil de usuário, usando o standard_user como referência, para descobrir diferenças funcionais, visuais e de desempenho | R4 — Cada perfil simula uma classe real de defeito; sem um mapa de diferenças não sabemos o que a regressão deve detectar. | 30 min | standard_user (referência), problem_user, performance_glitch_user, error_user, visual_user | Oráculo de comparação (consistência com produto de referência), SFDIPOT (Função, Interface, Tempo) |
| C6 | Explorar cookies e acesso direto às páginas internas com DevTools, URL e botão voltar, para descobrir controle de acesso feito só no cliente | R1 — Usuário bloqueado ou deslogado acessa páginas internas; troca manual de cookie muda a identidade. | 25 min | locked_out_user, standard_user, problem_user | SFDIPOT (Plataforma, Operações), Fronteiras de confiança, Mudar o estado por fora da interface |
| C7 | Explorar as telas principais em 375, 768 e 1366 px com o DevTools para descobrir elementos cortados, desalinhados, imagens quebradas e mensagens de erro pouco claras | R7 — Em tela pequena o usuário não consegue concluir a compra; sem cobertura visual automatizada. | 25 min | standard_user (referência) e visual_user | SFDIPOT (Interface), Consistência visual, Acessibilidade básica (teclado/zoom) |
| C8 | Explorar listagem, página de detalhe e ordenação combinadas com carrinho, F5 e voltar, para descobrir estado de botão divergente e ordenação perdida | R8 — Divergência de estado entre listagem e detalhe; ordenação (já coberta) perdida após navegação — variação de estado, não repetição. | 20 min | standard_user | SFDIPOT (Estrutura, Dados), Consistência entre telas |

## Ideias de teste por charter

Todas as ideias são **HIPÓTESES A VERIFICAR** — descrevem o que observar, não o resultado esperado garantido.

### C1 — Jornadas E2E alternativas

| # | Ideia de teste | Foco |
|---|---|---|
| C1.1 | Comprar **1 item** só: conferir Item total, Tax e Total no checkout-step-two (variação nova: limite inferior, a suíte usa 2). | Dados |
| C1.2 | Adicionar **os 6 produtos**, conferir contador = 6 e se o Item total é a soma exata dos 6 preços da listagem. | Dados |
| C1.3 | Adicionar 3 itens, ir até o checkout-step-one, clicar **Cancel**, remover 1 no carrinho e seguir: o total da etapa 2 reflete 2 itens? | Estado |
| C1.4 | Na etapa 2 (Overview), clicar **Cancel**: para onde volta? O carrinho ainda tem os itens? Os dados do formulário foram mantidos? | Navegação |
| C1.5 | Adicionar pela listagem, remover pela **página de detalhe** do produto e voltar: botão da listagem mostra 'Add to cart' de novo? Contador atualizou? | Estado |
| C1.6 | Remover **todos** os itens no carrinho e clicar Checkout: deixa prosseguir com carrinho vazio? Qual o total mostrado? | Estado |
| C1.7 | Após o 'Back Home' da compra concluída, adicionar 1 item e comprar de novo: algum resíduo da compra anterior? | Estado |
| C1.8 | Conferir se o Tax parece proporcional ao Item total nos cenários de 1, 2 e 6 itens (anotar os valores para comparar). | Dados |

### C2 — Estado e navegação no checkout

| # | Ideia de teste | Foco |
|---|---|---|
| C2.1 | Preencher a etapa 1, ir para a etapa 2 e apertar **F5**: os dados e o total continuam? Apertar F5 também em carrinho, etapa 1 e tela 'complete'. | Estado |
| C2.2 | Na etapa 2, usar **voltar do navegador**: o formulário da etapa 1 está preenchido ou vazio? Avançar de novo funciona? | Navegação |
| C2.3 | Com carrinho **vazio**, digitar direto `/checkout-step-two.html` e clicar Finish: o pedido é concluído sem itens? | Navegação |
| C2.4 | Digitar direto `/checkout-complete.html` sem ter comprado nada: a tela de sucesso aparece? | Navegação |
| C2.5 | Na tela 'complete', usar **voltar** do navegador: volta para o Overview com itens? Dá para clicar Finish de novo (pedido duplicado)? | Navegação |
| C2.6 | Abrir **duas abas** logadas: na aba A adicionar 2 itens; na aba B (sem F5) o contador mostra quanto? Após F5? | Estado |
| C2.7 | Duas abas no checkout: finalizar na aba A; na aba B, ainda na etapa 2, clicar Finish: o que acontece com carrinho já vazio? | Estado |
| C2.8 | Na aba A fazer **logout**; na aba B (ainda aberta) tentar adicionar item e navegar: a aba B percebe que a sessão acabou? | Estado |

### C3 — Persistência do carrinho entre sessões e usuários

| # | Ideia de teste | Foco |
|---|---|---|
| C3.1 | standard_user: adicionar 2 itens, logout, login com standard_user de novo: o carrinho persiste? Anotar como comportamento observado (não há requisito escrito). | Estado |
| C3.2 | standard_user: adicionar 2 itens, logout, login com **visual_user**: os itens do standard_user aparecem no carrinho do visual_user? | Estado |
| C3.3 | Repetir trocando para problem_user: o contador e os itens herdados são os mesmos? | Estado |
| C3.4 | Com itens no carrinho, fazer **Reset App State** pelo menu: contador zera? Os botões da listagem voltam para 'Add to cart' sem F5? | Estado |
| C3.5 | Adicionar itens, **fechar a aba** e abrir o site de novo: continua logado? Carrinho mantido? | Estado |
| C3.6 | No DevTools > Application, ver o que existe em Local Storage e Cookies antes e depois do logout (anotar chaves e valores, sem alterar ainda). | Estado |

### C4 — Entradas do formulário de checkout

| # | Ideia de teste | Foco |
|---|---|---|
| C4.1 | Preencher os 3 campos **só com espaços** (3 espaços cada) e clicar Continue: aceita? (variação nova da suíte: espaço não é vazio). | Dados |
| C4.2 | Nome com espaço antes/depois (`  Marco  `): aceita? Há trim? (não há onde ver o dado depois — anotar só se aceita). | Dados |
| C4.3 | Colar **300 caracteres** em First Name: o campo limita? O layout quebra? Continue funciona? | Dados |
| C4.4 | Acentos e cedilha: `José`, `Conceição`, `Ñuñez`, `Müller`: aceita e mostra corretamente ao voltar com o botão voltar? | Dados |
| C4.5 | Emojis: `Marco 😀` e só `🚀🚀`: aceita? Cursor e apagar com Backspace funcionam? | Dados |
| C4.6 | Caracteres especiais e marcação: `O'Brien`, `<b>teste</b>`, `"; --`: aceita? O texto aparece renderizado como HTML em algum lugar? (só observar). | Dados |
| C4.7 | **Números no nome** (`12345`) e **letras no CEP** (`ABCDE`), CEP `00000`, CEP com 20 dígitos: algum é rejeitado? | Dados |
| C4.8 | Preencher com erro, ver a mensagem, corrigir só 1 campo: a mensagem some/atualiza para o campo certo? O 'X' do erro fecha a mensagem? | Usabilidade |

### C5 — Comparação entre perfis de usuário

| # | Ideia de teste | Foco |
|---|---|---|
| C5.1 | Roteiro fixo para todos: login → ordenar Z-A → abrir detalhe de 1 produto → adicionar 2 itens → remover 1 → checkout com `Ana / Silva / 30130` → Finish → logout. Anotar cada desvio numa tabela por perfil. | Função |
| C5.2 | problem_user: as imagens dos produtos são as mesmas do standard_user? A ordenação funciona? Os botões Add/Remove respondem? | Visual |
| C5.3 | problem_user: digitar no campo **Last Name** — o texto fica no campo certo? | Função |
| C5.4 | performance_glitch_user: cronometrar (cronômetro do celular) login e cada clique; anotar qualquer ação acima de ~2 s. | Desempenho |
| C5.5 | error_user: tentar adicionar/remover todos os produtos e clicar Finish: algum passo falha? Aparece erro no Console do DevTools? | Função |
| C5.6 | visual_user: comparar com print do standard_user — preços, posição do botão, ícone do carrinho, imagens, alinhamento. | Visual |
| C5.7 | Para cada perfil, conferir se o **total** da etapa 2 bate com a soma dos preços **mostrados na listagem daquele perfil**. | Dados |
| C5.8 | Tirar print da listagem de cada perfil na mesma resolução (1366) para virar evidência lado a lado. | Visual |

### C6 — Sessão e segurança básica (no navegador)

| # | Ideia de teste | Foco |
|---|---|---|
| C6.1 | **Achado em aberto:** login com locked_out_user, ver a mensagem de bloqueio, conferir em DevTools > Application > Cookies se `session-username` foi criado e com qual valor. | Estado |
| C6.2 | Ainda nessa situação, digitar `/inventory.html` na barra de endereço: abre o catálogo? Dá para adicionar item e ir até o Finish? | Navegação |
| C6.3 | Sem logar: criar manualmente o cookie `session-username` = `standard_user` (DevTools > Cookies > nova linha) e abrir `/inventory.html`. | Estado |
| C6.4 | Logado como standard_user: editar o cookie para `problem_user` e dar F5: o comportamento passa a ser o do problem_user (ex.: imagens)? | Estado |
| C6.5 | Editar o cookie para um valor que não existe (`usuario_x`) e para vazio: abre as páginas internas? Mostra erro? | Estado |
| C6.6 | Fazer logout e usar o **botão voltar** do navegador: mostra a página interna (cache) ou redireciona? Clicar em algo nela. | Navegação |
| C6.7 | Após logout, digitar direto `/cart.html`, `/checkout-step-one.html`, `/inventory-item.html?id=4`: redireciona para o login? Qual mensagem? (variação nova: a suíte cobre 'acesso sem login', não 'após logout' nem 'com cookie forjado'). | Navegação |
| C6.8 | Após logout, conferir se o cookie foi apagado e se o Local Storage do carrinho continua lá. | Estado |

### C7 — Visual, responsividade e usabilidade

| # | Ideia de teste | Foco |
|---|---|---|
| C7.1 | Em **375 px** (DevTools > Toggle device > Responsive): login, listagem, detalhe, carrinho, etapas 1–2 e complete. Algo cortado ou com rolagem horizontal? | Visual |
| C7.2 | Em 375 px: o menu (☰) abre, fecha e todos os links são clicáveis? O ícone do carrinho continua visível? | Usabilidade |
| C7.3 | Em **768 px** e **1366 px**: grade de produtos com quantas colunas? Botões alinhados na mesma linha? | Visual |
| C7.4 | Nome de produto longo + botão: algum texto sobrepõe o preço ou o botão em 375 px? | Visual |
| C7.5 | No DevTools > Network, filtrar 'Img' e recarregar a listagem: alguma imagem com erro (404) ou imagem trocada? | Visual |
| C7.6 | Ler cada mensagem de erro (login inválido, bloqueado, campos do checkout): diz o que fazer? Fica visível no 375 px? | Usabilidade |
| C7.7 | Navegar só com **teclado** (Tab/Enter) do login até o Finish: dá para concluir a compra? O foco é visível? | Usabilidade |
| C7.8 | Zoom do navegador em 200% (Ctrl +) em 1366 px: layout continua utilizável? | Visual |

### C8 — Catálogo, detalhe do produto e ordenação com estado

| # | Ideia de teste | Foco |
|---|---|---|
| C8.1 | Ordenar Price (high to low), abrir um produto e voltar com 'Back to products': a ordenação foi mantida? E com o voltar do navegador? | Navegação |
| C8.2 | Ordenar Z-A e dar F5: a ordenação e o texto do select continuam iguais? | Estado |
| C8.3 | Adicionar item na listagem, ordenar de novo: o botão do item continua 'Remove'? O contador mantém? | Estado |
| C8.4 | Comparar nome, descrição, preço e imagem da listagem com a página de detalhe para os 6 produtos. | Dados |
| C8.5 | Digitar `/inventory-item.html?id=` com valores 0, 5, 6, 99, -1 e `abc`: o que aparece? Dá para adicionar ao carrinho um produto inexistente? | Dados |
| C8.6 | Clicar no nome e na imagem do produto: os dois levam ao mesmo detalhe? | Navegação |

**Cobertura do critério de aceite:** 44 de 60 ideias (73%) exploram estado, navegação ou dados.

---

# 3. Modelo de session sheet

Arquivo: `03-session-sheets/_MODELO-session-sheet.md`. Uma ficha pré-preenchida por charter na mesma pasta.

---

# 4. Modelo de bug report

Arquivo: `04-bug-reports/_MODELO-bug-report.md`. Bug reports nomeados `C<charter>-BR-<nº>.md` (numeração reinicia em cada charter: C1-BR-01, C2-BR-01...).

**Critério de severidade (impacto técnico/negócio):**
- **S1 Crítica:** falha de segurança/acesso, pedido com valor errado, ou compra impossível para todos.
- **S2 Alta:** funcionalidade principal falha com contorno difícil, ou dados inválidos aceitos em pedido.
- **S3 Média:** falha em fluxo secundário ou com contorno simples.
- **S4 Baixa:** cosmético, texto, alinhamento sem impedir uso.

**Critério de prioridade (urgência de correção):**
- **P1:** corrigir antes da próxima entrega — afeta o perfil padrão (standard_user) ou segurança.
- **P2:** próxima sprint — afeta fluxo frequente com contorno.
- **P3:** backlog — perfil específico de teste, cosmético ou raro.

> No SauceDemo, os perfis problem/error/performance/visual têm defeitos **propositais**. Por isso o bug report registra a severidade técnica que o defeito teria num produto real, e a prioridade considera que ele é restrito ao perfil.

---

# 5. Critério de automação



## Regra (aplicar na ordem; a primeira que bater decide)

1. **Fica só no exploratório** se o resultado depende de julgamento humano (estética, clareza de texto, "parece lento") **ou** não é reproduzível em pelo menos 3 de 3 tentativas.
2. **Vira teste automatizado** se é reproduzível, tem resultado esperado objetivo (uma asserção estável: URL, texto, valor, cookie) **e** o risco é Alto ou Médio no mapa (R1–R6).
3. **Fica como bug conhecido registrado** se é reproduzível e objetivo, mas o risco é Baixo, **ou** é defeito proposital restrito a um perfil de teste. Na suíte, entra no máximo como teste marcado (ex.: `it.skip` com o ID do bug) para ser reativado quando corrigido.

Teste rápido: **"Uma máquina consegue dizer passou/falhou sem eu olhar? Vale o custo de manter?"** Não → exploratório. Sim e risco alto/médio → automatizar. Sim e risco baixo/proposital → bug conhecido.

## Aplicação a 3 exemplos

| Achado | Reproduzível e objetivo? | Risco | Decisão | Como |
|---|---|---|---|---|
| C6-BR-01 — locked_out_user com cookie e acesso por URL | Sim (cookie existe ou não; URL redireciona ou não) | R1 Alto | **Automatizar** | Cypress: logar com locked_out_user, `cy.getCookie('session-username').should('not.exist')`, `cy.visit('/inventory.html', {failOnStatusCode:false})` e `cy.url().should('eq', Cypress.config().baseUrl + '/')`. Se o bug for confirmado, o teste falha até a correção — fica visível no CI. |
| C4-BR-01 — campos só com espaços aceitos | Sim (aparece mensagem de erro ou avança para step-two) | R6 Médio | **Automatizar** | Parametrizar o teste de obrigatórios que já existe com `'   '` além de `''` — baixo custo, reaproveita a suíte. |
| C7-BR-01 — desalinhamento do visual_user | Parcial: diferença de **preço** é objetiva; **alinhamento** depende de olho humano | R7 Médio, perfil proposital | **Bug conhecido** (preço) + **exploratório** (alinhamento) | Registrar o bug; alinhamento segue em sessões exploratórias até existir ferramenta de comparação visual (fora do escopo atual). |

---

---

# 6. Scripts Cypress e registro dos prompts (complemento de 01/10/2026 08:31)

- `00-prompt-ia/`: prompts enviados à IA com data/hora do pedido e da resposta, perguntas e respostas de esclarecimento, e um prompt reutilizável para a IA preencher a session sheet a partir dos achados.
- `06-cypress/`: projeto Cypress para abrir no VS Code — um spec `.cy.js` por charter (C1 a C8). Cada `it` tem o ID da ideia (C#.#) e o bug ligado [BR-xx]. Instruções em `06-cypress/README.md`.
- Regra de leitura: **vermelho na asserção = hipótese confirmada**; vermelho por *element not found* = ajustar `selectors.js`; `cy.registrar` = observação sem requisito, salva em `cypress/results/achados-*.json`.

---

# Se eu tivesse só 1 hora

1. **C6 — Sessão e segurança (25 min):** fecha o achado em aberto do cookie do locked_out_user; é o único risco com impacto de segurança e já tem indício real.
2. **C2 — Estado e navegação no checkout (25 min):** URL direta e voltar/F5 nas etapas podem gerar pedido vazio ou duplicado, e nada disso está na suíte.
3. **10 min de registro:** transformar o que for confirmado em bug report e marcar o que vira automação — achado sem registro não reduz risco.
