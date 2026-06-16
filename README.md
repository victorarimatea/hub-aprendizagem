# hub-aprendizagem

**Tipo:** Documento (D03)
**Versão:** v1.4 — 2026-06-16
**Repositório:** https://github.com/victorarimatea/hub-aprendizagem
**Mantenedor:** victorarimatea
**Visibilidade:** Público

> Memória intelectual do ecossistema DTD/SETIS.
> Boas práticas, benchmarks e lições aprendidas na construção
> de governança documental aumentada por IA.

---

## O que é este repositório

Este repositório não é um manual de instrução — não é consumido por agentes
de IA durante a execução de tarefas. É um **tratado vivo de aprendizagem**:
o lugar onde o conhecimento adquirido na prática é sedimentado, contextualizado
com benchmarks de mercado e transformado em referência própria do ecossistema.

A diferença em relação aos backlogs e changelogs do ecossistema:

| Instrumento | Responde à pergunta |
|---|---|
| `backlog-versoes.md` | O que foi alterado e por quê? |
| `CHANGELOG.md` | O que foi entregue e quando? |
| `hub-aprendizagem` | O que aprendemos, o que isso significa, e como se compara com o que o mundo já sabe? |

---

## Para quem é este repositório

- **Para Victor** — consolidar o conhecimento construído sessão a sessão,
  sem deixá-lo dissipado em históricos de conversa
- **Para futuros colaboradores** — entender não apenas o que o ecossistema
  faz, mas por que foi construído assim e quais escolhas foram feitas
- **Para a comunidade** — demonstrar que é possível construir infraestrutura
  de governança documental de alta qualidade com recursos escassos,
  método rigoroso e aprendizado contínuo

---

## Estrutura

```
hub-aprendizagem/
├── README.md                              — este arquivo
├── INDICE.md                              — mapa completo de conteúdo
├── backlog-versoes.md                     — histórico de versões
└── capitulos/
    ├── cap-01-engenharia-reversa.md       — Diagnóstico de falhas sistêmicas
    └── referencias/
        └── benchmarks.md                  — Referências externas consolidadas
```

---

## Capítulos publicados

| # | Título | Data | Temas |
|---|---|---|---|
| 01 | [Engenharia Reversa de um Ecossistema Vivo](./capitulos/cap-01-engenharia-reversa.md) | 2026-06-05 | Falhas sistêmicas, GAPs estruturais, Defense in Depth, SEV, AAR, ADR |
| 02 | [A Lição da Separação: Executor e Auditor](./capitulos/cap-02-separacao-executor-auditor.md) | 2026-06-05 | Viés de confirmação, Defense in Depth, W05, separação executor/auditor |

---

## Princípio editorial

Cada capítulo segue a estrutura de um **relato de experiência contextualizado**:

1. O problema que motivou o aprendizado
2. O que foi observado e diagnosticado
3. O que o mercado/literatura já sabe sobre isso
4. O que fizemos — e por quê escolhemos assim
5. O que ficou como aprendizado permanente

O objetivo não é descrever o estado atual do ecossistema — para isso existem
o `CONTEXTO.md` e o `sumario.md`. O objetivo é registrar o **raciocínio por
trás das escolhas** e o **conhecimento ganho pelo caminho**.

---

*Mantido por Victor Leonardo Arimatea Queiroz*
*Diretor de Transformação Digital — DTD/SETIS/SES-DF*
