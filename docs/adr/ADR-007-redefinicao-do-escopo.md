# ADR-007: Redefinição do Escopo — Postgres em Volume e Event Sourcing

## Status
Aceito (10/11/2026). Substitui o ADR-006 e descarta o ADR-003.

## Contexto
O projeto nasceu como um sistema doméstico para uma família (ADR-001 e
ADR-002) e foi redefinido como produto multi-família (ADR-006), com o
objetivo de praticar autenticação e multi-tenancy.

Ao mapear os demais projetos de estudo do autor, ficou claro que
autenticação, controle de acesso, concorrência, idempotência, mensageria
confiável e testes com banco real já têm projeto próprio, e de forma mais
profunda. Mantida a rota do ADR-006, a Liga repetiria esses temas sem ensinar
nada novo, e sem nenhum usuário real para justificá-la.

Dois temas não são cobertos por nenhum outro projeto e cabem naturalmente em
um domínio de tarefas com pontos:

- **A — Postgres com volume:** consultas analíticas (window functions, CTEs,
  `date_trunc` e fuso horário), `EXPLAIN ANALYZE`, índices, particionamento,
  views materializadas e rollups.
- **B — Event sourcing com projeções:** pontos como eventos imutáveis,
  com placar e sequências reconstruídos por replay, snapshots e versionamento
  de eventos.

Os dois se ligam: o replay de milhões de eventos é lento, e é isso que
justifica as otimizações de A. O domínio (tarefas e pontos) passa a ser o
pretexto para aprender os dois.

## Alternativas Consideradas

**Encerrar a Liga na Fatia 0:** Honesto, e foi considerado seriamente.
Descartado porque A e B são aprendizados reais que nenhum outro projeto
cobre, e porque o ambiente da Fatia 0 já está pronto.

**Manter o produto multi-família com autenticação (ADR-006):** Descartado
pela sobreposição com outros projetos.

**Focar em domínio e dados, em testes e qualidade, ou em jobs e
consistência isoladamente:** Cada um já é ensinado de forma mais forte em
outro projeto. A Liga os usa como meio, não como objetivo.

**Terminar o produto doméstico (recorrência, escala, níveis, títulos,
dashboard):** Descartado. Não ensina A nem B, e dependeria de uso real que
não vai existir.

## Decisão
O projeto passa a ter como objetivo **aprender A e B, provando com medição**.

**Escopo.** O sistema comporta várias famílias, com membros e tarefas, sem
autenticação. A família é apenas uma forma de organizar os dados, **sem
garantia de isolamento de segurança**: o identificador da família vem da
requisição. Esse limite é assumido. Sem hospedagem pública e sem deploy; o
CI permanece.

**Domínio mínimo.** Família, membro, tarefa e dois eventos:
`TarefaConcluida` e `PenalidadeAplicada`. Fechamento de período fica fora.

**Prova de A.** Sete consultas fixas, medidas antes e depois de cada técnica:

| # | Consulta |
|---|---|
| Q1 | Placar da semana de uma família |
| Q2 | Ranking dos membros na semana, com posição |
| Q3 | Série mensal dos últimos 12 meses de uma família |
| Q4 | Maior sequência de dias seguidos de um membro |
| Q5 | Total de um membro no ano |
| Q6 | Top 10 famílias do mês, de todas |
| Q7 | Eficiência por família: pontos por membro, pontos por atividade |

Os dados são sintéticos, com semente fixa: várias famílias de tamanhos e
nomes variados, **2 anos** de eventos, nos volumes de 100 mil, 1 milhão e 10
milhões. A medição usa `EXPLAIN (ANALYZE, BUFFERS)`, 30 execuções por
consulta, p50 e p95, cache frio e quente separados, mesma máquina e mesma
configuração, com os relatórios versionados no repositório. Metas: p95 abaixo
de 100 ms nas consultas por família (Q1 a Q5) com 10 milhões de eventos, e
ganho de pelo menos 10× por técnica sobre o baseline ou uma explicação
registrada de por que não houve.

**Prova de B.**
- A projeção incremental é idêntica à reconstruída por replay, e o replay é
  determinístico
- O banco recusa `UPDATE` e `DELETE` na tabela de eventos; correções são
  eventos compensatórios
- Eventos antigos (v1) continuam sendo lidos depois da mudança de formato
- Replay completo e replay a partir de snapshot são medidos nos três volumes

**Fatias.**

| Fatia | Entrega |
|---|---|
| 1 | Registrar conclusão como evento, com testes de integração e CI |
| 1b | Cadastro de famílias, membros e tarefas (criar, consultar e alterar, sem apagar), com API e tela mínima |
| 2 | Penalidade e projeção por replay |
| 3 | Gerador de dados sintéticos |
| 4a | Baseline das agregações simples |
| 4b | Baseline das windows e consultas globais, e do replay |
| 5 | Índices |
| 6 | Particionamento por mês |
| 7 | Rollups com `@Scheduled` e view materializada |
| 8a | Snapshots |
| 8b | Versionamento de eventos |

Fica como opcional, ao final, a detecção de trapaça (rajadas de conclusões).
Benchmarks não rodam no CI; o CI roda apenas testes de correção em volume
pequeno.

**Fora do escopo.** Autenticação, hospedagem e deploy, tela elaborada,
recorrência, escala, transferência de tarefas, níveis, títulos e dashboard
como produto. O `@Scheduled` é usado apenas para rollups.

**Congelamento.** Esta é a quarta redefinição do projeto. O escopo fica
congelado: qualquer mudança exige um novo ADR que justifique o que foi
aprendido de diferente.

## Consequências

### Positivas
- O projeto ensina dois temas que nenhum outro cobre, e o critério de pronto
  é mensurável
- A infraestrutura da Fatia 0 (Docker, PostgreSQL, Flyway) é reaproveitada
- Os ADRs anteriores ficam como registro da trajetória e do motivo de cada
  mudança

### Negativas
- O domínio é simples de propósito; o valor está nas medições, não no
  produto
- Dois temas grandes num só projeto; o fatiamento permite parar depois da
  Fatia 5 sem perder o conjunto
- Dados sintéticos refletem hipóteses sobre o uso, não uso real; as
  distribuições são decisão do gerador
- Medições dependem da máquina; todo relatório registra o hardware
- Sem isolamento entre famílias, o sistema não serve como produto multi-
  usuário real
- Os ADRs 002 e 005 citam nomes reais da família do autor; permanecem como
  registro histórico
