# 📊 PyPortfolio - Diagnóstico e Melhorias v2.3

## 1. DIAGNÓSTICO DO NOTEBOOK ORIGINAL (v2.2)

### 1.1 Arquitetura Atual (Etapas)

| Bloco | Descrição | Status |
|-------|-----------|--------|
| 1 | Definição de ativos (hardcoded) | ⚠️ Limitado |
| 2 | Imports | ✅ OK |
| Etapa 1 | Download yfinance + conversão USD→BRL | ⚠️ Sem fallback |
| Etapa 1.5 | CDI via BCB | ✅ Bem implementado |
| Etapa 2 | Funções de métricas | ✅ OK |
| Etapa 3 | Funções objetivo | ✅ OK |
| Etapa 4 | Otimização (minimize) | ✅ OK |
| Etapa 5 | Monte Carlo | ✅ Boa amostragem |
| Etapa 6 | Visualização Plotly | ⚠️ Falta carteira atual |
| Etapa 7 | Tabelas de pesos | ⚠️ Erro de formatação |
| Etapa 8 | Heatmap covariância | ✅ OK |

### 1.2 Pontos Fortes

1. **CDI bem implementado** - Usa API do BCB (SGS série 12) com modos "atual" e "média período"
2. **Monte Carlo eficiente** - Usa amostragem Dirichlet com mix concentrado/diversificado e cálculo vetorizado
3. **Flexibilidade de calendário** - Flag `USAR_CALENDARIO_DIARIO` para lidar com cripto
4. **Conversão USD→BRL automática** - Baixa câmbio e converte ativos US/cripto

### 1.3 Fragilidades e Riscos Identificados

#### 🔴 CRÍTICOS

| # | Problema | Impacto | Solução v2.3 |
|---|----------|---------|--------------|
| 1 | **Sem suporte a Excel de retornos** | Ignora dados fornecidos pelo usuário | ✅ Implementado parser de abas |
| 2 | **Sem leitura de pesos da carteira** | Impossível comparar carteira atual | ✅ Implementado leitura da aba Resumo |
| 3 | **Retorno usa média aritmética** | Superestima retorno esperado em ~1-2% a.a. | ✅ Opção de média geométrica |
| 4 | **Sem validação de matriz PSD** | Otimização pode falhar ou dar resultados errados | ✅ Eigenvalue clipping implementado |

#### 🟠 IMPORTANTES

| # | Problema | Impacto | Solução v2.3 |
|---|----------|---------|--------------|
| 5 | **Sem MIN_OBS** | Ativos com poucos dados entram na otimização | ✅ Filtro MIN_OBS=12 |
| 6 | **Carteira atual não aparece no gráfico** | UX incompleta | ✅ Estrela (⭐) com cor exclusiva |
| 7 | **Erro de formatação na célula 204** | ValueError ao formatar pesos | ✅ Corrigido (skip coluna 'Ativo') |
| 8 | **Sem hierarquia de fontes** | Comportamento imprevisível | ✅ Excel → yfinance → DEFAULT |

#### 🟡 MENORES

| # | Problema | Impacto |
|---|----------|---------|
| 9 | Ativos hardcoded no código | Difícil manutenção |
| 10 | Sem relatório de mapeamento | Falta transparência |
| 11 | Sem piso de volatilidade (VOL_FLOOR) | Ativos RF podem dominar GMV |
| 12 | Configurações espalhadas | Difícil ajustar parâmetros |

---

## 2. ANÁLISE DO ARQUIVO EXCEL (v7.0)

### 2.1 Estrutura Identificada

```
Rendimentos_Mensais_Ativos_v7.0.xlsx
├── Resumo (metadados + pesos)
├── PETR4, ITUB4, VALE3, BBAS3 (Ações BR)
├── GOOGL, NVDA, NDAQ, META, AMZN, VOO (Ações/ETFs US)
├── QBTC11, KDIF11 (ETFs BR)
├── BTC, USDC, SOL (Cripto)
├── CRA REDE SIM, CRA MARFRIG, CRA JBS, CRA M. DIAS BRANCO
├── DEB ENGIE, DEB AEGEA SPE4, DEB AEGEA SPE1
├── 11 Fundos XP (ARX Hedge, Absolute Atenas, etc.)
├── JP Morgan Global High Yield Bond
└── COE GS Commodities
```

