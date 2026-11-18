# NLP-mini-project

## Sammenlign features mellem Human og LLM

`compare_features.py` bruger datasættets labels: **0 = Human**, **1 = LLM**.
Scriptet sammenligner stopord både som antal pr. tekst og som antal pr. 100
tokens uden tegnsætning og whitespace. På den måde kan man undersøge forskelle
i sprogbrug og samtidig tage højde for tekstlængden.

Installer pakkerne og kør scriptet fra projektmappen:

```powershell
python -m pip install pandas matplotlib spacy
python compare_features.py --input train.jsonl
```

Stopord kræver kun den engelske tokenizer og stopordsliste. Den selvstændige
analyse behøver derfor ikke at downloade `en_core_web_sm`.

Output gemmes i `feature_comparison/`:

- `feature_values.csv`: featureværdier og metadata for hver tekst.
- `count_stopwords_summary.csv` og `stopwords_per_100_tokens_summary.csv`:
  antal tekster, gennemsnit, median, standardafvigelse, min./maks., antal nuller
  og procent nuller i hver gruppe.
- To PNG-filer med histogrammer og boxplots.

Histogrammerne viser tæthed, så grupper med forskelligt antal tekster kan
sammenlignes. Nulværdier indgår i både statistikken og figurerne. En tekst uden
ordtokens får stopordsantal 0, men dens rate er udefineret og rapporteres i
`missing_count`. `n` er antal definerede værdier; `zero_percent` bruger dette antal
som nævner. Standardafvigelsen er udefineret, hvis gruppen kun har én værdi.

Du kan også sammenligne inden for én genre:

```powershell
python compare_features.py --input train.jsonl --genre news --output-dir feature_comparison/news
```

Vælg et genrenavn fra `df["genre"].unique()`. Forskelle i den samlede analyse kan
også skyldes forskellig fordeling af genrer og LLM-modeller mellem grupperne.
Statistikken er beskrivende og dokumenterer ikke i sig selv, om en feature kan
identificere forfattertypen på nye tekster.

### Brug dine egne funktioner i notebooken

Kør denne celle efter definitionen af fx `count_stopwords`:

```python
import matplotlib.pyplot as plt
from compare_features import compare_feature

values, summary, figure = compare_feature(df, count_stopwords)
display(summary)
plt.show()
```

Udskift `count_stopwords` med en anden funktion, der returnerer ét tal pr. tekst,
fx `count_punctuation`. En featurekolonne, som allerede er beregnet, kan også
bruges: `compare_feature(df, "count_stopwords")`.

For POS-funktioner, der returnerer en dictionary, vælger du først en kategori:

```python
values, summary, figure = compare_feature(
    df,
    lambda text: count_pos_tags(text)["NOUN"],
    feature_name="count_nouns",
)
display(summary)
plt.show()
```

Når funktionen sendes direkte, kaldes den én gang pr. tekst. Den selvstændige
stopordsanalyse bruger `nlp.pipe()` og beregner antal og længde i samme gennemløb.
