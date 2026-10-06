# Central de Alertas Zanelato

> Documento do projeto: contexto, objetivo, como o painel funciona, integrações e próximos passos.
> **Este documento não contém credenciais.** Os tokens da Z-API ficam somente nos secrets do Supabase e no documento confidencial de conexão, que é compartilhado por fora.

Atualizado em 06/10/2026.

---

## 1. Contexto

Hoje os alertas automáticos das campanhas chegam no grupo de WhatsApp **🚨 Alertas - Zanelato Marketing de Performance**. Os principais tipos são:

- desvio de investimento em relação ao planejado;
- CPA ou CPL fora do planejado;
- perda de leads;
- problemas de configuração (CPL/CPA não configurado, programação vencida);
- ciclos.

O grupo funciona como canal de entrega, mas tem limites claros:

- Os alertas se perdem no meio das mensagens.
- Não fica visível quem é o gestor responsável por cada conta.
- Não existe status, então não dá para saber o que já foi tratado e o que segue em aberto.
- Não há histórico nem indicador de evolução (alertas repetidos, problemas resolvidos no mês).

## 2. Objetivo

**Sair do grupo de WhatsApp como lugar de gestão e ter um painel claro e fácil de acessar.** Nele, cada alerta aparece:

- por conta, com o gestor responsável;
- com severidade (crítico, atenção, configuração);
- com status (aberto → notificado → resolvido);
- com um caminho curto até a resolução: avisar o gestor direto pelo ClickUp, em tarefa e mensagem.

O WhatsApp continua sendo a origem dos dados até a integração automática ficar pronta (ver seção 7). A gestão e o acompanhamento passam para o painel.

## 3. O painel

**Link:** https://claude.ai/code/artifact/94a793ff-f9c1-4a18-95da-69326f8c0e1b

### 3.1 Estrutura da tela

| Área | O que mostra |
|---|---|
| Barra superior | Logo, botões **Tema escuro**, **Modo TV**, **Gestores** e **Importar alertas** |
| Cabeçalho | "Central de alertas: o que precisa de ação hoje." |
| Barra de data | Atalhos **Hoje / Ontem / 7 dias**, navegação dia a dia, seletor de data e indicador **Ao vivo** |
| Ticker | Faixa rolante com os alertas do período |
| KPIs (3 × 2) | Contas com alerta · Abertos · Notificados · Resolvidos (anel de %) · Críticos em aberto · Repetidos. Cada um traz um minigráfico dos últimos 14 dias e a variação contra o dia anterior. Tocar no número filtra os cartões. |
| Corrida do mês | Problemas resolvidos acumulados por dia útil: mês atual × mês anterior × meta ("resolver tudo o que chegou") |
| Categorias | Chips para filtrar por tipo de alerta |
| Filtros | Status (Todos, Abertos, Notificados, Resolvidos), gestor e busca por conta ou cliente |
| Cartões | Um cartão por conta, com os problemas, os valores realizado × planejado e as ações |
| Visão de 7 dias | Matriz conta × dia, para enxergar alertas que se repetem |
| Modo TV | Tela cheia com slides em rotação automática, relógio, pausa e manutenção da tela ligada, para exibir no escritório |

### 3.2 Categorias e severidade

| Categoria | Exemplos de título | Severidade |
|---|---|---|
| Investimento | "Investimento X% abaixo/acima do planejado" | **Crítico** a partir de 30% de desvio; abaixo disso, atenção |
| CPL / CPA | "CPL/CPA X% acima do planejado" | **Crítico** a partir de 50% de desvio; abaixo disso, atenção |
| Perda de leads | "Perda de leads" | Sempre **crítico** |
| Configuração | "CPL/CPA não configurado", "Programação vencida", "Programação…" | Configuração |
| Ciclos | Mensagens "Alerta de Ciclos — conta" | Conforme o alerta |
| Outros | Qualquer título não reconhecido (ROAS, Orçamento, Connect rate…) | Atenção |

"Repetido" é o mesmo alerta, na mesma conta, por 2 ou mais dias seguidos.

### 3.3 Fluxo de uso

1. **Importar.** Em **Importar alertas**, o usuário cola as mensagens copiadas do grupo. O painel reconhece os dois formatos de data do WhatsApp, os cabeçalhos "Alerta — Conta" e "Alerta de Ciclos — Conta", e as linhas *Ontem/Hoje/Atual* (realizado) e *Planejado/Aceitável/Meta* (planejado). Antes de importar, mostra uma prévia. Alertas repetidos do mesmo dia são atualizados sem perder o status.
2. **Triar.** Pelos KPIs e filtros, a equipe vê o que está aberto e crítico.
3. **Notificar.** No cartão, **Notificar** monta a mensagem para o gestor com @menção, conta, data de referência, problemas e realizado × planejado. A pergunta final muda conforme o caso: "Pode ajustar pra gente?" para configuração e ciclos, "Consegue verificar e me dar um retorno?" para os demais. O painel pode criar uma **tarefa no ClickUp** na lista do projeto e enviar a **mensagem no canal** correspondente. O alerta passa para *notificado*. Também é possível marcar como notificado manualmente.
4. **Resolver.** O gestor ou o time marca o problema (ou o cartão inteiro) como *resolvido*. O resultado entra na corrida do mês.

### 3.4 Gestores

O modal **Gestores** permite cadastrar, renomear e remover gestores e vincular cada um ao seu usuário no ClickUp. A conta fica associada ao gestor, e esse vínculo vale para os próximos alertas da mesma conta.

