# Arquitetura

[Voltar ao showcase](../README.md) · [English](architecture.en.md)

A aplicação separa interface estática, réplica local, API autenticada e banco de dados. A persistência offline é uma responsabilidade explícita da aplicação e não depende de cache de respostas autenticadas da API.

```mermaid
flowchart TB
    subgraph Device["Navegador / PWA instalada"]
        UI["React + Next.js: exportação estática"]
        SW["Service Worker: arquivos estáticos"]
        DB[("IndexedDB: réplica e fila por conta")]
        SYNC["Sincronização em primeiro plano"]
        UI -->|"ações locais suportadas"| DB
        SW -->|"arquivos da aplicação em cache"| UI
        DB <-->|"ler pendências / aplicar confirmação"| SYNC
    end

    UI -->|"API HTTPS na mesma origem"| API["API NestJS"]
    SYNC -->|"verificar sessão, carregar estado, enviar operações"| API
    API <-->|"Google OAuth / verificar identidade"| AUTH["Supabase Auth"]
    API --> PRISMA["Prisma: identidade verificada no escopo da transação"]
    PRISMA --> PG[("PostgreSQL: funções SQL, RLS e comprovantes")]
    CONTRACTS["Schemas Zod + tipos TypeScript compartilhados"] -.-> UI
    CONTRACTS -.-> API
```

## Responsabilidades

**Frontend.** O Next.js exporta arquivos estáticos, sem executar handlers de sessão no servidor. O navegador usa a API NestJS na mesma origem para operações autenticadas. A interface distingue persistência local de confirmação remota.

**Réplica local.** O IndexedDB mantém estado confirmado, catálogo autorizado e operações pendentes em partições por conta. O estado exibido é uma projeção das pendências sobre os dados confirmados. O rascunho da seleção de vários alimentos é separado: pode ser recuperado localmente, mas não representa consumo enfileirado nem é sincronizado entre dispositivos.

**Service Worker.** Mantém o cache dos arquivos da aplicação. Respostas autenticadas da API e tráfego OAuth ficam fora desse cache; os dados da conta são gerenciados pela réplica.

**Backend.** O NestJS conduz o Google OAuth por meio do Supabase Auth e verifica a identidade antes de acessar dados da conta. Tokens de autenticação ficam em cookies HttpOnly; o JavaScript recebe um perfil mínimo da conta. Contratos Zod compartilhados validam os dados trocados.

**Banco de dados.** O Prisma abre transações com claims derivados da sessão verificada. Funções SQL aplicam regras de domínio e mutações transacionais, com segurança por linha (RLS) protegendo os dados de cada conta. O navegador não se conecta diretamente ao PostgreSQL.

## Fluxo de uma ação offline suportada

1. Após a preparação online da conta, a pessoa registra uma ação individual do diário sem conexão.
2. Uma transação IndexedDB persiste a operação na partição da conta. A projeção local a torna visível; falhas de gravação não informam que o salvamento foi concluído.
3. Ao reconectar, voltar ao aplicativo, abri-lo ou solicitar sincronização manual, o remetente adquire o lock da conta e revalida a sessão.
4. As operações pendentes são enviadas em ordem, com identificadores e payloads originais.
5. O PostgreSQL aplica a operação e grava seu comprovante na mesma transação, ou devolve um conflito recuperável.
6. Na confirmação, o cliente remove apenas a operação reconhecida, atualiza o estado confirmado e projeta a fila restante, incluindo ações criadas durante a requisição.

O servidor verifica a conta esperada além da sessão. Uma sessão expirada preserva as pendências para reautenticação; trocar de conta não transfere esse trabalho para a nova conta.

## Concorrência e recuperação

Web Locks coordenam a sincronização entre abas. Quando indisponíveis, um lease no IndexedDB funciona como alternativa. O BroadcastChannel avisa as outras abas para reler o estado. Esses mecanismos reduzem envios simultâneos; os comprovantes no banco garantem repetição segura mesmo quando o cliente reenvia uma operação.

Respostas de bootstrap são validadas e comparadas à versão do estado confirmado capturada antes da requisição. Uma resposta antiga que chega depois de uma confirmação mais nova não pode sobrescrever esse estado atualizado.

Conflitos no histórico de metas ou nas referências nutricionais pausam a fila para resolução explícita. Uma versão não substitui silenciosamente a outra.

## Hospedagem

O projeto privado possui configuração para servir o frontend exportado e a API NestJS na Vercel, com Supabase fornecendo Auth e PostgreSQL. O diagrama descreve o projeto da aplicação. Não certifica o estado de um deploy hospedado, a configuração OAuth ou os headers de cache em produção.
