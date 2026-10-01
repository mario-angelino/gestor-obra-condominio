# PRD — App de Obras do Condomínio (MVP)

30/09/2026 · Mario Angelino

## 1. Visão e objetivo

O app mostra, em tempo real, **quanto tempo leva para executar todas as obras aprovadas dentro da taxa extra aprovada em assembleia**, respeitando a prioridade definida pelos condôminos.

**Problema:** a assembleia aprova obras e taxa extra sem enxergar o fluxo de caixa. Ninguém sabe quando cada obra cabe, quanto custa adiar, nem o efeito de reduzir ou aumentar a taxa.

**Objetivo do MVP:** um conselheiro de um prédio (60 unidades) cadastra obras, propostas, taxa extra e entradas pontuais, e o sistema gera o cronograma físico-financeiro recalculado a cada alteração. Validação: uso real em pelo menos uma assembleia.

**App vivo, revisitado a cada assembleia:** o planejamento não é um relatório fechado. Em cada nova assembleia, valores, prioridades, aprovações e propostas são alterados ao vivo no dashboard, e o cronograma de execução mostra na hora o reflexo de cada mudança. Cada assembleia congela uma versão do plano aprovado, para comparar com a anterior.

**Obra nasce com escopo:** ao incluir uma obra (ex.: "Reforma das ardósias da área comum"), o conselheiro descreve o escopo desejado, que serve de base padronizada para as empresas orçarem. Os orçamentos recebidos são cadastrados na obra, com o arquivo original anexado e visualizável na própria assembleia.

**Visão de produto (pós-MVP):** módulo multi-condomínio vendido a administradoras. O MVP deve nascer com modelo de dados preparado para isso (campo `condominio_id` em tudo), sem implementar multi-tenant ainda.

## 2. Escopo

| Dentro do MVP | Fora do MVP (2.0+) |
| --- | --- |
| Dashboard ao vivo: editar valores, prioridades e aprovações e ver o cronograma recalcular | Fundo de reserva com valor definido e impacto no fluxo |
| Modo assembleia (tela cheia para projetor) | Multi-condomínio / multi-tenant para administradoras |
| Cadastro do condomínio (nº de unidades, mês inicial) | Integração com ERP (Superlógica etc.) e boletos |
| Taxa extra com vigências (linha do tempo de valores) | Portal público para moradores |
| Inadimplência estimada (%) | Registro formal de votação e ata por obra |
| Entradas pontuais (confirmadas / esperadas) | Realizado × planejado com lançamentos reais |
| Obras com escopo desejado, prioridade, status e maturidade | Notificações (WhatsApp/e-mail) |
| Propostas por obra com escopo, nº de parcelas × valor da parcela e upload do orçamento (PDF/imagem) | Autenticação, perfis e permissões |
| Visualizador de orçamentos dentro do app | Envio do escopo direto às empresas pelo app |
| Contratação e renegociação de cronograma | Cenário pessimista |
| Motor de cronograma em tempo real | |
| Cenários (comparar até 3 lado a lado) | |
| Versão do plano congelada por assembleia | |
| Exportar cronograma (PDF/CSV) | |

## 3. Usuários e histórias

**Usuário único no MVP:** conselheiro/síndico (administrador). Apresenta o cronograma na assembleia pela tela ou PDF.

1. Como conselheiro, cadastro o condomínio com número de unidades e mês de início do planejamento.
2. Cadastro a taxa extra por vigência (valor por unidade + mês de início) para simular aumento ou redução a partir de um ponto de corte.
3. Informo a inadimplência estimada (%) para a receita ser líquida.
4. Lanço entradas pontuais (ex.: inadimplência recuperada) com data, valor e grau de certeza.
5. Incluo uma obra com título e **escopo desejado** (o que precisa ser feito, materiais, áreas, exigências), para usar como base igual para todas as empresas orçarem.
6. Defino prioridade, estágio de maturidade, estimativa, nº de parcelas estimado e prazo mínimo de maturação da obra.
7. Cadastro cada orçamento recebido com empresa, escopo ofertado, prazo, nº de parcelas, valor da parcela, validade e **upload do arquivo original** (PDF ou imagem).
8. Na assembleia, abro os orçamentos de uma obra lado a lado e visualizo os arquivos sem sair do app.
9. Escolho a proposta vencedora e marco a obra como contratada, travando o cronograma de pagamento.
10. No dashboard, altero taxa, prioridades, aprovações e valores e vejo o cronograma recalcular na hora.
11. Quando uma mudança de taxa ou receita inviabiliza parcelas contratadas, vejo o alerta de renegociação, o déficit e uma proposta de reescalonamento.
12. Registro a renegociação aceita, com novo cronograma de parcelas.
13. Salvo e comparo cenários (ex.: taxa R$ 500 × R$ 600).
14. Ao fim de cada assembleia, congelo a versão aprovada do plano e comparo com a versão da assembleia anterior.
15. Exporto o cronograma para registro e divulgação.

