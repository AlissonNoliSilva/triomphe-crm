# Backlog — Novo CRM Triomphe

**Épicos e user stories por fase de migração** (fonte: `PRD_Novo_CRM_Triomphe.md` — as referências `B§x` e `Cx` apontam para as secções do PRD).

Convenções:
- **ID:** `F<fase>.E<épico>.S<story>`
- **Prioridade:** M = Must · S = Should · C = Could
- **Tamanho:** P (≤2 dias) · M (≤1 semana) · G (1–2 semanas) · XG (>2 semanas, partir em tarefas)
- Personas: **Operador** (CO), **Gestora CO**, **Agente** (avaliador), **Coordenadora**, **Gestor** (equipa), **Financeiro** (sale process), **Admin**, **Cliente** (comprador/proprietário), **Sistema** (automação).
- Critérios de aceitação (CA) em forma resumida; o detalhe normativo está sempre na secção do PRD referida.

---

## FASE 1 — FUNDAÇÃO

### Épico F1.E1 — Projeto base e infraestrutura
*Ref: PRD A11 · arquitetura triomphe-whatsapp*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F1.E1.S1 | Como equipa de dev, quero o monorepo Django (DRF) + React (Vite/TanStack) com apps de domínio vazias (`accounts, contacts, properties, prospects, visits, proposals, finance, hr, communications, recommendations, automations, odoo_sync`), para ter a estrutura alvo desde o dia 1. | M | M |
| F1.E1.S2 | Como equipa de dev, quero CI/CD com testes, lint e deploy automático para dev/staging/prod, para garantir qualidade contínua. | M | M |
| F1.E1.S3 | Como equipa de dev, quero segredos em variáveis de ambiente/cofre (Usendit, GeoIP, Idealista, JWT, SMTP), para eliminar credenciais no código. **CA:** nenhum segredo em repositório; rotação documentada. | M | P |
| F1.E1.S4 | Como equipa de dev, quero logging estruturado, métricas e alertas de erro (observabilidade base), para diagnosticar produção. | M | M |
| F1.E1.S5 | Como equipa de dev, quero Celery + beat + fila com retry/backoff e DLQ, para suportar automações e integrações. | M | M |

### Épico F1.E2 — Organização: agências, equipas, colaboradores
*Ref: PRD B§2, B§11.1*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F1.E2.S1 | Como Admin, quero gerir agências (empresas) com distritos de atuação e exceções por concelho (ex.: Lourinhã/Torres Vedras/Cadaval → Leiria) **em backoffice**, para que a atribuição automática de empresa funcione sem código. **CA:** função "empresa por localização" testada com as exceções atuais. | M | M |
| F1.E2.S2 | Como Admin, quero gerir equipas com líder, coordenadora e membros, e flags de tipo (vendas/marketing-CO), para reproduzir a estrutura atual. | M | P |
| F1.E2.S3 | Como Admin, quero fichas de colaborador com: cargo (flag "agente"), zonas de ação (freguesias/concelhos), TPH único, escalão de comissão (1/2/3 calculado pelo volume anual — B§11.1), assinatura, virtual, idiomas, categorias de imóveis, flags de notificação, para paridade com hr.employee. | M | M |
| F1.E2.S4 | Como Sistema, quero resolver `responsible_users` de um utilizador (ele próprio + membros das equipas que coordena/lidera), para alimentar todas as regras de acesso. **CA:** serviço central com testes contra a matriz B§3. | M | M |
| F1.E2.S5 | Como Admin, quero papéis funcionais configuráveis (Gestora CO, responsável financeiro default, nº de emergência SMS, coordenadora por empresa), para eliminar IDs hard-coded. | M | P |
| F1.E2.S6 | Como Admin, quero que criar um utilizador agente/operador crie automaticamente o colaborador ligado com equipa obrigatória, para manter consistência (B§2.3). | M | P |

### Épico F1.E3 — RBAC e permissões finas
*Ref: PRD B§2.4, B§3, A10*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F1.E3.S1 | Como Admin, quero perfis base (Agente, Gestor, Operador CO, Operador Comercial, Gestora CO, Financeiro, Admin) com herança, para reproduzir a hierarquia atual. | M | M |
| F1.E3.S2 | Como Admin, quero as ~40 permissões finas (editar pontos, distribuir leads, ver documentos RS, reverter propostas, chat WhatsApp, auditoria de documentos, etc. — lista B§2.4) atribuíveis por utilizador, para paridade. | M | M |
| F1.E3.S3 | Como Sistema, quero aplicar as regras de escopo de dados por entidade (B§3: agente vê o seu, stock angariado é global, operador vê a fila do dia, comissões só paid/done…), para que nenhum utilizador veja mais do que hoje. **CA:** testes de autorização por perfil×entidade×operação (matriz completa). | M | XG |
| F1.E3.S4 | Como Admin, quero multi-company enforced em leads, visitas, equipas e colaboradores, para isolamento entre agências. | M | M |

### Épico F1.E4 — Localização geográfica PT
*Ref: PRD B§4*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F1.E4.S1 | Como equipa de dev, quero migrar a hierarquia Distrito→Concelho→Freguesia(zip)→Zona do legado, para base geográfica única. | M | M |
| F1.E4.S2 | Como utilizador, quero que preencher o código postal resolva a freguesia (e cascata concelho/distrito/país), e que alterar um nível limpe os inferiores, em qualquer formulário de morada, para consistência. **CA:** componente de morada reutilizável frontend+backend. | M | M |
| F1.E4.S3 | Como Gestora CO, quero gerir preço/m² por freguesia×categoria×condição com histórico trimestral, para alimentar valores estimados. **CA:** alteração notifica agentes com imóveis afetados (via motor de eventos, fase 2). | M | M |

