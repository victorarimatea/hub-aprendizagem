# Capítulo 1 — Engenharia Reversa de um Ecossistema Vivo

**Data:** 2026-06-05
**Sessão de origem:** Sessão de trabalho intensivo — DTD/SETIS/SES-DF
**Autores:** Victor Leonardo Arimatea Queiroz + Claude (Anthropic)
**Temas:** Falhas sistêmicas, GAPs estruturais, Defense in Depth, SEV, AAR, ADR

---

## 1. O que motivou este diagnóstico

Toda sessão de trabalho no ecossistema DTD/SETIS começa com a leitura dos
arquivos fundacionais: `CONTEXTO.md`, `sumario.md` e o `SKILL.md` da S04.
É nesse momento que inconsistências aparecem — não porque alguém procura,
mas porque dois arquivos que deveriam concordar apresentam valores diferentes.

Na sessão de 2026-06-05, o padrão se repetiu pela enésima vez: o cabeçalho
do `SKILL.md` da S04 registrava v2.1, o `CONTEXTO.md` registrava v2.3,
e o `backlog-versoes.md` já acumulava v2.4. Três arquivos, três versões,
nenhuma correta.

A correção imediata foi feita em minutos. Mas Victor nomeou o problema real:
**não é o erro em si que importa — é o padrão de repetição**. A cada nova
sessão, uma inconsistência nova. A causa raiz não estava nos erros individuais,
mas na ausência de mecanismos estruturais que impedissem sua recorrência.

A decisão foi parar de corrigir sintomas e diagnosticar o sistema.

---

## 2. O método — engenharia reversa

A engenharia reversa aplicada aqui tem um sentido específico: partir do
**estado final desejado** e mapear, para cada elo da cadeia que deveria
garantir esse estado, onde e por que a cadeia pode quebrar.

O estado final desejado do ecossistema pode ser enunciado em três propriedades:

- **P1 — Consistência:** todo arquivo que referencia outro tem informação
  idêntica à fonte de verdade
- **P2 — Rastreabilidade:** toda alteração tem registro de quem fez, quando,
  por quê e o que mudou
- **P3 — Executabilidade:** qualquer agente que leia o `CONTEXTO.md` e a S04
  consegue operar o ecossistema corretamente sem depender de memória de sessão

P1 falhava repetidamente. A questão era: **por que P1 falha se a S04 instrui
a mantê-la?** A resposta exigiu mapear todo o sistema — não apenas a instrução
que falhou.

---

## 3. O grafo de dependências — o que o ecossistema não via sobre si mesmo

O primeiro produto do diagnóstico foi a visualização do grafo completo de
dependências. Antes desse mapeamento, cada arquivo era tratado isoladamente.
O diagnóstico revelou que o ecossistema tem **14 arquivos de referência
cruzada** distribuídos em pelo menos 6 repositórios:

```
sumario.md  ←─────────────────────────────── FONTE DE VERDADE
    │
    ├──→ CONTEXTO.md
    ├──→ README.md (hub-entrada)
    ├──→ ROADMAP.md (hub-entrada)
    ├──→ docs/arquitetura.md (hub-entrada)
    └──→ ONBOARDING.md (hub-fonte)

Cada repositório ativo:
    ├──→ SKILL.md / WORKFLOW.md / README.md  (cabeçalho com versão)
    ├──→ INDICE.md  (lista de arquivos reais)
    └──→ backlog-versoes.md  (entrada para cada versão)
```

A implicação é direta: toda operação que muda a versão de qualquer componente
precisa propagar a mudança para **todos os nós dependentes**. A S04 gerencia
isso por checklists manuais executadas por um único agente. Esse é o núcleo
estrutural do problema.

---

## 4. Os 13 pontos de falha — cinco camadas

O diagnóstico identificou 13 pontos de falha distribuídos em 5 camadas do sistema.
A organização por camadas foi deliberada: revela que o problema não é pontual
— é sistêmico.

### Camada 1 — Execução da operação

**Ponto 1.1 — Instrução sem verificação embutida**
A checklist instruía "faça X". Não instruía "confirme que X foi feito".
Em engenharia de software, isso é chamado de *unchecked precondition* —
fonte clássica de erros silenciosos.

**Ponto 1.2 — Executor e verificador no mesmo agente**
O mesmo agente que definiu o plano, executou as operações e escreveu as
checklists também verifica o resultado. Não há separação entre executor
e verificador. Em qualidade e auditoria, isso é *confirmation bias* estrutural.

**Ponto 1.3 — Ausência de leitura de confirmação pós-escrita**
Após cada `PUT` na API GitHub, a S04 verificava apenas se a resposta HTTP
foi 200. Não releia o arquivo recém-escrito para confirmar que o conteúdo
estava correto.

