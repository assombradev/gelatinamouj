# Diagnóstico Completo — Lógica de Navegação e Telas do Quiz

Arquivo analisado: `js/index-Bqd6vyIR.js`
Componente do quiz: `PM=({onBack:e})=>{...}` (char 433.748)
Data da auditoria: 2026-04-05

---

## Estrutura geral

O componente `PM` é renderizado pela cadeia `App → LM → DM → PM`. Todo o estado do quiz e o `switch(t)` vivem dentro dele.

### Estado declarado no topo de PM

| Var | Setter | Inicial | Significado |
|---|---|---|---|
| `t` | `n` | `1` | Índice da tela atual |
| `r` | `s` | `{}` | Objeto de respostas (`r.name`, `r.fatAreas`, `r.bodyType`, `r.routine`, …) |
| `o` | `i` | `""` | Input de nome (string) |
| `a` | `l` | `75` | Peso atual (kg) |
| `u` | `c` | `165` | Altura (cm) |
| `f` | `h` | `65` | Peso desejado (kg) |
| `p` | `x` | `600` | Countdown timer da tela 31 (10 min) |

Derivações:
- `w = Math.round(t/31*100)` — barra de progresso (**31 hard-coded**)
- `m = r.name||o||"Você"` — nome exibido
- `g = (a/((u/100)²)).toFixed(1)` — IMC
- `v` — categoria de IMC

---

## 1. Lógica de avanço entre telas

**Avanço é sempre `t + 1`. Não existe nenhuma lógica de "próxima tela customizada".**

```js
// Salva resposta + auto-avança após 300ms (usado nas telas com botões de seleção)
S = b.useCallback((P,I) => {
  s(D => ({...D, [P]: I})),
  setTimeout(() => n(D => D+1), 300)
}, [])

// Continuar: avança imediatamente
E = () => n(P => P+1)
```

Botão "Voltar" usa `n(P => P-1)`.

Todos os `n(...)` encontrados dentro de PM:
- `n(D => D+1)` — S: auto-advance depois de responder
- `n(P => P+1)` — E: botão Continuar
- `n(P => P-1)` — botão Voltar
- `n(v)` — DevNav (só localhost)

**Saltos numéricos explícitos (`n(5)`, `n(15)`, etc): ZERO.**

---

## 2. Telas que pulam etapas

**Nenhuma.** A progressão é 1 → 2 → 3 → … → 31 de forma puramente linear. Não há condicionais tipo `if (x) n(15) else n(10)`.

---

## 3. Mapa completo das 31 telas

### Tela 01 — Intro do quiz
- **Título:** "Vamos começar sua jornada! 🚀"
- **Coleta:** Nenhuma
- **Dependências:** Nenhuma

### Tela 02 — Idade
- **Título:** "Qual a sua idade?"
- **Coleta:** `S("age", P)` → `r.age`
- **`r.age` lida depois?** ❌ 0 leituras em todo o arquivo
- **Veredito para remoção:** ✅ Seguro

### Tela 03 — Silhueta corporal
- **Título:** Seleção de silhueta (Médio / Plus Size / Acima do peso / Sobrepeso)
- **Coleta:** `r.bodyType` (seleção visual)
- **Dependência:** `r.bodyType` lida na própria tela 03 para marcar seleção

### Tela 04 — Áreas de gordura
- **Título:** Seleção de áreas de gordura
- **Coleta:** `T("fatAreas", ...)` → `r.fatAreas` (multi-select)
- **Dependência:** `r.fatAreas` lida na própria tela 04 para marcar checkboxes

### Tela 05 — Social proof famosas
- **Título:** "Sim, até as famosas estão usando! ⭐"
- **Coleta:** Nenhuma (tela informativa)

### Tela 06 — Nome
- **Título:** "Qual é o seu nome?"
- **Coleta:** Setter `i()` → estado `o` (nome do usuário)
- **Dependência downstream:** `m = r.name||o||"Você"` — usado em múltiplas telas para personalização

### Tela 07 — Saudação personalizada
- **Título:** "{nome}, como..."
- **Coleta:** Variável dependente da tela
- **Dependência:** Lê `m` (nome)

