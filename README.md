# Liga Familiar Gamificada

Gerenciador de tarefas domésticas gamificado em que cada tarefa concluída vale pontos. O domínio é simples de propósito: ele é o pretexto para estudar duas coisas, com medição:

- **Postgres com volume:** consultas analíticas (window functions, CTEs, `date_trunc` e fuso horário), `EXPLAIN ANALYZE`, índices, particionamento por mês, views materializadas e rollups.
- **Event sourcing com projeções:** os pontos são eventos imutáveis; placar e sequências são projeções reconstruídas por replay, com snapshots e versionamento de eventos.

As duas se ligam: o replay de milhões de eventos é lento, e é isso que justifica as otimizações. O projeto gera dados sintéticos (2 anos, de 100 mil a 10 milhões de eventos) e prova, com medição antes e depois, que as consultas ficaram mais rápidas.

Não há usuários, login nem hospedagem. O sistema comporta várias famílias, mas a família é só uma forma de organizar os dados, sem isolamento de segurança. Para conhecer, basta clonar e rodar localmente.

> **Status:** em desenvolvimento. A infraestrutura (Fatia 0) está concluída e as decisões de modelagem estão registradas nos ADRs 007 a 011. A Fatia 1 é a próxima.

## Stack

| Camada | Tecnologia |
|---|---|
| Linguagem | Java 21 |
| Framework | Spring Boot 4 (API REST) |
| Banco de dados | PostgreSQL 18 + Spring Data JPA (SQL nativo nas consultas analíticas) |
| Migrations | Flyway |
| Testes (planejado) | JUnit + Testcontainers com PostgreSQL 18 |
| CI (planejado) | GitHub Actions |
| Containers | Docker Compose |
| Frontend (planejado) | HTML5 + Alpine.js + Tailwind CSS, tela mínima |

## Decisões de arquitetura

As decisões relevantes são registradas como ADRs (Architecture Decision Records), com o contexto, as alternativas consideradas e as consequências de cada escolha. Os ADRs substituídos continuam no repositório como registro histórico: mostram como o projeto mudou de rota e por quê.

| ADR | Decisão | Status |
|---|---|---|
| [ADR-007](docs/adr/ADR-007-redefinicao-do-escopo.md) | Redefinição do escopo: Postgres em volume e event sourcing | Aceito |
| [ADR-008](docs/adr/ADR-008-modelo-de-eventos.md) | Eventos imutáveis como fonte da verdade | Aceito |
| [ADR-009](docs/adr/ADR-009-identidade-ordem-e-tempo.md) | IDs, ordem do replay e tratamento de fuso | Aceito |
| [ADR-010](docs/adr/ADR-010-seed-de-exemplo-e-reset.md) | Família de exemplo, geração de volume e reset | Aceito |
| [ADR-011](docs/adr/ADR-011-testes-e-ci.md) | Testcontainers com Postgres real e CI | Aceito |
| [ADR-004](docs/adr/ADR-004-flyway-dono-do-schema.md) | Flyway como único dono do schema do banco | Aceito |
| [ADR-006](docs/adr/ADR-006-produto-multi-familia.md) | Produto multi-família com autenticação | Substituído pelo ADR-007 |
| [ADR-005](docs/adr/ADR-005-historico-de-pontos.md) | Histórico de pontos com um registro por conclusão | Substituído pelo ADR-008 |
| [ADR-003](docs/adr/ADR-003-modelo-template-instancia.md) | Modelo de template, escala e instância | Descartado pelo ADR-007 |
| [ADR-002](docs/adr/ADR-002-autenticacao.md) | Sem autenticação, membros num Enum | Substituído pelo ADR-006 |
| [ADR-001](docs/adr/ADR-001-infraestrutura-local.md) | Infraestrutura local com Tailscale | Substituído pelo ADR-006 |

## Como rodar localmente

