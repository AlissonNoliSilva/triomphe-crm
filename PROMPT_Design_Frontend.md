# Prompts para criar o design do Novo CRM com o Claude

## PROMPT PRINCIPAL (colar primeiro — cria o design system + shell + 1º ecrã)

---

Quero que cries o design de um **CRM imobiliário interno** chamado **Triomphe CRM** (imobiliária portuguesa com várias agências: Lisboa, Leiria, Setúbal, Algarve). Cria um protótipo em **React (ou HTML/CSS/JS único se for artifact)** totalmente navegável com dados fictícios realistas em português de Portugal.

### Identidade visual (obrigatória)

- **Azul primário `#1E3A8A`** — estrutura: sidebar, cabeçalhos, links, botões secundários.
- **Laranja de ação `#F97316`** (hover `#ea580c`) — APENAS para a ação principal de cada ecrã e destaques importantes. Máximo 1 CTA laranja visível por zona do ecrã.
- Neutros: fundo da app `#f5f5f5`, superfícies brancas, texto `#333`, texto secundário `#666`, bordas `#d0d7e2`, fundo suave azulado `#f3f6fb` (linhas alternadas de tabelas, hovers).
- Semânticas: sucesso `#16a34a`, aviso `#f59e0b`, erro `#b91c1c`.
- Gera escalas completas 50→900 do azul e do laranja como CSS variables/design tokens.
- Tipografia: Inter (ou system-ui), escala 12/13/14/16/18/22/28px. **Densidade compacta** — é uma ferramenta de trabalho intensivo usada 8h/dia, não um site de marketing: alturas de linha justas, paddings pequenos (tabelas com linhas de ~40px), muita informação por ecrã sem parecer caótico.
- Raios de 6–8px, sombras subtis, base de espaçamento 4px. Ícones estilo Lucide.

### Cores de estado (usar em badges/chips consistentes em toda a app)

- **Prospecto:** Por validar cinza · Por fazer azul · Não atendido laranja · Adiado âmbar · Enviado azul-claro · Recebido violeta · Concluído verde · Descartado cinza-escuro
- **Imóvel:** Em prospecção azul-claro · Por validar âmbar · Angariação online violeta · Angariado verde · Reservado laranja · Vendido verde-escuro · Arrendado teal · Cancelado cinza
- **Visita:** Agendada azul · Confirmada azul-escuro · Realizada e Angariada verde · Realizada sem angariação teal · Não realizada vermelho · Reagendada âmbar
- Badges: fundo na tonalidade 100 da cor + texto na 700 + ponto/ícone à esquerda.

### Contexto do produto (para os dados fictícios fazerem sentido)

O CRM cobre: prospecção telefónica de proprietários (Centro Operacional com operadoras), angariação de imóveis por agentes, visitas (angariação/comercial/avaliação), compradores com perfis de pesquisa e **recomendações automáticas de imóveis com score de match**, propostas com contrapropostas, processos de venda, comissões. Moeda: € com separador de milhares por espaço (ex.: "285 000 €"). Localizações portuguesas reais (Caldas da Rainha, Óbidos, Lisboa, Setúbal…). Tipologias T0–T4. Nomes portugueses.

### O que construir NESTE pedido

**1) Mini design system visível** (uma página/secção "Styleguide"):
- Paleta com os tokens; tipografia; botões (primário laranja, secundário azul, ghost, destrutivo, loading); badges de todos os estados listados acima; inputs/selects/toggles; card de KPI; avatar+nome ("célula pessoa"); tabs; toast.

**2) App Shell:**
- Sidebar azul-escuro `#1E3A8A` colapsável com navegação: Dashboard, Fila do Dia, Prospectos, Imóveis, Contactos, Agenda, Recomendações (com badge laranja "12"), Propostas, Comissões, Relatórios, Configurações.
- Topbar branca: pesquisa global (placeholder "Pesquisar contactos, imóveis, propostas… (Ctrl+K)"), seletor de agência ("Triomphe Lisboa ▾"), sino de notificações com badge, avatar do utilizador.

