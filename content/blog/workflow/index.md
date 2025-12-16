---
title: "Reptile: Terminal-Agent with Human-in-the-loop Learning"
date: 2025-12-15T12:00:00+08:00
weight: 1
# aliases: ["/first"]
# tags: ["Research"]
# draft: true
# comments: false
# description: "Desc Text."
# disable_share: false
# hide_meta: false
# hide_summary: false # to hide summary in list
# hide_footer: false
math: true
# search_hidden: false # to hide from search page
show_reading_time: true
show_bread_crumbs: true
show_post_nav_links: false # the prev/next after the content
show_code_copy_buttons: true
show_word_count: true
# use_hugo_toc: true
# show_toc: true
# toc_open: true # default expand all
# cover:
#     image: "path"
#     # can also paste direct link from external site
#     # ex. https://i.ibb.co/K0HVPBd/paper-mod-profilemode.png
#     alt: "<alt text>"
#     caption: "<text>"
#     relative: true # To use relative path for cover image, used in hugo Page-bundles
#     responsive_images: true
#     hidden: false
# header:
#   background: "" # background css value
#   background_image: ""
#   gradient: false
#   blur: false
---
{{< button href="https://github.com/terminal-agent/reptile" label="GITHUB" external=true >}}


## Introduction

We propose **Reptile**, a terminal agent that operates under an extended **REPL (Read-Execute-Print-Learn Loop)** protocol, where human feedback is seamlessly integrated into the agent's execution loop.

Unlike traditional REPL (Read-Execute-Print Loop) environments that focus solely on code evaluation, our REPL protocol emphasizes the iterative cycle of human-agent collaboration, transforming the terminal from a passive command executor into an interactive learning environment.


<figure style="text-align: center; margin: 1rem 0;">
  <img src="https://hackmd.io/_uploads/SkGiD2BWWl.png" style="width: 60%; display: block; margin: 0 auto;">
</figure>



## What Makes Reptile Special?

Compared with other CLI agents (e.g., Claude Code and Mini SWE-Agent), Reptile stands out for the following reasons:

