---
title: "A Structured Guide to Learning MLflow and MLOps Fundamentals"
date: 2026-01-26
author: peng
categories: [DataScience, Development]
tags: [mlflow, mlops, machine-learning, experiment-tracking, model-management, tutorial, python]
math: false
image:
  path: assets/headers/2026-01-26-learning-mlflow-structured-guide.webp
  alt: "MLflow learning path visualization"
---

As a data scientist or ML practitioner, have you ever found yourself juggling multiple model experiments, losing track of which hyperparameters produced the best results, or struggling to reproduce your colleague's model from last month? If so, you're not alone. The challenges of managing ML experiments, tracking model versions, and maintaining reproducibility are universal pain points in the field.

This is precisely why I created a structured learning repository for MLflow—an open-source platform designed to tackle these challenges head-on. In this guide, I'll walk you through a systematic approach to learning MLflow and MLOps fundamentals, sharing insights from my own learning journey and providing a clear roadmap for mastering these essential skills.

---

## What is MLflow?

MLflow is an open-source platform for managing the end-to-end machine learning lifecycle. It addresses three critical pain points that most data scientists encounter:

1. **Experiment Tracking**: How do you keep track of hundreds of experiments with different hyperparameters, datasets, and model architectures?
2. **Model Management**: How do you version, organize, and deploy models in a reproducible way?
3. **Collaboration**: How do you share experiments and models with your team effectively?

### Core Components

MLflow consists of four primary components:

- **MLflow Tracking**: Log parameters, metrics, artifacts, and models during training
- **MLflow Projects**: Package ML code in a reusable, reproducible format
- **MLflow Models**: Deploy models to diverse serving environments
- **MLflow Model Registry**: Centralized model store for versioning and lifecycle management

### Key Concepts

Before diving into the hands-on learning, it's essential to understand these fundamental concepts:

- **Experiment**: A collection of runs for a specific task (e.g., "Iris Classification Model Tuning")
- **Run**: A single execution of your code containing parameters, metrics, artifacts, and models
- **Parameters**: Input values that don't change during a run (e.g., learning_rate=0.01)
- **Metrics**: Numeric values that track model performance (e.g., accuracy, loss)
- **Artifacts**: Files generated during a run (plots, models, datasets)
- **Models**: Trained models with input/output schemas for validation

### Why Systematic Learning Matters

MLflow has a rich feature set, and jumping in without structure can be overwhelming. A progressive learning approach—starting with fundamentals and building up to advanced concepts—ensures you develop a solid foundation before tackling complex use cases like GenAI evaluation or production deployment.

---

## The Learning Repository Structure

I've organized the [mlflow-learning repository](https://github.com/pengguanya/mlflow-learning) into four progressive phases, each building on the previous one:

```
mlflow-learning/
├── 01-fundamentals/          # Core tracking concepts
│   ├── 01-basic-tracking.py
│   ├── 02-autolog.py
│   ├── 03-step-by-step.py
│   └── 04-batch-training.py
│
├── 02-model-registry/        # Model versioning (planned)
│
├── 03-evaluation/            # Model evaluation framework
│   ├── 01-basic-evaluation.py
│   └── 02-advanced-evaluation.py
│
└── 04-genai-evaluation/      # LLM evaluation
    ├── 01-eval-quickstart.py
    ├── 02-eval-portkey.py
    ├── 03-compare-prompts.py
    └── 04-compare-prompts-portkey.py
```

This structure follows a deliberate progression: **Fundamentals → Registry → Evaluation → GenAI**. Each phase introduces new concepts while reinforcing previous learning.

### Getting Started Quickly

Setting up the environment is straightforward. The repository supports both modern (`uv`) and traditional (`pip`) package managers:

```bash
# Clone the repository
git clone https://github.com/pengguanya/mlflow-learning.git
cd mlflow-learning

# Setup with uv (recommended, faster)
uv python install 3.13
uv sync --python 3.13

# Or use traditional pip/venv
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# Configure environment variables
cp .env.example .env
# Edit .env to add your API keys

# Start MLflow UI
mlflow ui --port 5000
# Open http://localhost:5000 in your browser
```

