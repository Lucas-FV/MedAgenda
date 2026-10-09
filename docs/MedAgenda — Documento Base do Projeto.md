# MedAgenda — Documento Base do Projeto

Oct 9, 2026 · @Lucas Vilela

## Visão geral

O MedAgenda é um sistema web para clínicas pequenas gerenciarem agenda, pacientes e prontuários, com segurança e LGPD tratadas como requisitos de primeira classe.

**Problema.** Clínicas pequenas costumam usar planilhas, agenda de papel ou WhatsApp. Dados de saúde ficam espalhados, sem controle de quem acessou o quê, e a clínica não consegue atender pedidos de pacientes previstos na LGPD.

**Proposta.** Uma aplicação web com agendamento online, prontuário simples, área do paciente e trilha de auditoria de todo acesso a dados sensíveis.

**Objetivos do projeto de portfólio:**

- Demonstrar desenvolvimento web full-stack completo (Spring Boot + React), do domínio ao deploy
- Mostrar decisões de arquitetura documentadas (C4, ADRs), não só código funcionando
- Ter uma suíte de testes real: unitários, integração, E2E e segurança automatizada no CI
- Diferenciar o perfil com segurança aplicada e adequação à LGPD, temas raros em portfólios de estudantes
- Estar no ar com deploy público e dados fictícios para qualquer recrutador testar

## Escopo

O MVP atende uma única clínica com vários profissionais; multi-clínica fica para depois.

**Perfis de usuário**

| Perfil | O que faz | O que não pode fazer |
| --- | --- | --- |
| Paciente | Agenda e cancela consultas, vê seus dados, solicita exportação ou exclusão | Ver dados de outros pacientes ou prontuários completos |
| Recepção | Gerencia agenda, cadastra pacientes, confirma consultas | Ler o conteúdo de prontuários |
| Profissional de saúde | Vê sua agenda, registra e consulta prontuários dos seus pacientes | Acessar prontuários de pacientes que nunca atendeu |
| Administrador | Gerencia usuários, profissionais, horários, consulta a trilha de auditoria | Ler prontuários (separação de funções) |

**Dentro do escopo**

- Cadastro e autenticação, com 2FA para equipe
- Agenda por profissional, com horários disponíveis e bloqueios
- Prontuário simples: anotações de evolução por consulta
- Área do paciente com direitos da LGPD
- Trilha de auditoria e painel administrativo
- Notificações por e-mail (confirmação e lembrete)

**Fora do escopo (por enquanto)**

- Pagamentos, convênios e faturamento
- Telemedicina e prescrição eletrônica
- App mobile nativo (o front será responsivo)
- Integrações com sistemas de saúde externos (HL7/FHIR)

## Requisitos funcionais

Os requisitos estão agrupados pelos módulos que viram pacotes do backend; a prioridade segue MoSCoW (M = must, S = should, C = could).

| ID | Módulo | Requisito | Prioridade |
| --- | --- | --- | --- |
| RF01 | Identidade | Paciente se cadastra com e-mail e confirma a conta por link | M |
| RF02 | Identidade | Login com e-mail e senha; 2FA (TOTP) obrigatório para equipe | M |
| RF03 | Identidade | Recuperação de senha por e-mail com token de uso único | M |
| RF04 | Identidade | Administrador cria contas de equipe e atribui papéis | M |
| RF05 | Agenda | Profissional define horários de atendimento e bloqueios | M |
| RF06 | Agenda | Paciente vê horários livres e agenda uma consulta | M |
| RF07 | Agenda | Dois agendamentos nunca ocupam o mesmo horário do mesmo profissional | M |
| RF08 | Agenda | Paciente e recepção cancelam ou remarcam respeitando antecedência mínima | S |
| RF09 | Agenda | E-mail de confirmação e lembrete 24h antes da consulta | S |
| RF10 | Prontuário | Profissional registra evolução vinculada a uma consulta realizada | M |
| RF11 | Prontuário | Registros não são apagados; correções geram nova versão | S |
| RF12 | Prontuário | Profissional consulta o histórico apenas de pacientes que atendeu | M |
| RF13 | Paciente | Paciente vê e atualiza seus dados cadastrais | M |
| RF14 | Paciente | Paciente exporta seus dados em JSON e PDF | S |
| RF15 | Paciente | Paciente solicita exclusão; sistema anonimiza o que a lei permite | S |
| RF16 | Auditoria | Todo acesso a prontuário e dado sensível gera registro de auditoria | M |
| RF17 | Auditoria | Administrador filtra a trilha por usuário, paciente e período | S |
| RF18 | Painel | Dashboard com ocupação da agenda e taxa de faltas | C |

