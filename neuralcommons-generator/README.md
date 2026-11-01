# NeuralCommons App Generator

This project turns the CVP/Ollama concept into a practical app-generation scaffold.

It includes:
- Base64 prompt files for code generation
- A Python generator script that builds a backend and frontend scaffold
- A minimal FastAPI backend example
- A minimal Next.js-style frontend example
- Provenance and dashboard scaffolding

## Folder structure

```text
neuralcommons-generator/
├── README.md
├── config.json
├── prompts/
│   ├── backend_prompt.base64
│   ├── frontend_prompt.base64
│   └── provenance_viewer_prompt.base64
├── scripts/
│   ├── generate_app.py
│   └── decode_prompts.py
├── templates/
│   ├── backend/
│   │   ├── requirements.txt
│   │   └── main.py
│   └── frontend/
│       ├── package.json
│       └── app/page.tsx
├── output/
└── generated_artifacts/
```

## Quick start

```bash
cd neuralcommons-generator
python scripts/generate_app.py
```

This creates a minimal project under `output/`.

## Notes

- The project is intentionally a scaffold and not a production deploy.
- Use `ollama serve` and a local model, for example `mistral`.
- Public CIDs are safe to publish, but gateway tokens and private keys must never be committed.