### Épico F1.E5 — Contactos (compradores/proprietários)
*Ref: PRD B§7*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F1.E5.S1 | Como Agente, quero fichas de contacto com papéis acumuláveis (comprador, proprietário, investidor, agência externa) e pipelines de estágio separados por papel, para paridade com res.partner. | M | G |
| F1.E5.S2 | Como Financeiro, quero o bloco KYC completo (documento, estado civil, NIF/NIPC/IBAN, naturalidade…) com indicador "N/12 completo", para preparar processos. | M | M |
| F1.E5.S3 | Como Sistema, quero atualizar automaticamente o estágio do proprietário a partir dos estágios dos seus imóveis (regras B§7.1), para pipeline sempre correto. | M | P |
| F1.E5.S4 | Como Agente, quero contactos associados (cônjuge/procurador/fiador com tipo de ligação e "usar mesma morada"), para agregados familiares. | M | P |
| F1.E5.S5 | Como Gestora CO, quero deteção de duplicados por telefone/email na criação (com aviso "é agente" quando aplicável) e painel de merge assistido com histórico, para qualidade de dados. *(merge assistido = RF-02/C8)* | M | G |
| F1.E5.S6 | Como Admin, quero gerir agências externas (VAT único, grupos de agências) e angariadores externos, para vendas partilhadas. | M | P |
| F1.E5.S7 | Como Sistema, quero registar consentimento por canal (whatsapp_opt_out, email, SMS) com data/origem, para compliance desde o início. | M | P |

### Épico F1.E6 — Imóveis
*Ref: PRD B§6*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F1.E6.S1 | Como Agente, quero a ficha completa do imóvel (identificação, tipologia, características/28 comodidades, divisões com áreas, exposição solar, áreas, morada+geo, certificado energético), para paridade com product.template. | M | G |
| F1.E6.S2 | Como Sistema, quero estágios do imóvel com histórico de mudanças e operações especiais (default, angariado-por-visita, done-deal), para o workflow comercial. | M | M |
| F1.E6.S3 | Como Agente, quero negócios por imóvel (Venda/Arrendamento/Troca, único por tipo, valor+negociável) com **histórico de alterações de preço**, para rastreabilidade. **CA:** eventos de preço publicados no bus (consumidos na fase 2/4). | M | M |
| F1.E6.S4 | Como Agente, quero o bloco de angariação (datas início/fim, tipo, comissão %/fixa, % CPCV/reforço/escritura, chaves, placa, montra), para o contrato de mediação. | M | M |
| F1.E6.S5 | Como Backoffice, quero a checklist de documentos legais (CMI, caderneta, certidões, licença, CE, plantas, CC…) com estado de verificação e datas de validade, para controlo documental. | M | M |
| F1.E6.S6 | Como Agente, quero múltiplas imagens ordenáveis com marca d'água aplicável em lote (permissões: angariador/gestora/coordenadora/admin; notificação ao coordenador), para media do anúncio. | M | M |
| F1.E6.S7 | Como Admin, quero o cofre de documentos com desbloqueio por PIN, janelas de acesso (normal/prolongada) e log de auditoria de acessos/downloads, para paridade com B§13.1. | M | G |
| F1.E6.S8 | Como Gestor, quero imóveis externos (agência parceira, estados available/transacted/cancel) e flag Invest com campos multilíngue, para os fluxos especiais. | M | M |
| F1.E6.S9 | Como Sistema, quero valor estimado = área × preço/m² da freguesia/condição e score de qualidade de publicação com dicas, para apoiar pricing e anúncios. | M | P |
| F1.E6.S10 | Como utilizador, quero validação da referência (proibido prefixo "TPH" salvo admin/processo automático) e restantes unicidades (B§21), para integridade. | M | P |

### Épico F1.E7 — Prospectos e interações (núcleo)
*Ref: PRD B§5*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F1.E7.S1 | Como Operador, quero criar prospectos que geram automaticamente o imóvel-sombra ligado, com sincronização bidirecional dos campos do imóvel e da morada, para paridade com crm.lead. | M | G |
| F1.E7.S2 | Como Operador, quero estados de prospecção completos (draft/pending/no_answer/reschedule/sent/received/answer_accepted/disposed) com motivos de descarte/adiamento e contadores, para o funil atual. | M | M |
| F1.E7.S3 | Como Sistema, quero a mecânica de follow-up: **uma única interação pendente por prospecto**, fecho automático das anteriores com rótulo de origem, e sincronização do estado do prospecto com a última interação, para o motor da teleprospecção. | M | G |
| F1.E7.S4 | Como Agente, quero notas privadas por utilizador no prospecto, para apontamentos pessoais. | M | P |
| F1.E7.S5 | Como Sistema, quero o congelamento (snapshot imutável de imóvel+proprietário) na conclusão do prospecto, para histórico auditável. | M | M |
| F1.E7.S6 | Como utilizador, quero a timeline de interações unificada (chamada, SMS, email, WhatsApp, site, presencial; enviado/recebido; ligada a prospecto/contacto/imóvel), para substituir phonecalls+conversations. | M | G |
| F1.E7.S7 | Como Gestora CO, quero validações de duplicados na criação (telefone/advertID+origem) com alerta às gestoras quando repetido, para qualidade. | M | P |

### Épico F1.E8 — Intake API e importação
*Ref: PRD RF-01/02, B§5.3–5.4*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F1.E8.S1 | Como Sistema, quero uma API de intake com payload normalizado (source, channel, UTM, consent, locale, IP, UA) que devolve `lead_id`, com rate limit, captcha e idempotência, para receber leads de qualquer canal. | M | G |
| F1.E8.S2 | Como Gestora CO, quero importar leads/contactos/imóveis por ficheiro com pré-visualização e relatório de erros, para cargas manuais. | M | M |
| F1.E8.S3 | Como Sistema, quero o conector do scraper OLX/Imovirtual (dedupe por advertID/URL, filtro de concelhos configurável, empresa por distrito), para a captação automática. | M | M |
| F1.E8.S4 | Como Admin, quero gerir origens de lead (com flag "é página web"), para classificação de canais. | M | P |

### Épico F1.E9 — Comunicações: templates e envio
*Ref: PRD B§16, B§18, B§12.3*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F1.E9.S1 | Como Admin, quero templates de email e SMS com nome técnico único, corpo PT/EN/FR, variáveis tipadas, pré-visualização, remetente/reply-to e **flag ativo** (desligado = não envia mas regista), para paridade com triomphe.*.template. | M | M |
| F1.E9.S2 | Como Sistema, quero o gateway Usendit (validação nº PT, sender por template/operador, agendamento, mapeamento de erros) com credenciais em cofre, para envio de SMS. | M | M |
| F1.E9.S3 | Como Sistema, quero envio de email transacional multi-remetente (comercial, qualidade, no-reply…) com os layouts institucionais atuais, para paridade visual. | M | M |
| F1.E9.S4 | Como Gestora CO, quero o log central de todos os envios (destinatário, template, estado, detalhe do erro, utilizador) pesquisável + registo automático no histórico do registo de origem, para auditoria de comunicação. | M | M |

