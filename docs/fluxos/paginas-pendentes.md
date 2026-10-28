# Páginas HTML de demonstração — o que falta fazer

Lista de trabalho das páginas de apresentação do projeto. Todas usam a mesma stack (HTML único, Tailwind e Alpine.js por CDN) e o mesmo visual das páginas já feitas, em `docs/`.

## Já prontas

| Arquivo | O que mostra |
|---|---|
| `fluxos/01-fluxo-do-sistema.html` | Fluxo completo, rolando a página: entrada, pedido, autorização, separação, entrega e auditoria |
| `fluxos/02-fluxo-estocagem.html` | Só o ciclo do estoque: contrato, recebimento, saldo, patrimônio, alertas e exceções |
| `fluxos/00-configurador-autorizacao.html` | Montagem do fluxo de autorização por município, com simulação de um pedido |
| `fluxos/03-termos.html` | Termo de recebimento e termo de responsabilidade em A4, com as três formas de assinatura |

## Pendentes

Ordem sugerida: 6, 5, 4, 8, 7, 3, 9, 10. As duas primeiras destravam a modelagem de dados e as policies; a 9 é a maior e fica por último.

| # | Página | Para quê | Base nos documentos |
|---|---|---|---|
| 6 | **Modelo de dados** | Diagrama das tabelas (tenants, usuários, setores, itens, contratos, recebimentos, bens, pedidos, autorizações, termos, auditoria), com `tenant_id` e Row-Level Security | `analise-requisitos.md`, seções 5, 7 e 9 |
| 5 | **Perfis e permissões** | Matriz de quem pode o quê, com troca de perfil para ver as telas mudarem. Inclui a segregação de funções e a permissão específica de entrada fora de contrato | Seção 2 e seção 7, item 1 (Spatie Permission com *teams*) |
| 4 | **Mapa de módulos e dependências** | Ligar e desligar módulos, com o sistema bloqueando combinações inválidas (ex.: Distribuição sem Pedidos) e mostrando o que some do menu e das telas | Seção 6, incluindo os riscos de telas que cruzam módulos |
| 8 | **Máquina de estados do pedido** | Todos os status e as transições permitidas, com quem pode disparar cada uma | PDF (status do pedido) e seção 4 |
| 7 | **Multi-tenant** | Dois municípios lado a lado, com módulos e configurações diferentes, mostrando que os dados de um não aparecem no outro | Seção 5 |
| 3 | **Fluxo da entrega em detalhe** | Separação, termo, baixa do saldo e mudança de localização, passo a passo. O fluxo completo já cobre isso de forma resumida, então é a menos urgente | Seção 4, itens 4 e 5 |
| 9 | **Protótipo das telas** | Cadastro de produtos, usuários e perfis, movimentações, pedidos, aprovação, distribuição e relatórios, como base para os componentes Livewire | PDF (lista de telas) |
| 10 | **Dashboard de relatórios básicos** | Estoque atual, itens sem saldo, consumo por setor, produtos mais utilizados e aprovações por gestor | PDF (relatórios principais) e seção 8 (MVP só com relatórios básicos) |

## Convenções para manter

- Um arquivo `.html` por página, em `docs/`, sem build. Tailwind `cdn.tailwindcss.com/3.4.17` e Alpine `alpinejs@3.14.9` por jsDelivr.
- Dados fictícios (nomes, números, datas) e aviso disso no rodapé. Os dois setores autorizadores aparecem como "Setor A" e "Setor B" enquanto não estiverem definidos.
- Pontos em aberto com a SEMTRAS aparecem num cartão tracejado em âmbar: regra das duas autorizações e forma de assinatura do termo.
- Selects com opções geradas por `x-for` precisam de `:selected` em cada `<option>`, senão o Alpine mostra a primeira opção em vez do valor do estado.
