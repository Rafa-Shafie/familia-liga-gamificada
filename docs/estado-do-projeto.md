# Estado do Projeto

*Última atualização: 09/11/2026*

## Resumo
- **Onde paramos:** mudança de rota. O projeto deixou de ser o sistema de uma família e passou a ser um produto multi-família, para prática e portfólio (ADR-006). Fatia 0 concluída; roadmap refeito.
- **O que está commitado:** infraestrutura do compose (Postgres 18), `.env` fora do git com `.env.example`, configuração do datasource, Flyway com V1, CLAUDE.md, ADRs 001 a 006 e README.
- **Fatia atual:** Fatia 1 — Registrar conclusão, já com família e membros como tabelas (ainda não iniciada).

## Mudança de rota (09/11/2026)
- ADR-006 criado: produto multi-família, sem hospedagem pública, pronto para deploy.
- ADR-001 (infraestrutura local) e ADR-002 (sem autenticação) marcados como substituídos.
- ADR-003 revisado: membros como registros de uma família; fatia de recorrência renumerada.
- ADR-004 e ADR-005 continuam valendo sem mudança.
- CLAUDE.md: produto, decisões fechadas (1 a 4 e 8 mudaram, entrou a 10), stack e roadmap atualizados.
- A família do autor passa a ser a família de exemplo do seed e testa cada fatia, sem uso diário.

## Fatia 0 — Retomada (concluída)

- [x] Docker Desktop (WSL 2) instalado e testado
- [x] Postgres 18 subindo pelo compose (healthy) e visível no DBeaver
- [x] `.env` fora do git e `.env.example` criado
- [x] Spring conectado ao Postgres (datasource lendo o `.env` via `spring.config.import`)
- [x] Flyway aplicando `V1__init.sql`, Hibernate em `validate`, `open-in-view: false`
- [x] ADR-004 — Flyway como único dono do schema
- [x] CLAUDE.md com as regras de trabalho e as decisões fechadas
- [x] ADRs 001 a 003 revisados e ADR-005 — Histórico de pontos

## Próximo passo concreto
Modelar a V2 da Fatia 1: família, membro, tarefa e conclusão, com tudo pertencendo a uma família. Junto com ela, escrever o ADR-007 (como o isolamento entre famílias é armazenado) e decidir o tipo de ID e a forma de carregar o seed.

## Pendências técnicas (não são decisões)
- `Dockerfile` ainda é placeholder e não está commitado. Entra na Fatia 5 (pronto para deploy).
- Serviço `back-spring` do compose ainda não recebe as variáveis do banco, e a URL do datasource aponta para `localhost`. Fatia 5.

## Backlog de decisões adiadas
| Decisão | Contexto | Revisitar na |
|---|---|---|
| Forma de carregar o seed | A família de exemplo (a do autor) vai numa migration, e portanto para qualquer banco, ou só no ambiente de desenvolvimento? | Fatia 1 |
| IDs sequenciais × UUID | IDs sequenciais nas URLs facilitam adivinhar recursos de outra família (IDOR) | Fatia 1 |
| Estratégia de testes de integração | O teste `contextLoads` depende do Postgres do compose rodando; avaliar Testcontainers. Ganha peso com os testes de isolamento | Fatia 1 |
| Aplicação do isolamento entre famílias | Onde o filtro por família é garantido (em cada consulta, num ponto central, no banco) e como é testado | Fatia 2 |
| Mecanismo de autenticação | Sessão com cookie × token (JWT); a tela é servida pelo próprio Spring | Fatia 2 |
| Usuário em várias famílias | Separar usuário (quem faz login) de membro (pessoa numa família); um usuário pode estar em mais de uma família, por exemplo pais separados? | Fatia 2 |
| Fórmula de pontos | Esforço braçal, tempo e frequência semanal; número digitado ou calculado? | Fatia 3 |
| Contas de crianças e LGPD | A LGPD (art. 14) exige consentimento dos responsáveis para dados de crianças. Ideias: criança não cria conta; perfil criado pelo responsável, sem e-mail e senha; criança sem permissão para criar tarefas | Fatia 3 |
| Envio de e-mail | Convites e recuperação de senha precisam de SMTP; no desenvolvimento, um container que captura os e-mails (Mailpit) | Fatia 4 |
| Exclusão de conta × histórico | O direito de eliminação da LGPD conflita com o histórico imutável de conclusões (ADR-005) | Fatia 4 |
| Conversão de pontos em mesada | Regra ainda não definida; num produto, provavelmente configurada por família | Fatia 6 |
| Início da semana configurável | Segunda, domingo ou sábado; num produto, por família | Fatia 6 |
| Fuso horário por família | Hoje fixo em America/Sao_Paulo; num produto, cada família poderia ter o seu | Fatia 6 |
| Conclusão avulsa ↔ instância gerada | Como o registro da Fatia 1 se liga às instâncias do `@Scheduled` | Fatia 7 |
| Penalidade por card não concluído | O ADR-003 sugere auditoria, mas sem consequência definida | Fatia 7 |
| Motivo da transferência + penalidade | Tarefa atribuída a um membro e concluída por outro: registrar o motivo (ex.: "esqueceu", "estava fora") e penalizar conforme o motivo. Ideia do Rafa em 02/11/2026 | Fatia 7 |
| Frequência variável das tarefas | Tarefas como louça e passear com o cachorro acontecem zero, uma ou várias vezes por dia, o que não cabe em um card por instância | Fatia 7 |
