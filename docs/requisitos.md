# Requisitos do Sistema — TaskFlow

> **Atividade:** Requisitos e Regras de Negócio
> **Milestone:** Requisitos

---

## Sumário

- [Requisitos Funcionais (RF)](#requisitos-funcionais-rf)
- [Requisitos Não Funcionais (RNF)](#requisitos-não-funcionais-rnf)
- [Regras de Negócio (RN)](#regras-de-negócio-rn)

---

## Requisitos Funcionais (RF)

### RF01 – Cadastro de usuário

**Descrição:** O sistema deve permitir que um novo usuário se cadastre informando nome completo, e-mail e senha.

**Critérios de aceitação:**
- [ ] O formulário valida formato de e-mail antes do envio.
- [ ] A senha deve ter no mínimo 8 caracteres; o sistema rejeita senhas menores com mensagem de erro clara.
- [ ] Após cadastro bem-sucedido, o usuário recebe e-mail de confirmação.
- [ ] Não é permitido cadastrar dois usuários com o mesmo e-mail.

---

### RF02 – Autenticação com e-mail e senha

**Descrição:** O sistema deve permitir que usuários cadastrados façam login com e-mail e senha e encerre a sessão (logout).

**Critérios de aceitação:**
- [ ] Login com credenciais válidas redireciona o usuário ao painel principal.
- [ ] Tentativas com credenciais inválidas exibem mensagem de erro sem revelar qual campo está incorreto.
- [ ] Após 5 tentativas falhas consecutivas, a conta é bloqueada por 15 minutos.
- [ ] O botão de logout encerra a sessão e redireciona para a tela de login.

---

### RF03 – Criação e edição de tarefas

**Descrição:** O sistema deve permitir que o usuário autenticado crie tarefas com título, descrição, data de vencimento e prioridade (baixa, média, alta), e edite qualquer um desses campos depois.

**Critérios de aceitação:**
- [ ] Todos os campos exceto descrição são obrigatórios na criação.
- [ ] A data de vencimento não pode ser retroativa ao dia atual.
- [ ] Alterações são salvas imediatamente e refletidas na listagem sem recarregar a página.
- [ ] O histórico de edições é registrado com data/hora e autoria.

---

### RF04 – Filtragem e busca de tarefas

**Descrição:** O sistema deve permitir que o usuário filtre suas tarefas por status (pendente, em andamento, concluída) e prioridade, além de buscar por palavra-chave no título ou descrição.

**Critérios de aceitação:**
- [ ] Filtros podem ser combinados (ex.: status = "pendente" E prioridade = "alta").
- [ ] A busca por palavra-chave retorna resultados em tempo real (máx. 300 ms após digitação).
- [ ] Quando nenhum resultado é encontrado, exibe mensagem informativa ao usuário.
- [ ] Os filtros ativos são visíveis e podem ser removidos individualmente.

---

### RF05 – Compartilhamento de tarefa com outro usuário

**Descrição:** O sistema deve permitir que o criador de uma tarefa a compartilhe com outro usuário cadastrado, concedendo permissão de visualização ou edição.

**Critérios de aceitação:**
- [ ] O compartilhamento é feito via e-mail do usuário destino.
- [ ] O usuário destino recebe notificação (in-app e e-mail) sobre o compartilhamento.
- [ ] O criador pode revogar o acesso a qualquer momento.
- [ ] Usuário com permissão apenas de visualização não consegue editar ou excluir a tarefa.

---

### RF06 – Exclusão de tarefa

**Descrição:** O sistema deve permitir que o criador de uma tarefa a exclua permanentemente.

**Critérios de aceitação:**
- [ ] A exclusão exige confirmação explícita do usuário (diálogo de confirmação).
- [ ] Tarefas excluídas ficam em uma "lixeira" por 30 dias antes da remoção definitiva.
- [ ] Usuários com quem a tarefa foi compartilhada recebem notificação de exclusão.

---

## Requisitos Não Funcionais (RNF)

### RNF01 – Desempenho de carregamento

**Descrição:** O sistema deve responder às ações do usuário dentro de limites de tempo aceitáveis, garantindo boa experiência de uso.

**Critérios de verificação:**
- [ ] O tempo de carregamento inicial da página (First Contentful Paint) deve ser ≤ 2 s em conexão de 10 Mbps.
- [ ] Operações de criação, edição e exclusão de tarefas devem ter resposta do servidor em ≤ 500 ms (p95).
- [ ] A busca em tempo real (RF04) deve retornar resultados em ≤ 300 ms após a última tecla digitada.

**Ferramenta de medição:** Lighthouse / k6 para testes de carga.

---

### RNF02 – Compatibilidade com navegadores e dispositivos

**Descrição:** O sistema deve funcionar corretamente nos principais navegadores modernos e em dispositivos móveis.

**Critérios de verificação:**
- [ ] Interface funcional e sem quebras visuais nos navegadores: Chrome ≥ 110, Firefox ≥ 110, Safari ≥ 16, Edge ≥ 110.
- [ ] Layout responsivo para resoluções a partir de 360 px de largura (mobile-first).
- [ ] Taxa de erro de JavaScript em produção ≤ 0,1 % das sessões (monitorado via Sentry ou equivalente).

---

### RNF03 – Segurança de dados

**Descrição:** O sistema deve proteger as informações dos usuários contra acesso não autorizado.

**Critérios de verificação:**
- [ ] Senhas armazenadas utilizando bcrypt com fator de custo ≥ 12.
- [ ] Comunicação cliente-servidor exclusivamente via HTTPS (TLS 1.2+).
- [ ] Tokens de sessão expiram em 24 h de inatividade.

---

## Regras de Negócio (RN)

### RN01 – Propriedade e permissões de tarefa

**Descrição:** Somente o criador de uma tarefa pode excluí-la ou compartilhá-la. Usuários com quem a tarefa foi compartilhada só podem editá-la se a permissão de edição tiver sido concedida explicitamente.

**Impacto nos RFs:** RF03, RF05, RF06  
**Critério de verificação:** Testes de unidade cobrindo tentativas de exclusão/compartilhamento por usuário não-criador devem retornar HTTP 403.

---

### RN02 – Limite de tarefas por plano gratuito

**Descrição:** Usuários no plano gratuito podem ter no máximo 20 tarefas ativas (status ≠ "concluída") simultaneamente. Ao atingir o limite, o sistema bloqueia a criação de novas tarefas e exibe mensagem orientando o upgrade ou conclusão de tarefas existentes.

**Impacto nos RFs:** RF03  
**Critério de verificação:** Ao tentar criar a 21ª tarefa ativa, a API retorna HTTP 422 com mensagem `"limite_atingido"` e a UI exibe o modal de upgrade.

---

### RN03 – Notificações de vencimento

**Descrição:** O sistema deve enviar notificação ao responsável pela tarefa (criador e colaboradores com permissão de edição) 24 horas antes da data de vencimento, apenas se a tarefa ainda não estiver concluída.

**Impacto nos RFs:** RF03, RF05  
**Critério de verificação:** Job agendado verificado em ambiente de staging: disparo de notificação ocorre entre 23 h 55 min e 24 h 05 min antes do vencimento para tarefas pendentes/em andamento.

---

## Rastreabilidade

| ID     | Tipo | Issues | Milestone     |
|--------|------|--------|---------------|
| RF01   | RF   | #1     | Requisitos    |
| RF02   | RF   | #2     | Requisitos    |
| RF03   | RF   | #3     | Requisitos    |
| RF04   | RF   | #4     | Requisitos    |
| RF05   | RF   | #5     | Requisitos    |
| RF06   | RF   | #6     | Requisitos    |
| RNF01  | RNF  | #7     | Requisitos    |
| RNF02  | RNF  | #8     | Requisitos    |
| RNF03  | RNF  | #9     | Requisitos    |
| RN01   | RN   | #10    | Requisitos    |
| RN02   | RN   | #11    | Requisitos    |
| RN03   | RN   | #12    | Requisitos    |
