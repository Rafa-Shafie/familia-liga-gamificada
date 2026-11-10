# ADR-011: Testes e CI

## Status
Aceito (10/11/2026)

## Contexto
A prova do projeto (ADR-007) depende de testes que quebrem quando algo está
errado: replay igual ao incremental, imutabilidade dos eventos, leitura de
eventos antigos, idempotência dos rollups. O teste atual (`contextLoads`)
depende de o Postgres do Compose estar rodando, o que não é reproduzível.

O projeto usa recursos específicos do PostgreSQL: triggers, funções de data
e de janela, particionamento e views materializadas.

## Alternativas Consideradas

**Banco em memória (H2):** Não tem os recursos acima e se comporta de forma
diferente em fuso e agregação. Testar contra ele daria uma falsa segurança.
Descartado.

**Postgres do Compose nos testes:** Depende de um estado externo (o banco
estar de pé e limpo), e o CI precisaria de um serviço à parte.

**Serviço de Postgres do GitHub Actions:** Funciona no CI, mas deixa o teste
local diferente do teste no CI.

**Rodar benchmarks no CI:** Descartado. Demoram, e os números variam entre
máquinas, o que geraria falhas intermitentes.

## Decisão
- Testes de integração rodam contra um **PostgreSQL 18 real**, via
  Testcontainers, com a mesma imagem do Compose e com as migrations reais
- A regra de projeção é uma função pura (estado e evento resultam em um novo
  estado), testada sem banco. O replay é aplicar essa função aos eventos
- A limpeza entre testes usa `TRUNCATE`, porque a tabela de eventos recusa
  `DELETE` (ADR-008)
- O CI (GitHub Actions) roda `./mvnw verify` a cada push e pull request
- Os benchmarks ficam fora do CI, marcados e excluídos da execução padrão,
  e rodam manualmente. Os relatórios são versionados, com o hardware usado
- O CI roda apenas as provas de correção em volume pequeno: replay igual ao
  incremental, determinismo, imutabilidade, idempotência do rollup e leitura
  de eventos v1
- O `contextLoads` passa a usar Testcontainers

## Consequências

### Positivas
- Os testes são reproduzíveis na máquina do autor, no CI e na de quem clonar
- Testes contra o mesmo banco que roda de verdade
- O CI continua rápido e estável, sem depender de benchmarks

### Negativas
- Rodar os testes exige Docker, mesmo localmente
- Testes com container são mais lentos que testes unitários
- Regressões de desempenho não são detectadas automaticamente; dependem de
  rodar os benchmarks à mão
