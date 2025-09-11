## M2E2 Prompt-based Extraction (Text + Image)

This project reproduces parts of the M2E2 pipeline using prompt-based LLMs in a Jupyter notebook (`test.ipynb`). It:
- Loads an article and finds its corresponding images
- Organizes outputs per-article under `output/<base_name>/`
- Extracts structured events from text (LLM) and images (VLM)
- Converts model output to strict JSON, then appends to a single JSON file
- Optionally aligns text and image events via embedding similarity and appends “multimedia” events

### Repository structure
- `m2e2_rawdata/`
  - `article/` — input `.txt` articles (e.g., `VOA_EN_NW_2009.12.09.416313.rsd.txt`)
  - `image/image/` — input `.jpg` images
- `output/<base_name>/`
  - `articles/` — copied source article
  - `images/` — copied corresponding images
  - `OUTPUT_<base_name>.json` — cumulative JSON for text, image, and multimedia events
  - `IMAGE_OUTPUT_<base_name>.json` — optional image-only events JSON (if you save separately)
- `test.ipynb` — main notebook with end‑to‑end cells

### Environment
- Python 3.9+
- Key packages:
  - `transformers`, `torch`, `sentence-transformers`, `Pillow`

Install (example):
```bash
pip install transformers torch sentence-transformers pillow
```

### Workflow (Notebook)
1) Select article and prepare output folders
- Set `article_filename` in the first code cell.
- The cell copies the article and matching images into `output/<base_name>/articles` and `output/<base_name>/images`, and creates an empty `OUTPUT_<base_name>.json`.

2) Text-only extraction (LLM)
- Runs a prompt against a Gemma instruction-tuned model.
- The raw model output may include markdown fences and single quotes; the conversion cell cleans and parses to strict JSON.
- The save/append cell writes JSON to `OUTPUT_<base_name>.json` (read-modify-write) so repeated runs accumulate results.

3) Image-only extraction (VLM)
- Uses Gemma 3 VLM with `AutoProcessor`/`AutoModelForCausalLM`.
- Ensure the prompt includes Gemma’s image tokens (`<start_of_image><end_of_image>`) for each passed image.
- Parse and save image JSON. You can:
  - Append directly into `OUTPUT_<base_name>.json`, or
  - Save to `IMAGE_OUTPUT_<base_name>.json` first, then merge later.

4) Multimedia alignment (Text+Image)
- Functions: `comp_similarity`, `align_events`.
- Builds sentence and image event summaries, encodes with `sentence-transformers`, computes cosine similarity, and appends matches (with a similarity score) into `OUTPUT_<base_name>.json`.



