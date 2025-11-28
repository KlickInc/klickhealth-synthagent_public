# SynthAgent: Multi-Agent LLM Framework for Realistic Patient Simulation

Prompts and evaluation templates used in *SynthAgent: A Multi-Agent LLM Framework for Realistic Patient Simulation — A Case Study in Obesity with Mental Health Comorbidities*. The system builds high-fidelity synthetic obesity patients with mental health comorbidities (depression, anxiety, social phobia, binge eating disorder) by chaining specialized LLM agents.

![SynthAgent multi-agent pipeline](figures/pipeline.png)

## What this repository contains

- Prompt specs for every agent in the pipeline (summarizer, generator, augmenter, evaluator, refiner).
- PubMed augmentation prompt and abstraction rules.
- LLM-as-a-judge rubric for independent scoring.
- Statistical priors for demographics, lifestyle, comorbidities, and BMI classes.

> Image paths assume `figures/pipeline.png` (architecture) and `figures/samplepatient4.png` (sample output). Drop the provided images at those paths to render them in this README.

## Why SynthAgent

Obesity care needs clinical, behavioral, and psychological context that typical datasets under-represent. SynthAgent fuses NHANES survey data, medical claims, epidemiological priors, patient case reports, and personality scales to generate privacy-preserving virtual patients that reflect adherence, emotion regulation, and lifestyle dynamics.

## Data inputs

- **NHANES (1999–2023):** Demographics, anthropometrics, labs, lifestyle, and mental health indicators.
- **Medical claims (70k patients, 10-year lookback):** Longitudinal diagnoses, procedures, and medications with obesity ICD-10 codes.
- **Epidemiology priors:** BRFSS, World Obesity Federation, and NCS-R probabilities for demographics and comorbidities.
- **Case reports (PubMed):** Disease- and behavior-specific narratives that ground symptoms, psychological patterns, and treatment response.
- **Personality/behavioral models:** HEXACO, RST, and TCI dimensions used to modulate adherence and coping styles.

## Multi-agent flow

1. **Summarizer (Gemini 2.0 Flash):** Samples epidemiological priors, selects 1 best-fit NHANES record and 3 best-fit claims trajectories, and emits a constrained blueprint (demographics, BMI class, comorbidity, matched timelines).
2. **Generator (configurable LLM):** Converts the blueprint into a full longitudinal patient record (10 sections) with clinical and behavioral coherence.
3. **Augmenter (configurable LLM + Gemini 2.0 Flash):** Retrieves up to 10 PubMed case reports per disease keyword, filters demographically incompatible cases, summarizes evidence, and enriches symptoms/psych scales/role-play narrative.
4. **Evaluator (configurable LLM):** Audits demographics, medical logic, psychology, lifestyle, and temporal consistency; emits a structured issue list with severities.
5. **Refiner (configurable LLM):** Applies the evaluator’s findings, fixes contradictions while preserving identity, and outputs the final patient.
6. **LLM as Judge:** Optional external scorer that grades 10 dimensions (101-point scale) for realism and completeness.

![Sample simulated patient](figures/samplepatient4.png)

## Patient schema (10 required sections)

Demographics · Medical history · Current conditions · Symptoms · Habits · Labs · Treatments · Psychological scales (HEXACO, RST, TCI) · Role-play profile · Disease timeline. Labels for target/general conditions are included for downstream filtering.

## Prompt inventory

- `summarizer_agent_cdc_data_step.yaml`: NHANES matcher and cleanup; converts survey Q&A into clinical summaries for the blueprint.
- `summarizer_agent_claim_data_step.yaml`: Claims matcher; selects and summarizes three longitudinal trajectories before generation.
- `generator_agent.yaml`: Produces the initial full patient (all 10 sections) under provided constraints.
- `augmenter_agent_abstracts_summarization.yaml`: PubMed abstract summarizer (Gemini) that extracts symptoms, behaviors, and psychological impacts.
- `augmenter_agent.yaml`: Integrates PubMed evidence into the base patient (symptoms, psych scales, role-play).
- `evaluator_agent.yaml`: Consistency and plausibility checker across demographic, medical, psychological, temporal, and lifestyle axes; returns a structured JSON report.
- `refiner_agent.yaml`: Resolves evaluator flags, preserves core identity, and guarantees complete YAML structure.
- `llm_as_judge.yaml`: Independent evaluation rubric; scores demographics, history, conditions, symptoms, habits, labs, treatments, psych scales, role-play, and timeline (101 total points).
- `statistical_info.yaml`: Priors for demographics, lifestyle, BMI class mix, comorbidity probabilities, and occupation mappings.

## Evaluation highlights (paper)

- **Models tested as core engine:** GPT-5, Claude 4.5 Sonnet, Gemini 2.5 Pro, DeepSeek-R1 (summarizer fixed to Gemini 2.0 Flash; 30 identical blueprints per model).
- **Quality (mean ± SD):** GPT-5 76.27 ± 2.84; Claude 76.17 ± 3.90; Gemini 71.17 ± 2.96; DeepSeek 67.83 ± 3.26. GPT-5 ≈ Claude (p = 0.91); both >> Gemini/DeepSeek.
- **Dimension strengths:** GPT-5 led on medical history, treatments, role-play, and timeline; Claude led on symptoms, habits, labs, and psych scales; DeepSeek had the strongest “current conditions” scores.
- **Cohort diversity:** Claude generated the most semantically diverse patients (diversity score = 0.353); GPT-5 and DeepSeek were mostly clustered (diversity score = 0.275).
- **Single-agent ablation:** One-shot Claude baseline scored 73.67 ± 3.38, below the multi-agent Claude pipeline (76.17 ± 3.90), showing the value of explicit decomposition.

## How to use these prompts

1. **Prepare inputs:** Provide NHANES samples, claims timelines, and statistical priors expected by the summarizer prompts.
2. **Chain the agents:** Run prompts in the order Summarizer → Generator → Augmenter → Evaluator → Refiner. Keep the evaluator model independent if you want a true second opinion.
3. **Optional scoring:** Use `llm_as_judge.yaml` to grade outputs dimension-by-dimension and track regressions when swapping models.
4. **Swap engines:** Generator/Augmenter/Evaluator/Refiner can share or mix LLM backbones; summarizer and PubMed summarization were tuned for Gemini 2.0 Flash in the paper.

## Citation

```bibtex
@article{synthagent2025,
  title={SynthAgent: A Multi-Agent LLM Framework for Realistic Patient Simulation - A Case Study in Obesity with Mental Health Comorbidities},
  author={Aghaee, Arman and Asgarian, Sepehr and Jeon, Clare},
  year={2026}
}
```

## Contact and license

Specify license and contact info for access or collaboration. Contributions that extend the prompts to new diseases or validation settings are welcome.

**Disclaimer:** Synthetic patients are for research and education only and must not be used for clinical decision-making or regulatory submissions.
