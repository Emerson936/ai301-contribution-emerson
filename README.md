# Contribution #652: test(monitoring): add W&B integration test for _log_figure dispatch (offline mode)

**Contribution Number:** 1  
**Student:** Emerson Palaganas  
**Issue:** [https://github.com/tinaudio/synth-setter/issues/652]
**Status:** Phase II [In Progress]

---

## Why I Chose This Issue

I chose "add W&B integration test for _log_figure dispatch (offline mode)" because it aligns with my background knowledge in Python as well as my experience with training AI models. The owner of the repo has stated clear goals/endpoints in the issue, making it easy to follow and understand. From reading the issue, I understand the core problem is figuring out how to "trick" W&B to run completely offline so we can verify the AI’s graphs are saving properly without an internet connection. I have already left a comment on the issue, and while waiting for a response, I am forking and cloning the repository to dive into the code and get comfortable debugging the setup.

---

## Understanding the Issue

### Problem Description

The repository is missing a real end-to-end integration test to verify that graphs are sucessfully saved when using Weights & Biases (WandbLogger). While TensorBoard has an offline integration test, W&B is only covered by shallow unit tests with mock data because it normally requires an internet connection to run.

### Expected Behavior

There should be an automated test that forces W&B into an offline mode, runs a quick 1-step fake training loop, and saves the graphs locally. The test should then peek into the local files to prove that the graph key exists and matches the correct training step.

### Current Behavior

Right now, if a future update accidentally breaks the way our plotting callbacks talk to W&B, our testing robot will not catch it. The code runs fine in unit tests, but is not being tested as a complete system.

### Affected Components

"src/utils/callbacks.py" houses the _log_figure dispatch helper and plotting the callback that we need to execute. "tests/test_callbacks.py" is the test file where the new offline testing function will be written to mirror the TensorBoard setup.

---

## Reproduction Process

### Environment Setup

The repository has a `getting-started.md` file, which is really helpful and clear on how to setup the project. The issue also has clear instructions and endpoints.

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

The root cause is a gap in test coverage: the W&B logging branch of `_log_figure` is never tested with a real, running PyTorch Lighting Trainer. Because W&B normally requires a cloud login and internet connection, it was skipped from integration testing.

### Proposed Solution

We will write an isolated integration test that forces `WandbLogger` to run in an offline environment `(WANDB_MODE=offline)`. This allows a mini-training loop to execute completely on the local machine and dump its telemetry data into a temporary directory, where we can verify that the graphs are saved.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** We need to verify that `_log_figure` successfully sends plots to W&B during a real training run without using the internet or requiring an API key.

**Match:** We can match the exact pattern of the existing TensorBoard integration test in `tests/test_callbacks.py`, which downsizes the model and uses a fast development run to test file serialization locally.

**Plan:** [Step-by-step implementation plan]
1. Add a new test function `test_wandb_figure_logging_integration` to `tests/test_callbacks.py` and decorate it with `@pytest.mark.slow`.
2. Use `monkeypatch` to set `WANDB_MODE="offline"` and initialize WandbLogger with a temporary `save_dir`.
3. Spin up a mini `Trainer` using `fast_dev_run=True`, run `trainer.fit()`, and call `wandb.finish()` to flush files to disk.
4. Write file-parsing logic to scan the generated local `wandb-history.jsonl` file and assert that the "plot" key exists and matches the trainer's `global_step`.

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
