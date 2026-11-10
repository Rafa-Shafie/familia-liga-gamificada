# ADR-012: Justificativa do Event Sourcing por Medição, Estorno e Nova Ordem das Fatias

## Status
Aceito (10/11/2026). Altera o ADR-007 (prova de B, consultas e ordem das
fatias) e complementa o ADR-008 (novo tipo de evento e segunda linha do tempo).

## Contexto
O domínio da Liga, tarefas com pontos, não pede event sourcing. O ADR-008 já
trata os pontos como um razão append-only, e uma tabela imutável com `SUM`
resolve o placar. Event sourcing está no projeto porque o autor quer aprendê-lo
(ADR-007). Isso é legítimo, mas, sem mais nada, a justificativa fica em "usei
porque queria praticar".

Foram avaliadas três exigências de domínio para justificar o mecanismo:

- **Estorno de uma conclusão marcada por engano.** É natural no domínio e
  reforça a correção compensatória que o ADR-008 já prevê.
- **Placar numa data passada.** Com uma única linha do tempo, é um filtro por
  data que qualquer tabela faz. Só ganha sentido com duas linhas do tempo:
  quando o fato aconteceu e quando o sistema soube dele.
- **Regra de pontuação nova aplicada ao histórico.** Contradiz os pontos
  congelados do ADR-008, a menos que seja uma simulação. Mesmo como simulação,
  uma regra "a tarefa X passa a valer N" sai de um `GROUP BY`, sem replay. Seria
  cara e artificial.

A conclusão é que nenhuma exigência de domínio torna o event sourcing
indispensável aqui, e afirmar o contrário não se sustenta. A pergunta "event
sourcing vale a pena neste domínio?" pode, porém, ser respondida com medição,
que é o que o projeto já se propõe a fazer.

## Alternativas Consideradas

**Manter o escopo como está:** a justificativa de B continuaria apenas no
desejo de aprender. Descartado.

**Adotar as três exigências:** o estorno e as duas linhas do tempo entram; a
simulação de regra nova é descartada pelos motivos acima.

**Criar regras de domínio que dependam do histórico (bônus por sequência, teto
diário):** tornariam o replay necessário, mas inventariam regras para
justificar o mecanismo, contra o congelamento do ADR-007. Descartado.

## Decisão

**A Liga é um exercício de aprendizado.** O event sourcing não é justificado
pelo domínio, e sim medido contra a alternativa mais simples.

**Prova de B por comparação.** O placar e as sequências são calculados de várias
formas sobre a mesma tabela de eventos, e cada forma é medida nos três volumes:

| Forma | Fatia |
|---|---|
| SQL direto sobre o log (`SUM`, window functions) | 4a, 4b e 5 |
| Projeção incremental | 2 |
| Replay completo | 2 e 4b |
| Rollup e view materializada | 7 |
| Replay a partir de snapshot | 8a |

O relatório final responde quando cada forma compensa. O resultado não é
presumido: se o SQL direto vencer em tudo, esse é o resultado registrado. A
prova de B do ADR-007 continua valendo.

**Estorno.** Uma conclusão marcada por engano é corrigida por um evento
`ConclusaoEstornada`, que referencia o evento original e tem `pontos` igual ao
oposto. Nada é apagado ou editado. Estornar duas vezes o mesmo evento é recusado
por um índice único parcial na referência ao original, sem leitura de saldo e sem
controle otimista. Assim continua válido o que o ADR-008 afirma: nenhuma regra
depende do estado atual para aceitar um evento. O estorno também afeta as
consultas que não são somas: um dia cuja única conclusão foi estornada não conta
na sequência (Q4), e as atividades da Q7 descontam os estornos. Entra na Fatia 2.

**Duas linhas do tempo.** Além de `ocorreu_em` (quando o fato aconteceu), cada
evento guarda `registrado_em` (quando o sistema soube dele), ambos em
`timestamptz` (ADR-009). A coluna entra na migration da Fatia 2, antes do gerador
da Fatia 3; a V2 da Fatia 1 não a inclui, porque essa fatia não precisa dela.
Consultar "como era conhecido" usa `registrado_em`, não o `id`, pela armadilha da
ordem de confirmação (ADR-009).

**Q8.** Uma nova consulta fixa: *placar de uma família numa data passada, com o
que se sabia naquela data*, comparado ao placar do mesmo período corrigido hoje.
É medida na Fatia 4b e entra na meta de p95 abaixo de 100 ms das consultas por
família (Q1 a Q5 e Q8). A Q5 permanece. O gerador (Fatia 3) produz estornos e
registros tardios, deterministicamente pela semente fixa; as proporções são
decididas na Fatia 3.

**Nova ordem das fatias.** 0, 1, 1b, 2, 3, 4a, 4b, 5, 7, 8a, 8b. O
particionamento (Fatia 6) e a detecção de trapaça passam a ser opcionais, ao
final. "Liga concluída" significa todas as fatias, exceto essas duas.

**Fronteira com o Projeto A (livro-razão).** A Liga ensina o mecanismo de event
sourcing e a medição. A garantia fica com o Projeto A: partidas dobradas,
idempotência e saga. Dois critérios sustentam a fronteira: os pontos da Liga não
têm invariante de conservação (nascem sem conta de origem), e a Liga evita de
propósito invariantes que dependam de saldo (sem piso, ADR-008). O índice único
do estorno é a única proteção desse tipo, e fica restrito à restrição do banco.

**Pendente para a Fatia 2:** se o estorno vale só para conclusões ou para qualquer
evento (uma penalidade aplicada por engano também precisa de correção), e se o
estorno herda o `ocorreu_em` do evento original ou usa o instante em que foi feito.

**O escopo continua congelado** fora destas mudanças.

## Quando eu NÃO usaria event sourcing
- Quando só o estado atual interessa e o histórico não será consultado, corrigido
  nem reinterpretado
- Quando o domínio é um CRUD com regras simples e pouca necessidade de auditoria
- Quando as leituras são quase todas do estado atual e a consistência imediata
  importa mais que reconstruir o passado
- Quando a equipe não pode arcar com projeções, versionamento de eventos e
  correção por compensação
- Quando uma tabela append-only com `SUM` já atende, que é a situação da Liga sem
  o objetivo de aprendizado

## Consequências

### Positivas
- A justificativa de B passa a ser verificável por números, não por argumento
- A seção "quando eu não usaria" é sustentada pela própria medição
- Q4 e Q7 ficam mais ricas, porque precisam respeitar estornos
- A Q8 mostra o que uma tabela editável não consegue responder
- A segunda linha do tempo custa uma coluna, porque entra antes do volume

### Negativas
- O trabalho cresce: estorno, Q8 e um gerador com estornos e registros tardios
- O índice único do estorno não pode ser mantido como está se a Fatia 6 for feita,
  porque em tabela particionada todo índice único inclui a chave de partição
- O resultado da comparação pode mostrar que o event sourcing não compensa em
  nenhuma consulta; isso é aceito como resultado
- Esta é a quinta alteração de escopo; o congelamento volta a valer fora dela
