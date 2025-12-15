---
title: "On Policy Annotation: Minimal Human Edits Unlock Massive Gains in LLM Agents"
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
{{< button href="https://github.com/terminal-agent/AnnotationGuidelines" label="GUIDELINE" external=true >}}
{{< button href="https://www.notion.so/On-Policy-SFT-Annotation-How-Minimal-Human-Edits-Unlock-Massive-Gains-in-LLM-Agents-2b80ba07baa880d6ba7fca50816d33f2?source=copy_link" label="NOTION-Version" external=true >}}



## 1. The Need for a Fast System in Terminal-Based SWE Data Collection

In terminal-based SFT data collection for software engineering (SWE) tasks, system responsiveness is a crucial determinant of both data efficiency and data quality. For annotators who are not proficient with command-line interfaces, two fundamental issues—slow typing speed and difficulty recalling commands—create significant friction in the labeling process.

First, **the interaction cost of terminal input is inherently high**. Unlike graphical interfaces that offer affordances such as buttons, menus, or auto-completion, *terminals rely entirely on textual command entry*. Each operation requires the annotator to **recall and retype precise commands and parameters, often under the risk of syntax errors**. When annotators frequently make syntax or command errors due to limited terminal proficiency, the repeated cycles of editing, rerunning, and verifying become time-consuming and mentally exhausting, causing substantial inefficiency and fatigue throughout the labeling process.

Second, **human aversion to repetitive manual typing further amplifies the problem**. Typing long or complex commands is cognitively taxing and error-prone, especially for users without strong muscle memory or command-line experience. Combined with slow system feedback, this friction discourages annotators from engaging deeply with each task. In practice, many tend to skip difficult samples or submit minimal edits just to complete the assignment, resulting in lower data diversity and weaker supervision signals for downstream model training.

Together, these factors raise the competence threshold for participation in terminal-based annotation. Only annotators with sufficient technical proficiency and patience can maintain throughput and consistency, which limits the scalability of data collection and increases training and management costs. Consequently, a fast and responsive system is not merely a convenience but a prerequisite for obtaining high-quality SFT data in SWE settings.

## 2. Edit over Write: An LLM-in-the-Loop Workflow for Terminal Based SWE Annotation

### 1) Problem and the Shift in Paradigm

In terminal based annotation for software engineering tasks, many annotators type slowly and do not remember command syntax well. Writing every command from scratch invites errors and repeated retries. Efficiency suffers because each correction requires new typing and renewed recall of flags and parameters.

We address this by shifting from a workflow where humans author the full annotation trajectory to a workflow where humans edit the model’s proposal. The model produces the next candidate command. The human makes a minimal correction only where the proposal is wrong. This keeps human effort focused on judgment rather than on transcription.

**Example**

Previously, to inspect the most recent 50 lines that contain *Timeout*, the annotator had to compose:

```bash
grep -n "Timeout" /var/log/app.log | tail -n 50
```

With LLM in the loop, the model might propose:

```bash
grep -n "Error" /var/log/app.log | tail -n 50
```

The annotator edits only one token: change `"Error"` to `"Timeout"`. No full rewrite is needed.

{{< youtube izcwX2tMsnU >}}


---

### 2) Why this Division of Labor Works

Empirically, LLM are usually competent at command **syntax**. The more common weaknesses are in **strategy** or **step selection**. Humans are better at deciding what to do next, such as which file region to inspect first or which tool is more suitable, while the model can handle the exact flags, quoting, and piping.

This division of labor assigns the human to decision making and assigns the model to concrete string production. The human does not need to recall details of `sed`, `awk`, `jq`, or intricate `pytest` flags. The model turns concise human intent into a runnable command.

**Examples**

* The human decides to read lines 100 to 140 of a large file. The model emits:

  ```bash
  sed -n '100,140p' big.txt
  ```

* The human decides to view the JSON field `user.id`. The model emits:

  ```bash
  jq '.user.id' data.json | head -n 20
  ```

---

### 3) System Desgin: Interaction Protocol (p & e)

Each step begins with a model proposal for the next operation.

* If the human judges it reasonable, press **p** to proceed and execute it.
* If the human judges it needs adjustment, press **e** to edit only at the error point and apply a minimal change.

After a minimal edit, the system triggers **completion**. Completion exists because modern language models are causal. Given a prefix, the model continues naturally to the next tokens based on ( P(x_{t} \| x_{< t}) ). The goal of completion is to reduce human input and memory load. The human points out the key change and the model fills in the routine details, such as flags, pipes, and ordering.

**Examples**

* The model proposes:

  ```bash
  pytest tests/test_user.py::test_login -q
  ```

  The human wants to run only slow tests first. Press **e** and introduce the key change `-m slow`. Completion can then add commonly used parameters the team prefers.

* The model proposes:

  ```bash
  cat config.yaml
  ```

  The human wants only lines that contain `timeout`. Press **e** and state that intent. Completion yields a concrete command such as:

  ```bash
  grep -n "timeout" config.yaml | head -n 20
  ```

---

### 4) Additional Benefits

**Closer to on policy data.** Most tokens are produced by the model and only a small portion is edited by humans. The resulting samples are closer to the model’s own generation distribution, which is useful for subsequent alignment.

**Safer real use.** In real usage the same edit based protocol applies. Every step is either accepted or minimally edited before execution. This keeps the process controllable, auditable, and reversible, rather than handing full control to an autonomous agent.

**Positive feedback over time.** Continued use accumulates edit data. Fine tuning on these edits makes the model better aligned with human preferences. Over time the accept to edit ratio increases and the average edit distance decreases. The workflow becomes easier the more it is used.

---

## Discussion
An interesting future direction lies in developing **human–AI collaborative systems** that unify model assistance and data collection. Instead of maintaining purely human-driven or LLM-driven workflows, future systems could adopt a collaborative design in which **humans and LLMs jointly perform tasks**. During this process, user edits, acceptances, and rejections can be automatically captured as on-policy SFT data, closely reflecting the model’s real deployment environment.

A successful example of this paradigm is *Cursor*, which automatically completes code and allows users to decide whether to accept the suggestions. While users adopt Cursor primarily to enhance coding efficiency, the system simultaneously accumulates rich behavioral data, such as acceptance or rejection signals—that can be used to further fine-tune the model. As this collaborative loop continues, both the model’s performance and the user experience improve over time.


## Citation

```
@misc{reptile2025annotation,
  title={On Policy Annotation: How Minimal Human Edits Unlock Massive Gains in LLM Agents},
  author={Du, Cunxiao and Wang, Tianduo and Dou, Longxu and Li, Shenggui and Zhang, Tianjie and Liu, Tianyu and Chen, Xianwei and Lin, Min},
  year={2025},
  howpublished={\url{https://terminal-agent.github.io/blog/annotation/}},
  note={Blog},
}
```