## Requisitos não funcionais

Segurança e manutenibilidade pesam mais que desempenho neste projeto; as metas abaixo são verificáveis no CI ou em testes de carga.

| ID | Atributo | Meta |
| --- | --- | --- |
| RNF01 | Segurança | Nenhum alerta High no OWASP ZAP e nenhuma dependência com CVE crítica no merge para main |
| RNF02 | Segurança | Senhas com Argon2 ou BCrypt; dados clínicos criptografados em repouso |
| RNF03 | Segurança | Todo tráfego em HTTPS, com HSTS e CSP configurados |
| RNF04 | Privacidade | Logs de aplicação sem dados pessoais (mascaramento de CPF, e-mail, telefone) |
| RNF05 | Desempenho | p95 abaixo de 300 ms nas rotas de agenda com 50 usuários simultâneos (k6) |
| RNF06 | Disponibilidade | Health checks e reinício automático no ambiente de deploy |
| RNF07 | Manutenibilidade | Cobertura de linhas mínima de 80% no domínio; regras de camada verificadas por ArchUnit |
| RNF08 | Observabilidade | Logs estruturados em JSON com ID de correlação por requisição |
| RNF09 | Usabilidade | Interface responsiva e navegável por teclado (WCAG 2.1 AA como referência) |
| RNF10 | Portabilidade | Projeto inteiro sobe com um único `docker compose up` |

## Arquitetura

O MedAgenda começa como um monolito modular: um único deploy, com módulos isolados por domínio que podem virar serviços no futuro se houver motivo real.

&#91;embedded content: arquitetura · containers e módulos do backend\]

Cada módulo usa arquitetura hexagonal por dentro: domínio puro no centro, portas como interfaces e adaptadores (web, banco, e-mail) na borda. A auditoria recebe eventos de domínio dos outros módulos, sem que eles dependam dela diretamente.

**Stack**

| Camada | Tecnologia | Por quê |
| --- | --- | --- |
| Backend | Java 25 (LTS), Spring Boot, Spring Modulith | Módulos com fronteiras verificadas em teste |
| Segurança | Spring Security, JWT, TOTP | Padrão de mercado, bem documentado |
| Persistência | PostgreSQL, Spring Data JPA, Flyway | Dados relacionais e restrições fortes no banco |
| Frontend | React, TypeScript, Vite, TanStack Query | Tipagem e cache de dados do servidor |
| Infra | Docker Compose, GitHub Actions | Ambiente reproduzível e CI desde o primeiro commit |
| Observabilidade | Logs JSON, Spring Boot Actuator | Health checks e rastreio por requisição |

**ADRs iniciais**

1. ADR-001: monolito modular em vez de microsserviços
2. ADR-002: PostgreSQL como banco principal
3. ADR-003: JWT de vida curta com refresh token em cookie HttpOnly
4. ADR-004: criptografia em nível de campo para CPF e prontuário
5. ADR-005: auditoria por eventos de domínio em tabela somente inserção
6. ADR-006: SSRF (OWASP A10) registrado como risco aceito

## Modelo de domínio inicial

O coração do domínio é a Consulta: ela liga paciente e profissional a um horário e é a única porta para criar registros de prontuário.

| Entidade | Atributos principais | Regras |
| --- | --- | --- |
| Usuário | id, e-mail, hash da senha, papel, segredo 2FA, ativo | E-mail único; equipe exige 2FA |
| Paciente | id, usuário, nome, CPF (criptografado), nascimento, telefone, consentimentos | CPF único; dados sensíveis cifrados |
| Profissional | id, usuário, nome, registro profissional, especialidade | Registro profissional único |
| Disponibilidade | profissional, dia da semana, início, fim, duração do slot | Sem sobreposição para o mesmo profissional |
| Bloqueio | profissional, início, fim, motivo | Impede agendamentos no intervalo |
| Consulta | id, paciente, profissional, início, fim, status | Status: AGENDADA, CONFIRMADA, REALIZADA, CANCELADA, FALTA |
| Registro de prontuário | id, consulta, autor, texto (criptografado), versão, criado em | Só criado em consulta REALIZADA; imutável, correção gera nova versão |
| Consentimento | paciente, finalidade, versão do termo, aceito em, revogado em | Histórico preservado |
| Solicitação do titular | paciente, tipo (exportação, exclusão), status, datas | Prazo de resposta controlado |
| Evento de auditoria | quem, ação, recurso, id do recurso, quando, IP, resultado | Somente inserção; nunca editado |

