# Estudos de engenharia

[Voltar ao showcase](../README.md) · [English](engineering.en.md)

Estes casos explicam decisões de implementação na aplicação privada. As referências a testes indicam cobertura existente; não significam que essas suítes foram executadas novamente durante a preparação desta documentação.

## 1. Repetir uma requisição bem-sucedida sem registrar a refeição duas vezes

**Cenário de falha.** O PostgreSQL confirma um registro alimentar offline, mas a conexão cai antes de o navegador receber a resposta. A operação continua na fila local, e o navegador não pode deduzir com segurança se ela foi concluída.

**Decisão.** Manter um `operationId` estável e preservar o payload original nas repetições. O banco grava um comprovante associado à conta autenticada na mesma transação da mutação. Repetir a mesma operação devolve uma confirmação de repetição e o estado atual. Reutilizar o identificador com outro payload retorna conflito de idempotência.

Fluxos de lote e registro de refeições usam valores estáveis de `attemptId`. Quando a resposta pode ter sido perdida, a interface conserva a tentativa original para repetição antes de permitir uma nova submissão editada. A confirmação em lote é transacional: todos os itens são registrados juntos ou nenhum é.

**Resultado.** Um timeout pode ser recuperado sem exigir que a pessoa adivinhe se outro clique duplicará o consumo. Isso garante efeitos idempotentes na repetição; não pressupõe que a rede entregue cada requisição exatamente uma vez.

**Cobertura existente.** `frontend/e2e/offline-sync.spec.ts`, fluxos HTTP do backend e testes SQL de comprovantes exercitam perda de resposta e repetição. Os fluxos de lote e refeições também testam tentativas repetidas e payloads alterados.

## 2. Resolver conflitos sem reescrever o histórico nutricional

**Cenário de falha.** A pessoa altera metas offline enquanto outro dispositivo as atualiza online. Um registro alimentar offline também pode usar valores nutricionais que mudaram antes da sincronização.

**Decisão.** Operações de metas carregam uma revisão-base; consumos do catálogo carregam um hash nutricional. O servidor retorna `goals_conflict` ou `food_changed` junto ao estado atual. O cliente preserva a operação pendente e pausa o envio para uma decisão. A recuperação de metas permite descartar a alteração ou aplicá-la a partir de hoje; a recuperação nutricional compara os valores usados originalmente com a referência atual.

O consumo registrado também conserva seu snapshot nutricional e, quando disponível, uma base de cálculo imutável. Edições posteriores no catálogo não recalculam o passado. Macronutrientes ausentes continuam desconhecidos, em vez de serem convertidos para zero.

**Custo da escolha.** A recuperação explícita exige interface e lógica de domínio adicionais. Em troca, preserva o significado do registro, sem esconder uma alteração atrás de um indicador de sincronização concluída.

**Cobertura existente.** `frontend/e2e/offline-conflicts.spec.ts` exercita recuperação de metas, referências alteradas e alimentos indisponíveis. As suítes de quantidade e precisão nutricional cobrem cálculos e preservação dos valores históricos.

## 3. Manter o estado correto entre abas e respostas atrasadas

**Cenário de falha.** Duas abas enviam pendências simultaneamente, ou uma requisição antiga de bootstrap termina depois da confirmação de uma mutação mais nova. A pessoa também pode criar outra ação local enquanto uma requisição está em andamento.

**Decisão.** Serializar remetentes com Web Locks ou um lease no IndexedDB e avisar as demais abas por BroadcastChannel. Armazenar uma versão do estado confirmado e rejeitar gravações de bootstrap desatualizadas. Aplicar confirmações em transações e reconstruir a projeção visível a partir da fila pendente atual, não da fila que existia no início da requisição.

**Custo da escolha.** O armazenamento local precisa de transições explícitas, verificações de versão e lógica de recuperação. O benefício é preservar tanto os dados confirmados mais novos quanto as pendências recém-criadas.

**Cobertura existente.** `frontend/e2e/offline-bootstrap-race.spec.ts` cobre respostas antigas de bootstrap. `frontend/e2e/offline-sync.spec.ts` cobre o lease alternativo e ações criadas durante uma requisição. Testes unitários verificam lock, ciclo de sincronização e projeção.

## 4. Preservar pendências sem cruzar os limites entre contas

**Cenário de falha.** A conta A possui alterações pendentes quando sua sessão expira. A próxima pessoa entra na conta B no mesmo navegador.

**Decisão.** Separar os dados locais por conta, revalidar a sessão antes do envio e informar a conta esperada nas requisições. Manter a fila da conta A para a própria conta A, sem enviá-la pela sessão de B. Uma resposta `401` solicita reautenticação sem apagar pendências. A saída local desativa o acesso local sem afirmar que a sessão remota foi revogada.

**Custo da escolha.** Identidade, acesso local e sessão de rede precisam de tratamento separado. O armazenamento do navegador continua sujeito a limpeza e expurgo; conservar uma fila não equivale a garantir backup permanente.

**Cobertura existente.** `frontend/e2e/offline-accounts.spec.ts` cobre expiração da sessão, troca de conta e retomada do trabalho pela conta original. Testes HTTP e SQL verificam os limites de autorização.

## 5. Ampliar fontes e permitir edição sem perder referências históricas

**Problema.** Somar outra fonte ao catálogo pode colidir códigos ou alterar IDs existentes. Permitir que o autor edite um alimento compartilhado também exige separar a referência atual do valor efetivamente consumido no passado.