### Épico F1.E10 — Auditoria e activity log
*Ref: PRD A9, B§12.3*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F1.E10.S1 | Como utilizador, quero um histórico por registo (equivalente ao chatter: mudanças de campos tracked, mensagens automáticas, envios), para contexto completo. | M | G |
| F1.E10.S2 | Como Admin, quero trilha de auditoria de ações sensíveis (quem viu/alterou/exportou/apagou), para compliance. | M | M |

### Épico F1.E11 — Segurança de login
*Ref: PRD B§13.2*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F1.E11.S1 | Como Admin, quero registo de todos os logins (IP/hora/user), IPs autorizados, bloqueio de operadores fora de IPs de agência, bloqueio por tentativas com alertas SMS ao nº de segurança, para paridade com login.registry. | M | M |
| F1.E11.S2 | Como Admin, quero gestão de sessões (timeout, sessões múltiplas), para substituir tko_web_sessions. | S | P |

### Épico F1.E12 — Migração de dados (odoo_sync)
*Ref: PRD D1*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F1.E12.S1 | Como equipa de dev, quero ETL inicial do Odoo (agências, equipas, colaboradores, geografia, contactos, imóveis+media, prospectos+interações) com reconciliação de contagens, para arrancar com dados reais. | M | XG |
| F1.E12.S2 | Como equipa de dev, quero sincronização incremental bidirecional dos objetos ainda vivos no Odoo durante a convivência, para operar em paralelo. | M | XG |
| F1.E12.S3 | Como equipa de dev, quero mapear todos os IDs hard-coded do legado para papéis/configuração (tabela de de-para documentada), para nunca depender de IDs. | M | M |

### Épico F1.E13 — Admin Console: configurabilidade total ⭐
*Ref: PRD C10. Base transversal — a UI de cada área deve consumir esta configuração desde o início; funcionalidades avançadas de config chegam faseadas.*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F1.E13.S1 | Como equipa de dev, quero um **serviço de resolução de configuração em camadas** (Padrão do sistema → Grupo → Utilizador, override individual vence), consumido por menus, vistas, campos e botões, para que tudo seja configurável sem deploy. **CA:** cache com invalidação; qualquer ecrã pergunta "o que este utilizador vê/pode aqui?" a este serviço. | M | XG |
| F1.E13.S2 | Como Admin, quero um editor visual da **árvore de navegação** com visibilidade de cada menu **por grupo e por utilizador** (ex.: Utilizador Y não vê Relatórios; Grupo Operador vê Fila/Prospectos/Agenda), reordenação e menu inicial por grupo/utilizador. | M | G |
| F1.E13.S3 | Como Admin, quero configurar **vistas de listagem** por grupo/utilizador (tipo por defeito tabela/kanban/mapa, colunas visíveis+ordem, filtros pré-aplicados), para adaptar cada ecrã ao papel — ex.: a vista do assistente do CO. | M | G |
| F1.E13.S4 | Como Admin, quero configurar **layout de formulários** por grupo/utilizador (separadores/secções/campos visíveis, obrigatórios, só-leitura), para restringir edições por papel (ex.: operador não edita descrição pública). | M | G |
| F1.E13.S5 | Como equipa de dev, quero um **registo central de botões/ações** onde cada ação declara os seus parâmetros de comportamento, visibilidade e permissão, para que os botões sejam configuráveis a partir de um sítio único. | M | G |
| F1.E13.S6 | Como Admin, quero configurar **cada botão**: visibilidade por grupo/utilizador, permissão de execução, confirmação, texto/ícone/cor, e os **parâmetros de comportamento** — ex.: **"Não Atendido"** → dias de reagendamento (o prospecto fica na vista do assistente durante o dia e reagenda ao final do dia para **X dias configuráveis**), hora, envia SMS + template, anti-duplo-clique; **"Descartar"** → motivos disponíveis, desativa imóvel?, exige comentário?. **CA:** todos os valores hard-coded atuais (+5 dias, 22h, 60s, listas de motivos) viram parâmetros com defaults iguais aos de hoje. | M | XG |
| F1.E13.S7 | Como Admin, quero gerir **grupos e permissões** (criar/editar grupos, herança/implied, atribuir as ~40 permissões finas B§2.4, papéis funcionais) e overrides por utilizador, com **matriz de acesso visual** (grupos × entidades × CRUD). | M | G |
| F1.E13.S8 | Como Admin, quero **catálogos e parâmetros de negócio** editáveis (estados/estágios+cores, motivos, origens, tipos de negócio/imóvel, categorias de pontos, mapeamento agência↔distritos com exceções, tolerância/pesos do matching, janelas de PIN, limites de automação, IPs autorizados), para configurar sem código. | M | M |
| F1.E13.S9 | Como Admin, quero **"Ver como este utilizador"** (impersonation read-only) para validar menus/vistas/botões antes de guardar. | M | M |
| F1.E13.S10 | Como Admin, quero **governança da configuração**: auditoria (quem/quando/antes→depois), versões com reversão, rascunho→publicar, exportar/importar entre ambientes, salvaguardas (não remover último admin nem tirar permissões críticas a si próprio). | M | G |

---

## FASE 2 — OPERAÇÃO CO E VISITAS

### Épico F2.E1 — Fila do operador e distribuição de leads
*Ref: PRD B§3, B§5.4*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F2.E1.S1 | Como Operador, quero ver apenas a minha fila do dia (leads distribuídas com follow-up ≤ hoje), ordenada por hora agendada, para trabalhar sem ruído. | M | M |
| F2.E1.S2 | Como Gestora CO, quero distribuir leads manualmente (individual/lote) e transferir/retirar operadores com trilha de motivo, para gerir a operação. | M | M |
| F2.E1.S3 | Como Gestora CO, quero regras de distribuição automática configuráveis por distrito×tipo de imóvel com log de atribuições, para paridade com crm.assign.lead.config. | M | G |
| F2.E1.S4 | Como Sistema, quero limpar diariamente o "operador a trabalhar" (com exceções configuráveis), para libertar leads não trabalhadas. | M | P |
| F2.E1.S5 | Como Gestora CO, quero ações em massa (descartar, agendar chamadas, SMS, email, mover), para gestão de carteira. | M | M |