**Invariante crítica (RF07):** dois agendamentos não podem ocupar o mesmo horário do mesmo profissional. Ela será garantida no banco (restrição de exclusão ou índice único), e não só no código, para resistir a requisições concorrentes.

## Segurança e OWASP Top 10

Cada categoria do OWASP Top 10 (edição 2021) tem uma mitigação concreta e um teste que a comprova; essa tabela vira uma página da documentação do repositório.

| Categoria | Risco no MedAgenda | Mitigação | Como é verificado |
| --- | --- | --- | --- |
| A01 Controle de acesso quebrado | Paciente lê consulta de outro trocando o ID na URL (IDOR) | Checagem de posse no serviço, não só no controller; IDs UUID | Testes de integração tentando acessar recurso alheio, esperando 404 |
| A02 Falhas criptográficas | Vazamento de CPF ou prontuário em backup | Criptografia de campo com chave fora do banco; HTTPS + HSTS | Teste que lê a coluna direto no banco e confirma texto cifrado |
| A03 Injeção | SQL ou NoSQL injection em filtros de busca | Queries parametrizadas, validação com Bean Validation | Testes com payloads conhecidos + ZAP |
| A04 Design inseguro | Recuperação de senha que revela se o e-mail existe | Resposta idêntica para e-mail existente ou não; tokens de uso único | Teste de API comparando respostas |
| A05 Configuração incorreta | Stack trace exposto, CORS aberto | Handler global de erros, CORS restrito ao front, cabeçalhos de segurança | ZAP baseline no CI |
| A06 Componentes vulneráveis | Biblioteca com CVE | Dependabot + OWASP Dependency-Check | Pipeline falha em CVE crítica |
| A07 Falhas de autenticação | Força bruta no login | Rate limiting por IP e por conta, bloqueio temporário, 2FA para equipe | Teste de carga no login esperando 429 |
| A08 Integridade de software e dados | Edição silenciosa de prontuário | Registros imutáveis com versionamento; builds reproduzíveis | Teste de domínio impedindo update |
| A09 Falhas de log e monitoramento | Acesso indevido sem rastro | Trilha de auditoria somente inserção; alertas de muitos 403 | Teste confirmando evento a cada leitura de prontuário |
| A10 SSRF | Pouco aplicável: o sistema não busca URLs informadas pelo usuário | Nenhuma funcionalidade que faça requisição para URL do usuário | Registrado como risco aceito em ADR |

**Autenticação.** JWT de acesso com vida curta (15 min) e refresh token rotativo em cookie HttpOnly, Secure, SameSite=Strict. O front nunca guarda tokens em localStorage.

## LGPD

Dados de saúde são dados pessoais sensíveis na LGPD, então o sistema trata cada dado com uma finalidade e uma base legal declaradas. As bases abaixo são uma proposta de estudo e devem ser revisadas na fase de documentação; o projeto é de portfólio e não substitui orientação jurídica.

| Dado | Finalidade | Base legal proposta | Retenção |
| --- | --- | --- | --- |
| Nome, e-mail, telefone | Identificar o paciente e enviar avisos | Execução de contrato (art. 7º, V) | Enquanto a conta estiver ativa |
| CPF | Identificação única e documentos | Execução de contrato (art. 7º, V) | Enquanto a conta estiver ativa |
| Prontuário | Registro do atendimento | Tutela da saúde (art. 11, II, f) | Prazo legal de guarda de prontuário (a confirmar) |
| Logs de auditoria | Rastrear acessos a dados sensíveis | Obrigação legal e segurança | Período definido em ADR |
| E-mails de lembrete | Reduzir faltas | Consentimento (revogável) | Até a revogação |

**Direitos do titular implementados**

- Acesso e portabilidade: exportação dos dados em JSON e PDF pela área do paciente
- Correção: edição dos dados cadastrais
- Eliminação: solicitação de exclusão; dados cadastrais são anonimizados e o prontuário é mantido pelo prazo legal, com a justificativa exibida ao paciente
- Revogação de consentimento: desligar lembretes a qualquer momento, com histórico preservado