## 4. Entidades e dados

Valores monetários em centavos (inteiro), meses como `AAAA-MM`. Toda entidade carrega `condominio_id` e `cenario_id` quando aplicável.

| Entidade | Campos principais | Observações |
| --- | --- | --- |
| Condomínio | nome, nº de unidades, mês inicial | Um registro no MVP; saldo inicial sempre R$ 0 |
| Parâmetros | inadimplência %, contingência por estágio (%), permitir antecipação (sim/não) | Por cenário |
| Vigência da taxa | valor por unidade, mês de início | Linha do tempo; a vigência seguinte encerra a anterior |
| Entrada pontual | descrição, mês, valor, certeza (confirmada / esperada), % de realização | Acordo parcelado = várias entradas |
| Obra | título, escopo desejado (texto formatado), prioridade (ordem), estágio, estimativa total, nº de parcelas estimado, prazo mínimo de maturação (meses), data de aprovação, status | Estágios: ideia, escopo definido, com propostas, proposta escolhida, contratada, em execução, concluída |
| Proposta | obra, empresa, CNPJ, escopo ofertado, nº de parcelas, valor da parcela, prazo de execução (meses), validade, escolhida (sim/não) | Valor total = nº de parcelas × valor da parcela; vencida é descartada do cálculo e não pode ser escolhida |
| Reajuste | obra, %, mês de vigência, motivo | Só existe se informado; vale do mês de vigência em diante |
| Anexo | proposta ou obra, nome do arquivo, tipo (PDF/JPG/PNG), tamanho, caminho no storage, data de envio | Até 20 MB por arquivo; vários por proposta; exclusão lógica |
| Contrato | obra, proposta, mês de início, cronograma de parcelas (mês, valor), versão | Gerado das parcelas da proposta; nova versão a cada renegociação; histórico preservado |
| Pagamento realizado | contrato, mês, valor | Marca parcela como paga; meses passados viram realizado |
| Cenário | nome, base (sim/não), cópia dos parâmetros, vigências, entradas e ordem de prioridade | Obras, propostas e contratos são compartilhados; cenário varia só premissas e ordem das futuras |
| Assembleia | data, descrição, snapshot completo do plano aprovado (estado + resultado do motor) | Imutável depois de congelada; permite comparar versões entre assembleias |

## 5. Regras do motor de cronograma

O motor é uma função pura: recebe o estado (condomínio, parâmetros, vigências, entradas, obras, propostas, contratos, reajustes) e devolve a projeção mês a mês. Roda a cada alteração, em menos de 200 ms para 30 obras.

### 5.1 Receita mensal

```
Receita(m) = Unidades × Taxa(m) × (1 − Inad) + EntradasConfirmadas(m) + EntradasEsperadas(m) × %Realização
```

- Taxa(m) é o valor da vigência ativa no mês m. Alterar a taxa cria nova vigência a partir de um mês de corte; meses anteriores ao corte não mudam.
- No cenário base, entradas esperadas entram pelo % de realização informado; no otimista, a 100%.

### 5.2 Ordem de consumo do caixa

1. **Contratadas** — parcelas do contrato vigente, na data travada.
2. **Futuras** — alocadas por prioridade sobre o que sobra.

Mudar a prioridade nunca move contratadas. Mudar taxa ou receitas pode torná-las inviáveis (5.4).

### 5.3 Valor projetado de uma obra futura

```
Parcela = ParcelaBase × (1 + Contingência[estágio]) × (1 + Reajuste[informado])
```

| Estágio | Nº de parcelas e parcela base | Contingência padrão |
| --- | --- | --- |
| Ideia | nº estimado da obra; estimativa ÷ nº | 30% |
| Escopo definido | nº estimado da obra; estimativa ÷ nº | 20% |
| Com propostas | proposta válida de maior valor total (conservador) | 10% |
| Proposta escolhida | parcelas da proposta escolhida | 5% |
| Contratada | parcelas do contrato | 0% (firme) |

