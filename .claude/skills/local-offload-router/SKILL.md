---
name: local-offload-router
description: Decide whether an operator's AI workflow is eligible to run on a self-hosted open-weights model (high-volume, low-sensitivity, latency-tolerant, quality-tolerant) or should stay on a hosted API (high-stakes, sensitive data, top quality), then emit a routing decision, a copy-paste config stub for the local endpoint, and an API fallback rule. Provider-agnostic and copy-paste runnable. Built for operators at enough volume that the per-token API bill is a real tax and a near-frontier open-weights model (e.g. NVIDIA Nemotron 3 Ultra) makes self-hosting math flip. Use when you say /local-offload-router, "should I self-host this", "route this job to my own box", "build me an offload router", or hands over a workflow and wants to know if it can run local with an API fallback.
when_to_use: When an operator running high-volume AI work wants to move eligible jobs off a per-token API and onto a self-hosted open-weights model to kill marginal cost, while keeping the hard or sensitive work on a hosted API. Takes a workflow (or a list) and returns an eligibility decision, the local endpoint config stub, and the API fallback rule. The self-hosting companion to [[volume-lane-router]] → volume-lane routes between model classes; this routes between your own box and a hosted API.
allowed-tools:
  - Read
  - Write
  - Grep
  - Glob
  - AskUserQuestion
---

# Local Offload Router

Take a workflow and decide one thing: does this job run on **your own box** against a self-hosted open-weights model, or does it stay on a **hosted API**? Then emit the routing decision, a config stub for the local endpoint, and an API fallback rule so a local failure never strands the job.

The point is not "self-host everything." Self-hosting has real overhead → GPU rental or hardware, an inference server, ops, security, a quality gap against the frontier. This skill is for operators at enough volume that the per-token API bill is a tax, and a near-frontier open-weights model (e.g. NVIDIA Nemotron 3 Ultra, ~550B params / ~55B active, 1M context, open license) makes the math flip on the *repetitive, non-sensitive* slice of their work.

So you build a router: eligible jobs go local at near-zero marginal cost, everything else stays on the API. Provider-agnostic → it reasons in eligibility, not brand names, so it survives the next model launch and the next price cut.

## The eligibility checklist (all four must pass to go local)

A job is **local-eligible** only if it passes all four. One fail → it stays on the API.

1. **High-volume.** The job runs at a count where per-token API cost is a real line item, not a rounding error. Bulk tagging, classification, first-pass summaries, extraction, PDP variant drafts, routing. If you run it a handful of times a week, the API is fine → self-hosting overhead won't pay back.
2. **Low-sensitivity.** No regulated data, no client PII, no secrets, no anything you'd be uncomfortable processing on a box you patch yourself. Self-hosting moves the security burden onto you. Only send data you'd be fine owning the breach surface for.
3. **Latency-tolerant.** The job can wait. Batch overnight, queue-and-drain, async pipelines. If a human is staring at a spinner waiting for the answer, a self-hosted box (especially a cold or shared one) is the wrong place for it.
4. **Quality-tolerant.** "Good enough" is genuinely good enough here, and a wrong answer is cheap to catch or cheap to ignore. The open-weights model is near-frontier, not frontier → there's a measurable gap on the hard stuff. Only offload work where that gap doesn't change the outcome.

If a job fails any line, it is **API-only**. The clearest API-only signals: high-stakes (a wrong answer costs real money, trust, or a decision), sensitive data, needs top quality, or a human is waiting on it live.

## The routing decision

For each workflow, output exactly this:

```
WORKFLOW: <name>
VOLUME: <high / low>            → fail if low
SENSITIVITY: <low / high>       → fail if high
LATENCY: <tolerant / live>      → fail if live
QUALITY: <tolerant / top>       → fail if top
DECISION: LOCAL  (all four pass)   |   API  (one or more fail)
REASON: <one line: which check decided it>
FALLBACK: <the API model + the exact trigger that bumps a LOCAL job to the API>
```

A LOCAL decision always carries a fallback. A self-hosted box is never the only path → if it's down, slow, or the output fails a check, the job spills to the API. The fallback is what makes offloading safe instead of fragile.

## The local endpoint config stub (copy-paste, provider-agnostic)

Self-hosted open-weights models serve an **OpenAI-compatible** `/v1/chat/completions` endpoint (vLLM, NVIDIA NIM, TGI, llama.cpp server, Ollama all do this). So your code points at a local base URL instead of a vendor's, with the same request shape. Swap the model id and base URL for whatever you run.

