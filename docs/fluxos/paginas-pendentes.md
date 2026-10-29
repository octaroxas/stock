# Páginas HTML de demonstração — o que falta fazer

Lista de trabalho das páginas de apresentação do projeto. Todas usam a mesma stack (HTML único, Tailwind e Alpine.js por CDN) e o mesmo visual das páginas já feitas, em `docs/`.

## Já prontas

| Arquivo | O que mostra |
|---|---|
| `fluxos/01-fluxo-do-sistema.html` | Fluxo completo, rolando a página: entrada, pedido, autorização, separação, entrega e auditoria |
| `fluxos/02-fluxo-estocagem.html` | Só o ciclo do estoque: contrato, recebimento, saldo, patrimônio, alertas e exceções |
| `fluxos/00-configurador-autorizacao.html` | Montagem do fluxo de autorização por município, com simulação de um pedido |
| `fluxos/03-termos.html` | Termo de recebimento e termo de responsabilidade em A4, com as três formas de assinatura |
| `fluxos/04-modelo-de-dados.html` | Diagrama navegável das 26 tabelas por módulo, com `tenant_id`, Row-Level Security, restrições e a conferência de que os módulos só se ligam ao núcleo |
| `fluxos/05-perfis-e-permissoes.html` | Matriz de perfis e permissões, com montagem do usuário (perfis acumulados e permissão específica), menu resultante e teste de ações que separa o que é Permissão do que é Policy (segregação de funções) |
| `fluxos/06-mapa-de-modulos.html` | Liga e desliga os módulos, com o sistema bloqueando combinações inválidas (nos dois sentidos), o diagrama de dependências, o que some do menu, o comportamento das telas que cruzam módulos e os eventos entre módulos |
| `fluxos/07-maquina-de-estados-do-pedido.html` | Os sete status e as transições permitidas, com quem dispara cada uma (permissão, Policy e módulo ligado), simulação de um pedido do rascunho à entrega e o mapa no código |
| `fluxos/08-multi-tenant.html` | Santarém e um município fictício lado a lado (módulos, etapas, numeração, termos), o mesmo banco visto de cada um com as barreiras do global scope, do RLS e da FK composta ligáveis, cinco tentativas de acesso cruzado e o código do trait, do middleware e da policy |
| `fluxos/09-fluxo-geral.html` | Fluxo da solicitação à assinatura do termo em raias (setor solicitante, NSUAS 1 e NSUAS 2 na mesma raia como caixa de entrada em que basta um veredito, Setor de Compras e sistema), com simulação de um pedido (aprovação total, parcial ou reprovação, registrando usuário, setor, data e hora) que move o diagrama, o pedido e o estoque (físico, reservado, disponível e em posse do setor), histórico e pontos a confirmar |

## Pendentes

Ordem sugerida: 3, 9, 10. A 9 é a maior e fica por último.

| # | Página | Para quê | Base nos documentos |
|---|---|---|---|
| 3 | **Fluxo da entrega em detalhe** | Separação, termo, baixa do saldo e mudança de localização, passo a passo. O fluxo completo já cobre isso de forma resumida, então é a menos urgente | Seção 4, itens 4 e 5 |
| 9 | **Protótipo das telas** | Cadastro de produtos, usuários e perfis, movimentações, pedidos, aprovação, distribuição e relatórios, como base para os componentes Livewire | PDF (lista de telas) |
| 10 | **Dashboard de relatórios básicos** | Estoque atual, itens sem saldo, consumo por setor, produtos mais utilizados e aprovações por gestor | PDF (relatórios principais) e seção 8 (MVP só com relatórios básicos) |

## Convenções para manter

- Um arquivo `.html` por página, em `docs/`, sem build. Tailwind `cdn.tailwindcss.com/3.4.17` e Alpine `alpinejs@3.14.9` por jsDelivr.
- Dados fictícios (nomes, números, datas) e aviso disso no rodapé. Os dois setores autorizadores são "NSUAS 1" e "NSUAS 2": uma etapa só, e basta um servidor de qualquer um decidir.
- Pontos em aberto com a SEMTRAS aparecem num cartão tracejado em âmbar: regra das duas autorizações e forma de assinatura do termo.
- Selects com opções geradas por `x-for` precisam de `:selected` em cada `<option>`, senão o Alpine mostra a primeira opção em vez do valor do estado.
