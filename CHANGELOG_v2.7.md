# 📊 PyPortfolio v2.7 - Changelog

## Metadados

| Campo | Valor |
|-------|-------|
| **Versão** | v2.7 |
| **Data** | Janeiro/2026 |
| **Versão anterior** | v2.6 |

---

## Resumo das Mudanças

### PARTE A — Implementações Críticas (P0) ✅

#### 1. Conversão USD→BRL ✅

**Problema resolvido:** Notebook assumia todos os retornos em BRL. Ativos USD tinham retornos incorretos.

**Solução implementada:**

1. **Estrutura de metadados por ativo** (`MetadadosAtivo`):
   - `nome`, `moeda` (BRL/USD), `fonte` (excel/yfinance/manual), `ticker`, `convertido`

2. **Lista configurável `ATIVOS_USD`** com ativos em dólar

3. **Download de série de câmbio** via yfinance (`USDBRL=X`)

4. **Fórmula de conversão:**
   ```
   r_BRL = (1 + r_USD) * (1 + r_FX) - 1
   ```

5. **Bloco de validação** mostrando 5 linhas de cálculo

**Novos parâmetros:**
- `ATIVOS_USD`: Lista de ativos em USD
- `FX_TICKER`: Ticker do câmbio (default: "USDBRL=X")
- `CACHE_DIR`: Pasta de cache (default: "cache/")
- `CACHE_DIAS_VALIDADE`: Dias de validade do cache (default: 7)

---

#### 2. Política de Peso Fora do Universo ✅

**Novo parâmetro `POLITICA_PESO_FORA`:**
- `"reportar"` (padrão): Apenas reporta, não inclui na otimização
- `"renormalizar"`: Renormaliza para 100% com aviso
- `"caixa_cdi"`: Trata como Caixa CDI fora do Σ

**Auditoria detalhada:**
- peso_ok, peso_revisar, peso_excluido_min_obs, peso_excluido_manual, peso_nao_mapeado

**Novo status: `EXCLUIDO_MANUAL`** para ativos na lista ATIVOS_EXCLUIR

---

#### 3. Monte Carlo com Bounds ✅

- Função `simular_monte_carlo()` recebe `peso_min` e `peso_max`
- Amostras clippadas e renormalizadas
- Preparado para limites por classe

---

### PARTE B — Implementações P1 ✅

#### 4. Integração yfinance + Cache ✅

**Perguntas interativas:**
- "Carregar dados do Excel? (s/n)"
- "Se faltar ativo, tentar yfinance? (s/n)"

**Hierarquia:** Excel → yfinance → DEFAULT_UNIVERSE

**Cache:** Dados salvos em parquet com validade configurável

---

## Lista de Mudanças

1. Classe `MetadadosAtivo` para estrutura de metadados
2. Parâmetro `ATIVOS_USD` com lista de ativos em dólar
3. Parâmetro `FX_TICKER` para ticker do câmbio
4. Parâmetro `POLITICA_PESO_FORA` com 3 opções
5. Parâmetros `CACHE_DIR` e `CACHE_DIAS_VALIDADE`
6. Parâmetro `DEFAULT_UNIVERSE` para fallback
7. Funções de cache: `cache_valido()`, `salvar_cache()`, `carregar_cache()`
8. Funções de câmbio: `baixar_cambio_usdbrl()`, `converter_retorno_usd_para_brl()`
9. Função `validar_conversao_fx()` para auditoria
10. Status de mapeamento `EXCLUIDO_MANUAL`
11. Monte Carlo com respeito aos bounds
12. Perguntas interativas sobre Excel e yfinance

---

## Checklist de Validação

### Conversão USD→BRL
- [ ] Ativos USD identificados (lista ATIVOS_USD)
- [ ] Câmbio baixado do yfinance
- [ ] Fórmula r_BRL aplicada corretamente
- [ ] Validação exibe 5 linhas de cálculo

### Cobertura de Pesos
- [ ] Auditoria mostra todos os status
- [ ] Política funciona (reportar/renormalizar/caixa_cdi)
- [ ] Nunca remapeia para ativo errado

### yfinance
- [ ] Cache funciona (salva e carrega)
- [ ] Fallback quando Excel indisponível

### Rodar do Zero
- [ ] Com Excel + pesos ✓
- [ ] Com Excel sem pesos ✓
- [ ] Sem Excel (yfinance) ✓
- [ ] Ativos USD + BRL juntos ✓

---

## Arquivos

| Arquivo | Descrição |
|---------|-----------|
| `Otimizacao_de_Portfolio_v2_7.ipynb` | Notebook principal |
| `CHANGELOG_v2.7.md` | Este documento |

---

*Versão: 2.7 | Data: Janeiro/2026*