- A primeira parcela cai no mês de início s; as demais, uma por mês.
- Forma de pagamento vem sempre da proposta (nº de parcelas × valor da parcela). Sem proposta, usa o nº de parcelas estimado da obra.
- Proposta vencida é descartada: não entra no cálculo nem pode ser escolhida.
- **Reajuste** só existe quando informado: % e mês de vigência, por obra. Aplica-se às parcelas daquele mês em diante, inclusive de contratadas. Sem reajuste informado, os valores ficam fixos.
- Contingência é editável nos parâmetros.

### 5.4 Viabilidade das contratadas

1. Projeta o saldo acumulado só com receitas e parcelas contratadas, do mês atual até a última parcela contratada.
2. Se o saldo fica negativo em algum mês, as contratadas com parcelas a partir desse mês ganham o status **Renegociação necessária**, a começar pela de menor prioridade.
3. Para cada uma, o motor mostra o déficit mensal e acumulado e sugere um reescalonamento: mesmo saldo devedor, parcelas limitadas à capacidade mensal disponível e novo mês final.
4. O sistema nunca altera o contrato sozinho. O conselheiro aceita a sugestão ou digita o cronograma negociado, gerando nova versão do contrato.
5. Enquanto houver renegociação pendente, as futuras são projetadas sobre o reescalonamento sugerido e ficam marcadas "condicionadas".

### 5.5 Encaixe das futuras

Para cada obra futura, em ordem de prioridade:

1. Mês mínimo = mês atual + meses de maturação restantes.
2. Em modo estrito (antecipação desligada), o mês mínimo não pode ser anterior ao início da obra de prioridade imediatamente superior. Pode ser o mesmo mês, se o caixa comportar a soma das parcelas.
3. Testa cada mês s a partir do mínimo: monta as parcelas da obra e verifica se o saldo acumulado projetado fica ≥ 0 em **todos** os meses até a última parcela já alocada, somando os compromissos existentes.
4. O primeiro s viável é o início. Fim = s + prazo de execução − 1 (obra de 2 meses iniciada em jul termina em ago).
5. Horizonte dinâmico: o motor avança quantos meses forem necessários até encaixar todas as obras. Trava técnica de 240 meses; se atingida (ex.: taxa zerada), a obra fica **Não cabe com a receita atual**.

Com antecipação ligada, a regra 2 é ignorada. Como as obras de maior prioridade são alocadas antes, uma de menor prioridade só é antecipada se não atrasar nenhuma superior.

### 5.6 Faixa de término

O motor roda duas vezes:

- **Base:** parâmetros informados, com entradas esperadas pelo % de realização.
- **Otimista:** entradas esperadas a 100%.

A tela mostra o término base e, para cada futura, a faixa até o otimista. Cenário pessimista fica para versão futura.

### 5.7 Saídas do motor

- Por mês: receita, entradas, pagamentos por obra, saldo final e alertas.
- Por obra: status, início, fim, valor projetado, faixa e alertas.
- Resumo: mês de conclusão da última obra, total comprometido, total projetado e menor saldo do período.

## 6. Telas e requisitos funcionais

| Tela | Conteúdo | Requisitos-chave |
| --- | --- | --- |
| Dashboard (tela principal) | Resumo no topo (término, menor saldo, alertas); painel lateral editável com taxa, inadimplência, lista de obras (prioridade, aprovação, valor); Gantt mensal com barras sólidas (contratadas) e tracejadas (futuras) e faixa base–otimista; gráfico de saldo acumulado; tabela mês a mês | Toda edição feita no próprio dashboard recalcula o cronograma na hora; arrastar obra muda a prioridade; alternar aprovada/não aprovada; desfazer última alteração |
| Modo assembleia | Dashboard em tela cheia, fonte ampliada, sem menus | Mesmas edições ao vivo; botão "Congelar versão da assembleia" |
| Obras | Lista ordenada por prioridade com status, estágio, valor projetado, início e fim | Filtros por status; badge de alerta |
| Detalhe da obra | Escopo desejado, propostas lado a lado (valor, prazo, parcelas, escopo ofertado), anexos, contrato e histórico de versões | Upload por arrastar arquivo; lançar reajuste (% e mês); visualizador de PDF/imagem embutido; escolher proposta; botão Contratar; tela de renegociação com sugestão do motor |
| Receitas | Vigências da taxa (linha do tempo) e entradas pontuais | CRUD; validar sobreposição de vigências |
| Parâmetros | Unidades, inadimplência, contingência por estágio, antecipação on/off | Valores padrão pré-preenchidos |
| Cenários | Lista e comparação lado a lado de até 3 | Duplicar cenário; comparar término, menor saldo e início de cada obra |
| Assembleias | Histórico de versões congeladas | Comparar duas versões: obras incluídas/removidas, prioridade, taxa, término |
| Exportar | PDF de uma página (resumo + Gantt + tabela) e CSV mês a mês | Layout legível em projetor e impressão |

