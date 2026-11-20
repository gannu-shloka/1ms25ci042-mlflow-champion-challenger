# Champion–Challenger Model Comparison with Hugging Face and MLflow

**Name / USN:** [FILL: your name and USN]

## 1. Task and models

- **Task:** Sentiment classification (binary: POSITIVE / NEGATIVE)
- **Dataset:** GLUE SST-2 validation set, random sample of **[FILL: 100]** sentences (seed 42), saved in `data/eval_inputs.csv`
- **Model A (current Champion):** `distilbert-base-uncased-finetuned-sst-2-english` (~67M parameters)
- **Model B (Challenger):** `siebert/sentiment-roberta-large-english` (~355M parameters)
- **New data for the final step:** 10 sentences I wrote myself, in `data/new_data.csv` (not part of the evaluation set)

## 2. Champion rule (written BEFORE the final run)

*Rule written on: [FILL: date] — committed before running `evaluate.py` for the final time.*

| Part | Rule |
| --- | --- |
| Primary metric | **Macro-F1** on the shared evaluation set. **Higher is better.** |
| Minimum improvement margin | The Challenger must beat the Champion by **at least 0.02 (2 percentage points)**. With 100 examples, one example is about 1 percentage point, so a 2-point margin means the Challenger must get roughly two more examples right. This stops a lucky single example from being called an improvement. |
| Tie-breaker | If the difference is inside the margin (smaller than 0.02 either way), the **faster model** (lower average latency) is the Champion. |
| Guardrail | The Challenger is rejected if its average latency is **more than 10×** the Champion's. |
| Direction of every metric | accuracy, macro_f1, macro_precision, macro_recall: **higher is better**. avg_latency_ms, total_inference_s: **lower is better**. |

Only the primary metric, margin, tie-breaker and guardrail decide the winner. The other metrics are reported for information.

## 3. How to run

Run everything from this folder, in this order.

```bash
# 0. install (use a virtual environment if you can)
pip install -r requirements.txt

# 1. Terminal 1: start MLflow and leave it running
python -m mlflow server --host 127.0.0.1 --port 5000 --backend-store-uri sqlite:///mlflow.db

# 2. Terminal 2: edit STUDENT_USN in config.py, then:
python make_data.py                  # creates data/eval_inputs.csv (use --n 10 to debug)
python evaluate.py                   # runs both models, logs to MLflow, makes decision
python register_champion.py          # registers both models, sets champion / challenger aliases
python predict_from_mlflow.py        # loads models:/sentiment-model@champion, predicts new data

# optional stretch goal: swap the aliases and rerun WITHOUT editing the script
python register_champion.py --flip
python predict_from_mlflow.py
```

Open http://127.0.0.1:5000 to see the experiment and the Models page.

## 4. Results

*Paste the table from `outputs/results_table.md` here after the final run.*

[FILL: results table]

Plots are in `outputs/` (confusion matrices, comparison bar chart, latency chart). Cases where the models disagree are in `outputs/disagreements.csv`, with my comments for 5 of them.

## 5. Decision

*(150–250 words. Answer: Does the Challenger improve on the Champion? By how much? Is the difference meaningful given the sample size? What are the trade-offs (speed, size, errors)? Quote the numbers from the table above, and say whether the decision followed the rule in Section 2.)*

[FILL: write your own explanation after you see the results]

## 6. Limitations and declaration

- **Sample size:** only [FILL: 100] sentences, so small differences (1–2 points) may just be noise.
- **Data:** SST-2 sentences are short movie-review snippets, so results may not carry over to other kinds of text.
- **Timing:** latency was measured on [FILL: my laptop CPU / Colab] one example at a time, and will differ on other hardware.
- **Models:** both are pretrained models used as they are. No training was done.
- **Tools used:** [FILL: be honest. Example: "The code was drafted with help from Claude (an AI assistant by Anthropic). I have read it and can explain every line."]
