# Análise de requisitos — Sistema de controle de estoque e pedidos (SEMTRAS Santarém)

Documento de trabalho, baseado no PDF `docs/descricao basica/Projeto Controle de Pedidos.pdf` e nas instruções adicionais do solicitante. Pontos marcados como **em aberto** dependem de decisão.

## 1. Contexto

A Secretaria Municipal de Assistência Social de Santarém (SEMTRAS) e seus setores precisam pedir itens de consumo ou permanentes. Dois setores são responsáveis por autorizar os pedidos. Pedidos autorizados seguem para o setor de Compras, que controla o inventário e o estoque, levanta os itens autorizados e os prepara para entrega. É emitido um termo de recebimento com a relação completa dos itens e quem recebeu.

Requisitos adicionais:

- Itens de patrimônio devem ser rastreáveis: saber onde cada um está no momento.
- A entrada no estoque deve ser baseada em itens recebidos de contratos de licitação.
- Registro completo de quem fez qualquer ação (cadastro de item, movimentações etc.), com nome, data e hora.
- Arquitetura multi-tenant, para implantar o sistema em outros municípios como produto separado e independente.

## 2. O que o PDF define

- **Quatro fases:** entrada no estoque, pedido, autorização e distribuição, com rastreio de usuário e status em cada etapa.
- **Perfis:** Estoque, Solicitante, Gestor_autorizador e Distribuição. Um usuário pode acumular perfis.
- **Tabelas sugeridas:** usuários, produtos, estoque atual, movimentações, pedidos (cabeçalho) e itens do pedido.
- **Status do pedido:** Rascunho, Enviado_para_aprovação, Aprovado, Reprovado, Parcialmente_atendido, Atendido, Cancelado.
- **Quantidades por item:** solicitada, aprovada e distribuída.
- **Regras:**
  - saldo nunca negativo;
  - só o solicitante (ou seu setor) altera o próprio rascunho e não altera após o envio;
  - o solicitante não aprova o próprio pedido (segregação de funções);
  - a saída não pode passar da quantidade aprovada nem do saldo;
  - caixas são registradas em unidade padrão (1 caixa = 12 unidades, registrar 12);
  - histórico de alterações (quem aprovou e quando);
  - alerta de estoque mínimo (saldo ≤ mínimo).
- **Comprovante de distribuição:** nº do pedido, solicitante, setor, gestor autorizador, data e assinatura de quem recebeu.
- **Relatórios:** estoque atual por produto, consumo por setor e período, produtos mais utilizados, pedidos aprovados por gestor e período, itens sem saldo para atender pedidos.
- **Telas:** cadastro de produtos, cadastro de usuários/perfis, movimentações (entradas), pedidos, aprovação, distribuição, relatórios.

O PDF é um modelo genérico, com sugestões de implementação em planilha, e não cobre vários pontos pedidos para este sistema.

## 3. Lacunas entre o PDF e o pedido

| Tema         | PDF                               | Pedido                                                                            | Impacto                                                                                                         |
| ------------ | --------------------------------- | --------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| Autorização  | 1 gestor                          | 2 setores responsáveis                                                            | Fluxo com duas autorizações                                                                                     |
| Compras      | Não existe                        | Compras controla inventário e estoque, levanta os autorizados e prepara a entrega | "Estoque" e "Distribuição" viram o papel de Compras, com etapa de separação                                     |
| Patrimônio   | Só quantidade                     | Rastrear onde cada item permanente está                                           | Cada bem é uma unidade individual (nº de tombamento) com histórico de localização e responsável                 |
| Entrada      | Compra, doação, ajuste, devolução | Só a partir de itens recebidos de contratos de licitação                          | Entram fornecedor, processo/contrato/ata, itens contratados com saldo, nota fiscal e conferência de recebimento |
| Comprovante  | Opcional, com assinatura          | Termo de recebimento com lista completa e quem recebeu                            | Documento formal do sistema; para patrimônio, termo de responsabilidade                                         |
| Auditoria    | Só movimentações de estoque       | Quem fez qualquer coisa, com nome, data e hora                                    | Log de auditoria geral, só de inclusão, com valor antes e depois                                                |
| Multi-tenant | Não existe                        | Produto independente por município                                                | Decisão de arquitetura desde o início                                                                           |

## 4. Fluxo proposto

1. **Entrada:** contrato de licitação → recebimento conferido (nota fiscal) → itens de consumo entram no saldo; itens permanentes viram unidades tombadas.
2. **Pedido:** o solicitante de um setor monta o pedido com itens de consumo ou permanentes.
3. **Autorização:** o pedido passa pelas duas autorizações, com quantidade aprovada por item.
4. **Separação:** Compras levanta os itens autorizados e prepara a entrega.
5. **Entrega:** gera o termo de recebimento, o saldo baixa e os bens passam a ter o setor ou pessoa como localização.
6. **Auditoria:** cada passo registra autor, data e hora.