**Total: 36 abas (1 Resumo + 35 ativos)**

### 2.2 Formato das Abas de Ativo

```
Linha 0: Nome do ativo
Linha 1: Tipo: [Ação BR/US/ETF/Cripto/CRA/Debenture/Fundo]
Linha 2: Fonte: [Yahoo Finance/Posição Histórica XP/XP Investimentos]
Linha 4: Ano | Jan | Fev | Mar | Abr | Mai | Jun | Jul | Ago | Set | Out | Nov | Dez | Acumulado
Linhas 5+: Dados por ano (2022-2025)
```

### 2.3 Formato da Aba Resumo

```
Linha 15: Ativo | Peso Carteira Atual | Tipo | Fonte | Meses com Dados | Observações
Linhas 16+: Dados de cada ativo
```

### 2.4 Qualidade dos Dados

| Categoria | Ativos | Meses (máx 48) | Qualidade |
|-----------|--------|----------------|-----------|
| Ações BR/US | 11 | 47-48 | ✅ Excelente |
| ETFs | 2 | 18-47 | ⚠️ KDIF11 curto |
| Cripto | 3 | 47-48 | ✅ Excelente |
| CRAs | 4 | 9-47 | ⚠️ Variável |
| Debêntures | 3 | 15-20 | ⚠️ Curto |
| Fundos XP | 11 | 1-48 | ⚠️ Muito variável |
| Internacional | 1 | 45 | ✅ Bom |
| COE | 1 | 48 | ✅ Excelente |

**Problema identificado:** Valora CRI CDI Renda+ tem apenas **1 mês** de dados!

---

## 3. MELHORIAS IMPLEMENTADAS (v2.3)

### 3.1 Prioridade ALTA ✅

| # | Melhoria | Justificativa |
|---|----------|---------------|
| 1 | **Leitura de retornos do Excel** | Usar dados fornecidos pelo usuário (fonte primária) |
| 2 | **Leitura de pesos da carteira** | Permitir análise da carteira atual |
| 3 | **Hierarquia de fontes** | Comportamento determinístico e documentado |
| 4 | **Carteira atual como estrela** | Visualização clara no gráfico |
| 5 | **Média geométrica** | Estimativa de retorno mais conservadora/realista |
| 6 | **Correção de matriz PSD** | Evitar falhas na otimização |
| 7 | **Filtro MIN_OBS** | Excluir ativos com dados insuficientes |

### 3.2 Prioridade MÉDIA ✅

| # | Melhoria | Justificativa |
|---|----------|---------------|
| 8 | **Configurações centralizadas** | Facilita ajustes e manutenção |
| 9 | **Relatório de mapeamento** | Transparência sobre cobertura de pesos |
| 10 | **Fuzzy matching de nomes** | Mapear ativos com nomes diferentes |
| 11 | **Renormalização de pesos** | Tratar pesos que não somam 100% |
| 12 | **Diagnóstico da carteira** | Comparar com GMV e Max Sharpe |

### 3.3 Prioridade BAIXA (futuro)

| # | Melhoria | Status |
|---|----------|--------|
| 13 | Black-Litterman | 📋 Backlog |
| 14 | Restrições por classe | 📋 Backlog |
| 15 | Risk Parity | 📋 Backlog |
| 16 | Backtesting janela móvel | 📋 Backlog |
| 17 | Penalidade de turnover | 📋 Backlog |

---

## 4. RISCOS ESTATÍSTICOS/NUMÉRICOS

### 4.1 Anualização de Retornos

| Método | Fórmula | Viés |
|--------|---------|------|
| **Aritmético** | μ × 12 | Superestima ~1-2% a.a. |
| **Geométrico** | (∏(1+rᵢ))^(12/n) - 1 | ✅ Mais realista |

**Recomendação:** Usar geométrico (padrão v2.3)

### 4.2 Matriz de Covariância

| Problema | Causa | Solução |
|----------|-------|---------|
| Não PSD | Dados incompletos, ativos com vol ~0 | Eigenvalue clipping |
| Singularidade | Ativos perfeitamente correlacionados | VOL_FLOOR |
| Instabilidade | Poucos dados | MIN_OBS, shrinkage |

### 4.3 Calendário Cripto vs Ações