**Regras de interface:**

- Valores em R$ no formato brasileiro; meses como "mar/2027".
- Mudança de taxa sempre pede o mês de corte (padrão: próximo mês).
- Alertas em vermelho: déficit, renegociação necessária, não cabe com a receita atual, proposta vencida.
- Desktop primeiro; leitura em celular aceitável.

## 7. Critérios de aceite (caso de teste de referência)

Este caso vira teste automatizado do motor. Todos os resultados abaixo foram calculados à mão.

**Premissas:** 60 unidades, taxa R$ 500, inadimplência 10% (receita R$ 27.000/mês), saldo inicial R$ 0, mês atual jan/2027, sem reajuste informado, antecipação desligada, cenário base.

| Obra | Situação | Parcelas | Total | Maturação | Prazo |
| --- | --- | --- | --- | --- | --- |
| A | Contratada, prioridade 1 | 6 × R$ 10.000 (jan–jun) | R$ 60.000 | — | 6 meses |
| B | Proposta escolhida, prioridade 2 | 1 × R$ 100.000 + 5% = R$ 105.000 | R$ 105.000 | 2 meses | 2 meses |
| C | Ideia, prioridade 3 | estimativa R$ 40.000 em 4 parcelas + 30% = 4 × R$ 13.000 | R$ 52.000 | 3 meses | 4 meses |

| # | Ação | Resultado esperado |
| --- | --- | --- |
| 1 | Rodar o motor | Saldo com A: jan 17.000, fev 34.000, mar 51.000, abr 68.000, mai 85.000, jun 102.000 |
| 2 | Encaixe de B | Início jul/2027 (em jun o saldo ficaria em −3.000); saldo de jul = 24.000 |
| 3 | Encaixe de C | Início jul/2027, mesmo mês de B (modo estrito permite); saldo de jul = 11.000, ago = 25.000 |
| 4 | Ligar antecipação | C continua em jul/2027: qualquer início entre abr e jun deixa o saldo negativo em jul, atrasando B |
| 5 | Entrada pontual confirmada de R$ 10.000 em mar/2027 | B antecipa para jun/2027 (saldo de jun = 7.000) |
| 6 | Reordenar prioridades (C acima de B) | A não se move; só B e C são recalculadas |
| 7 | Taxa R$ 100 a partir de fev/2027 (receita R$ 5.400) | A fica "Renegociação necessária"; saldo −1.400 em mai e −6.000 em jun/2027; motor sugere reescalonamento |
| 8 | Taxa R$ 300 a partir de fev/2027 (receita R$ 16.200) | A continua viável; B passa para out/2027 (saldo de out = 7.800) |
| 9 | Reajuste de 10% em C a partir de set/2027 (sobre o caso 3) | Parcelas de C de set em diante = R$ 14.300; saldo de set = 37.700 |
| 10 | Marcar uma proposta como vencida | Sai do cálculo e não pode ser escolhida |
| 11 | Estado com 30 obras | Recalcular em menos de 200 ms |
| 12 | Aceitar renegociação | Nova versão do contrato A; versão anterior visível no histórico |

## 8. Requisitos não funcionais e stack sugerida

**Não funcionais**

- Motor isolado como módulo puro em TypeScript, sem dependência de UI ou banco, com cobertura de testes ≥ 90%.
- Recálculo no cliente, em menos de 200 ms, sem ida ao servidor a cada ajuste.
- Persistência automática; nenhum dado perdido ao fechar a aba.
- Sem autenticação no MVP (acesso por URL, não divulgar o link); login e RLS entram em versão futura.
- Toda alteração em contrato gera nova versão; nada é apagado (exclusão lógica).
- Dinheiro sempre em centavos inteiros; arredondamento só na exibição.

**Stack sugerida**

