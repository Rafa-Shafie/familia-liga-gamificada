# ADR-002: Decisão de Autenticação no MVP

## Status
Substituído pelo [ADR-006](ADR-006-produto-multi-familia.md) em 09/11/2026.

Com o produto multi-família, entram cadastro, login com Spring Security e
membros como registros de uma família. O texto abaixo é mantido como registro
histórico.

*Antes: Aceito (revisado em 31/10/2026).*

## Contexto
O sistema será usado exclusivamente pelos 4 membros da família, acessado via rede Wi-Fi local e, fora de casa, pela rede privada do Tailscale (ADR-001). A experiência de uso precisa ser simples e sem fricção — especialmente para os membros mais velhos da família. O sistema precisa identificar quem realizou cada tarefa para distribuir pontos corretamente, e esses pontos servem de base para a mesada.

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
- Qualquer pessoa na rede local pode acessar o sistema — aceitável dado que a rede guest é separada; fora de casa, apenas aparelhos da rede Tailscale da família têm acesso
- Sem auditoria de autenticidade — quem conclui a tarefa declara quem foi, sem verificação. Como os pontos definem a mesada, um membro poderia registrar uma tarefa em nome de outro. Risco aceito: as conclusões são conferidas diariamente e um registro indevido pode ser removido individualmente (ADR-005)