**Princípios aplicados no código:** minimização (só coletar o que tem finalidade), mascaramento de dados pessoais nos logs, separação de funções (admin não lê prontuário) e dados fictícios em todos os ambientes não produtivos.

## Estratégia de testes

A suíte segue a pirâmide de testes: muitos testes rápidos de domínio na base, poucos E2E no topo, e segurança automatizada rodando a cada pull request.

| Camada | Ferramentas | O que cobre | Quando roda |
| --- | --- | --- | --- |
| Unitário (domínio) | JUnit 5, AssertJ | Regras de agenda, status da consulta, versionamento de prontuário, sem Spring | Todo push |
| Arquitetura | ArchUnit | Domínio não depende de infraestrutura; controllers não acessam repositórios | Todo push |
| Integração | Spring Boot Test, Testcontainers | Repositórios, transações, restrição de horário duplicado, criptografia de campo | Todo push |
| API e autorização | MockMvc ou RestAssured | Cada endpoint testado com cada papel, incluindo tentativas de IDOR | Todo push |
| Frontend | Vitest, React Testing Library | Componentes, formulários, estados de erro | Todo push |
| E2E | Playwright | Fluxos críticos: cadastro, agendar, atender, exportar dados | Pull request |
| Segurança (DAST) | OWASP ZAP baseline | Cabeçalhos, exposição de erros, vulnerabilidades comuns | Pull request para main |
| Dependências (SCA) | Dependabot, OWASP Dependency-Check | CVEs em bibliotecas | Pull request e semanal |
| Carga | k6 | Meta de p95 da agenda (RNF05) e rate limiting do login | Manual, antes de cada release |

**Concorrência.** Um teste de integração dispara várias requisições simultâneas para o mesmo horário e confirma que só uma vence. Esse teste costuma render uma boa explicação no README e no post do LinkedIn.

## Documentação e entregáveis

A documentação mora no próprio repositório, em `docs/`, para evoluir junto com o código.

| Entregável | Onde fica | Conteúdo |
| --- | --- | --- |
| README | Raiz do repositório | Problema, GIF da aplicação, link do deploy, stack, como rodar com Docker, usuários de teste |
| Diagramas C4 | `docs/architecture/` | Contexto, containers e componentes do backend |
| ADRs | `docs/adr/` | Uma decisão por arquivo: contexto, decisão, consequências |
| Mapa OWASP | `docs/security.md` | A tabela da seção de segurança, mantida atualizada |
| Mapa LGPD | `docs/privacy.md` | Dados, finalidades, bases legais, direitos implementados |
| API | `/swagger-ui` no deploy | OpenAPI gerado pelo springdoc |
| Estratégia de testes | `docs/testing.md` | Pirâmide, ferramentas, como rodar cada camada |
| Contribuição | `CONTRIBUTING.md` | Padrão de commits (Conventional Commits), branches, checklist de PR |
| Changelog | `CHANGELOG.md` | O que mudou em cada versão |
| Deploy público | Render, Railway ou Fly.io | Ambiente com dados fictícios, resetado periodicamente |

**Para o LinkedIn:** um post de lançamento com o GIF e o link, e posts curtos ao longo do caminho sobre uma decisão específica (por exemplo, como o sistema impede agendamento duplicado, ou como a trilha de auditoria funciona).

## Roadmap

O MVP leva de 10 a 13 semanas em ritmo de meio período (estimativa); cada fase só termina quando o seu critério de saída (gate) é cumprido.

&#91;embedded content: roadmap · 6 fases e seus critérios de saída\]

Segurança e testes entram desde a Fase 0, não no fim. Bons momentos para posts curtos no LinkedIn: o fim da Fase 2 (algo visual e o teste de concorrência) e o fim da Fase 3 (auditoria).

## Decisões em aberto

Cada item vira um ADR quando for decidido.

- [ ] Nome final do projeto (MedAgenda é provisório)
- [ ] Banco: PostgreSQL (recomendado pelos dados relacionais e pela restrição de horário) ou MongoDB (onde você já tem experiência)
- [ ] Onde guardar a chave de criptografia de campo: variável de ambiente no MVP ou um cofre de segredos
- [ ] Provedor de deploy: Render, Railway ou Fly.io
- [ ] Serviço de e-mail: Mailpit no ambiente local e qual provedor no deploy
- [ ] Prazo de retenção da trilha de auditoria e de dados após pedido de exclusão
- [ ] Biblioteca de UI no front: Tailwind + shadcn/ui ou Material UI
