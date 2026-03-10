# Dane bez Twarzy (Data Without a Face)

## Application Demo

A quick overview of the application in practice
 [Demo video](https://drive.google.com/file/d/1zjd5R_XPt_ISOmOvj2ORvCBK8GyUF-XW/view)

**Personal Data Anonymization System for Polish Texts**

A Python library for automatic detection and anonymization of sensitive data in Polish-language texts. Designed for the PLLuM project (Polish Large Language Model).

## Project Goals

- **Data Security** - detection and anonymization of personal data in compliance with GDPR
- **Structure Preservation** - replacing entities with tokens that maintain meaning and grammar
- **Polish Language Support** - full handling of inflection and linguistic context
- **Performance** - scalable solution for processing large datasets

## Architecture

The system uses a multi-layered architecture:

```
┌─────────────────────────────────────────────────────────────┐
│                          INPUT                              │
│              "Jan Kowalski, PESEL 90010112345"              │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                 STAGE 1: REGEX LAYER                        │
│                    "Quick Filter"                           │
│  • PESEL (with checksum validation)                         │
│  • Email, phone, IBAN                                       │
│  • Dates (various formats)                                  │
│  • Document numbers                                         │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                STAGE 2: ML LAYER (NER)                      │
│                    "Intelligence"                           │
│  • First names and surnames                                 │
│  • Cities vs addresses (context distinction)                │
│  • Companies, schools, job titles                           │
│  • Sensitive data (health, religion, beliefs)               │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│             STAGE 3: MORPHOLOGICAL ANALYSIS                 │
│  • Case (nominative, genitive, accusative...)               │
│  • Gender (masculine, feminine, neuter)                     │
│  • Number (singular, plural)                                │
└─────────────────────────────────────────────────────────────┘
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
    ┌───────────────────────┐   ┌───────────────────────┐
    │   PATH A              │   │   PATH B              │
    │   Evaluation          │   │   Synthetic Data      │
    │   (F1-score)          │   │   (20% Bonus)         │
    └───────────────────────┘   └───────────────────────┘
                    │                   │
                    ▼                   ▼
    ┌───────────────────────┐   ┌───────────────────────┐
    │ "{name} {surname},    │   │ "Maria Nowak,         │
    │  PESEL {pesel}"       │   │  PESEL 12345678901"   │
    └───────────────────────┘   └───────────────────────┘
```

## Installation

### Requirements

- Python 3.8+
- pip

### Basic Installation

```bash
# Clone the repository
git clone git@github.com:stanislawMarciniak/dane-bez-twarzy.git
cd dane-bez-twarzy

# Create a virtual environment
python -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Download the SpaCy model for Polish
python -m spacy download pl_core_news_lg
```

### GPU Support Installation (optional)

```bash
# PyTorch with CUDA
pip install torch --index-url https://download.pytorch.org/whl/cu118

# Using GPU
python main.py --device cuda --input data.txt --output results.txt
```

## Usage

### Command Line Interface (CLI)

```bash
# Single text
python main.py -t "Jan Kowalski mieszka w Warszawie"

# Single text with replacement
python main.py -t "Jan Kowalski mieszka w Warszawie" --synthetic

# File processing
python main.py -i ./data/orig.txt -o ./results/results.txt

# With synthetic data generation
python main.py -i ./data/orig.txt -o ./results/results.txt --synthetic

# Interactive mode
python main.py --interactive

# Detailed options
python main.py --help
```

### Python API

```python
from anonymizer import anonymize_text, Anonymizer

# Simple usage
result = anonymize_text(
    "Nazywam się Jan Kowalski, mój PESEL to 90010112345.",
    generate_synthetic=True
)
print(result['anonymized'])
# "Nazywam się {name} {surname}, mój PESEL to {pesel}."

print(result['synthetic'])
# "Nazywam się Adam Nowak, mój PESEL to 85032145678."
```

### Advanced Usage

```python
from anonymizer import Anonymizer

# Full configuration
anonymizer = Anonymizer(
    use_ml=True,                    # Enable ML layer (NER)
    use_transformer=False,          # Use SpaCy (faster) instead of Transformer
    morphology_backend="spacy",     # Morphology backend
    generate_synthetic=True,        # Generate synthetic data
    include_intermediate=True,      # Keep intermediate representation
    device="cpu",                   # Device (cpu/cuda)
    num_workers=4                   # Number of workers
)

# Single text
result = anonymizer.anonymize("""
Pacjent: Jan Kowalski
PESEL: 90010112345
Adres: ul. Długa 5, 00-001 Warszawa
Email: jan.kowalski@gmail.com
Diagnoza: cukrzyca typu 2
""")

print(result.anonymized)
print(result.intermediate)  # With morphological metadata
print(result.synthetic)     # Synthetic data
print(result.entities)      # List of detected entities

# Batch processing
texts = ["Text 1...", "Text 2...", "Text 3..."]
results = anonymizer.anonymize_batch(texts, show_progress=True)
```

### File Processing

```python
from anonymizer import anonymize_file

# JSONL
anonymize_file(
    "input_data.jsonl",
    "output_data.jsonl",
    format="jsonl",
    generate_synthetic=True
)

# TXT (one text per line)
anonymize_file(
    "texts.txt",
    "anonymized_texts.txt",
    format="txt"
)
```

## Supported Data Categories

### 1. Personal Identification Data

| Token                  | Description        | Example             |
| ---------------------- | ------------------ | ------------------- |
| `{name}`               | First names        | Jan, Anna           |
| `{surname}`            | Surnames           | Kowalski, Nowak     |
| `{age}`                | Age                | 25 years old        |
| `{date-of-birth}`      | Date of birth      | 15.03.1990          |
| `{date}`               | Other dates        | admitted 23.09.2023 |
| `{sex}`                | Sex                | male, female        |
| `{religion}`           | Religion           | Catholic            |
| `{political-view}`     | Political views    | conservative        |
| `{ethnicity}`          | Ethnic origin      | Ukrainian           |
| `{sexual-orientation}` | Sexual orientation | heterosexual        |
| `{health}`             | Health data        | type 2 diabetes     |
| `{relative}`           | Family relations   | my brother Piotr    |

### 2. Contact and Location Data

| Token       | Description             | Example                      |
| ----------- | ----------------------- | ---------------------------- |
| `{city}`    | City (general location) | I'm going to Kraków          |
| `{address}` | Full address            | ul. Długa 5, 00-001 Warszawa |
| `{email}`   | Email address           | jan@gmail.com                |
| `{phone}`   | Phone number            | +48 123 456 789              |

### 3. Document Identifiers

| Token               | Description      | Example     |
| ------------------- | ---------------- | ----------- |
| `{pesel}`           | PESEL number     | 90010112345 |
| `{document-number}` | Document numbers | ABC123456   |

### 4. Professional and Educational Data

| Token           | Description  | Example              |
| --------------- | ------------ | -------------------- |
| `{company}`     | Company name | TechPol Sp. z o.o.   |
| `{school-name}` | School name  | University of Warsaw |
| `{job-title}`   | Job title    | programmer           |

### 5. Financial Information

| Token                  | Description    | Example             |
| ---------------------- | -------------- | ------------------- |
| `{bank-account}`       | Account number | PL12 1234 ...       |
| `{credit-card-number}` | Card number    | 4111 1111 1111 1111 |

### 6. Digital Identifiers

| Token        | Description         | Example             |
| ------------ | ------------------- | ------------------- |
| `{username}` | Login/username      | @janek123           |
| `{secret}`   | Passwords, API keys | password: secret123 |

## Configuration

### CLI Options

```
Input/Output:
  -i, --input       Input file (JSONL or TXT)
  -o, --output      Output file
  -t, --text        Single text to anonymize
  --interactive     Interactive mode
  --format          File format: jsonl or txt

Processing:
  --synthetic       Generate synthetic data
  --intermediate    Keep intermediate representation
  --no-ml           Disable ML layer (regex only)

Models:
  --transformer     Use Transformer model
  --model-path      Path to custom NER model
  --morphology      Backend: spacy or stanza
  --device          cpu or cuda

Performance:
  --workers         Number of workers (default: 1)
  --seed            Seed for generator

Logging:
  -v, --verbose     Detailed logs
  -q, --quiet       Minimal logs
```

## Data Format

### Input (JSONL)

```json
{"text": "Jan Kowalski mieszka w Warszawie."}
{"text": "Kontakt: anna@email.pl, tel. 123456789"}
```

### Output (JSONL)

```json
{
  "original": "Jan Kowalski mieszka w Warszawie.",
  "anonymized": "{name} {surname} mieszka w {city}.",
  "intermediate": "{name|case=nom|gender=m} {surname|case=nom|gender=m} mieszka w {city|case=loc}.",
  "synthetic": "Adam Nowak mieszka w Krakowie.",
  "entities": [
    { "text": "Jan", "type": "name", "start": 0, "end": 3, "confidence": 0.95 },
    {
      "text": "Kowalski",
      "type": "surname",
      "start": 4,
      "end": 12,
      "confidence": 0.95
    },
    {
      "text": "Warszawie",
      "type": "city",
      "start": 24,
      "end": 33,
      "confidence": 0.9
    }
  ],
  "processing_time_ms": 45.2
}
```

## Testing

```bash
# Run tests
python -m pytest tests/ -v

# With code coverage
python -m pytest tests/ --cov=anonymizer --cov-report=html

# Performance tests
python -m pytest tests/test_performance.py -v
```
