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
| [0008](0008-test-with-nunit-nsubstitute-and-microsoft-testing-platform.md) | Proposta | Propõe ferramentas e práticas de testes para projetos .NET, incluindo testes unitários e de integração. |
| [0009](0009-return-successful-collection-responses-as-http-200.md) | Aceita | Retorna `200 OK` e uma coleção JSON, inclusive vazia, em consultas bem-sucedidas. |
| [0010](0010-gate-deployment-on-readiness-and-degrade-unavailable-capabilities-explicitly.md) | Proposta | Propõe gates de readiness para implantação e degradação explícita de capacidades indisponíveis. |

## Escopo

As decisões são independentes de tecnologia quando indicado no próprio documento. A ADR 8 é específica para .NET e ainda está **Proposta**; suas ferramentas e práticas não devem ser interpretadas como decisões já aceitas.

## Ferramentas e créditos

- **[adrs](https://github.com/joshrotenberg/adrs)** — ferramenta CLI e servidor MCP usados para criar, consultar, validar e relacionar ADRs. Criada por [Josh Rotenberg](https://github.com/joshrotenberg).
- **[Architectural Decision Records](https://adr.github.io/)** — iniciativa e recursos comunitários que orientam este formato de registro.

A ADR 8 propõe as ferramentas abaixo para testes .NET. Os créditos apontam para os projetos ou organizações mantenedoras; confira a ADR para o contexto e o status da decisão.

| Ferramenta | Crédito / projeto |
| --- | --- |
| NUnit e NUnit.Analyzers | [NUnit project](https://github.com/nunit) |
| NSubstitute | [NSubstitute project](https://github.com/nsubstitute/NSubstitute) |
| Microsoft.Testing.Platform | [Microsoft](https://github.com/microsoft/testfx) |
| Microsoft.Testing.Extensions.CodeCoverage | [Microsoft](https://www.nuget.org/packages/Microsoft.Testing.Extensions.CodeCoverage) |
| Bogus | [Brian Chavez](https://github.com/bchavez/Bogus) |
| WebApplicationFactory | [ASP.NET Core / Microsoft](https://github.com/dotnet/aspnetcore) |
| PostgreSQL | [PostgreSQL Global Development Group](https://www.postgresql.org/about/) |
| Respawn | [Jimmy Bogard](https://github.com/jbogard/Respawn) |
| Docker | [Docker](https://www.docker.com/) |

## Contribuições

Cada ADR deve registrar uma decisão. Mudanças de direção devem preservar o histórico e explicitar a relação com decisões anteriores, em vez de apagar silenciosamente o contexto original.