**Ponto 1.4 — Execução sequencial sem checkpoint de estado**
Se um item falhasse silenciosamente e o agente avançasse para o próximo,
o estado do ecossistema ficava parcialmente atualizado sem detecção.

### Camada 2 — Verificação pós-execução

**Ponto 2.1 — Verificação contaminada pelo contexto de execução**
A Etapa 6 era executada imediatamente após a Etapa 5, pelo mesmo agente,
no mesmo contexto de sessão. O agente carregava em contexto o que *deveria*
ter sido feito — isso contaminava o que ele *verificava* ter sido feito.

**Ponto 2.2 — Verificações pontuais, não exaustivas**
A Verificação 1 comparava os arquivos *alterados na operação atual*.
Não fazia varredura de todos os 14 nós do grafo. Drift de sessões anteriores
ficava invisível.

**Ponto 2.3 — Verificação não detecta omissões**
A lógica era: "verifique os arquivos que você modificou". Mas drift surge
justamente em arquivos que *deveriam ter sido modificados* mas não foram.
A S04 não tinha mecanismo para detectar omissões — apenas para verificar
o que foi feito.

**Ponto 2.4 — Cobertura incompleta da Verificação 5**
A Verificação 5 cobria 5 arquivos centrais. O `ONBOARDING.md` foi adicionado
só na v2.4. Outros arquivos como o `INDICE.md` de cada repositório alterado
não tinham verificação cruzada sistemática.

### Camada 3 — Registro

**Ponto 3.1 — Backlog registra intenção, não resultado verificado**
A entrada no `backlog-versoes.md` era escrita *antes* da verificação ser
concluída. O registro descrevia o que se pretendia fazer — não o que foi
efetivamente confirmado.

**Ponto 3.2 — Sem distinção entre planejado, executado e verificado**
Não existia no backlog um campo ou estado que diferenciasse esses três
momentos. Um backlog maduro em DevOps teria esses estados distintos.

**Ponto 3.3 — Ausência de índice de criticidade**
Os Erros #001 a #013 estavam todos no mesmo nível. Sem classificação de
severidade, é impossível priorizar correções, medir tendência de melhoria
ou identificar se o ecossistema está ficando mais ou menos estável.

### Camada 4 — Inicialização de sessão

**Ponto 4.1 — Leitura de inicialização não é auditoria**
A abertura de sessão carregava o estado *declarado* do ecossistema.
Se o estado declarado já tinha drift, o agente começava com um mapa incorreto.

**Ponto 4.2 — Sem verificação ativa na abertura**
A inicialização era passiva: ler e entender. Não existia comparação ativa
entre o que os arquivos diziam e o que os repositórios realmente continham.
O drift aparecia só por acidente — quando dois arquivos que deveriam concordar
eram lidos na mesma sessão.

**Ponto 4.3 — Dependência de perspicácia do agente**
O sistema dependia de que o agente fosse suficientemente atento para notar
incoerências. Isso funcionou várias vezes. Mas perspicácia não é mecanismo —
é qualidade. Mecanismos são confiáveis; qualidades são variáveis.

### Camada 5 — A própria S04

**Ponto 5.1 — A S04 não se auto-audita**
A S04 crescia em tamanho a cada erro incorporado. Mas não tinha mecanismo
para verificar se as instruções adicionadas eram respeitadas nas operações
subsequentes.

**Ponto 5.2 — Instrução dispersa em arquivo longo**
Com mais de 600 linhas, instruções críticas ficavam enterradas em checklists.
Em contexto longo, atenção distribuída significa que instruções no meio de
listas longas têm probabilidade menor de serem executadas com precisão.

**Ponto 5.3 — Sem mecanismo de detecção de instrução ignorada**
Se o agente executou 9 dos 10 itens de uma checklist e omitiu o décimo,
não havia nada na S04 que detectasse essa omissão.

---

## 5. Os 4 GAPs estruturais

Consolidando os 13 pontos em 4 gaps de nível superior — o nível onde as
intervenções precisam atuar:

**GAP 1 — Ausência de verificação independente**
Toda verificação era feita pelo mesmo agente que executou, no mesmo contexto.
Não há separação executor/verificador em nenhum nível do sistema.

**GAP 2 — Verificação cobre o que foi feito, não o que deveria ter sido feito**
A Etapa 6 verificava arquivos modificados. Não detectava omissões — arquivos
que deveriam ter sido modificados mas não foram.

**GAP 3 — Registros sem estados de ciclo de vida**
O backlog não distingue planejado / executado / verificado. Falhas parciais
de execução não ficam registradas.

**GAP 4 — Sem métrica de saúde do ecossistema**
Sem índice de criticidade, sem série histórica de frequência, sem indicador
de tendência. Impossível saber se o ecossistema está melhorando.

---

## 6. O que o mercado já sabia — benchmarks que validam o diagnóstico

