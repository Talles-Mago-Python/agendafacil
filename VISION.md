# Visão do produto: AgendaFácil

## Problema
Pequenos negócios de atendimento (barbearias, clínicas, estúdios de estética, personal trainers) ainda marcam horários por WhatsApp e telefone. Isso gera:
- tempo perdido respondendo "tem horário amanhã?";
- conflitos de agenda e esquecimentos;
- nenhuma visão clara da agenda do dia ou da semana.

## Solução
Uma página onde o **cliente** vê os serviços, escolhe um horário livre e confirma o agendamento sozinho, e um **painel** onde o **dono do negócio** controla a agenda, os serviços, o horário de funcionamento e os bloqueios.

## Usuários

| Persona | Quem é | O que quer |
|---|---|---|
| **Cliente** | Pessoa que agenda um serviço, geralmente pelo celular | Agendar em menos de 1 minuto, ver e cancelar seus horários |
| **Admin** | Dono ou recepcionista do negócio | Ver a agenda do dia, cadastrar serviços, bloquear horários, marcar atendimentos como concluídos ou como falta |

## Escopo do MVP (versão 1)

### Dentro do escopo ✅
1. Cadastro e login de clientes (e-mail + senha)
2. Admin cadastra, edita e desativa **serviços** (nome, duração, preço)
3. Admin define o **horário de funcionamento** por dia da semana
4. Admin cria **bloqueios** (folga, almoço, feriado)
5. Cliente vê os **horários disponíveis** de um serviço numa data e **agenda**
6. Cliente vê seus agendamentos e **cancela** (respeitando a antecedência mínima)
7. Admin vê a **agenda** (dia/semana), cancela e marca como **concluído** ou **falta**

### Fora do escopo (NÃO fazer na v1) ❌
- Vários profissionais ou várias agendas paralelas (o MVP tem **um profissional**)
- Vários negócios na mesma instalação (multi-tenant)
- Pagamento online
- Notificações por e-mail, SMS ou WhatsApp
- Login social (Google etc.)
- Recuperação de senha por e-mail (na v1 o admin redefine manualmente)
- App mobile nativo (o site deve ser responsivo)
- Avaliações e comentários

> Agentes: se uma tarefa parecer exigir algo desta lista, **pare e pergunte**.

## Métricas de sucesso do MVP
- O cliente conclui um agendamento em até **4 telas/cliques** a partir da home
- Zero agendamentos sobrepostos (garantido pelo banco)
- O admin vê a agenda do dia em **1 clique** após o login
- Lighthouse mobile ≥ 90 em performance e acessibilidade

## Princípios de produto
1. **Mobile first.** A maioria dos clientes vai usar o celular.
2. **Simples antes de completo.** Menos opções e fluxo mais curto.
3. **A agenda nunca mente.** Se o horário aparece como disponível, ele está disponível.
4. **Português claro.** Nada de jargão técnico na interface.
