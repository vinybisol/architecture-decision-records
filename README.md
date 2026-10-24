# Architecture Decision Records

Este repositório reúne decisões arquiteturais e técnicas para consulta por pessoas e agentes. Cada ADR registra uma decisão, seu contexto e suas consequências. As decisões podem ser propostas ou aceitas; consulte o status antes de tratá-las como orientação vigente.

## Índice

| ADR | Status | Resumo |
| --- | --- | --- |
| [0001](0001-adopt-clean-architecture-with-four-responsibility-layers.md) | Aceita | Organiza sistemas em quatro camadas com dependências direcionadas às regras de negócio. |
| [0002](0002-use-the-result-pattern-instead-of-exceptions-for-expected-failures.md) | Aceita | Representa falhas esperadas explicitamente com o padrão Result, reservando exceções para falhas inesperadas. |
| [0003](0003-return-all-errors-as-rfc-9457-problemdetails.md) | Aceita | Define RFC 9457 Problem Details como contrato para erros HTTP, com mensagens seguras e identificadores estáveis. |
| [0004](0004-validate-application-configuration-before-startup-completes.md) | Aceita | Valida a configuração efetiva antes de concluir a inicialização e aceitar tráfego. |
| [0005](0005-use-repositories-and-unit-of-work-for-persistence.md) | Aceita | Separa os contratos de persistência em Application e coordena gravações com Unit of Work. |
| [0006](0006-apply-database-migrations-in-cd-not-at-application-startup.md) | Aceita | Executa migrações de banco de dados em CD, não durante a inicialização da aplicação. |
| [0007](0007-use-an-injectable-clock-and-explicit-business-time-zones.md) | Aceita | Usa uma fonte de tempo substituível e fusos explícitos para regras de negócio testáveis. |
| [0008](0008-test-with-nunit-nsubstitute-and-microsoft-testing-platform.md) | Aceita | Define ferramentas e práticas de testes para projetos .NET, incluindo testes unitários e de integração. |
| [0009](0009-return-successful-collection-responses-as-http-200.md) | Aceita | Retorna `200 OK` e uma coleção JSON, inclusive vazia, em consultas bem-sucedidas. |
| [0010](0010-gate-deployment-on-readiness-and-degrade-unavailable-capabilities-explicitly.md) | Aceita | Bloqueia implantações sem readiness crítica e explicita a degradação de capacidades indisponíveis. |
| [0011](0011-establish-a-consistent-process-for-creating-and-maintaining-adrs.md) | Aceita | Define como criar, relacionar e manter ADRs, priorizando ADRS via MCP, com CLI como último recurso, conferência de status e validação final com doctor. |

## Ferramentas e créditos

- **[adrs](https://github.com/joshrotenberg/adrs)** — ferramenta CLI e servidor MCP usados para criar, consultar, validar e relacionar ADRs. Criada por [Josh Rotenberg](https://github.com/joshrotenberg).
- **[Architectural Decision Records](https://adr.github.io/)** — iniciativa e recursos comunitários que orientam este formato de registro.

## Instruções para agentes

As regras de trabalho neste repositório estão em [docs/AGENTS.md](docs/AGENTS.md).

## Contribuições

Cada ADR deve registrar uma decisão. Mudanças de direção devem preservar o histórico e explicitar a relação com decisões anteriores, em vez de apagar silenciosamente o contexto original.
