# Referências Externas — Benchmarks do Ecossistema DTD/SETIS

**Versão:** v1.0 — 2026-06-05
**Mantido por:** victorarimatea

> Consolidação das referências externas que contextualizam e validam
> as práticas desenvolvidas no ecossistema DTD/SETIS. Cada entrada
> indica onde o ecossistema chegou de forma intuitiva ou deliberada,
> e o que o mercado/literatura já sabe sobre aquele problema.

---

## 1. After Action Review (AAR)

**Origem:** US Army — adotado amplamente em DevOps e organizações de aprendizagem
**O que é:** Processo estruturado de revisão pós-atividade com quatro perguntas:
o que planejamos, o que aconteceu, por que divergiu, o que aprendemos.

**Relação com o ecossistema:** Os backlogs de versões do ecossistema fazem AAR
implicitamente — registram o que mudou e por quê. O `hub-aprendizagem` eleva
esse padrão para AAR explícito: narrativa, diagnóstico e lição sedimentada.

**Referências:**
- US Army Field Manual FM 6-01.1 — Knowledge Management Operations
- Darling, M. et al. "Learning in the Thick of It" — Harvard Business Review, 2005

---

## 2. Architecture Decision Records (ADR)

**Origem:** Michael Nygard (2011) — padrão amplamente adotado em engenharia de software
**O que é:** Documentos curtos que registram cada decisão arquitetural significativa
com contexto, opções consideradas, decisão tomada e consequências previstas.

**Relação com o ecossistema:** O ecossistema registra *o que* foi decidido
(backlogs) mas não sistematicamente *por que* e *quais alternativas foram
consideradas*. O `hub-aprendizagem` incorpora a lógica ADR nos capítulos —
cada escolha de design registrada com raciocínio completo.

**Formato padrão ADR:**
```
# ADR-NNN: [Título da decisão]
Status: [Proposto | Aceito | Depreciado | Substituído]
Contexto: [Situação que motivou a decisão]
Decisão: [O que foi decidido]
Consequências: [O que muda, o que fica mais difícil, o que fica mais fácil]
```

**Referências:**
- Nygard, M. "Documenting Architecture Decisions" — cognitect.com, 2011
- Thoughtworks Technology Radar — ADRs como prática recomendada

---

## 3. Defense in Depth (Defesa em Profundidade)

**Origem:** Segurança da informação — adaptado para DevOps e engenharia de confiabilidade
**O que é:** Princípio que nenhum controle crítico depende de uma única camada
de proteção. Falhas são esperadas; o sistema é projetado para que a falha de
uma camada não comprometa o todo.

**Relação com o ecossistema:** Identificado como vulnerabilidade estrutural
na sessão de 2026-06-05: a S04 operava como camada única de controle —
executor, verificador e registrador no mesmo agente. As Execuções 1 e 2
introduziram a segunda camada (verificações embutidas CONFIRMAR). O W05
(Workflow de Auditoria) será a terceira camada independente.

**Referências:**
- NIST SP 800-53 — Security and Privacy Controls
- Google SRE Book — "Defense in Depth" (Capítulo sobre segurança de produção)

---

## 4. DORA Metrics

**Origem:** DevOps Research and Assessment (DORA) — Google
**O que são:** Quatro métricas de desempenho de entrega de software:
Deployment Frequency, Lead Time for Changes, Change Failure Rate,
Time to Restore Service.

**Relação com o ecossistema:** O ecossistema não mede performance de entrega
ainda, mas a lógica DORA é relevante para o futuro: frequência de operações,
tempo entre identificação de drift e correção, taxa de erros por operação,
tempo médio para restaurar consistência após falha. A escala SEV1–SEV4
adotada na Execução 1 é o primeiro passo para medir Change Failure Rate.

**Referências:**
- Forsgren, N. et al. "Accelerate: The Science of Lean Software and DevOps" — 2018
- DORA State of DevOps Report — dora.dev

---

## 5. Escala de Severidade SEV (ITIL / ISO 20000)

**Origem:** ITIL (Information Technology Infrastructure Library) — padrão global
de gerenciamento de serviços de TI; ISO/IEC 20000
**O que é:** Classificação de incidentes e erros por impacto e urgência,
tipicamente em 4 níveis (SEV1 crítico → SEV4 baixo).