- **Terminal-only beyond Bash-only**: Simple and stateful execution, which is more efficient than bash-only (you don't need to specify the environment in every command). It doesn't require the complicated MCP protocol—just a naive bash tool under the REPL protocol.

- **Human-in-the-Loop Learning**: Users can inspect every step and provide prompt feedback, i.e., give feedback under the USER role or edit the LLM generation under the ASSISTANT role.


This blog focus on **workflow** and **benchmarking**.

> See [TTY-use blog](https://terminal-agent.github.io/blog/tool/) for technical details on how to make terminal backend work.
>
> See [on-policy annotation blog](https://terminal-agent.github.io/blog/annotation/) for annotation details on SWE tasks.


## Our Insights in Building General Agents


**Workflow**: Build the universal action space for the LLM, reserving specialized workflows only for high-risk operations.

**Evaluation**: Focus on learning efficiency on meta-actions like inspecting file in right way, besides end2end benchmark, which makes the optimization more trackable.

**Annotation**: Correct Agent's behaviour with clever annotation (like use PDB Debugging for coding), which enjoys stateful re-run and on-policy prediction.

## First Target Milestone

<figure style="text-align: center; margin: 1rem 0;">
  <img src="https://hackmd.io/_uploads/SyKX2MTMWg.png" style="width: 50%; display: block; margin: 0 auto;">
</figure>


> Extrapolating this trend predicts that, in under a decade, we will see AI agents that can independently complete a large fraction of software tasks that currently take humans days or weeks.
> https://metr.org/blog/2025-03-19-measuring-ai-ability-to-complete-long-tasks/
>

## Why Terminal Agent?


### Promising testbed for Agent Learning
- **Wide Applicability**: Spans everyday tasks to professional workflows (software engineering, DevOps, containerization) through a single universal interface.
- **Native LLM Compatibility**: Terminal protocols are inherently understood through pretraining—no prompt engineering needed, unlike heavyweight protocols like MCP.
- **Core Research Challenges**: Naturally encompasses long-horizon reasoning, context management, error recovery, and compositional tool use.


### Native Universal Protocol

The Unix terminal has always been the universal text interface between human and machine. We believe it can serve the same role for AI agents.

At its core, the terminal is a text-based REPL protocol with half a century of history and refinement:

- **Interpreters**: `bash`, `python`, `node`, `perl`, `ruby` and countless others
- **Debuggers**: `gdb`, `pdb`, `lldb` for interactive debugging
- **Development tools**: `git`, `make`, `docker` for workflows
- **System utilities**: Thousands of battle-tested Unix tools


This mature ecosystem means agents can leverage decades of tooling without reinventing the wheel.

### TTY Implementation Details

We build our LLM agent with https://github.com/sail-sg/tty-use, it implements the REPL protocol to use terminal interactively. A key challenge we solve is how to detect that the foreground process has finished its current job and waiting for next interaction.

For technique details in tty-use, please see our post at https://terminal-agent.github.io/blog/tool/ and https://x.com/mavenlin/status/1977758827366817929.



## How REPL enables Human-in-the-loop Learning?


Human-in-the-loop isn't just for runtime—it's also central to our data collection strategy for further model training.


<figure style="text-align: center; margin: 1rem 0;">
  <img src="https://hackmd.io/_uploads/r1G2tMpG-g.png" style="width: 60%; display: block; margin: 0 auto;">
</figure>


Please refer to https://reptile.github.io/blog/annotation/ for more annotation details and cases.


### Data Collection Workflow



<figure style="text-align: center; margin: 1rem 0;">
  <img src="https://hackmd.io/_uploads/SyRziFHWWl.png" style="width: 60%; display: block; margin: 0 auto;">
</figure>


Our data pipeline source

- All branches are automatically logged with a checkpointing hook.
- User approval / disapproval means something.
- LLM after feedback > LLM before feedback.
- LLM after edit > LLM before edit.
- Feedback & edit are natural to user.
- The more you use, the more data you generate to make the model more like you.



### Usage of the data

- Supervised finetuning.
- Preference optimization.
- RLHF (use the data to train reward model then RL)

## Benchmarking


![image](https://hackmd.io/_uploads/SJF1dfTMbe.png)


We annotate tasks on [SWEgym](https://huggingface.co/SWE-Gym). After training with 200 interactions, this improves Devstral-2505-22B performance:

- [Terminal-bench](https://www.tbench.ai/): 11.3% -> 18.9%
- [SWE-Bench-Verified](https://huggingface.co/datasets/princeton-nlp/SWE-bench_Verified): 18.6 -> 32.8%



## Looking Forward

We are actively working on several exciting directions:


### 1. Terminal Gym for RL Training
We aim to build an **Terminal Gym** that provides a structured environment for reinforcement learning.
This includes (1) procise reward modeling (2) robust and scalable dockerized envs (3) easy-to-hard task sets.

### 2. Advanced Learning Algorithms
We are exploring offline RL, imitation learning, and other techniques to improve sample efficiency for extra-long agent trajectories (>30K length) and ultimately reduce the need for human supervision.


## Open Source & Community

Reptile is open source and we welcome contributions! Whether you're interested in:
- Adding new benchmarks and evaluation tasks
- Improving the hook system
- Contributing training data
- Building integrations with other tools like training/inference backends
- Research discussion and resource collaboration

Visit our GitHub repository: [https://github.com/terminal-agent/reptile](https://github.com/terminal-agent/reptile)

We are inspired by excellent community work such as [terminal-bench](https://www.tbench.ai/) and [mini-SWE-agent](https://github.com/SWE-agent/mini-swe-agent). We thank the community for their efforts and valuable insights!

## Conclusion

The terminal has been humanity's interface to computers for 50 years. With Reptile, it becomes the interface between humans and AI agents.
Reptile represents a new paradigm for terminal agents: one that embraces human collaboration rather than trying to eliminate it.


By extending the familiar REPL protocol with a learning layer, we create a system that:

- Leverages the mature Unix ecosystem without reinvention
- Provides transparency and control through human-in-the-loop interaction
- Scales naturally to complex, multi-step tasks



---

## Citation

If you find Reptile useful in your research or applications, please cite:

```bibtex
@misc{reptile2025workflow,
  title={Reptile: Terminal-Agent with Human-in-the-loop Learning},
  author={Dou, Longxu and Li, Shenggui and Du, Cunxiao and Wang, Tianduo and Zhang, Tianjie and Liu, Tianyu and Chen, Xianwei and Tang, Chenxia and Zhao, Yuanheng and Lin, Min},
  year={2025},
  howpublished={\url{https://terminal-agent.github.io/blog/workflow/}},
  note={Blog}
}
```

---

> **Fun fact**: The name "Reptile" has a dual meaning: it refers to the REPL (Read-Eval-Print-Learning Loop) workflow in terminal interactions, and also pays homage to OpenAI's Reptile meta-learning algorithm (2018), which pioneered few-shot adaptation. Like its namesake, our Reptile learns to quickly adapt to new tasks—but through human-in-the-loop collaboration rather than pure algorithmic optimization. Both share the same philosophy: learning efficiently from minimal examples to master diverse tasks.
>
> *Reference: [On First-Order Meta-Learning Algorithms](https://arxiv.org/abs/1803.02999)*
