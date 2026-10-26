# ADR-001: Monolito modular em vez de microsserviços

**Status:** Aceito
**Data:** 2026-10-12

## Contexto

O MedAgenda atende uma única clínica, com poucos usuários simultâneos,
e é desenvolvido por uma pessoa em meio período. O domínio tem áreas
bem definidas (identidade, agenda, prontuário, paciente, auditoria),
e a auditoria precisa observar eventos de todas elas.

Microsserviços trariam deploy independente e escala por serviço, mas
também chamadas de rede sujeitas a falhas, transações distribuídas e
a operação de vários deploys, sem um problema real que justifique
esse custo.

## Decisão

Construir um monolito modular com Spring Modulith: um único deploy,
com um módulo por área de negócio. As fronteiras entre módulos são
verificadas por teste automatizado, e a comunicação entre eles usa
APIs públicas de cada módulo ou eventos de domínio.

Internamente, os módulos com regras de negócio relevantes seguem
arquitetura hexagonal: domínio sem dependência de framework, portas
como interfaces e adaptadores na borda.

## Consequências

**Positivas**
- Um único deploy e um único banco, simples de operar e de rodar localmente
- Fronteiras entre módulos garantidas no CI, e não só por convenção
- Domínio testável com testes unitários, sem Spring nem banco
- Um módulo pode ser extraído para um serviço no futuro, se houver motivo

**Negativas**
- Mais classes e mapeamentos (modelo de domínio separado da entidade JPA)
- Todos os módulos escalam juntos
- Exige disciplina para não contornar as fronteiras