**Relação com o ecossistema:** Adotado na Execução 1 da sessão de 2026-06-05
para classificar retroativamente os Erros #001–#013 da S04. Escolhido em
detrimento de P0–P3 (Google) e C1–C4 (notação própria) por ser o padrão
mais alinhado com o contexto de governança documental e serviços públicos de TI.

**Distribuição nos erros do ecossistema (2026-06-05):**
- SEV1 (Crítico): 1 erro — instrução operacional que causava falha silenciosa
- SEV2 (Alto): 5 erros — dados incorretos em arquivos de referência central
- SEV3 (Médio): 6 erros — inconsistências detectáveis apenas por auditoria
- SEV4 (Baixo): 1 erro — perda de valor sem falha de consistência

**Referências:**
- ITIL 4 Foundation — AXELOS
- ISO/IEC 20000-1:2018 — IT Service Management

---

## 6. Checked Operations (Operações Verificadas)

**Origem:** Engenharia de software — padrão de programação defensiva
**O que é:** Distinção entre *unchecked preconditions* (instrua e confie)
e *checked operations* (instrua, execute e confirme). Em sistemas críticos,
toda operação que pode falhar silenciosamente deve ter verificação embutida.

**Relação com o ecossistema:** Identificado como Ponto de Falha 1.1 na
engenharia reversa de 2026-06-05. A checklist OP-B instruía "incremente o
cabeçalho" mas não instruía "confirme que foi incrementado". A Execução 2
converteu todas as instruções críticas de escrita em checked operations,
adicionando blocos CONFIRMAR obrigatórios após cada PUT na API GitHub.

**Referências:**
- McConnell, S. "Code Complete" — Microsoft Press, 2004 (Capítulo Defensive Programming)
- Nygard, M. "Release It!" — Pragmatic Bookshelf, 2018

---

## 7. Separation of Concerns na Verificação (executor ≠ verificador)

**Origem:** Auditoria, controle de qualidade, engenharia de segurança
**O que é:** Princípio que quem executa uma tarefa não deve ser o único a
verificá-la. O viés de confirmação — tendência de confirmar o que se espera
encontrar — é estruturalmente inevitável quando executor e verificador são
o mesmo agente.

**Relação com o ecossistema:** Identificado como GAP 1 na engenharia reversa
de 2026-06-05. A S04 opera com executor e verificador no mesmo agente de IA,
no mesmo contexto de sessão. As checked operations (Execução 2) mitigam
parcialmente esse risco. O W05 (Workflow de Auditoria) será a solução
estrutural completa: verificação genuinamente independente, com lógica
própria e escopo exaustivo.

**Referências:**
- ISACA — COBIT 2019 (segregação de funções em controles de TI)
- ISO 9001:2015 — Sistemas de Gestão da Qualidade (princípio de verificação independente)

---

*Atualizar sempre que uma nova referência externa for identificada e validada.*
*Cada entrada deve ter: origem, definição, relação com o ecossistema e referências.*
---

## Cache Coherence / Read-After-Write Consistency

**Origem:** Sistemas distribuídos / literatura de banco de dados e CDN
**Referência no ecossistema:** cap-04 (API vs raw.githubusercontent.com)

**O que é:**
Cache coherence descreve a garantia de que todas as réplicas de um dado
compartilhado convergem para o mesmo valor. Read-after-write consistency é
uma propriedade específica: após uma escrita bem-sucedida, qualquer leitura
subsequente reflete aquela escrita imediatamente.

**Por que importa aqui:**
CDNs (como a que serve `raw.githubusercontent.com`) operam sob *consistência
eventual* — a réplica converge, mas não instantaneamente. Isso significa que
uma leitura via raw logo após um PUT pode devolver a versão anterior do arquivo.
A GitHub Contents API (`api.github.com`) não passa por esse cache, oferecendo
read-after-write consistency para operações autenticadas.

**Princípio derivado:**
Canais de acesso ao mesmo dado fazem trade-offs diferentes. O canal mais
conveniente costuma relaxar alguma garantia. A escolha correta depende da
garantia que a operação exige — não da conveniência do canal.
