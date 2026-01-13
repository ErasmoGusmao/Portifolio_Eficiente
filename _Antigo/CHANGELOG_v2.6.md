# 📊 PyPortfolio v2.6 - Changelog

## Metadados

| Campo | Valor |
|-------|-------|
| **Versão** | v2.6 |
| **Data** | Janeiro/2026 |
| **Versão anterior** | v2.5 |

---

## Resumo das Mudanças

### PARTE A — Correções Críticas (P0) ✅

#### 1. Seletor de Arquivos Corrigido ✅

**Diagnóstico (antes):**
- A função `selecionar_arquivo_excel()` retornava imediatamente se o arquivo padrão existisse
- Não fazia varredura completa da pasta
- Com 5+ arquivos, só mostrava 1

**Mudanças realizadas:**
1. Nova função `varrer_arquivos_excel()` que:
   - Sempre varre o diretório completo
   - Aceita `.xlsx` e `.XLSX` (case insensitive)
   - Ignora arquivos temporários do Excel (`~$...`)
   - Ignora arquivos ocultos (`.nome`)
   - Retorna metadados: nome, data modificação, tamanho
   
2. Nova função `selecionar_arquivo_excel_interativo()` que:
   - SEMPRE lista todos os arquivos encontrados
   - Se arquivo padrão existe, pergunta: "Usar este arquivo? (s/n) [s]"
   - Se "n", permite escolha por índice
   - Mostra quantidade de arquivos, nomes e datas
   - Mensagem clara se nenhum arquivo encontrado

**Checklist de validação:**
- [ ] Com 5+ arquivos na pasta, TODOS são listados
- [ ] Arquivo padrão existente: pergunta se quer usar
- [ ] Responder "n" permite escolher outro
- [ ] Arquivos `~$temp.xlsx` são ignorados
- [ ] Pasta vazia: mensagem clara de erro

---

#### 2. Documentação de Parâmetros ✅

**Diagnóstico (antes):**
- Parâmetros definidos no código sem explicação detalhada
- Difícil entender quando ajustar cada um

**Mudanças realizadas:**
1. Nova seção Markdown "Parâmetros e Configuração (Documentação)"
2. Tabelas com colunas:
   - Parâmetro
   - Valor Padrão
   - O que controla
   - Quando ajustar
   - Impacto / Trade-off
3. Cobertura completa:
   - Qualidade de dados: `MIN_OBS`, `MIN_OVERLAP`, `VOL_FLOOR`
   - Mapeamento: `THRESHOLD_OK`, `THRESHOLD_REVISAR`, `ATIVOS_EXCLUIR`
   - Monte Carlo: `NUM_PORTFOLIOS`, `MC_ALPHA_*`, `MC_FRAC_*`
   - Fronteira: `N_PONTOS_FRONTEIRA`
   - RF: `RF_MODO`, `RF_MANUAL`, `RF_FALLBACK`
   - Otimização: `PESO_MIN_ATIVO`, `PESO_MAX_ATIVO`, `USAR_MEDIA_GEOMETRICA`

4. Bloco "Metadados da Versão" no topo:
```python
VERSAO = "v2.6"
DATA_VERSAO = "Janeiro/2026"
MUDANCAS_VERSAO = [...]
```

**Checklist de validação:**
- [ ] Seção "Parâmetros e Configuração" existe após o header
- [ ] Todas as tabelas renderizam corretamente
- [ ] Cada parâmetro tem descrição completa

---

### PARTE B — Melhorias de Alto Impacto (P1) ✅

#### 3. Pergunta Explícita sobre Carregar Pesos ✅

**Diagnóstico (antes):**
- Se aba Resumo existia, carregava pesos automaticamente
- Sem controle do usuário

**Mudanças realizadas:**
1. Verifica se aba Resumo existe e tem pesos
2. Mostra preview: quantidade de ativos, soma de pesos
3. Pergunta: "Deseja carregar os pesos da carteira atual? (s/n) [s]"
4. Se "n": segue sem carteira atual (sem estrela, sem tabelas dela)
5. Se "s": carrega e faz mapeamento completo

**Checklist de validação:**
- [ ] Pergunta aparece quando Resumo existe
- [ ] "n" segue sem carteira atual
- [ ] "s" carrega e exibe mapeamento

---

#### 4. Auditorias Reforçadas ✅

**Diagnóstico (antes):**
- Relatório de mapeamento existia mas não alertava problemas graves
- Cobertura baixa passava despercebida

