# ADR-010: Seed de Exemplo e Reset

## Status
Aceito (10/11/2026)

## Contexto
Quem clonar o repositório deve encontrar uma família de exemplo com tarefas
prontas, poder alterá-la e criar outras, sem afetar a cópia de ninguém. Como
cada clone sobe o próprio banco, em um volume próprio, as alterações já são
isoladas por construção.

Há dois tipos de dado inicial, com naturezas diferentes:

- **Exemplo:** pequeno (uma família, quatro membros, algumas tarefas), para
  quem clonar abrir e ver o domínio
- **Volume:** milhões de eventos sintéticos, que existem para medição (ADR-007)

Os eventos do exemplo também precisam de atenção: se tiverem datas fixas, o
placar da semana fica vazio poucos dias depois do clone.

## Alternativas Consideradas

**Eventos do exemplo em SQL, com datas fixas:** Simples, mas o placar fica
vazio com o passar do tempo.

**Seed por perfil do Spring (executado na subida), em vez de migration:**
Permitiria gerar eventos relativos à data atual, mas esconde dados fora do
histórico do Flyway e exige uma verificação para não duplicar a carga.

**Volume numa migration:** Descartado. Seria um arquivo enorme no
repositório público e atrasaria toda subida do banco.

## Decisão
- A estrutura do exemplo (a família com os membros `PAPAI`, `MAMAE`, `FILHA`
  e `FILHO` e um catálogo de tarefas) fica numa migration do Flyway,
  determinística e sem datas
- Os eventos do exemplo e o volume são gerados por um comando, com tamanho e
  semente configuráveis. Os do exemplo são gerados relativos à data atual
- O volume nunca entra numa migration
- Para voltar ao estado de exemplo: `docker compose down -v` e subir de novo.
  É o caminho porque a tabela de eventos recusa `DELETE` (ADR-008)
- Os nomes do exemplo são genéricos; os gerados sinteticamente saem de listas
  fixas, sem dado pessoal

## Consequências

### Positivas
- Quem clona tem o exemplo funcionando logo depois de subir o banco
- Alterar o exemplo não afeta nenhuma outra cópia
- A geração do volume é reproduzível, porque a semente é fixa

### Negativas
- O reset apaga também o volume gerado, que precisa ser gerado de novo (e
  isso demora nos volumes maiores)
- Uma migration aplicada não pode ser editada (checksum do Flyway); quem
  quiser mudar o exemplo faz isso pelos dados, não pelo arquivo
