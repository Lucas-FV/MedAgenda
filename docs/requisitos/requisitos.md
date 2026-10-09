# Requisitos

Requisitos do MedAgenda agrupados por módulo do backend. Cada requisito
tem um ID usado em testes e commits para rastreabilidade (ex.: `RF07`).

**Prioridade (MoSCoW):** M = must (obrigatório) · S = should (importante) · C = could (desejável)
**Status:** ⬜ pendente · 🟡 em andamento · ✅ concluído

## Requisitos funcionais

| ID | Módulo | Requisito | Prioridade | Status |
| --- | --- | --- | :---: | :---: |
| RF01 | Identidade | Paciente se cadastra com e-mail e confirma a conta por link | M | ⬜ |
| RF02 | Identidade | Login com e-mail e senha; 2FA (TOTP) obrigatório para equipe | M | ⬜ |
| RF03 | Identidade | Recuperação de senha por e-mail com token de uso único | M | ⬜ |
| RF04 | Identidade | Administrador cria contas de equipe e atribui papéis | M | ⬜ |
| RF05 | Agenda | Profissional define horários de atendimento e bloqueios | M | ⬜ |
| RF06 | Agenda | Paciente vê horários livres e agenda uma consulta | M | ⬜ |
| RF07 | Agenda | Dois agendamentos nunca ocupam o mesmo horário do mesmo profissional | M | ⬜ |
| RF08 | Agenda | Paciente e recepção cancelam ou remarcam respeitando antecedência mínima | S | ⬜ |
| RF09 | Agenda | E-mail de confirmação e lembrete 24h antes da consulta | S | ⬜ |
| RF10 | Prontuário | Profissional registra evolução vinculada a uma consulta realizada | M | ⬜ |
| RF11 | Prontuário | Registros não são apagados; correções geram nova versão | S | ⬜ |
| RF12 | Prontuário | Profissional consulta o histórico apenas de pacientes que atendeu | M | ⬜ |
| RF13 | Paciente | Paciente vê e atualiza seus dados cadastrais | M | ⬜ |
| RF14 | Paciente | Paciente exporta seus dados em JSON e PDF | S | ⬜ |
| RF15 | Paciente | Paciente solicita exclusão; sistema anonimiza o que a lei permite | S | ⬜ |
| RF16 | Auditoria | Todo acesso a prontuário e dado sensível gera registro de auditoria | M | ⬜ |
| RF17 | Auditoria | Administrador filtra a trilha por usuário, paciente e período | S | ⬜ |
| RF18 | Painel | Dashboard com ocupação da agenda e taxa de faltas | C | ⬜ |

## Requisitos não funcionais

Em breve...