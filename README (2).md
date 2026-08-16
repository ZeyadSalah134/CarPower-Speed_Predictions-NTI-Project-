# Deploying AUTOPOWER AI permanently (free)

## 1. Get your trained model files out of Colab
In your Colab notebook, run all cells up through the one that saves
`/content/models/*.joblib` (the "ALL STREAMLIT FILES SAVED" cell).
Then download the whole folder:

```python
from google.colab import files
import shutil
shutil.make_archive('models', 'zip', '/content/models')
files.download('models.zip')
```

Unzip it — you should have:
```
models/
  best_model.joblib
  dataset.joblib
  results_df.joblib
  target_col.joblib
  feature_importance.joblib
```
(Skip `trained_models.joblib` — the app only ever loads `best_model.joblib`,
and that file alone can be 3-4x smaller since it drops the other 3 unused
trained pipelines. Leaving it out makes the repo lighter and the app start faster.)

## 2. Put it in a GitHub repo
Create a new repo with this structure:
```
your-repo/
  app.py                <- included here
  requirements.txt      <- included here
  models/
    best_model.joblib
    dataset.joblib
    results_df.joblib
    target_col.joblib
    feature_importance.joblib
```

## 3. Deploy on Streamlit Community Cloud (free, permanent URL)
1. Go to https://share.streamlit.io and sign in with GitHub.
2. Click "New app" → pick your repo, branch `main`, main file `app.py`.
3. Click "Deploy".

You'll get a permanent link like `https://your-app-name.streamlit.app`
that works even when your computer and Colab are both off. If nobody visits
it for a while it goes to sleep, but it wakes itself up automatically the
next time someone opens the link — no need to re-run anything.

Alternative if you prefer: Hugging Face Spaces (choose the "Streamlit" SDK
when creating a Space) — same idea, also free and permanent.

## Making it faster
- The app already caches the model and dataset load with
  `@st.cache_resource` / `@st.cache_data`, which is the main lever — don't
  remove those decorators.
- Only ship `best_model.joblib`, not all four trained pipelines (see step 1)
  — smaller file = faster cold start.
- Community Cloud/HF Spaces run on a dedicated small server, which is
  usually more consistent than a shared Colab CPU + a tunnel hop through
  cloudflare — so this alone should feel snappier than the current setup.
- If it's still slow, the biggest win is shrinking `dataset.joblib`
  (drop columns the app's UI doesn't actually use, e.g. anything shown
  only during EDA in the notebook but never referenced in `app.py`).