| Abordagem | Prós | Contras |
|-----------|------|---------|
| **252 dias úteis** | Consistente com mercado | Subestima vol cripto |
| **365 dias corridos** | Inclui fins de semana | Ações ficam "flat" |
| **Retornos mensais (Excel)** | ✅ Evita problema | Menor granularidade |

**Recomendação:** Usar retornos mensais do Excel (padrão v2.3)

### 4.4 Ativos de Renda Fixa

Ativos com volatilidade muito baixa (RF "suave" com marcação estável):
- Podem dominar a carteira GMV artificialmente
- Solução: VOL_FLOOR = 0.1% mensal (ajustável)

---

## 5. CHECKLIST DE VALIDAÇÃO

### Ao rodar o notebook v2.3 do zero:

- [ ] Configurações carregam sem erro
- [ ] Bibliotecas importam corretamente
- [ ] Excel é lido com sucesso (se disponível)
- [ ] Relatório mostra ativos carregados vs excluídos
- [ ] Pesos da carteira são mapeados com relatório
- [ ] CDI é obtido (ou usa fallback)
- [ ] Estatísticas são calculadas para todos os ativos
- [ ] Matriz PSD é verificada/corrigida se necessário
- [ ] Otimização GMV converge
- [ ] Otimização Max Sharpe converge
- [ ] Monte Carlo executa sem erro
- [ ] Gráfico da fronteira mostra:
  - [ ] Nuvem de portfólios colorida por Sharpe
  - [ ] GMV (losango azul)
  - [ ] Max Sharpe (triângulo verde)
  - [ ] Carteira Atual (estrela dourada) - se pesos carregados
  - [ ] Linha de RF (CDI)
  - [ ] Hover com top 10 pesos
- [ ] Heatmap de correlação renderiza
- [ ] Resumo final imprime sem erro

---

## 6. ANTES/DEPOIS - PRINCIPAIS MUDANÇAS

### 6.1 Configurações (ANTES: espalhadas)

```python
# ANTES (v2.2)
DATA_INICIO = "2020-01-01"  # Bloco 1
DATA_FIM = "2026-01-01"     # Bloco 1
# ... outras configs espalhadas pelo código
```

```python
# DEPOIS (v2.3) - Centralizado no topo
# --- Caminhos ---
EXCEL_PATH = "..."
# --- Período ---
DATA_INICIO = "2020-01-01"
DATA_FIM = "2026-01-01"
# --- Qualidade ---
MIN_OBS = 12
VOL_FLOOR = 0.001
# --- RF ---
RF_MODO = "atual"
# ... etc
```

### 6.2 Retorno Esperado (ANTES: aritmético)

```python
# ANTES (v2.2)
retornos_medios_anuais = retornos.mean() * FATOR_ANUALIZACAO
```

```python
# DEPOIS (v2.3) - Geométrico (opção)
def retorno_geometrico_anual(retornos_mensais):
    ret = retornos_mensais.dropna()
    prod = np.prod(1 + ret)
    return float(prod ** (12 / len(ret)) - 1)
```

### 6.3 Carteira Atual no Gráfico (ANTES: ausente)

```python
# DEPOIS (v2.3)
if TEM_CARTEIRA_ATUAL:
    carteiras_especiais['Carteira Atual'] = {
        'retorno': ret_atual,
        'volatilidade': vol_atual,
        ...
    }

# No gráfico:
estilos = {
    'Carteira Atual': {'color': 'gold', 'symbol': 'star', 'size': 24}
}
```

---

## 7. LIMITAÇÕES CONHECIDAS (v2.3)

1. **yfinance não implementado** - Apenas Excel como fonte primária nesta versão
2. **Sem restrições por classe** - Long-only genérico apenas
3. **Sem Black-Litterman** - Apenas Markowitz clássico
4. **Fuzzy matching simples** - Pode não mapear nomes muito diferentes
5. **Sem backtesting** - Análise apenas in-sample

---

## 8. ARQUIVOS ENTREGUES

| Arquivo | Descrição |
|---------|-----------|
| `Otimizacao_de_Portfolio_v2_3.ipynb` | Notebook revisado e organizado |
| `DIAGNOSTICO_E_MELHORIAS_v2.3.md` | Este documento |

---

*Documento gerado em: Janeiro/2026*
*Versão: 2.3*
