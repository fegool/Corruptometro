# Arquitetura do Sistema de Investigação de Corrupção Política

## Visão Geral

O sistema é composto por três camadas principais: coleta de dados, processamento e análise, e apresentação. A arquitetura foi projetada para integrar múltiplas fontes de dados públicos brasileiros, armazená-los de forma estruturada, analisar conexões através de grafos e apresentar descobertas através de uma interface web intuitiva.

## 1. Arquitetura em Camadas

### Camada de Coleta de Dados
A camada de coleta é responsável por extrair dados de múltiplas fontes públicas. Cada fonte possui um coletor específico que implementa a lógica de autenticação, paginação e tratamento de erros. Os coletores funcionam de forma assíncrona e podem ser executados em intervalos regulares ou sob demanda.

As fontes de dados incluem o Portal da Transparência (via API REST com autenticação por token), o TSE (via download de arquivos CSV), o Compras.gov.br (via API REST), e portais de dados abertos como CEIS, CNEP e CEPIM. Cada coletor normaliza os dados para um formato comum antes de armazená-los.

### Camada de Processamento e Análise
Esta camada recebe dados da camada de coleta, realiza validação, desduplicação e enriquecimento. Os dados são armazenados no MySQL para consultas SQL rápidas e também são projetados no Neo4j para análise de grafos.

O motor de detecção de padrões executa algoritmos que identificam anomalias como duplo vínculo público-privado, funcionários fantasmas, contratos com parentes e outros padrões suspeitos. Os alertas gerados são armazenados e podem ser consultados através da interface.

### Camada de Apresentação
A camada de apresentação oferece uma interface web construída com React, permitindo busca por CPF/CNPJ, visualização de grafos, consulta de alertas e geração de relatórios em PDF. A interface comunica-se com o backend através de uma API tRPC.

## 2. Banco de Dados MySQL

### Tabelas Principais

#### Tabela: `public_servants` (Servidores Públicos)
Armazena informações de servidores do Poder Executivo Federal. Cada servidor é identificado por um ID único e um CPF. A tabela inclui nome, órgão, cargo, remuneração, data de admissão e data da última atualização dos dados.

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
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  INDEX idx_cpf (cpf),
  INDEX idx_organ (organ)
);
```

#### Tabela: `companies` (Empresas/Fornecedores)
Armazena informações de empresas que recebem contratos públicos. Cada empresa é identificada por um CNPJ único. A tabela inclui razão social, nome fantasia, setor e histórico de contratos.

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
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  INDEX idx_cnpj (cnpj)
);
```

#### Tabela: `contracts` (Contratos Públicos)
Armazena informações de contratos entre órgãos públicos e fornecedores. Cada contrato é identificado por um ID único e inclui valor, data, órgão contratante e fornecedor.

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
  FOREIGN KEY (company_id) REFERENCES companies(id),
  INDEX idx_company_id (company_id),
  INDEX idx_organ (organ),
  INDEX idx_contract_date (contract_date)
);
```

#### Tabela: `candidates` (Candidatos Eleitorais)
Armazena informações de candidatos nas eleições. Cada candidato é identificado por um CPF único e inclui partido, cargo, estado e bens declarados.

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
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  INDEX idx_cpf (cpf),
  INDEX idx_party (party),
  INDEX idx_state (state)
);
```

