# Estado do Projeto

*Última atualização: 10/11/2026*

## Resumo
- **Onde paramos:** escopo redefinido (ADR-007) e ajustado pelo ADR-012: o event sourcing passa a ser justificado por medição comparativa, entram o estorno, a coluna `registrado_em` e a consulta Q8, e o particionamento (Fatia 6) vira opcional ao final. As decisões de modelagem da Fatia 1 estão fechadas nos ADRs 008 a 011.
- **O que está commitado:** ADRs 007 a 011, CLAUDE.md, README e este arquivo (commits de 10/11/2026). **O ADR-012 e as atualizações que ele causou estão escritos, mas ainda não commitados.**
- **Fatia atual:** Fatia 1 — Registrar conclusão como evento (ainda não iniciada).
- **Planejamento do Rafa:** a Liga é um projeto de cerca de 3 meses, o item 02 do roadmap geral, depois do agente de candidaturas. As Fatias 1 e 1b avançam até 17/01/2027 se couberem, sem pressão.

## Ajuste do ADR-012 (10/11/2026)
- **Prova de B por comparação:** o placar e as sequências são calculados por SQL direto, projeção incremental, replay completo, rollup e replay a partir de snapshot, e o relatório final responde quando cada forma compensa.
- **Estorno** (`ConclusaoEstornada`), com índice único contra estorno duplicado: Fatia 2.
- **`registrado_em`** (quando o sistema soube do evento), na migration da Fatia 2, antes do gerador.
- **Q8:** placar numa data passada, com o que se sabia nela. Medida na Fatia 4b; entra na meta de p95.
- **Nova ordem:** 0, 1, 1b, 2, 3, 4a, 4b, 5, 7, 8a, 8b. Opcionais ao final: Fatia 6 e detecção de trapaça. "Liga concluída" = todas, exceto essas duas.
- **Descartado:** simulação de regra de pontuação nova sobre o histórico.

## Mudança de rota (10/11/2026)
- **ADR-007** (escopo): substitui o ADR-006 e descarta o ADR-003.
- **ADR-008** (modelo de eventos): substitui o ADR-005.
- **ADR-009** (identidade, ordem e tempo), **ADR-010** (seed de exemplo e reset) e **ADR-011** (testes e CI): novos.
- ADR-001 e ADR-002 já eram históricos desde o ADR-006.
- Saem do projeto: autenticação, multi-tenancy, hospedagem, deploy, convites, papéis, e-mail, LGPD, recorrência, escala, níveis, títulos e dashboard como produto.

## Fatia 0 — Retomada (concluída)

- [x] Docker Desktop (WSL 2) instalado e testado
- [x] Postgres 18 subindo pelo compose (healthy) e visível no DBeaver
- [x] `.env` fora do git e `.env.example` criado
- [x] Spring conectado ao Postgres (datasource lendo o `.env` via `spring.config.import`)
- [x] Flyway aplicando `V1__init.sql`, Hibernate em `validate`, `open-in-view: false`
- [x] ADR-004 — Flyway como único dono do schema

## Próximo passo concreto
Resolver as decisões abertas abaixo e então modelar a **V2** da Fatia 1: família, membro, tarefa e a tabela `evento`, com o trigger que recusa `UPDATE` e `DELETE`. Em paralelo, trocar o `contextLoads` por Testcontainers e criar o workflow do CI.

## Decisões abertas antes da Fatia 1
| Decisão | Contexto |
|---|---|
| Valores do catálogo de exemplo | Proposta abaixo, aguardando a confirmação do Rafa. Os nomes dos membros seguem genéricos (ADR-010) |
| Colunas exatas da V2 | Definidas na migration, respeitando o ADR-008 |

## Catálogo da família de exemplo (proposta)
Tarefas da casa do Rafa (apartamento de cerca de 100 m²). Régua: 1 ponto ≈ 1 minuto de trabalho, mais 5 se for desagradável e mais 5 se exigir esforço físico. Os pontos são um número digitado; não há fórmula no sistema.