## 4. Arquitetura

### 4.1 Dados do painel

O painel usa um banco compartilhado próprio, com atualização em tempo real para todos que estiverem com ele aberto.

| Coleção | Conteúdo |
|---|---|
| `alerts` | Um documento por problema: conta, título, categoria, severidade, data de envio, realizado, planejado, status e carimbos de importação, notificação e resolução (quem e quando) |
| `gestores` | Nome do gestor e vínculo com o usuário do ClickUp |
| `contas` | Conta → gestor responsável e preferências de notificação |

Importar e alterar exigem estar logado no claude.ai com permissão de escrita.

### 4.2 Integração com o ClickUp

- Workspace `3079757`, space `90142416486`.
- O painel lê a hierarquia de listas, os canais de chat e os membros.
- Ao notificar, cria a tarefa (`clickup_create_task`) e envia a mensagem no canal (`clickup_send_chat_message`).
- A lista é sugerida pelo código do produto no nome da conta (ex.: conta "Marcelo - PIER 2.0" → lista com `[PIER]`). O canal é o que tem o mesmo nome da lista.

### 4.3 Origem dos alertas: Supabase + Z-API

Os alertas são gerados por edge functions nos projetos Supabase e enviados ao WhatsApp pela Z-API.

| Origem | Função | Horário (BRT) | Cron (UTC) |
|---|---|---|---|
| Projetos de cliente com alerta de campanha (ex.: Vanisio) | `alerta-campanha` | 06:03 | `3 9 * * *` |
| Vanisio | `alerta-ciclo` ("Alerta de Ciclos") | 08:00 | `0 11 * * *` |

- **Grupo de destino:** `120363351788933902-group`, configurado na coluna `grupo_id` da tabela de configuração de cada função (ex.: `config_alerta_campanha`). O ID do grupo não é secret.
- **Secrets (em cada projeto Supabase, Project Settings → Edge Functions → Secrets):** `ZAPI_INSTANCE`, `ZAPI_TOKEN` e `ZAPI_CLIENT_TOKEN`. Os valores não estão neste documento.
- **Token de segurança da conta ativo:** toda chamada precisa do header `Client-Token`, senão a Z-API recusa.
- A mesma instância também envia relatórios para **outros grupos**. Exemplo: a `report-whatsapp` da Angélica Hendges (MDI e MDIBR) vai para outro grupo e não deve cair no de Alertas.
- **Troca de credenciais:** gerar um novo token invalida o antigo na hora. É preciso atualizar os secrets em **todos** os projetos que usam a Z-API, senão alertas e relatórios param.

## 5. Segurança

- As credenciais da Z-API **nunca** podem ir para o painel. A página roda no navegador de quem a abre, e qualquer pessoa com o link conseguiria ler os tokens e enviar mensagens pelo número da agência. Além disso, a página publicada bloqueia chamadas diretas para `api.z-api.io`.
- As credenciais também não devem ser coladas em grupos, chats ou documentos compartilhados. Elas ficam só nos secrets do Supabase e no documento confidencial de conexão.
- Qualquer integração entre painel e Z-API precisa passar por um backend que guarde os secrets, como as edge functions do Supabase.

## 6. Status atual

| Item | Status |
|---|---|
| Painel publicado (importação, KPIs, filtros, cartões, matriz de 7 dias, Modo TV, tema claro/escuro) | Pronto |
| Banco compartilhado (alerts, gestores, contas) com atualização em tempo real | Pronto |
| Notificação via ClickUp (tarefa + mensagem no canal) | Pronto |
| Revisão da integração Z-API | **Em andamento.** Pedido enviado à Ariane Crosa no canal #Claude do ClickUp em 06/10/2026 |
| Entrada automática dos alertas no painel (sem copiar e colar) | A definir |

### Checklist de revisão da Z-API (Ariane)

- [ ] A instância "Meu número" está conectada e ativa.
- [ ] `ZAPI_INSTANCE`, `ZAPI_TOKEN` e `ZAPI_CLIENT_TOKEN` estão cadastrados e atualizados em todos os projetos Supabase que enviam alertas ou relatórios.
- [ ] O `grupo_id` na `config_alerta_campanha` aponta para o grupo de Alertas.
- [ ] Os crons rodam e entregam: `alerta-campanha` às 06:03 e `alerta-ciclo` às 08:00 (BRT).
- [ ] Os envios para outros grupos (ex.: MDI e MDIBR da Angélica) vão para o grupo certo e não caem no de Alertas.

## 7. Próximos passos

1. **Fechar a revisão da Z-API** com a Ariane e corrigir o que ela encontrar.
2. **Definir a entrada automática dos alertas no painel**, para eliminar o copiar e colar do WhatsApp. O caminho é fazer as próprias edge functions (que já montam os alertas) gravarem também numa base que o painel leia, sem expor nenhuma credencial na página.
3. **Cadastrar todos os gestores** e vincular cada um ao seu usuário no ClickUp.
4. **Associar cada conta ao seu gestor**, para a notificação já sair com a pessoa certa.
5. **Migrar a rotina da equipe**: o painel passa a ser o lugar de triagem, notificação e resolução, e o grupo de WhatsApp vira só um canal de entrega até a integração automática ficar pronta.
6. **Avaliar o Modo TV** numa tela do escritório para acompanhamento diário.
