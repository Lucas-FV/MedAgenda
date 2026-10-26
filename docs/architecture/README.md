# Arquitetura

Diagramas no modelo C4, do nível mais amplo ao mais detalhado.
As decisões por trás deles estão em [ADRs](../adr/).

## Nível 1: Contexto

Quem usa o MedAgenda e com quais sistemas externos ele se comunica.

```mermaid
flowchart TB
    paciente["Paciente<br/>Agenda consultas e exerce<br/>seus direitos da LGPD"]
    recepcao["Recepção<br/>Gerencia agenda e cadastros"]
    profissional["Profissional de saúde<br/>Atende e registra prontuários"]
    admin["Administrador<br/>Gerencia usuários e<br/>consulta a auditoria"]

    medagenda["MedAgenda<br/>Agenda, prontuário, área do paciente<br/>e trilha de auditoria"]

    email["Serviço de e-mail<br/>Entrega confirmações e lembretes"]

    paciente -->|"Agenda, cancela, exporta dados<br/>[HTTPS]"| medagenda
    recepcao -->|"Gerencia agenda e pacientes<br/>[HTTPS]"| medagenda
    profissional -->|"Consulta agenda, registra evolução<br/>[HTTPS]"| medagenda
    admin -->|"Gerencia equipe, filtra auditoria<br/>[HTTPS]"| medagenda
    medagenda -->|"Envia e-mails<br/>[SMTP]"| email
    email -->|"Entrega e-mails"| paciente

    classDef pessoa fill:#08427b,stroke:#052e56,color:#fff
    classDef sistema fill:#1168bd,stroke:#0b4884,color:#fff
    classDef externo fill:#8a8a8a,stroke:#6b6b6b,color:#fff
    class paciente,recepcao,profissional,admin pessoa
    class medagenda sistema
    class email externo
```

## Nível 2: Containers

As partes que rodam separadamente.

```mermaid
flowchart TB
    usuarios["Usuários<br/>Paciente, recepção,<br/>profissional, administrador"]

    subgraph medagenda["MedAgenda"]
        spa["Aplicação web<br/>[React, TypeScript, Vite]<br/>Interface responsiva no navegador"]
        api["API<br/>[Java 25, Spring Boot, Spring Modulith]<br/>Regras de negócio, segurança e auditoria"]
        db[("Banco de dados<br/>[PostgreSQL]<br/>Agenda, cadastros, prontuários cifrados<br/>e trilha de auditoria")]
    end

    email["Serviço de e-mail<br/>[Mailpit local / provedor no deploy]"]

    usuarios -->|"Usa<br/>[HTTPS]"| spa
    spa -->|"Chama a API<br/>[JSON/HTTPS, JWT]"| api
    api -->|"Lê e grava<br/>[JDBC]"| db
    api -->|"Envia e-mails<br/>[SMTP]"| email

    classDef pessoa fill:#08427b,stroke:#052e56,color:#fff
    classDef container fill:#438dd5,stroke:#2e6295,color:#fff
    classDef externo fill:#8a8a8a,stroke:#6b6b6b,color:#fff
    class usuarios pessoa
    class spa,api,db container
    class email externo
```

## Nível 3: Componentes da API

Os módulos do monolito modular. Linhas contínuas são chamadas à API
pública de outro módulo; linhas tracejadas são eventos de domínio.

```mermaid
flowchart LR
    subgraph api["API (Spring Boot)"]
        identidade["Identidade<br/>Cadastro, login, 2FA, papéis"]
        agenda["Agenda<br/>Disponibilidade, bloqueios, consultas"]
        prontuario["Prontuário<br/>Registros de evolução versionados"]
        paciente["Paciente<br/>Dados cadastrais e direitos LGPD"]
        painel["Painel<br/>Indicadores da agenda"]
        auditoria["Auditoria<br/>Trilha somente inserção"]
    end

    prontuario -->|"Consulta foi realizada?"| agenda
    paciente -->|"Dados para exportação"| prontuario
    painel -->|"Ocupação e faltas"| agenda

    identidade -.->|"Eventos de login e papéis"| auditoria
    agenda -.->|"Eventos de consulta"| auditoria
    prontuario -.->|"Prontuário acessado"| auditoria
    paciente -.->|"Exportação, exclusão"| auditoria

    classDef componente fill:#85bbf0,stroke:#5d82a8,color:#000
    class identidade,agenda,prontuario,paciente,painel,auditoria componente
```

O módulo de Identidade é usado por todos os outros por meio do Spring
Security; essas ligações foram omitidas para manter o diagrama legível.