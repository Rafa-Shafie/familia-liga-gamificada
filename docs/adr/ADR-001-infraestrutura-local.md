# ADR-001: Infraestrutura Local vs. Cloud

## Status
Aceito

## Contexto
Para organizar e gamificar essa divisão de tarefas, foi necessário escolher onde o sistema vai rodar. O sistema será acessado apenas pelos membros da família, dentro da própria residência, via Wi-Fi. O custo de infraestrutura é uma restrição não negociável — o projeto deve ter custo zero.

## Alternativas Consideradas

**Cloud (AWS, Railway, Render):** Resolveria o problema de disponibilidade e acesso externo, mas introduziria custo mensal, dependência de internet externa e complexidade desnecessária para um sistema usado por 4 pessoas dentro de casa.

**Local (computador da casa):** Custo zero, sem dependência de serviços externos, acesso via IP local na rede Wi-Fi. Complexidade de setup compatível com o escopo do projeto.

## Decisão
O sistema rodará localmente no computador da casa, acessado via `http://192.168.1.X:8080` na rede Wi-Fi. No MVP, o servidor será o computador Windows atual. Após o MVP, será migrado para um Raspberry Pi para rodar de forma contínua sem ocupar o computador principal.

## Consequências

### Positivas
- Custo zero de infraestrutura
- Sem dependência de serviços externos
- Boa performance via rede local
- Raspberry Pi como servidor dedicado de baixo consumo energético no pós-MVP

### Negativas
- Se a rede Wi-Fi cair, o sistema fica indisponível — aceitável dado o escopo
- Se o computador desligar, o sistema cai — mitigado pela migração para Raspberry Pi
- Não acessível fora de casa — não é um requisito do projeto