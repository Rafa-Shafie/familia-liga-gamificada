# ADR-001: Infraestrutura Local vs. Cloud

## Status
Aceito (revisado em 31/10/2026)

## Contexto
Para organizar e gamificar essa divisão de tarefas, foi necessário escolher onde o sistema vai rodar. O sistema é de uso exclusivo dos 4 membros da família e é usado principalmente dentro da própria residência, via Wi-Fi. Eventualmente, é útil registrar uma tarefa fora de casa (ir ao mercado, por exemplo). O custo de infraestrutura é uma restrição não negociável — o projeto deve ter custo zero.

O backend precisa ficar sempre ligado: tarefas recorrentes serão geradas por `@Scheduled`, que só executa num processo contínuo.

## Alternativas Consideradas

**Cloud paga (AWS, Railway):** Resolveria disponibilidade e acesso externo, mas introduziria custo mensal e complexidade desnecessária para um sistema usado por 4 pessoas.

**Vercel:** Gratuita, mas voltada a frontend e funções serverless. Não roda um backend Spring Boot como processo contínuo, o que inviabiliza o `@Scheduled`. Além disso, publicar o app na internet exigiria autenticação, contrariando o ADR-002.

**Hospedagens gratuitas de backend (Render, por exemplo):** Os planos gratuitos suspendem a aplicação após um período sem acessos, o que também impede o `@Scheduled` de executar no horário.

**App em casa com banco gerenciado na nuvem (Supabase, Neon):** Criaria dependência de internet sem nenhum ganho: com o app em casa, se a internet cair, ele não alcança o banco. Os planos gratuitos também pausam projetos inativos.

**Abrir uma porta no roteador para acesso externo:** Exporia o sistema publicamente na internet, exigindo autenticação. Descartado.

**Local (computador da casa) com Tailscale para acesso externo:** Custo zero, sem dependência de serviços externos para o uso em casa. O Tailscale cria uma rede privada gratuita apenas com os aparelhos da família, permitindo o acesso de fora sem expor o sistema na internet.

## Decisão
O sistema rodará localmente, em Docker Compose, com dois containers: PostgreSQL e Spring Boot (servindo o backend e o front estático). O acesso em casa é feito pelo IP local na rede Wi-Fi (`http://192.168.1.X:8080`).

No MVP, o servidor é o computador Windows da casa, que fica permanentemente ligado. Depois, o sistema será migrado para um Raspberry Pi, servidor dedicado de baixo consumo.

O acesso fora de casa será feito via Tailscale, configurado na fatia de deploy.

## Consequências

### Positivas
- Custo zero de infraestrutura
- Uso em casa independente da internet: se a internet cair, a rede local continua funcionando
- Boa performance via rede local
- Acesso externo sem expor o sistema publicamente, mantendo o "sem login" seguro
- Raspberry Pi como servidor dedicado de baixo consumo energético no pós-MVP

### Negativas
- Se faltar energia ou o roteador desligar, o sistema fica indisponível — aceitável dado o escopo
- Reinícios do computador (atualizações do Windows, por exemplo) derrubam o sistema até ele subir de novo — mitigado com inicialização automática do Docker e, depois, pela migração para o Raspberry Pi
- O acesso fora de casa depende da internet e exige o app do Tailscale em cada aparelho da família
- O backup do banco é responsabilidade própria, sem serviço gerenciado
