# Sentimento de notícias de cripto × retorno do Bitcoin

Pipeline completo de **coleta → NLP → alinhamento temporal** para responder uma pergunta de pesquisa:
**o sentimento de notícias de cripto precede o movimento do Bitcoin?**

## Pipeline

```
cryptorank.io (40 scrolls, Selenium)
        │
        ├─► BeautifulSoup extrai título + texto das notícias
        │
        ├─► filtro por fonte (AMBCrypto, Bitcoin Magazine, Bitcoinist,
        │   Consensys, U.Today, Watcher.Guru, NewsBTC, The Daily Hodl...)
        │
        ├─► DistilRoBERTa fine-tuned para notícia FINANCEIRA
        │   (mrm8488/distilroberta-finetuned-financial-news-sentiment-analysis)
        │       → label: positive / negative / neutral
        │
        ├─► yfinance: BTC-USD → Return e Return_D1 (retorno do dia seguinte)
        │
        └─► merge por data → DataFrame com Sentiment_Fact × Direction_D1_Fact
```

## O achado técnico do projeto

O modelo de sentimento **não é genérico** — é um **DistilRoBERTa fine-tuned especificamente em notícia
financeira**, carregado via `transformers.pipeline("text-classification")` direto do Hugging Face.

Escolher o modelo pelo domínio (financeiro) em vez de usar um genérico é a diferença entre captar
"o mercado caiu" como negativo e captar como neutro.

## Estado do notebook

> ⚠️ **Resultado não registrado.** O DataFrame final é construído (sentimento + retorno do dia seguinte),
> mas a **correlação sentimento × retorno não foi calculada nem salva** na versão publicada. O notebook
> para em `df` com as duas colunas prontas para análise.
>
> O scraper também depende de **XPath/CSS do cryptorank.io de 2024** — o site mudou e o seletor não roda
> mais como está.

**Próximo passo:** desacoplar a coleta (salvar o CSV uma vez) da análise (que vira reprodutível), e rodar
a correlação com teste de significância.

## Limitações

1. Sentimento por título/resumo truncado em 500 caracteres.
2. Correlação ≠ causalidade — notícia e preço podem responder a um terceiro fator comum.
3. Alinhamento por data (não por horário) — notícia publicada após o fechamento entra no dia seguinte.
4. Sem controle para dias sem notícia relevante.

## Stack

`Python` · `Selenium` · `BeautifulSoup` · `transformers` (DistilRoBERTa financeiro) · `pandas` · `yfinance`

## Rodar

```bash
pip install selenium beautifulsoup4 transformers torch pandas yfinance
jupyter notebook sentiment_notices.ipynb
```

## Contexto

Projeto de 2024 explorando NLP aplicado a mercado financeiro. É o predecessor direto do serviço de IA do
Aurah (FastAPI + transformers + Qdrant).
