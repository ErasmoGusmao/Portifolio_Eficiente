# 📊 PyPortfolio v2.5 - Changelog e Checklist

## RESUMO DAS MUDANÇAS

### PARTE A - Correções Críticas ✅

#### 1. Mapeamento de pesos ANTES do filtro MIN_OBS ✅

**Problema:** O código anterior carregava dados e aplicava `MIN_OBS` DENTRO da função `carregar_dados_excel()`, e só depois fazia o fuzzy match. Quando uma aba era excluída por `MIN_OBS`, o peso era erroneamente remapeado para OUTRO ativo similar.

**Solução:** Reestruturei o fluxo em etapas separadas:

```
ANTES (v2.4):
1. carregar_dados_excel() → já aplica MIN_OBS → retorna apenas abas filtradas
2. Faz fuzzy match usando apenas as abas que passaram
3. Bug: se aba correta foi excluída, mapeia para outra errada

DEPOIS (v2.5):
1. carregar_metadados_excel() → lista TODAS as abas (sem filtrar)
2. ler_pesos_resumo_robusto() → lê pesos do Resumo
3. filtrar_por_min_obs() → separa abas_ok e abas_excluidas_min_obs
4. mapear_pesos_para_abas() → fuzzy match usando TODAS as abas
5. carregar_retornos_das_abas() → carrega só as que passaram
```

**Nova função `mapear_pesos_para_abas()`:**
- Recebe: `pesos_resumo`, `abas_candidatas`, `abas_excluidas_min_obs`, thresholds
- Faz fuzzy match contra TODAS as abas (incluindo excluídas)
- Classifica em 4 status:
  - `OK`: score >= 0.90, aba no universo
  - `REVISAR`: 0.75 <= score < 0.90, aba no universo
  - `NAO_MAPEADO`: score < 0.75
  - `MAPEADO_EXCLUIDO_MIN_OBS`: mapeou corretamente, mas aba foi excluída

**DataFrame de mapeamento:**
```
| ativo_resumo | peso | aba_mapeada | score | status | motivo |
```

**Relatório numérico:**
- `peso_total_resumo`
- `peso_mapeado_ok`
- `peso_mapeado_revisar`
- `peso_mapeado_no_universo`
- `peso_mapeado_excluido_min_obs`
- `peso_nao_mapeado`

---

#### 2. Tabelas estruturadas após gráfico de pesos ✅

**Tabela 1 - Métricas das carteiras:**
```
| Carteira       | Retorno Anual (%) | Volatilidade Anual (%) | Sharpe |
|----------------|-------------------|------------------------|--------|
| GMV            | X.XX              | X.XX                   | X.XXX  |
| Max Sharpe     | X.XX              | X.XX                   | X.XXX  |
| Carteira Atual | X.XX              | X.XX                   | X.XXX  |
```

**Tabela 2 - Pesos por ativo (comparativo):**
```
| Ativo | GMV (%) | Max Sharpe (%) | Carteira Atual (%) |
```
- Ordenada por peso da Carteira Atual (ou Max Sharpe se não houver)
- Mostra Top N (configurável) + tabela completa disponível

---

### PARTE B - Melhorias Importantes ✅

#### 3. Checagem de sucesso do solver + fallback ✅

**Nova classe `ResultadoOtimizacao`:**
```python
class ResultadoOtimizacao:
    pesos: np.ndarray
    sucesso: bool
    mensagem: str
    metodo: str  # "GMV_SLSQP" ou "GMV_FALLBACK_EW"
```

**Comportamento:**
- Se `minimize()` retornar `success=False`:
  - Registra mensagem de erro
  - Aplica fallback (equal-weight)
  - Marca `metodo` como `*_FALLBACK_EW`
- No resumo final, mostra status de cada otimização

---

#### 4. MIN_OVERLAP efetivo na covariância ✅

**Nova função `calcular_matriz_overlap()`:**
- Calcula matriz NxN com número de meses em comum entre cada par

**Nova função `aplicar_min_overlap_covariancia()`:**
- Para pares com overlap < MIN_OVERLAP:
  - Zera covariância (cov[i,j] = cov[j,i] = 0)
- Reporta quantos pares foram afetados

**Integração:**
- `calcular_estatisticas_portfolio()` agora recebe `min_overlap` como parâmetro
- Retorna `pares_zerados` para relatório

---

## CHECKLIST DE VALIDAÇÃO

### Bug de mapeamento corrigido:

- [ ] Execute o notebook com um arquivo Excel que tenha ativos excluídos por MIN_OBS
- [ ] Verifique se esses ativos aparecem com status `MAPEADO_EXCLUIDO_MIN_OBS`
- [ ] Confirme que o peso NÃO foi remapeado para outro ativo
- [ ] O DataFrame de mapeamento deve mostrar:
  - `aba_mapeada` = nome correto da aba (mesmo excluída)
  - `status` = MAPEADO_EXCLUIDO_MIN_OBS
  - `motivo` = "Aba 'X' excluída por MIN_OBS"

### Tabelas estruturadas:

- [ ] Tabela de métricas aparece após os gráficos
- [ ] Contém 3 linhas (GMV, Max Sharpe, Carteira Atual) ou 2 se não houver atual
- [ ] Valores formatados: 2 casas para %, 3 casas para Sharpe
- [ ] Tabela de pesos aparece logo após
- [ ] Ordenada corretamente (Atual ou Max Sharpe)
- [ ] Top N mostrado + menção à tabela completa

### Robustez das otimizações:

- [ ] Se uma otimização falhar, mensagem de warning aparece
- [ ] Fallback para equal-weight é aplicado
- [ ] Resumo final mostra status (✅ ou ⚠️ fallback) para cada carteira

### MIN_OVERLAP aplicado:

- [ ] Se houver pares com overlap < MIN_OVERLAP:
  - [ ] Mensagem indica quantos pares foram zerados
  - [ ] Covariâncias desses pares são 0 na matriz final

---

## LISTA NUMERADA DE MUDANÇAS

1. **Nova função `carregar_metadados_excel()`** - carrega lista de abas e contagem de meses SEM filtrar
2. **Nova função `filtrar_por_min_obs()`** - separa abas em OK vs excluídas
3. **Nova função `mapear_pesos_para_abas()`** - fuzzy match usando TODAS as abas, classifica em 4 status
4. **Nova função `gerar_relatorio_mapeamento()`** - resumo numérico do mapeamento
5. **Nova função `carregar_retornos_das_abas()`** - carrega retornos apenas das abas selecionadas
6. **Nova classe `ResultadoOtimizacao`** - armazena pesos + metadados de sucesso
7. **Funções `otimizar_*` modificadas** - retornam `ResultadoOtimizacao`, usam fallback se falhar
8. **Nova função `calcular_matriz_overlap()`** - calcula meses em comum entre pares
9. **Nova função `aplicar_min_overlap_covariancia()`** - zera covariâncias com overlap baixo
10. **Função `calcular_estatisticas_portfolio()` modificada** - recebe `min_overlap`, retorna `pares_zerados`
11. **BLOCO 2 reestruturado** - 7 etapas sequenciais com logging claro
12. **Novo BLOCO 7** - Tabelas estruturadas (métricas + pesos)
13. **Resumo final** - inclui status de sucesso das otimizações
14. **Roadmap** - lista priorizada de melhorias futuras

---

## ROADMAP PRIORIZADO

### Alta Prioridade (v2.6-v2.7)

| # | Melhoria | Por que é importante | Complexidade |
|---|----------|----------------------|--------------|
| 1 | **Conversão USD→BRL (FX)** | Retornos de ativos US estão em USD; precisam ser convertidos para BRL para comparação correta | Média |
| 2 | **Fonte yfinance + cache** | Quando Excel não disponível, baixar de yfinance; cache evita chamadas repetidas | Média |
| 3 | **rapidfuzz para matching** | SequenceMatcher é OK, mas rapidfuzz (token_set_ratio) é mais robusto | Baixa |

### Média Prioridade (v2.8-v3.0)

| # | Melhoria | Por que é importante | Complexidade |
|---|----------|----------------------|--------------|
| 4 | **Restrições por classe** | Limites por classe (cripto max 10%, RF min 20%, etc.) | Média |
| 5 | **Backtesting out-of-sample** | Validar se a otimização "funciona" fora da amostra | Alta |
| 6 | **Ledoit-Wolf shrinkage** | Covariância mais estável com poucos dados | Média |

### Baixa Prioridade (v3.1+)

| # | Melhoria | Complexidade |
|---|----------|--------------|
| 7 | **Black-Litterman** | Alta |
| 8 | **Export CSV/HTML** | Baixa |

---

## ARQUIVOS ENTREGUES

| Arquivo | Descrição |
|---------|-----------|
| `Otimizacao_de_Portfolio_v2_5.ipynb` | Notebook com correções críticas |
| `CHANGELOG_v2.5.md` | Este documento |

---

*Versão: 2.5*
*Data: Janeiro/2026*
