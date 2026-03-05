# Pesquisa de APIs Públicas Brasileiras para Investigação de Corrupção

## 1. Portal da Transparência do Governo Federal

### Documentação
- **URL Base**: https://api.portaldatransparencia.gov.br
- **Swagger**: https://api.portaldatransparencia.gov.br/swagger-ui.html
- **Documentação**: https://portaldatransparencia.gov.br/api-de-dados

### Autenticação
- Requer cadastro de email em: http://portaldatransparencia.gov.br/api-de-dados/cadastrar-email
- Token fornecido por email após cadastro
- Limite de requisições: 90 req/min (6:00-23:59), 300 req/min (00:00-5:59)

### Endpoints Principais
- `GET /api-de-dados/servidores` - Lista todos servidores do Poder Executivo Federal
- `GET /api-de-dados/servidores/{id}` - Servidor específico
- `GET /api-de-dados/servidores/remuneracao` - Remunerações por CPF
- `GET /api-de-dados/servidores/por-orgao` - Servidores agregados por órgão
- `GET /api-de-dados/peps` - Pessoas Expostas Politicamente (PEPs)
- `GET /api-de-dados/contratos` - Todos contratos do Poder Executivo Federal
- `GET /api-de-dados/contratos/cpf-cnpj` - Contratos por CPF/CNPJ do fornecedor
- `GET /api-de-dados/licitacoes` - Todas licitações
- `GET /api-de-dados/licitacoes/participantes` - Participantes de licitações
- `GET /api-de-dados/viagens-por-cpf` - Viagens por CPF
- `GET /api-de-dados/imoveis` - Imóveis funcionais

### Dados Disponíveis
- Servidores públicos federais
- Remunerações e benefícios
- Contratos e fornecedores
- Licitações e participantes
- Viagens a serviço
- Pessoas Expostas Politicamente (PEPs)
- Imóveis funcionais

---

## 2. Portal de Dados Abertos do TSE

### Documentação
- **URL**: https://dadosabertos.tse.jus.br
- **Formato**: Arquivos CSV/TXT (download)

### Datasets Disponíveis
- **Candidatos 2024**: Informações de candidatos, bens, coligações
- **Doações de Campanha**: Doadores, valores, receptores
- **Resultados Eleitorais**: Votos por candidato
- **Filiações Partidárias**: Histórico de filiações

### Estrutura de Dados
- Candidatos: CPF, nome, cargo, partido, estado, bens declarados
- Doações: CPF/CNPJ doador, valor, data, candidato receptor
- Bens: Tipo, descrição, valor, candidato

### Frequência de Atualização
- Diária durante períodos eleitorais
- Histórico completo disponível

---

## 3. API Compras.gov.br

### Documentação
- **URL Base**: https://dadosabertos.compras.gov.br
- **Swagger**: https://dadosabertos.compras.gov.br/swagger-ui/index.html
- **Manual**: https://www.gov.br/compras/pt-br/acesso-a-informacao/manuais/manual-dados-abertos/manual-api-compras.pdf

### Endpoints Principais
- `GET /v3/public/compras` - Compras públicas
- `GET /v3/public/fornecedores` - Fornecedores
- `GET /v3/public/itens` - Itens de compra
- `GET /v3/public/licitacoes` - Licitações

### Dados Disponíveis
- Compras governamentais
- Fornecedores (CNPJ, razão social)
- Valores de contratos
- Modalidades de compra
- Histórico de compras

---

## 4. Dados Abertos Governamentais (dados.gov.br)

### Documentação
- **URL**: https://dados.gov.br
- **API**: https://www.gov.br/conecta/catalogo/apis/api-portal-de-dados-abertos

### Datasets Relevantes
- RAIS (Relação Anual de Informações Sociais)
- CEIS (Cadastro Nacional de Empresas Inidôneas e Suspensas)
- CNEP (Cadastro Nacional de Empresas Punidas)
- CEPIM (Entidades Privadas sem Fins Lucrativos Impedidas)

---

## 5. Tesouro Transparente (Siconfi)

### Documentação
- **URL**: https://www.tesourotransparente.gov.br
- **API Siconfi**: https://www.tesourotransparente.gov.br/consultas/consultas-siconfi/siconfi-api-de-dados-abertos

### Dados Disponíveis
- Execução orçamentária
- Despesas por órgão
- Receitas e transferências

---

## 6. Banco Central do Brasil

### Dados Relevantes
- Pessoas Expostas Politicamente (PEPs)
- Informações de instituições financeiras
- Regulações e compliance

---

## Resumo de Integração

| Fonte | Tipo | Autenticação | Frequência | Dados Principais |
|-------|------|--------------|-----------|-----------------|
| Portal Transparência | API REST | Token | Real-time | Servidores, contratos, licitações, PEPs |
| TSE | CSV Download | Nenhuma | Diária | Candidatos, doações, bens |
| Compras.gov.br | API REST | Nenhuma | Real-time | Compras, fornecedores, licitações |
| dados.gov.br | API/CSV | Nenhuma | Variável | RAIS, CEIS, CNEP, CEPIM |
| Tesouro Transparente | API | Nenhuma | Real-time | Execução orçamentária |

---

## Próximos Passos

1. Implementar coletores para cada API
2. Estruturar dados no MySQL
3. Criar grafos no Neo4j
4. Implementar detecção de padrões
5. Construir interface web
