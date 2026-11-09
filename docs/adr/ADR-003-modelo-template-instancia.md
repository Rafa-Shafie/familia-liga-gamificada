# ADR-003: Modelo de Domínio — Template, Escala e Instância

## Status
Proposto (revisado em 09/11/2026)

A implementação foi adiada para a fatia de escala e recorrência. Por ser a
parte mais complexa e menos validada do domínio, este modelo será reavaliado
com os testes da família antes de ser implementado. Até lá, a Fatia 1 registra
conclusões avulsas (ADR-005); como elas se relacionam com `InstanciaTarefa`
será decidido na fatia de recorrência.

Com o produto multi-família (ADR-006), os membros deixam de ser um Enum e
passam a ser registros de uma família; as três estruturas abaixo também
pertencem a uma família.

## Contexto
O sistema precisa gerenciar tarefas recorrentes com regras complexas para os
membros de uma família. Os exemplos vêm da família do autor: tarefas diárias obrigatórias com dono fixo (arrumar
a cama, colocar louça na máquina), tarefas diárias de pool aberto sem dono
fixo (passear com o Bob, dar comida e água), tarefas semanais (aspirar
cômodos) e mensais (ir ao mercado). Além disso, um membro pode assumir a
tarefa de outro — e os pontos devem ir para quem executou, não para quem
foi originalmente atribuído.

## Alternativas Consideradas

**Tabela única de tarefas:** Criar uma única estrutura com todas as
informações — nome, pontos, responsável e data — em cada registro.
Descartado pois misturaria o cadastro da tarefa (nome, categoria,
recorrência, responsável) com cada ocorrência dela, repetindo essas
informações em centenas de registros por ano. Renomear uma tarefa ou mudar
sua recorrência exigiria atualizar todos esses registros. Viola os
princípios de normalização de banco de dados relacional.

**Criação manual pelo usuário:** O usuário cria os cards diariamente pela
interface. Descartado pois dependeria de memória humana — membros poderiam
simplesmente não criar o card de tarefas que não fizeram, tornando a
auditoria impossível.

## Decisão
O domínio foi separado em 3 estruturas:

- **ModeloTarefa:** Define o template da tarefa com suas informações base
  — nome, pontos, categoria e recorrência (DIARIA, SEMANAL, MENSAL).
  Criado uma única vez. Todas as instâncias herdam essas informações.

- **RegraEscala:** Referencia o ModeloTarefa e define o responsável e os
  dias de execução. Tarefas do tipo POOL_ABERTO não têm responsável fixo
  — qualquer membro pode assumir.

- **InstanciaTarefa:** O card real gerado automaticamente pelo `@Scheduled`
  conforme a recorrência de cada tarefa. Contém dois campos de membro:
    - `membroAtribuido` — quem recebeu o card originalmente
    - `membroExecutor` — quem realmente concluiu

  Quando diferentes, a tarefa foi transferida — status `TRANSFERIDA` —
  e os pontos vão para o `membroExecutor`.

## Consequências

### Positivas
- Alterações no ModeloTarefa refletem automaticamente em todas as
  instâncias futuras. Os pontos de conclusões já registradas não mudam
  (ADR-005)
- Geração automática garante auditoria completa — card não concluído
  significa tarefa não feita
- Transferência de responsabilidade registrada e auditável
- Pontos sempre vão para quem realmente executou a tarefa
- Cadastro de tarefas normalizado. A única cópia intencional é o valor de
  pontos no momento da conclusão, que é um fato histórico (ADR-005)

### Negativas
- Maior complexidade de modelagem em relação a uma tabela única
- Requer o `@Scheduled` rodando corretamente — se o servidor estiver
  desligado no horário programado, os cards não são gerados