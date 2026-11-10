# ADR-008: Modelo de Eventos — Eventos Imutáveis como Fonte da Verdade

## Status
Aceito (10/11/2026). Substitui o ADR-005.

## Contexto
O ADR-007 define como objetivo aprender event sourcing com projeções (B) e
Postgres com volume (A). O ADR-005 decidia um registro por conclusão, com os
pontos congelados no momento e nenhum total armazenado.

Parte dessa decisão continua verdadeira e parte foi superada. O ADR-005 se
apoiava em uma mesada que não será paga, em cerca de 7 mil linhas por ano e
em totais sempre calculados por consulta. O cenário agora tem milhões de
eventos, em que recalcular a cada consulta é justamente o problema a medir, e
em que projeções armazenadas (placar, rollups) passam a ter um papel.

## Alternativas Consideradas

**Manter o ADR-005 e acrescentar eventos por cima:** Deixaria dois ADRs em
vigor com regras conflitantes sobre armazenar totais. Descartado.

**Estado mutável (um contador de pontos por membro):** Sem histórico não há
replay, recorte por período nem correção auditável. Já descartado no ADR-005
e continua descartado.

**Totais por dia ou por mês como fonte da verdade:** Seriam duas fontes que
podem divergir, e não respondem a janelas móveis. Só são permitidos como
projeção descartável (ver Decisão).

**Uma tabela por tipo de evento, ou por agregado:** Multiplicaria índices,
particionamento (Fatia 6) e migrações, e perderia a ordem global usada pelo
replay. Com dois tipos de evento, separar não traz ganho.

**Todo o evento em JSONB:** É o mais flexível, mas as consultas deste projeto
somam e agrupam milhões de linhas por membro, tarefa e período. Esses campos
em colunas tipadas são bem mais simples de indexar e otimizar.

## Decisão
**Fonte da verdade.** Eventos imutáveis. Há dois tipos: `TarefaConcluida` e
`PenalidadeAplicada`.

**Princípios mantidos do ADR-005.**
- Os pontos ficam congelados no evento. Alterar os pontos de uma tarefa vale
  apenas para eventos futuros
- O passado é imutável. Um erro se corrige com um evento compensatório, nunca
  editando ou apagando
- Nada é resetado na virada do ano. Ver um ano anterior é um filtro

**Dado derivado.** A regra padrão continua sendo "dado derivado não se
armazena, se consulta". Uma projeção só pode ser armazenada se cumprir as três
condições: (1) é reconstruível por replay a partir dos eventos; (2) foi
justificada por medição; (3) nunca é a fonte da verdade.

**Estrutura.** Uma única tabela `evento`, com o fluxo organizado por família.
Colunas tipadas para o que as consultas somam e agrupam (identificador,
família, tipo, versão, instante, membro, tarefa e pontos) e um `payload`
JSONB para o que só o replay lê (por exemplo, o motivo de uma penalidade). A
lista exata de colunas é definida na migration da Fatia 1.

**Pontos com sinal.** O campo `pontos` é negativo na penalidade, para a soma
não precisar distinguir o tipo. **Não há piso**: o saldo de um membro, em
qualquer recorte, pode ser negativo. Um piso por recorte quebraria a
aditividade (a soma das semanas deixaria de ser igual ao mês), e um piso no
total exigiria olhar o saldo atual antes de aceitar cada evento.

**Sem verificação de versão por fluxo.** Nenhuma regra do domínio depende do
estado atual para aceitar um evento: uma tarefa pode ser concluída várias
vezes por dia e os pontos não têm piso. Por isso o registro de um evento é
incondicional, sem controle otimista de concorrência.

**Imutabilidade garantida pelo banco.** A tabela recusa `UPDATE` e `DELETE`
por um trigger, com um teste que tenta violar a regra. A limpeza total, usada
por testes e pelo gerador de dados, é feita com `TRUNCATE`, que não dispara
triggers de linha.

**Versionamento.** Cada evento guarda a versão do seu formato. A leitura
converte versões antigas para o modelo atual (Fatia 8b), e eventos v1 ficam
versionados no repositório como fixture de teste.

## Consequências

### Positivas
- Qualquer recorte de período pode ser calculado, inclusive os ainda não
  previstos
- O passado é imutável e auditável; erros têm correção explícita
- Projeções são descartáveis: se uma estiver errada, reconstrói-se por replay
- Novos tipos de evento (por exemplo, recorrência ou redirecionamento de
  tarefas, se um dia entrarem) podem ser acrescentados sem alterar os
  existentes

### Negativas
- Totais precisam ser recalculados ou mantidos por projeções, que exigem
  cuidado com consistência
- O sinal em `pontos` acopla o evento à interpretação do placar
- A tabela única mistura tipos de evento, o que deixa colunas nulas em alguns
  tipos (uma penalidade pode não ter tarefa)
- Sem piso, placares podem ser negativos
- Corrigir um erro exige um evento compensatório, nunca uma edição
