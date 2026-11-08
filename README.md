# RecursosApi

API de gerenciamento de recursos (salas, equipamentos etc.) em **ASP.NET Core (.NET 10)**, com front-end em **React + Vite**. Projeto de estudo do curso de Análise e Desenvolvimento de Sistemas.

## O que tem no projeto

- CRUD de recursos com paginação, filtro por nome e validação (erros em Problem Details)
- Autenticação com **ASP.NET Identity + JWT**, com roles `Usuario` e `Administrador`
- Cadastro e login de usuários, com bloqueio após 5 tentativas erradas
- Fila de análises processada em segundo plano (`BackgroundService`)
- Atualização em tempo real com **SignalR**
- Banco **SQLite** com **Entity Framework Core** e migrations
- Testes com xUnit
- Swagger em ambiente de desenvolvimento

## Tecnologias

| Camada | Tecnologias |
|---|---|
| API | C#, ASP.NET Core (Minimal APIs + Controllers), EF Core, SQLite, Identity, JWT, SignalR, Swagger |
| Front-end | React, Vite, `@microsoft/signalr` |
| Testes | xUnit, `WebApplicationFactory` |

## Principais endpoints

| Método | Rota | Descrição |
|---|---|---|
| POST | `/auth/login` | Login, devolve o JWT |
| POST | `/auth/cadastro` | Cria usuário com role `Usuario` |
| GET | `/auth/eu` | Dados do usuário autenticado |
| GET | `/recursos` | Lista com paginação e filtro (`nomeContem`, `pagina`, `tamanho`) |
| GET | `/recursos/{id}` | Busca um recurso |
| POST / PUT / DELETE | `/recursos` | Cria, atualiza e remove |
| GET | `/recursos/{id}/recomendacao` | Recomendação para o recurso |
| POST / GET | `/recursos/{id}/analise` | Enfileira e consulta a análise |
| GET | `/fila/status` | Estado da fila de análises |
| Hub | `/hubs/analises` | SignalR (tempo real) |

Exemplos de requisições em `RecursosApi/analises.http` e `RecursosApi/recursos-auth.http` (extensão REST Client do VS Code).

## Como rodar

### Pré-requisitos

- [.NET 10 SDK](https://dotnet.microsoft.com/download) (`dotnet --version` deve mostrar 10.x)
- [Node.js LTS](https://nodejs.org/)

### 1. Instalar dependências

```bash
dotnet restore
npm install
cd recursos-frontend && npm install && cd ..
```

### 2. Configurar os segredos (User Secrets)

Os segredos **não ficam no repositório**. Rode dentro da pasta `RecursosApi`:

```powershell
cd RecursosApi
dotnet user-secrets init

# chave JWT aleatória de 32 bytes
$bytes = New-Object byte[] 32
[Security.Cryptography.RandomNumberGenerator]::Create().GetBytes($bytes)
dotnet user-secrets set "Jwt:Chave" ([Convert]::ToBase64String($bytes))

# senhas dos usuários de teste (8+ caracteres, maiúscula, minúscula, número e símbolo)
dotnet user-secrets set "SeedSenhas:Ana" "<senha-da-ana>"
dotnet user-secrets set "SeedSenhas:Admin" "<senha-do-admin>"
```

Em Linux/macOS, gere a chave com `openssl rand -base64 32`.

### 3. Criar o banco

```bash
dotnet tool install --global dotnet-ef   # se ainda não tiver
cd RecursosApi
dotnet ef database update
cd ..
```

As migrations já estão no repositório. O comando cria o `recursos.db` com todas as tabelas.

### 4. Subir API e front-end

Na pasta raiz (onde está o `package.json`):

```bash
npm run dev
```

- API: http://localhost:5137 (Swagger em `/swagger`)
- Front-end: http://localhost:5173

Espere aparecer `Now listening on: http://localhost:5137` antes de abrir o front-end. Os usuários `ana` (Usuario) e `admin` (Administrador) são criados na primeira execução, em Development, com as senhas definidas no passo 2.

## Testes

```bash
dotnet test
```

## Estrutura

```
.
├── RecursosApi/            # API (endpoints, serviços, Identity, SignalR, migrations)
├── RecursosApi.Tests/      # testes xUnit
├── recursos-frontend/      # React + Vite
├── package.json            # script `npm run dev` (API + front juntos)
└── SolucaoDotNet.slnx
```

## Solução de problemas

| Erro | Causa | O que fazer |
|---|---|---|
| `Configure Jwt:Chave` / respostas 500 | chave JWT ausente ou curta | refaça o passo 2 e reinicie |
| `no such table: AspNetRoles` | banco sem migrations aplicadas | passo 3 |
| `Could not find the global property 'UserSecretsId'` | faltou `dotnet user-secrets init` | passo 2 |
| `Could not find a MSBuild project file` | comando rodado fora de `RecursosApi` | `cd RecursosApi` |
| Tela "desconectado" | front abriu antes da API | espere o `Now listening` e aperte F5 |
| `Não foi possível criar ana` | senha fraca no User Secrets | use 8+ caracteres com maiúscula, minúscula, número e símbolo |