## 5. Multi-tenant

- **Tenant:** o município. Dentro dele ficam a secretaria e os setores.
- **Estratégia recomendada:** banco PostgreSQL compartilhado com `tenant_id` em todas as tabelas e Row-Level Security. É barato de operar e garante o isolamento no banco, não só no código. Se um município grande exigir isolamento total, ele pode migrar depois para banco próprio.
- **Configurações por tenant:** logo e cabeçalho dos termos, subdomínio, quantidade de autorizadores, numeração de pedidos e termos.
- **Usuários:** pertencem a um tenant.

## 6. Arquitetura modular

### Decisões tomadas

- **Objetivo da modularidade:** para a SEMTRAS, organização, entendimento e arquitetura do projeto. No futuro, venda por módulo. Por isso o código nasce modular e o cadastro de módulos ativos por tenant deixa espaço para licenciamento (ex.: vigência e plano), sem implementar cobrança agora.
- **Divisão em módulos:** aprovada (tabela abaixo).
- **Fluxo de autorização:** configurável por tenant.
- **Auditoria:** sempre ativa, não configurável, parte do núcleo.

### Núcleo (sempre ativo, não é módulo)

- Tenants, usuários, perfis e permissões.
- Estrutura organizacional (secretaria e setores).
- Auditoria.
- Catálogo de itens (tipo consumo ou permanente, unidade, categoria).
- Cadastro de módulos ativos e configurações do tenant.

### Módulos opcionais

| Módulo | O que faz | Depende de |
|---|---|---|
| Estoque | Saldo, movimentações, estoque mínimo e alertas dos itens de consumo | Núcleo |
| Patrimônio | Unidades tombadas, localização e responsável atuais, histórico, transferência, baixa e etiqueta/QR | Núcleo |
| Contratos e Recebimento | Fornecedor, processo/contrato/ata, saldo contratado e conferência de recebimento com nota fiscal; alimenta Estoque e Patrimônio | Estoque e/ou Patrimônio |
| Pedidos e Autorização | Pedido do setor e fluxo de aprovação com N etapas configuráveis | Estoque e/ou Patrimônio |
| Distribuição e Termos | Separação, termo de recebimento, termo de responsabilidade e assinatura | Pedidos |
| Relatórios | Consumo por setor, itens sem saldo, aprovações por gestor etc.; mostra os dos módulos ativos | Módulos ativos |

### Princípios de implementação

- **Monólito modular:** um único deploy, um módulo por pasta e tabelas próprias. Os módulos conversam por interfaces e eventos de domínio (ex.: "item recebido" publicado pelo recebimento, consumido por Estoque ou Patrimônio). Nenhum módulo lê tabelas de outro diretamente.
- **Esquema do banco único para todos os tenants:** ativar ou desativar um módulo só muda o acesso (menu, rotas e API), não cria nem remove tabelas.
- **Desativar esconde, não apaga:** os dados são preservados e a auditoria continua valendo.
- **Dependências declaradas no código:** o sistema impede ativar um módulo sem os que ele exige (ex.: Distribuição sem Pedidos).
- **Regras de negócio como configuração, não como módulo:** ex.: "entrada de estoque exige contrato" (SEMTRAS liga; outro município pode permitir entrada manual com justificativa).
- **Fluxo de autorização configurável por tenant:** etapas em ordem, com quem aprova cada uma (perfil ou setor), regra da etapa (qualquer um ou todos) e, se necessário, escopo por tipo de item (consumo ou permanente). A SEMTRAS configura uma etapa só, com a NSUAS 1 e a NSUAS 2 como aprovadoras (basta um veredito).

### Riscos

1. **Granularidade:** módulos demais multiplicam as combinações de teste. Manter os seis acima no início.
2. **Acoplamento escondido:** se um módulo ler dados de outro diretamente, a independência se perde.
3. **Telas e relatórios que cruzam módulos:** precisam degradar bem (ex.: o pedido mostra o saldo só se Estoque estiver ativo).
4. **Itens de consumo e permanentes no mesmo pedido:** se o tenant tiver só um dos módulos, catálogo e pedido filtram o tipo correspondente.

## 7. Stack e convenções de desenvolvimento

### Decisões tomadas

