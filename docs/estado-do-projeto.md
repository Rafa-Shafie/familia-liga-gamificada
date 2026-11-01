# Estado do Projeto

*Última atualização: 31/10/2026*

## Resumo
- **Onde paramos:** Spring Boot conectado ao Postgres, com o Flyway aplicando a migration inicial (V1). Falta só revisar os ADRs para fechar a Fatia 0.
- **O que está commitado:** infraestrutura do compose (Postgres 18), `.env` fora do git com `.env.example`, configuração do datasource, Flyway com V1, ADR-004 e CLAUDE.md.
- **Fatia atual:** Fatia 0 — Retomada.

## Fatia 0 — Retomada

- [x] Docker Desktop (WSL 2) instalado e testado
- [x] Postgres 18 subindo pelo compose (healthy) e visível no DBeaver
- [x] `.env` fora do git e `.env.example` criado
- [x] Spring conectado ao Postgres (datasource lendo o `.env` via `spring.config.import`)
- [x] Flyway aplicando `V1__init.sql`, Hibernate em `validate`, `open-in-view: false`
- [x] ADR-004 — Flyway como único dono do schema
- [x] CLAUDE.md com as regras de trabalho e as decisões fechadas
- [ ] Revisar ADR-001: PC sempre ligado no MVP, Raspberry Pi depois, Tailscale para acesso externo
- [ ] Revisar ADR-002: risco de fraude aceito, conferência manual, relação com a mesada
- [ ] Revisar ADR-003: status (recorrência foi para a Fatia 4) e pontos gravados no momento da conclusão
- [ ] Decidir quais decisões fechadas do CLAUDE.md sem ADR merecem um (ex.: histórico por conclusão + pontos só daqui pra frente)

## Próximo passo concreto
Revisar o ADR-001 junto, fechando a Fatia 0. Depois: começar a modelagem da Fatia 1 (registrar conclusão).

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
