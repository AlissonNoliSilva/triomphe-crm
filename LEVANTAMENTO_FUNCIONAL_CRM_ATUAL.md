# Levantamento Funcional Completo — CRM Triomphe (Odoo 8)

**Documento base para a construção do novo CRM (backend Django + frontend React, arquitetura de referência: `triomphe-whatsapp`)**

> Este documento descreve **tudo o que existe hoje** no CRM imobiliário Triomphe construído sobre Odoo 8 (módulos `rdx_remax_crm`, `triomphe_web`, `rdx_base_ext`, `rdx_hr_teams`, `rdx_sales_team_extended`, `rdx_pt_addresses_ext`, `rdx_product_images`, `rdx_olx_real_estates_crawler`, `rdx_l10n_pt`). Nada do que está aqui pode ficar de fora do novo sistema sem decisão explícita.
>
> Data do levantamento: 2026-07-20. Fonte: leitura direta do código (`Triomphe addons/`).

---

## Índice

1. [Visão geral do sistema atual](#1-visão-geral)
2. [Multi-empresa, equipas, utilizadores e perfis](#2-multi-empresa-equipas-utilizadores-e-perfis)
3. [Regras de acesso (record rules) — quem vê o quê](#3-regras-de-acesso)
4. [Localização geográfica portuguesa](#4-localização-geográfica)
5. [Entidade: Prospecto (crm.lead) e Chamadas (crm.phonecall)](#5-prospecto)
6. [Entidade: Imóvel (product.template / product.product)](#6-imóvel)
7. [Entidade: Pessoa — Proprietário / Comprador (res.partner)](#7-pessoa)
8. [Entidade: Visitas (angariação, avaliação, comercial)](#8-visitas)
9. [Entidade: Propostas e Processos de Venda](#9-propostas-e-processos)
10. [Área Financeira: Comissões, Atos, Minutas, Movimentos](#10-financeiro)
11. [RH: Colaboradores, Pontos, Escalas, Tarefas, Recrutamento](#11-rh)
12. [Dashboard, Notificações e Mensagens](#12-dashboard-e-notificações)
13. [Segurança de documentos (PIN) e segurança de login](#13-segurança-de-documentos-e-login)
14. [Fluxos de negócio detalhados](#14-fluxos-de-negócio)
15. [Website público e Área de Cliente](#15-website)
16. [Integrações externas](#16-integrações)
17. [Automações (crons) — tabela completa](#17-crons)
18. [Templates de Email e SMS](#18-templates)
19. [Relatórios e wizards](#19-relatórios-e-wizards)
20. [Comunicação Exterior (motor de campanhas)](#20-comunicação-exterior)
21. [Validações e regras de integridade](#21-validações)
22. [Recomendações para o novo projeto](#22-recomendações)

---

## 1. Visão geral

O sistema atual é um **CRM imobiliário completo** que cobre o ciclo de vida integral do negócio:

```
Captação (scraper OLX / site / manual / WhatsApp)
   → Prospecto (crm.lead) + ciclo de chamadas do Centro Operacional (CO)
      → Visita de Angariação (calendar.event)
         → Imóvel Angariado (product.product, estágio "Gathered")
            → Publicação (site próprio + portais + Habitar)
               → Compradores (res.partner) + contactos/pesquisas + matches
                  → Visita Comercial (commercial.visit)
                     → Proposta (sale.proposal) + contrapropostas
                        → Processo de Venda (sale.process)
                           → Atos: CPCV / Reforço / Escritura / Arrendamento (sale.acts)
                              → Comissões (sale.commissions) + pontos aos agentes
```

Em paralelo existem: gamificação por pontos, escalas de trabalho, tarefas internas, recrutamento de agentes, relatórios semanais de atividade, notificações internas, envio massivo de SMS/email, área de cliente no website, angariação 100% online e módulo "Invest" com acesso restrito.

### Módulos que compõem o sistema

| Módulo | Papel |
|---|---|
| `rdx_remax_crm` | Núcleo do CRM (23.000 linhas): prospectos, imóveis, visitas, propostas, processos, comissões, RH, notificações |
| `triomphe_web` | Website público (montras de imóveis, área de cliente, angariação online, propostas online, equipa) |
| `rdx_base_ext` | Períodos (mês/semana/trimestre), extensões base, chatter |
| `rdx_hr_teams` | Equipas com líder, coordenador e membros (hr.employee) |
| `rdx_sales_team_extended` | Dashboards de equipas de vendas |
| `rdx_pt_addresses_ext` | Hierarquia geográfica PT: Distrito → Concelho → Freguesia → Zona |
| `rdx_product_images` | Múltiplas imagens por imóvel (`product.images`) |
| `rdx_olx_real_estates_crawler` | Crawler OLX/Imovirtual para captação de prospectos |
| `rdx_l10n_pt` | Plano de contas / fiscalidade PT |
| `triomphe-whatsapp` | **Projeto novo já em Django + React** (apps: accounts, billing, conversations, odoo_sync, whatsapp) — arquitetura de referência para o novo CRM |

---

## 2. Multi-empresa, equipas, utilizadores e perfis

### 2.1 Multi-empresa (agências)

- O grupo é composto por várias empresas/agências (`res.company`): **Triomphe Lisboa, Triomphe Leiria / Caldas da Rainha, Triomphe Setúbal, Triomphe Algarve, Triomphe Invest** (a Invest é excluída da lista de "agências" nos dashboards).
- Cada empresa tem **`district_ids`** (distritos de atuação). A atribuição automática de empresa a um prospecto/imóvel é feita **pelo distrito do imóvel**, com exceção hard-coded: concelhos **Lourinhã, Torres Vedras e Cadaval** pertencem à Triomphe Leiria mesmo estando no distrito de Lisboa. Existe função utilitária `get_company_by_location(obj)` que resolve empresa por distrito.
- Campo `do_not_share_real_estates` na empresa: não partilha imóveis com as outras agências.
- Cada empresa configura as **categorias de pontos** próprias (ex.: `visit_made_points_category_id`, `visit_gathered_points_category_id`, `visit_sold_points_category_id`, `weekly_ten_visits_made_category_id`, `weekly_zero_visits_gathered_category_id`, `monthly_ten_visits_gathered_category_id`, `sale_price_reduction_category_id`) — ver §11.
- **Regras multi-company globais** (aplicam-se a todos): equipas, colaboradores, visitas de angariação, visitas comerciais, visitas canceladas e leads só são visíveis se `company_id` for filho das empresas do utilizador (ou vazio).

### 2.2 Equipas (crm.case.section, estendido)

- Campos adicionados: `leader_employee_id` (Team Leader), `coordinator_employee_id` (Coordenadora), `employee_ids` (membros, M2M com `hr.employee`), `member_ids` (utilizadores derivados), flags `sales_team` e `marketing_team` (equipa de operadores CO), `send_gathering_visit_reminder_to_manager`.
- O líder/coordenador definem as cadeias de responsabilidade usadas pelas record rules (ver §3).
- Dashboards por equipa (leads do dia, oportunidades, gráficos mensais).

### 2.3 Utilizadores (res.users, estendido)

Campos/métodos computados centrais para TODA a lógica de permissões:

| Campo | Definição |
|---|---|
| `is_agent` | Está em `base.group_sale_salesman` e NÃO em CO Manager nem em "all leads" |
| `is_operator` | Está em `crm.group_scheduled_calls` (Gathering Operator) e não é manager/agente/distribuidor de leads |
| `is_manager` | Está em `base.group_sale_salesman_all_leads` e não é CO/ERP manager/sale process |
| `is_co` | Operacional: em `crm.group_scheduled_calls` ou CO Manager (e não agente puro) |
| `employee_id` | hr.employee ligado ao user |
| `team_id` | equipa do employee (por empresa) |
| `responsible_user_ids` / `responsible_employee_ids` | **conjunto de pessoas pelas quais o utilizador é responsável**: ele próprio + (se for coordenador de equipas) todos os membros dessas equipas + (se for líder) membros da equipa que lidera. É a base de quase todas as record rules |
| `group_user_ids` | utilizadores das mesmas equipas |
| `mute_notifications` | silencia sino de notificações |
| `logins_ips` | histórico de logins |

Na **criação de um utilizador**:
- É obrigatório preencher a "Default Team" (raise se faltar).
- Se for agente → cria automaticamente `hr.employee` (departamento 1, cargo Agent, parent = líder da equipa) e define ação inicial = Dashboard.
- Se for operador → cria `hr.employee` (departamento 2, cargo Operador id 4) e ação inicial = lista de leads.

### 2.4 Grupos de permissão (perfis)

**Perfis principais (hierarquia comercial):**

| Grupo | Significado prático |
|---|---|
| `base.group_sale_salesman` (renomeado **"Agent"**) | Agente/avaliador — vê apenas o que é seu |
| `base.group_sale_salesman_all_leads` | Gestor/Coordenador — vê a sua equipa (via responsible_ids) |
| `crm.group_scheduled_calls` → **"Gathering Operator"** | Operador do CO (teleprospecção de angariação) |
| `commercial_group_scheduled_calls` → **"Commercial Operator"** | Operador comercial (inclui Gathering Operator) |
| `manager_group_scheduled_calls` → **"CO Manager"** | Gestora do Centro Operacional (inclui os dois anteriores) |
| `base.group_erp_manager` | Administração — vê tudo |
| `sale_process_group` → **"Sale Process"** | Gestão de processos de venda/financeiro (vê todos os processos, propostas e comissões) |
| `group_centro_operacional` | Escrita liberada em crm.lead para o CO |
| `group_crm_co_basic` → "Acesso Básico Propostas" | Gestão comercial com acesso restrito às propostas |

**Grupos de tarefas:** Task Operator, Task Middle, Task Manager (hierarquia com implied groups).

**Grupos funcionais/finos (todos têm de existir no novo CRM como permissões):**

- `co_calls_statistic_report_group` — Tabela estatística Teleprospecção CO
- `gathering_visits_statistic_report_group` — Estatística visitas de angariação por agente
- `rescheduled_gathering_visits_report_group` — Relatório de visitas reagendadas
- `mass_activity_report_group`, `website_report_group`, `indicators_report_group` (recebe email semanal de indicadores)
- `prospects_advert_edit_group` — pode editar descrição do anúncio do prospecto
- `points_edit_group` — pode editar pontos
- `extra_configs_group` — pode editar algumas configurações
- `see_activity_reports_group` — vê relatórios de atividade nos imóveis
- `web_published_edit_group` — pode editar publicação web
- `gathering_visits_quality_control_group` — Controlo de Qualidade
- `invest_contacts_group` — vê contactos/imóveis/propostas Invest
- `hr_recruiter_group` — recrutamento
- `see_rs_documents_group` — vê documentos dos imóveis
- `change_teams_group` — pode mudar equipas
- `change_referred_assistant_group` — pode mudar "Referred Assistant"
- `give_points_employees_group` — pode atribuir pontos
- `delete_documents_rs_group` — pode apagar documentos de imóveis
- `see_all_dashboard_group` — vê dashboard global
- `quality_non_visit_group` — QC de não-visitas
- `create_rs_prospects_group` — pode criar prospectos do CO (implica Gathering Operator)
- `revert_proposals_group` — pode reverter propostas
- `access_proposals_group` — acesso a propostas
- `group_selecionar_avaliador_lead` — pode selecionar avaliador da lead
- `group_external_communication_user` — vê módulo Comunicação Exterior
- `group_ultimo_contato_proprietario` — relatório "Último Contato Proprietário"
- `group_distribuir_leads` — pode distribuir leads (vê todas)
- `group_conversacoes_report_menu` — menu Conversações (relatórios)
- `group_whatsapp_notification_recipient` — recebe notificação interna quando operador usa "Send WhatsApp"
- `group_whatsapp_chat` — acesso ao botão "Conversar WhatsApp" (Triomphe Chat)
- `group_show_chatter` / `group_hide_chatter` — mostrar/ocultar chatter
- `group_hide_sensitive_menus` — bloqueia menus Proprietários/Compradores/Visitas
- `group_documents_security_audit` — vê logs de acesso a documentos sensíveis
- `group_documents_unlock_extended` — desbloqueio prolongado de documentos por PIN

---

## 3. Regras de acesso

Regras de registo (record rules) exatas do sistema atual — **críticas para o novo CRM**:

### Prospectos (crm.lead)
- **Agente:** vê leads não-descartadas onde `user_id` ∈ `responsible_user_ids`, **OU** leads onde `operator_working_id` ∈ `responsible_employee_ids` (lead em que está a trabalhar).
- **Gestor (all_leads):** vê leads da sua equipa (`user_id` ∈ `group_user_ids`) ou sem responsável.
- **CO Manager / create_rs_prospects_group / group_distribuir_leads:** vê todas (distribuir tem CRUD total).
- **Operador (sem distribuir leads):** `search_read`/`read_group` são **forçados** a `phonecall_datetime <= hoje 23:59` e `operator_working_id = eu` — o operador só vê as leads que lhe foram distribuídas para hoje.
- Multi-company global.
- `group_centro_operacional`: read+write em todas (sem create/unlink).
- `group_external_communication_user`: read-only em todas.

### Imóveis (product.product)
- **Agente (leitura):** vê o imóvel se **(é o agente do imóvel E não está "Angariado") OU (está no estágio "Angariado")** — ou seja, *todos veem o stock angariado; imóveis em prospecção só o próprio agente*.
- **Agente (escrita/criação/eliminação):** apenas imóveis cujo `employee_id` ∈ `responsible_employee_ids`.
- **Gestor (leitura):** imóveis sem agente OU da sua equipa não angariados (o stock angariado é global).
- **Invest:** grupo invest vê imóveis `is_invest = True`.

### Pessoas (res.partner)
- **Agente:** vê agências externas + contactos que não são compradores/proprietários + **proprietários** cujo `user_id` (ou do parceiro-pai) ∈ responsáveis + **compradores** cujo `employee_id` ∈ responsáveis. Pode criar/editar compradores (`customer=True`), não pode eliminar.
- **Operador comercial:** CRUD (sem unlink) sobre compradores.
- **Gestor/Operador CO:** cria contactos não-clientes; CO vê todos os partners.
- **Invest:** vê partners `is_invest`.
- Agente **não pode alterar email/telemóvel de cliente com Online ID** (área de cliente) — só admin.

### Visitas
- **Visitas de angariação (calendar.event):** agente vê as suas (`employee_id` ∈ responsáveis); gestor vê todas; multi-company.
- **Visitas comerciais:** agente vê as suas + **leitura** das visitas a imóveis seus (`product_id.employee_id` ∈ responsáveis); gestor vê todas.
- **Visitas de avaliação:** idem angariação/comercial.
- **Relatórios de visita comercial:** agente vê os seus; admin/operador comercial vê todos.

### Propostas e Processos
- **Agente:** vê propostas/processos onde é agente do comprador OU agente do imóvel.
- **sale_process_group + ERP manager:** veem tudo.
- **Invest:** propostas `is_invest`.

### Comissões (sale.commissions)
- **Agente:** só as **suas** comissões e apenas nos estados `paid`/`done` (nunca vê em preparação).
- **sale_process_group / HR manager / ERP manager:** todas.

### Outros
- Contactos de compradores (`res.partner.prospection.message` e respostas): agente vê por empresa; CO vê todos.
- Relatório semanal de atividade: agente/gestor vê os dos seus responsáveis; admin tudo.
- Reuniões semanais e prospetos de RH: por empresa; admin tudo.
- Disponibilidades: agente vê as suas; admin/CO manager todas.
- Tarefas: operador lê as suas (sem write); Task Manager tudo; agente lê as dos responsáveis.
- Pontos: agente vê os seus; gestor vê os da equipa.
- Logs de segurança de documentos: só `group_documents_security_audit` (read-only).

---

## 4. Localização geográfica

Módulo `rdx_pt_addresses_ext` define a hierarquia obrigatória em todo o sistema:

```
res.country → res.country.state (Distrito) → res.country.county (Concelho)
   → res.country.parish (Freguesia, com zip de 4 dígitos) → res.country.zone (Zona)
```

Comportamentos transversais (repetidos em lead, partner, imóvel, mensagens de pesquisa):
- Preencher **zip** → resolve automaticamente a freguesia (primeiros 4 dígitos); erro se não existir.
- Selecionar zona → preenche freguesia; freguesia → concelho + zip; concelho → distrito; distrito → país. Alterar um nível superior limpa os inferiores inconsistentes.
- Endereço completo composto: rua + porta + andar + fração + freguesia/concelho + distrito + zip.
- Existe wizard para **remover localidades duplicadas** e wizard para **definir preço por m² por freguesia**.
- `product.parish.price.per.square.meter`: preço/m² por freguesia × categoria × condição do imóvel, com histórico por trimestre → alimenta o **valor estimado** de imóveis e prospectos (área × preço/m² mais recente) e o relatório "Sugestão de Preço". Quando o preço de referência muda, os agentes com imóveis nessa freguesia/condição recebem email de "valor de referência alterado".

---

## 5. Prospecto

### 5.1 Modelo `crm.lead` (herdado — "Proprietor/Landlord Prospect")

O prospecto representa **um proprietário potencial com um imóvel para angariar**. Na criação, o sistema **cria automaticamente um imóvel rascunho** (`product.product`) ligado à lead (a menos que se passe um imóvel existente). Todos os campos `prd_*` da lead são *related* para o imóvel — editar na lead escreve no imóvel (e o sistema garante sincronização do endereço em qualquer escrita, `_certify_real_estate_address`).

**Campos próprios principais:**

| Grupo | Campos |
|---|---|
| Identificação | `name` (computado: "Origem:AdvertID - Contacto (Imóvel)"), `contact_name`, `partner_id` (Proprietor), `origin_id` (origem: OLX, Imovirtual, FSBO's, site…), `advertID` (id do anúncio, **único por origem**), `web_url`, `import_information` |
| Estado | `prospection_state`: **draft (Por Validar) / pending (Por Fazer) / no_answer (Não Atendido) / reschedule (Adiado) / sent (Enviado) / received (Recebido) / answer_accepted (Concluído) / disposed (Descartado)** |
| Descarte | `disposed_reason` (é agente / cancelado / arrendou / sem efeito / já vendido) + `disposed_description`; `reschedule_reason` (angariado / cancelado temporariamente / pediu para ligar depois) |
| Contadores | `cancelled_reason_counter`, `rented_reason_counter`, `sold_reason_counter`, `unanswered_counter`, `emails_sent`, `phonecalls_len`, `gathering_visits_count` |
| Contacto | `phone`, `mobile`, `mobile2`, `email_from`, `email2`, `lang` (pt_PT/en_US/fr_FR), morada completa (com zona/freguesia/concelho), `old_address`, `has_whatsapp`, `whatsapp_opt_out`, `preferred contact` |
| Atribuição | `user_id` (Responsável/avaliador), `employee_id` (agente FSBO), `section_id` (equipa), `operator_working_id` (operador a trabalhar), `share_with_co`, `company_id` (obrigatória; inversa escreve `create_company_id` no imóvel) |
| Chamadas | `phonecall_ids`, `phonecall_id` (pendente ativa), `phonecall_datetime` (data agendada, computada/inversa), `last_phonecall_date`, `last_request`, `contacted_today`, `never_answered_warning`, `not_answered_pending_reschedule`, `last_sms_sent` |
| Automáticos | `auto_mail_sent/date`, `auto_sms_sent/date`, `website_email_sent/date`, `comunicacao_pos_visita_enviada`, `ja_teve_contato` |
| Scripts | `script_id` + `approach_script` + `objection_script` (guiões de chamada HTML ordenáveis) |
| Duplicados | `duplicate_lead_ids` (procura por telefone/telemóvel via SQL) |
| Privado | `private_message` (nota privada **por utilizador** — tabela `lead.private.message`, upsert por lead+user) |
| Imóvel (related, ~60 campos) | tipo, tipologia T0–T9+, condição, tipo terreno, certificado energético, ano construção, morada completa, nº quartos/WC/pisos/roupeiros, box/estacionamento, ~28 comodidades boolean (elevador, varanda, suite, piscina, vista mar…), áreas (útil/bruta/terreno), itinerário, observações internas, descrição pública, imagens, negócios (venda/arrendamento c/ valores), `prd_estimated_price` (área × preço m² freguesia), `inhabited_by_owner` |
| **Congelamento** | ~75 campos `fixed_*` que fotografam imóvel + proprietário no momento da conclusão (`freeze_date`). Ao concluir prospecção (`btn_conclude` ou criação de visita), `_freeze_related_fields()` copia todos os valores correntes para os `fixed_*` — histórico imutável |
| Valores | `max_sale` / `max_rent` (máximo dos negócios de venda/arrendamento — pesquisáveis via SQL) |
| Visitas | `has_visita_angariacao` (computado+pesquisável: existe calendar.event em estados agendada/confirmada/draft/not_made/made_*) |
| Sugestão horários | `suggested_times`: tabela HTML semanal (9h–19h, sem domingos) que cruza ocupação dos agentes da zona (escalas, visitas de todos os tipos, atos, indisponibilidades) e pinta livre/parcial/ocupado, destacando a melhor hora |

**Validações na criação/edição:**
- Telefone/telemóvel: remove espaços; avisa se já existe lead com o número; avisa "Is agent" se houver lead descartada como agente com esse número; avisa se número já pertence a comprador/proprietário.
- `advertID` único por origem (constraint + onchange).
- Tipologia ↔ nº quartos sincronizados (T3 → 3 quartos e vice-versa; >9 → T9+).
- Se criador é operador → `operator_working_id` = ele próprio.
- Se novo prospecto tem email/telemóvel repetido noutras leads ativas → **notificação interna às Gestoras CO** (com botão "copiar telefone").
- Escrever contacto na lead com partner ligado **propaga para o partner** (email, telefones, morada, nome) e regista log de alteração de nome no chatter da lead e do imóvel.
- Utilizador com cargo id 4 (operador) **não pode editar** `prd_public_description` depois de criado.
- `default_get`: agente/gestor a criar lead recebe-se a si próprio como employee/user/section.

**Botões e ações da lead (todos têm de existir no novo CRM):**

| Botão | Comportamento |
|---|---|
| **Atendida** (`btn_call_answered`) | Abre wizard "phonecall → visita de prospecção" pré-preenchido com dados do imóvel (negócio, valor, morada). Deste wizard sai: agendamento de visita de angariação, envio de email, etc. |
| **Não Atendida** (`btn_call_not_answered`) | Anti-duplo-clique (<60s). Fecha pendentes, marca chamada como `no_answer`, cria pendente para +1 minuto, incrementa `unanswered_counter`, marca `not_answered_pending_reschedule`, envia **SMS "Liguei-lhe por causa do seu imóvel"** (template `nao_atendido_sms`; se operador de cargo 4, o sender é o telefone de trabalho do próprio operador), regista tudo no chatter. O **cron das 22h** reagenda todas estas leads para **+5 dias às 09:00** |
| **WhatsApp** (`btn_whatsapp`) | Abre wizard equivalente ao "Atendida" mas por WhatsApp; marca `has_whatsapp` |
| **Send WhatsApp** (`btn_send_whatsapp`) | Wizard que envia instrução ao Gestor CO (grupo `group_whatsapp_notification_recipient` também notificado) |
| **Conversar WhatsApp** (`btn_open_whatsapp_chat`) | Gera **JWT assinado** e abre o Triomphe Chat (webapp externa, URL em `whatsapp.chat_url`). Bloqueado se `whatsapp_opt_out` ou sem telemóvel; exige grupo `group_whatsapp_chat` |
| **Anexos WhatsApp** | Lista `ir.attachment` com prefixo `[WhatsApp]` da lead |
| **Email Website** (`btn_website_email_sent`) | Marca envio, agenda follow-up +5 dias (ou +3 se sexta) |
| **SMS Recebido** (`btn_sms_received`) | Abre registo de chamada pré-preenchido (received/sms) |
| **Email Recebido** | = Atendida |
| **Agendar Visita** (`btn_schedule_visit`) | Abre criação de visita de angariação com defaults (nome = contacto + imóvel + freguesia) |
| **Concluir** (`btn_conclude`) | Congela campos + estado `answer_accepted` |
| **Descartar** (`btn_dispose`) | Estado `disposed` + desativa o imóvel |
| **Repor descartada** (`btn_reset_disposed`) | Limpa responsável, reativa imóvel, recria chamada pendente |
| **Reatribuir** (`btn_reassign_lead`) | Reagenda pendente para agora |
| **Imprimir Avaliação** (`btn_print_evaluation_report`) | PDF de avaliação |
| SMS lembrete (`send_sms_delayed`) | SMS agendado para as 20h "Vai angariar online" + follow-up +5 dias |

**Mecânica das chamadas pendentes (motor da teleprospecção):**
- Cada lead tem no máximo **uma chamada pendente aberta** (`_create_pending_prospection_followup` fecha as anteriores com mensagem automática e cria a próxima com rótulo de origem ex.: "Reagendado pelo botão X em …").
- `_update_phonecall_id` mantém `prospection_state` da lead sincronizado com a última chamada (pendente → lead pending; concluída → estado da chamada).
- `_needaction_domain_get`: operadores CO veem contador de leads com chamada agendada para hoje.

### 5.2 Chamadas `crm.phonecall` (herdado)

- Campos: `prospection_state` (pending/no_answer/reschedule/answer_accepted/disposed/sent/received), `message` (HTML resumo), `message_in_out` (sent/received), `contact_method` (**sms/email/call/whatsapp**), `whatsapp_agent_name`, `reason` (é agente, angariado/cancelado, vendido, arrendado, sem efeito, cancelado temporário, pediu ligar depois, sem nº/email, email arrendamento, email reforço…), `product_id` + freguesia/concelho/distrito (related, stored), `is_external_automatic` (enviado pela Comunicação Exterior), `stype` (partner_prospection).
- Campos ficam **readonly quando done**.
- `btn_done` (Recebido): fecha pendentes; se SMS recebido → follow-up amanhã 09:00; senão → +5 dias 09:00 (+1 se sexta); se enviado → atualiza `last_request`.
- Categorias de chamada: "Prospection" (dados fixos).
- API para o Triomphe Chat: `whatsapp_chat_lead_snapshot`, `whatsapp_chat_find_lead_by_mobile` (match por últimos 9 dígitos), `whatsapp_chat_create_phonecall` (regista mensagem WhatsApp in/out como chamada), `whatsapp_chat_create_attachment` (anexo na chamada + cópia na lead/partner com prefixo `[WhatsApp]`), `whatsapp_chat_visit_reminder_context` (avaliador + telefone p/ templates).

### 5.3 Captação automática (scraper)

- `_cron_scrape_competition`: lê ficheiro `/usr/local/sbin/scrapper/worked_imoveis` (gerado pelo crawler OLX/Imovirtual), filtra concelhos alvo (lista fixa de ~35 concelhos de Leiria/Lisboa/Setúbal), deduplica por advertID/URL, resolve empresa por distrito e cria leads com tipologia/preço/negociável/freguesia.
- Módulo `rdx_olx_real_estates_crawler` inclui projeto Scrapy + wizard manual.
- Origens de lead são geríveis (`crm.lead.origin`, com flag `is_web_page`): FSBO's, OLX (automatizado), etc.

### 5.4 Importação e gestão em massa

- Wizard **Importar Leads** (ficheiro), wizard **Atribuir Leads** (manual e automático com configuração por tipo de imóvel/distrito — `crm.assign.lead.config` com linhas por tipo e distrito, log `crm.lead.assign.log`), wizard **Distribuir/transferir/desatribuir operadores** (`crm.lead.operator.*`), wizard **Descartar em massa**, **agendar chamadas em massa**, **SMS em massa**, **email em massa** (`crm.mass.email.leads`), **mover imóveis em massa** (`product.move.mass`).

---

## 6. Imóvel

### 6.1 `product.template` / `product.product` = Imóvel

**Identificação e classificação:**
- `ref` (referência interna — **não pode começar por "TPH"** exceto admin/processo automático `tph_auto`), `name`, `categ_id` (Tipo: Apartamento, Moradia, Loja/Escritório, Terreno/Quinta, Casa de Férias, Garagem/Parqueamento, Estabelecimento Comercial, Quarto para Arrendar, Prédio — **categorias filtráveis por empresa**), `typology` T0–T9+, `condition_id` (condição, geríveis), `energy_certificate_id` (+ `quality_order`, data fim, código), `property_category` (moradia simples/geminada/em banda/andar moradia), `terrain_type` (rústico/urbano/outro), `construction_year`, `condo_expense`.

**Estágios (`product.stage`, com kanban + criação automática de menu/ação por estágio):**
`In Prospection` → `Por Validar` → `Angariação Online` → **`Gathered` (Angariado)** → `Reserved` → `Sold` / `Let` → `Canceled`.
- `product_operation`: default / gathering_visits_gathered / done_deal.
- Histórico de mudanças de estágio (`product.stage.changes`) e de preço (`product.business.changes` com de→para e data).

**Negócios (`product.business`):** tipos Sale / Rent / Exchange (geríveis), único por tipo+imóvel, com `amount`, `negotiable`. Alterações de valor:
- registam log no imóvel e nas leads ligadas;
- **descida de preço** → notificação interna a todos os agentes/coordenadores/gestoras (jobs 1, 6, 3 + employee 60) com "Redução de preço em REF de X€ para Y€";
- **email automático aos compradores interessados** (template `reducao_valor_comprador` com nome do imóvel, valor antigo/novo e link) — compradores com contacto registado nesse imóvel e não "concluídos";
- redução ≥ 5.000€ (CO) ou ≥ 10.000€ (agente) em imóvel angariado à venda → **pontos** ao colaborador (categoria de redução de preço da empresa);
- alteração de valor dispara email "Real estate value changed" aos agentes com visita marcada.

**Endereço e geo:** morada completa PT + `maps_lat/lng`, link/widget Google Maps, `full_address` computada.

**Características:** nº quartos, WC, pisos, roupeiros, box (nº carros), parqueamento, ~28 comodidades boolean, exposição solar (N/S/E/O, O2M), divisões (`product.divisions`: tipo de divisão + área, ordenáveis), áreas útil/bruta/terreno/implantação, `number_specifications` e `highlighted_specifications` computados, `qr_code` gerado.

**Angariação/contrato:** `partner_id` (Proprietário), `lead_id` (prospecto de origem), `employee_id` (Agente angariador — inverse), `co_scheduled_id` (Referred Assistant), `evaluator_id` (Avaliador — com `can_see_evaluator`), `rs_is_from_company`, `gathering_date`, `gathering_end`, duração e dias-para-fim computados, `gathering_type` (seleção parametrizada — aberto/exclusivo), `commission_type` (percentagem/valor fixo), `commission_percentage` (default 5%), `percentage_paid_reinf` / `percentage_paid_cpcv` (default 50%) / `percentage_paid_deed` (computado como resto), `commission_observations`, `no_share`, `keys_identifier` (ID das chaves), `on_display` (montra), `display_board` (placa: interior 70x70 / board 100x70 / autocolante), `inhabited_by_owner`, `published_by_owner` + `published_price` + data (anúncio do proprietário).

**Documentação legal:** `conservatory_land_register` + número, `matrix_article` + validade, `property_logbook` (caderneta) + datas; checklist booleana de documentos com flags readonly por documento: **CMI, Caderneta Predial, Certidão Registo Predial, Licença de Utilização, Certificado Energético, Ficha Técnica, Certidão Registo Comercial, Plantas, Cartão de Cidadão**; `documents_verified` (waiting/verified) + `validation_datetime` / `cancellation_datetime`; anexos protegidos `files_test` (M2M attachments) com **segurança por PIN** (ver §13).

**Preços:** `sale_price`, `rent_price`, `sale_price_per_m2`, `estimated_price` (+ por m²), `sale_estimated_price_diff` (+ %), `internal_price`, formatações `*_fmt`.

**Publicação/web:** `published` (waiting/published), `website_published` (módulo site), `publication_quality` + `publication_quality_tips` (score de qualidade do anúncio com dicas), `tracking_ids` (cliques web com IP, geo-IP país/cidade, likes, verificação anti-bot contra IPs de logins internos e blacklist de datacenters), `total_requests_received`, `last_request`, `youtube_link`, `virtual_visit`, `highlighted`, `portal_link`, `online_id` (angariação online).

**Imagens (`product.images` de `rdx_product_images`):** múltiplas imagens ordenáveis por imóvel + **wizard de marca d'água** (aplica logotipo Triomphe centrado a 25% da largura, 80% opacidade, corrige EXIF; permitido apenas a Angariador do imóvel/Gestora (job 3)/Coordenadora (job 6)/Admin; notifica coordenador + employee 60 e regista no chatter).

**Externos/partilha:** `external_real_estate` + `external_agency_id` + `external_real_estate_state` (available/transacted/cancel) — imóveis de agências parceiras.

**Invest:** `is_invest`, `rating` A–F, `rental_yield`, `resale_yield`, títulos/descrições multilíngue (PT/FR/EN), `web_published_invest`, estágio próprio.

**Relações:** visitas de angariação/avaliação/comerciais + relatórios (contadores), propostas (contador exclui waiting/not_approved), conversas do proprietário (visíveis ao avaliador do imóvel/grupos autorizados), matches de compradores (`desired_partner_ids` via `btn_matched_partner`), imóveis semelhantes por preço (`similar_products`), atividade mensal (`activity_report_ids`).

**Empresa:** `create_company_id` (entidade criadora, obrigatória), `rs_company_id` (computada/stored), `company_id` "In Agency Area" (por distrito).

### 6.2 Workflow do imóvel (botões)

| Botão | Efeito |
|---|---|
| `btn_real_estate_reserved` | → Reservado |
| `btn_real_estate_cancel_reserve` | volta a Angariado |
| `btn_real_estate_sold` / `btn_real_estate_let` | → Vendido / Arrendado (done deal; dá pontos ao operador da visita angariada via categoria "visit_sold" da empresa) |
| `btn_real_estate_cancel` | → Cancelado (wizard com motivo) |
| `btn_rollback_to_gathered` | reverte para Angariado (wizard) |
| `btn_real_estate_in_prospection` | volta a prospecção (com motivo) |
| `btn_verify_documents` | valida documentos → `notify_sms`: notificação interna + **SMS** ("imovel_validado_s_id"/"imovel_validado_c_id") ao angariador, coordenador, gestoras (job 3) e employee 60; se a visita origem foi criada por operador, notifica as gestoras identificando o operador |
| `btn_external_real_estate_*` | transacionado / cancelar reserva / repor disponível (externos) |
| `btn_schedule_visit` | agendar visita de angariação (exige lead registada) |
| `btn_print_report` | PDF apresentação do imóvel |
| `btn_print_market_study` | PDF estudo de mercado |
| Habitar: enviar / reenviar / remover / testar ligação | ver §16 |

**Match comprador↔imóvel (`match_value`):** score começa em 25; +8 por elevador/suite/varanda/cozinha equipada/estacionamento suficiente/WC suficientes; +3 por cada outra comodidade pedida e presente; +15 distrito desejado; +20 concelho; +25 freguesia; cap 100.

**Crons do imóvel:** lembrete de fim de angariação (`_cron_remind_gathering_end`), match automático de compradores (`_cron_match_partners`, desativado), verificação "RS from company" diária, criação mensal de cliques para relatórios de atividade.

### 6.3 Mapa de imóveis

`real.estates.map` — vista de mapa com todos os imóveis (menu "Real Estates Map").

---

## 7. Pessoa

### 7.1 `res.partner` (herdado) — Comprador e/ou Proprietário

Uma pessoa pode ser **Comprador/Inquilino** (`customer`) e/ou **Proprietário/Senhorio** (`proprietary`), cada papel com o seu pipeline:

- **Estágios de proprietário** (`res.partner.stage`, type proprietary): In Prospection → Qualified → **Linked** (com imóvel angariado) → Concluded → Archived. Atualização **automática diária** conforme estágios dos imóveis do proprietário (angariado/reservado → Linked; angariação online → Qualified; só prospecção → In Prospection; vendidos/arrendados → Concluded; cancelados → Archived).
- **Estágios de comprador** (type customer): In Prospection → In Negotiation → Concluded (buyer_operation `done`) → (+Archived). Crons: comprador sem contacto há 180 dias → arquivado; arquivado há 7 dias → email automático "archived leads"; comprador com `financing_required` respondido → passa a In Negotiation; 3 dias após 1º contacto sem visita/proposta → email tipo "E".
- Kanban agrupado por estágio com `_group_by_full` (mostra todos os estágios).

**Campos principais (além do Odoo base):**
- Papéis e ligações: `customer`, `proprietary`, `partner_parent_id`/`partner_child_ids` (associados, ex.: cônjuge co-proprietário — validação anti-auto-referência), `married_to`, `connection` (cônjuge/irmão/procurador/fiador/…), `is_company`.
- Contacto: `mobile`/`email`/`email2` (tracked), `preferred_contact_way` (phone/email/whatsapp), `whatsapp_opt_out`, `lang` auto por país (PT→pt, GB→en, FR/CH/BE→fr), `spoken_languages_ids` + `available_employee_ids` (funcionários compatíveis por idioma).
- Dados pessoais/KYC (para processos): `birth_date`, `gender`, `marital_status` (7 regimes), `occupation`, `born_place`, `nationality_county_id`, documento (`card_type` passaporte/CC/título residência/carta condução/BI/outro + entidade emissora, número, datas), `vat`, `nipc`, `pcc` (certidão permanente), `nib`/IBAN + `bank_id`, `informations_completed` ("N / 12").
- Comercial: `employee_id` (Agente responsável — default agente atual se cargo de agente), `origin_id` (fonte crm.tracking.source com flag active), `created_by_agent`, `contact_form` (entrou na agência/ligou/email/prospecção/pedido website), `contact_date`, `scale_period` (manhã/tarde), `do_not_send_activity_report`, `financing_required`.
- Pesquisa desejada (nível partner, espelho do que existe nas mensagens): tipos de imóvel, condições, min/max quartos/áreas, tipo de negócio, `value_willing_to_invest`, localização desejada (país→zona), `show_only_gathered_real_estates`, **`desired_matched_product_ids`** (matching em tempo real com domínio composto — valor até +10% do orçamento) + contador.
- Agregados de prospecção (computados/stored, a partir das mensagens): distrito/concelho/freguesia/país "principais", strings com todos os distritos/concelhos/freguesias de interesse, tipo de negócio de interesse, maior valor disposto a pagar.
- Área de cliente online: `online_id` (10 dígitos único), código de mudança + data, IP de login, datas de último/penúltimo login, tentativas; `online_gathering_stage_ids` (fases de angariação online).
- Agências externas: `is_external_agency` (VAT único), `from_external_agency` + `external_agency_id` (angariador externo), `agency_group_id` (`agency.group` com nome único), `legal_name`, estágio próprio (`res.agency.partner.stage`).
- Invest: `is_invest`, `invest_access` (+ geração de Online ID + email), `invest_employee_id`, `gen_code`, `letter_sent`/data, `building_certificate_file`.
- Vendas: propostas feitas/recebidas + contadores, processos participados + contador, contadores de visitas de angariação/comerciais.
- Conversas: `conversation_ids` (ver 7.3), `last_conversation_datetime`.
- `automatic_followup_enabled` (default True) — opt-out do follow-up automático.
- `process_color` computada para kanban.

**Comportamentos:**
- Na criação/alteração de comprador com agente diferente do criador → **email "new_lead" ao agente** (ou ao coordenador da equipa de vendas da empresa se o agente for virtual) com nome/telemóvel/email e link; gestores de processo (employees 21/22 no sale_process_group) também recebem.
- `_needaction`: contador de compradores com resposta "following" cuja data de seguimento é hoje (agente/CO).
- `name_get`: acrescenta "(Agência)" ao nome de angariadores externos.
- Emails transacionais próprios: envio de código de acesso (PT/FR/EN), recuperação de ID, geração de ID de cliente, "informação recebida" (pedido de pesquisa sem resultados) + notificação interna aos employees 22/60; SMS de recuperação de ID.

### 7.2 Contactos de comprador — `res.partner.prospection.message` (+ respostas)

Cada interação recebida de um comprador é um registo "Contacto":

- `message_type`: pedir **informação** ou pedir **visita**; `info_type`: imóvel específico vs **pesquisa**; `product_id` (imóvel angariado), `origin_id` (fonte), `contact_form` (agência/chamada/email/prospecção/website), `contact_datetime`, `contact_details`, `during_scale_period` + `scale_period` (morning/afternoon/weekend), `discarded`, `employee_id` (agente — computado), `activity_report_id`.
- Se for pesquisa: réplica completa dos critérios (tipos, condições, quartos, áreas, negócio, valor, comodidades precisadas — ~24 flags `need_*` com `choose_and_or` interseção/união, parqueamento e WC mínimos, localizações desejadas **multi-linha** via `filtro.condicao.localidade` (país/distrito/concelho/freguesia por linha)) + **matched products** computados.
- `proposal_amount`, `requested_visit`.
- **Respostas** (`res.partner.prospection.message.response`): datetime, resultado (**visita agendada / em seguimento / não interessado**), `failed_contact` (não atendeu), `follow_date`, `commercial_visit_id`, criador; estado computado (visita/seguimento/não interessado/não atendido/visita cancelada). Resposta "não interessado" descarta o contacto. Botão "Agendar visita" abre visita comercial pré-preenchida.
- Ao registar contacto: **email automático ao comprador** conforme tipo (templates `lead_email_e` — imóvel específico/visita; `lead_email_d` — pesquisa com concelho; `lead_email_da` — pesquisa sem zona), multilíngue, reply-to do agente; não envia para compradores de agências externas.
- `last_followup_sent` — controlo do follow-up automático.

**Follow-ups automáticos de compradores** (crons):
- `_cron_send_automatic_buyer_followup`: 10h–19h, compradores em estágios 1–2 com email, sem contacto há ≥7 dias e sem follow-up há ≥7 dias → email `followup_buyer` + regista contacto "Follow-up automático" (máx. 20/dia).
- Relatório **"Follow Up Leads"** semanal por avaliador (tabela HTML com comprador, data, objetivo, imóvel, forma, detalhes, data seguimento, estado, dias desde último contacto) → email individual a cada avaliador ativo (job 1, exclusões configuradas), resumo agregado à coordenadora, log de execução para o responsável técnico; parâmetro de redirect para testes.
- `_cron_send_agent_contact_buyer_reminder_email`: compradores criados há <90 dias cujo último contacto foi há exatamente 5 dias úteis → notificação interna ao agente "You have not contacted X in 5 days".

### 7.3 Conversas — `res.partner.conversation`

Registo de conversas com proprietários/compradores: `message` (HTML), `message_in_out`, `contact_method` (**sms/email/call/website/presencial/whatsapp**), datetime, `client_read` (lida pelo cliente na área online), `is_external_automatic`. Mensagens do website marcam-se automaticamente. Integração WhatsApp cria conversas automaticamente (`create_from_whatsapp`).

---

## 8. Visitas

Três tipos de visita + agenda unificada.

### 8.1 Agenda unificada (`triomphe.event` — vista SQL)

Junta num só calendário: Visitas de Angariação (ativas, não reagendadas), Visitas Comerciais (agente + "acompanhar" para o agente do imóvel), Visitas de Avaliação, **CPCV** e **Escritura** (por gestor financeiro, agente do imóvel e agente do comprador) e **Escalas**. Agente vê só os seus responsáveis; gestores/CO veem tudo (vista de calendário própria para managers). Criação direta bloqueada ("use os menus próprios").

### 8.2 Visita de Angariação (`calendar.event` herdado, type='gathering')

**Campos:** empresa, `employee_id` (Agente) + user, `section_id` (Equipa), `product_id` (obrigatório), `partner_id` (related do imóvel), `lead_id` (Prospecto), `phonecall_id` (chamada origem), `operator_id`/`operator_name` (operador que agendou — user da chamada ou criador), `gathering_state` (fases: criação → definir equipa → definir agente → pronto), **`visit_state`: draft (Agendada) / made_and_gathered (Realizada e Angariada) / made_but_not_gathered (Realizada não Angariada) / not_made (Não Realizada)**, `visit_survey`, `suggested_value` (valor sugerido na visita), observações + info adicional, reagendamento (`recheduled`, contador, motivo, visita nova ligada, data), `date_done`, `visit_confirmed` (yes/no), `confirmed_by_manager`, `visit_not_done_type` (falta do agente / falta do proprietário), `calendar_title` (nome + operador), itinerário/concelho/habitado (related), maps.

**Controlo de qualidade (QC) embutido:** `qc_visit_done`, `qc_observations`, `qc_visit_rescheduled` (+ por quem/quando), `qc_visit_refused` (+ motivo), `qc_aware_of_conditions` (proprietário conheceu condições Triomphe?), `qc_presentation_file_delivered` (entregou dossier?), `qc_property_survey`, `qc_market_study` (impresso), `qc_sales_values`, `qc_quality_commitment`, `qc_partners` (portais impressos), `qc_reschedule_visit` + nova data, `qc_done`.

**Criação de visita (fluxo crítico):**
1. Validação: não-gestores não podem criar no passado (margem 5 min).
2. Se vier de lead: usa imóvel da lead; se lead sem partner → **cria o partner proprietário** a partir da lead (`_lead_create_contact`: copia contactos/morada, `proprietary=True`, `created_by_agent`).
3. Chamada de origem → marcada como `answer_accepted` concluída.
4. Propaga equipa/agente para lead, partner e imóvel (partner do imóvel + employee).
5. **Congela** os campos da lead (`_freeze_related_fields`) e conclui a prospecção.
6. Notificações (módulo `visits_notifications`): SMS ao proprietário, notificação/email às gestoras+coordenadora (`receive_creation_gathering_visit_email` por empresa, gestoras CO job 3, coordenadora da empresa), tudo com **log de comunicação no chatter** (canal, template, destinatário, ENVIADO/FALHA/SEM_EMAIL/SEM_TELEFONE).
7. Agente que cria → empresa/equipa dele.

**Atribuição em cascata:** definir equipa → email ao líder da equipa; definir agente → **email ao agente com a Ficha de Visita em PDF anexa** + itinerário + descrição, e **email ao proprietário** (template `scheduled_visit_email` multilíngue); mudanças de dados relevantes reenviam "visita alterada" com PDF (com throttle de 1 min); grelha de **disponibilidade dos agentes da equipa** (tabela HTML verde/amarelo/vermelho cruzando: fora da zona de ação (freguesias/concelhos do agente), escala, indisponibilidade, outras visitas de qualquer tipo, atos agendados) — igual nas 3 modalidades de visita.

**Botões de resultado (via wizards com motivo):**
- **Confirmar** (`btn_confirmed_visit`) → `visit_confirmed=yes` + SMS ao agente (template `confirmacao_visita_angariacao`).
- **Não confirmada**, **Cancelar QC**.
- **Realizada e Angariada** (`real_estate_gathered_wizard`): passa imóvel para estágio "Angariado" (via `product_operation`), atribui pontos (categoria visita angariada), etc.
- **Realizada mas não angariada** / **Não realizada** (wizards próprios com razões; tipo de não-realização).
- **Reagendar** (`btn_reschedule` + wizard): agente só pode reagendar **uma vez**; cria nova visita ligada, marca antiga como `not_made` + "(Visit Rescheduled)", contador+motivo, notifica.
- **Cancelar** (`btn_cancel_gathering_visit`): limpa responsável da lead, recria chamada pendente se necessário e **apaga** a visita (SQL delete); log em `visit.canceled` (data, por quem, agente, equipa, proprietário, motivo, empresa) para o relatório de canceladas.
- **Repor rascunho** (`btn_set_draft`): reverte imóvel para estágio default, apaga pontos atribuídos, repõe responsáveis.
- Lembrete de preenchimento: email ao agente (CC ao gestor se equipa configurada) "ficha por preencher — o não preenchimento origina reagendamento".
- Aprovação: `visit_notify_manager` — notificação + email ao chefe do agente ("visita à espera de aprovação") e ao employee 60.
- Pós-visita: crons de **comunicação pós-visita realizada/não realizada** (emails ao proprietário) e **SMS de véspera** (desativáveis).

**Restrições de edição:** utilizadores fora de "all leads" **não conseguem alterar datas** por write direto (campos removidos silenciosamente) — só via wizard de reagendamento.

### 8.3 Visita de Avaliação (`evaluation.visit`)

Estrutura análoga à comercial: empresa, agente (default próprio se agente), equipa computada, imóvel angariado (domain estágio 2), `product_employee_id`, datas início/fim (fim default +30min), observações obrigatórias, estados **draft (Agendada) / made / not_made / cancelled**, relatórios (`evaluation.visit.report`), motivos de não-realização/cancelamento, contacto de origem, reagendamento, confirmação (+observações), `verified`, criador, disponibilidade dos agentes. Na criação: se proprietário sem agente → atribui; liga resposta de contacto ("visita agendada"); integra relatório semanal. SMS de confirmação/cancelamento ao agente. Notificações dedicadas (`visit_evaluation_notifications`).

### 8.4 Visita Comercial (`commercial.visit`)

Visita de comprador a imóvel angariado.

**Campos:** empresa (obrig.), `employee_id` (Agente que acompanha, obrig.), equipa computada, criador, `external_visit` (comprador de agência externa), `product_id` (angariado ou externo, obrig.), `product_employee_id` (agente do imóvel), `partner_id` (comprador, obrig.), datas (fim default +30m), observações obrigatórias, estados **draft/made/not_made/cancelled**, relatórios, motivos, contacto origem, reagendada, confirmação, verified, disponibilidade, guião (script id 2) com approach/objection.

**QC da visita comercial:** `appreciation_employee_info`, `appreciation_expectations` (+flag "no"), `appreciation_other_rs_proposal`, `appreciation_bellow_value_proposal`, `appreciation_reconsider_proposal`, `qc_done` computado.

**Criação:**
- **Obriga a existir contacto registado** do comprador ("Please register the contact… use 'Regist New Contact'").
- Se o comprador ainda não tem contacto para este imóvel → cria automaticamente mensagem de contacto tipo "visita" + resposta "visita agendada"; senão anexa resposta ao contacto existente.
- Comprador sem agente → atribui o agente da visita (se não externa); empresa do comprador ← empresa do agente.
- Notifica responsáveis (`visit_commercial_notifications`) se a visita for futura.
- **Anti-conflito:** constraint impede duas visitas draft do mesmo agente ao mesmo imóvel em períodos sobrepostos.
- Datas só mudam pelo botão "reschedule visit".

**Emails:** ao agente (novo agendamento, com dados do comprador), ao comprador (com contactos do agente; reply-to conforme perfil), ao **agente do imóvel** ("acompanhar visita"; exclui agentes virtuais 196/197/198/257).

**Relatório de visita comercial (`commercial.visit.report`):** por visita — impressões do comprador, `make_proposal` (fez proposta? sim/não/relatório não preenchido), `would_buy`, observações; alimenta o relatório semanal do agente e as estatísticas. (Idem `evaluation.visit.report`.)

**Wizards de resultado:** confirmada / não realizada / cancelada (com motivos), inquérito de visita (`visit_survey_wizard`), reagendamento.

---

## 9. Propostas e Processos

### 9.1 Proposta (`sale.proposal`)

**Campos:** `product_id` (imóvel angariado ou externo), `partner_id` (comprador), agentes de ambos os lados (`product_employee_id`, `partner_employee_id` + flags "sou eu"), `business_type_id`, `proposal_amount`, `proposal_datetime`, **estado**: `waiting` (aprovação) → `not_approved` | `pending_proprietor` ⇄ `pending_buyer` (contrapropostas) → `accepted` | `cancel_proprietor` | `cancel_buyer` → `process` (processo aberto); condições de pagamento (financiamento sim/não + banco + valor, prazo escritura, sinal + data, reforços 1/2 + datas, valor entrega final computado), condições de comissão divergentes (`different_conditions`, tipo/valor proposto, `product_commission_accepted` — divergência exige **aprovação dos admins** users 7/8 com email+notificação), `observations` (máx. 600 chars), `client_notes`, `online_id` (proposta online), **código de validação** (`validation_code` enviado por email ao comprador; `buyer_code` introduzido; `code_confirmed`), `verified`, `is_invest`, `external_buyer`, anexos (contador; **ações exigem anexo** — `check_counter` bloqueia botões sem ficheiro anexado, exceto admin), `is_restricted` (grupo básico), movimentos.

**Movimentos (`proposal.movement`):** histórico de valores — original / correção / contraproposta do proprietário / contraproposta do comprador / cancelado / aceite, com valor e data.

**Fluxo:**
1. Proposta nasce (online → `waiting`; manual CRM → `pending_proprietor` com notificações: internas+email a employees 21/22/60, agente do comprador + coordenador, agente do imóvel; **email ao proprietário** template `nova_proposta_recebida` com Online ID gerando área de cliente se necessário; log de despachos no chatter).
2. **Aprovar** (`btn_approve_proposal`): → pendente proprietário + emails de aprovação ao comprador (com Online ID/área cliente) e proprietário. **Não aprovar** (wizard com motivo): → `not_approved` + email com motivo.
3. **Contraproposta** (wizard): alterna pending_proprietor ⇄ pending_buyer, regista movimento e mensagem.
4. **Cancelar** (por proprietário ou comprador): estados próprios; imóvel externo → estado cancel.
5. **Aceitar** (`btn_proposal_accepted`): exige valor ≠ 0 e comissão aprovada; **cancela automaticamente todas as outras propostas pendentes do mesmo imóvel** (com mensagem); comprador → estágio "Concluded"; emails aos coordenadores das duas equipas.
6. **Reverter** (grupo próprio): para pending_proprietor/pending_buyer; **eliminar** (admin).
7. Respostas do cliente pela área online (`new_client_reply`: aceitou/contrapropôs/cancelou) → email aos agentes+coordenadores do lado pendente.
8. Cron diário `_cron_give_agent_points` — pontos por propostas.
9. PDF "Proposal Report".

### 9.2 Processo de Venda (`sale.process`)

Aberto a partir de proposta aceite. **Estados:** `pending_employees` → `pending_manager` → `pending_approval` → `accepted` | `cancelled` — com needaction por perfil (agente vê pendentes dele, gestor os pending_manager, sale_process os pending_approval).

**Campos:** proposta (+ related: imóvel, tipo negócio, comprador, valor final, comissão do imóvel), agentes dos dois lados, **participantes** em duas formas: `participant_partner_ids` (M2M res.partner — default: proprietário do imóvel + associados + comprador + associados; validações "processo interno precisa de ≥2 participantes") e `participants_ids` (**`sale.process.participant`** — ficha KYC completa por participante: comprador/vendedor/procurador/fiador/empresa, NIPC/PCC/NIF/IBAN, documento de identificação completo, estado civil + cônjuge, profissão, naturalidade, nacionalidade, residência, "procurador de"); referências (vendedor/comprador referenciado → % + agente referenciador); condições: valor de reserva, dias para escritura + deadline, tipo de habitação (própria/secundária/outra), `cpcv_request` (ou direto a escritura), sinal, data CPCV, reforço (sim/não + data), percentagens CPCV/reforço/escritura (related do imóvel), arrendamento (data contrato, propósito habitação/comercial, valor renda, duração, início/fim, caução, rendas a pagar, total), observações, ficheiros do processo, `external_sale` (venda partilhada), `rs_from_company`, flags no_buyers/no_sellers, `commissions_ids`, `acts_ids`.

**Botões:** submeter ao gestor / submeter a revisão / rollback (a gestor ou a agentes, com notificações) / cancelar / **aceitar** (gera atos e comissões) / reabrir / imprimir. Emails de atualização a líderes e agentes.

---

## 10. Financeiro

### 10.1 Comissões (`sale.commissions`) — "Ficha de Comissão"

**Identificação:** `file_id` gerado (VEN001 / ARR001 / OUT001 sequencial por tipo de negócio), `file_date`, `name` = "ID - Agente (REF)". **Estados:** `waiting` (em revisão) → `done` (revista) → `paid` | `cancel`. Needaction: fichas waiting cuja data do ato já passou.

**Campos de negócio:** `relative_to` (**a pagar no CPCV / Escritura / Reforço / Renda**), `relative_percentage` (% desta parcela), `partner_id` + designação (comprador/inquilino/proprietário/senhorio), processo (domain accepted), imóvel, `employee_id` (agente), datas (proposta, reserva, CPCV, escritura, renda, reforço; `org_date` computada+cron diário para agrupar por mês), `amount_transacted`, `commission_type` (percentagem/fixo) + `commission_value`/`commission_amount`, venda partilhada (agência/agente externos), banco, referências (`is_reference`, `reference_from_agent`, `have_reference`, `refereed_employee_id`, `percentage_reference`/`reference_amount` com retro-compatibilidade para valores manuais antigos).

**Cálculo financeiro (recalculado a cada onchange e em qualquer write relevante — `_recalculate_financial_fields`):**
```
commission_amount   = amount_transacted × commission_value%   (ou valor fixo)
commission_to_company = relative_percentage% × (commission_amount × parcel_to_receive%)
commission_to_agent = agent_commission% × commission_to_company
owner_agent_part    = commission_to_agent / 2
buyer_agent_part    = commission_to_agent − owner_agent_part
owner_penalty       = owner_agent_part × owner_penalty%      ← automático: comissão <4% → 15%; <5% → 10%; senão 0 (editável manualmente)
agent_base          = (owner_part − penalty) + buyer_part
reference_amount    = percentage_reference% × agent_base     (referências)
receive_base        = agent_base − reference_amount
retention_to_receive= receive_base × percentage_retention%
iva_to_receive      = receive_base × percentage_iva%
total_receive       = receive_base − retenção + IVA
company_amount      = commission_to_company − commission_to_agent + penalty + retenção
```
(Fichas de **referência** têm ramo próprio: total = reference_amount − retenção + IVA; company = 0.) Todos os campos com limites ≤100%. As fichas finalizadas (done/paid/cancel) **congelam** os valores para auditoria (métodos `*_for_report` para o PDF bater certo com a ficha).

**Fluxo:** primeira abertura em waiting mostra **wizard de configuração** (tipo, valores, referências). **Rever** (`btn_commission_reviewed`): gera `private_code` (8 chars) e envia ao agente a **Folha de Comissão em PDF** por email + notificação ("para confirmar use o código X"); o agente confirma introduzindo `agent_code` (`code_confirmed`); atribui **pontos** (categoria 14 por equipa; multiplicador 0 para referências/valor 0; **×2 se CPCV com 100%**; data = data do ato); estado done. **Pagar** / **Cancelar** / reverter para done/waiting (limpa códigos). Alterar data de um ato atualiza as datas das comissões ligadas + notificação a todo o grupo sale_process.

### 10.2 Atos (`sale.acts`)

Eventos formais do processo: **Adenda / Renda / CPCV / Escritura / Reforço**. Estados `waiting` → `concluded` | `cancel`. Campos: gestor financeiro (default employee 249 — Anabela), imóvel + agente vendedor, cliente + agente comprador, datas início/fim (mín. 1h), processo, minutas, flag documentos anexos. Needaction: atos waiting já vencidos.

**Concluir ato:**
- Escritura/Renda → imóvel **Vendido**/**Arrendado** (mensagem com data), externos → transacted, **pontos CO** (angariação vendida + 3 angariações vendidas/mês ao criador da visita ou referred assistant).
- CPCV → imóvel **Reservado**.
- Notificações: employee 60 (mudança de estado), todo o grupo sale_process (ato concluído; exclusões configuradas).
- Escritura → **SMS automáticos ao proprietário e ao comprador** (templates `automatic_sms_act_done_owner/buyer`).
- Reverter ato: repõe estágio anterior do imóvel (via stage_changes).

Os atos (CPCV/Escritura) aparecem na agenda unificada dos três envolvidos.

### 10.3 Minutas e cláusulas

`sale.minute` (por ato, com M2M de `minute.clause` — cláusulas reutilizáveis ordenáveis com corpo de texto) + impressão da minuta (PDF "sheet_process_report").

### 10.4 Movimentos de dinheiro (`money.handling`)

Lançamentos financeiros simples: nome, data, valor, tipo (`money.handling.type` interno-agentes / externo-parceiros), denominação, agente, agência, `is_prevision` (previsão) com período início/fim. Usado para mapa financeiro por agência.

### 10.5 Registry Book

`registry.book` — livro de registo de contratos com **numeração automática semanal por cron** (números de contrato para angariações online).

### 10.6 Contabilidade PT

`rdx_l10n_pt`: plano de contas português, impostos, posições fiscais (base Odoo accounting). Facturação Odoo standard disponível (invoices ligadas a equipas pelo `rdx_sales_team_extended`).

---

## 11. RH

### 11.1 Colaborador (`hr.employee` estendido)

Campos: flags de notificação (recebe email de criação de visitas de angariação, recebe relatórios semanais), **zonas de ação** (`parish_ids`, `county_ids` — usadas na disponibilidade/sugestões), `no_points_reports`, `no_weekly_report`, `invest_mail`, **`agent_tph`** (nº TPH único, exceto agências virtuais fixas 196/197/198/332 = '001'), `vat`, `signature` (imagem), `is_virtual` (entidade virtual "agência"), `agent_job` (related do cargo), **`commission_step`** (escalão 1=45% / 2=50% / 3=55%, computado do volume de comissões pagas à empresa no ano anterior: ≥50k → 3; ≥30k → 2; virtuais fixos = 2) + `sales_volume`, `qr_code` vCard (link página da equipa no site), categorias de imóveis que trabalha (6 flags), conhecimentos (estudo rentabilidade, fiscalidade, mais-valias, jurídico), idiomas falados, `is_invest`.

Cargos (`hr.job` + `agent_job`): **1=Agente/Avaliador, 3=Gestora CO, 4=Operador, 6=Coordenadora** (IDs usados na lógica), com departamentos Comercial/CO.

`get_coordinator()` — resolve o coordenador do agente (usado em dezenas de notificações).

**Relatórios/cron do employee:** indicadores semanais por agência (tabela Lisboa/Leiria/Algarve com 18 indicadores + legenda, enviada ao grupo `indicators_report_group`), relatório mensal de horas de sessão dos operadores, relatório semanal de pendências ao coordenador, lembrete diário de visitas.

### 11.2 Pontos (gamificação)

- **Categorias** (`hr.employee.points.category`): nome, pontos globais ou **por equipa** (`hr.employee.points.category.team`, único por equipa+categoria), ativo, `dev_code` (código técnico único para lookup, ex. 'zmg'), sequência.
- **Pontos** (`hr.employee.points`): colaborador, equipa, data, categoria, pontos, **origem** (reference genérico: visita, imóvel, comissão, login…), referência do imóvel auto-preenchida.
- **Atribuições automáticas:** visita angariação marcada (operador), visita angariada, visita→vendido, redução de preço, comissão revista (×2 CPCV 100%), semanal: operadores com ≥10 visitas realizadas (categoria própria) e equipas de marketing sem visitas angariadas (pontos negativos), mensal: operador com ≥10 angariações, agente **sem angariações no mês** (categoria 'zmg'), faltas de login (cron `cron_fechardias`: sem login até 18:15 em dia útil → pontos categoria 33/32), escalas abertas (cron 22h).
- Dashboard mostra pontos e **ranking** do agente no período.
- Relatórios: pontos por colaborador, todos os pontos, pontos operacionais (PDFs), grupo `points_edit_group` para editar, `give_points_employees_group` para atribuir manualmente.

### 11.3 Escalas (`hr.employee.scale`)

Turnos de atendimento na agência: agente, empresa, início/fim (edição de horas bloqueada exceto gestores), `open_scale`/`close_scale` (registo de abertura/fecho **com IPs autorizados** — `scale_allowed_ips` data file), painel da escala com tudo o que aconteceu no turno (alterações de preço, imóveis, contactos, respostas, prospectos, conversas — contadores M2M computados), observações, relatório anexo, **checklist** (`hr.employee.scale.checklist` + vista própria), lembrete diário por email (cron 09:00), pontos por escalas abertas (cron 22:00), email quando muda o agente da escala. Aparece na agenda unificada.

### 11.4 Tarefas (`hr.employee.tasks`)

Título, atribuído a, revisor, período, estado (**to_do / done / not_done** + motivo), observações, **recorrência** (diária/semanal/mensal/trimestral com data de fim; cron recria ocorrências) e cron de lembretes. Grupos Task Operator/Middle/Manager.

### 11.5 Disponibilidade (`hr.employee.availability`)

Indisponibilidades: agente (ou empresa inteira, para gestores/admins — `user_type` agent/manager/admin), empresa (criação em empresa-mãe replica para filhas), período, motivo, recorrência semanal (cron). Consumida nas grelhas de disponibilidade das visitas e nas sugestões de horários. O cron aproveita para **verificar/geolocalizar** tracking web (cliques, angariações online, pesquisas) e apagar acessos de IPs internos.

### 11.6 Reuniões (`hr.employee.meeting`)

Reuniões semanais de equipa: data, responsável, equipa, empresa, presenças (M2M employees).

### 11.7 Recrutamento (`hr.prospect`)

Pipeline de recrutamento de agentes (tipicamente angariadores externos): nome, estado (**não contactado → em seguimento → entrevista → formação → a trabalhar | não interessado | cancelado**), telefone/email, tem agência? + grupo de agências, para que agência (domínio exclui empresas 3/5), nº de imóveis, histórico de contactos (`hr.prospect.contacts`) e entrevistas (`hr.prospect.interview`), agendamento de reuniões (entrevista/sessão esclarecimento/formação teórica/prática) com data, envio de SMS/email com templates default ("Nova perspetiva de angariar"), datas de seguimento com needaction diário, resposta (método/estado). Grupo `hr_recruiter_group`.

### 11.8 Relatório Semanal de Atividade (`employe.weekly.activity.report`)

Documento semanal por agente, **criado automaticamente por cron** (01:00) para agentes ativos (exceto `no_weekly_report`):
- Linhas automáticas (não elimináveis): visitas de angariação da semana (`ewar.gathering.visit`: contacto, data, estado agendado/realizada/não realizada/reagendada/**angariado**, valor sugerido, preço estimado, obs → obs escreve na visita), visitas comerciais (`ewar.commercial.visit`: comprador, data, estado, obs, **fez proposta?**, **compraria?** → escrevem no relatório da visita), visitas de avaliação (`ewar.evaluation.visit`), contactos de compradores (`ewar.buyer.contacts`), mensagens de prospecção (`ewar.prospect.message`).
- Semana corrente definida por cron (05:00, 7 em 7 dias); lembrete por email (09:00); **fecho e envio automático** (16:00) do PDF aos gestores com `receive_weekly_activity_reports_email`; botão de recriação (config parameter `recreate_weekly_report`).
- Criar visitas/contactos a partir do relatório valida que a data pertence à semana do relatório.

---

## 12. Dashboard e notificações

### 12.1 Dashboard (`triomphe.dashboard`)

Página inicial por perfil (`view_feedback` admin/agency/agent — escolhível por utilizador com `see_all_dashboard_group`):
- Cartão do utilizador: foto, nome, **escalão de comissão**, **pontos do período**, **ranking entre agentes**.
- Seletor de período: dia/semana/mês/trimestre/ano (períodos vindos de `base.period*` do `rdx_base_ext`).
- **Notícias** (`dashboard.news` — mensagens até 90 chars, 4 mais recentes).
- Gráficos (queries por perfil: admin = por agência; gestor = por membro da equipa; agente = por imóvel/estado):
  1. Chamadas de prospecção (enviadas/recebidas)
  2. Leads/contactos de compradores
  3. Imóveis angariados em stock (aberto/exclusivo)
  4. Propostas
  5. Cliques web por imóvel (product.tracking)
  6. Visitas (angariação por visit_state; comerciais por estado) — barras temporais
- Séries temporais com granularidade ligada ao seletor (7 dias/6 semanas/6 meses/6 trimestres/3 anos).

### 12.2 Notificações internas

- `user.notification`: texto (HTML com links), tipo (`notification.type`: **warning / contacts / real_estates / visits / sales**, com ícone), destinatário (employee) ou empresa, `already_read`.
- `system.notifications`: centro de notificações (HTML) com agrupamento Novas/Antigas, datas por extenso, tempos relativos (5m/3h/2d), badge de não-lidas no topbar, mute por utilizador, marcar como lida ao clicar, dropdown das últimas 10.
- Emissores (exemplos já descritos): lead duplicada→gestoras CO; redução de preço→agentes; imóvel validado→angariador+coordenador+gestoras; visita aguarda aprovação→chefe; proposta nova/atualizada→agentes+coordenadores+21/22/60; comissão enviada→agente; ato concluído→sale_process; contrato online→coordenador; marca d'água→coordenador; comprador sem contacto 5 dias→agente; pedido de pesquisa sem resultados→22/60.

### 12.3 Registo de mensagens enviadas

- `triomphe.sms.message` e `triomphe.email.message`: log central de todos os envios (destinatário, mensagem, estado sent/error/not_activated + detalhe, template, utilizador). Menus "Messages", "Communication Report", "Medidor Estatístico de Comunicação" e relatórios combinados email+SMS.

---

## 13. Segurança de documentos e login

### 13.1 Documentos protegidos por PIN (`document_security.py`)

- Aplica-se a: documentos de **imóveis** (tab Documentos / `files_test`) e documentos de **proprietários** (attachments do partner com `proprietary=True`).
- Acesso requer **desbloqueio por PIN** (wizard `documents.pin.wizard`); cria janela de acesso temporária (`document.security.access.has_valid_access`) com duração normal ou **prolongada** para o grupo `group_documents_unlock_extended` (durações em parâmetros de sistema).
- Intercepta leitura/download de `ir.attachment` (não bloqueia caminhos técnicos/superuser).
- **Auditoria** (`document.security.log`): evento (abertura de área, download…), utilizador, alvo (partner/template/product); visível só ao grupo de auditoria; menus "Documentos"/"Relatório de Documentos". Grupo `delete_documents_rs_group` para apagar.

### 13.2 Segurança de login (`login.registry` + `res.users.authenticate`)

- Regista **todos os logins** (IP, hora, user).
- Parâmetros: `allowed_ips` (IPs sempre permitidos — ex. agências), `blocked_user_ids_permanent` (bloqueio total), `blocked_user_id_external_login` (users que só entram de IPs autorizados).
- Regras: **operadores (cargo 4) só entram de IPs autorizados**; ≥5 tentativas/minuto → **ban do IP** + SMS de alerta ("aviso_atividade_suspeita") para o número de segurança (932384448) + SMS ao próprio ("aviso_multiplos_logins"); primeiro login de IP novo externo → SMS "aviso_novo_login" ao colaborador; IP banido → recusa + alertas periódicos.
- Cron `cron_fechardias`: penaliza com pontos quem não fez login até 18:15 (grupo "Faltas de atraso").
- Sessões web geridas pelo módulo `tko_web_sessions_management` (timeout/sessões múltiplas).

---

## 14. Fluxos de negócio

Sequências completas ponta-a-ponta (síntese executável do que está nos §5–§11 — usar como user-stories no novo CRM):

### F1 — Captação de prospecto
Entradas: scraper OLX/Imovirtual (cron), criação manual (CO com `create_rs_prospects_group`, agentes FSBO), website, WhatsApp (Triomphe Chat cria lead/conversa), importação de ficheiro. Sempre: cria imóvel-sombra, deteção de duplicados, SMS automático de inserção (template `automatic_sms`, exceto se criado por avaliador job 1; respeita "pisco" ativado do template e regista no chatter), primeira chamada pendente imediata (ou +5 dias se já houver histórico).

### F2 — Teleprospecção (CO)
Operador só vê as leads distribuídas para hoje. Distribuição manual/automática por wizard (config por distrito/tipo). Trabalha a lead com os botões (Atendida/Não Atendida/WhatsApp/SMS/email). Não atendidas: SMS imediato + reagendamento automático noturno (+5 dias 09:00). Emails automáticos em cadeia: `documentation_email_co` (3 dias após "Email Automatico padrao"), `email_prospectos_objecao` (5 dias depois, recorrente a cada 5 dias), com reagendamento da chamada +1 dia útil. Leads com ≥3 emails sem resposta 7 dias após o último → **descartadas automaticamente** pelo cron. Estatísticas do CO (relatório de chamadas por operador com períodos manhã/tarde).

### F3 — Visita de angariação
Marcada pelo CO (a partir da chamada atendida) ou pelo agente. Cascata: criar → equipa → agente → pronto; notificações/SMS/emails em cada passo (proprietário, agente com ficha PDF, líder, gestoras, coordenadora); confirmação da visita com SMS; véspera com SMS (opcional); grelha de disponibilidade; conclusão da prospecção + congelamento da lead.

### F4 — Resultado da visita → angariação
Wizards: Realizada e Angariada (imóvel → estágio Angariado, pontos), Realizada não angariada, Não realizada (falta agente/proprietário), Reagendar (1× para agentes), Cancelar (log + libertação da lead). QC preenchido pela equipa de qualidade. Relatório semanal reflete tudo. Depois: validação de documentos (SMS/notificações), fotos + marca d'água, publicação no site/portais/Habitar, relatórios mensais de atividade ao proprietário.

### F5 — Angariação online
Website `/gathering` em fases (dados pessoais → dados do imóvel → fotos upload → contrato): cria proprietário + imóvel em estágio "Angariação Online", gera **contrato eletrónico PDF** (relatório `online_gathering_contract_report` com nº de contrato do Registry Book), anexa ao imóvel, notifica coordenador. Tracking das fases por cliente (`online.gathering.clients` com IP/geo/fase atingida). Comunicações externas específicas para "vai angariar online" (SMS lembrete 20h).

### F6 — Comprador
Entrada: website (pedido de contacto/pesquisa guardada/favoritos), chamada, agência, prospecção. Regista-se contacto (mensagem) + respostas; matching automático com stock (imediato + notificações "novo imóvel corresponde à sua pesquisa"); follow-ups automáticos (7 dias), lembrete ao agente (5 dias), arquivo automático (180 dias), emails segmentados (D/DA/E). Estágios atualizados por crons.

### F7 — Visita comercial
Exige contacto registado; QC; relatório de visita (fez proposta? compraria?); emails a comprador/agente/agente do imóvel; conflitos bloqueados.

### F8 — Proposta
Online (validação por código email) ou manual. Aprovação → ping-pong de contrapropostas com histórico → aceite (mata concorrentes) → processo. Anexo obrigatório para transitar. Comissões divergentes precisam de aprovação da administração.

### F9 — Processo → Atos → Comissões
Processo com KYC completo dos participantes e condições financeiras → aprovação em 3 níveis → atos agendados (agenda unificada) → conclusão de atos muda estado do imóvel + SMS às partes → fichas de comissão com cálculo automático, revisão, confirmação por código pelo agente, pagamento; pontos.

### F10 — Invest
Contactos e imóveis `is_invest` isolados por grupo; acesso do cliente por Online ID + código; conteúdo multilíngue; gestor invest dedicado; cartas registadas (letter_sent).

---

## 15. Website

Módulo `triomphe_web` (QWeb + controllers) — a lógica de negócio destas páginas tem de ser reimplementada no novo frontend público:

### Montra de imóveis (`/shop`)
Listagem/pesquisa de imóveis publicados, página de detalhe, favoritos (`/shop/add_fav`), **guardar pesquisa com notificação** (`/shop/save_search_notify` → cria comprador+pesquisa; `online.searches` com tracking), pedido de contacto (`/shop/request_contact` → cria contacto no CRM), informação do imóvel via AJAX, imagens panorâmicas 360º (`/shop/get_pano_images`), **comparador de imóveis** (`/shop/compare`), localizações (`/shop/get_all_locations`), página da equipa com ficha de cada agente (`/shop/page/our_team/<id>` + QR vCard).

### Área de Cliente (`/client`)
Login por **Online ID + código** enviado por email/SMS: dashboard, informação pessoal (edição), **os meus imóveis**, **as minhas propostas** com resposta online (aceitar/contrapropor/cancelar — `/client/proposal_reply`), eventos/visitas, suporte (mensagens ↔ conversas do CRM com `client_read`), recuperação de ID (`/client/send_recover`, `/client/recover`), processo (`/client/process`), logout.

### Angariação online (`/gathering`, `/g`)
Fluxo multi-fase com upload de imagens (`/gathering_save_image`), obrigado (`/gathering_thanks`).

### Propostas online (`/proposal`)
Formulário + validação por código (`/proposal/validate_code/<id>`, `submit`), obrigado.

### Invest (`/access`, `/redirect_invest`)
Pedido de acesso à área restrita de investimento.

### Outros
`/privacy`, bandeiras de idioma (PT/EN/FR em todo o site), logo, cookies (`website_cookie_notice`), tracking de cliques por imóvel com GeoIP e anti-bot.

---

## 16. Integrações

| Integração | Detalhe |
|---|---|
| **Usendit (SMS)** | SOAP `api.usendit.pt/v2` SendMessage. Sender configurável por template (default "TRIOMPHE"; operadores enviam com o próprio nº de trabalho). Validação de nº PT (9 dígitos começado em 9, ou 351+), agendamento (`scheduleDatetime`), mapeamento completo de códigos de erro, CA bundle próprio, log obrigatório em `triomphe.sms.message`. **Credenciais hard-coded no código atual — migrar para configuração segura** |
| **Email (SMTP)** | Múltiplos `ir.mail_server` (ids 6, 13, 14 referenciados por fluxo: comercial, angariação, qualidade). Remetentes: no-reply@, atcomercial@, comercial@imo-triomphe.pt, qualidade@, proposta-online@. Log em `triomphe.email.message` |
| **Habitar Portugal** | `partnersapi.habitarportugal.pt/api/v1` — publicação de imóveis: payload construído de todo o imóvel (validação + hint de coordenadas), estados por imóvel (pending/sent/error/disabled), tentativas, última resposta, advert_id remoto, botões enviar/reenviar/remover/testar, log próprio (`habitar.log`); parâmetros: partner_id, secret, import_key, agency_id; contexto `skip_habitar` para evitar loops |
| **Idealista** | API partners (OAuth client_credentials + feedKey) — criação/consulta de imóveis e contactos (sandbox no código) |
| **Export XML portais** | Wizard admin exporta imóveis em XML (formato CasaSapo-like: advert + atributos + imagens por URL pública), individual/ZIP/lista |
| **OLX/Imovirtual crawler** | Scrapy + ficheiro de resultados consumido pelo cron de leads |
| **Triomphe Chat (WhatsApp)** | Webapp externa (o projeto `triomphe-whatsapp`, Django+React): SSO por **JWT assinado** (secret partilhado `whatsapp.jwt_secret`, URL `whatsapp.chat_url`); Odoo expõe APIs XML-RPC para o bot (snapshots de leads/partners, procura por telemóvel, criação de phonecalls/conversas/anexos); opt-out respeitado; grupo de acesso próprio |
| **GeoIP** | ipgeolocation.io para geolocalizar cliques/acessos, com blacklist de datacenters/bots |
| **Google Maps** | Link/widget/lat/lng por imóvel |
| **QR codes** | vCard dos agentes; QR do imóvel |

---

## 17. Crons

| Cron | Frequência | Função |
|---|---|---|
| Employee Contact Buyer Email Reminder | diário 09:00 | Notifica agentes de compradores sem contacto há 5 dias |
| Weekly Activity Set Current Week | 7 dias 05:00 | Define semana corrente dos relatórios |
| Weekly Activity Reports Create | diário 01:00 | Cria relatórios semanais |
| Weekly Activity Email Reminder | diário 09:00 | Lembrete de preenchimento |
| Weekly Activity Close & Send | diário 16:00 | Fecha e envia PDFs aos gestores |
| Update Proprietaries Linked / Update Proprietary Stage | diários | Estágios de proprietário conforme imóveis |
| Registry Book Contract Number | 7 dias 05:00 | Numeração de contratos |
| Reset Leads Operator | diário 00:00 | Limpa `operator_working_id` (exceto 265) |
| Send Agents Real Estates Report | 7 dias | Stock list aos agentes |
| Paulo Real Estates Report | mensal | Stock list à administração |
| Remind Gathering End | diário | Avisos de fim de contrato de angariação |
| If RS From Company | diário | Flag RS da empresa |
| Remind Visits Notification | diário | Lembretes de visitas |
| Recreate recurrent tasks / Reminder Tasks | diários | Tarefas recorrentes + lembretes |
| Recreate recurrent availability | diário | Indisponibilidades recorrentes + verificação/geo de tracking web |
| Agent Points For 0 Month Gatherings | mensal 01:00 | Pontos negativos por mês sem angariações |
| Scheduled Points | diário 02:00 | Pontos semanais (segunda) e mensais (dia 1) de operadores |
| Dispose Lead After Prospection Email | diário 07:00 | Descarta leads com 3 emails sem resposta há 7 dias |
| Scale Email Reminder | diário 09:00 | Lembrete de escala |
| Points Open Scales | diário 22:00 | Pontos por escalas abertas |
| Indicadores Semanais | 7 dias 08:00 | Email de indicadores por agência |
| Proposals Points To Agents | diário 23:59 | Pontos por propostas |
| Create Monthly Clicks | diário 06:25 | Cliques → relatórios de atividade |
| Reagendar chamadas após 22h | diário 22:10 | Não atendidas → +5 dias 09:00 |
| Follow-up compradores (`_cron_send_automatic_buyer_followup`) | recorrente | Follow-up 7 dias (máx 20/dia, 10h–19h) |
| Follow Up Leads (avaliadores + coordenadora) | semanal | Relatórios por email |
| Sale Commissions org_date | diário | Datas de organização |
| Operator Session Hours Report | mensal dia 1 08:00 | Horas de sessão dos operadores |
| Coordinator Weekly Pending Report | diário 08:00 | Pendências ao coordenador |
| Login faults (`cron_fechardias`) | diário | Pontos por falta de login |
| Comunicação Exterior (vários) | diários | Ver §20 |
| *(desativados mas existentes)* | — | Birthday SMS, External Communication SMS/email, Update Financing, Update Buyers, Send Email E, Scraper, Disable Gathering Visits, comunicações pós-visita, SMS véspera, Reset comunicação pós-visita, Match Partners |

---

## 18. Templates

### Modelo de template
- **Email** (`triomphe.email.template`): nome técnico único (lookup por código), descrição, assunto, corpo **PT + EN + FR**, preview, anexos, from/reply-to, **flag `activated`** (pisco liga/desliga o envio — quando desligado o sistema regista "não enviado" no chatter), categoria (informação/argumentação/objeções). Placeholders por `format()` e `${var}`.
- **SMS** (`triomphe.sms.template`): idem + contadores de caracteres por língua + **sender** configurável.

### Templates referenciados no código (mínimo a migrar)
**SMS:** `automatic_sms` (inserção CRM), `nao_atendido_sms`, `lembrete` (20h angariação online), `prospecto_ang_online`, `birthday_users`, `mensagem_vazia` (genérico), `recuperar_id_cliente`, `confirmacao_visita_angariacao`, `confirmacao_visita_comercial`, `cancelamento_visita_comercial`, `imovel_validado_s_id`, `imovel_validado_c_id`, `automatic_sms_act_done_owner`, `automatic_sms_act_done_buyer`, `aviso_atividade_suspeita`, `aviso_multiplos_logins`, `aviso_novo_login`.

**Email:** `email_prospectos`, `documentation_email_co`, `email_prospectos_objecao`, `email_parish_price`, `lead_email_e`, `lead_email_d`, `lead_email_da`, `automatic_email_archived_leads`, `followup_buyer`, `reducao_valor_comprador`, `scheduled_visit_email`, `nova_proposta_recebida`, `partner_email_code`, `new_lead`, + emails HTML institucionais hard-coded (código de validação de proposta, aprovação/rejeição de proposta, ID de cliente, recuperação de ID, código de acesso — layout com logo, botão laranja "Visite Triomphe.pt", telefone, AMI 13164) + `email.template` Odoo: Visit Sheet to Proprietors, Activity Report.

---

## 19. Relatórios e wizards

### Relatórios PDF (QWeb)
Avaliação do prospecto, Ficha de Visita (sheet_visit), Escala RH, Declarações, Ficha de Processo/Minuta, Proposta, **Contrato de Angariação Online**, Apresentação do Imóvel, Relatório de Atividade do Imóvel, Folha de Comissão, Pontos (agente/todos/ops), Sugestão de Preço, Estatística de chamadas CO, Estatística preço por zona, Estatística visitas por agente, Visitas Canceladas, Relatório Semanal de Atividade, Cartão de agradecimento, **Estudo de Mercado**.

### Wizards de relatório/exportação (Excel/CSV/email)
Stock List (+ cron), lista de visitas, visitas canceladas, chamadas CO (períodos manhã/tarde), chamadas de compradores, preço m²/por zona (+ email de preço de freguesia), estatística de publicações, cliques web Excel, comunicações ST (email/SMS/combinado), exportar proprietários CSV, angariações realizadas, último contacto proprietário, relatório em massa RS (send_mass_rs_report), imóveis semelhantes (+ janela + envio ao comprador `send_similar_rs_wizard`), notificar gestor de RS, sessões de operadores.

### Wizards operacionais
Todos os de resultado de visita (3 tipos × confirmada/não realizada/cancelada), reagendamento, contraproposta, proposta não aprovada, angariado/cancelado/rollback de imóvel, prospecto concluído/cancelado, atribuição/distribuição/transferência de leads, dispose em massa, SMS/email/WhatsApp de prospecto, mass SMS, mover imóveis em massa, marca d'água, PIN de documentos, primeira abertura de comissão, exportar XML, conversa de proprietário, phonecall→visita (call e WhatsApp), agendar chamadas em massa, importar leads, remover localidades duplicadas, definir preço de freguesia.

---

## 20. Comunicação Exterior

Motor de campanhas automáticas sobre prospectos (menu próprio, grupo `group_external_communication_user`), organizado por segmento:

- **Em prospecção — existentes sem contacto**: wizard + settings + cron de envio.
- **Em prospecção — existentes com contacto**: idem.
- **Em prospecção — inserção CRM**: SMS automático na criação da lead (síncrono).
- **Qualificados — existentes**: wizard de envio em massa (`wizard.envio.comunicacao.qualificados` com template de email selecionável) + settings + cron.
- **Qualificados — marcação de angariação**: cron sobre `calendar.event`.
- **Qualificados — visita realizada não angariada** e **visita não realizada**: automações pós-visita.
- Infraestrutura: utilitários de email/SMS próprios, **log de execução de crons** (`external_communication_cron_log` + linhas), flag `is_external_automatic` propagada a phonecalls/conversas/partners para excluir envios automáticos das métricas de contacto humano.
- Configurações por segmento em `res.config.settings` dedicados.

---

## 21. Validações

Resumo das regras de integridade a preservar:

1. `advertID` + origem únicos por lead; telefone/telemóvel deduplicados contra leads e partners; aviso "é agente".
2. VAT único para agências externas/partners com VAT.
3. TPH de agente único (exceto virtuais).
4. Referência de imóvel não pode começar por "TPH" (reservado a processos automáticos).
5. Um negócio por tipo por imóvel; relatório de atividade único por período+imóvel; pontos de equipa únicos por equipa+categoria; grupo de agências com nome único.
6. Datas: visitas não podem nascer no passado (exceto gestores); atos ≥1h; validações cruzadas de datas de proposta (sinal ≤ reforços ≤ escritura); visitas de relatório semanal dentro da semana.
7. Observações de proposta ≤600 chars; mensagens de dashboard ≤90 chars.
8. Anexo obrigatório para transitar propostas; contacto registado obrigatório para visita comercial; ≥2 participantes em processos internos.
9. Reagendamento de visita limitado a 1× para agentes; edição de datas de visita apenas por wizard.
10. Agente não altera contactos de cliente com Online ID; operador não edita descrição pública após criação.
11. Envios de SMS apenas para números móveis PT válidos; templates desativados nunca enviam mas registam.
12. Estados readonly em cascata (chamadas done, propostas, comissões concluídas congeladas para auditoria).
13. Congelamento (freeze) da lead na conclusão — snapshot imutável de imóvel+proprietário.

---

## 22. Recomendações

Para o novo projeto Django + React (alinhado com `triomphe-whatsapp`):

1. **Modelo de domínio:** manter as entidades e estados deste documento 1:1 numa primeira fase (migração de dados direta), limpando apenas nomenclatura (ex.: `product.product` → `Property`, `crm.lead` → `Prospect`, `res.partner` → `Contact` com papéis).
2. **Eliminar IDs hard-coded** (employees 21/22/60/249/265, users 7/8, jobs 1/3/4/6, stages por índice, empresas por nome, distritos por ID): tudo isto deve virar **configuração** (roles funcionais: "gestora CO", "responsável financeiro", "nº de segurança SMS", "distritos por agência", mapeamentos de estágio por operação).
3. **Motor de automações declarativo:** os ~40 crons e cadeias de emails/SMS devem ser regras configuráveis (gatilho + condição + ação + template + calendário), com o log de comunicações no histórico do registo como já existe hoje.
4. **Permissões:** reproduzir o modelo `responsible_users` (coordenador/líder → membros) como serviço central de autorização; manter os grupos finos como permissions Django.
5. **Credenciais** (Usendit, Habitar, GeoIP, JWT) para variáveis de ambiente/secret manager — hoje há credenciais no código.
6. **Área de cliente e site**: substituir Online ID+código por autenticação moderna (magic link/OTP), preservando as funcionalidades (propostas online com resposta, angariação online multi-fase com contrato PDF, pesquisas guardadas com notificação de match).
7. **Auditoria**: chatter/track_visibility do Odoo é usado extensivamente — prever um sistema de activity log por registo com posts automáticos (o negócio depende disto para controlo de qualidade).
8. **Congelamentos**: implementar snapshots (lead concluída, comissão fechada) como registos imutáveis versionados.
9. **Integração WhatsApp**: o Triomphe Chat já é o padrão — o novo CRM deve absorver as APIs que hoje o Odoo expõe por XML-RPC (snapshot, find-by-mobile, criar mensagem/anexo).
10. **Migração faseada sugerida:** (1º) núcleo prospecção+imóveis+visitas+pessoas com regras de acesso; (2º) propostas/processos/financeiro; (3º) gamificação/escalas/RH; (4º) site público e área de cliente; (5º) automações de comunicação e relatórios.

---

*Fim do documento. Qualquer funcionalidade encontrada no código que não esteja aqui descrita deve ser adicionada antes de ser considerada fora de âmbito.*
