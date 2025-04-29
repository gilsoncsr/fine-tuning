# 🧠 Fine-Tuning Dataset Preparation for OpenAI - Hate Speech Classification (Portuguese)

This project demonstrates how to **prepare, clean, and convert a Portuguese hate speech dataset** for **fine-tuning OpenAI language models**, using secure environment variable management via `.env` files.

---

## 📂 Project Overview

- Loads the [`hate_speech_portuguese`](https://huggingface.co/datasets/hate-speech-portuguese/hate_speech_portuguese) dataset from Hugging Face.
- Preprocesses and splits the data for training and validation.
- Converts the data into JSONL format suitable for OpenAI fine-tuning.
- Uploads the datasets using the OpenAI API.
- Supports `.env` file usage to protect your API key.

---

## ✅ Prerequisites

Make sure the following packages are installed:

```bash
pip install python-dotenv datasets openai
```

---

## 🔐 API Key Setup (Using `.env`)

To avoid exposing your `OPENAI_API_KEY`, create a `.env` file in the same directory as your notebook:

```
OPENAI_API_KEY=your_openai_api_key_here
```

The notebook uses `python-dotenv` to load the key:

```python
%pip install python-dotenv
%load_ext dotenv
%dotenv
```

---

## 🧼 Data Processing Steps

1. **Dataset Loading:**

   Loads 10% of the dataset for testing purposes:

   ```python
   datasets = load_dataset("hate-speech-portuguese/hate_speech_portuguese", split='train[:10%]')
   ```

2. **Cleaning and Filtering:**

   - Removes unnecessary annotator columns.
   - Splits into training and testing sets.
   - Replaces newline characters in text.

3. **Label Mapping:**

   Maps:
   - `0` ➝ `"No Hate Speech"`
   - `1` ➝ `"Hate Speech"`

4. **Conversion to JSONL (OpenAI Fine-tune Format):**

   Each example is structured as:

   ```json
   {
     "messages": [
       {"role": "system", "content": "Classify user comments as Hate Speech or No Hate Speech."},
       {"role": "user", "content": "<comment_text>"},
       {"role": "assistant", "content": "<label_text>"}
     ]
   }
   ```

5. **Alternate AWS Format:**

   The AWS-style format (prompt-completion pairs):

   ```json
   {
     "prompt": "<comment_text>",
     "completion": "<label_text>"
   }
   ```

---

## 📤 Upload to OpenAI

Uploads `train.jsonl` and `validation.jsonl` via the API:

```python
file_train = client.files.create(file=open("train.jsonl", "rb"), purpose="fine-tune")
file_validation = client.files.create(file=open("validation.jsonl", "rb"), purpose="fine-tune")
```

Starts the fine-tuning job:

```python
client.fine_tuning.jobs.create(
    training_file=file_train.id,
    validation_file=file_validation.id,
    model="gpt-4o-mini-2024-07-18"
)
```

> ⚠️ Note: Fine-tuning for `davinci-002` has been deprecated; newer models such as `gpt-4o-mini` are recommended.

---

## 📁 Output Files

- `train.jsonl`, `validation.jsonl` – for OpenAI fine-tuning
- `train_aws.jsonl`, `validation_aws.jsonl` – prompt-completion format

---

## 🧪 Sample Output

Example from the OpenAI-formatted JSONL:

```json
{
  "messages": [
    {"role": "system", "content": "Classify user comments as Hate Speech or No Hate Speech."},
    {"role": "user", "content": "Esse comentário é odioso."},
    {"role": "assistant", "content": "Hate Speech"}
  ]
}
```
