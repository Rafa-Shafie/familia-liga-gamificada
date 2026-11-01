# ADR-004: Flyway como Único Dono do Schema do Banco

## Status
Aceito

## Contexto
O projeto usa duas ferramentas capazes de alterar a estrutura do banco de dados:

- **Flyway:** executa scripts SQL versionados (`V1__...sql`, `V2__...sql`) em ordem,
  uma única vez cada, e registra o que já foi aplicado na tabela
  `flyway_schema_history`. Funciona como um controle de versão da estrutura do banco.
- **Hibernate** (implementação do JPA usada pelo Spring): seu papel principal é
  traduzir objetos Java em operações SQL (`save` vira `INSERT`, por exemplo). Como
  recurso adicional, pode criar e alterar tabelas automaticamente a partir das
  classes Java, comportamento controlado por `spring.jpa.hibernate.ddl-auto`.

Os dados do sistema têm valor real: o histórico de pontos serve de base para a
mesada. Além disso, o banco será recriado em outra máquina quando o sistema migrar
do PC para o Raspberry Pi. A estrutura precisa ser previsível, rastreável e
reproduzível.

## Alternativas Consideradas

**Hibernate gerencia o schema (`ddl-auto: update`):** Rápido no início, sem
escrever SQL. Descartado porque as alterações acontecem de forma invisível, sem
histórico no repositório. O Hibernate não remove nem renomeia colunas (cria novas e
deixa as antigas para trás), e o resultado pode variar entre máquinas. As opções
`create` e `create-drop` apagariam os dados a cada inicialização, o que é
inaceitável com dados reais.

**Flyway e Hibernate alterando o schema juntos:** Descartado. Dois donos da mesma
estrutura fazem o banco real divergir do que as migrations descrevem, e o histórico
do Flyway deixa de ser confiável.

**Flyway como dono, Hibernate em `ddl-auto: none`:** Seguro, mas o Hibernate não
verifica nada. Um erro de mapeamento (coluna esquecida numa migration, por exemplo)
só apareceria durante o uso, quando a consulta afetada rodasse.

## Decisão
O Flyway é o único responsável por criar e alterar a estrutura do banco. Toda
mudança de schema é feita por uma nova migration em `src/main/resources/db/migration/`.

O Hibernate é configurado com `spring.jpa.hibernate.ddl-auto: validate`: não altera
nada, apenas confere na inicialização se as tabelas correspondem às entidades Java,
e impede a aplicação de subir se houver divergência.

## Consequências

### Positivas
- Toda mudança de estrutura fica versionada no git, com histórico e revisão
- Banco reproduzível em qualquer máquina (PC, Raspberry Pi, máquina de quem clonar)
- Erros de mapeamento são detectados na inicialização, não durante o uso pela família
- Uma única fonte de verdade para o schema

### Negativas
- Toda mudança de estrutura exige escrever SQL à mão
- Migrations já aplicadas são imutáveis (o Flyway valida checksums): corrigir um
  erro exige uma nova migration, nunca editar a antiga
- O `validate` confere tabelas, colunas e tipos, mas não garante que índices e
  constraints estejam como esperado; isso continua sob responsabilidade das migrations
