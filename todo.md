# Investigador de Corrupção Política Brasil - TODO

## Fase 1: Pesquisa e Arquitetura
- [x] Pesquisar APIs públicas brasileiras
- [x] Documentar endpoints e autenticação
- [x] Definir schema do banco de dados MySQL
- [x] Definir estrutura de grafos Neo4j

## Fase 2: Backend - Coletores de Dados
- [x] Implementar coletor Portal da Transparência (servidores, contratos, licitações, PEPs)
- [x] Implementar coletor TSE (candidatos, doações, bens)
- [ ] Implementar coletor Compras.gov.br
- [ ] Implementar coletor CEIS/CNEP/CEPIM
- [ ] Implementar sistema de cache e atualização de dados
- [ ] Criar testes para coletores

## Fase 3: Backend - Banco de Dados
- [x] Criar tabelas MySQL para servidores
- [x] Criar tabelas MySQL para contratos e licitações
- [x] Criar tabelas MySQL para empresas/fornecedores
- [x] Criar tabelas MySQL para candidatos e doações
- [x] Criar tabelas MySQL para relacionamentos
- [x] Implementar índices e otimizações

## Fase 4: Backend - Grafos Neo4j
- [ ] Configurar conexão Neo4j
- [ ] Criar nós para pessoas (CPF)
- [ ] Criar nós para empresas (CNPJ)
- [ ] Criar nós para contratos
- [ ] Criar relacionamentos entre nós
- [ ] Implementar queries de análise de grafos

## Fase 5: Backend - Detecção de Padrões
- [x] Implementar detecção de duplo vínculo público-privado (estrutura)
- [x] Implementar detecção de funcionários fantasmas
- [x] Implementar detecção de contratos com valores anormais
- [x] Implementar detecção de bens declarados anormais
- [x] Implementar sistema de alertas

## Fase 6: Backend - API tRPC
- [x] Implementar procedimento de busca por CPF/CNPJ
- [x] Implementar procedimento de detalhes de servidor
- [x] Implementar procedimento de detalhes de empresa
- [x] Implementar procedimento de listagem de alertas
- [x] Implementar procedimento de estatísticas

## Fase 7: Frontend - Dashboard
- [x] Criar layout principal do dashboard
- [x] Implementar estatísticas gerais
- [x] Implementar sistema de busca por CPF/CNPJ
- [ ] Criar visualizações de grafos
- [ ] Implementar filtros e busca avançada

## Fase 8: Frontend - Relatórios
- [ ] Implementar geração de relatórios em PDF
- [ ] Incluir evidências e conexões nos relatórios
- [ ] Implementar exportação de dados

## Fase 9: Documentação
- [x] Documentar instalação e configuração
- [x] Documentar uso do sistema
- [x] Criar guia de APIs
- [x] Criar exemplos de investigações
- [x] Documentação técnica completa

## Fase 10: Testes e Validação
- [ ] Testes unitários dos coletores
- [ ] Testes de integração
- [ ] Testes de performance
- [ ] Validação de dados

## Fase 11: Entrega
- [ ] Revisão final
- [ ] Checkpoint final
- [ ] Entrega ao usuário

## Fase 12: Visualização de Grafos
- [x] Instalar Cytoscape.js e dependências
- [x] Criar componente GraphVisualization.tsx
- [x] Adicionar página de visualização de grafos
- [x] Integrar busca com visualização de grafos
- [x] Adicionar filtros e controles de zoom
- [ ] Implementar procedimento tRPC para buscar dados de grafos do banco