### Tela 08 — Satisfação corporal
- **Título:** "Você está feliz com..."
- **Coleta:** Via `S()`

### Tela 09 — Impedimentos
- **Título:** "O que te impede de emagrecer?"
- **Coleta:** Multi-select

### Tela 10 — Objetivos
- **Título:** "O que você quer conquistar?"
- **Coleta:** Multi-select

### Tela 11 — Bloco informativo
- **Título:** "Ótimo, {nome}..."
- **Coleta:** Nenhuma (bloco longo ~4,7 KB)

### Tela 12 — Peso atual
- **Título:** "Qual é seu peso atual?"
- **Coleta:** `onChange:l` → estado `a` (peso atual, kg)
- **`a` usada depois?** ⚠️ SIM, em múltiplos lugares:
  - IMC: `g = (a/((u/100)²)).toFixed(1)`
  - Tela 14: `const P = a-1` (limite máximo para peso desejado)
  - Tela 20: `parseFloat(g)` que depende de `a`
  - Tela 26: `Math.round((a-f)*.6)` e `Math.round((a-f)*1.1)`
  - Componente MM (tela 31): `weightLoss: a-f`
- **Veredito para remoção:** ⚠️ Se removida, `a` fica no default **75 kg**. Cálculos funcionam mas com valores fixos/erráticos.

### Tela 13 — Altura
- **Título:** "Qual é sua altura?"
- **Coleta:** `onChange:c` → estado `u` (altura, cm)
- **Dependência downstream:** Usada no cálculo de IMC `g`

### Tela 14 — Peso desejado
- **Título:** "Qual é seu peso desejado?"
- **Coleta:** `onChange:h` → estado `f` (peso desejado, kg)
- **Dependência downstream:** Usada em tela 26 (`a-f`) e MM (`weightLoss: a-f`)

### Tela 15 — Cálculo/animação
- **Título:** (sem título — tela de animação/loading)
- **Coleta:** Nenhuma

### Tela 16 — Gestações
- **Título:** "Quantas gestações você já teve?"
- **Coleta:** `S("pregnancies", ...)` → `r.pregnancies`
- **`r.pregnancies` lida depois?** ❌ 0 leituras
- **Veredito para remoção:** ✅ Seguro

### Tela 17 — Rotina diária
- **Título:** "Como é sua rotina diária?"
- **Coleta:** `T("routine", ...)` → `r.routine` (multi-select)
- **`r.routine` lida depois?** Apenas na própria tela 17 (para marcar checkboxes e validação `r.routine.length===0`). ❌ Não usada em nenhuma outra tela.
- **Veredito para remoção:** ✅ Seguro

### Tela 18 — Sono
- **Título:** "Quantas horas você dorme por dia?"
- **Coleta:** `S("sleep", ...)` → `r.sleep`
- **`r.sleep` lida depois?** ❌ 0 leituras
- **Veredito para remoção:** ✅ Seguro

### Tela 19 — Água
- **Título:** "Quanta água você bebe por dia?"
- **Coleta:** `S("water", ...)` → `r.water`
- **`r.water` lida depois?** ❌ 0 leituras
- **Veredito para remoção:** ✅ Seguro

### Tela 20 — Resultado da análise metabólica
- **Título:** "Resultado da sua Análise Metabólica"
- **Coleta:** Nenhuma (exibição)
- **Dependências de entrada:** Lê `g` (IMC, depende de `a` e `u`)
- **Conteúdo:** Lista problemas ("Metabolismo desacelerado", "Hormônios de saciedade desregulados"…), apresenta a Gelatina Mounjaro como solução

### Tela 21 — Como usar
- **Título:** "Como usar a Gelatina Mounjaro"
- **Coleta:** Nenhuma (instrucional)
- **Dependências:** Nenhum dado dinâmico — conteúdo estático

### Tela 22 — Compromisso
- **Título:** "Você se compromete a aplicar por pelo menos 1 semana…?"
- **Coleta:** `S("commitment", ...)` → `r.commitment`
- **`r.commitment` lida depois?** ❌ 0 leituras
- **Veredito para remoção:** ✅ Seguro