### Épico F2.E2 — Botões de teleprospecção
*Ref: PRD B§5.1*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F2.E2.S1 | Como Operador, quero o botão **Não Atendida**: anti-duplo-clique, fecha pendentes, SMS automático "liguei-lhe por causa do seu imóvel" (sender = meu nº se operador), incrementa contador, marca para reagendamento noturno — tudo registado, para o fluxo mais usado do CO. | M | M |
| F2.E2.S2 | Como Operador, quero o botão **Atendida** que abre o formulário de resultado (dados do imóvel pré-preenchidos) donde posso marcar visita, enviar email ou registar objeção, para converter a chamada. | M | G |
| F2.E2.S3 | Como Operador, quero botões WhatsApp / SMS recebido / Email website / SMS lembrete 20h, cada um com o seu registo e reagendamento próprio (regras B§5.1), para todos os desfechos atuais. | M | M |
| F2.E2.S4 | Como Operador, quero guiões de abordagem e objeções visíveis na ficha durante a chamada, para seguir o script. | M | P |
| F2.E2.S5 | Como Operador, quero a grelha de horários sugeridos (cruza agenda dos agentes da zona: visitas, escalas, atos, indisponibilidades) com a melhor hora destacada, para propor visita na hora. | M | G |
| F2.E2.S6 | Como Gestora CO, quero botões Concluir (congela) / Descartar (desativa imóvel) / Repor descartada, com permissões próprias, para fechar o ciclo. | M | P |

### Épico F2.E3 — Motor de automações v1 + automações de prospecção
*Ref: PRD C2, B§17*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F2.E3.S1 | Como Admin, quero criar regras de automação (gatilho evento/tempo + condições + ações + calendário/janela horária + limite diário + modo teste com redirect) em backoffice, com log por execução/destinatário, para substituir crons. | M | XG |
| F2.E3.S2 | Como Sistema, quero a regra "SMS de inserção" na criação de prospecto (exceto criado por avaliador; respeita flag do template), para paridade. | M | P |
| F2.E3.S3 | Como Sistema, quero o reagendamento noturno das não atendidas (+5 dias 09:00) e a cadeia de emails do CO (documentação D+3 → objeção D+5 recorrente, follow-up +1 dia útil), para paridade com os crons atuais. | M | M |
| F2.E3.S4 | Como Sistema, quero o descarte automático de leads com ≥3 emails sem resposta há 7 dias, com mensagem no histórico, para higiene do funil. | M | P |
| F2.E3.S5 | Como Admin, quero migrar as campanhas da "Comunicação Exterior" (segmentos em prospecção/qualificados, com/sem contacto, pós-visita) como regras configuráveis com log, para paridade com B§20. | M | G |

### Épico F2.E4 — Visita de Angariação
*Ref: PRD B§8.2, B§14 F3–F4*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F2.E4.S1 | Como Operador, quero criar visita de angariação a partir do prospecto (cria proprietário se não existir, conclui+congela o prospecto, propaga equipa/agente a lead/contacto/imóvel), para o fluxo central de marcação. | M | G |
| F2.E4.S2 | Como Gestora CO, quero a atribuição em cascata (criar → equipa → agente → pronto) com notificações/emails em cada passo (líder; agente com ficha de visita PDF; proprietário com template multilíngue), para paridade. | M | G |
| F2.E4.S3 | Como Operador, quero a grelha de disponibilidade dos agentes da equipa (fora de zona/escala/ocupado/livre) ao escolher agente/horário, para evitar conflitos. | M | M |
| F2.E4.S4 | Como Agente, quero registar o resultado por formulário guiado: Realizada e Angariada (imóvel → Angariado + pontos) / Realizada não angariada / Não realizada (falta agente/proprietário) — com motivos, para fechar a visita. | M | M |
| F2.E4.S5 | Como Agente, quero reagendar (limitado a 1× para agentes; nova visita ligada, antiga marcada reagendada com contador/motivo) e a Gestora cancelar (log de canceladas + libertação do prospecto), para os desvios do fluxo. | M | M |
| F2.E4.S6 | Como Gestora CO, quero confirmar a visita (SMS ao agente) e o fluxo de aprovação (notificação ao chefe), para controlo. | M | P |
| F2.E4.S7 | Como Qualidade, quero o formulário QC completo da visita (condições apresentadas, dossier entregue, estudo de mercado, reagendou/recusou…), para auditoria comercial. | M | M |
| F2.E4.S8 | Como Sistema, quero lembretes de ficha por preencher (email agente + CC gestor conforme equipa) e comunicações pós-visita/véspera como regras de automação, para paridade. | M | P |
| F2.E4.S9 | Como Agente, quero que datas de visita só mudem por reagendamento formal (não por edição direta), para integridade. | M | P |

### Épico F2.E5 — Visitas Comercial e de Avaliação
*Ref: PRD B§8.3–8.4*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F2.E5.S1 | Como Agente, quero visitas comerciais com comprador obrigatório e **contacto registado obrigatório** (auto-criação de contacto+resposta "visita agendada" quando aplicável), para paridade. | M | M |
| F2.E5.S2 | Como Sistema, quero anti-conflito de agenda (mesmo agente/imóvel/período) e emails a comprador, agente e agente do imóvel, para coordenação. | M | M |
| F2.E5.S3 | Como Agente, quero o relatório de visita comercial (impressões, fez proposta?, compraria?) e estados made/not_made/cancelled com motivos, para follow-up e estatística. | M | M |
| F2.E5.S4 | Como Agente, quero visitas de avaliação com o mesmo ciclo (agendar/confirmar/resultado/reagendar + SMS), para paridade. | M | M |
| F2.E5.S5 | Como Qualidade, quero o QC da visita comercial (5 perguntas de apreciação), para controlo. | M | P |