#### Tabela: `donations` (Doações de Campanha)
Armazena informações de doações para campanhas eleitorais. Inclui CPF/CNPJ do doador, valor, data e candidato receptor.

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
  FOREIGN KEY (candidate_id) REFERENCES candidates(id),
  INDEX idx_donor (donor_cpf_cnpj),
  INDEX idx_candidate_id (candidate_id),
  INDEX idx_donation_date (donation_date)
);
```

#### Tabela: `alerts` (Alertas de Irregularidades)
Armazena alertas gerados pelo motor de detecção de padrões. Inclui tipo de alerta, descrição, entidades envolvidas e data de geração.

```sql
CREATE TABLE alerts (
  id INT PRIMARY KEY AUTO_INCREMENT,
  alert_type VARCHAR(100) NOT NULL,
  description TEXT,
  severity VARCHAR(20),
  entity_cpf_cnpj VARCHAR(14),
  related_entities TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  resolved BOOLEAN DEFAULT FALSE,
  INDEX idx_alert_type (alert_type),
  INDEX idx_severity (severity),
  INDEX idx_entity (entity_cpf_cnpj)
);
```

#### Tabela: `relationships` (Relacionamentos)
Armazena relacionamentos entre entidades (pessoas, empresas, contratos). Permite análise de conexões e redes.

```sql
CREATE TABLE relationships (
  id INT PRIMARY KEY AUTO_INCREMENT,
  source_type VARCHAR(50),
  source_id VARCHAR(50),
  target_type VARCHAR(50),
  target_id VARCHAR(50),
  relationship_type VARCHAR(100),
  relationship_strength INT DEFAULT 1,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_source (source_type, source_id),
  INDEX idx_target (target_type, target_id),
  INDEX idx_relationship_type (relationship_type)
);
```

## 3. Banco de Dados Neo4j

### Modelo de Grafos

O Neo4j armazena entidades como nós e relacionamentos como arestas, permitindo análise rápida de conexões. O modelo inclui os seguintes tipos de nós:

**Nó: Person (Pessoa)**
Representa uma pessoa identificada por CPF. Propriedades incluem nome, tipo (servidor público, candidato, doador), órgão/partido e data da última atualização.

**Nó: Company (Empresa)**
Representa uma empresa identificada por CNPJ. Propriedades incluem razão social, nome fantasia, setor e data da última atualização.

**Nó: Contract (Contrato)**
Representa um contrato público. Propriedades incluem número do contrato, valor, data e tipo.

**Nó: Alert (Alerta)**
Representa um alerta de irregularidade. Propriedades incluem tipo de alerta, descrição e severidade.

### Tipos de Relacionamentos

Os relacionamentos conectam os nós e representam conexões entre entidades:

- `WORKS_FOR`: Pessoa trabalha para empresa ou órgão
- `OWNS`: Pessoa é proprietária de empresa
- `RECEIVES_CONTRACT`: Empresa recebe contrato de órgão
- `DONATES_TO`: Pessoa/empresa doa para candidato
- `RELATED_TO`: Relacionamento genérico entre entidades
- `TRIGGERS_ALERT`: Relacionamento dispara alerta

## 4. API REST Backend

### Endpoints Principais

**GET /api/search**
Busca por CPF ou CNPJ. Retorna informações da pessoa ou empresa, contratos associados e alertas.

**GET /api/graph/:cpf_cnpj**
Retorna o subgrafo de conexões para uma entidade específica. Usado para visualização de redes.

**GET /api/alerts**
Lista alertas gerados. Suporta filtros por tipo, severidade e data.

**POST /api/collect**
Inicia coleta de dados de uma fonte específica. Requer autenticação de administrador.

**GET /api/statistics**
Retorna estatísticas gerais do sistema (número de servidores, contratos, alertas, etc.).

## 5. Interface Web

### Componentes Principais

**Dashboard**
Página inicial com estatísticas gerais, alertas recentes e acesso rápido às funcionalidades.

**Busca**
Interface de busca por CPF/CNPJ com resultados detalhados e histórico de buscas.

**Visualizador de Grafos**
Exibe grafos interativos mostrando conexões entre entidades. Permite zoom, pan e filtragem.

**Alertas**
Lista de alertas com filtros por tipo, severidade e data. Permite marcar como resolvido.

**Relatórios**
Geração de relatórios em PDF com evidências e conexões encontradas.

## 6. Fluxo de Dados

1. **Coleta**: Coletores extraem dados de APIs públicas e arquivos CSV
2. **Normalização**: Dados são normalizados para formato comum
3. **Armazenamento**: Dados armazenados em MySQL e Neo4j
4. **Análise**: Motor de padrões analisa dados e gera alertas
5. **Apresentação**: Interface web exibe resultados e permite interação

## 7. Detecção de Padrões

O sistema implementa os seguintes algoritmos de detecção:

**Duplo Vínculo Público-Privado**
Identifica pessoas que trabalham simultaneamente em órgãos públicos e empresas privadas, especialmente aquelas que recebem contratos do governo.

**Funcionários Fantasmas**
Detecta pessoas com registros de remuneração muito baixa ou inexistente, ou que aparecem em múltiplos órgãos simultaneamente.

**Contratos com Parentes**
Identifica contratos entre órgãos públicos e empresas cujos proprietários são parentes de servidores públicos ou políticos.

**Padrões de Fraude**
Detecta padrões como múltiplas empresas com mesmo endereço, CPF/CNPJ duplicados, ou valores de contrato anormalmente altos.

## 8. Segurança e Privacidade

O sistema implementa as seguintes medidas de segurança:

- Autenticação de usuários através de OAuth
- Autorização baseada em papéis (admin, usuário)
- Criptografia de dados sensíveis em trânsito (HTTPS)
- Logs de auditoria de todas as operações
- Conformidade com LGPD (Lei Geral de Proteção de Dados)

## 9. Escalabilidade

A arquitetura foi projetada para escalabilidade:

- Coletores podem ser executados em paralelo
- MySQL com índices otimizados para consultas rápidas
- Neo4j para análise de grafos em larga escala
- Cache de resultados frequentes
- Paginação de resultados

## 10. Tecnologias Utilizadas

| Componente | Tecnologia |
|-----------|-----------|
| Frontend | React 19, Tailwind CSS, Wouter |
| Backend | Express.js, tRPC, Node.js |
| Banco de Dados | MySQL, Neo4j |
| Autenticação | OAuth Manus |
| Relatórios | ReportLab/WeasyPrint |
| Visualização de Grafos | Cytoscape.js ou similar |
| Hospedagem | Manus Platform |

Esta arquitetura oferece uma base sólida para investigação jornalística de corrupção política, permitindo análise profunda de dados públicos e detecção de padrões suspeitos.