### Tela 23 — Loading (componente RM)
- **Componente:** `d.jsx(RM, {onComplete:E})`
- **Conteúdo:** "Analisando suas respostas…", "Calculando seu metabolismo basal…" (loading animado)
- **Coleta:** Nenhuma

### Tela 24 — VSL 1 (componente jM) ⚠️
- **Componente:** `d.jsx(jM, {onComplete:E})`
- **Conteúdo:** Iframe `id:"panda-player"` com VSL 1 + countdown de 70s via hook `c1(70)`
- **Coleta:** Nenhuma
- **NOTA:** Esta é a tela do VSL 1, **não** a tela 25

### Tela 25 — Corpo dos sonhos
- **Título:** "Qual o corpo dos seus sonhos?"
- **Coleta:** `S("dreamBody", P.label)` → `r.dreamBody`
- **`r.dreamBody` lida depois?** ❌ 0 leituras
- **NOTA:** Esta tela **não** é o VSL 1 — é uma pergunta de seleção
- **Veredito para remoção/reordenação:** ✅ Seguro

### Tela 26 — Faixa de perda de peso
- **Título:** "{nome}, você gostaria de perder entre X e Y kilos em poucas semanas?"
- **Coleta:** Nenhuma (exibição + botão "SIM! Quero muito começar! 🔥")
- **Dependências de entrada:** Lê `a` (peso atual), `f` (peso desejado), `m` (nome)
- **Veredito para remoção:** ✅ Seguro (não define estado)

### Tela 27 — Histórias de transformação
- **Título:** "Histórias de Transformação"
- **Coleta:** Nenhuma (carrossel: Giovanna, Sandra, Cláudia, Patrícia)
- **Veredito para remoção:** ✅ Seguro

### Tela 28 — Loading 2 (componente NM)
- **Componente:** `d.jsx(NM, {onComplete:E})`
- **Conteúdo:** Barra de progresso animada ("Analisando perfil", "Meta calculada"…)
- **Coleta:** Nenhuma

### Tela 29 — VSL 2 (componente kM)
- **Componente:** `d.jsx(kM, {onComplete:E, userName:m})`
- **Conteúdo:** Iframe `id:"panda-player-2"` com VSL 2 + countdown de 141s via hook `c1(141)`
- **Coleta:** Nenhuma

### Tela 30 — Loading 3 (componente AM)
- **Componente:** `d.jsx(AM, {onComplete:E})`
- **Conteúdo:** Animação "Protocolo de 30 dias montado", "Bônus exclusivos selecionados"
- **Coleta:** Nenhuma
- **Tratamento especial:** Header esconde botão Voltar (`t!==30`)

### Tela 31 — Oferta final (componente MM)
- **Componente:** `d.jsx(MM, {userName:m, weightLoss:a-f})`
- **Props recebidas:** nome do usuário e perda de peso calculada
- **Tratamento especial:** Header esconde progresso (`t!==31`), mostra countdown (`t===31`), esconde botão Voltar (`t!==31`)

---

## 4. Componentes externos — análise de isolamento

| Componente | Usado em | Props recebidas | Lê `t` do escopo PM? |
|---|---|---|---|
| `RM` | case 23 | `{onComplete:E}` | Não |
| `jM` (VSL 1) | case 24 | `{onComplete:E}` | Não — usa hook próprio `c1(70)` |
| `NM` | case 28 | `{onComplete:E}` | Não — tem `useState(0)` próprio |
| `kM` (VSL 2) | case 29 | `{onComplete:E, userName:m}` | Não — usa `c1(141)` próprio |
| `AM` | case 30 | `{onComplete:E}` | Não — tem animação própria |
| `MM` | case 31 | `{userName:m, weightLoss:a-f}` | Não — só consome os dois props |

**Nenhum componente externo referencia `t` por closure.** Podem ser movidos livremente entre slots.

---

## 5. Referências diretas a números de tela fora do switch

