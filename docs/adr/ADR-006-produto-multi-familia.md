# ADR-006: Produto Multi-Família

## Status
Aceito (09/11/2026). Substitui o ADR-001 e o ADR-002.

## Contexto
A Liga Familiar Gamificada nasceu como o sistema de uma família só: rodando no
PC de casa, sem login, com os quatro membros fixos num Enum (ADR-001 e
ADR-002). A motivação era medir as tarefas de cada membro como base para a
mesada.

O objetivo do projeto mudou. Ele passa a ser um projeto de prática e de
portfólio: quem avaliar o repositório deve ver que autenticação,
multi-tenancy e controle de acesso já foram praticados. A família do autor
continua testando cada fatia, mas não deve usar o sistema no dia a dia, então
deixa de ser o motivo do projeto.

A mudança acontece no fim da Fatia 0. Nenhuma tabela de domínio existe ainda,
então o custo de mudar é praticamente só de documentação.

## Alternativas Consideradas

**Manter o sistema de uma família só:** O mais simples, e continua resolvendo
o problema original. Descartado porque não pratica autenticação, isolamento
entre clientes nem controle de acesso, que passaram a ser o objetivo.

**Família única com login:** Pratica autenticação, mas não multi-tenancy: com
uma família só, não há dados de outra família para proteger. Descartado.

**Produto multi-família com hospedagem pública:** Mostraria o sistema rodando
na internet, mas um recrutador avalia pelo repositório, e o autor já tem um
portfólio hospedado publicamente. Somaria domínio, HTTPS, banco de produção,
backup e um host sempre ligado (exigido pelo `@Scheduled`), sem nenhum usuário
real. Descartado por ora.

## Decisão
O projeto passa a ser um produto multi-família:

- Qualquer pessoa cria uma conta, faz login, cria uma família, adiciona ou
  convida pessoas e cadastra tarefas e rotinas.
- Cada família enxerga apenas os próprios dados. O isolamento é tratado em ADRs
  próprios: a forma de armazenar, junto com a primeira migration de domínio;
  onde o filtro é aplicado e como é testado, junto com o login.
- Os membros deixam de ser um Enum e passam a ser registros que pertencem a
  uma família.
- A autenticação usa Spring Security. O mecanismo (sessão, token etc.) é
  decidido na fatia de cadastro e login.
- Os papéis dentro da família (por exemplo, quem administra e quem só conclui
  tarefas) entram na fatia de convites.
- Não há hospedagem pública. O projeto fica **pronto para deploy**: sobe com
  um comando via Docker Compose, tem configuração por ambiente e integração
  contínua. Quem quiser avaliar clona o repositório e roda localmente.
- A família do autor é a família de exemplo dos dados iniciais e testa cada
  fatia.

Continuam valendo: o Flyway como dono do schema (ADR-004), o histórico de
pontos com um registro por conclusão (ADR-005) e o modelo proposto de
template, escala e instância (ADR-003).

## Consequências

### Positivas
- O projeto pratica autenticação, multi-tenancy e controle de acesso
- O schema nasce multi-família desde a primeira migration de domínio, sem
  precisar ser refeito depois
- A infraestrutura de desenvolvimento da Fatia 0 (Docker, PostgreSQL, Flyway)
  continua igual
- Sem custo de hospedagem nem operação de um ambiente público

### Negativas
- O escopo antes da parte gamificada cresce: cadastro, login, convites,
  e-mail e papéis. Aceito, porque faz parte do aprendizado
- Sem usuários reais, as decisões perdem o feedback do uso diário. Mitigado
  pelos testes da família a cada fatia
- Um erro no isolamento expõe os dados de uma família para outra. Por isso o
  isolamento tem ADRs próprios e testes automatizados
- Convites e recuperação de senha trazem uma dependência nova: o envio de
  e-mail
- A LGPD passa a importar: dados de crianças exigem o consentimento dos
  responsáveis, e a exclusão de uma conta conflita com o histórico imutável do
  ADR-005. Os dois pontos estão no backlog de decisões adiadas
- Os nomes da família do autor ficam no repositório público, como dados de
  exemplo
- O sistema nunca roda em produção de verdade: problemas que só aparecem com
  carga ou uso real não são exercitados
