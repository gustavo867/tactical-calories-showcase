# Notas das capturas

[Voltar ao showcase](../../README.md) · [English](README.en.md)

Estas imagens são capturas existentes de desenvolvimento e validação da aplicação, copiadas sem alterações visuais. Mostram a interface em português brasileiro e registros ilustrativos. Foram inspecionadas visualmente antes da inclusão; não foram encontrados e-mails, credenciais ou identificadores de conta visíveis nas capturas selecionadas.

As imagens vêm de diferentes versões de desenvolvimento. Rótulos de navegação e ações disponíveis podem variar; não representam uma captura sincronizada de uma única versão. O painel diário, em particular, é de uma versão anterior, enquanto o detalhe offline demonstra um fluxo posterior.

| Arquivo no showcase                                            | Caminho da captura no projeto privado              | Conteúdo                                                                          |
| -------------------------------------------------------------- | -------------------------------------------------- | --------------------------------------------------------------------------------- |
| [daily-dashboard-desktop.png](daily-dashboard-desktop.png)     | `docs/screenshots/meal-cards/desktop.png`          | Saldo de calorias, progresso de macros, refeições e seção de hidratação           |
| [meal-actions-desktop.png](meal-actions-desktop.png)           | `docs/screenshots/issue-100-diario-desktop.png`    | Copiar refeição, salvar favorita, observações e consumo de água                   |
| [food-search-mobile.png](food-search-mobile.png)               | `docs/validacao/issue-102/issue-102-busca-390.png` | Busca de alimentos e resumo da seleção                                            |
| [favorite-meals-mobile.png](favorite-meals-mobile.png)         | `docs/screenshots/refeicoes-favoritas/mobile.png`  | Combinação de refeição salva e ações de gestão                                    |
| [offline-food-entry-mobile.png](offline-food-entry-mobile.png) | `docs/evidence/issue-116-offline-mobile.png`       | Registro individual offline ao lado de uma seleção cuja confirmação exige conexão |

A captura offline inclui uma página longa de detalhe do alimento e uma ação separada para registro individual offline. A presença de uma seleção nessa página não significa que a confirmação em lote funcione offline.

Os caminhos de origem documentam a procedência; não é necessário acessar o repositório privado para visualizar as imagens deste showcase. Nenhum histórico do repositório original, relatório de testes ou exportação de dados de usuários foi copiado.

## Capturas incorporadas na atualização v0.7.0

As capturas abaixo foram inspecionadas antes da publicação. Os arquivos de catálogo e Meus alimentos foram obtidos via `gh` na `main` confirmada do produto, commit `8f5a586480bca3ac2c049ad40b0c7d7531c4a734`. As duas capturas de cópia são os originais locais da PR #136, também publicados no comentário indicado abaixo. As datas e nomes ilustrativos das telas pertencem aos cenários de teste; não indicam a data do release.

| Arquivo no showcase                                                    | Origem no produto                                       | Conteúdo                                                                  |
| ---------------------------------------------------------------------- | ------------------------------------------------------- | ------------------------------------------------------------------------- |
| [catalog-tilapia-desktop.png](catalog-tilapia-desktop.png)             | `docs/evidence/issue-133-ampliacao/tilapia-desktop.png` | Tilápia preparada, fonte USDA e prévia de 50 g no desktop                 |
| [catalog-tilapia-mobile.png](catalog-tilapia-mobile.png)               | `docs/evidence/issue-133-ampliacao/tilapia-mobile.png`  | Mesmo fluxo no celular                                                    |
| [catalog-mustard-desktop.png](catalog-mustard-desktop.png)             | `docs/evidence/issue-133/mostarda-desktop.png`          | Mostarda preparada com atribuição USDA no desktop                         |
| [catalog-mustard-mobile.png](catalog-mustard-mobile.png)               | `docs/evidence/issue-133/mostarda-mobile.png`           | Mesmo fluxo no celular                                                    |
| [copy-yesterday-desktop.png](copy-yesterday-desktop.png)               | `.codex/tmp/issue-132-shortcut.png`, PR #136            | Atalho no card de almoço da data selecionada                              |
| [copy-yesterday-review-desktop.png](copy-yesterday-review-desktop.png) | `.codex/tmp/issue-132-final.png`, PR #136               | Revisão com origem em 01/10 e destino em 02/10/2026                       |
| [my-foods-desktop.png](my-foods-desktop.png)                           | `docs/validation/assets/issue-134.png`                  | Edição de itens próprios públicos e privados; exclusão somente no privado |

Referências privadas de origem: [cópia de ontem](https://github.com/gustavo867/tactical-calories/pull/136#issuecomment-6695937164), [Meus alimentos](https://github.com/gustavo867/tactical-calories/pull/137#issuecomment-6696234174) e [seleção da promoção](https://github.com/gustavo867/tactical-calories/pull/142#issuecomment-6716315339). Os PNGs completos estão neste showcase e não dependem desses links para visualização. As imagens demonstram telas de validação, sem comprovar deploy hospedado, interação contínua ou uso em dispositivo físico.
