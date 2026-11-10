# ADR-009: Identidade, Ordem e Tempo

## Status
Aceito (10/11/2026)

## Contexto
A tabela de eventos (ADR-008) terá dezenas de milhões de linhas. O replay
precisa de uma ordem total estável, e as consultas agrupam por períodos
(dia, semana, mês, ano) que dependem de fuso horário.

Como não há autenticação nem isolamento entre famílias (ADR-007), o fato de
identificadores sequenciais serem adivinháveis deixou de ser um risco a
mitigar.

## Alternativas Consideradas

**UUID aleatório (v4):** Mais difícil de adivinhar, mas ocupa 16 bytes contra 8
e, por ser aleatório, espalha as inserções pelo índice e prejudica a
localidade. Ruim para uma tabela de dezenas de milhões de linhas.

**UUID v7:** O PostgreSQL 18 o gera nativamente (`uuidv7()`) e ele é ordenado
por tempo, o que resolve a localidade. Mas ainda ocupa 16 bytes e não dá uma
ordem estrita entre inserções simultâneas. Sem o benefício de segurança, não
compensa.

**Guardar uma coluna de data local:** Facilitaria agrupar por dia, mas seria
dado derivado armazenado sem medição que o justifique (ADR-008).

## Decisão
**Identificadores.** `bigint generated always as identity` em todas as
tabelas.

**Ordem do replay.** O replay ordena pelo `id` do evento. Há uma armadilha
conhecida: o `identity` reserva o número quando o `INSERT` começa, não quando
a transação é confirmada. Uma transação com `id` 100 pode confirmar depois de
uma com `id` 101, e um leitor incremental que lê "tudo com `id` maior que o
último visto" pula o 100 para sempre. Por isso, projeções e rollups
incrementais não se baseiam no último `id` visto: recalculam janelas de tempo
(por exemplo, o dia ou a semana afetados). Os snapshots (Fatia 8a) devem
respeitar a mesma regra. O replay completo roda sobre dados já confirmados.

**Tempo.**
- O instante do evento é gravado em `timestamptz`
- A conversão para o horário de São Paulo acontece na consulta, com
  `AT TIME ZONE 'America/Sao_Paulo'`. O fuso é fixo para todo o sistema
- Os filtros de período são escritos como intervalos sobre o instante, e a
  conversão de fuso fica apenas no agrupamento, para o índice continuar útil
- O Hibernate é configurado em UTC

## Consequências

### Positivas
- Identificadores pequenos e inserções com boa localidade no índice
- Ordem total simples para o replay
- Uma única representação do tempo, sem dado derivado armazenado

### Negativas
- Identificadores sequenciais são previsíveis. Aceito, porque não há
  isolamento de segurança a proteger (ADR-007)
- A ordem por `id` não coincide exatamente com a ordem de confirmação, o que
  restringe o desenho de projeções incrementais
- Agrupar por período exige a conversão de fuso em cada consulta, com custo
  que a Fatia 5 vai medir
- A chave primária terá de incluir a coluna de partição quando a tabela for
  particionada (Fatia 6), o que exige uma migration que altera a chave
