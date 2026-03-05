# Documentação Técnica - Investigador de Corrupção Política Brasil

## Índice

1. [Visão Geral](#visão-geral)
2. [Instalação e Configuração](#instalação-e-configuração)
3. [Estrutura do Projeto](#estrutura-do-projeto)
4. [APIs Públicas Integradas](#apis-públicas-integradas)
5. [Banco de Dados](#banco-de-dados)
6. [Coletores de Dados](#coletores-de-dados)
7. [Motor de Detecção de Padrões](#motor-de-detecção-de-padrões)
8. [Interface Web](#interface-web)
9. [Procedimentos tRPC](#procedimentos-trpc)
10. [Exemplos de Uso](#exemplos-de-uso)

## Visão Geral

O Investigador de Corrupção Política Brasil é uma ferramenta de análise de dados públicos que cruza informações de múltiplas fontes governamentais para identificar padrões suspeitos de corrupção, conflitos de interesse e irregularidades administrativas.

### Características Principais

- **Integração com APIs Públicas**: Coleta dados do Portal da Transparência, TSE, Compras.gov.br e outros
- **Análise de Grafos**: Visualiza conexões entre pessoas, empresas e contratos usando Neo4j
- **Detecção Automática de Padrões**: Identifica funcionários fantasmas, duplo vínculo público-privado e contratos suspeitos
- **Interface Web Intuitiva**: Dashboard para busca, análise e geração de relatórios
- **Relatórios em PDF**: Exporta investigações com evidências e conexões

## Instalação e Configuração

### Pré-requisitos

- Node.js 18+
- MySQL 8.0+
- Neo4j 4.4+ (opcional, para análise de grafos)
- pnpm 10.0+

### Passos de Instalação

1. **Clonar o repositório**
   ```bash
   git clone <repository-url>
   cd br-corruption-investigator
   ```

2. **Instalar dependências**
   ```bash
   pnpm install
   ```

3. **Configurar banco de dados**
   ```bash
   # Criar arquivo .env com DATABASE_URL
   echo "DATABASE_URL=mysql://user:password@localhost:3306/corruption_db" > .env
   
   # Executar migrações
   pnpm db:push
   ```

4. **Configurar APIs Públicas**
   - Portal da Transparência: Cadastrar email em http://portaldatransparencia.gov.br/api-de-dados/cadastrar-email
   - Adicionar token ao arquivo `.env`: `TRANSPARENCY_API_TOKEN=seu_token`

5. **Iniciar servidor de desenvolvimento**
   ```bash
   pnpm dev
   ```

## Estrutura do Projeto

```
br-corruption-investigator/
├── client/                    # Frontend React
│   ├── src/
│   │   ├── pages/            # Páginas da aplicação
│   │   │   ├── Home.tsx
│   │   │   └── Dashboard.tsx
│   │   ├── components/       # Componentes reutilizáveis
│   │   ├── lib/              # Utilitários e configurações
│   │   └── App.tsx           # Componente principal
│   └── public/               # Arquivos estáticos
├── server/                    # Backend Node.js/Express
│   ├── collectors/           # Coletores de dados
│   │   ├── transparencyCollector.ts
│   │   └── tseCollector.ts
│   ├── analysis/             # Análise de dados
│   │   └── patternDetector.ts
│   ├── routers/              # Rotas tRPC
│   │   └── investigation.ts
│   ├── db.ts                 # Utilitários de banco de dados
│   └── routers.ts            # Roteador principal
├── drizzle/                  # Schema e migrações
│   └── schema.ts
├── shared/                   # Código compartilhado
├── ARCHITECTURE.md           # Documentação de arquitetura
├── API_RESEARCH.md          # Pesquisa de APIs
└── package.json             # Dependências do projeto
```

## APIs Públicas Integradas

### 1. Portal da Transparência

**URL Base**: `https://api.portaldatransparencia.gov.br`

**Autenticação**: Token (obtido via cadastro de email)

**Endpoints Principais**:
- `GET /api-de-dados/servidores` - Servidores públicos federais
- `GET /api-de-dados/contratos` - Contratos públicos
- `GET /api-de-dados/licitacoes` - Licitações
- `GET /api-de-dados/peps` - Pessoas Expostas Politicamente
- `GET /api-de-dados/viagens-por-cpf` - Viagens a serviço

**Limite de Requisições**: 90 req/min (6:00-23:59), 300 req/min (00:00-5:59)

### 2. TSE (Tribunal Superior Eleitoral)

**URL Base**: `https://dadosabertos.tse.jus.br`

**Formato**: Arquivos CSV/TXT

**Datasets**:
- Candidatos (CPF, nome, cargo, partido, bens declarados)
- Doações de campanha (doador, valor, candidato)
- Resultados eleitorais

### 3. Compras.gov.br

**URL Base**: `https://dadosabertos.compras.gov.br`

**Endpoints**:
- `GET /v3/public/compras` - Compras públicas
- `GET /v3/public/fornecedores` - Fornecedores
- `GET /v3/public/licitacoes` - Licitações

### 4. Dados Abertos (dados.gov.br)

**Datasets Relevantes**:
- RAIS (Relação Anual de Informações Sociais)
- CEIS (Cadastro Nacional de Empresas Inidôneas e Suspensas)
- CNEP (Cadastro Nacional de Empresas Punidas)
- CEPIM (Entidades Privadas sem Fins Lucrativos Impedidas)

## Banco de Dados

### Schema MySQL

#### Tabela: public_servants
Armazena informações de servidores públicos federais.

```sql
CREATE TABLE public_servants (
  id INT PRIMARY KEY AUTO_INCREMENT,
  cpf VARCHAR(11) UNIQUE NOT NULL,
  name VARCHAR(255) NOT NULL,
  organ VARCHAR(255),
  position VARCHAR(255),
  salary DECIMAL(12, 2),
  admission_date DATE,
  data_source VARCHAR(50),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

#### Tabela: companies
Armazena informações de empresas fornecedoras.

```sql
CREATE TABLE companies (
  id INT PRIMARY KEY AUTO_INCREMENT,
  cnpj VARCHAR(14) UNIQUE NOT NULL,
  legal_name VARCHAR(255) NOT NULL,
  trade_name VARCHAR(255),
  sector VARCHAR(100),
  founded_date DATE,
  data_source VARCHAR(50),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

#### Tabela: contracts
Armazena informações de contratos públicos.

```sql
CREATE TABLE contracts (
  id INT PRIMARY KEY AUTO_INCREMENT,
  contract_number VARCHAR(50) UNIQUE,
  company_id INT,
  organ VARCHAR(255),
  contract_value DECIMAL(15, 2),
  contract_date DATE,
  contract_type VARCHAR(100),
  status VARCHAR(50),
  data_source VARCHAR(50),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (company_id) REFERENCES companies(id)
);
```

#### Tabela: candidates
Armazena informações de candidatos eleitorais.

```sql
CREATE TABLE candidates (
  id INT PRIMARY KEY AUTO_INCREMENT,
  cpf VARCHAR(11) UNIQUE NOT NULL,
  name VARCHAR(255) NOT NULL,
  party VARCHAR(100),
  position VARCHAR(100),
  state VARCHAR(2),
  election_year INT,
  declared_assets DECIMAL(15, 2),
  data_source VARCHAR(50),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

#### Tabela: donations
Armazena informações de doações de campanha.

```sql
CREATE TABLE donations (
  id INT PRIMARY KEY AUTO_INCREMENT,
  donor_cpf_cnpj VARCHAR(14),
  donor_name VARCHAR(255),
  candidate_id INT,
  donation_amount DECIMAL(15, 2),
  donation_date DATE,
  election_year INT,
  data_source VARCHAR(50),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (candidate_id) REFERENCES candidates(id)
);
```

#### Tabela: alerts
Armazena alertas de irregularidades detectadas.

```sql
CREATE TABLE alerts (
  id INT PRIMARY KEY AUTO_INCREMENT,
  alert_type VARCHAR(100) NOT NULL,
  description TEXT,
  severity VARCHAR(20),
  entity_cpf_cnpj VARCHAR(14),
  related_entities TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  resolved BOOLEAN DEFAULT FALSE
);
```

## Coletores de Dados

### TransparencyCollector

Coleta dados do Portal da Transparência.

```typescript
import { TransparencyCollector } from "./server/collectors/transparencyCollector";

const collector = new TransparencyCollector({
  apiToken: "seu_token_aqui"
});

// Coletar servidores públicos
await collector.collectPublicServants(5000);

// Coletar contratos
await collector.collectContracts(5000);

// Coletar PEPs
await collector.collectPEPs();
```

### TSECollector

Coleta dados do TSE (Tribunal Superior Eleitoral).

```typescript
import { TSECollector } from "./server/collectors/tseCollector";

const collector = new TSECollector({
  electionYear: 2024
});

// Coletar candidatos
await collector.collectCandidates();

// Coletar doações
await collector.collectDonations();
```

## Motor de Detecção de Padrões

### PatternDetector

Identifica padrões suspeitos nos dados.

```typescript
import { PatternDetector } from "./server/analysis/patternDetector";

const detector = new PatternDetector();

// Detectar funcionários fantasmas
await detector.detectGhostEmployees();

// Detectar contratos com valores anormais
await detector.detectAnomalousContracts();

// Detectar bens declarados anormais
await detector.detectAnomalousAssets();

// Executar todas as detecções
await detector.runAllDetections();
```

### Tipos de Alertas

- **ghost-employee**: Funcionário com remuneração zero ou muito baixa
- **anomalous-contract**: Contrato com valor anormalmente alto
- **anomalous-assets**: Candidato com bens declarados anormalmente altos
- **dual-binding**: Pessoa com duplo vínculo público-privado

## Interface Web

### Dashboard

A página principal do sistema exibe:

- **Estatísticas Gerais**: Total de servidores, empresas, contratos, candidatos e alertas
- **Barra de Busca**: Busca por CPF, CNPJ ou nome
- **Alertas Recentes**: Últimas irregularidades detectadas
- **Ações Rápidas**: Acesso rápido a funcionalidades principais

### Componentes Principais

- **SearchBar**: Interface de busca com filtros
- **StatisticsCard**: Exibe estatísticas do sistema
- **AlertsList**: Lista de alertas com filtros
- **SearchResults**: Resultados de busca por entidade

## Procedimentos tRPC

### investigation.search

Busca por CPF, CNPJ ou nome.

```typescript
const results = await trpc.investigation.search.query({
  query: "12345678901",
  type: "cpf" // "cpf" | "cnpj" | "all"
});
```

### investigation.getServantDetails

Obtém detalhes de um servidor público.

```typescript
const details = await trpc.investigation.getServantDetails.query({
  cpf: "12345678901"
});
```

### investigation.getCompanyDetails

Obtém detalhes de uma empresa.

```typescript
const details = await trpc.investigation.getCompanyDetails.query({
  cnpj: "12345678901234"
});
```

### investigation.listAlerts

Lista alertas com filtros.

```typescript
const alerts = await trpc.investigation.listAlerts.query({
  alertType: "ghost-employee",
  severity: "high",
  limit: 50,
  offset: 0
});
```

### investigation.getStatistics

Obtém estatísticas gerais do sistema.

```typescript
const stats = await trpc.investigation.getStatistics.query();
```

### investigation.getRecentAlerts

Obtém alertas recentes.

```typescript
const alerts = await trpc.investigation.getRecentAlerts.query({
  limit: 10
});
```

## Exemplos de Uso

### Exemplo 1: Buscar Servidor Público

```typescript
// Frontend
const { data } = trpc.investigation.search.useQuery({
  query: "12345678901",
  type: "cpf"
});

// Exibir resultados
if (data?.servants.length > 0) {
  data.servants.forEach(servant => {
    console.log(`${servant.name} - ${servant.organ}`);
  });
}
```

### Exemplo 2: Investigar Empresa

```typescript
// Frontend
const { data } = trpc.investigation.getCompanyDetails.useQuery({
  cnpj: "12345678901234"
});

// Analisar contratos
if (data?.contracts) {
  const totalValue = data.contracts.reduce((sum, c) => 
    sum + Number(c.contractValue || 0), 0
  );
  console.log(`Total de contratos: R$ ${totalValue}`);
}
```

### Exemplo 3: Monitorar Alertas

```typescript
// Frontend
const { data: alerts } = trpc.investigation.listAlerts.useQuery({
  severity: "high",
  limit: 20
});

// Filtrar alertas críticos
const criticalAlerts = alerts?.alerts.filter(a => 
  a.severity === "high"
) || [];
```

## Próximos Passos

1. **Integração com Neo4j**: Implementar análise de grafos para visualizar conexões
2. **Geração de Relatórios**: Criar exportação em PDF com evidências
3. **Machine Learning**: Treinar modelos para detecção de fraude
4. **API Pública**: Expor endpoints para uso externo
5. **Visualizações Avançadas**: Gráficos e mapas interativos

## Suporte e Contribuições

Para reportar bugs, sugerir melhorias ou contribuir com código, entre em contato através do repositório do projeto.

## Licença

Este projeto está sob licença MIT. Veja o arquivo LICENSE para detalhes.
