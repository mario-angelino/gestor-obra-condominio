# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Estado atual do repositório

Ainda não há código de aplicação — o repositório tem o PRD (`.docs/PRD.md`) e o Spec Kit instalado (`.specify/`, `.claude/skills/speckit-*`), mas nenhuma spec, plano, tarefa ou implementação foi gerada ainda. Não existe `package.json`, projeto Next.js, motor, testes, nem configuração de lint/build. Não invente comandos de build/lint/test até que o projeto seja de fato inicializado; verifique primeiro se `package.json` e as pastas descritas abaixo já existem.

O PRD (`.docs/PRD.md`) é a fonte da verdade para escopo, entidades, regras de negócio e critérios de aceite. Leia-o antes de planejar ou implementar qualquer coisa — não repita seu conteúdo de memória, releia o arquivo, pois ele pode ser atualizado.

O repositório é um projeto git local (inicializado ao instalar o Spec Kit), sem remoto configurado.

## Fluxo de bootstrap via Spec Kit

O Spec Kit está instalado nesta pasta (CLI `specify`, via `uvx --from git+https://github.com/github/spec-kit.git specify ...`; scripts em PowerShell, `.specify/scripts/powershell/`). A integração instalada expõe os workflows como **skills do Claude Code em `.claude/skills/speckit-*`**, invocados como `/speckit-nome` (sem ponto) — não `/speckit.nome`, como a seção 9 do PRD escreve (nomenclatura da versão antiga do Spec Kit).

Ordem recomendada, adaptando a seção 9 do PRD aos nomes atuais:

1. `/speckit-constitution` — princípios do projeto (usar o texto da seção 9 do PRD); preenche `.specify/memory/constitution.md`, hoje só com placeholders.
2. `/speckit-specify` — especificação funcional (seções 1–7 do PRD), sem falar de stack.
3. `/speckit-clarify` (opcional, mas recomendado antes do plano) — responder usando a seção 10 do PRD ("Decisões tomadas").
4. `/speckit-plan` — stack definida na seção 8/9 do PRD.
5. `/speckit-tasks` — pedir que a primeira fase seja o motor (`/lib/engine`) com os testes da seção 7, antes de qualquer tela.
6. `/speckit-checklist` (opcional) — validar completude/clareza dos requisitos após o plano.
7. `/speckit-analyze` (opcional) — checar consistência entre spec, plano e tarefas.
8. `/speckit-implement` — fase por fase, revisando os testes do motor ao final da primeira fase.
9. `/speckit-converge` (quando já houver código) — avalia o que falta e adiciona como tarefas.

Cada `/speckit-specify` cria uma feature nova (branch + pasta sob `.specify/` ou `specs/`, conforme o template) — se já existir uma feature em andamento, continue nela em vez de recomeçar.

## Arquitetura alvo (definida no PRD, ainda não implementada)

Quando o código existir, a estrutura esperada é:

- **Motor de cronograma** (`/lib/engine`): módulo TypeScript puro, sem dependência de UI ou banco de dados. Recebe o estado completo (condomínio, parâmetros, vigências de taxa, entradas pontuais, obras, propostas, contratos, reajustes) e devolve a projeção financeira mês a mês. É a peça mais crítica do sistema — testado com Vitest, cobertura ≥ 90%, e deve rodar em <200ms para 30 obras. Os 12 casos de teste da seção 7 do PRD são a especificação executável desse motor e devem ser implementados como testes automatizados antes de qualquer tela.
- **Front-end**: Next.js (App Router) + TypeScript + Tailwind + shadcn/ui. Todo ajuste no dashboard chama o motor no cliente e recalcula na hora, sem round-trip ao servidor.
- **Banco/arquivos**: Supabase (Postgres + Storage para orçamentos). Sem autenticação no MVP. RLS por `condominio_id` é trabalho futuro, não implementar agora.
- **Gráficos**: Recharts para saldo acumulado; Gantt mensal (barras sólidas para obras contratadas, tracejadas para futuras) implementado à mão em SVG/CSS.
- **Drag-and-drop**: dnd-kit, para reordenar prioridade de obras.
- **Exportação**: PDF via página de impressão (CSS print) + CSV mês a mês.
- **Deploy**: Vercel.

## Regras de domínio que a implementação precisa respeitar

Estas regras vêm da seção 5 do PRD e são fáceis de violar acidentalmente porque cruzam várias entidades — vale reler a seção 5 completa antes de mexer no motor:

- **Dinheiro é sempre inteiro em centavos.** Arredondamento só acontece na exibição. Meses são strings `AAAA-MM`.
- **O motor é uma função pura.** Mesmo estado de entrada → mesma saída. Nenhuma leitura de banco ou side effect dentro dele.
- **Ordem de consumo do caixa:** primeiro as parcelas de obras já **contratadas** (na data travada), depois as **futuras** por prioridade sobre o que sobra. Mudar prioridade nunca move contratadas.
- **Valor de obra futura depende do estágio de maturidade** (ideia, escopo definido, com propostas, proposta escolhida, contratada), cada um com sua contingência padrão (30%, 20%, 10%, 5%, 0%) e sua fonte de valor (estimativa da obra vs. proposta escolhida vs. proposta válida de maior valor).
- **Propostas vencidas são descartadas** do cálculo e não podem ser escolhidas.
- **Reajuste só existe se informado explicitamente** (% + mês de vigência), e vale daquele mês em diante — inclusive para parcelas já contratadas.
- **Viabilidade das contratadas:** se o saldo projetado (só receitas + parcelas contratadas) fica negativo em algum mês, o motor marca a(s) contratada(s) afetada(s) como "Renegociação necessária" e sugere reescalonamento — mas **nunca altera o contrato sozinho**; o conselheiro precisa aceitar ou digitar o novo cronograma, gerando nova versão.
- **Encaixe das futuras** respeita mês mínimo de maturação e, em modo estrito (antecipação desligada), não pode começar antes do início da obra de prioridade imediatamente superior (pode empatar no mesmo mês se o caixa comportar). Com antecipação ligada, essa restrição de ordem cai, mas uma obra de menor prioridade só antecipa se não atrasar nenhuma superior.
- **Horizonte é dinâmico** (o motor avança meses até encaixar tudo), com trava técnica em 240 meses — além disso a obra fica "Não cabe com a receita atual".
- **Faixa de término:** o motor roda duas vezes por alteração — cenário base (entradas esperadas pelo % de realização) e otimista (entradas esperadas a 100%) — para mostrar uma faixa, não um número único.
- **Nada é apagado, tudo é versionado.** Contratos geram nova versão a cada renegociação; anexos usam exclusão lógica; assembleias congelam snapshots imutáveis do plano.
- **Todo modelo de dados carrega `condominio_id`** desde o MVP, preparando para multi-tenant futuro — mesmo que o MVP opere com um único condomínio e sem autenticação/RLS.

## Idioma e formato

O produto e sua interface são em português do Brasil. Valores monetários em formato brasileiro (R$), meses exibidos como "mar/2027" mas armazenados como `AAAA-MM`. Mantenha esse idioma/formato em telas, mensagens e nomes de domínio (obra, proposta, vigência, assembleia etc.); código (nomes de variáveis/funções) pode seguir inglês ou português conforme o que for estabelecido quando o projeto for de fato inicializado — verifique o padrão já em uso antes de misturar.
