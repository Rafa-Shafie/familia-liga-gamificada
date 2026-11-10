# ADR-005: Histórico de Pontos — Um Registro por Conclusão

## Status
Substituído pelo [ADR-008](ADR-008-modelo-de-eventos.md) em 10/11/2026.

Os princípios que continuam verdadeiros (pontos congelados no registro, passado
imutável, nada resetado na virada do ano) foram reescritos no ADR-008. A regra
"nenhum total é armazenado" foi trocada por uma exceção medida. O texto abaixo
é mantido como registro histórico.

*Antes: Aceito.*

## Contexto
Os pontos são a base da mesada dos membros da família, então a métrica precisa
ser confiável e auditável. O dashboard planejado exige recortes variados:
últimos 7 e 30 dias, últimos 6 meses e 1 ano, totais por mês do ano, por semana
do mês e por dia da semana, com um seletor de ano.

Três fatores adicionais moldam a decisão:

- O valor de pontos de uma tarefa pode mudar ao longo do tempo (ajuste de
  esforço, correção de um valor mal calibrado).
- Não há autenticação (ADR-002): um registro indevido precisa poder ser
  identificado e removido individualmente.
- O volume é pequeno: cerca de 20 conclusões por dia, ou ~7 mil registros
  por ano.

## Alternativas Consideradas

**Contador acumulado no membro (campo `pontos`):** A forma mais simples, mas
cada conclusão apenas soma no total e desaparece. Nenhuma pergunta sobre
período ("quanto a Manu fez em março?") pode ser respondida. Descartado.

**Totais armazenados por mês ou por dia:** Totais mensais não respondem a
janelas móveis: "últimos 30 dias" a partir de 23/10 começa em 23/09, e não há
como saber quanto do total de setembro caiu nesse intervalo. Totais diários
resolvem as janelas, mas perdem quais tarefas compõem o dia, o que impede
remover uma única conclusão fraudulenta. Guardar os totais junto com os
registros individuais criaria duas fontes de verdade que podem divergir.
Descartado.

**Reset anual com "snapshot" do dashboard:** Proposto para evitar acúmulo de
volume. Descartado porque o problema não existe (~7 mil linhas por ano são
irrelevantes para o PostgreSQL) e a solução traria custos reais: perda da
comparação entre anos, dois modos de dashboard (dados vivos e foto do passado)
e dependência de um job na virada do ano, que falharia se o servidor estivesse
desligado.

**Pontos lidos do cadastro da tarefa, sem cópia no registro:** Manteria o
banco totalmente normalizado, mas qualquer mudança no valor da tarefa
alteraria retroativamente todo o histórico, incluindo mesadas já pagas e
campeões já anunciados. Descartado.

## Decisão
Cada conclusão de tarefa é gravada como um registro próprio, contendo:

- o membro que executou;
- a tarefa concluída;
- a data e hora da conclusão;
- os **pontos da tarefa no momento da conclusão**, copiados do cadastro.

Nenhum total é armazenado. Pontos por dia, semana, mês e ano são calculados
por consulta sobre esses registros, com os períodos calculados no fuso
`America/Sao_Paulo`. Nada é resetado na virada do ano: ver um ano anterior é
apenas um filtro por data.

Alterar os pontos de uma tarefa no cadastro vale apenas para conclusões
futuras.

Princípio adotado para o projeto: **dado derivado não se armazena, se consulta.**

## Consequências

### Positivas
- Qualquer recorte de período pode ser calculado, inclusive recortes ainda não
  previstos, sem mudança de estrutura
- Cada conclusão é auditável e pode ser removida individualmente
- O passado é imutável: mesadas pagas e campeões anunciados não mudam
- Uma única fonte de verdade para a pontuação
- Comparação entre anos disponível para sempre

### Negativas
- Os totais são recalculados a cada consulta do dashboard. Irrelevante no
  volume previsto; se um dia deixar de ser, uma view materializada resolve sem
  mudar o modelo
- Corrigir os pontos de conclusões passadas exige uma ação explícita e
  deliberada, nunca acontece como efeito colateral de editar o cadastro
- A cópia dos pontos em cada registro é uma desnormalização intencional. O
  cadastro da tarefa continua normalizado (ADR-003); o que se copia é um fato
  histórico, que por natureza não deve mudar
