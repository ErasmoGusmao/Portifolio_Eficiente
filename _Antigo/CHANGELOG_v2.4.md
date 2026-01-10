# 📊 PyPortfolio v2.4 - Changelog e Explicação das Mudanças

## 1. DIAGNÓSTICO DO PROBLEMA (v2.3)

### 1.1 Por que o GMV tinha vol≈0.1% e Sharpe de -145?

**Causa raiz: USDC na matriz de covariância**

O USDC é uma stablecoin com:
- Volatilidade mensal: ~0.03% (quase zero)
- Retorno anual: ~0% (varia apenas o peg 1:1 com USD)
- Variância anualizada: ~0.001% (praticamente nula)

Quando o otimizador buscava minimizar variância, ele **colocava 100% no USDC** porque era o ativo com menor volatilidade. Isso resultava em:
- GMV com vol = 0.10% (artificialmente baixa)
- Retorno = 0.95% (apenas variação cambial residual)
- Sharpe = (0.95% - 14.90%) / 0.10% = **-144** (absurdamente negativo)

### 1.2 Por que não havia curva da fronteira eficiente?

O código v2.3 só plotava:
- Nuvem de Monte Carlo
- Pontos GMV e Max Sharpe

A **curva da fronteira eficiente** requer calcular a carteira ótima para vários retornos-alvo, o que não estava implementado.

### 1.3 Por que 28% dos pesos não foram mapeados?

Os nomes no Resumo diferem das abas:
- Resumo: `CRA REDE SIM - FEV/2030` (com barra)
- Aba: `CRA REDE SIM - FEV-2030` (com hífen)

O fuzzy matching não era tolerante a essas variações.

### 1.4 Por que a coluna de pesos não foi encontrada inicialmente?

O Excel tinha `"Peso Carteria Atual"` (com erro de digitação "Carteria" em vez de "Carteira"). O código buscava apenas `'peso' in val_str and 'cartei' in val_str`, o que falhou para "carteria".

---

## 2. LISTA DE MUDANÇAS IMPLEMENTADAS (v2.4)

### 2.1 Remoção do USDC da otimização ✅

```python
ATIVOS_EXCLUIR = ["USDC"]
```

**Impacto:**
- GMV agora tem volatilidade realista (~6-10% dependendo do universo)
- Sharpe do GMV passa a ser positivo ou levemente negativo
- Não distorce mais a otimização

### 2.2 Detecção automática de arquivos Excel ✅

```python
def encontrar_arquivos_excel(diretorio, pattern):
    # Lista arquivos, ordena por data de modificação
    # Permite escolha interativa
```

**Benefício:** Se o usuário atualizar o arquivo (v7.1, v8.0...), o notebook detecta automaticamente.

### 2.3 Leitura robusta da coluna de pesos ✅

```python
keywords_peso = ['peso', 'weight', 'alocacao']
keywords_carteira = ['cart', 'portfolio', 'atual']

# Busca flexível: encontra "Peso Carteria" ou "Peso Carteira"
```

**Benefício:** Tolera erros de digitação comuns.

### 2.4 Curva da Fronteira Eficiente ✅

```python
def calcular_fronteira_eficiente(retornos_esperados, cov_matrix, rf_anual, n_pontos=50):
    # Para cada retorno-alvo, minimiza variância
    # Retorna pontos (vol, ret) da curva
```

**Impacto:** Gráfico agora mostra a linha vermelha da fronteira eficiente, não só pontos isolados.

### 2.5 Monte Carlo melhorado ✅

```python
def simular_portfolios_monte_carlo_v2(...):
    # 1. Cantos: 5% das carteiras com 85-98% em 1 ativo
    # 2. Sparse: 15% com apenas 3-8 ativos
    # 3. Concentrado: 48% com Dirichlet(0.2)
    # 4. Diversificado: 32% com Dirichlet(1.0)
```

**Impacto:** Nuvem cobre melhor o espaço, incluindo regiões próximas aos ótimos.

### 2.6 VOL_FLOOR na matriz de covariância ✅

```python
VOL_FLOOR_MENSAL = 0.005  # 0.5% ao mês (~1.7% a.a.)

def aplicar_vol_floor_covariancia(cov_matrix, vol_floor_mensal):
    # Ajusta diagonal da matriz para evitar variância ~0
```