**Mudanças realizadas:**
1. Nova função `exibir_auditoria_mapeamento()` que mostra:
   - Soma de pesos no Resumo
   - % mapeado OK no universo
   - % mapeado REVISAR
   - % total no universo
   - % excluído por MIN_OBS
   - % não mapeado
   - **COBERTURA EFETIVA**
   - Se renormalizou (sim/não) e qual base

2. Parâmetro `ALERTA_COBERTURA_MINIMA = 0.60`

3. Se cobertura < 60%:
   - Bloco de ALERTA com `!!!`
   - Explica possíveis causas
   - Recomenda revisar mapeamento ou ajustar MIN_OBS

**Checklist de validação:**
- [ ] Seção "AUDITORIA DE MAPEAMENTO" aparece
- [ ] Todos os % são mostrados
- [ ] Com cobertura < 60%, ALERTA aparece
- [ ] Indica se renormalizou

---

### PARTE C — Roadmap (P2) ✅

Seção final do notebook com 8 itens priorizados:

1. **Conversão USD→BRL (FX)** - Alta prioridade
2. **Fonte yfinance + cache** - Alta prioridade
3. **Fuzzy matching com rapidfuzz** - Média prioridade
4. **Restrições por classe** - Média prioridade
5. **Backtesting out-of-sample** - Média prioridade
6. **Black-Litterman** - Baixa prioridade
7. **Covariância robusta (Ledoit-Wolf)** - Média prioridade
8. **Melhorias de UX/export** - Baixa prioridade

Cada item inclui: por que importa, complexidade, dependências.

---

## Lista Numerada de Mudanças

1. Nova função `varrer_arquivos_excel()` com varredura completa
2. Nova função `selecionar_arquivo_excel_interativo()` com escolha sempre oferecida
3. Seção "Parâmetros e Configuração" com tabelas detalhadas
4. Bloco "Metadados da Versão" no início do código
5. Novo parâmetro `ARQUIVO_PADRAO` separado de `EXCEL_PATTERN`
6. Novo parâmetro `ALERTA_COBERTURA_MINIMA`
7. ETAPA 3 adicionada: pergunta sobre carregar pesos
8. Nova função `exibir_auditoria_mapeamento()` com alertas
9. Seção "Roadmap de Próximas Versões" no final
10. Seção "Checklist de Validação" no final

---

## Notas de Migração (v2.5 → v2.6)

### Parâmetros novos:
```python
ARQUIVO_PADRAO = "Rendimentos_Mensais_Ativos_v7.0.xlsx"  # Novo
ALERTA_COBERTURA_MINIMA = 0.60  # Novo
```

### Parâmetros alterados:
```python
EXCEL_PATTERN = "*.xlsx"  # Antes: "Rendimentos_Mensais_Ativos*.xlsx"
```

### Comportamento alterado:
- Seleção de arquivo agora é sempre interativa (lista todos)
- Carregamento de pesos agora pede confirmação
- Auditoria de mapeamento mais detalhada

---

## Checklist Completo de Validação

### Rodar do Zero - Cenário 1: Pasta com 5+ arquivos
- [ ] Todos os 5+ arquivos são listados
- [ ] Arquivo padrão (se existe) é marcado com ⭐
- [ ] Pergunta se quer usar o padrão
- [ ] Consegue escolher outro por índice

### Rodar do Zero - Cenário 2: Com pesos
- [ ] Pergunta "Deseja carregar pesos?"
- [ ] Responder "s" → mapeamento completo
- [ ] Auditoria mostra % de cobertura
- [ ] Se cobertura baixa → ALERTA

### Rodar do Zero - Cenário 3: Sem pesos
- [ ] Responder "n" → segue sem carteira atual
- [ ] Gráfico não tem estrela dourada
- [ ] Tabelas não têm coluna "Carteira Atual"

### Rodar do Zero - Cenário 4: Ativos excluídos por MIN_OBS
- [ ] Status "MAPEADO_EXCLUIDO_MIN_OBS" aparece
- [ ] Peso NÃO é remapeado para outro ativo
- [ ] % excluído aparece na auditoria

---

## Arquivos Entregues

| Arquivo | Descrição |
|---------|-----------|
| `Otimizacao_de_Portfolio_v2_6.ipynb` | Notebook completo |
| `CHANGELOG_v2.6.md` | Este documento |

---

*Versão: 2.6*
*Data: Janeiro/2026*
