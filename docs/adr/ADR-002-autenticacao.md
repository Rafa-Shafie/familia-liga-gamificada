# ADR-002: Decisão de Autenticação no MVP

## Status
Aceito

## Contexto
O sistema será usado exclusivamente pelos 4 membros da família dentro da própria residência, acessado via rede Wi-Fi local. A experiência de uso precisa ser simples e sem fricção — especialmente para os membros mais velhos da família. O sistema precisa identificar quem realizou cada tarefa para distribuir pontos corretamente.

## Alternativas Consideradas

**Spring Security com login e senha:** Adicionaria fricção desnecessária — membros precisariam criar conta, lembrar senha e fazer login toda vez. Descartado por impacto negativo na UX.

**Autenticação via provedor externo (Google OAuth, Face ID):** Reduziria a fricção do login, mas introduziria dependência de serviços externos e complexidade de implementação incompatível com o escopo do MVP.

**Sem autenticação — seleção de membro por Enum:** Ao criar ou concluir uma tarefa, o usuário seleciona o membro responsável via interface. Tarefas sem dono fixo entram como `POOL_ABERTO` e exigem identificação no momento do check-in.

## Decisão
O MVP não terá autenticação. A identificação do membro é feita por seleção explícita na interface — via Enum com os 4 membros da família: `RAFA`, `MANU`, `MAMAE`, `PAPAI`. Autenticação poderá ser adicionada em versão futura como exercício técnico isolado, sem impacto no core do sistema.

## Consequências

### Positivas
- Zero fricção de acesso — abre o app e usa
- Sem dependência de serviços externos
- Desenvolvimento mais rápido no MVP
- Decisão consciente e documentada — reconhece a ausência como escolha técnica, não omissão

### Negativas
- Qualquer pessoa na rede local pode acessar o sistema — aceitável dado que a rede guest é separada e o acesso externo não é um requisito
- Sem auditoria de autenticidade — quem conclui a tarefa declara quem foi, sem verificação