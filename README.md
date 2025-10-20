# Projeto de Qualidade e Normalização de Endereços

Este repositório reúne estudos e utilitários para:
- Extração de campos de endereços de arquivos texto fixos.
- Normalização de logradouros, nomes e componentes.
- Críticas de qualidade (CEP, UF, CIDADE, BAIRRO, presença de bad words).
- Geração de CSV com resultados e classificação.

## Estrutura

- `requirements.txt`: dependências.
- `endereco/`:
  - `endereco.ipynb`: pipeline principal (carregamento, normalização, críticas).
  - Arquivos texto de apoio: `base_ENDERECO.TXT`, `NORMA_ENDERCO.TXT`, `bad_words.TXT`.
  - Saída gerada: `resultado_endereco_bw.csv`.
- Pastas adicionais de estudos: `analise_preditiva/`, `fatec/`, `modelos/`, `tg/`.

## Principais Classes

- [`Critica`](endereco/endereco.ipynb): aplica regras regex para CEP, UF, CIDADE, BAIRRO e detecção de bad words.
- [`NormalizadorEndereco`](endereco/endereco.ipynb): usa tabela `NORMA_ENDERCO.TXT` para abreviações e tratamentos especiais.

## Fluxo Resumido

1. Ler base fixa e fatiar colunas.
2. Padronizar (maiúsculas, trim, remover acentos).
3. Classificar tipo de logradouro (regex).
4. Rodar críticas de qualidade.
5. Carregar bad words e marcar ocorrências.
6. Normalizar endereço completo.
7. Exportar resultados.

## Pré-requisitos

```
python -m venv .venv
source .venv/Scripts/activate  # Windows PowerShell
pip install -r requirements.txt
```

## Exemplo de Uso (script simplificado)

````python
import pandas as pd
from endereco.endereco import Critica, NormalizadorEndereco  # se extraído do notebook

#base a ser tratada (leia a sua base aqui)
base_endereco = pd.read_csv('endereco/base_ENDERECO.TXT', header=None, on_bad_lines='skip', encoding='latin1')

# bad words 
with open('endereco/bad_words.TXT','r',encoding='utf-8') as f:
    bad_words = [ln.strip().upper().split(' ')[0] for ln in f if ln.strip()]


#formatando base
df = pd.DataFrame()
df["ID"] = base_endereco[0].str[0:6]
df["ENDERECO"] = base_endereco[0].str[6:126]
df["BAIRRO"] = base_endereco[0].str[126:186]
df["CIDADE"] = base_endereco[0].str[186:246]
df["UF"] = base_endereco[0].str[246:249]
df["CEP"] = base_endereco[0].str[255:264]


#Instanciando critica
crit = Critica(df.copy())

#rodando criticas - ajustas dps
df_criticas = crit.rodar_criticas()



df_criticas = crit.rodar_bad_word(bad_words) #colocar  antes

normalizador = NormalizadorEndereco("endereco/NORMA_ENDERCO.TXT")
df_criticas['ENDERECO_NORMALIZADO'] = df_criticas['ENDERECO'].apply(normalizador.normalizar_endereco)

df_criticas.to_csv('endereco/resultado_endereco_bw.csv', index=False, sep=';')
````