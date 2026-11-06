# Liga Familiar Gamificada

Gerenciador de tarefas domésticas gamificado, estilo RPG, feito para uso exclusivo da minha família. Cada tarefa concluída vale pontos, e esses pontos formam um histórico que mostra o quanto cada membro contribui em casa ao longo do tempo. Esse histórico serve de base para a mesada.

O sistema roda em casa, na rede Wi-Fi, com custo zero de infraestrutura. O código é público para quem quiser estudar ou rodar localmente.

> **Status:** em desenvolvimento. A infraestrutura (Fatia 0) está concluída; a Fatia 1, que registra a conclusão de tarefas, está em andamento.

## Stack

| Camada | Tecnologia |
|---|---|
| Linguagem | Java 21 |
| Framework | Spring Boot 4 (API REST) |
| Banco de dados | PostgreSQL 18 + Spring Data JPA |
| Migrations | Flyway |
| Containers | Docker Compose |
| Frontend (planejado) | HTML5 + Tailwind CSS + Alpine.js |
| Hospedagem (planejada) | PC de casa → Raspberry Pi, com acesso externo via Tailscale |

## Decisões de arquitetura

As decisões relevantes são registradas como ADRs (Architecture Decision Records), com o contexto, as alternativas consideradas e as consequências de cada escolha:

| ADR | Decisão |
|---|---|
| [ADR-001](docs/adr/ADR-001-infraestrutura-local.md) | Infraestrutura local em vez de cloud, com Tailscale para acesso externo |
| [ADR-002](docs/adr/ADR-002-autenticacao.md) | Sem autenticação no MVP |
| [ADR-003](docs/adr/ADR-003-modelo-template-instancia.md) | Modelo de domínio com template, escala e instância (proposto) |
| [ADR-004](docs/adr/ADR-004-flyway-dono-do-schema.md) | Flyway como único dono do schema do banco |
| [ADR-005](docs/adr/ADR-005-historico-de-pontos.md) | Histórico de pontos com um registro por conclusão |

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
| `docker compose down -v` | ⚠️ Remove os containers **e apaga os dados** do banco |

## Estrutura do projeto

```
.
├── docs/
│   ├── adr/                     # Architecture Decision Records
│   └── estado-do-projeto.md     # Fatia atual, pendências e backlog de decisões
├── src/main/
│   ├── java/                    # Código da aplicação
│   └── resources/
│       ├── application.yaml     # Configuração do Spring
│       └── db/migration/        # Migrations do Flyway (V1__, V2__, ...)
├── docker-compose.yaml          # Banco de dados (e, futuramente, a aplicação)
├── .env.example                 # Modelo das variáveis de ambiente
└── pom.xml                      # Dependências (Maven)
```

## Roadmap

O projeto avança em **fatias verticais**: cada fatia é uma funcionalidade pequena que atravessa o sistema inteiro (banco, domínio, API e tela) e fica funcionando de ponta a ponta antes de a próxima começar.

- [x] **Fatia 0 — Retomada:** Docker, PostgreSQL, Spring conectado ao banco, Flyway, ADRs
- [ ] **Fatia 1 — Registrar conclusão:** um membro registra uma tarefa concluída e vê os pontos do dia
- [ ] **Fatia 2 — Catálogo de tarefas:** cadastro e edição de tarefas e pontos
- [ ] **Fatia 3 — Dashboard:** pontos por dia, semana, mês e ano, por membro
- [ ] **Fatia 4 — Escala e recorrência:** tarefas recorrentes geradas automaticamente
- [ ] **Fatia 5 — Níveis e progressão**
- [ ] **Fatia 6 — Títulos:** campeão da semana, do mês e do ano
- [ ] **Fatia 7 — Deploy definitivo:** Raspberry Pi + Tailscale
- [ ] **Fatia 8 — Portfólio**

O andamento detalhado fica em [docs/estado-do-projeto.md](docs/estado-do-projeto.md).

## Licença

Distribuído sob a licença MIT. Veja [LICENSE](LICENSE).
