# Estado do Projeto

*Última atualização: 31/10/2026*

## Resumo
- **Onde paramos:** Fatia 0 concluída. Spring Boot conectado ao Postgres, Flyway aplicando a V1, ADRs 001 a 005 coerentes com as decisões fechadas.
- **O que está commitado:** infraestrutura do compose (Postgres 18), `.env` fora do git com `.env.example`, configuração do datasource, Flyway com V1, CLAUDE.md e ADRs 001 a 005.
- **Fatia atual:** Fatia 1 — Registrar conclusão (ainda não iniciada).

## Fatia 0 — Retomada

- [x] Docker Desktop (WSL 2) instalado e testado
- [x] Postgres 18 subindo pelo compose (healthy) e visível no DBeaver
- [x] `.env` fora do git e `.env.example` criado
- [x] Spring conectado ao Postgres (datasource lendo o `.env` via `spring.config.import`)
- [x] Flyway aplicando `V1__init.sql`, Hibernate em `validate`, `open-in-view: false`
- [x] ADR-004 — Flyway como único dono do schema
- [x] CLAUDE.md com as regras de trabalho e as decisões fechadas
- [x] ADR-001 revisado: PC sempre ligado no MVP, Raspberry Pi depois, Tailscale, alternativas Vercel/Supabase
- [x] ADR-002 revisado: risco de fraude aceito, conferência manual, relação com a mesada
- [x] ADR-003 revisado: status Proposto (Fatia 4), argumento de normalização alinhado ao ADR-005
- [x] ADR-005 — Histórico de pontos: um registro por conclusão, pontos congelados, totais calculados

## Próximo passo concreto
Começar a Fatia 1: modelar o registro de conclusão e o cadastro mínimo de tarefas que o seed precisa (só o que a fatia exige).

## Pendências técnicas (não são decisões)
- `Dockerfile` ainda é placeholder e não está commitado. Necessário ao fim da Fatia 1, para a família usar em casa.
- Serviço `back-spring` do compose ainda não recebe as variáveis do banco. Mesma época do Dockerfile.
- Fatia de deploy: Docker Desktop iniciando com o Windows e containers com `restart`, por causa dos reinícios do Windows Update.

## Backlog de decisões adiadas
| Decisão | Contexto | Revisitar na |
|---|---|---|
| Fórmula de pontos | Esforço braçal, tempo e frequência semanal; número digitado ou calculado? | Fatia 2 |
| Conversão de pontos em mesada | Motivação central do projeto, regra ainda não definida | Fatia 3 |
| Conclusão avulsa ↔ instância gerada | Como o registro da Fatia 1 se liga às instâncias do `@Scheduled` | Fatia 4 |
| Penalidade por card não concluído | O ADR-003 sugere auditoria, mas sem consequência definida | Fatia 4 |
| Início da semana configurável | Segunda, domingo ou sábado | Fatia 3 |
| Estratégia de testes de integração | O teste `contextLoads` depende do Postgres do compose rodando; avaliar Testcontainers | Fatia 1 |