| Tarefa | Pontos |
|---|---|
| Aspirar banheiro | 5 |
| Arrumar quarto 1 | 8 |
| Arrumar quarto 2 | 8 |
| Arrumar suíte | 15 |
| Aspirar varanda | 10 |
| Aspirar quarto 1 | 10 |
| Aspirar quarto 2 | 10 |
| Aspirar suíte | 30 |
| Aspirar sala e cozinha | 55 |
| Passear com o cachorro | 60 |
| Lavar roupa na máquina | 10 |
| Estender roupas | 20 |
| Passar roupa | 50 |
| Lavar louça | 25 |
| Limpar fogão | 25 |
| Cozinhar | 45 |
| Lavar banheiro | 35 |
| Lavar varanda | 40 |
| Trocar o pinico do cachorro | 10 |
| Limpar o pinico do cachorro | 10 |

Relações pedidas: quartos 1 e 2 mais a suíte (50) valem menos que sala e cozinha (55); passear com o cachorro (60) vale mais que aspirar sala e cozinha; limpar o fogão é igual a lavar a louça; limpar o pinico é igual a trocar. Passear tem valor fixo, sem considerar a duração.

Notas do Rafa (10/11/2026): quem clonar não precisa de distinção entre "quarto 1" e "quarto 2"; o catálogo de exemplo pode ter variedade além da casa dele (por exemplo, banheiro × lavabo, lavar roupa à mão). Na casa dele, arrumar e aspirar o quarto costumam ser feitos juntos, então "arrumar quarto" pode ser fundido com "aspirar quarto" (a decidir ao montar o seed). O "aspirar sala e cozinha" valendo mais que os três quartos é a regra confirmada.

## Pendências técnicas (não são decisões)
- O `Dockerfile` placeholder continua fora do git. Não tem uso no escopo atual.
- O serviço `back-spring` do compose não recebe as variáveis do banco e a URL do datasource aponta para `localhost`. Sem uso por enquanto, já que a aplicação roda fora do container.

## Backlog de decisões adiadas
| Decisão | Contexto | Revisitar na |
|---|---|---|
| Onde vive a projeção e como é atualizada | Em memória ou em tabela; síncrona na transação ou assíncrona | Fatia 2 |
| Alcance do estorno | Só conclusões ou qualquer evento (uma penalidade aplicada por engano também precisa de correção) | Fatia 2 |
| Instante do estorno | Herda o `ocorreu_em` do original (corrige o passado) ou usa o instante em que foi feito | Fatia 2 |
| Proporção de estornos e registros tardios | Quanto o gerador produz de cada um, e com que atraso | Fatia 3 |
| Gerador pelo domínio × direto no banco | Pelo domínio respeita as regras, mas demora para 10 milhões; direto é rápido, mas pode gerar o que o domínio recusaria | Fatia 3 |
| Como medir | `pgbench` com scripts × harness em Java; como separar cache frio e quente | Fatia 4a |
| Métrica do ranking da Q7 | Pontos por membro, pontos por atividade ou atividades por membro. A consulta pode calcular as três | Fatia 4b |
| Chave e granularidade da partição | Por mês sobre o instante do evento, o efeito na chave primária e o redesenho do índice único do estorno | Fatia 6 (opcional) |
| Estratégia de snapshot | Por quantidade de eventos ou por período, respeitando a armadilha da ordem de confirmação (ADR-009) | Fatia 8a |
| Conversão de eventos antigos | Como a leitura converte v1 em v2 | Fatia 8b |

## Fora do escopo (ideias registradas)
- **Tarefas configuráveis** (Rafa, 10/11/2026): periodicidade escolhida pelo usuário (diária, semanal), horário fixo ou não, responsável opcional, e redirecionamento ao concluir com motivo e possível penalidade. Seria a recorrência e a escala do ADR-003. O motivo da penalidade já cabe no `payload` do evento.
- Detecção de trapaça: opcional, ao final.
- Fechamento de período, fuso e início da semana por família, conversão de pontos em mesada.
- Autenticação, convites, papéis, e-mail, contas de crianças e LGPD, exclusão de conta: saíram com o ADR-007.
