# Backlog Frontend — Novo CRM Triomphe

**Objetivo:** guiar o design (Figma) e a implementação React do frontend interno do CRM. Ordenado pela sequência recomendada de design.

Convenções:
- **ID:** `FE.<área>.<item>` · **Prio:** M/S/C · Referências `B§x`/`Cx`/`Fx.Ey` apontam para o PRD e o Backlog geral.
- Cada ecrã deve ser desenhado com os 4 estados: **normal, vazio, carregando (skeleton), erro** — não repito isto em cada story.
- Personas com UI própria: **Operador CO**, **Agente**, **Gestora CO / Coordenadora**, **Financeiro**, **Admin**.

---

## FE.0 — Design System e Fundações **[desenhar primeiro]**

### FE.0.1 Identidade e cores

Cores da marca (extraídas dos templates institucionais atuais):

| Token | Valor | Uso |
|---|---|---|
| `brand-blue-900` | **#1E3A8A** | Cor primária: navegação, cabeçalhos, links, botões primários |
| `brand-blue-700` | #2f4bad *(derivar escala)* | Hover/estados do azul |
| `brand-blue-100…50` | derivar | Fundos suaves, linhas de tabela alternadas (#f3f6fb já usado nos emails) |
| `brand-orange-500` | **#F97316** | Cor de ação/destaque: CTAs, botão principal, badges de novidade |
| `brand-orange-600` | #ea580c | Hover do laranja |
| `neutral` | #f5f5f5 fundo · #333 texto · #666 secundário · #d0d7e2 bordas | Base |
| Semânticas | sucesso #16a34a · aviso #f59e0b · erro #b91c1c · info = brand-blue | Estados |

Regras de uso: **azul = estrutura e navegação; laranja = ação principal e destaque** (1 CTA laranja por ecrã, máx.). Gradientes suaves tipo email institucional apenas em cabeçalhos de páginas públicas — na app interna, cores sólidas.

| ID | Item | Prio |
|---|---|---|
| FE.0.1.1 | Definir escalas completas (50→900) de azul e laranja + neutros + semânticas, em tokens (CSS vars/Tailwind config), modo claro; preparar tokens para dark mode futuro | M |
| FE.0.1.2 | Tipografia: família (sugestão: Inter/system), escala (12/13/14/16/18/22/28), pesos, alturas de linha; densidade compacta (é uma ferramenta de trabalho intensivo) | M |
| FE.0.1.3 | Grelha e espaçamento (base 4px), raios (6–8px como nos emails), sombras, breakpoints (≥1280 desktop-first; 768 tablet; PWA móvel tratada em FE.15) | M |
| FE.0.1.4 | Iconografia (sugestão: Lucide) + logotipo/versões (o logo atual é laranja/azul sobre claro) | M |

### FE.0.2 Cores de estado do negócio (crítico — usar em badges, kanbans e calendários)

| Domínio | Estado → cor sugerida |
|---|---|
| Prospecto | Por validar cinza · Por fazer azul · Não atendido laranja · Adiado âmbar · Enviado azul-claro · Recebido violeta · Concluído verde · Descartado cinza-escuro |
| Imóvel | Em prospecção azul-claro · Por validar âmbar · Angariação online violeta · **Angariado verde** · Reservado laranja · Vendido verde-escuro · Arrendado teal · Cancelado cinza |
| Visita | Agendada azul · Confirmada azul-escuro · Realizada+Angariada verde · Realizada s/ angariação teal · Não realizada vermelho · Reagendada âmbar · Cancelada cinza |
| Proposta | Aguarda aprovação âmbar · Pendente proprietário azul · Pendente comprador violeta · Aceite verde · Cancelada vermelho · Não aprovada cinza · Em processo verde-escuro |
| Processo | Pendente agentes azul · Pendente gestor âmbar · Pendente aprovação laranja · Aceite verde · Cancelado cinza |
| Comissão | Em revisão âmbar · Revista azul · Paga verde · Cancelada cinza |
| Agenda unificada (tipo) | Angariação azul · Comercial laranja · Avaliação violeta · CPCV/Escritura verde · Escala cinza-azulado |

| ID | Item | Prio |
|---|---|---|
| FE.0.2.1 | Mapa completo estado→cor→ícone por domínio (tabela acima refinada com o negócio), documentado no design system | M |

### FE.0.3 Biblioteca de componentes (desenhar como stickersheet)

| ID | Componente | Notas | Prio |
|---|---|---|---|
| FE.0.3.1 | Botões (primário laranja, secundário azul, ghost, destrutivo, com ícone, loading) | | M |
| FE.0.3.2 | Badges/chips de estado (usando FE.0.2) + contadores | | M |
| FE.0.3.3 | **Tabela de dados**: ordenação, filtros por coluna, seleção múltipla + barra de ações em massa, paginação, colunas configuráveis, agrupamento, export | componente mais usado do CRM | M |
| FE.0.3.4 | **Kanban** por estágio (cards com foto/badge/valor, colunas dobráveis, drag & drop com permissão) | imóveis, contactos, propostas | M |
| FE.0.3.5 | Formulários: inputs, selects pesquisáveis (async), multi-select com chips, datas/horas, moeda (€ com separador de milhares — ver formato atual "1 234 567"), toggles, upload com drag&drop e pré-visualização | M |
| FE.0.3.6 | **Componente de morada PT** (zip→freguesia→concelho→distrito em cascata, B§4) | reutilizado em 6+ ecrãs | M |
| FE.0.3.7 | **Timeline de interações** (chamada/SMS/email/WhatsApp/site/nota; enviado vs recebido; automático vs humano com distinção visual; filtros por canal) | coração das fichas | M |
| FE.0.3.8 | Painel lateral de detalhe (drawer) e modais de wizard multi-passo (usados em todos os resultados de visita) | M |
| FE.0.3.9 | Calendário (mês/semana/dia/agenda) com cores por tipo e por agente | M |
| FE.0.3.10 | Cards de KPI + gráficos (barras, linhas, donut, funil) na paleta da marca | M |
| FE.0.3.11 | **Grelha de disponibilidade** (agentes × horas, verde/amarelo/vermelho com motivo — B§8) e **grelha de horários sugeridos** (dias × horas com melhor slot destacado — B§5.1) | componentes únicos do negócio | M |
| FE.0.3.12 | Toasts, banners, tooltips, popovers de confirmação (ações destrutivas sempre com confirmação) | M |
| FE.0.3.13 | Score de match 0–100 (anel/barra com cor gradual vermelho→laranja→verde) + breakdown expandível (C1) | M |
| FE.0.3.14 | Galeria de imagens (ordenar drag&drop, capa, marca d'água aplicada, lightbox) | M |
| FE.0.3.15 | Componente de log de comunicações (template, canal, destinatário, estado ENVIADO/FALHA com detalhe) | M |
| FE.0.3.16 | Avatares (utilizador/equipa), célula "pessoa" (avatar+nome+papel) | M |

---

## FE.1 — App Shell e Navegação **[desenhar segundo]**

| ID | Ecrã/Item | Conteúdo | Prio |
|---|---|---|---|
| FE.1.1 | **Layout geral**: sidebar azul-900 colapsável (navegação por módulo, filtrada por permissões) + topbar (pesquisa global, seletor de empresa/agência, sino de notificações com badge, avatar/menu) | A navegação muda por persona: Operador vê Fila/Prospectos; Agente vê Dashboard/Carteira/Agenda/Recomendações; Financeiro vê Processos/Comissões/Atos | M |
| FE.1.2 | **Pesquisa global** (cmd+K): resultados agrupados por tipo (contactos, imóveis, prospectos, propostas) com atalhos | M |
| FE.1.3 | **Centro de notificações**: dropdown (últimas 10) + página completa (agrupado Novas/Antigas, por data, tipos com ícone/cor, marcar lida, mute) — paridade B§12.2 | M |
| FE.1.4 | Seletor de empresa (multi-agência) afetando o escopo de dados visível | M |
| FE.1.5 | Ecrãs de autenticação: login (aviso de IP não autorizado para operadores), recuperação, sessão expirada | M |
| FE.1.6 | Página de perfil do utilizador (dados, preferências de notificação, idioma PT/EN/FR) | S |

---

## FE.2 — Dashboard (por perfil)
*Ref: B§12.1, F3.E7.S3*

| ID | Ecrã | Conteúdo | Prio |
|---|---|---|---|
| FE.2.1 | **Dashboard do Agente**: cartão pessoal (foto, escalão de comissão, pontos do período, ranking), seletor de período (dia/semana/mês/trimestre/ano), notícias da empresa, gráficos (minhas chamadas, meus imóveis por estado, propostas, cliques nos meus anúncios, visitas por estado) | M |
| FE.2.2 | **Dashboard do Gestor/Coordenadora**: mesmos blocos mas por membro da equipa; comparativo entre agentes | M |
| FE.2.3 | **Dashboard da Direção/Admin**: por agência; indicadores semanais (18 indicadores B§11.1) em tabela Lisboa/Leiria/Algarve/Total | M |
| FE.2.4 | **Cockpit do Agente** (pendentes): visitas por confirmar, relatórios por preencher, recomendações novas, follow-ups em atraso, compradores sem contacto há 5 dias — cards acionáveis (C4, F4.E6.S4) | M |
| FE.2.5 | Gestão de notícias do dashboard (admin, mensagens ≤90 chars) | S |

---

## FE.3 — Teleprospecção (Operador CO) **[área de maior densidade de interação — desenhar cedo]**
*Ref: B§5, F2.E1–E3*

| ID | Ecrã | Conteúdo | Prio |
|---|---|---|---|
| FE.3.1 | **Fila do dia do Operador**: lista ordenada por hora agendada (atrasadas no topo a vermelho), colunas: hora, contacto, telefone (click-to-call futuro), origem, estado, nº tentativas, último contacto; contador needaction no menu | M |
| FE.3.2 | **Ficha do Prospecto — ecrã de trabalho da chamada** (o ecrã mais importante do CO): cabeçalho com nome/telefones/estado + **barra de botões de desfecho** (Atendida · Não Atendida · WhatsApp · SMS Recebido · Email Website · Lembrete 20h · Concluir · Descartar); coluna esquerda: dados do imóvel (tipo, tipologia, zona, valores, características editáveis inline); coluna direita: **guião de abordagem/objeções** (colapsável), notas privadas, timeline de interações; rodapé: grelha de horários sugeridos (FE.0.3.11) | M |
| FE.3.3 | Wizard "Atendida" (multi-passo): resultado → marcar visita (com grelha de disponibilidade) / enviar email / registar objeção / reagendar — pré-preenchido com dados do imóvel | M |
| FE.3.4 | Feedback do botão "Não Atendida": confirmação com resumo do que vai acontecer (SMS enviado + reagendamento automático +5 dias), proteção anti-duplo-clique visível (60s) | M |
| FE.3.5 | Lista geral de prospectos (Gestora): filtros por estado/origem/operador/empresa/datas, ações em massa (distribuir, transferir, descartar, SMS/email em massa), indicador de duplicados | M |
| FE.3.6 | **Distribuição de leads**: ecrã de atribuição manual (selecionar leads → operador) + configuração de regras automáticas (matriz distrito × tipo de imóvel → operador) + histórico de atribuições | M |
| FE.3.7 | Alerta visual de duplicado na criação (painel com leads/contactos coincidentes por telefone, botão copiar) | M |
| FE.3.8 | Importação de leads: upload, mapeamento de colunas, pré-visualização com erros linha a linha | M |

---

## FE.4 — Contactos (Compradores / Proprietários)
*Ref: B§7, F1.E5, F2.E6*

| ID | Ecrã | Conteúdo | Prio |
|---|---|---|---|
| FE.4.1 | Lista de compradores e lista de proprietários: tabela + **kanban por estágio** (estágios B§7.1), filtros (agente, empresa, origem, estágio, "com pesquisa ativa"), pesquisa por imóvel de interesse | M |
| FE.4.2 | **Ficha do Contacto** com separadores: **Resumo** (dados, papéis comprador/proprietário com badge, agente, estágio, consentimentos por canal) · **Timeline** (FE.0.3.7) · **Pesquisas/Perfis** (FE.8) · **Recomendações** (FE.8) · **Imóveis** (se proprietário) · **Visitas** · **Propostas/Processos** · **Documentos** (cofre com PIN — ver FE.4.6) · **KYC** | M |
| FE.4.3 | Registo de contacto recebido (inquérito): formulário rápido objetivo info/visita, imóvel específico ou pesquisa, forma de contacto, durante escala; + registo de respostas (visita/seguimento/não interessado/não atendeu, data de seguimento) | M |
| FE.4.4 | Contactos associados (cônjuge/procurador/…): sub-lista na ficha com "usar mesma morada" | M |
| FE.4.5 | Painel de duplicados/merge: comparação lado a lado, escolha campo a campo, histórico de merges | M |
| FE.4.6 | **Cofre de documentos**: área bloqueada com estado visual (cadeado), modal de PIN, contagem decrescente da janela de acesso, log de acessos visível a auditores | M |
| FE.4.7 | Agências externas e grupos: lista + ficha simplificada com angariadores | S |
| FE.4.8 | Ficha KYC do participante (impressável) com indicador de completude "N/12" | M |

---

## FE.5 — Imóveis
*Ref: B§6, F1.E6*

| ID | Ecrã | Conteúdo | Prio |
|---|---|---|---|
| FE.5.1 | **Lista/kanban/mapa de imóveis**: alternância de vista; kanban por estágio; cards com foto de capa, referência, tipologia, zona, preço, badge de estado, agente; mapa com pins e cluster; filtros avançados (tipo, tipologia, faixa de preço, zona multi-nível, características, agente, empresa, dias em mercado) | M |
| FE.5.2 | **Ficha do Imóvel** com separadores: **Resumo** (galeria hero, dados-chave, negócios com histórico de preço em mini-gráfico, score de qualidade do anúncio com checklist de dicas) · **Características** (comodidades em grid de toggles, divisões com áreas, exposição solar) · **Localização** (morada + mapa editável lat/lng) · **Angariação** (contrato, comissões %, datas com alerta de fim, chaves, placa) · **Documentos** (checklist legal com validades + cofre PIN) · **Media** (galeria FE.0.3.14, vídeo, visita virtual) · **Atividade** (cliques com geo, contactos recebidos, relatórios mensais) · **Visitas** · **Propostas** · **Compradores compatíveis** (FE.8.3) · **Publicação** (site, portais, Habitar — estado por canal com erros) · **Histórico** | M |
| FE.5.3 | Barra de workflow do imóvel: transições de estágio como botões contextuais (Reservar, Vender, Arrendar, Cancelar, Reverter…) com wizards de motivo | M |
| FE.5.4 | Wizard de marca d'água em lote com pré-visualização antes/depois | M |
| FE.5.5 | Ecrã de publicação/portais: matriz imóvel×portal com estado, última sincronização, erros, ações enviar/remover | M |
| FE.5.6 | Preço/m² por freguesia: tabela de gestão + gráfico de evolução; sugestão de preço na ficha do imóvel (estimado vs pedido com % de desvio destacada) | M |
| FE.5.7 | Imóveis externos (agências parceiras) e área Invest (rating A–F, yields, conteúdo multilíngue) — variações da ficha | S |

---

## FE.6 — Visitas e Agenda
*Ref: B§8, F2.E4–E6*

| ID | Ecrã | Conteúdo | Prio |
|---|---|---|---|
| FE.6.1 | **Agenda unificada** (FE.0.3.9): filtros por tipo/agente/equipa/empresa; vista de gestor com sobreposição de agentes; legenda de cores FE.0.2 | M |
| FE.6.2 | **Criação/edição de visita de angariação**: fluxo em cascata com steps visíveis (Dados → Equipa → Agente → Pronto), grelha de disponibilidade embutida, resumo das notificações que serão disparadas | M |
| FE.6.3 | Ficha da visita: dados, estado com badge, ações (Confirmar, Resultado, Reagendar, Cancelar), histórico de notificações enviadas (FE.0.3.15), QC | M |
| FE.6.4 | **Wizards de resultado** (angariação: realizada+angariada / não angariada / não realizada; comercial/avaliação: made/not_made/cancelled) com motivos obrigatórios e consequências mostradas ("o imóvel passa a Angariado") | M |
| FE.6.5 | Formulário QC (angariação: checklist completa B§8.2; comercial: 5 perguntas) + fila de visitas a auditar para a equipa de Qualidade | M |
| FE.6.6 | Relatório de visita comercial (fez proposta? compraria? impressões) — mobile-friendly (preenchido no terreno) | M |
| FE.6.7 | Lista de visitas canceladas/reagendadas (relatório operacional com motivos) | S |

---

## FE.7 — Recomendações ⭐ **[novo — dar destaque visual laranja]**
*Ref: C1, F4.E1–E4*

| ID | Ecrã | Conteúdo | Prio |
|---|---|---|---|
| FE.7.1 | **Editor de perfil de pesquisa** do comprador: orçamento com slider+input e tolerância, negócio, tipos/tipologias, áreas, comodidades em dois grupos (obrigatórias/preferidas), localizações multi-linha (componente morada), financiamento; painel lateral live com "N imóveis correspondem agora" | M |
| FE.7.2 | **Separador Recomendações (ficha do comprador)**: cards de imóvel com foto, preço, score FE.0.3.13 com breakdown expandível ("porquê: orçamento ✓, concelho ✓, T3 ✓, piscina ✗"), estado da recomendação (badge), ações: Enviar (email/WhatsApp), Marcar visita, Rejeitar (com motivo); histórico de envios/vistas | M |
| FE.7.3 | **Separador Compradores compatíveis (ficha do imóvel)**: lista com score, orçamento, agente do comprador, ação contactar | M |
| FE.7.4 | **Fila "Novas recomendações"** (agente): pares comprador↔imóvel por trabalhar ordenados por score×recência, ações rápidas em linha | M |
| FE.7.5 | Modal de envio de recomendações: seleção de imóveis, pré-visualização do email/WhatsApp com template, modo assistido vs automático | M |
| FE.7.6 | Badge no prospecto: "🔥 N compradores procuram um imóvel como este" (recomendação inversa) com drill-down | S |
| FE.7.7 | Backoffice de pesos do score (sliders por critério com pré-visualização de impacto) | S |

---

## FE.8 — Propostas e Processos
*Ref: B§9, F3.E1–E2*

| ID | Ecrã | Conteúdo | Prio |
|---|---|---|---|
| FE.8.1 | **Pipeline de propostas** (kanban por estado + tabela), filtros por agente/imóvel/comprador/valor | M |
| FE.8.2 | **Ficha da proposta**: cabeçalho comprador↔imóvel↔valor com estado, linha do tempo de **movimentos** (original→contrapropostas→aceite como stepper/histórico visual), condições de pagamento, alerta de comissão divergente (pendente de aprovação), anexos (com bloqueio visual "anexo obrigatório para avançar"), validação por código (estado confirmado/pendente), ações contextuais por estado | M |
| FE.8.3 | Wizard de contraproposta (valor + condições, mostra de/para) | M |
| FE.8.4 | **Ficha do processo**: stepper de aprovação (Agentes → Gestor → Financeiro → Aceite), participantes com cards KYC (completude), condições (CPCV/reforço/escritura ou arrendamento), ficheiros, ações de rollback com motivo | M |
| FE.8.5 | Gestão de participantes: adicionar/editar ficha KYC completa em drawer | M |

---

## FE.9 — Financeiro
*Ref: B§10, F3.E3–E4*

| ID | Ecrã | Conteúdo | Prio |
|---|---|---|---|
| FE.9.1 | **Ficha de comissão**: cabeçalho (nº VEN/ARR/OUT, estado, agente, imóvel), **painel de cálculo em árvore/cascata** mostrando cada linha da fórmula (transacionado → comissão → empresa → agente → metades → penalização → referência → retenção/IVA → total) com valores a atualizar em tempo real ao editar percentagens; wizard de primeira abertura; estados com ações (Rever→envia PDF+código, Pagar, Cancelar, Reverter); banner "valores congelados" após revisão | M |
| FE.9.2 | Lista de comissões: agrupada por mês (org_date), filtros por estado/agente/tipo de ato; **vista do agente** restrita (só done/paid) com confirmação por código | M |
| FE.9.3 | **Atos**: lista + ficha (tipo, partes, datas, estado), ação Concluir com resumo de consequências ("imóvel passa a Vendido, SMS enviados às partes"), minutas com editor de cláusulas (biblioteca reordenável) | M |
| FE.9.4 | Movimentos de dinheiro: tabela por agência/agente/período, previsões vs reais | S |
| FE.9.5 | Simulador de comissão embutido na proposta (C6) | S |

---

## FE.10 — RH e Gamificação
*Ref: B§11, F3.E5–E7*

| ID | Ecrã | Conteúdo | Prio |
|---|---|---|---|
| FE.10.1 | Pontos: extrato por colaborador (categoria, origem clicável, data), **leaderboard** por período/equipa, gestão de categorias (pontos globais vs por equipa) | M |
| FE.10.2 | Escalas: calendário de turnos, abertura/fecho (com validação de IP e feedback claro), painel do turno (atividade do período), checklist | M |
| FE.10.3 | Tarefas: lista minha/equipa, estados, recorrência, lembretes | M |
| FE.10.4 | Indisponibilidades: calendário pessoal/agência com recorrência | M |
| FE.10.5 | **Relatório semanal de atividade**: formulário do agente (linhas automáticas de visitas/contactos + observações), estado de preenchimento, vista do gestor com todos os agentes | M |
| FE.10.6 | Recrutamento: kanban do pipeline (não contactado→a trabalhar), ficha com contactos/entrevistas/agendamentos | S |

---

## FE.11 — Comunicações
*Ref: B§18, C2–C3, F2.E3*

| ID | Ecrã | Conteúdo | Prio |
|---|---|---|---|
| FE.11.1 | **Editor de templates** (email/SMS): abas PT/EN/FR lado a lado, variáveis inseríveis, pré-visualização com dados de exemplo, contador de caracteres SMS, toggle Ativo com aviso do efeito, remetente | M |
| FE.11.2 | Log de comunicações: tabela global filtrada (canal, template, estado, destinatário, período) com detalhe do erro; estatísticas por template (enviados/falhados; abertura/clique quando disponível) | M |
| FE.11.3 | **Construtor de automações**: lista de regras com toggle ativo, editor (gatilho → condições → ações → calendário/limites) em formato de passos, modo teste com banner visível, log de execuções por regra | M |
| FE.11.4 | **WhatsApp (Triomphe Chat integrado)**: lista de conversas (atribuição, não lidas), janela de conversa com timeline, envio de templates aprovados, indicação de opt-out, ligação à ficha do contacto/prospecto | M |
| FE.11.5 | Campanhas em massa (SMS/email): seleção de segmento, template, agendamento, relatório de envio | S |

---

## FE.12 — Relatórios e Backoffice
*Ref: B§19, C8*

| ID | Ecrã | Conteúdo | Prio |
|---|---|---|---|
| FE.12.1 | Hub de relatórios: cards por relatório com filtros próprios e export (PDF/Excel) — priorizar: chamadas CO, visitas canceladas, stock list, comunicações, preço/m², últimos contactos | M |
| FE.12.2 | Backoffice de configuração: agências+distritos (com exceções), equipas, papéis funcionais, estágios e motivos, categorias de pontos, parâmetros (janelas PIN, limites de automação, tolerância de orçamento) | M |
| FE.12.3 | Gestão de utilizadores e permissões: perfis + permissões finas com pesquisa, simulador "ver como" (impersonation read-only) | M |
| FE.12.4 | Auditoria: pesquisa de eventos (quem/o quê/quando), logs de acesso a documentos, logins/IPs | M |

---

## FE.13 — PWA Mobile do Agente **[design próprio mobile-first]**
*Ref: C4, F4.E6.S1*

| ID | Ecrã | Conteúdo | Prio |
|---|---|---|---|
| FE.13.1 | Home mobile: agenda do dia + pendentes (cockpit reduzido) + atalhos | M |
| FE.13.2 | Ficha de visita mobile: dados essenciais, mapa/navegar, contactar proprietário, iniciar visita | M |
| FE.13.3 | Fluxo "durante a visita": captura de fotos multi (upload em fila, funciona offline), formulário de resultado passo-a-passo, **assinatura do proprietário no ecrã** | M |
| FE.13.4 | Recomendações mobile: rever e enviar ao cliente em 2 toques | S |
| FE.13.5 | Notificações push com deep-links | M |

---

## FE.14 — Website público e Área de Cliente (fase 5 — design separado da app interna)
*Ref: B§15, F5 — usa a mesma paleta mas linguagem "marketing" (gradientes suaves, fotografia grande, CTA laranja)*

| ID | Ecrã | Conteúdo | Prio |
|---|---|---|---|
| FE.14.1 | Montra: home, listagem com filtros, detalhe do imóvel (galeria, mapa, características, agente, CTA contacto laranja), comparador, favoritos, multi-idioma | M |
| FE.14.2 | Pesquisa guardada + alertas de novos imóveis (liga ao motor de recomendações) | M |
| FE.14.3 | Área de cliente: login OTP/magic link, dashboard, meus imóveis + estatísticas, propostas com resposta online, mensagens de suporte, meu perfil de pesquisa | M |
| FE.14.4 | Angariação online multi-fase com upload de fotos e contrato (assinatura digital/código) | M |
| FE.14.5 | Página da equipa; página Invest (acesso restrito) | S |

---

## FE.16 — Admin Console (configurabilidade total) ⭐ **[expande FE.12.2/12.3]**
*Ref: PRD C10, F1.E13 — o gestor/admin configura menus, vistas, botões e permissões por grupo e por utilizador, sem código.*

| ID | Ecrã | Conteúdo | Prio |
|---|---|---|---|
| FE.16.1 | **Editor de navegação/menus**: árvore de menus com toggle de visibilidade por item, alvo selecionável (grupo ou utilizador), reordenação drag&drop, definição de menu inicial; painel "resultado efetivo" mostrando a sidebar como o alvo a verá | M |
| FE.16.2 | **Editor de vistas de listagem**: por grupo/utilizador escolher tipo por defeito (tabela/kanban/mapa), colunas visíveis+ordem (drag), filtros pré-aplicados; pré-visualização ao vivo da tabela | M |
| FE.16.3 | **Editor de layout de formulários**: mapa de separadores/secções/campos de cada entidade com estados visível/obrigatório/só-leitura por grupo/utilizador | M |
| FE.16.4 | **Configurador de botões/ações** ⭐: lista de todas as ações do sistema; ficha por botão com abas **Visibilidade** (grupos/utilizadores), **Permissão**, **Aparência** (texto/ícone/cor/confirmação) e **Comportamento** (parâmetros específicos — ex.: "Não Atendido": dias de reagendamento [campo numérico], hora, envia SMS + template, anti-duplo-clique; "Descartar": motivos, desativa imóvel?, exige comentário?). Mostrar sempre o valor por defeito do sistema ao lado do override | M |
| FE.16.5 | **Gestão de grupos e permissões**: lista de grupos com herança, editor de permissões finas por grupo, atribuição a utilizadores + overrides; **matriz de acesso** (grupos × entidades × CRUD) filtrável | M |
| FE.16.6 | **Catálogos e parâmetros**: editores de estados/estágios (com escolha de cor por estado, alimentando FE.0.2), motivos, origens, tipos, mapeamento agência↔distritos (com exceções), pesos/tolerância do matching, janelas de PIN, limites de automação, IPs autorizados | M |
| FE.16.7 | **"Ver como este utilizador"** (impersonation read-only): seletor de utilizador + banner persistente "a ver como X" | M |
| FE.16.8 | **Governança**: histórico de alterações (antes→depois), versões com reversão, fluxo rascunho→publicar (com diff), exportar/importar config | S |

## FE.17 — Central de Comunicações (documentação viva) ⭐ **[expande FE.11.2/11.3]**
*Ref: PRD C2-A, F2.E8 — resolve "ninguém sabe o que está ativo, o que é enviado e quando".*

| ID | Ecrã | Conteúdo | Prio |
|---|---|---|---|
| FE.17.1 | **Ecrã principal por audiência**: separadores Prospectos / Compradores / Proprietários / Clientes / Internos, sub-agrupados por etapa; cada comunicação como cartão com nome legível, ícone do canal (SMS/Email/WhatsApp/Notif), **toggle ativo/inativo** proeminente, gatilho em linguagem natural, template (com pré-visualização em popover), estatística (último disparo, envios/falhas 30d), responsável pela última alteração. Cabeçalho com contadores "X ativas, Y inativas, Z com falhas" | M |
| FE.17.2 | **Filtros e barra de pesquisa**: por audiência, canal, estado (ativas/inativas/com falhas), template, texto livre; **badges de saúde** nos cartões (cadeia interrompida, template desativado, falhas recorrentes) com cor de aviso | M |
| FE.17.3 | **Vista de Jornada**: diagrama temporal horizontal por audiência (nós D0→D+3→D+5→D+22 com canal e estado; inativos a cinzento tracejado; ramificações "se atendeu, sai"), nós clicáveis para a regra; botão exportar PDF | M |
| FE.17.4 | **Painel na ficha do prospecto/contacto**: "Comunicações automáticas" — enviadas (com link ao log) e agendadas (data/hora + regra); **simulador** "que comunicações vai receber e quando" | M |
| FE.17.5 | Detalhe/edição de uma comunicação: liga ao construtor de automações (FE.11.3), com banner de modo de teste quando ativo; histórico de versões | S |

---

## Sequência recomendada de design

1. **FE.0** Design system (tokens, cores de estado, stickersheet de componentes) — desbloqueia tudo.
2. **FE.1** Shell + navegação por persona.
3. **FE.3** Teleprospecção (FE.3.2 primeiro — é o ecrã mais denso; valida a densidade do design system).
4. **FE.5.2** Ficha do imóvel e **FE.4.2** ficha do contacto (padrão de "ficha com separadores + timeline" que se replica em tudo).
5. **FE.6** Visitas/agenda (calendário + wizards + grelha de disponibilidade).
6. **FE.2** Dashboards e **FE.7** Recomendações (a área nova ⭐ — vender o conceito internamente).
7. **FE.8/FE.9** Propostas, processos e ficha de comissão (painel de cálculo).
8. **FE.17** Central de Comunicações ⭐ e **FE.11** Comunicações/templates — resolvem a maior dor atual; desenhar cedo para validar com o negócio.
9. **FE.16** Admin Console ⭐ (menus/vistas/botões configuráveis) e **FE.10** RH.
10. **FE.13** PWA mobile (kit mobile próprio).
11. **FE.14** Site público (projeto de design separado).

> Nota de arquitetura de design: como o **FE.16 Admin Console** controla menus/vistas/botões de toda a app, o design system (FE.0) deve prever desde já os "estados configuráveis" — um botão pode estar oculto/visível/desativado por config; uma coluna pode não existir para um perfil. Desenhar os componentes já a pensar nisso evita retrabalho.

## Resumo

| Área | Itens | Prioridade dominante |
|---|---|---|
| FE.0 Design system | 21 | M |
| FE.1 Shell | 6 | M |
| FE.2 Dashboards | 5 | M |
| FE.3 Teleprospecção | 8 | M |
| FE.4 Contactos | 8 | M |
| FE.5 Imóveis | 7 | M |
| FE.6 Visitas/Agenda | 7 | M |
| FE.7 Recomendações ⭐ | 7 | M |
| FE.8 Propostas/Processos | 5 | M |
| FE.9 Financeiro | 5 | M |
| FE.10 RH | 6 | M |
| FE.11 Comunicações | 5 | M |
| FE.12 Relatórios/Backoffice | 4 | M |
| FE.13 PWA Mobile | 5 | M |
| FE.14 Site público | 5 | M (fase 5) |
| FE.16 Admin Console ⭐ | 8 | M |
| FE.17 Central de Comunicações ⭐ | 5 | M |
| **Total** | **117** | |
