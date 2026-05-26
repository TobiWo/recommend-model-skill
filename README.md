# recommend-model-skill

Copilot CLI skill that recommends the most cost-effective AI model for a given task. Returns a single structured recommendation block and does not execute tasks.

## Installation

1. Clone or download this repository.
2. Copy the skill folder to your Copilot skills directory: `cp -r recommend-model ~/.copilot/skills/`
3. Restart Copilot/CLI or reload skills with `/skill reload` to apply the change.

## Usage

/recommend-model "Short description of the task"

Output: Recommended Model, Primary Metric, Cost Estimate, Fallback, Rationale.
