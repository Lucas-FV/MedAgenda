# MedAgenda

> Sistema web para clínicas pequenas gerenciarem agenda, pacientes e
> prontuários, com segurança e LGPD como requisitos desde o primeiro dia.

🚧 **Em desenvolvimento.** Acompanhe o progresso pelos commits e pelos
[registros de decisão](docs/adr/).

## O problema

Clínicas pequenas costumam organizar consultas em planilhas, agenda de
papel ou WhatsApp. Dados de saúde ficam espalhados, sem controle de quem
acessou o quê, e a clínica não consegue atender pedidos de pacientes
previstos na LGPD.

## A proposta

- Agendamento online, sem conflito de horários
- Prontuário com registros imutáveis e versionados
- Área do paciente para exportar ou pedir exclusão dos próprios dados
- Trilha de auditoria de todo acesso a dados sensíveis

## Stack planejada

Java 25 · Spring Boot · Spring Modulith · PostgreSQL · React · TypeScript · Docker · GitHub Actions

## Documentação

- [Decisões de arquitetura (ADRs)](docs/adr/)