**3) O ecrã mais importante — "Ficha do Prospecto" (ecrã de trabalho da operadora de telemarketing durante a chamada):**
- **Cabeçalho:** nome do contacto ("Maria Fernandes"), telefones clicáveis, origem (badge "OLX"), estado (badge "Por fazer"), nº de tentativas (3), último contacto (há 5 dias).
- **Barra de ações de desfecho da chamada** (a zona mais usada — botões grandes e claros): `Atendida` (laranja, primário) · `Não Atendida` · `WhatsApp` · `SMS Recebido` · `Email Website` · separador · `Concluir` · `Descartar` (destrutivo ghost). Ao clicar "Não Atendida" mostrar um toast: "SMS enviado ao proprietário · Chamada reagendada automaticamente para +5 dias às 09:00".
- **Coluna esquerda (~55%):** cartão "Imóvel" com dados editáveis inline: tipo Apartamento, T3, Caldas da Rainha — Freguesia Nossa Sra. do Pópulo, 120 m², Venda 285 000 € (negociável), valor estimado da zona "≈ 262 000 €" com indicador de desvio +8,8%; grelha compacta de características (elevador ✓, varanda ✓, garagem ✗…).
- **Coluna direita (~45%):** painel colapsável "Guião de chamada" (abordagem + objeções em texto); "Nota privada" (textarea amarelada); **Timeline de interações** (chamada não atendida há 5 dias, SMS automático enviado, email enviado há 12 dias — cada item com ícone do canal, direção enviado/recebido, hora, e etiqueta "automático" quando aplicável).
- **Rodapé:** "Horários sugeridos para visita" — grelha semana × horas (Seg–Sáb, 9h–19h) com células verdes (livre), amarelas (parcial), vermelhas (ocupado) e a melhor hora destacada a laranja.

### Regras de qualidade

- Tudo em PT-PT. Dados fictícios coerentes (mesmos nomes/valores entre ecrãs).
- Estados de UI: mostrar pelo menos um exemplo de estado vazio e um skeleton loading.
- Interações reais: sidebar colapsa, tabs mudam, botões dão feedback (toasts), badges corretos.
- Responsivo até tablet (≥768px); o alvo principal é desktop 1440px.
- Acessibilidade: contraste AA no texto sobre azul/laranja, focus visible.

Começa por me mostrar o styleguide, depois o shell com a Ficha do Prospecto dentro.

---

## PROMPTS DE CONTINUAÇÃO (colar um de cada vez, na mesma conversa)

**2 — Ficha do Imóvel:** "Mantendo o mesmo design system e shell, cria a Ficha do Imóvel com separadores: Resumo (galeria de fotos com capa, dados-chave, preço com mini-histórico de alterações '295 000 → 285 000', score de qualidade do anúncio 78/100 com dicas), Características (grid de comodidades toggles + divisões com áreas), Angariação (datas do contrato com alerta 'termina em 23 dias', comissão 5%), Documentos (checklist legal: CMI ✓, Caderneta ✓, Certificado Energético ⚠ expira em 30 dias, Plantas ✗ + área de documentos bloqueada por PIN com cadeado), Visitas, Propostas, Compradores compatíveis (lista com score de match 0–100 em anel colorido). Barra de workflow no topo: Reservar / Vender / Arrendar / Cancelar."

**3 — Recomendações ⭐:** "Cria a área de Recomendações: (a) na ficha do comprador João Silva (orçamento 300 000 €, procura T2/T3 em Caldas e Óbidos): cards de imóveis ordenados por score com anel 0–100, breakdown expandível do score (Orçamento ✓ · Concelho ✓ · T3 ✓ · Piscina ✗), estado da recomendação (Nova/Enviada/Vista/Rejeitada) e ações Enviar · Marcar visita · Rejeitar com motivo; (b) fila 'Novas recomendações' do agente com pares comprador↔imóvel; (c) modal de envio com pré-visualização do email. O score usa gradiente vermelho→laranja→verde."

**4 — Dashboard + Agenda:** "Cria o Dashboard do agente (cartão pessoal com foto, escalão de comissão '2 (50%)', 1 240 pontos, ranking #3, seletor de período, 4 gráficos: chamadas, imóveis por estado, propostas, visitas) e a Agenda unificada semanal com cores por tipo (angariação azul, comercial laranja, avaliação violeta, escritura verde, escala cinza) e filtro por agente."

