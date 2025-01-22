MedNERN-CR-JA にバイオマーカの正規化機能を追加したバージョンです．
MedNERN-CR-JA 関して詳しくは，[MedNERN-CR-JA GitHub page](https://github.com/sociocom/MedNERN-CR-JA)をご覧ください．

モデルは[HuggingFace model page](https://huggingface.co/sociocom/MedNERN-CR-JA)よりダウンロードしてください．

## How to use

Install the requirements:

Using pip:

```
pip install -r requirements.txt
```

Using uv:

```
uv sync
```

## Prediction

The prediction script will output the results in the same XML format as the input file. It can be run with the following
command:

```
python3 predict.py
```

The default parameters will take the model located in `pytorch_model.bin` and the input file `text.txt`.
The resulting predictions will be output to the screen.

To select a different model or input file, use the `-m` and `-i` parameters, respectively:

```
python3 predict.py -m <model_path> -i <your_input_file>.txt
```

The input file can be a single text file or a folder containing multiple `.txt` files, for batch processing. For example:

```
python3 predict.py -m <model_path> -i <your_input_folder>
```

### Entity normalization

This model supports entity normalization via dictionary matching. The dictionary is a list of medical terms or
drugs and their standard forms.

Two different dictionaries are used for drug and disease normalization, stored in the `dictionaries` folder as
`drug_dict.csv` and `disease_dict.csv`, respectively.

To enable normalization you can add the `--normalize` flag to the `predict.py` command.

```
python3 predict.py -m <model_path> --normalize
```

Normalization will add the `norm` attribute to the output XML tags. This attribute can be empty if a normalized form of
the term is not found.

The default disease normalization dictionary (`dictionaties/disease_dict.csv`) is based on
the [Manbyo Dictionary](https://sociocom.naist.jp/manbyo-dic-en/) and provides normalization to the standard ICD code
for the diseases.

The default drug dictionary (`dictionaties/drug_dict.csv`) is based on
the [Hyakuyaku Dictionary](https://sociocom.naist.jp/hyakuyaku-dic-en/).

**NOTE:**Those files are not provided in this github to prevent issues with git LFS. Please download them from either the links
above or the HuggingFace repository.

The dictionary is a CSV file with three columns: the first column is the surface form term and the third column contain
its standard form. The second column is not used.

### Replacing the default dictionaries

User can freely change the dictionary to fit their needs by passing the path to a custom dictionary file.
The dictionary file must have at least a column containing the list of surface forms and a column containing the list of
normalized forms.

The parameters `--drug_dict`, `--disease_dict` and `--test-dict` can be used to specify the path to the drug and disease dictionaries,
respectively.
When doing so, the respective parameters informing the column index of the surface form and normalized form must also be
provided.
You don't need to replace both dictionaries at the same time, you can replace only one of them.

E.g.:

```
uv run python predict.py --normalize \
    --drug-dict dictionaries/drug_dict.csv \
    --drug-candidate-col 0 \
    --drug-normalization-col 2 \
    --disease-dict dictionaries/disease_dict.csv \
    --disease-candidate-col 0 \
    --disease-normalization-col 2 \
    --input text_biomarker.txt \
    --test-dict dictionaries/test_dict.csv \
    --test-candidate-col 0 \
    --test-normalization-col 2 \
    --output ner_output_test.txt
```

### Input Example

```
ERとPRは陽性である（陽性細胞はそれぞれ15％および1-5％。MB-1 indexは約23％である。）
抗ＨＥＲ２抗体 score 1＋．
```

### Output Example

```
<t-key norm="ER, PgR (PR)">ERとPR</t-key>は<t-val>陽性</t-val>である（<a>陽性細胞</a>はそれぞれ<t-val>15％</t-val>および<t-val>1-5％</t-val>。
<t-key norm="Ki-67">MB-1 index</t-key>は約23％である。）
<t-key norm="HER2">抗ＨＥＲ２抗体</t-key> <t-val>score 1＋．</t-val>
```