The MLflow UI is your visualization dashboard where you'll see all experiments, compare runs, and analyze model performance. Keep it running in a separate terminal as you work through the examples.

---

## Phase 1: MLflow Fundamentals

The fundamentals phase covers the essential skills every MLflow user needs to master. This is where you'll spend most of your initial learning time, and it forms the foundation for everything that follows.

### 1. Basic Tracking: Logging Parameters, Metrics, and Models

The first script (`01-basic-tracking.py`) introduces the core tracking workflow. Here's a simplified example based on the repository:

```python
import mlflow
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score
from mlflow.models import infer_signature

# Load and prepare data
X, y = load_iris(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# Define hyperparameters
params = {
    "solver": "lbfgs",
    "max_iter": 100,
    "random_state": 42,
}

# Train model
model = LogisticRegression(**params)
model.fit(X_train, y_train)

# Calculate metrics
y_pred = model.predict(X_test)
accuracy = accuracy_score(y_test, y_pred)

# MLflow tracking
mlflow.set_tracking_uri("http://127.0.0.1:5000")
mlflow.set_experiment("Iris Classification")

with mlflow.start_run():
    # Log parameters
    mlflow.log_params(params)

    # Log metrics
    mlflow.log_metric("accuracy", accuracy)

    # Log model with signature
    signature = infer_signature(X_train, model.predict(X_train))
    mlflow.sklearn.log_model(
        sk_model=model,
        artifact_path="model",
        signature=signature,
        input_example=X_train[:5]
    )
```

**What you'll learn**:
- How to start an MLflow run
- Logging parameters vs metrics (and why the distinction matters)
- Model signatures for input/output validation
- Viewing results in the MLflow UI

**Key takeaway**: The `with mlflow.start_run()` context manager ensures all logging happens within a single run. Parameters are logged once, while metrics can be logged multiple times at different steps.

### 2. Automatic Logging with autolog()

Manual logging is great for learning, but MLflow offers automatic logging for popular frameworks. The `02-autolog.py` script demonstrates this powerful feature:

```python
import mlflow

# Enable autologging for sklearn
mlflow.sklearn.autolog()

# Train your model as usual
model = LogisticRegression(**params)
model.fit(X_train, y_train)
# MLflow automatically logs params, metrics, and the model!
```

**What you'll learn**:
- How `mlflow.autolog()` captures parameters, metrics, and artifacts automatically
- Which frameworks support autologging (sklearn, TensorFlow, PyTorch, XGBoost, etc.)
- When to use autolog vs manual logging

**Key takeaway**: Autologging is perfect for rapid experimentation, but manual logging gives you finer control over what gets tracked.

### 3. Step-by-Step Tracking for Training Loops

The `03-step-by-step.py` script introduces temporal tracking—logging metrics at different training steps or epochs:

```python
with mlflow.start_run():
    mlflow.log_params(params)

    for epoch in range(num_epochs):
        # Training logic here
        train_loss = train_one_epoch(model, train_loader)
        val_loss = validate(model, val_loader)

        # Log metrics at each step
        mlflow.log_metric("train_loss", train_loss, step=epoch)
        mlflow.log_metric("val_loss", val_loss, step=epoch)
```

**What you'll learn**:
- Using the `step` parameter to create learning curves
- Visualizing training progress over time in MLflow UI
- Comparing convergence rates across experiments

**Key takeaway**: Step-based logging is essential for debugging training issues and understanding model convergence behavior.

### 4. Batch Experiments: Hyperparameter Tuning

The final fundamentals script (`04-batch-training.py`) shows how to run multiple experiments systematically:

```python
import mlflow

mlflow.set_experiment("Hyperparameter Tuning")

# Grid search over hyperparameters
for solver in ['lbfgs', 'sag', 'saga']:
    for max_iter in [50, 100, 200]:
        with mlflow.start_run():
            params = {"solver": solver, "max_iter": max_iter}
            mlflow.log_params(params)

            model = LogisticRegression(**params)
            model.fit(X_train, y_train)

            accuracy = model.score(X_test, y_test)
            mlflow.log_metric("accuracy", accuracy)
```

**What you'll learn**:
- Running multiple experiments in a loop
- Using MLflow's comparison view to find best hyperparameters
- Organizing experiments hierarchically with nested runs

**Key takeaway**: Batch experiments transform ad-hoc tuning into a reproducible, analyzable process. The MLflow UI's parallel coordinates plot makes it easy to visualize how different parameters affect performance.

### Estimated Time for Phase 1

Budget **30-60 minutes** to work through these four scripts. Take time to experiment with different parameters and explore the MLflow UI—hands-on practice is essential.

---

## Phase 2: Model Evaluation

While Phase 1 focused on tracking training, Phase 2 introduces MLflow's evaluation framework for systematic model assessment.

### Using mlflow.evaluate()

The `03-evaluation/01-basic-evaluation.py` script demonstrates the `mlflow.evaluate()` function:

```python
import mlflow

# Assume model is already trained and logged
logged_model_uri = "runs:/abc123/model"

# Evaluate on test set
results = mlflow.evaluate(
    model=logged_model_uri,
    data=test_data,
    targets="target_column",
    model_type="classifier",
    evaluators=["default"]
)

print(f"Accuracy: {results.metrics['accuracy_score']}")
print(f"F1 Score: {results.metrics['f1_score']}")
```

**What you'll learn**:
- Built-in metrics for classification and regression tasks
- Automatic generation of evaluation artifacts (confusion matrices, ROC curves)
- Comparing models using standardized evaluation criteria

### Custom Evaluators

The advanced evaluation script (`02-advanced-evaluation.py`) shows how to define custom metrics:

```python
from mlflow.models import make_metric

def custom_metric(eval_df, builtin_metrics):
    # Your custom scoring logic
    return score

custom_eval_metric = make_metric(
    eval_fn=custom_metric,
    greater_is_better=True,
    name="custom_score"
)

results = mlflow.evaluate(
    model=model_uri,
    data=test_data,
    targets="target",
    extra_metrics=[custom_eval_metric]
)
```

**What you'll learn**:
- Creating domain-specific evaluation metrics
- Setting metric thresholds for model acceptance
- Integrating custom evaluators into MLflow workflows

**Key takeaway**: Evaluation shouldn't be an afterthought. The `mlflow.evaluate()` framework standardizes assessment and makes it easy to compare models objectively.

### Estimated Time for Phase 2

Budget **45 minutes** for the evaluation phase. Focus on understanding how evaluation differs from training metrics and when to use built-in vs custom evaluators.

---

## Phase 3: GenAI Evaluation

Evaluating generative AI models—especially large language models (LLMs)—requires different approaches than traditional ML. This phase introduces the LLM-as-judge paradigm and enterprise integration patterns.

### LLM-as-Judge Evaluation

The `04-genai-evaluation/01-eval-quickstart.py` script demonstrates using an LLM to evaluate another LLM's responses:

```python
import mlflow

# Evaluation dataset with prompts and expected responses
eval_data = pd.DataFrame({
    "prompt": ["What is the capital of France?", ...],
    "expected_response": ["Paris", ...]
})

# Evaluate using LLM-as-judge
results = mlflow.evaluate(
    model=llm_model_uri,
    data=eval_data,
    model_type="question-answering",
    evaluators=["default"],
    evaluator_config={
        "judge_model": "gpt-4",
        "criteria": ["correctness", "relevance", "coherence"]
    }
)
```

**What you'll learn**:
- Why traditional metrics (BLEU, ROUGE) often fail for LLMs
- Using more capable models to judge less capable ones
- Defining evaluation criteria and guidelines

