# Reordenação de Telas — Relatório Final

**Data:** 2026-04-05
**Arquivo modificado:** `js/index-Bqd6vyIR.js`
**Status:** ✅ CONCLUÍDO COM SUCESSO

---

## Resumo da operação

**Tamanho do arquivo:** 497.526 → 489.816 chars (−7.710 chars — os 5 cases removidos)

---

## PASSO 1 — 5 cases removidos

| Case removido | Conteúdo | Tamanho aprox. |
|---|---|---|
| `case 1` | "Vamos começar sua jornada! 🚀" (intro) | ~773 chars |
| `case 11` | "Ótimo, {nome}…" (bloco informativo) | ~4.683 chars |
| `case 25` | "Qual o corpo dos seus sonhos?" | ~439 chars |
| `case 26` | Faixa de perda de peso | ~774 chars |
| `case 27` | "Histórias de Transformação" | ~1.014 chars |

---

## PASSO 2+3+4 — Reordenação e renumeração

| Novo case | Case original | Conteúdo |
|---|---|---|
| **1** | 2 | Qual a sua idade? |
| **2** | 3 | Silhueta corporal |
| **3** | 4 | Áreas de gordura |
| **4** | 5 | Social proof famosas |
| **5** | 6 | Qual é o seu nome? |
| **6** | 7 | Saudação personalizada |
| **7** | 8 | Satisfação corporal |
| **8** | 9 | Impedimentos |
| **9** | 10 | Objetivos |
| **10** | 12 | Peso atual |
| **11** | 13 | Altura |
| **12** | 14 | Peso desejado |
| **13** | 15 | Cálculo/loading |
| **14** | 16 | Gestações |
| **15** | 17 | Rotina diária |
| **16** | 18 | Sono |
| **17** | 19 | Água |
| **18** | 23 | Loading (RM) |
| **19** | 24 | **VSL 1 (jM)** ← movido para antes da análise |
| **20** | 20 | Resultado análise metabólica |
| **21** | 21 | Como usar a Gelatina |
| **22** | 22 | Compromisso |
| **23** | 28 | Loading 2 (NM) |
| **24** | 29 | VSL 2 (kM) |
| **25** | 30 | Loading 3 (AM) |
| **26** | 31 | **Oferta final (MM)** |

---

## PASSO 5 — 6 referências hardcoded atualizadas

| # | Referência | De | Para | Finalidade | Status |
|---|---|---|---|---|---|
| 1 | Progresso | `t/31*100` | `t/26*100` | Barra de progresso | ✅ |
| 2 | Countdown | `if(t!==31)return` | `if(t!==26)return` | Countdown só na última tela | ✅ |
| 3+4 | Botão Voltar | `t!==30&&t!==31` | `t!==25&&t!==26` | Esconde Voltar nas últimas 2 telas | ✅ |
| 5 | Barra progresso | `t!==31` (progresso) | `t!==26` | Esconde barra na última tela | ✅ |
| 6 | Countdown | `t===31` (countdown) | `t===26` | Mostra countdown na última tela | ✅ |

### Referências NÃO alteradas (conforme esperado)

- `t===1?"py-1":"py-2"` — padding do header na tela 1 (tela 1 continua sendo tela 1)
- `(t>1||e)` — botão Voltar aparece se não for tela 1
- `t===1&&e?e():n(P=>P-1)` — na tela 1, Voltar chama onBack

**Status:** ✅ Intocadas

---

## PASSO 6 — Validação completa

| Verificação | Resultado | Status |
|---|---|---|
| Total de cases no switch | 26 | ✅ |
| Cases contíguos 1–26 sem lacunas | Sim | ✅ |
| Comentários `/* TELA 01 */` a `/* TELA 26 */` | Sim | ✅ |
| `default:return null}` presente | Sim | ✅ |
| 6 hardcoded refs atualizados | 6/6 | ✅ |
| Refs obsoletas (31/30) fora do switch | 0 | ✅ |
| `jM` (VSL 1) no case 19 | Confirmado | ✅ |
| `MM` (oferta) no case 26 | Confirmado | ✅ |

---

## Primeiros 3 cases (prova de integridade)

```
/* TELA 01 */case 1: ... "Qual a sua idade?"
/* TELA 02 */case 2: ... silhueta corporal (Médio, Plus Size, Acima do peso, Sobrepeso)
/* TELA 03 */case 3: ... áreas de gordura (fatAreas)
```

---

## Últimos 2 cases (prova de integridade)

```
/* TELA 25 */case 25: d.jsx(AM,{onComplete:E})  ← loading final
/* TELA 26 */case 26: d.jsx(MM,{userName:m,weightLoss:a-f})  ← oferta final
```

---

## Mudanças de estrutura

- **Quizzes anteriormente:** 31 telas
- **Quizzes atualmente:** 26 telas
- **VSL 1 (jM):** Movido de case 24 (original) para case 19 (agora) — **antes** do resultado da análise metabólica (case 20)
- **Oferta final (MM):** Permanece como última tela (case 26, era case 31)

---

## Próximos passos recomendados

1. **Teste no navegador com DevNav** (`http://localhost:8000`) para confirmar navegação entre as 26 telas
2. **Verifique o fluxo de dados** — certifique-se de que:
   - Peso (tela 10, ex-12) alimenta corretamente o IMC na tela 20
   - Nome (tela 5) aparece corretamente em todas as referências
   - VSL 1 (tela 19) roda o countdown correto antes de exibir o resultado (tela 20)
3. **Validação completa do funnel** — do início ao fim da oferta
4. **Backup:** O arquivo `index-Bqd6vyIR.v3.js` contém o estado ainda anterior a esta reordenação

---

## Backups disponíveis

| Arquivo | Conteúdo |
|---|---|
| `index-Bqd6vyIR.backup.js` | Estado original (pré-marcadores TELA) |
| `index-Bqd6vyIR.v1.js` | v1: marcadores TELA + preço + DevNav |
| `index-Bqd6vyIR.v2.js` | v2: v1 + URLs VSL atualizadas |
| `index-Bqd6vyIR.v3.js` | v3: snapshot pré-reordenação |
| `index-Bqd6vyIR.js` | **Versão atual (com reordenação)** |