### Épico F2.E6 — Agenda unificada e contactos de compradores
*Ref: PRD B§8.1, B§7.2*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F2.E6.S1 | Como Agente, quero uma agenda unificada (3 tipos de visita + atos + escalas) filtrada pelo meu escopo, com vista de gestor para equipas, para visão do dia/semana. | M | G |
| F2.E6.S2 | Como Operador, quero registar contactos de compradores (objetivo info/visita; imóvel específico ou pesquisa com critérios completos e localizações multi-linha) e respostas (visita/seguimento/não interessado/não atendeu, data de seguimento), para o funil de compradores. | M | G |
| F2.E6.S3 | Como Sistema, quero os emails automáticos ao comprador no registo do contacto (específico/pesquisa com/sem zona, multilíngue) como regras, para paridade. | M | P |
| F2.E6.S4 | Como Agente, quero needaction/lembrete de seguimentos do dia e alerta "sem contacto há 5 dias", para não perder follow-ups. | M | P |
| F2.E6.S5 | Como Sistema, quero follow-up automático de compradores (7 dias, máx. configurável/dia, janela 10h–19h, opt-out por contacto) e arquivamento a 180 dias, para paridade. | M | P |

### Épico F2.E7 — Notificações internas
*Ref: PRD B§12.2*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F2.E7.S1 | Como utilizador, quero centro de notificações (tipos warning/contactos/imóveis/visitas/vendas, lidas/não lidas, badge, mute, links para o registo), para paridade com user.notification. | M | M |
| F2.E7.S2 | Como Sistema, quero todos os emissores de notificação da Parte B (§12.2) implementados via motor de eventos, para nenhum alerta se perder. | M | M |

### Épico F2.E8 — Central de Comunicações (documentação viva) ⭐
*Ref: PRD C2-A. Depende do motor de automações F2.E3.S1 — resolve o problema "ninguém sabe o que está ativo/quando é enviado".*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F2.E8.S1 | Como Gestora CO/Admin, quero a página **Central de Comunicações** organizada por audiência (Prospectos / Compradores / Proprietários / Clientes / Internos) e por etapa do funil, listando **todas** as comunicações automáticas em cartões com nome legível, canal, estado, template e responsável pela última alteração. **CA:** a lista é gerada do registo de regras do motor — nenhuma comunicação pode existir fora daqui. | M | G |
| F2.E8.S2 | Como Admin, quero, em cada comunicação, o **gatilho descrito em linguagem natural gerada automaticamente** (ex.: "Todos os dias às 22h, para prospectos não atendidos nesse dia"), condições/exclusões, janela horária e limite diário, para perceber o que dispara sem ler código. | M | M |
| F2.E8.S3 | Como Admin, quero **ativar/desativar** cada comunicação por toggle (com permissão `gerir comunicações`, auditado quem/quando/antes→depois) e ver **estatísticas** (último disparo, envios/falhas 30 dias), para controlo direto. | M | M |
| F2.E8.S4 | Como Admin, quero **indicadores de saúde**: aviso quando uma comunicação inativa interrompe uma cadeia (passo 2 de 3 desligado), quando há falhas recorrentes, ou quando o template está desativado (o "pisco" atual), para nada falhar em silêncio. | M | M |
| F2.E8.S5 | Como Admin, quero a **Vista de Jornada**: linha temporal visual da sequência por audiência (D0 → D+3 → D+5 → D+22…) com nós clicáveis, ramificações ("se atendeu, sai") e nós inativos a cinzento, exportável para PDF como documentação oficial. | M | G |
| F2.E8.S6 | Como Agente/Operador, quero na ficha do prospecto/contacto o painel "Comunicações automáticas" (já enviadas com link ao log + agendadas com data/hora e regra de origem), e um **simulador** que responde "que comunicações vai receber e quando", para responder ao "porquê recebeu isto?". | M | M |
| F2.E8.S7 | Como Admin, quero **modo de teste por comunicação** (redirect de destinatário com banner visível na Central) e **histórico de versões** de regras e templates ("o que estava ativo em março?"), para governança. | S | M |

---

## FASE 3 — VENDAS E FINANCEIRO

### Épico F3.E1 — Propostas
*Ref: PRD B§9.1*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F3.E1.S1 | Como Agente, quero criar propostas (comprador+imóvel+tipo negócio+valor+condições de pagamento: financiamento, sinal, reforços, prazo escritura) com validações de datas, para formalizar interesse. | M | M |
| F3.E1.S2 | Como Sistema, quero o ciclo de estados completo (waiting → not_approved / pending_proprietor ⇄ pending_buyer → accepted / cancelados → process) com histórico de movimentos de valor, para paridade. | M | M |
| F3.E1.S3 | Como Gestor, quero o fluxo de contrapropostas alternadas com registo de cada movimento e mensagens automáticas aos envolvidos, para a negociação. | M | M |
| F3.E1.S4 | Como Financeiro, quero que comissões divergentes das do contrato exijam aprovação da administração (notificação+email), para controlo. | M | P |
| F3.E1.S5 | Como Sistema, quero validação do comprador por código enviado por email (gerar/enviar/confirmar) e anexo obrigatório para transitar estados (exceto admin), para paridade. | M | M |
| F3.E1.S6 | Como Sistema, quero que aceitar uma proposta cancele automaticamente as concorrentes do mesmo imóvel (com mensagem) e mova o comprador para "Concluído", para consistência. | M | P |
| F3.E1.S7 | Como Gestor, quero notificações/emails de criação e mudança de estado a agentes, coordenadores e gestão (com log de despachos), para visibilidade. | M | M |
| F3.E1.S8 | Como utilizador autorizado, quero reverter/eliminar propostas com permissões próprias, e imprimir o PDF da proposta, para operações excecionais. | M | P |

### Épico F3.E2 — Processos de venda
*Ref: PRD B§9.2*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F3.E2.S1 | Como Agente, quero abrir processo a partir de proposta aceite, com participantes por defeito (proprietário+comprador+associados) e fichas KYC por participante, para formalizar. | M | G |
| F3.E2.S2 | Como Agente, quero as condições do processo (reserva, CPCV sim/não, sinal, reforço, percentagens, datas; ramo arrendamento: renda, duração, caução, rendas), para os dois tipos de negócio. | M | M |
| F3.E2.S3 | Como Gestor, quero o fluxo de aprovação em 3 níveis (agentes → gestor → financeiro → aceite) com rollbacks, needactions por perfil e validação de nº mínimo de participantes, para paridade. | M | M |
| F3.E2.S4 | Como Financeiro, quero que aceitar o processo gere os atos e as fichas de comissão, para arrancar a execução. | M | M |
| F3.E2.S5 | Como Financeiro, quero referências (vendedor/comprador referenciado com % e agente), venda partilhada e ficheiros do processo, para os casos especiais. | M | P |