| Camada | Escolha |
| --- | --- |
| Front-end | Next.js (App Router) + TypeScript + Tailwind + shadcn/ui |
| Motor | Pacote `/lib/engine` em TypeScript puro, testado com Vitest |
| Banco e arquivos | Supabase (Postgres + Storage para orçamentos); sem autenticação no MVP; RLS futura por `condominio_id` |
| Gráficos | Recharts (saldo); Gantt próprio em SVG/CSS |
| Arrastar e soltar | dnd-kit |
| Exportação | PDF via página de impressão (CSS print) + CSV |
| Deploy | Vercel |

## 9. Roteiro Spec Kit

Este PRD já está salvo como `.docs/PRD.md` no repositório. Rode os comandos abaixo no Claude Code, em ordem.

1. **Inicializar:** `specify init --here --ai claude` (na pasta do projeto).
2. **Constitution:**

```text
/speckit.constitution Projeto: app de planejamento financeiro de obras de condomínio. Princípios: (1) o motor de cronograma é uma função pura em TypeScript, isolada de UI e banco, e todo comportamento dele nasce de um teste; (2) dinheiro sempre em centavos inteiros; (3) contratos nunca são sobrescritos nem apagados, só versionados; (4) o sistema alerta, nunca altera contrato sozinho; (5) toda tabela tem condominio_id para futuro multi-tenant, mas o MVP é de um condomínio só; (6) MVP sem autenticação, com código preparado para receber Auth e RLS depois; (7) simplicidade: nada fora do escopo de .docs/PRD.md seção 2; (8) interface em português do Brasil, formato monetário brasileiro.
```

3. **Specify** (só o quê e o porquê, sem stack):

```text
/speckit.specify Construir o MVP descrito em .docs/PRD.md, seções 1 a 7. Um conselheiro de condomínio cadastra obras aprovadas em assembleia (com escopo desejado), propostas de empresas (com parcelas e arquivo do orçamento), a taxa extra com vigências, a inadimplência estimada e entradas pontuais. O sistema projeta mês a mês, num dashboard editável ao vivo, quando cada obra cabe no caixa, respeitando a prioridade da assembleia, travando obras contratadas, deixando as futuras flutuarem com contingência por maturidade, prazo mínimo de maturação e reajuste quando informado, alertando quando uma mudança de taxa inviabiliza contratos e sugerindo reescalonamento. Cada assembleia congela uma versão do plano. Os critérios de aceite são os 12 casos da seção 7.
```

4. **Clarify:** `/speckit.clarify` — responda usando a seção 10 deste PRD.
5. **Plan:**

```text
/speckit.plan Stack: Next.js App Router + TypeScript + Tailwind + shadcn/ui; Supabase como banco (Postgres) e Storage para arquivos de orçamento, sem autenticação no MVP; motor em /lib/engine em TypeScript puro com Vitest; Recharts para o saldo; Gantt próprio em SVG; dnd-kit para reordenar; visualizador de PDF/imagem embutido; exportação por CSS print e CSV; deploy na Vercel. Recalcular o motor no cliente a cada alteração.
```

6. **Tasks:** `/speckit.tasks` — peça que a primeira fase seja o motor com os testes da seção 7, antes de qualquer tela.
7. **Analyze:** `/speckit.analyze` para checar consistência entre spec, plano e tarefas.
8. **Implement:** `/speckit.implement`, fase por fase, revisando os testes do motor ao fim da primeira.

## 10. Decisões tomadas

Decisões fechadas em 30/09/2026 e já refletidas no PRD; use como resposta no `/speckit.clarify`.

- [x] Forma de pagamento: nº de parcelas e valor da parcela informados no cadastro da proposta; base da projeção.
- [x] Reajuste: só quando informado expressamente, valendo daquele mês em diante.
- [x] Cenário pessimista: fora do MVP.
- [x] Obra pode começar no mesmo mês da superior, se o caixa comportar as parcelas.
- [x] Pagamento por medição: não existe; tudo é informado como parcelas na proposta.
- [x] Horizonte: dinâmico, quantos meses forem necessários para encaixar todas as obras.
- [x] Proposta vencida: descartada.
- [x] Saldo inicial: zero.
- [x] Banco: Supabase, sem autenticação no MVP.
- [x] Obra sem proposta: estimativa ÷ nº de parcelas estimado, informado na obra.
- [x] Obra com várias propostas e nenhuma escolhida: usa a proposta válida de maior valor (conservador).