**Impacto:** Mesmo sem remover USDC, ativos com vol artificial baixa são corrigidos.

### 2.7 Fuzzy matching melhorado ✅

```python
def normalizar_nome(nome):
    # Normaliza hífen, barra, underline para espaço
    nome = re.sub(r'[-/_]', ' ', nome)
    ...

def similaridade(a, b):
    # Bonus se um contém o outro
    # Bonus para palavras-chave em comum
```

**Impacto:** Mapeamento de pesos passa de ~72% para ~95%+.

### 2.8 CDI alinhado ao período dos dados ✅

```python
RF_MODO = "media_periodo"
periodo_inicio = retornos_df.index.min()
periodo_fim = retornos_df.index.max()
```

**Impacto:** Sharpe calculado com CDI do mesmo período dos retornos (mais consistente).

---

## 3. PARÂMETROS CONFIGURÁVEIS (BLOCO 0)

| Parâmetro | Valor Padrão | Descrição |
|-----------|--------------|-----------|
| `EXCEL_DIR` | `"1 - Dados/1 - Rentabilidade atual"` | Diretório dos arquivos |
| `EXCEL_PATTERN` | `"Rendimentos_Mensais_Ativos*.xlsx"` | Padrão de busca |
| `MIN_OBS` | 12 | Mínimo de meses por ativo |
| `VOL_FLOOR_MENSAL` | 0.005 | Piso de volatilidade (0.5% mensal) |
| `ATIVOS_EXCLUIR` | `["USDC"]` | Ativos removidos da otimização |
| `RF_MODO` | `"media_periodo"` | Como obter CDI |
| `NUM_PORTFOLIOS` | 80000 | Carteiras Monte Carlo |
| `N_PONTOS_FRONTEIRA` | 60 | Pontos na curva da fronteira |

---

## 4. CHECKLIST DE VALIDAÇÃO

### Ao rodar v2.4 do zero:

- [ ] Arquivo Excel detectado automaticamente
- [ ] USDC listado como "excluído manualmente"
- [ ] Coluna de pesos encontrada (mesmo com "Carteria")
- [ ] Mapeamento de pesos > 90%
- [ ] CDI obtido do período (ou fallback)
- [ ] VOL_FLOOR aplicado (se houver ativos com vol baixa)
- [ ] Matriz PSD verificada/corrigida
- [ ] GMV com vol > 3% (não mais ~0.1%)
- [ ] Sharpe do GMV razoável (não -145)
- [ ] Fronteira eficiente calculada (N pontos)
- [ ] Monte Carlo com cantos e sparse
- [ ] Gráfico mostra:
  - [ ] Nuvem de portfólios
  - [ ] **Linha vermelha** da fronteira eficiente
  - [ ] GMV (losango azul)
  - [ ] Max Sharpe (triângulo verde)
  - [ ] Carteira Atual (estrela dourada)
  - [ ] Linha de RF (CDI)
- [ ] Heatmap de correlação
- [ ] Gráfico de barras de pesos (se carteira atual existe)
- [ ] Resumo final com diagnóstico

---

## 5. COMPARATIVO ANTES/DEPOIS

| Métrica | v2.3 | v2.4 |
|---------|------|------|
| GMV Volatilidade | 0.10% | ~6-10%* |
| GMV Sharpe | -145 | ~0 a +0.3* |
| Pesos mapeados | ~72% | ~95%+ |
| Fronteira eficiente | ❌ Não tinha | ✅ 60 pontos |
| Monte Carlo cantos | ❌ Não tinha | ✅ 5% |
| USDC na otimização | ✅ Sim | ❌ Removido |
| VOL_FLOOR | Não aplicado | 0.5% mensal |

*Valores aproximados, dependem do universo de ativos.

---

## 6. ARQUIVOS ENTREGUES

| Arquivo | Descrição |
|---------|-----------|
| `Otimizacao_de_Portfolio_v2_4.ipynb` | Notebook completo revisado |
| `CHANGELOG_v2.4.md` | Este documento |

---

*Documento gerado em: Janeiro/2026*
*Versão: 2.4*