### Épico F3.E3 — Atos e minutas
*Ref: PRD B§10.2–10.3*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F3.E3.S1 | Como Financeiro, quero atos (Adenda/Renda/CPCV/Escritura/Reforço) com gestor financeiro, partes, datas (mín. 1h) e estados waiting/concluded/cancel, agendados na agenda unificada, para gerir a execução. | M | M |
| F3.E3.S2 | Como Sistema, quero os efeitos de concluir ato: escritura/renda → imóvel vendido/arrendado + pontos CO + SMS às duas partes; CPCV → reservado; notificações ao grupo financeiro, para paridade. | M | M |
| F3.E3.S3 | Como Sistema, quero que mudar a data de um ato atualize as datas das comissões ligadas e notifique o financeiro, para consistência. | M | P |
| F3.E3.S4 | Como Financeiro, quero minutas por ato compostas de cláusulas reutilizáveis ordenáveis, com PDF, para os contratos. | M | M |
| F3.E3.S5 | Como Financeiro, quero reverter atos (repõe estágio anterior do imóvel), para correções. | M | P |

### Épico F3.E4 — Comissões
*Ref: PRD B§10.1*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F3.E4.S1 | Como Financeiro, quero fichas de comissão com numeração automática (VEN/ARR/OUT###) e o **cálculo exato da Parte B §10.1** (parcela, % empresa/agente, metades proprietário/comprador, penalização automática <4%→15% / <5%→10%, referências, retenção, IVA), recalculado a cada alteração, para paridade total. **CA:** bateria de testes com casos reais do legado. | M | G |
| F3.E4.S2 | Como Financeiro, quero o wizard de primeira abertura (tipo, valores, referência) e o fluxo waiting → done (gera código, envia PDF ao agente) → paid / cancel, com reversões, para o ciclo da ficha. | M | M |
| F3.E4.S3 | Como Agente, quero confirmar a minha comissão introduzindo o código recebido, e ver apenas comissões done/paid, para transparência controlada. | M | P |
| F3.E4.S4 | Como Sistema, quero o congelamento dos valores após revisão (PDF = ficha, sempre), e pontos ao agente na revisão (multiplicadores: referência=0, CPCV 100%=×2), para paridade. | M | M |
| F3.E4.S5 | Como Financeiro, quero movimentos de dinheiro (tipos internos/externos, previsões por período, por agência/agente) e o livro de registo de contratos com numeração automática, para o mapa financeiro. | M | M |

### Épico F3.E5 — Gamificação (pontos)
*Ref: PRD B§11.2*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F3.E5.S1 | Como Admin, quero categorias de pontos (globais ou por equipa, código técnico, ativo) e lançamentos com origem rastreável, para paridade. | M | M |
| F3.E5.S2 | Como Sistema, quero todas as atribuições automáticas da Parte B §11.2 (visitas, angariações, vendas, reduções de preço, comissões, escalas, faltas de login, zero angariações mensais, semanais/mensais de operadores) implementadas como regras, para nenhum ponto se perder. | M | G |
| F3.E5.S3 | Como Agente, quero ver os meus pontos e ranking no dashboard por período, para acompanhar a competição. | M | P |
| F3.E5.S4 | Como Gestor autorizado, quero atribuir/editar pontos manualmente com permissão própria e relatórios PDF de pontos, para gestão. | M | P |

### Épico F3.E6 — RH operacional
*Ref: PRD B§11.3–11.7*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F3.E6.S1 | Como Coordenadora, quero escalas com abertura/fecho restritos a IPs autorizados, painel do turno (atividade do período), checklist, lembretes e pontos por abertura, para paridade. | M | M |
| F3.E6.S2 | Como Gestor, quero tarefas com revisor, estados, recorrência (diária→trimestral) e lembretes automáticos, para a operação interna. | M | M |
| F3.E6.S3 | Como Agente, quero registar indisponibilidades (pessoais ou da agência, recorrentes) que alimentam as grelhas de disponibilidade, para agenda fiável. | M | P |
| F3.E6.S4 | Como Coordenadora, quero reuniões semanais com presenças, para registo. | S | P |
| F3.E6.S5 | Como Recrutador, quero o pipeline de recrutamento (estados, contactos, entrevistas, agendamento de sessões, SMS/email com templates), para paridade com hr.prospect. | S | M |

### Épico F3.E7 — Relatório semanal e relatórios/dashboards
*Ref: PRD B§11.8, B§12.1, B§19*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F3.E7.S1 | Como Sistema, quero gerar semanalmente o relatório de atividade por agente (linhas automáticas de visitas/contactos, edição limitada à semana), com lembrete e fecho+envio automático em PDF aos gestores, para paridade. | M | G |
| F3.E7.S2 | Como Direção, quero o email semanal de indicadores por agência (18 indicadores + legenda) gerado do novo modelo, para continuidade. | M | M |
| F3.E7.S3 | Como Gestor, quero o dashboard por perfil (cartão do agente com pontos/ranking/escalão, notícias, 6 gráficos com seletor de período), para paridade com B§12.1. | M | G |
| F3.E7.S4 | Como Gestora CO, quero os relatórios operacionais da Parte B §19 (chamadas CO, visitas canceladas, stock list com envio automático, preço/m², cliques web, comunicações, últimos contactos, exportações), priorizados por uso real, para continuidade. **CA:** lista fechada com o negócio; mínimo M: chamadas CO, canceladas, stock list, comunicações. | M | XG |
| F3.E7.S5 | Como Proprietário, quero receber o relatório mensal de atividade do meu imóvel (cliques, contactos, visitas, propostas, observações) por email/carta, para paridade com product.activity.report. | M | M |

---

## FASE 4 — RECOMENDAÇÕES E COMUNICAÇÃO

### Épico F4.E1 — Perfis de pesquisa do comprador ⭐
*Ref: PRD C1.1*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F4.E1.S1 | Como Agente, quero criar múltiplos perfis de pesquisa por comprador (orçamento + tolerância configurável, negócio, tipos, tipologias, áreas, comodidades obrigatórias vs preferidas, localizações multi-nível, financiamento, horizonte), para estruturar a procura. | M | M |
| F4.E1.S2 | Como Sistema, quero migrar as pesquisas existentes (`res.partner.prospection.message` tipo pesquisa + campos do partner) para perfis, para não perder dados. | M | M |
| F4.E1.S3 | Como Cliente, quero editar o meu perfil de pesquisa na área de cliente (fase 5 liga aqui), para autonomia. | S | P |

### Épico F4.E2 — Motor de matching e ciclo de vida da recomendação ⭐
*Ref: PRD C1.2–C1.3, C1.5*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F4.E2.S1 | Como Sistema, quero calcular score 0–100 por par comprador↔imóvel com pesos configuráveis em backoffice (base: fórmula atual `match_value` + orçamento com tolerância) e breakdown explicável, para transparência. | M | G |
| F4.E2.S2 | Como Sistema, quero persistir recomendações com estados (gerada → enviada → vista → interessado/rejeitada → convertida), dedupe (nunca repetir salvo alteração relevante) e reativação em redução de preço, para o ciclo de vida completo. | M | M |
| F4.E2.S3 | Como Agente, quero registar rejeições com motivo (caro/zona/tipologia/outro) e que isso ajuste o perfil/score, para o feedback loop. | M | M |
| F4.E2.S4 | Como Sistema, quero recalcular matches quando um imóvel muda (preço, características, estágio) ou um perfil muda, de forma incremental e assíncrona, para escala. | M | M |
| F4.E2.S5 | Como Sistema, quero incorporar sinais comportamentais do site (cliques, favoritos, pesquisas guardadas) no perfil, para melhorar relevância. | S | G |

### Épico F4.E3 — Área "Recomendações" ⭐
*Ref: PRD C1.2*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F4.E3.S1 | Como Agente, quero na ficha do comprador o separador "Recomendações" ordenado por score, com breakdown, estado, histórico de envios e ações (enviar, marcar visita, rejeitar), para trabalhar a carteira. | M | M |
| F4.E3.S2 | Como Angariador, quero na ficha do imóvel a lista de compradores compatíveis com score e ação de contacto, para ativar procura ao angariar. | M | M |
| F4.E3.S3 | Como Agente, quero a fila "Novas recomendações" (pares por trabalhar, ordenados por score×recência) no meu cockpit, para nada ficar parado. | M | M |
| F4.E3.S4 | Como Operador, quero ver o top-N de matches imediatamente ao criar/editar um perfil durante a chamada, para propor imóveis na hora. | M | P |

### Épico F4.E4 — Gatilhos e envios de recomendações ⭐
*Ref: PRD C1.4*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F4.E4.S1 | Como Sistema, quero gerar recomendações quando um imóvel é angariado/publicado e notificar os agentes dos compradores compatíveis, para reação imediata. | M | M |
| F4.E4.S2 | Como Agente, quero enviar recomendações ao cliente por email/WhatsApp com template e tracking de abertura/clique (modo assistido: eu aprovo; modo automático por perfil), para controlo da comunicação. | M | M |
| F4.E4.S3 | Como Sistema, quero o alerta de redução de preço aos compradores com recomendação ativa ("baixou de X para Y"), substituindo o email atual, para paridade+. | M | P |
| F4.E4.S4 | Como Cliente, quero um digest semanal "N novos imóveis para si" com frequência configurável e opt-out, para me manter informado sem spam. | S | M |
| F4.E4.S5 | Como Operador, quero no prospecto o contador "temos N compradores à procura de um imóvel como este" (recomendação inversa), para argumento de angariação. | S | M |

### Épico F4.E5 — Comunicação omnicanal e WhatsApp nativo
*Ref: PRD C3*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F4.E5.S1 | Como Agente, quero a timeline única de comunicações por contacto/prospecto (todas as fontes), para contexto completo. *(consolida F1.E7.S6 com WhatsApp e site)* | M | M |
| F4.E5.S2 | Como equipa, quero o Triomphe Chat integrado no novo CRM (mesma base Django: conversas, atribuição, templates, opt-in/out), substituindo o SSO por JWT, para plataforma única. | M | XG |
| F4.E5.S3 | Como Agente, quero click-to-call com registo automático da chamada e desfecho num clique, para produtividade. | S | M |
| F4.E5.S4 | Como Admin, quero estatísticas de abertura/clique por template de email, para otimizar comunicação. | S | M |

### Épico F4.E6 — Experiência do agente (PWA, push, pesquisa)
*Ref: PRD C4*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F4.E6.S1 | Como Agente, quero uma PWA mobile com agenda do dia, ficha da visita offline, upload de fotos (marca d'água no servidor), relatório de visita no local e assinatura do proprietário no ecrã, para trabalhar no terreno. | M | XG |
| F4.E6.S2 | Como utilizador, quero notificações push (substituindo SMS internos), para alertas imediatos e custo menor. | M | M |
| F4.E6.S3 | Como utilizador, quero pesquisa global full-text (nome, telefone, email, referência, morada), para encontrar tudo em segundos. | M | M |
| F4.E6.S4 | Como Agente, quero o cockpit de pendentes (visitas por confirmar, relatórios por preencher, recomendações novas, follow-ups em atraso), para um só ponto de trabalho. | M | M |
| F4.E6.S5 | Como Agente, quero sincronizar a minha agenda com Google/Outlook, para calendário único. | S | M |

---

## FASE 5 — SITE E ÁREA DE CLIENTE

### Épico F5.E1 — API pública
*Ref: PRD A11, B§15*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F5.E1.S1 | Como equipa de dev, quero a API pública documentada (OpenAPI) para montra, pesquisa, pedidos de contacto, favoritos e área de cliente, com baseline de segurança, para desacoplar o site. | M | G |
| F5.E1.S2 | Como Sistema, quero tracking de cliques/visitas por imóvel com GeoIP e anti-bot (blacklist datacenters, exclusão de IPs internos), para métricas fiáveis. | M | M |

### Épico F5.E2 — Website público
*Ref: PRD B§15*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F5.E2.S1 | Como Cliente, quero a montra de imóveis (listagem, filtros, detalhe, comparador, panorâmicas, favoritos, multi-idioma PT/EN/FR), para procurar imóveis. | M | XG |
| F5.E2.S2 | Como Cliente, quero guardar pesquisas com notificação de novos matches (liga ao motor de recomendações F4), para ser avisado. | M | M |
| F5.E2.S3 | Como Cliente, quero pedir contacto/visita a partir do imóvel (cria contacto+inquérito no CRM via intake), para ser atendido. | M | P |
| F5.E2.S4 | Como Cliente, quero a página da equipa com ficha dos agentes (QR vCard), para conhecer quem me atende. | S | P |

### Épico F5.E3 — Área de cliente
*Ref: PRD B§15, C6*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F5.E3.S1 | Como Cliente, quero autenticação moderna (magic link/OTP por email/SMS) mantendo migração dos Online IDs atuais, para acesso seguro. | M | M |
| F5.E3.S2 | Como Proprietário, quero ver os meus imóveis, estatísticas de atividade e o estado do processo, para acompanhar. | M | M |
| F5.E3.S3 | Como Comprador, quero ver e responder a propostas online (aceitar/contrapropor/cancelar com validação), para negociar à distância. | M | M |
| F5.E3.S4 | Como Cliente, quero mensagens de suporte bidirecionais (timeline no CRM com marcação de lida) e edição dos meus dados/perfil de pesquisa, para autonomia. | M | M |
| F5.E3.S5 | Como Cliente, quero upload de documentos pedidos no processo (alimenta KYC), para acelerar a venda. | S | M |

### Épico F5.E4 — Angariação online
*Ref: PRD B§14 F5*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F5.E4.S1 | Como Proprietário, quero o fluxo multi-fase de angariação online (dados → imóvel → fotos → contrato) com tracking de fase, para angariar sem visita. | M | G |
| F5.E4.S2 | Como Sistema, quero gerar o contrato eletrónico PDF com número do livro de registos, anexá-lo ao imóvel (estágio "Angariação Online") e notificar o coordenador, para paridade. | M | M |
| F5.E4.S3 | Como Cliente, quero assinatura digital do contrato (CMD ou equivalente), com o circuito por código como fallback, para validade formal. | S | G |

### Épico F5.E5 — Publicação em portais e Invest
*Ref: PRD C5, B§16*

| ID | Story | Prio | Tam |
|---|---|---|---|
| F5.E5.S1 | Como Marketing, quero feeds automáticos por portal (Idealista, Imovirtual, Casa Sapo) com estado/erros por imóvel-portal, para substituir o XML manual. | M | G |
| F5.E5.S2 | Como Sistema, quero a integração Habitar migrada (payload, estados, tentativas, remover, testar ligação, logs), para paridade. | M | M |
| F5.E5.S3 | Como Investidor, quero a área Invest (acesso restrito por aprovação, imóveis com rating/yields, conteúdo multilíngue), para o segmento de investimento. | S | M |
| F5.E5.S4 | Como Admin, quero RGPD self-service (exportação de dados do titular, esquecimento, retenção), para compliance completa. | M | M |

---

## FASE 6 — CUTOVER

### Épico F6.E1 — Operação paralela e reconciliação

| ID | Story | Prio | Tam |
|---|---|---|---|
| F6.E1.S1 | Como equipa de dev, quero relatórios diários de reconciliação Odoo↔novo (contagens e amostras por entidade), para detetar divergências na convivência. | M | M |
| F6.E1.S2 | Como Direção, quero rollout por ondas (equipa piloto → agência → todas) com feature flags e critérios de saída por onda, para reduzir risco. | M | M |
| F6.E1.S3 | Como Gestora CO, quero formação e guias por perfil (operador, agente, financeiro) alinhados com os fluxos migrados, para adoção. | M | M |

### Épico F6.E2 — Desativação do legado

| ID | Story | Prio | Tam |
|---|---|---|---|
| F6.E2.S1 | Como equipa de dev, quero desligar módulo a módulo no Odoo (escrita → leitura → arquivo), com o Odoo em modo leitura durante o período de garantia, para cutover seguro. | M | M |
| F6.E2.S2 | Como Admin, quero arquivo consultável do histórico não migrado (dumps + acesso read-only), para obrigações legais. | M | P |
| F6.E2.S3 | Como equipa de dev, quero redirecionamentos e desativação do site antigo após migração do público, para SEO e continuidade. | M | P |

---

## Roadmap pós-cutover (Could — sem fase atribuída)

| ID | Story | Ref |
|---|---|---|
| R.S1 | Matching semântico (embeddings descrição×notas) e modelo de propensão a visita/proposta | C1.5 |
| R.S2 | Lead scoring preditivo para priorizar a fila do CO | C9 |
| R.S3 | Resumos automáticos de conversas WhatsApp/chamadas na ficha | C9 |
| R.S4 | Geração de descrições de anúncio por IA (PT/EN/FR) com revisão humana | C5 |
| R.S5 | Sugestão de resposta ao operador baseada nos guiões | C9 |
| R.S6 | AVM completo (comparáveis + dias em mercado) e estudo de mercado automático | C5 |
| R.S7 | Badges/leaderboard em tempo real; escalas self-service com aprovação | C7 |
| R.S8 | Deteção de anomalias (comissões fora do padrão, visitas fantasma) | C9 |
| R.S9 | Gravação e transcrição de chamadas | C3 |
| R.S10 | Metas por agente/equipa com acompanhamento automático | C7 |

---

## Resumo quantitativo

| Fase | Épicos | Stories | Peso dominante |
|---|---|---|---|
| 1 — Fundação | 13 | 67 | Modelo de dados + RBAC + **Admin Console** + migração |
| 2 — Operação CO e visitas | 8 | 44 | Teleprospecção + visitas + automações + **Central de Comunicações** |
| 3 — Vendas e financeiro | 7 | 37 | Propostas→comissões + RH + relatórios |
| 4 — Recomendações e comunicação | 6 | 26 | ⭐ Motor de recomendação + PWA + WhatsApp |
| 5 — Site e área de cliente | 5 | 18 | Público + portais + RGPD |
| 6 — Cutover | 2 | 6 | Rollout e desativação |
| Roadmap | — | 10 | IA e otimizações |
| **Total** | **41** | **208** | |

**Dependências críticas entre fases:** o **Admin Console (F1.E13)** é transversal — o serviço de resolução de configuração em camadas (F1.E13.S1) deve nascer na fundação porque menus/vistas/botões de TODAS as áreas o consomem; F2 depende de F1.E7 (prospectos) e F1.E9 (comunicações); a **Central de Comunicações (F2.E8)** depende do motor de automações (F2.E3.S1); F3 depende de F2.E5 (visitas comerciais) e F1.E5/E6; F4.E2 depende de F1.E5/E6 e beneficia de F5.E1.S2 (sinais do site); F5.E2.S2 depende de F4.E2; o motor de automações (F2.E3.S1) serve F2–F5 e deve nascer cedo.
