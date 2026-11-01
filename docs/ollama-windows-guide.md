# Ollama på Windows: Import, fine-tuning og direkte bruk av CVP-data

Dette er en praktisk guide for å bruke Ollama på Windows til å:

1. Importere treningsdata, inkludert CVP-baserte datasett
2. Bygge en AI/LLM med fine-tuning
3. Peste inn data direkte i modellen

---

## 1. Installer Ollama på Windows

### 1.1 Last ned og installer

1. Gå til den offisielle Ollama-siden:
   https://ollama.com/
2. Last ned Windows-installeren (.exe).
3. Kjør installasjonen.
4. Sørg for at Windows 10/11 er oppdatert.

### 1.2 Start serveren

Åpne PowerShell som administrator og kjør:

```powershell
ollama serve
```

La vinduet stå åpent. Serveren kjører i bakgrunnen.

### 1.3 Verifiser installasjon

```powershell
ollama --version
```

---

## 2. Last ned en base-modell

Noen gode valg:

- `mistral`
- `llama2`
- `phi`

Eksempel:

```powershell
ollama pull mistral
```

Sjekk modeller:

```powershell
ollama list
```

---

## 3. Forbered treningsdata for Ollama

Ollama bruker JSONL-format for fine-tuning.

### 3.1 Eksempel: `train_data.jsonl`

```json
{"prompt": "Hva er Commons Vector Protocol?", "response": "CVP er et IPFS-basert provenance-rammeverk for å verifisere data via tri-vector addressing og Ed25519-signaturer."}
{"prompt": "Hvordan fungerer Det Levende Arkivet?", "response": "Det Levende Arkivet er en historisk database der alle endringer lagres som CVP-instrumenter med provenance-chain."}
{"prompt": "Hva er TRUST-ORDBOK?", "response": "TRUST-ORDBOK er en semantisk ordbok for CVP som definerer termer som tri_vector, spatial_vector, og rule_vector."}
```

Lagre filen i for eksempel:

```text
C:\ollama\train_data.jsonl
```

### 3.2 Konverter CVP-data til JSONL

```python
import json


def cvp_to_ollama_jsonl(cvp_file, output_file="train_data.jsonl"):
    with open(cvp_file, "r", encoding="utf-8") as f:
        instrument = json.load(f)

    data = []
    if "cid" in instrument:
        pass
    else:
        source_file = instrument["metadata"]["source"]
        with open(source_file, "r", encoding="utf-8") as f:
            data = json.load(f)

    with open(output_file, "w", encoding="utf-8") as f:
        for item in data:
            f.write(json.dumps({
                "prompt": "",
                "response": item["text"]
            }, ensure_ascii=False) + "\n")


if __name__ == "__main__":
    cvp_to_ollama_jsonl("instruments/20261101120000.car.json", "train_data.jsonl")
```

Kjør:

```powershell
python convert_cvp_to_ollama.py
```

---

## 4. Fine-tune en modell med LoRA

Ollama har begrenset støtte for direkte fine-tuning, så mange bruker Axolotl for avansert trening.

### 4.1 Installer Axolotl

```powershell
git clone https://github.com/OpenAccess-AI-Collective/axolotl
cd axolotl
pip install -r requirements.txt
```

### 4.2 Opprett konfigurasjon

Lag `fine-tune-mistral.yml`:

```yaml
base_model: mistral
model_type: Mistral
output_dir: ./output

train_file: C:\ollama\train_data.jsonl
val_file: null

lora:
  r: 16
  lora_alpha: 32
  lora_dropout: 0.05
  bias: none
  task_type: CAUSAL_LM

batch_size: 4
micro_batch_size: 4
num_epochs: 3
learning_rate: 3e-4
cutoff_len: 2048
val_set_size: 0.1
save_steps: 100
save_total_limit: 2

device: cuda
use_fp16: true
```

### 4.3 Kjør fine-tuning

```powershell
accelerate launch -m axolotl.cli.train fine-tune-mistral.yml
```

### 4.4 Eksporter til Ollama-format

```powershell
ollama create neuralcommons-mistral -f ./output
```

---

## 5. Bruk den trenede modellen

### 5.1 Kjør modellen

```powershell
ollama run neuralcommons-mistral
```

### 5.2 Test prompt

```powershell
echo "Hva er CVP?" | ollama run neuralcommons-mistral
```

### 5.3 Peste data direkte i modellen

```powershell
curl http://localhost:11434/api/generate -d '{
  "model": "neuralcommons-mistral",
  "prompt": "Basert på følgende data, forklar CVP:\n\nCommons Vector Protocol (CVP) bruker tri-vector addressing (tri_vector, spatial_vector, rule_vector) for å definere kontekst. Hvert instrument er signert med Ed25519 og lagret på IPFS med en CID.",
  "stream": false
}'
```

---

## 6. Viktige praksisregler

- Offentlig CID er OK å publisere.
- Gateway-token og private credentials bør ikke legges i repoet.
- Hvis token fortsatt er aktivt, bør det roteres eller revokeres før repoet gjøres offentlig.
- CVP er et retningsgivende rammeverk, ikke en juridisk autoritet.

---

## 7. Kort oppsummering

For å bruke Ollama med CVP-data:

1. Installer Ollama på Windows
2. Last ned en base-modell som `mistral`
3. Forbered JSONL-data
4. Finetune via Axolotl eller lignende
5. Kjør modellen med `ollama run`
6. Bruk `curl` eller direkte prompts for å teste og iterere

---

Dette er en praktisk, teknisk oppskrift. For prosjektets teoriske og protokollmessige grunnlag, se hoveddokumentasjonen i [README.md](../README.md) og under [protocol/](../protocol/).
