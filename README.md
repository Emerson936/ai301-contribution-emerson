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

1. Open tests/test_callbacks.py and analyze the existing integration test for TensorBoard (@pytest.mark.slow). Note how it creates a miniature KSinFlowMatchingModule and monitors file creation.
2. Search the entire test suite for any instance where WandbLogger is initialized inside a real, running PyTorch Lightning Trainer block to see if end-to-end telemetry file writing is verified.
3. There are zero integration tests exercising the WandbLogger execution paths of our _log_figure dispatch helper. The only test coverage for W&B is restricted to unit tests using isolated mock objects (unittest.mock), meaning the actual file-writing and integration layers are entirely unchecked.

### Reproduction Evidence

- **Commit showing reproduction:** [[Link to commit in your fork]](https://github.com/Emerson936/synth-setter)
- **My findings:** After analyzing src/utils/callbacks.py, I confirmed that _log_figure contains explicit conditional routing specifically for WandbLogger. However, because the test suite lacks a test that sets WANDB_MODE=offline, there is currently no safety net preventing a future PR from breaking the tracking system's interaction with real W&B internal telemetry files.

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
1. Read `src/utils/callbacks.py` to understand `_log_figure`'s W&B branch exactly
2. Read the existing `tests/test_callbacks.py` to find the TB integration test and use it as a template
3. Find `KSinFlowMatchingModule` and `PlotLossPerTimestep` to understand their minimal config
4. Write a new test function in `tests/test_callbacks.py`
5. Run `make test-fast` to make sure nothing is broken, then `make test-full-cpu` to run the new slow test
6. Run `make format` before commiting

**Implement:** https://github.com/Emerson936/synth-setter

**Review:** *Need to check this with their standards

**Evaluate:** Run pytest tests/test_callbacks.py -k test_wandb_figure_logging_integration -m slow locally with Wi-Fi turned off to ensure it passes successfully and strictly offline.

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: Verify `_log_figure` dispatches to WandbLogger
- [ ] Test case 2:
- [ ] Test case 3: 

### Integration Tests

- [ ] Integration scenario 1: Test if Plots are saved locally in offline mode
- [ ] Integration scenario 2

### Manual Testing

(Will update this)

---

## Implementation Notes

### Week 3 Progress

**What I built**
- Fixes in `callback.py` (explained in challenges)
- New integration test in `test_callbacks.py`
    - Runs a training loop (one epoch, tiny dataset, CPU) with a real W&B logger attached in “offline” mode
    - Looks inside W&B’s local output folder for any PNG file whose name starts with _plot. If it exists the test passes
 
**Challenges faced**
- `PlotLossPerTimestep._compute_losses` unpacked the batch with 3 items.
- Collate function adds a 4th item, Python would crash trying to unpack 4 things into 3 variables
- Fixed it by adding `*` to the list of items (python’s way of adding an extra spot but ignoring it when used)

**Commits**
- No commits just yet: want to do more testing and check if requirements/asks are met

### Code Changes

- **Files modified:** callback.py, test_callbacks.py
- **Key commits:** [Links to important commits]
- **Approach decisions:** Took some time to understand the code itself to help get a bigger picture of what I am doing


### Week 4 Progress

**What I built**
- Subclass `WandbLogger` with a think wrapper that record the step from each `log_image` call, then after trainer.fit() assert that the recorded step equals `trainer.global_step`
 
**Challenges faced**
- `Fast_dev_run=True` suppresses logging. This means we cannot use it even though the developer asks for it in the issue. Using `max_epochs=1` works. This will need to be talked with dev.

**Commits**
- Committed the code from last week. 2/4 endpoints are done, and working on the other half:
    - Need to use `max_epochs=1`
    - Need to read the step back from the offline file (This is actually the code written for the “What I built” section)

### Code Changes

- **Files modified:** callback.py, test_callbacks.py
- **Key commits:** [Links to important commits]
- **Approach decisions:** Took some time to understand the code itself to help get a bigger picture of what I am doing

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
