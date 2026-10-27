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

## 6. Decisões em aberto

1. **Duas autorizações:** em sequência (A e depois B) ou qualquer uma basta? Cada setor autoriza o pedido inteiro ou parte dele (por exemplo, consumo para um e permanente para o outro)?
2. **Patrimônio:** existe inventário atual para importar? Haverá etiqueta com QR code ou código de barras? Precisa de transferência entre setores, devolução e baixa (desfazimento)?
3. **Termo de recebimento:** impresso e assinado à mão, com upload do escaneado, ou assinatura eletrônica? Quem recebe é sempre um servidor com login?
4. **Entradas:** o controle é só do contrato (saldo por item) ou também de empenho e nota? Doações e ajustes de inventário ficam fora ou entram como exceção com justificativa?
5. **Stack e hospedagem:** linguagem, framework e onde vai rodar (servidor da prefeitura, nuvem).
6. **Escopo do MVP:** qual é o mínimo para a SEMTRAS começar a usar?

## 7. Próximos passos

- Responder as decisões em aberto.
- Consolidar requisitos funcionais e modelo de dados (tenants, usuários e perfis, setores, itens, contratos, recebimentos, unidades patrimoniais, pedidos, autorizações, termos, auditoria).
- Definir a stack e o escopo do MVP.
