# ADR-002: PostgreSQL como banco principal

**Status:** Aceito
**Data:** 2026-10-26

## Contexto

O domínio do MedAgenda é relacional: uma consulta liga paciente,
profissional e horário, e quase todas as telas cruzam essas informações.

A regra RF07 exige que dois agendamentos nunca ocupem o mesmo horário
do mesmo profissional, inclusive com requisições simultâneas. Verificar
isso apenas no código não resiste a condições de corrida, então a regra
precisa ser garantida pelo banco.

As opções consideradas foram PostgreSQL e MongoDB. Há experiência prévia
com MongoDB, o que aceleraria o início.

## Decisão

Usar PostgreSQL como banco principal, com o esquema versionado pelo Flyway
e acesso pelo Spring Data JPA.

A regra RF07 será garantida por uma restrição de exclusão sobre o intervalo
de tempo da consulta, impedindo intervalos sobrepostos para o mesmo
profissional.

## Consequências

**Positivas**
- RF07 garantido pelo banco, resistente a requisições concorrentes
- Joins, chaves estrangeiras e transações naturais para o domínio
- Compatível com o restante da stack (Flyway, Spring Modulith, Testcontainers)
- Amplia a experiência com bancos relacionais, complementando o MongoDB

**Negativas**
- Curva de aprendizado em SQL, JPA e recursos específicos do PostgreSQL
- Mudanças no esquema exigem migrações, em vez de documentos flexíveis

## Alternativa considerada

**MongoDB:** encaixaria bem nos registros de prontuário em texto livre, mas
a regra RF07 exigiria modelar cada horário como documento com índice único,
ou depender de travas no código, e parte da stack assume um banco relacional.