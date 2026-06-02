# Module 7 Week A — Integration Task: Domain-Shift Analysis

Apply your fine-tuned classifier (from Lab 7A, hosted on Hugging Face Hub) to the tech / entertainment news corpus and analyze the domain-shift behavior.

Full instructions: see the **Integration Task 7A guide** linked in TalentLMS.

## Quick start

```bash
pip install -r requirements.txt
cp .env.example .env       # then edit MODEL_HUB_ID
make smoke                 # CI substitute model on 5-row fixture
make apply                 # your real model on full 1,033-row tech-news corpus
```

## TODO for learner — fill these in before submitting
- **Hugging Face Hub model URL:** `https://huggingface.co/alradi/m7-app-review-sentiment`
- **Reproducibility command:** `cp .env.example .env` (set MODEL_HUB_ID), then `make apply`.
- **What the model was trained on and why we're applying it here:**

  The model is a fine-tuned DistilBERT classifier trained on mobile app reviews labeled `positive`, `neutral`, or `negative`. Reviews are short, first-person, and opinion-driven ("the app crashes", "love the new update") - where sentiment is explicit and the author is a real user expressing a direct experience with a product.
  Here we apply it to tech and entertainment news articles from sources like CNN, Wired, and Mashable. These articles are longer, written in third-person journalistic prose, and describe events rather than express personal opinions. The vocabulary overlaps — both domains discuss software, gadgets, and public reactions — but the intent and structure are fundamentally different. This is a deliberate domain-shift experiment: we want to see whether the model's confidence scores remain meaningful outside its training distribution, which surface features trigger false positives or negatives, and how severely the class balance breaks down. 
  Findings are documented in `domain-shift-analysis.md`.

## Submission

Open a PR from `integration-7a-domain-shift` into `main`. Paste the PR URL into TalentLMS → Module 7 → Integration Task 7A.

---

## License

This repository is provided for educational use only. See [LICENSE](LICENSE) for terms.

You may clone and modify this repository for personal learning and practice, and reference code you wrote here in your professional portfolio. Redistribution outside this course is not permitted.