### A/B Testing Prompts Systematically

The `03-compare-prompts.py` script shows how to systematically compare prompt variations:

```python
import mlflow

prompts = {
    "v1": "Answer concisely: {question}",
    "v2": "You are a helpful assistant. {question}",
    "v3": "Think step-by-step, then answer: {question}"
}

mlflow.set_experiment("Prompt Engineering")

for version, prompt_template in prompts.items():
    with mlflow.start_run(run_name=version):
        # Log prompt as parameter
        mlflow.log_param("prompt_template", prompt_template)

        # Generate responses and evaluate
        results = evaluate_prompt(prompt_template, eval_data)
        mlflow.log_metrics(results)
```

**What you'll learn**:
- Treating prompts as hyperparameters
- Quantifying prompt quality with metrics
- Running A/B/C tests on prompt variations

### Enterprise Integration with Portkey

For enterprise environments, direct OpenAI/Anthropic API access may not be available. The `02-eval-portkey.py` and `04-compare-prompts-portkey.py` scripts demonstrate using an API gateway:

```python
import os
from openai import OpenAI

# Configure Portkey gateway
client = OpenAI(
    base_url=os.getenv("PORTKEY_GATEWAY_URL"),
    api_key=os.getenv("PORTKEY_API_KEY"),
    default_headers={
        "x-azure-deployment": os.getenv("AZURE_OPENAI_DEPLOYMENT")
    }
)

# Use as normal OpenAI client
response = client.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "Hello"}]
)
```

**What you'll learn**:
- Routing through enterprise gateways (Portkey, Azure API Management)
- Managing API keys and deployments
- Maintaining local dev/enterprise prod parity

### Why GenAI Evaluation is Different

Traditional ML evaluation focuses on quantitative metrics computed deterministically. GenAI evaluation requires:

- **Subjective criteria**: Relevance, coherence, and helpfulness can't be reduced to a single number
- **Human-in-the-loop**: LLM-as-judge approximates human judgment but doesn't replace it
- **Prompt sensitivity**: Small wording changes can dramatically affect outputs
- **Stochastic outputs**: Temperature > 0 means different runs produce different results

**Key takeaway**: GenAI evaluation is more exploratory and qualitative than traditional ML. MLflow provides structure to what would otherwise be ad-hoc experimentation.

### Estimated Time for Phase 3

Budget **1-2 hours** for GenAI evaluation. This phase requires setup (API keys, gateway configuration) and benefits from iterative experimentation.

---

## Learning Path and Best Practices

### Recommended Progression

1. **Start with Fundamentals (Phase 1)**: Even if you're experienced in ML, don't skip this. Understanding basic tracking patterns is essential.

2. **Explore the MLflow UI**: After each script, spend time in the UI comparing runs, viewing artifacts, and exploring the search functionality.

3. **Modify and Experiment**: Change hyperparameters, try different models, and see how metrics change. Learning by doing is far more effective than passive reading.

4. **Move to Evaluation (Phase 2)**: Once comfortable with tracking, learn to evaluate systematically.

5. **Tackle GenAI (Phase 3)**: If working with LLMs, this phase is essential. Otherwise, you can skip it.

### Tips for Returning After a Break

I designed this repository to be "returnable"—easy to pick up even after months away. If you're coming back:

1. **Re-read the README**: It's your anchor point and refresher
2. **Re-run Fundamentals**: Start with `01-basic-tracking.py` to jog your memory
3. **Check `.env` file**: Ensure API keys are still valid
4. **Review Past Runs**: The MLflow UI shows your historical experiments

### Common Pitfalls to Avoid

- **Forgetting to start MLflow UI**: You need `mlflow ui --port 5000` running to visualize experiments
- **Not logging enough context**: Future you will thank present you for logging more parameters and tags
- **Inconsistent experiment naming**: Use descriptive names like "bert-sentiment-analysis" not "exp1"
- **Skipping model signatures**: Signatures enable type checking and prevent deployment errors
- **Ignoring artifacts**: Log plots, confusion matrices, and feature importance—visual artifacts are invaluable for debugging