Ao nomear os problemas, ficou evidente que o ecossistema havia chegado
intuitivamente a soluções que o mercado já conhece — e que estava reinventando
a roda em alguns pontos onde soluções robustas existem há décadas.

**Defense in Depth** (Segurança da Informação / SRE): o princípio de que
nenhum controle crítico depende de camada única. A S04 era camada única.
A solução natural é adicionar camadas independentes — exatamente o que
as Execuções 1, 2 e o futuro W05 fazem.

**AAR — After Action Review** (US Army / DevOps): processo estruturado
de revisão pós-atividade. O ecossistema fazia AAR implícito nos backlogs.
O `hub-aprendizagem` eleva isso para AAR explícito com narrativa e lição
sedimentada.

**ADR — Architecture Decision Records** (Engenharia de Software): registrar
não apenas o que foi decidido, mas por que e quais alternativas foram
consideradas. O ecossistema registrava o quê — mas não sistematicamente
o raciocínio.

**Escala SEV / ITIL**: classificação de incidentes por severidade. Adotada
na Execução 1 para os Erros #001–#013. O padrão SEV foi escolhido por ser
o mais alinhado com o contexto de governança de serviços públicos de TI.

**Checked Operations** (Programação Defensiva): distinção entre instruir
e confirmar. Toda instrução crítica de escrita deve ter verificação
embutida — convertida em prática obrigatória na Execução 2.

**Separation of Concerns na Verificação** (Auditoria / ISO 9001): quem
executa não deve ser o único a verificar. Princípio clássico de controle
de qualidade que o ecossistema violava estruturalmente.

A referência completa de cada benchmark está em
[`referencias/benchmarks.md`](./referencias/benchmarks.md).

---

## 7. As respostas aplicadas — o que foi feito

Na mesma sessão em que o diagnóstico foi produzido, três intervenções
foram executadas:

**Execução 1 — Escala de severidade SEV1–SEV4**
Classificação retroativa dos 13 erros históricos. Distribuição resultante:
SEV1=1, SEV2=5, SEV3=6, SEV4=1. Padrão obrigatório adotado para todos os
erros futuros. Responde ao GAP 4.

**Execução 2 — Verificações embutidas obrigatórias (Nível B Estrutural)**
14 blocos `CONFIRMAR` inseridos nas checklists de todas as operações OP-X.
Regra de dois registros obrigatórios para erros novos. Padrão de campos
obrigatórios na seção de erros. Responde parcialmente ao GAP 1 e ao GAP 3.

**Execução 3 — Criação do `hub-aprendizagem`**
Este repositório. Responde ao GAP 4 em sua dimensão qualitativa —
não apenas métricas, mas narrativa e aprendizado sedimentado.

**Pendente — Execução 4 — W05 Workflow de Auditoria de Consistência**
Processo independente que varre o grafo completo de dependências contra
o `sumario.md` como fonte de verdade. A solução estrutural completa
para o GAP 1 e o GAP 2.

---

## 8. Lições aprendidas permanentes

**Lição 1 — Instrução e verificação são ações distintas e igualmente obrigatórias**
"Faça X" e "confirme que X foi feito" são instruções diferentes. Um sistema
que tem a primeira mas não a segunda está operando com precondições não
verificadas — fonte clássica de erros silenciosos.

**Lição 2 — Perspicácia não é mecanismo**
O ecossistema funcionou durante semanas dependendo da atenção do agente
para detectar drift. Isso é frágil por design. Mecanismos sistemáticos
não dependem de qualidades variáveis.

**Lição 3 — Chegar intuitivamente ao certo não é o mesmo que aprender com quem errou antes**
O ecossistema chegou a várias práticas corretas por raciocínio próprio.
Isso demonstra capacidade de análise. Mas o caminho mais eficiente é
combinar essa capacidade com o conhecimento acumulado — evitar erros
que outros já cometeram e documentaram.

**Lição 4 — Crescer em tamanho não é crescer em confiabilidade**
A S04 passou de 200 para 600+ linhas ao longo de 13 erros corrigidos.
Cada erro virou uma instrução nova. Mas instruções sem verificação embutida
não aumentam a confiabilidade — apenas aumentam a complexidade.
Confiabilidade vem de checked operations, não de instruções mais longas.

**Lição 5 — O documento mais importante não é o que instrui — é o que verifica**
A S04 era o documento central do ecossistema por instruir todas as operações.
Mas o documento que poderia ter evitado todos os drifts seria um processo
de auditoria independente. O que faltava não era mais instrução — era
verificação com olhos diferentes.

---

*Este capítulo foi produzido na sessão de 2026-06-05 e representa o*
*diagnóstico mais completo já realizado sobre a arquitetura de controle*
*de qualidade do ecossistema DTD/SETIS.*
