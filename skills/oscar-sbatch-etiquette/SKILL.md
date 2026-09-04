---
name: oscar-sbatch-etiquette
description: Use when creating, reviewing, modifying, or debugging Oscar/Slurm sbatch scripts, Slurm array jobs, experiment launchers, dry-runable HPC training or evaluation scripts, job submission workflows, or replacing sbatch --wrap commands with durable scripts. Enforce readable shell variables, observable COMMAND arrays, semantic logging metadata, direct seed/task mapping, DRY_RUN=1 output, safe shell defaults, and bash/sbatch validation.
---

# Oscar sbatch Script Etiquette

Use this skill to create or review durable Oscar Slurm scripts. Make launchers easy to audit, rerun, modify, test, and recover from logs without reverse-engineering a long command string from shell history.

## Workflow

When working on an sbatch script:

1. Identify the experiment entrypoint, resource needs, task mapping, seed semantics, and logging metadata.
2. Prefer a committed or saved script over a long `sbatch --wrap` command for durable experiment runs.
3. Put tunable settings in named shell variables, then pass those variables through a shell `COMMAND` array.
4. Keep array-task, condition, and seed mapping explicit near the top of the script.
5. Add `DRY_RUN=1` output that prints the project root, condition, seed, task id, logging metadata, and final quoted command.
6. Validate with `bash -n`, representative dry-runs, and `sbatch --test-only` before submission.

## Script Structure

Organize durable scripts in this order:

1. Slurm resource directives.
2. Safe shell defaults.
3. Project-root discovery and log-directory setup.
4. Array-task to condition/seed mapping.
5. Shared training or evaluation hyperparameters.
6. Condition-specific hyperparameters.
7. Logging metadata.
8. Command construction.
9. `DRY_RUN` output.
10. Final execution.

Use:

```bash
set -euo pipefail
```

Before executing the workload, prefer:

```bash
cd "$PROJECT_ROOT"
unset LD_LIBRARY_PATH
unset VIRTUAL_ENV
```

## Command Construction

Prefer named settings and a shell array:

```bash
TOTAL_STEPS="${TOTAL_STEPS:-20000000}"
BATCH_SIZE="${BATCH_SIZE:-256}"
LEARNING_RATE="${LEARNING_RATE:-1e-4}"
MODEL_SIZE="${MODEL_SIZE:-256 256}"
read -r -a MODEL_SIZE_ARGS <<< "$MODEL_SIZE"
SEED="${SLURM_ARRAY_TASK_ID:-0}"

COMMAND=(
    uv run python -m package.train
    --total_steps "$TOTAL_STEPS"
    --batch_size "$BATCH_SIZE"
    --learning_rate "$LEARNING_RATE"
    --model_size "${MODEL_SIZE_ARGS[@]}"
    --seed "$SEED"
)
```

Avoid making durable launchers depend on long handwritten wrappers:

```bash
sbatch --wrap="uv run python -m package.train --total_steps 20000000 --batch_size 256 --learning_rate 1e-4 ..."
```

Use direct `sbatch --wrap` only for one-off emergency runs. If the run becomes scientifically important, promote it into a readable script with explicit condition names, direct seed mapping, semantic logging metadata, `DRY_RUN=1`, and validation output.

## Condition And Seed Mapping

Keep condition logic before command construction. Each condition should set all values that differ across conditions, even if that repeats a default.

```bash
TASK_ID="${SLURM_ARRAY_TASK_ID:-0}"

if (( TASK_ID < 3 )); then
    CONDITION="baseline"
    SEED="$TASK_ID"
    LEARNING_RATE="1e-4"
    REGULARIZATION="0.0"
    MODEL_SIZE="256 256"
elif (( TASK_ID < 6 )); then
    CONDITION="regularized"
    SEED="$((TASK_ID - 3))"
    LEARNING_RATE="1e-4"
    REGULARIZATION="0.01"
    MODEL_SIZE="256 256"
else
    echo "Unknown task id: $TASK_ID" >&2
    exit 1
fi
```

Prefer one Slurm array task mapping to one direct run seed:

```bash
SEED="$TASK_ID"
COMMAND+=(--seed "$SEED")
```

If a codebase uses both a base `--seed` and a `--seed_idx`, document those semantics in the script before using them. Do not assume they are equivalent to direct seeds.

## Logging Metadata

Do not rely on Slurm job IDs as the primary meaning of logging groups or run names. Job IDs help locate logs, but they are poor scientific labels.

Use semantic groups:

```bash
LOG_GROUP="${LOG_GROUP:-project-task-ablation}"
```

Use run names that encode the condition and important switches:

```bash
RUN_NAME="${RUN_NAME:-${CONDITION}__seed${SEED}__lr-${LEARNING_RATE}__reg-${REGULARIZATION}}"
```

For factorial runs, include factor values:

```bash
RUN_NAME="${CONDITION}__seed${SEED}__factor-a-${FACTOR_A}__factor-b-${FACTOR_B}"
```

## Dry Runs

Every durable script should expose what it is about to run before it runs. Build the final command first, then quote it for logs:

```bash
printf -v FINAL_COMMAND "%q " "${COMMAND[@]}"

if [[ "${DRY_RUN:-0}" == "1" ]]; then
    echo "DRY_RUN=1"
    echo "Project root: ${PROJECT_ROOT}"
    echo "Condition: ${CONDITION}"
    echo "Seed: ${SEED}"
    echo "Task id: ${TASK_ID:-none}"
    echo "Log group: ${LOG_GROUP:-none}"
    echo "Run name: ${RUN_NAME:-none}"
    echo "Final command: ${FINAL_COMMAND}"
    exit 0
fi
```

This makes the entrypoint observable from tests, terminal dry-runs, and Slurm logs.

## Credentials

Load external credentials only when needed and only if they are not already set:

```bash
KEY_FILE="${KEY_FILE:-${PROJECT_ROOT}/secrets/service_key.txt}"
if [[ -z "${SERVICE_API_KEY:-}" && -f "$KEY_FILE" ]]; then
    SERVICE_API_KEY="$(< "$KEY_FILE")"
    export SERVICE_API_KEY
fi
```

## Validation

Every durable script should pass:

```bash
bash -n path/to/script.sh
DRY_RUN=1 SLURM_ARRAY_TASK_ID=0 bash path/to/script.sh
sbatch --test-only path/to/script.sh
```

For array scripts, dry-run at least one representative task per condition. When adding tests, assert the array range, condition mapping, seed mapping, key hyperparameter flags, logging group, and run-name pattern.

## Review Checklist

Before approving or submitting an sbatch script, verify:

- The script name describes the experiment.
- Slurm resource directives are explicit.
- The array range matches the condition and seed mapping.
- The workload entrypoint is visible in one `COMMAND` array.
- Hyperparameters are named shell variables, not buried in a long string.
- Logging group and run names are meaningful without opening Slurm logs.
- `DRY_RUN=1` prints the final command.
- `bash -n` passes.
- `sbatch --test-only` passes when Slurm is available.