- **Backend:** Laravel 13.
- **Front-end:** Livewire v4, Alpine.js para interações em JavaScript e Tailwind CSS para estilização.
- **Autorização:** Laravel Permission (pacote `spatie/laravel-permission`, a confirmar) e Policies.
- **Rotas:** apontam para componentes de página Livewire, que renderizam views Blade.
- **Formulários:** a lógica de negócio dos cadastros fica nos formulários Livewire (Form objects). Cada form tem um método que retorna o array de regras de validação e outro com as mensagens em português do Brasil.
- **PDF:** exportação com DomPDF (termos de recebimento e de responsabilidade, relatórios).
- **Banco e infraestrutura:** PostgreSQL como serviço `db` no `docker-compose.yml` do projeto (ainda não existe). PostgreSQL é necessário para a Row-Level Security.
- **Multi-tenancy e módulos:** estrutura própria, sem pacotes de multi-tenancy ou de módulos.
- **Operações com estado:** classes de ação do módulo (ver ponto 4 abaixo); os Form objects ficam só com os cadastros.
- **Auditoria:** `owen-it/laravel-auditing`.

### Detalhamento

1. **Spatie Permission e multi-tenant:** usar o recurso de *teams* do pacote com `team_id` = `tenant_id`, para que perfis e permissões sejam por município. Os perfis do PDF (Estoque, Solicitante, Gestor_autorizador, Distribuição) viram papéis por tenant, e os papéis são semeados na criação do tenant.
2. **Isolamento de tenant:** trait `BelongsToTenant` com global scope, tenant resolvido por subdomínio em middleware, e Row-Level Security no PostgreSQL como segunda barreira. Implementação própria (decidido).
3. **Módulos:** um módulo por pasta (ex.: `app/Modules/Estoque`), com service provider próprio, rotas, componentes Livewire, models, policies e migrations. Um middleware `modulo:<nome>` bloqueia rotas de módulos inativos no tenant, e o menu só lista os ativos. Estrutura própria (decidido).
4. **Forms e operações com estado:** os Form objects cuidam dos cadastros (validação, `rules()`, `messages()` e persistência). Operações que movem estado entre módulos (receber contrato, aprovar etapa, separar, entregar, transferir bem) ficam em classes de ação do módulo, chamadas pelo componente, dentro de transação e emitindo evento. Assim a regra não fica presa à tela e a comunicação entre módulos segue por eventos.
5. **Nomes de campo nas mensagens:** além de `rules()` e `messages()`, cada form pode ter `validationAttributes()` para exibir os nomes dos campos em português.
6. **PDF com DomPDF:** o suporte a CSS é limitado (sem flex nem grid, e o Tailwind não se aplica bem). As views de PDF devem ser Blade próprias, com CSS simples e tabelas, separadas das views das telas. Pacote: `barryvdh/laravel-dompdf`.
7. **Auditoria:** `owen-it/laravel-auditing` (decidido), registrando autor, data, hora, IP e valores antes e depois. O registro deve ser somente de inclusão, com o usuário do banco da aplicação sem permissão de UPDATE ou DELETE na tabela.

## 8. Decisões de requisitos

### Decididas

- **Entradas fora de contrato** (doação, ajuste de inventário, saldo inicial): permitidas como exceção, com justificativa obrigatória e permissão específica, tudo auditado. A regra "entrada exige contrato" é configurável por tenant.
- **Nível de controle de contratos:** contrato com saldo por item (fornecedor, processo/contrato/ata, itens contratados com quantidade e saldo a receber, recebimento com nota fiscal). Empenho fica fora.
- **Patrimônio:** além da localização atual, o módulo faz importação do inventário existente, transferência entre setores, e devolução e baixa. Etiqueta com QR code não entra no MVP.
- **Banco e infraestrutura:** PostgreSQL como serviço `db` no `docker-compose.yml`.
- **Escopo do MVP:** núcleo e todos os módulos, exceto Relatórios avançados (só os relatórios básicos).

### Em aberto

1. **Autorizações da SEMTRAS (definido):** uma etapa só, com a NSUAS 1 e a NSUAS 2 como setores avaliadores. O pedido cai numa caixa de entrada vista pelos dois, e basta um servidor de qualquer um decidir (aprovar totalmente, aprovar parcialmente ou reprovar). A decisão grava o usuário, o setor dele, a data e a hora. O fluxo continua configurável por tenant. Em aberto: dois servidores decidindo o mesmo pedido ao mesmo tempo e se uma decisão pode ser revista.
2. **Termo de recebimento:** impresso com upload do escaneado, assinatura eletrônica ou ambos. Sem definição, o MVP começa com o PDF impresso. Falta também confirmar se quem recebe é sempre um servidor com login.

## 9. Próximos passos

- Conversar com a SEMTRAS sobre as duas pendências acima.
- Consolidar requisitos funcionais e modelo de dados (tenants, usuários e perfis, setores, itens, contratos, recebimentos, unidades patrimoniais, pedidos, autorizações, termos, auditoria).
- Montar a base do projeto: Laravel 13, `docker-compose.yml` com `db` PostgreSQL, Livewire v4, Tailwind, Alpine, Spatie Permission, owen-it/laravel-auditing e DomPDF.
