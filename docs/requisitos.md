# Atividade - UNIFEOB - Ciência da Computação

> **Atividade:** Requisitos e Regras de Negócio  
> **Branch de referência:** `dev`  
> **Milestone:** Requisitos

---

## Índice

1. [Requisitos Funcionais (RF)](#requisitos-funcionais)
2. [Requisitos Não Funcionais (RNF)](#requisitos-não-funcionais)
3. [Regras de Negócio (RN)](#regras-de-negócio)

---

## Requisitos Funcionais

### RF-01 — Cadastro de usuário

**Descrição:** O sistema deve permitir que um novo usuário se cadastre informando nome completo, e-mail e senha.

**Critérios de aceitação:**
- [ ] O e-mail deve ser único no sistema; duplicatas retornam erro com mensagem clara.
- [ ] A senha deve ter no mínimo 8 caracteres, 1 letra maiúscula e 1 número.
- [ ] Após o cadastro, o usuário recebe um e-mail de confirmação.
- [ ] O cadastro sem confirmação de e-mail não concede acesso completo ao sistema.

---

### RF-02 — Criação de tarefa com prazo e prioridade

**Descrição:** O sistema deve permitir que um usuário autenticado crie uma tarefa contendo título, descrição, data de vencimento e nível de prioridade (Baixa, Média, Alta, Crítica).

**Critérios de aceitação:**
- [ ] Título é obrigatório (mínimo 3, máximo 120 caracteres).
- [ ] Data de vencimento não pode ser anterior à data atual.
- [ ] Se nenhuma prioridade for selecionada, o sistema atribui "Média" por padrão.
- [ ] A tarefa criada aparece imediatamente na lista do usuário sem necessidade de reload.

---

### RF-03 — Atribuição de tarefa a membro da equipe

**Descrição:** O sistema deve permitir que o criador de uma tarefa a atribua a um ou mais membros de sua equipe.

**Critérios de aceitação:**
- [ ] Somente membros da mesma equipe aparecem na lista de atribuição.
- [ ] O membro atribuído recebe uma notificação (in-app e e-mail) no momento da atribuição.
- [ ] É possível reatribuir ou remover a atribuição a qualquer momento antes da conclusão.

---

### RF-04 — Filtragem e busca de tarefas

**Descrição:** O sistema deve permitir que o usuário filtre tarefas por status, prioridade, responsável e intervalo de datas, além de realizar busca textual por título e descrição.

**Critérios de aceitação:**
- [ ] Múltiplos filtros podem ser combinados simultaneamente.
- [ ] A busca textual retorna resultados em até 1 segundo para bases de até 10.000 tarefas.
- [ ] Os filtros aplicados persistem durante a sessão do usuário.
- [ ] Existe opção de limpar todos os filtros com um único clique.

---

### RF-05 — Comentários em tarefas

**Descrição:** O sistema deve permitir que qualquer membro atribuído ou o criador da tarefa adicione, edite e exclua comentários.

**Critérios de aceitação:**
- [ ] Comentários exibem autor, data e hora de criação.
- [ ] Apenas o autor do comentário pode editá-lo ou excluí-lo.
- [ ] Comentários excluídos exibem "[comentário removido]" para manter o histórico de thread.

---

## Requisitos Não Funcionais

### RNF-01 — Desempenho de carregamento

**Descrição:** O sistema deve carregar a lista principal de tarefas em no máximo **2 segundos** em conexões de 10 Mbps, para listas de até 500 tarefas.

**Critérios de verificação:**
- [ ] Medido via Lighthouse com conexão simulada de 10 Mbps.
- [ ] Percentil 95 das requisições abaixo de 2 s em produção.
- [ ] Alerta automático disparado se p95 ultrapassar 3 s.

---

### RNF-02 — Compatibilidade de navegadores e dispositivos

**Descrição:** O sistema deve funcionar corretamente nos navegadores Chrome ≥ 110, Firefox ≥ 110, Safari ≥ 16 e Edge ≥ 110, em desktop (≥ 1280 px) e mobile (≥ 360 px).

**Critérios de verificação:**
- [ ] Testes de regressão visual cobrindo os 4 navegadores.
- [ ] Taxa de erros JS inferior a 0,1% das sessões por navegador.
- [ ] Nenhum elemento crítico fica inacessível em viewport de 360 px.

---

## Regras de Negócio

### RN-01 — Limite de tarefas abertas no plano gratuito

**Descrição:** Usuários no plano gratuito podem ter no máximo **15 tarefas** com status "Aberta" ou "Em andamento" simultaneamente.

**Critérios de verificação:**
- [ ] Ao tentar criar a 16ª tarefa ativa, o sistema bloqueia e exibe link para upgrade.
- [ ] Tarefas "Concluída" ou "Cancelada" não contam para o limite.
- [ ] O contador de tarefas ativas é exibido no dashboard.

---

### RN-02 — Prazo mínimo para cancelamento de tarefa atribuída

**Descrição:** Uma tarefa com responsável atribuído só pode ser cancelada pelo criador se o prazo for superior a **24 horas**, ou se o responsável confirmar o cancelamento.

**Critérios de verificação:**
- [ ] Cancelamentos dentro de 24 h do vencimento exibem modal solicitando aprovação do responsável.
- [ ] O responsável tem 1 hora para responder; silêncio implica aprovação.
- [ ] O histórico de cancelamento é registrado no log da tarefa.
