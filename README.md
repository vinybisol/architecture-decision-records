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
| [0012](0012-maintain-a-human-written-changelog-following-keep-a-changelog.md) | Aceita | Mantém um changelog escrito por pessoas no formato Keep a Changelog, descrevendo o impacto para quem consome o sistema, e não as mensagens de commit. |
| [0013](0013-version-releases-with-semantic-versioning-derived-from-changelog-categories.md) | Aceita | Versiona releases com Semantic Versioning, escolhendo o incremento a partir das categorias do changelog. |
| [0014](0014-use-annotated-version-tags-as-the-single-source-of-truth-for-release-versions.md) | Aceita | Usa tags Git anotadas `vX.Y.Z` como única fonte da versão, sem versão gravada em arquivos do projeto, e trata versões publicadas como imutáveis. |
| [0015](0015-use-a-simplified-git-flow-branching-model-with-develop-and-main.md) | Aceita | Adota um Git Flow simplificado com `develop` e `main`, branches `feature/*` e `hotfix/*`, sem cherry-pick e com feature flags para controlar o escopo da release. |
| [0016](0016-squash-feature-merges-and-use-merge-commits-between-long-lived-branches.md) | Aceita | Usa squash de `feature/*` para `develop`, merge commit entre branches de longa duração e back-merge obrigatório de `main` para `develop`. |
| [0017](0017-trigger-releases-from-version-tags-and-build-each-release-artifact-once.md) | Aceita | Dispara releases pelo push da tag de versão, constrói o artefato uma única vez e o promove entre ambientes, publicando as notas a partir do changelog. |
| [0018](0018-build-disposable-pre-release-versions-from-every-integration-into-develop.md) | Aceita | Gera builds de pré-release descartáveis a cada integração na `develop`, versionados a partir da última tag alcançável. |
| [0019](0019-expose-the-build-assigned-version-at-runtime.md) | Aceita | Exige que backends e aplicações de navegador exponham em runtime a versão atribuída no build, deixando o mecanismo a cargo de cada framework. |

## Ferramentas e créditos

- **[adrs](https://github.com/joshrotenberg/adrs)** — ferramenta CLI e servidor MCP usados para criar, consultar, validar e relacionar ADRs. Criada por [Josh Rotenberg](https://github.com/joshrotenberg).
- **[Architectural Decision Records](https://adr.github.io/)** — iniciativa e recursos comunitários que orientam este formato de registro.

## Instruções para agentes

As regras de trabalho neste repositório estão em [docs/AGENTS.md](docs/AGENTS.md).

## Contribuições

Cada ADR deve registrar uma decisão. Mudanças de direção devem preservar o histórico e explicitar a relação com decisões anteriores, em vez de apagar silenciosamente o contexto original.