**5 — Kanbans + listas:** "Cria: (a) kanban de imóveis por estágio com cards (foto, ref, tipologia, zona, preço, agente); (b) tabela de prospectos da gestora com filtros, seleção múltipla e barra de ações em massa (Distribuir, SMS em massa, Descartar); (c) kanban de propostas por estado."

**6 — Financeiro:** "Cria a Ficha de Comissão VEN042: painel de cálculo em cascata mostrando cada linha (Valor transacionado 285 000 € → Comissão 5% = 14 250 € → Parcela CPCV 50% = 7 125 € → Empresa → Agente 50% = 3 562,50 € → metade proprietário/comprador → penalização → retenção/IVA → Total a receber) com os valores a recalcular ao editar percentagens; estados Em revisão/Revista/Paga; banner 'valores congelados' quando revista."

**7 — Mobile PWA:** "Cria as versões mobile (375px): home do agente com agenda do dia + pendentes; ficha de visita com botão 'Iniciar visita'; fluxo durante a visita: captura de fotos, formulário de resultado passo-a-passo e assinatura do proprietário no ecrã."

**8 — Central de Comunicações ⭐ (resolve a maior dor atual):** "Cria a página 'Central de Comunicações' — uma consola que serve ao mesmo tempo de configuração e de documentação viva de todas as comunicações automáticas do CRM. Cabeçalho com contadores '18 ativas · 6 inativas · 2 com falhas'. Separadores por audiência: **Prospectos · Compradores · Proprietários · Clientes · Internos**. Dentro de cada, cartões agrupados por etapa do funil, cada cartão com: nome legível (ex.: 'SMS de inserção', 'Reagendamento de não atendidas', 'Email de objeção D+5'), ícone do canal (SMS/Email/WhatsApp/Notificação), **toggle ativo/inativo proeminente**, gatilho em linguagem natural (ex.: 'Todos os dias às 22h, para prospectos não atendidos nesse dia'), template usado (com botão de pré-visualização), mini-estatística (último disparo, X enviados / Y falhas nos últimos 30 dias) e quem foi o último a alterar. Mostra badges de aviso a laranja/vermelho em cartões com problema ('cadeia interrompida', 'template desativado', 'falhas recorrentes'). Barra de filtros (audiência, canal, estado, texto). Inclui também uma segunda vista **'Jornada'**: um diagrama temporal horizontal da sequência de comunicações dos Prospectos (D0 SMS inserção → D+3 email documentação → D+5 email objeção recorrente → D+22 descarte automático), com nós clicáveis, nós inativos a cinzento tracejado e uma ramificação 'se atendeu → sai da jornada'. Usa azul para estrutura e laranja só nos destaques/CTAs."

**9 — Admin Console ⭐ (tudo configurável pelo gestor):** "Cria o 'Admin Console' onde o gestor configura a app sem programar, com o conceito de camadas (Padrão do sistema → Grupo → Utilizador, em que o override do utilizador vence). Faz 3 ecrãs: (a) **Menus & Navegação** — seletor de alvo (Grupo 'Operador CO' / Utilizador específico) + árvore de menus com toggle de visibilidade por item e um painel à direita que mostra a sidebar resultante para esse alvo (ex.: Operador vê Fila do Dia, Prospectos, Agenda; não vê Comissões nem Relatórios); (b) **Configurador de Botões** — lista de ações do sistema (Não Atendida, Atendida, Descartar, Concluir, Reservar…) e a ficha do botão 'Não Atendida' aberta com abas Visibilidade / Permissão / Aparência / **Comportamento**; na aba Comportamento mostra os parâmetros editáveis: 'Dias de reagendamento' = campo numérico com valor 5 (e ao lado, em cinza, 'padrão do sistema: 5'), 'Hora de reagendamento' = 09:00, 'Enviar SMS' = toggle ligado + seletor de template, 'Anti-duplo-clique (segundos)' = 60, com um texto explicativo 'O prospecto permanece na vista do assistente durante o dia e é reagendado automaticamente ao final do dia para X dias'; (c) **Matriz de Permissões** — tabela grupos × entidades (Prospectos, Imóveis, Propostas, Comissões…) × operações (Ver/Criar/Editar/Apagar) com checkboxes, e um botão 'Ver como este utilizador'. Mantém o design system; usa toggles e campos claros, densidade compacta."