### Pré-requisitos
- [Java 21](https://adoptium.net/)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (no Windows, com WSL 2)

O Maven não precisa ser instalado: o projeto usa o Maven Wrapper (`mvnw`).

### Passo a passo

1. **Clone o repositório**
   ```bash
   git clone https://github.com/Rafa-Shafie/familia-liga-gamificada.git
   cd familia-liga-gamificada
   ```

2. **Crie o arquivo de variáveis de ambiente** a partir do modelo e preencha os valores:
   ```bash
   cp .env.example .env            # Linux/macOS
   Copy-Item .env.example .env     # Windows (PowerShell)
   ```
   ```properties
   POSTGRES_USER=liga
   POSTGRES_PASSWORD=uma-senha-qualquer
   POSTGRES_DB=liga_familiar
   ```

3. **Suba o banco de dados**
   ```bash
   docker compose up -d db-postgres
   docker compose ps               # aguarde o status "healthy"
   ```

4. **Rode a aplicação**
   ```bash
   ./mvnw spring-boot:run          # Linux/macOS
   .\mvnw spring-boot:run          # Windows (PowerShell)
   ```
   Na inicialização, o Flyway aplica as migrations automaticamente. A aplicação fica disponível em `http://localhost:8080`.

### Comandos úteis

| Comando | O que faz |
|---|---|
| `docker compose ps` | Mostra os containers e o status de cada um |
| `docker compose logs -f db-postgres` | Acompanha os logs do banco |
| `docker compose stop` | Para os containers, mantendo tudo |
| `docker compose down` | Remove os containers; os dados continuam no volume |
| `docker compose down -v` | ⚠️ Remove os containers **e apaga os dados** do banco. É também o caminho para voltar ao estado de exemplo (ADR-010) |

## Estrutura do projeto

```text
.
├── docs/
│   ├── adr/                     # Architecture Decision Records
│   └── estado-do-projeto.md     # Fatia atual, decisões abertas e backlog
├── src/main/
│   ├── java/                    # Código da aplicação
│   └── resources/
│       ├── application.yaml     # Configuração do Spring
│       └── db/migration/        # Migrations do Flyway (V1__, V2__, ...)
├── docker-compose.yaml          # Banco de dados
├── .env.example                 # Modelo das variáveis de ambiente
└── pom.xml                      # Dependências (Maven)
```

## Roadmap

O projeto avança em **fatias verticais**: cada fatia é uma funcionalidade pequena que atravessa o sistema e fica funcionando, testada, antes de a próxima começar. O escopo está congelado no ADR-007.

- [x] **Fatia 0 — Retomada:** Docker, PostgreSQL, Spring conectado ao banco, Flyway, ADRs
- [ ] **Fatia 1 — Registrar conclusão como evento:** família, membro, tarefa e tabela de eventos; placar da semana; Testcontainers e CI
- [ ] **Fatia 2 — Penalidade e projeção por replay:** o placar reconstruível a partir dos eventos
- [ ] **Fatia 3 — Gerador de dados sintéticos:** 2 anos de eventos, de 100 mil a 10 milhões
- [ ] **Fatia 4a — Baseline das agregações simples:** harness de medição e `EXPLAIN ANALYZE`
- [ ] **Fatia 4b — Baseline das windows e consultas globais:** e o tempo de replay completo
- [ ] **Fatia 5 — Índices:** antes e depois, por consulta
- [ ] **Fatia 6 — Particionamento por mês:** com prova de partition pruning
- [ ] **Fatia 7 — Rollups e view materializada:** atualizados por `@Scheduled`, com prova de idempotência
- [ ] **Fatia 8a — Snapshots:** replay a partir de snapshot
- [ ] **Fatia 8b — Versionamento de eventos:** evolução de v1 para v2

Os relatórios de medição ficarão em `docs/medicoes/`, a partir da Fatia 4a.

O andamento detalhado fica em [docs/estado-do-projeto.md](docs/estado-do-projeto.md).

## Licença

Distribuído sob a licença MIT. Veja [LICENSE](LICENSE).
