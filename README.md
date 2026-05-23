# Turkish PII Detection with OpenAI Privacy Filter

This repository documents an end-to-end Turkish PII detection project based on **OpenAI Privacy Filter**.  
The project adapts the original English-oriented privacy filtering model to Turkish by creating a large synthetic Turkish privacy NER dataset, fine-tuning the model with Turkish-specific privacy labels, and evaluating the resulting checkpoint on a held-out synthetic test split.

Author: **Boran Toktay**

---

## Project Overview

Personally identifiable information (PII) detection is an important NLP task for privacy-preserving AI systems, especially in logs, customer support messages, forms, educational records, medical-style documents, indexing pipelines, and model training data.

The original [`openai/privacy-filter`](https://huggingface.co/openai/privacy-filter) model is designed for detecting and redacting privacy-sensitive spans in text. However, Turkish contains language-specific challenges such as suffixes, local identifiers, Turkish address patterns, TCKN-like numbers, VKN-like numbers, Turkish phone formats, and IBAN variations.

This project fine-tunes OpenAI Privacy Filter for Turkish PII detection using a custom synthetic dataset and a Turkish-specific privacy label space.

---

## Hugging Face Resources

| Resource | Link |
|---|---|
| Dataset | [`BTX24/turkish-privacy-pii-ner`](https://huggingface.co/datasets/BTX24/turkish-privacy-pii-ner) |
| Fine-tuned model | [`BTX24/turkish-privacy-filter-pii`](https://huggingface.co/BTX24/turkish-privacy-filter-pii) |
| Base model | [`openai/privacy-filter`](https://huggingface.co/openai/privacy-filter) |
| Base GitHub repository | [`openai/privacy-filter`](https://github.com/openai/privacy-filter) |

---

## Main Contributions

This project includes:

- A fully synthetic Turkish privacy NER dataset with **103,923 JSONL rows**
- A Turkish-specific privacy label space
- A Colab-based fine-tuning workflow for OpenAI Privacy Filter
- A fine-tuned Turkish Privacy Filter checkpoint
- Evaluation results on a held-out synthetic Turkish test split
- Documentation for dataset preparation, model training, inference, and limitations

---

## Repository Structure

Recommended structure:

```text
turkish-pii-detection-openai-privacy-filter/
├── README.md
├── LICENSE
├── privacy_filter_tr_pii_colab.ipynb
├── configs/
│   └── label_space.json
├── reports/
│   └── Turkish_Privacy_Filter_Final_Report_English_Boran_Toktay.docx
```

The full dataset and model weights are hosted on Hugging Face, not directly in this GitHub repository.

---

## Dataset

The dataset used in this project is available at:

```text
https://huggingface.co/datasets/BTX24/turkish-privacy-pii-ner
```

### Dataset Summary

| Metric | Value |
|---|---:|
| Total JSONL rows | 103,923 |
| Label classes | 10 |
| Training examples | 83,138 |
| Validation examples | 10,392 |
| Test examples | 10,393 |
| Total characters | 7,404,532 |
| Average text length | 71.3 characters |
| Average entity length | 19.2 characters |
| Dataset type | Synthetic Turkish privacy NER |
| Annotation type | Character-level spans |

The dataset is fully synthetic. No real personal data was intentionally collected or used.

---

## Label Space

The final Turkish privacy label space is:

```json
{
  "category_version": "tr_privacy_v1",
  "span_class_names": [
    "O",
    "tckn",
    "secret",
    "iban",
    "vkn",
    "account_number",
    "private_address",
    "private_date",
    "private_phone",
    "private_email",
    "private_person"
  ]
}
```

### Label Descriptions

| Label | Description |
|---|---|
| `O` | Outside / non-PII token |
| `tckn` | Synthetic Turkish national identity-like values |
| `secret` | Synthetic passwords, OTPs, API-key-like strings, access tokens, recovery codes |
| `iban` | Synthetic Turkish IBAN-like values |
| `vkn` | Synthetic Turkish tax identification-like values |
| `account_number` | Account numbers, customer numbers, reference codes, membership IDs, order/ticket references |
| `private_address` | Synthetic Turkish-style address expressions |
| `private_date` | Privacy-relevant date expressions |
| `private_phone` | Turkish mobile-number-like synthetic phone numbers |
| `private_email` | Synthetic non-routable e-mail addresses |
| `private_person` | Synthetic Turkish-style person names |

---

## Dataset Format

The dataset uses JSONL format with character-level span annotations.

Example:

```json
{
  "text": "Ahmet Yılmaz için telefon numarası 0532 000 00 00 olarak kaydedildi.",
  "spans": {
    "private_person: Ahmet Yılmaz": [[0, 12]],
    "private_phone: 0532 000 00 00": [[35, 49]]
  },
  "info": {
    "id": "synthetic_tr_000001",
    "source": "synthetic_tr"
  }
}
```

Character offsets use Python slicing semantics:

```python
text[start:end]
```

The `end` index is exclusive.

---

## Model

The fine-tuned model is available at:

```text
https://huggingface.co/BTX24/turkish-privacy-filter-pii
```

It is based on:

```text
openai/privacy-filter
```

The model was fine-tuned on the Turkish synthetic privacy NER dataset using a custom Turkish privacy label space.

---

## Training Summary

| Metric | Value |
|---|---:|
| Base model | `openai/privacy-filter` |
| Fine-tuned model | `BTX24/turkish-privacy-filter-pii` |
| Dataset | `BTX24/turkish-privacy-pii-ner` |
| Best epoch | 3 |
| Best metric | `validation_loss` |
| Best validation loss | 0.002157915852249276 |
| Training examples | 83,138 |
| Validation examples | 10,392 |
| Device | Google Colab Pro GPU environment |

### Epoch Metrics

| Epoch | Train Loss | Train Token Accuracy | Validation Loss | Validation Token Accuracy |
|---:|---:|---:|---:|---:|
| 1 | 0.038908 | 0.990768 | 0.003961 | 0.999100 |
| 2 | 0.002714 | 0.999463 | 0.002181 | 0.999583 |
| 3 | 0.001393 | 0.999704 | 0.002158 | 0.999641 |

The best checkpoint was selected at epoch 3 based on validation loss.

---

## Test Evaluation

Evaluation was performed on the held-out synthetic Turkish test split.

| Metric | Value |
|---|---:|
| Test examples | 10,393 |
| Test tokens | 241,020 |
| Eval mode | typed |
| Loss | 0.0028 |
| Token accuracy | 0.9996 |
| Inference tokens/sec | 3027.20 |

### Detection Metrics

| Metric | Value |
|---|---:|
| Detection precision | 0.9998 |
| Detection recall | 0.9996 |
| Detection F1 | 0.9997 |
| Detection F2 | 0.9996 |
| Span precision | 0.9988 |
| Span recall | 0.9978 |
| Span F1 | 0.9983 |
| Span F2 | 0.9980 |

### Per-Class Span Metrics

| Label | Precision | Recall | F1 | F2 |
|---|---:|---:|---:|---:|
| `tckn` | 1.0000 | 0.9990 | 0.9995 | 0.9992 |
| `secret` | 0.9990 | 1.0000 | 0.9995 | 0.9998 |
| `iban` | 1.0000 | 1.0000 | 1.0000 | 1.0000 |
| `vkn` | 0.9990 | 0.9990 | 0.9990 | 0.9990 |
| `account_number` | 1.0000 | 0.9992 | 0.9996 | 0.9993 |
| `private_address` | 0.9980 | 0.9940 | 0.9960 | 0.9948 |
| `private_date` | 1.0000 | 1.0000 | 1.0000 | 1.0000 |
| `private_phone` | 0.9991 | 1.0000 | 0.9995 | 0.9998 |
| `private_email` | 1.0000 | 0.9902 | 0.9951 | 0.9922 |
| `private_person` | 0.9928 | 0.9959 | 0.9943 | 0.9953 |

These results are measured on a synthetic test split. Real-world performance may differ.

---

## Colab Notebook

The main training notebook is:

```text
notebooks/privacy_filter_tr_pii_colab.ipynb
```

The notebook covers:

- Installing OpenAI Privacy Filter
- Downloading the OPF-native base checkpoint
- Creating `label_space.json`
- Loading and validating the Turkish privacy dataset
- Converting span annotations into OPF-compatible format
- Running fine-tuning
- Running evaluation
- Testing inference on Turkish examples
- Uploading the fine-tuned checkpoint to Hugging Face
- Verifying the uploaded model from Hugging Face

---

## Important Implementation Notes

During development, several practical engineering issues were resolved:

### JSONL newline issue

A JSONL writing function originally wrote records without newline characters, causing:

```text
JSONDecodeError: Extra data
```

The fix was to ensure each JSON object is written on a separate line:

```python
f.write(json.dumps(row, ensure_ascii=False) + "\n")
```

### OPF checkpoint path issue

The OPF training command expects the OPF-native checkpoint under the `original/` directory of `openai/privacy-filter`, not only the root Transformers-style checkpoint.

Correct checkpoint loading pattern:

```python
from huggingface_hub import snapshot_download

snapshot_download(
    repo_id="openai/privacy-filter",
    repo_type="model",
    local_dir=str(BASE_SNAPSHOT_DIR),
    allow_patterns=["original/*"],
)
```

Then use:

```text
--checkpoint path/to/base_openai_privacy_filter_snapshot/original
```

### OPF JSON output parsing issue

Some OPF CLI outputs may include JSON followed by additional color legend text. To avoid `JSONDecodeError`, parse only the first JSON object:

```python
payload, end_idx = json.JSONDecoder().raw_decode(stdout.lstrip())
```

---

## Local Usage

### 1. Install OpenAI Privacy Filter

```bash
git clone https://github.com/openai/privacy-filter.git
cd privacy-filter
pip install -e .
```

### 2. Download the fine-tuned checkpoint

```bash
python -c "from huggingface_hub import snapshot_download; snapshot_download(repo_id='BTX24/turkish-privacy-filter-pii', local_dir='tr_privacy_filter_pii')"
```

### 3. Run inference

CUDA:

```bash
opf --checkpoint ./tr_privacy_filter_pii --device cuda --format json "Mehmet Kaya TCKN 12345678901"
```

CPU:

```bash
opf --checkpoint ./tr_privacy_filter_pii --device cpu --format json "Mehmet Kaya TCKN 12345678901"
```

More examples:

```bash
opf --checkpoint ./tr_privacy_filter_pii --device cuda --format json "Ahmet Yılmaz için telefon numarası 0532 000 00 00 olarak kaydedildi."
```

```bash
opf --checkpoint ./tr_privacy_filter_pii --device cuda --format json "İade için IBAN TR00 0000 0000 0000 0000 0000 00 bilgisi girildi."
```

```bash
opf --checkpoint ./tr_privacy_filter_pii --device cuda --format json "Doğrulama kodu OTP-482193 destek kaydına yazılmış."
```

---

## Python Download Example

```python
from huggingface_hub import snapshot_download

checkpoint_dir = snapshot_download(
    repo_id="BTX24/turkish-privacy-filter-pii",
    local_dir="tr_privacy_filter_pii",
)

print(checkpoint_dir)
```

---

## Example Turkish Test Text

```text
Hasta Mehmet Kaya, 14.03.2025 tarihinde Ankara Çankaya'daki Atatürk Mahallesi No: 12 Daire: 5 adresinden başvuru yaptı. TCKN bilgisi 12345678901 olarak, VKN bilgisi ise 1234567890 olarak kaydedildi.

Ödeme için verilen IBAN: TR330006100519786457841326. İletişim telefonu 0555 123 45 67. E-posta adresi mehmet.kaya@example.com.

Sistem entegrasyonu için secret değeri sk-demo-1234567890abcdef olarak not edildi.
```

Expected behavior:

- Person names → `private_person`
- Dates → `private_date`
- Addresses → `private_address`
- TCKN-like values → `tckn`
- VKN-like values → `vkn`
- IBAN-like values → `iban`
- Phone numbers → `private_phone`
- E-mails → `private_email`
- Secret-like values → `secret`

---

## Limitations

This project uses synthetic data. Therefore:

- The model may overfit to synthetic templates.
- Real-world Turkish text may contain noisier, longer, or more ambiguous expressions.
- OCR errors, informal spelling, mixed-language text, and domain-specific documents may reduce performance.
- The model may produce false positives for numeric or code-like strings.
- The model may miss rare PII formats not represented in the synthetic dataset.
- Results should be validated on real or manually curated in-domain test sets before production use.

This model should not be treated as a legal anonymization guarantee.

---

## Ethical Considerations

The dataset was generated synthetically to avoid intentionally collecting or distributing real personal information.

However, models trained on synthetic data can still make mistakes. For sensitive or production use cases, this model should be used as one component of a broader privacy-preserving pipeline, together with:

- Human review
- Rule-based checks
- Domain-specific validation
- Conservative redaction policies
- Logging and monitoring
- Privacy-by-design safeguards

---

## Project Deliverables

This project includes:

- Synthetic Turkish privacy NER dataset
- Fine-tuned Turkish Privacy Filter model
- Colab training notebook
- Final academic report
- Dataset card
- Model card
- Evaluation metrics
- Example inference workflow

---

## License

This repository’s code and documentation are released under the **Apache License 2.0**, unless otherwise stated.

Related resources:

| Resource | License |
|---|---|
| Project code | Apache-2.0 |
| Fine-tuned model | Apache-2.0 |
| Training dataset | CC BY 4.0 |
| Base model | See `openai/privacy-filter` license |

Please review the licenses of the base model, dataset, and fine-tuned checkpoint before use.

---

## Citation

If you use this project or model, please cite:

```bibtex
@model{toktay_2026_turkish_privacy_filter_pii,
  title        = {Turkish Privacy Filter PII},
  author       = {Boran Toktay},
  year         = {2026},
  publisher    = {Hugging Face},
  howpublished = {\url{https://huggingface.co/BTX24/turkish-privacy-filter-pii}},
  note         = {Fine-tuned OpenAI Privacy Filter checkpoint for Turkish PII span detection}
}
```

If you use the dataset, please also cite:

```bibtex
@dataset{toktay_2026_turkish_privacy_pii_ner,
  title        = {Turkish Privacy PII NER Dataset},
  author       = {Boran Toktay},
  year         = {2026},
  publisher    = {Hugging Face},
  howpublished = {\url{https://huggingface.co/datasets/BTX24/turkish-privacy-pii-ner}},
  note         = {Synthetic Turkish privacy-oriented named entity recognition dataset for PII detection}
}
```

---

## Acknowledgements

This project builds on:

- [`openai/privacy-filter`](https://github.com/openai/privacy-filter)
- [`openai/privacy-filter` on Hugging Face](https://huggingface.co/openai/privacy-filter)
- Hugging Face Hub and Datasets tooling
- Synthetic Turkish PII data generation workflows

---

## Contact

For questions, suggestions, or issues, please open an issue in this repository.
