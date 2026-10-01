# Gestor de Obras do Condomínio Constitution

## Core Principles

### I. Motor de Cronograma como Função Pura (Test-First)
O motor de cronograma vive isolado em `/lib/engine`, em TypeScript puro, sem dependência de UI,
banco de dados ou qualquer efeito colateral: mesmo estado de entrada DEVE sempre produzir a mesma
saída. Nenhum comportamento novo do motor é aceito sem um teste que o descreva primeiro — os 12
casos de aceite da seção 7 do PRD são a especificação executável mínima e DEVEM passar antes de
qualquer tela ser construída sobre o motor.
Rationale: o motor é o núcleo de valor do produto (projeção financeira que a assembleia usa para
decidir); isolá-lo e testá-lo primeiro é o que permite confiar no recálculo em tempo real.

### II. Dinheiro em Centavos Inteiros
Todo valor monetário é armazenado e calculado como inteiro em centavos. Arredondamento para
exibição (R$) acontece só na camada de apresentação, nunca dentro do motor ou do armazenamento.
Meses são representados como string `AAAA-MM`.
Rationale: ponto flutuante em dinheiro introduz erros de arredondamento cumulativos; um projeto
que recalcula a cada edição não pode tolerar essa deriva.

### III. Contratos Imutáveis e Versionados
Um contrato (obra + proposta vencedora + cronograma de parcelas) nunca é sobrescrito ou apagado.
Toda renegociação gera uma nova versão do contrato, preservando o histórico de versões anteriores.
Anexos usam exclusão lógica. Assembleias congeladas são snapshots imutáveis do plano aprovado.
Rationale: o produto existe para comparar decisões ao longo do tempo (assembleia a assembleia);
apagar ou sobrescrever histórico destrói essa capacidade de auditoria e comparação.

### IV. O Sistema Alerta, Nunca Altera Contrato Sozinho
Quando uma mudança de taxa, receita ou prioridade torna uma obra contratada inviável, o motor
DEVE sinalizar "Renegociação necessária" com déficit e sugestão de reescalonamento, mas NUNCA
aplicar essa mudança automaticamente ao contrato. Só o conselheiro, aceitando a sugestão ou
digitando um cronograma negociado, gera a nova versão do contrato (ver Princípio III).
Rationale: compromissos financeiros com terceiros (empresas contratadas) exigem decisão humana
explícita; automatizar essa alteração seria uma mudança de contrato sem consentimento.

### V. Multi-tenant Ready (`condominio_id` em Tudo)
Toda entidade do modelo de dados carrega `condominio_id`, mesmo que o MVP opere com um único
condomínio e nenhuma tela exponha seleção de condomínio. Nenhuma query ou regra de negócio pode
assumir implicitamente "o único condomínio existente" de um jeito que impeça filtrar por
`condominio_id` depois.
Rationale: a visão de produto pós-MVP é multi-condomínio para administradoras; a seção 1 do PRD
exige que o modelo de dados já nasça preparado para isso, sem pagar o custo de implementar
multi-tenant agora.

### VI. MVP Sem Autenticação, Preparado para Auth e RLS Futuros
O MVP não implementa autenticação, perfis ou permissões; o acesso é por URL não divulgada. Nenhuma
decisão de modelagem (schema, chamadas ao Supabase, estrutura de rotas) pode tornar inviável
adicionar depois autenticação e Row Level Security por `condominio_id` sem reescrita estrutural.
Rationale: autenticação está explicitamente fora do escopo do MVP (seção 2 do PRD), mas é uma
adição planejada e conhecida — não deve virar retrabalho.

### VII. Simplicidade e Escopo Fechado
Nenhuma funcionalidade fora da coluna "Dentro do MVP" da seção 2 do PRD é implementada sem
decisão explícita de mudar o escopo. Itens listados como "Fora do MVP (2.0+)" (fundo de reserva,
multi-tenant, integração com ERP, notificações, autenticação, cenário pessimista, etc.) não
nascem como código morto, flags desligadas ou abstrações antecipadas.
Rationale: o PRD já fechou essas decisões (seção 10); reabri-las silenciosamente durante a
implementação é a forma mais comum de um MVP de orçamento fechado estourar prazo e complexidade.

### VIII. Interface em Português do Brasil, Formato Monetário Brasileiro
Toda interface voltada ao usuário (telas, mensagens, rótulos, exportações em PDF/CSV) é escrita em
português do Brasil, com valores em formato R$ brasileiro e meses exibidos como "mar/2027"
(armazenados como `AAAA-MM`, ver Princípio II).
Rationale: o único usuário do MVP é um conselheiro de condomínio brasileiro apresentando em
assembleia; qualquer inconsistência de idioma ou formato quebra a credibilidade do número na hora
da apresentação.

## Restrições Técnicas
<!-- Seção 8 do PRD: requisitos não funcionais -->

- Recálculo do motor no cliente em menos de 200ms para 30 obras, sem round-trip ao servidor a cada
  ajuste do dashboard.
- Cobertura de testes do motor (`/lib/engine`) de no mínimo 90%.
- Persistência automática: nenhum dado é perdido ao fechar a aba.
- Toda alteração em contrato gera nova versão (ver Princípio III); nada é apagado via exclusão
  física — apenas lógica.

## Fluxo de Desenvolvimento
<!-- Seção 9 do PRD: roteiro Spec Kit -->

O projeto segue o fluxo Spec Kit: `/speckit-constitution` → `/speckit-specify` → `/speckit-clarify`
(opcional) → `/speckit-plan` → `/speckit-tasks` → `/speckit-checklist`/`/speckit-analyze`
(opcionais) → `/speckit-implement`. A primeira fase de tarefas de qualquer feature que toque o
motor de cronograma DEVE implementar e passar os 12 casos de teste da seção 7 do PRD antes de
qualquer tarefa de tela ser iniciada (ver Princípio I).

## Governance

Esta constituição tem precedência sobre qualquer outra prática, convenção ou preferência de
implementação neste repositório, incluindo `CLAUDE.md` e decisões ad-hoc tomadas durante o
desenvolvimento. Em caso de conflito, a constituição prevalece.

Emendas exigem: (1) descrição explícita da mudança e sua motivação; (2) atualização do número de
versão conforme semver — MAJOR para remoção ou redefinição incompatível de princípio, MINOR para
novo princípio ou seção, PATCH para esclarecimentos sem mudança de regra; (3) um Sync Impact
Report no topo do arquivo descrevendo a mudança, removido antes do commit da versão emendada.

Todo plano (`/speckit-plan`) ou revisão de tarefas DEVE verificar conformidade com os princípios
acima; complexidade que viole o Princípio VII (Simplicidade e Escopo Fechado) precisa ser
justificada explicitamente ou removida.

**Version**: 1.0.0 | **Ratified**: 2026-09-30 | **Last Amended**: 2026-09-30