```python
# local_offload.py → route eligible jobs to a self-hosted endpoint, fall back to API.
import os
from openai import OpenAI

# Your self-hosted open-weights endpoint (vLLM / NIM / TGI / Ollama → all OpenAI-compatible).
LOCAL = OpenAI(
    base_url=os.environ.get("LOCAL_BASE_URL", "http://localhost:8000/v1"),
    api_key=os.environ.get("LOCAL_API_KEY", "not-needed"),  # local servers often ignore this
)
LOCAL_MODEL = os.environ.get("LOCAL_MODEL", "nvidia/nemotron-3-ultra")  # whatever you serve

# Your hosted API fallback (any provider with an OpenAI-compatible client).
API = OpenAI(api_key=os.environ["API_KEY"])
API_MODEL = os.environ.get("API_MODEL", "your-hosted-model")

def run(messages, *, eligible: bool, quality_check=None, timeout=30):
    """eligible: did the job pass all four checklist items?
    quality_check: optional fn(text)->bool, returns True if the local output is acceptable."""
    if eligible:
        try:
            out = LOCAL.chat.completions.create(
                model=LOCAL_MODEL, messages=messages, timeout=timeout,
            ).choices[0].message.content
            # Fallback trigger: local errored above, OR output fails the quality gate.
            if quality_check is None or quality_check(out):
                return {"source": "local", "text": out}
        except Exception:
            pass  # spill to API on any local failure / timeout
    # API path: ineligible jobs, local outages, or failed quality gate.
    out = API.chat.completions.create(
        model=API_MODEL, messages=messages,
    ).choices[0].message.content
    return {"source": "api", "text": out}
```

Server stub (one line to stand up an OpenAI-compatible local endpoint with vLLM):

```bash
# Serve an open-weights model on your own box; exposes http://localhost:8000/v1
vllm serve nvidia/nemotron-3-ultra --host 0.0.0.0 --port 8000
# (NIM, TGI, Ollama `ollama serve`, or llama.cpp `--server` all expose the same /v1 shape.)
```

## The API fallback rule

Every LOCAL job inherits one fallback rule. State it explicitly so it's not implicit hope:

> If the local endpoint errors, times out (> N seconds), or the output fails its quality gate, re-run the job on `<API_MODEL>`. Log every spill. If spill rate climbs above a threshold (e.g. >10% of runs), the job is no longer truly local-eligible → revisit the checklist.

Spill rate is the health metric. A LOCAL job that quietly spills half its runs to the API is paying both costs and saving nothing → demote it back to API-only and stop pretending.

## How to run it

1. **List the workflows.** One line each → what the job does, how often it runs, what data it touches.
2. **Score each against the four checks** → high-volume, low-sensitivity, latency-tolerant, quality-tolerant.
3. **Emit the routing decision block** per workflow (above). All four pass → LOCAL with a fallback. Any fail → API, with the failing check named.
4. **For LOCAL jobs, drop in the config stub** → set `LOCAL_BASE_URL`, `LOCAL_MODEL`, and the `API_MODEL` fallback. Add a `quality_check` for jobs where a bad output is worth catching.
5. **Wire the fallback rule** and log spills. Watch spill rate for the first week before you trust the lane.

## Make it yours

- **Swap the model + server.** Nemotron 3 Ultra is the worked example; any open-weights model on vLLM / NIM / TGI / Ollama drops into the same OpenAI-compatible shape. Change `LOCAL_MODEL` and `LOCAL_BASE_URL`, nothing else.
- **Tune the four checks to your risk tolerance.** A regulated operator sets the sensitivity bar high and offloads almost nothing. A high-volume content shop offloads aggressively. The checklist is the dial.
- **Add a quality gate per job.** `quality_check` can be a length check, a schema validation, a regex, or a cheap second model grading the output → whatever cheaply catches a bad local answer before it ships.
- **Batch the latency-tolerant work.** The biggest local wins are overnight batch jobs → queue eligible work, drain it against your box while you sleep, spill stragglers to the API in the morning.

## Gotchas

- **Self-hosting overhead is real.** GPU rental or hardware, an inference server, patching, monitoring, and your time. Below a volume threshold the API is simply cheaper once you price your hours. This skill is for operators past that threshold → if you're not, the honest answer is "stay on the API."
- **The quality gap is real.** Near-frontier is not frontier. Open-weights models trail the best hosted models on the hardest reasoning. Offload the work where "near" is fine; keep the frontier for the work where the last few points matter.
- **You own the security now.** A self-hosted box is your patch surface, your access control, your breach. That's exactly why low-sensitivity is a hard eligibility gate, not a soft preference.
- **A local-only path is fragile.** Never route a job to the box with no fallback. The fallback rule is load-bearing → it's what lets you offload aggressively without a single outage stranding a pipeline.

## Output

A routing decision block per workflow, the filled config stub for the LOCAL jobs, and the API fallback rule. Provider-agnostic, copy-paste runnable, no machine-specific paths baked in → set the env vars and run.

## Related
- [[volume-lane-router]] → routes between model *classes* (cheap → frontier). This routes between your *box* and a hosted API. Run volume-lane-router first to find the cheap-eligible work, then this to decide which of it goes local.
- [[which claude model when - 2026-06-08]] → the model-choice logic the routing checklist inherits.