**Decisão.** Preservar os 597 IDs TACO e adicionar 19 referências USDA por migrations novas, com identidade global estável, código textual por fonte e proveniência conservada. Conferir os lotes contra o arquivo oficial, incluindo checksum e nutrientes, permite reproduzir a verificação sem depender de uma consulta externa em tempo de uso. TBCA não foi incorporada.

A edição de alimentos privados e públicos próprios mantém ID e visibilidade, com autorização por autor. A referência atual serve a novos registros; snapshots e bases históricas permanecem intactos. A exclusão pública continua bloqueada. Se um consumo offline usar uma referência que mudou, o conflito precisa de revisão em vez de substituição silenciosa.

**Cobertura existente.** As suítes `additional-catalog.spec.ts` e `catalog-expansion.spec.ts` verificam tilápia, mostarda, fonte, cálculo proporcional e catálogo offline atualizado. Integração HTTP e SQL verificam autoria, IDs e histórico; as specs `public-food-management*.spec.ts` cobrem edição, recuperação, conflito e sincronização.

## 6. Selecionar CI sem depender apenas do grafo de imports

**Problema.** Schemas compilados, imports dinâmicos e migrations enumeradas pelo sistema de arquivos criam dependências que um seletor por imports pode não enxergar.

**Decisão.** Selecionar suítes completas por pacote em `staging`, ampliando para todas as suítes unitárias diante de mudanças compartilhadas, remoções, renomeações, caminhos desconhecidos ou falhas de comparação. O job de seleção sempre testa os próprios scripts. Alterações somente documentais podem dispensar jobs da aplicação. `main` e execução manual mantêm banco, integração HTTP e toda a matriz Playwright.

O projeto também habilita o armazenamento global de dependências do Bun 1.4.4 com `linker = "isolated"` e `globalStore = true`, compartilhando pacotes elegíveis entre projetos e worktrees. Isso é distinto do cache de downloads restaurado pela CI. O Docker mantém `--linker hoisted` para copiar dependências locais entre estágios.

O cache global de downloads Bun (`~/.bun/install/cache`) não substitui `bun install --frozen-lockfile`. O cache intermediário Next.js é separado do artifact estático produzido na execução atual. O gate aceita jobs ignorados somente quando previstos no plano e rejeita falhas, cancelamentos ou jobs ausentes. Não há medição de economia de tempo ou custo publicada aqui.

**Qualidade da promoção.** Foram corrigidas fixtures de refresh e restauração do catálogo, além da espera pela requisição abortada no teste de lote. O limite E2E passou para 45 minutos sem reduzir cobertura. O teste de concorrência de rascunhos passou a controlar notificações entre duas abas, mantendo a verificação de rejeição da revisão antiga; essa última correção não alterou a aplicação.

## Estratégia de validação

| Camada                                   | O que verifica                                                                              |
| ---------------------------------------- | ------------------------------------------------------------------------------------------- |
| Testes unitários de contratos e frontend | Schemas, transições da fila, projeção, locks, classificação de erros e cálculos             |
| Integração HTTP do backend               | NestJS, identidade verificada, autorização, Prisma e PostgreSQL em conjunto                 |
| Integração SQL                           | Invariantes do domínio, limites entre contas, transações e comprovantes                     |
| Playwright                               | Fluxos visíveis pelo frontend estático e API, incluindo funcionamento offline e recuperação |
| GitHub Actions                           | Seleção conservadora por pacote em `staging`; validação completa em `main` e manual         |

Os testes de integração usam PostgreSQL isolado. A fixture local de Auth substitui o provedor externo de identidade, não o catálogo alimentar ou o banco da aplicação. Os testes de navegador incluem configurações desktop, Chromium mobile e WebKit mobile; casos individuais podem ser específicos de um navegador.

O showcase evita afirmações sem verificação sobre latência, percentuais de cobertura, disponibilidade em produção ou contagens fixas de testes aprovados. Esses dados exigem evidências de execução datadas e associadas à revisão correspondente da aplicação.

## Referência da v0.7.0

Conferido via `gh` em 08/11/2026: a promoção v0.7.0 foi integrada à `main` no commit `8f5a586480bca3ac2c049ad40b0c7d7531c4a734`. A execução completa aprovada é a [42449310809](https://github.com/gustavo867/tactical-calories/actions/runs/42449310809), no HEAD pré-merge `f9768bb44837b078e95757bc13ab4fbca05adb87`: base, desktop, mobile, mobile Safari e gate aprovados. Supabase Preview foi ignorado pela configuração. Esse resultado é datado e não representa CI reexecutada no showcase nem validação manual de deploy hospedado.

As referências abaixo pertencem ao repositório privado do produto; o conteúdo público do showcase e suas imagens não dependem de acesso a ele.

- Entregas: PRs [#135](https://github.com/gustavo867/tactical-calories/pull/135), [#136](https://github.com/gustavo867/tactical-calories/pull/136) e [#137](https://github.com/gustavo867/tactical-calories/pull/137).
- CI: PRs [#139](https://github.com/gustavo867/tactical-calories/pull/139) e [#140](https://github.com/gustavo867/tactical-calories/pull/140).
- Promoção e correções: PRs [#142](https://github.com/gustavo867/tactical-calories/pull/142), [#143](https://github.com/gustavo867/tactical-calories/pull/143), [#144](https://github.com/gustavo867/tactical-calories/pull/144) e [#145](https://github.com/gustavo867/tactical-calories/pull/145).
- Documentação consultada no commit confirmado: `README.md`, `docs/backend-architecture.md`, `docs/ci.md`, `docs/validation/issue-133-ampliacao.md`, `docs/validation/issue-134.md`, `docs/validation/pr-142.md` e `docs/validation/pr-142-draft-concurrency.md`.
