# Apoio computacional com IA para mapeamento sistemático em Engenharia de Software

Este repositório reúne scripts e notebooks em Python desenvolvidos para apoiar a condução de um mapeamento sistemático da literatura sobre **ferramentas e abordagens para detecção de *code smells* em JavaScript**.

O objetivo do projeto é oferecer um fluxo computacional reprodutível para auxiliar etapas operacionais do mapeamento, especialmente triagem, leitura de texto completo, extração estruturada de dados e geração de arquivos de auditoria. As decisões metodológicas finais permanecem sob responsabilidade do pesquisador.

## Fundamentação metodológica

O fluxo implementado foi inspirado em trabalhos recentes sobre o uso de modelos de linguagem em revisões e mapeamentos sistemáticos:

- Syriani, E.; David, I.; Kumar, G. **Screening articles for systematic reviews with ChatGPT**. *Journal of Computer Languages*, v. 80, 101287, 2024. DOI: [10.1016/j.cola.2024.101287](https://doi.org/10.1016/j.cola.2024.101287).
- Teperikidis, L.; Boulmpou, A.; Papadopoulos, C.; Biondi-Zoccai, G. **Using ChatGPT to perform a systematic review: a tutorial**. *Minerva Cardiology and Angiology*, v. 72, n. 6, p. 547--567, 2024. DOI: [10.23736/S2724-5683.24.06568-2](https://doi.org/10.23736/S2724-5683.24.06568-2).

Do primeiro trabalho, foram incorporados princípios relacionados à triagem automatizada com ChatGPT, como uso de tópico explícito, critérios de exclusão, respostas padronizadas, configuração de baixa variabilidade e estratégia conservadora para reduzir exclusões indevidas. Do segundo, foram incorporadas práticas de apoio ao processo de revisão, como uso de IA na triagem, divisão de textos longos em blocos, extração estruturada, documentação das interações, controle de alucinações e validação manual.

## Escopo do repositório

Este repositório não automatiza integralmente um mapeamento sistemático. Ele oferece apoio computacional para tarefas específicas:

1. triagem por título e resumo;
2. leitura e processamento de PDFs em texto completo;
3. divisão de textos longos em blocos (*chunking*);
4. identificação de sinais de inclusão e exclusão;
5. consolidação de decisões sugeridas de elegibilidade;
6. extração estruturada de dados dos estudos incluídos;
7. geração de arquivos intermediários em `CSV`, `XLSX` e `JSON`;
8. registro de evidências e informações para auditoria;
9. preparação de campos para validação humana.

A ferramenta deve ser utilizada como **apoio à decisão**, não como substituta da avaliação humana.

## Estrutura sugerida do projeto

A organização dos arquivos pode seguir a estrutura abaixo:

```text
.
├── README.md
├── notebooks/
│   ├── 01_triagem_titulo_resumo.ipynb
│   ├── 02_triagem_texto_completo.ipynb
│   └── 03_extracao_dados.ipynb
├── scripts/
│   └── TCC_script_redigido.py
├── dados/
│   ├── entrada/
│   │   ├── registros_busca.csv
│   │   └── pdfs/
│   ├── saida/
│   │   ├── screened_artigos.csv
│   │   ├── fulltext_screening_results.xlsx
│   │   └── extraction_matrix.xlsx
│   └── auditoria/
│       ├── fulltext_audit_json/
│       └── extraction_audit_json/
└── referencias/
    └── referencias-ia.bib
```

A estrutura real pode ser ajustada conforme a organização local do Google Drive ou do ambiente de execução.

## Requisitos

O código foi desenvolvido para execução em ambiente **Google Colab**, com linguagem **Python**. Também pode ser adaptado para execução local, desde que as dependências sejam instaladas e os caminhos dos arquivos sejam ajustados.

Principais dependências:

```bash
pip install openai pandas openpyxl pymupdf tenacity tqdm pydantic pypdf
```

Dependências principais utilizadas:

- `openai`: comunicação com a API do modelo de linguagem;
- `pandas`: leitura e escrita de planilhas e arquivos tabulares;
- `openpyxl`: exportação de resultados em formato `.xlsx`;
- `pymupdf`/`fitz`: extração de texto de arquivos PDF;
- `pydantic`: definição de esquemas estruturados de extração;
- `tenacity`: repetição automática de chamadas em caso de falha;
- `tqdm`: acompanhamento visual do progresso.

## Configuração da chave de API

Para executar os scripts com modelos da OpenAI, é necessário configurar uma chave de API. No Google Colab, recomenda-se armazenar a chave em **Secrets**, evitando registrá-la diretamente no código.

Exemplo recomendado:

```python
from google.colab import userdata
OPENAI_API_KEY = userdata.get("OPENAI_API_KEY")
```

**Não é recomendado incluir chaves de API, tokens, senhas ou credenciais em notebooks, scripts, repositórios públicos ou apêndices acadêmicos**.

## Dados de entrada

Os dados de entrada esperados variam conforme a etapa:

### Triagem por título e resumo

Arquivo `.csv` contendo, no mínimo, as colunas:

```text
title, abstract
```

Colunas adicionais podem ser utilizadas quando disponíveis:

```text
year, source, doi, conclusion
```

### Triagem por texto completo

Diretório contendo arquivos `.pdf` dos estudos candidatos à leitura completa.

### Extração de dados

Diretório contendo os PDFs dos estudos incluídos após a etapa de seleção.

## Fluxo de execução

### 1. Triagem por título e resumo

Nesta etapa, o script recebe registros bibliográficos e solicita ao modelo uma decisão preliminar com base no título e no resumo. As decisões possíveis podem ser:

```text
INCLUDE, EXCLUDE, MAYBE
```

A opção `MAYBE` deve ser usada quando as informações forem insuficientes para uma exclusão segura.

Saídas esperadas:

- decisão sugerida pelo modelo;
- critério acionado;
- justificativa curta;
- nível de confiança;
- modelo utilizado;
- data da execução;
- versão do prompt;
- campos para validação humana.

### 2. Triagem por texto completo

Nesta etapa, os PDFs são convertidos em texto, divididos em blocos e analisados separadamente. Cada bloco é usado para identificar sinais de inclusão, sinais de exclusão e trechos de evidência textual. Em seguida, uma etapa de consolidação gera uma decisão sugerida para o estudo completo:

```text
INCLUDE, EXCLUDE, UNCERTAIN
```

Saídas esperadas:

- título extraído do PDF;
- número de blocos analisados;
- decisão sugerida;
- critérios acionados;
- evidências textuais;
- nível de confiança;
- arquivo `.json` de auditoria;
- campos para decisão humana.

### 3. Extração estruturada dos dados

Nesta etapa, os estudos incluídos são processados para gerar uma matriz de extração. Os campos extraídos incluem:

- identificação do estudo;
- nome da ferramenta ou abordagem;
- tipo de artefato;
- escopo da linguagem;
- tipos de *code smells*;
- técnica de detecção;
- tipo de análise;
- tipo de validação;
- base de teste, projetos ou *dataset*;
- limitações relatadas;
- principais achados;
- evidências textuais associadas.

Quando uma informação não é encontrada no texto, o script deve registrar valores como `not_reported`, `not_clear` ou `NOT FOUND`, evitando inferências não sustentadas.

## Saídas geradas

O fluxo pode produzir os seguintes arquivos:

```text
screened_artigos.csv
fulltext_screening_results.csv
fulltext_screening_results.xlsx
extraction_matrix.csv
extraction_matrix.xlsx
fulltext_audit_json/*.json
extraction_audit_json/*.json
```

Os arquivos `CSV` e `XLSX` apoiam a análise tabular. Os arquivos `JSON` registram saídas intermediárias, evidências textuais e respostas brutas do modelo, funcionando como material de auditoria.

## Validação humana

As saídas geradas pela IA devem ser verificadas manualmente. Em especial, recomenda-se revisar:

- artigos classificados como `EXCLUDE`;
- artigos classificados como `MAYBE` ou `UNCERTAIN`;
- campos de extração com baixa confiança;
- informações marcadas como `not_clear`, `not_reported` ou `NOT FOUND`;
- trechos em que a evidência textual não sustenta claramente a classificação ou extração.

A decisão final de inclusão, exclusão, classificação e interpretação dos resultados deve permanecer com o pesquisador.

## Reprodutibilidade e rastreabilidade

Para permitir reprodutibilidade, recomenda-se registrar:

- versão do modelo utilizado;
- versão do prompt;
- data e hora de execução;
- critérios de inclusão e exclusão usados no prompt;
- arquivos de entrada;
- arquivos de saída;
- decisões sugeridas pela IA;
- decisões finais revisadas pelo pesquisador;
- justificativas para divergências entre IA e decisão humana.

O uso de arquivos de auditoria em `JSON` é recomendado para preservar o histórico das respostas intermediárias do modelo.

## Limitações

O uso de modelos de linguagem neste fluxo possui limitações importantes:

- possibilidade de alucinação ou geração de informação não presente no texto;
- sensibilidade à formulação dos prompts;
- perda de contexto em textos longos;
- dependência de extração correta do texto dos PDFs;
- variação de desempenho entre modelos e versões;
- risco de classificação incorreta em artigos ambíguos;
- necessidade de revisão humana para validação metodológica.

Essas limitações devem ser consideradas na seção de ameaças à validade do trabalho científico.

## Segurança e privacidade

Antes de compartilhar notebooks, scripts ou apêndices, verifique se foram removidos:

- chaves de API;
- credenciais de acesso;
- caminhos pessoais sensíveis;
- dados privados ou não autorizados;
- identificadores que não devam ser publicados.

## Como citar este material

Caso este repositório seja usado como apoio metodológico, recomenda-se citar os trabalhos que fundamentaram o uso de IA no processo:

```bibtex
@article{syriani2024screening,
  author  = {Syriani, Eugene and David, Istvan and Kumar, Gauransh},
  title   = {Screening articles for systematic reviews with ChatGPT},
  journal = {Journal of Computer Languages},
  volume  = {80},
  pages   = {101287},
  year    = {2024},
  doi     = {10.1016/j.cola.2024.101287}
}

@article{teperikidis2024using,
  author  = {Teperikidis, Lefteris and Boulmpou, Aristi and Papadopoulos, Christodoulos and Biondi-Zoccai, Giuseppe},
  title   = {Using ChatGPT to perform a systematic review: a tutorial},
  journal = {Minerva Cardiology and Angiology},
  volume  = {72},
  number  = {6},
  pages   = {547--567},
  year    = {2024},
  doi     = {10.23736/S2724-5683.24.06568-2}
}
```

## Observação final

Este projeto foi desenvolvido como apoio computacional a um trabalho acadêmico. O uso de IA deve ser relatado de forma transparente na metodologia, nos apêndices e, quando necessário, nas ameaças à validade. O código auxilia a organização e a rastreabilidade do processo, mas não substitui o julgamento crítico do pesquisador.