Existem **9 comparações diretas de `t` a números literais** + 1 divisor hard-coded:

| # | Trecho | Finalidade | Precisa atualizar se renumerar? |
|---|---|---|---|
| 1 | `t/31*100` | Cálculo do progresso | ✅ Mudar denominador |
| 2 | `if(t!==31)return` | Countdown só roda na última tela | ✅ |
| 3 | `t===1?"py-1":"py-2"` | Padding menor do header na tela 1 | ❌ (tela 1 não muda) |
| 4 | `(t>1\|\|e)` | Botão Voltar aparece se não for tela 1 | ❌ |
| 5 | `t!==30` | Esconde Voltar na penúltima tela | ✅ |
| 6 | `t!==31` | Esconde Voltar na última tela | ✅ |
| 7 | `t===1&&e?e():n(P=>P-1)` | Na tela 1, Voltar chama onBack | ❌ |
| 8 | `t!==31` (progresso) | Esconde barra na última tela | ✅ |
| 9 | `t===31` (countdown) | Mostra countdown na última tela | ✅ |
| 10 | `t===1?"-mt-2":"pt-2"` | Padding do conteúdo na tela 1 | ❌ |

**Total que precisa atualizar: 6 pontos** (todos referentes às telas 30/31 e ao divisor 31).

---

## 6. Variáveis de `r` (respostas) — leituras downstream

| Chave | Gravada na tela | Lida em outra tela? |
|---|---|---|
| `r.age` | 02 | ❌ 0 leituras |
| `r.bodyType` | 03 | Só na própria 03 |
| `r.fatAreas` | 04 | Só na própria 04 |
| `r.name` | 06 | ✅ Lida no topo (`m = r.name\|\|o\|\|"Você"`) |
| `r.pregnancies` | 16 | ❌ 0 leituras |
| `r.routine` | 17 | Só na própria 17 |
| `r.sleep` | 18 | ❌ 0 leituras |
| `r.water` | 19 | ❌ 0 leituras |
| `r.commitment` | 22 | ❌ 0 leituras |
| `r.dreamBody` | 25 | ❌ 0 leituras |

---

## 7. ⚠️ Correção importante: VSL 1 ≠ Tela 25

| O que o usuário disse | O que realmente é no código |
|---|---|
| "Tela 25 (VSL 1)" | Tela 25 = "Qual o corpo dos seus sonhos?" (pergunta dream body) |
| — | **VSL 1 = Tela 24** (componente `jM`, iframe `panda-player`) |
| — | **VSL 2 = Tela 29** (componente `kM`, iframe `panda-player-2`) |

---

## 8. Recomendação técnica final

### A reordenação é factível e de baixo risco estrutural, com 3 cuidados obrigatórios:

**1. Tela 12 (peso atual) é crítica:**
Se removida, `a` fica no default 75 kg e todos os cálculos de IMC, faixa de perda e "perdeu X kg" ficam fixos. Recomenda-se manter ou relocar a coleta de peso.

**2. Renumeração exige atualizar 6 pontos fora do switch:**
Todas as comparações `t===31`, `t!==31`, `t!==30` e o divisor `t/31*100` precisam ser atualizadas coordenadamente. Se renumerar sem atualizar, o header/barra/countdown aparece nos lugares errados.

**3. Confirmar tela 24 vs 25 antes de mover:**
O VSL 1 está na tela 24 (componente `jM`), não na 25.

### Telas 100% seguras para remoção (sem impacto):
2 (idade), 16 (gestações), 17 (rotina), 18 (sono), 19 (água), 22 (compromisso), 25 (dream body), 26 (faixa de perda), 27 (transformações)

### Telas de exibição seguras para remoção:
5 (social proof), 11 (info), 15 (loading), 20 (resultado IMC — se aceitável perder essa exibição), 21 (como usar)

### Telas que NÃO devem ser removidas sem mitigação:
6 (nome — alimenta `m` usado em múltiplas telas), 12 (peso — alimenta `a` usado em cálculos), 13 (altura — alimenta `u` usado no IMC), 14 (peso desejado — alimenta `f` usado na oferta)
