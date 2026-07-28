# ADR-003: Modelo de Domínio — Template, Escala e Instância

## Status
Aceito

## Contexto
O sistema precisa gerenciar tarefas recorrentes com regras complexas para
4 membros da família: tarefas diárias obrigatórias com dono fixo (arrumar
a cama, colocar louça na máquina), tarefas diárias de pool aberto sem dono
fixo (passear com o Bob, dar comida e água), tarefas semanais (aspirar
cômodos) e mensais (ir ao mercado). Além disso, um membro pode assumir a
tarefa de outro — e os pontos devem ir para quem executou, não para quem
foi originalmente atribuído.

## Alternativas Consideradas

**Tabela única de tarefas:** Criar uma única estrutura com todas as
informações — nome, pontos, responsável e data — em cada registro.
Descartado pois geraria dados duplicados em centenas de registros por ano.
Qualquer alteração de pontos ou nome exigiria atualizar todos os registros
históricos. Viola os princípios de normalização de banco de dados relacional.

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
  instâncias futuras
- Geração automática garante auditoria completa — card não concluído
  significa tarefa não feita
- Transferência de responsabilidade registrada e auditável
- Pontos sempre vão para quem realmente executou a tarefa
- Banco de dados normalizado, sem duplicação de dados

### Negativas
- Maior complexidade de modelagem em relação a uma tabela única
- Requer o `@Scheduled` rodando corretamente — se o servidor estiver
  desligado no horário programado, os cards não são gerados