---

## Practical Tips for Your Own Projects

### Integration with Existing Workflows

MLflow integrates seamlessly with existing ML workflows:

```python
# Wrap your existing training script
def train_model(params):
    with mlflow.start_run():
        mlflow.log_params(params)
        model = train(params)  # Your existing code
        metrics = evaluate(model)  # Your existing code
        mlflow.log_metrics(metrics)
        mlflow.sklearn.log_model(model, "model")
    return model
```

No need to rewrite everything—add MLflow calls incrementally.

### Scaling from Local to Production

The repository uses local `mlruns/` directory for simplicity, but MLflow supports remote tracking servers:

```python
# Local development
mlflow.set_tracking_uri("file:///path/to/mlruns")

# Shared team server
mlflow.set_tracking_uri("http://mlflow-server:5000")

# Cloud-hosted
mlflow.set_tracking_uri("databricks")
```

### Version Control and Collaboration

**Best practices**:
- Commit your MLflow tracking code alongside model code
- Use `.mlflowignore` (similar to `.gitignore`) to exclude large artifacts from Git
- Share experiment names and run IDs in code reviews
- Tag runs with team member names or feature branches

**Example workflow**:
```python
with mlflow.start_run():
    mlflow.set_tag("developer", "alice")
    mlflow.set_tag("branch", "feature/new-architecture")
    mlflow.set_tag("jira_ticket", "DS-1234")
    # ... rest of tracking code
```

### Model Governance

As you mature your MLflow usage, consider:

- **Model Registry**: Transition models from Staging → Production
- **Model Aliases**: Tag production models with `champion` or `challenger`
- **Webhooks**: Trigger CI/CD on model registration
- **Access Controls**: Restrict who can transition models to Production

---

## Conclusion and Next Steps

Learning MLflow systematically transforms chaotic experimentation into organized, reproducible research. The structured repository I've shared provides a clear path from fundamentals through advanced GenAI evaluation, ensuring you build solid foundations before tackling complex use cases.

### Key Takeaways

1. **MLflow solves real problems**: Experiment tracking, model versioning, and collaboration challenges are universal—MLflow provides battle-tested solutions.

2. **Progressive learning works**: Following the repository's structure (Fundamentals → Evaluation → GenAI) ensures you don't get overwhelmed.

3. **Hands-on practice is essential**: Reading about MLflow isn't enough—run the scripts, modify parameters, and explore the UI.

4. **It's a returnable resource**: The repository is designed for learning and reference. Come back whenever you need a refresher.

### Start Learning Today

The best way to learn MLflow is to start using it. Here's your action plan:

1. **Clone the repository**: `git clone https://github.com/pengguanya/mlflow-learning.git`
2. **Set up your environment**: Use `uv` or `pip` to install dependencies
3. **Start the MLflow UI**: `mlflow ui --port 5000`
4. **Run your first experiment**: `python 01-fundamentals/01-basic-tracking.py`
5. **Explore the UI**: See your first logged run and explore its parameters, metrics, and artifacts

### Additional Resources

- **Official Documentation**: [mlflow.org/docs](https://mlflow.org/docs/latest/index.html)
- **GitHub Repository**: [github.com/pengguanya/mlflow-learning](https://github.com/pengguanya/mlflow-learning)
- **MLflow Slack Community**: [mlflow.org/slack](https://mlflow.org/slack)
- **MLflow GitHub**: [github.com/mlflow/mlflow](https://github.com/mlflow/mlflow)

Whether you're a data scientist struggling with experiment sprawl, an ML engineer setting up model deployment pipelines, or a researcher in biomedical domains needing reproducibility, MLflow has tools to make your life easier. The structured learning path I've shared will help you master these tools systematically.

Happy learning, and may your experiments be ever reproducible!
