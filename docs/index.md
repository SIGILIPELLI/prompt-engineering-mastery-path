---
title: "Free Prompt Engineering Course: Design, Test & Evaluate LLM Prompts"
description: "Free vendor-neutral prompt engineering course — 40 lessons on prompt design, few-shot, JSON output, chaining, evals, injection defense and production prompts."
---

# Prompt Engineering Mastery Path


<div class="mp-hero-lockup">
  <div class="mp-hero-globe">
    <svg viewBox="0 0 160 160" class="mp-globe-svg mp-mark">
      <circle cx="80" cy="80" r="74" fill="var(--md-default-bg-color, #FAF7EF)" stroke="#2FA98D" stroke-width="5"/>
      <rect class="mp-bar mp-bar-1" x="36" y="90" width="17" height="24" rx="5" fill="#2FA98D"/>
      <rect class="mp-bar mp-bar-2" x="59" y="74" width="17" height="40" rx="5" fill="#7FC9B7"/>
      <rect class="mp-bar mp-bar-3" x="82" y="56" width="17" height="58" rx="5" fill="#E0B457"/>
      <rect class="mp-bar mp-bar-4" x="105" y="38" width="17" height="76" rx="5" fill="#C97B4A"/>
      <circle class="mp-dot" cx="44.5" cy="82" r="7" fill="#22261f"/>
    </svg>
  </div>
  <div class="mp-hero-divider"></div>
  <div class="mp-hero-text">
    <div class="mp-hero-title">Mastery Path</div>
    <div class="mp-hero-sub">BOOTCAMP</div>
    <div class="mp-hero-rule"></div>
    <div class="mp-hero-byline">VINOD BHANUJA</div>
  </div>
</div>

A prompt is a program written in plain language. Like any program it has inputs,
assumptions, edge cases and bugs — and like any program it can be designed carefully,
tested against examples, measured, and maintained. Most "prompt tips" you will find
online are folklore: magic phrases that seemed to work once, for one person, on one
model version. This course is about the opposite: the craft and discipline of
prompting. You will learn why a prompt works, how to tell whether a change actually
made it better, and how to keep it working when the model underneath changes.

The course is written for two audiences at once. **Everyday users** — writers, analysts,
students, managers — who type prompts into a chat window and want reliable, useful
answers. **Developers** who put prompts inside applications, where a prompt runs
thousands of times on inputs nobody has seen yet. Levels 1 and 2 serve both groups;
Levels 3 and 4 lean toward people who evaluate, secure and ship prompts, but every
lesson explains its ideas in plain terms first.

Level 1 builds the foundations: how a model turns your text into tokens and predicts a
continuation, what the parts of a good prompt are, clarity, context, personas and why
they are weaker than people think, few-shot examples, formatting, iteration, and the
classic failure modes. Level 2 covers the working techniques: reasoning prompts and
when they don't help, chaining, structured JSON output, system prompts, style control,
extraction, classification, summarization, templates and transformations. Level 3 is
about evidence and safety: test sets, rubrics, LLM-as-judge and its biases, A/B tests,
prompt injection from a defender's point of view, guardrails, tool-calling, long
contexts, images, and sampling settings — ending with a small eval harness you run
yourself. Level 4 treats prompts as production artifacts: versioning, regression tests,
cost and latency, agents, domain-specific prompting, ethics, team libraries, and knowing
when retrieval or fine-tuning beats a better prompt.

Every lesson has a **How It Actually Works** section explaining the mechanism behind
the advice — why examples steer format so strongly, why instructions buried in the
middle of a long context get missed, why a model grading its own kind of output is
biased, why "think step by step" changes what the model conditions on.

!!! note "Vendor-neutral, and honest about outputs"
    The techniques here apply to any modern instruction-tuned model — hosted or open
    weights. Examples are written as plain prompts you can paste into any chat
    interface or API. Models change often, and the same prompt can behave differently
    across models and even across versions of one model, so **this course never shows a
    model response as if it were a recorded output**. Where it describes what a model
    typically does, it says so in words. The Python examples use a small *mock model*
    function so they run without an API key; their printed output is real output from
    running that code.

## How the program is organized

| Level | Focus | Modules |
|-------|-------|---------|
| [Level 1 · Entry](level-1/index.md) | Tokens & prediction, prompt anatomy, clarity, context, personas, few-shot, formatting, iteration, failure modes | 9 topics + 1 project (a reusable prompt with before/after) |
| [Level 2 · Intermediate](level-2/index.md) | Reasoning prompts, chaining, JSON output, system prompts, style control, extraction & classification, summarization, templates, transformations | 9 topics + 1 project (support-ticket triage chain) |
| [Level 3 · Advanced](level-3/index.md) | Test sets & rubrics, LLM-as-judge, A/B tests, prompt injection defense, guardrails, tool calling, long context, multimodal, sampling | 9 topics + 1 project (a small eval harness) |
| [Level 4 · Master](level-4/index.md) | Versioning, regression tests, cost & latency, agents, code/data/writing prompts, ethics, team libraries, RAG vs fine-tuning | 9 topics + 1 capstone (a production prompt package) |

## How to use this site

- **Keep a prompt notebook.** A plain text file per task, with each version of the
  prompt, what you changed, and what happened. From Level 1 lesson 08 onward the
  lessons assume you have one.
- **Try every prompt on a model you have access to.** Any capable chat assistant works.
  If you can, try the same prompt on two different models — the differences teach more
  than either result alone.
- **Run the Python.** Code examples need only Python 3.9+ and the standard library.
  Swap the mock model for a real API call when you are ready.
- Do the **Exercise** at the end of each lesson; the module-10 projects build on them.
- **Related courses:** for building LLM applications (API calls, streaming, agents,
  serving) see [LLM Dev Mastery Path](https://sigilipelli.github.io/llm-dev-mastery-path/);
  for retrieval pipelines see [RAG Mastery Path](https://sigilipelli.github.io/rag-mastery-path/);
  for choosing and adopting AI tools see
  [AI Tools Mastery Path](https://sigilipelli.github.io/ai-tools-mastery-path/); and for
  one specific assistant in depth see
  [Claude Training Mastery Path](https://sigilipelli.github.io/claude-training-mastery-path/).
  This course deliberately stays on the prompt itself and links to those where they
  go deeper.

Start here → [Level 1 · Entry](level-1/index.md)


🎥 **Prefer video?** Watch the [Mastery Path video series](https://youtube.com/@sigilipelli) on YouTube — Shorts and full walkthroughs of lessons across the series.

## More from the Mastery Path series

Free, structured, module-wise training across 97 other languages, platforms and disciplines — expand a category to browse:

<div class="mastery-tracks">
<details class="mastery-card">
<summary>Languages <span class="mastery-count">20</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/python-mastery-path/">🐍 Python</a>
  <a href="https://sigilipelli.github.io/java-mastery-path/">☕ Java</a>
  <a href="https://sigilipelli.github.io/javascript-mastery-path/">🟨 JavaScript</a>
  <a href="https://sigilipelli.github.io/typescript-mastery-path/">🔷 TypeScript</a>
  <a href="https://sigilipelli.github.io/csharp-mastery-path/">🔵 C#</a>
  <a href="https://sigilipelli.github.io/shell-mastery-path/">🐚 Shell/Bash</a>
  <a href="https://sigilipelli.github.io/powershell-mastery-path/">💻 PowerShell</a>
  <a href="https://sigilipelli.github.io/c-mastery-path/">🇨 C</a>
  <a href="https://sigilipelli.github.io/cpp-mastery-path/">➕ C++</a>
  <a href="https://sigilipelli.github.io/go-mastery-path/">🐹 Go</a>
  <a href="https://sigilipelli.github.io/rust-mastery-path/">🦀 Rust</a>
  <a href="https://sigilipelli.github.io/sql-mastery-path/">🗄️ SQL</a>
  <a href="https://sigilipelli.github.io/ruby-mastery-path/">💎 Ruby</a>
  <a href="https://sigilipelli.github.io/php-mastery-path/">🐘 PHP</a>
  <a href="https://sigilipelli.github.io/kotlin-mastery-path/">🟣 Kotlin</a>
  <a href="https://sigilipelli.github.io/swift-mastery-path/">🐦 Swift</a>
  <a href="https://sigilipelli.github.io/dart-mastery-path/">🎯 Dart</a>
  <a href="https://sigilipelli.github.io/scala-mastery-path/">🔴 Scala</a>
  <a href="https://sigilipelli.github.io/r-mastery-path/">📊 R</a>
  <a href="https://sigilipelli.github.io/matlab-mastery-path/">🟧 MATLAB</a>
</div>
</details>
<details class="mastery-card">
<summary>Web Frameworks <span class="mastery-count">11</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/react-mastery-path/">⚛️ React</a>
  <a href="https://sigilipelli.github.io/nextjs-mastery-path/">▲ Next.js</a>
  <a href="https://sigilipelli.github.io/angular-mastery-path/">🅰️ Angular</a>
  <a href="https://sigilipelli.github.io/vue-mastery-path/">💚 Vue.js</a>
  <a href="https://sigilipelli.github.io/html-css-mastery-path/">🎨 HTML & CSS</a>
  <a href="https://sigilipelli.github.io/tailwind-mastery-path/">🌬️ Tailwind CSS</a>
  <a href="https://sigilipelli.github.io/django-mastery-path/">🎸 Django</a>
  <a href="https://sigilipelli.github.io/fastapi-mastery-path/">🚀 FastAPI</a>
  <a href="https://sigilipelli.github.io/graphql-mastery-path/">🕸️ GraphQL</a>
  <a href="https://sigilipelli.github.io/nodejs-mastery-path/">🟩 Node.js</a>
  <a href="https://sigilipelli.github.io/spring-boot-mastery-path/">🍃 Spring Boot</a>
</div>
</details>
<details class="mastery-card">
<summary>Mobile <span class="mastery-count">2</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/flutter-mastery-path/">💙 Flutter</a>
  <a href="https://sigilipelli.github.io/react-native-mastery-path/">📲 React Native</a>
</div>
</details>
<details class="mastery-card">
<summary>Testing & QA <span class="mastery-count">6</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/java-testing-mastery-path/">🧪 Java Testing</a>
  <a href="https://sigilipelli.github.io/cpp-testing-mastery-path/">🧪 C/C++ Testing</a>
  <a href="https://sigilipelli.github.io/python-testing-mastery-path/">🧪 Python Testing</a>
  <a href="https://sigilipelli.github.io/automotive-testing-mastery-path/">🚗 Automotive Testing</a>
  <a href="https://sigilipelli.github.io/selenium-mastery-path/">🧭 Selenium</a>
  <a href="https://sigilipelli.github.io/cypress-mastery-path/">🌲 Cypress</a>
</div>
</details>
<details class="mastery-card">
<summary>Security <span class="mastery-count">2</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/cybersecurity-mastery-path/">🛡️ Cybersecurity</a>
  <a href="https://sigilipelli.github.io/ethical-hacking-mastery-path/">🎯 Ethical Hacking</a>
</div>
</details>
<details class="mastery-card">
<summary>Cloud Platforms <span class="mastery-count">5</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/aws-mastery-path/">☁️ AWS</a>
  <a href="https://sigilipelli.github.io/azure-mastery-path/">☁️ Azure</a>
  <a href="https://sigilipelli.github.io/gcp-mastery-path/">☁️ GCP</a>
  <a href="https://sigilipelli.github.io/ibm-cloud-mastery-path/">☁️ IBM Cloud</a>
  <a href="https://sigilipelli.github.io/adobe-mastery-path/">🎨 Adobe</a>
</div>
</details>
<details class="mastery-card">
<summary>Data & Analytics <span class="mastery-count">7</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/data-engineering-mastery-path/">🛠️ Data Engineering</a>
  <a href="https://sigilipelli.github.io/data-science-mastery-path/">📈 Data Science</a>
  <a href="https://sigilipelli.github.io/tableau-mastery-path/">📊 Tableau</a>
  <a href="https://sigilipelli.github.io/excel-mastery-path/">📗 Excel</a>
  <a href="https://sigilipelli.github.io/pyspark-mastery-path/">⚡ PySpark</a>
  <a href="https://sigilipelli.github.io/etl-datalake-mastery-path/">🏞️ ETL & Data Lake</a>
  <a href="https://sigilipelli.github.io/power-bi-mastery-path/">📉 Power BI</a>
</div>
</details>
<details class="mastery-card">
<summary>Databases <span class="mastery-count">5</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/postgresql-mastery-path/">🐘 PostgreSQL</a>
  <a href="https://sigilipelli.github.io/mongodb-mastery-path/">🍃 MongoDB</a>
  <a href="https://sigilipelli.github.io/redis-mastery-path/">🟥 Redis</a>
  <a href="https://sigilipelli.github.io/snowflake-mastery-path/">❄️ Snowflake</a>
  <a href="https://sigilipelli.github.io/databricks-mastery-path/">🧱 Databricks</a>
</div>
</details>
<details class="mastery-card">
<summary>AI / ML / LLM <span class="mastery-count">10</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/ai-ml-mastery-path/">🤖 AI/ML</a>
  <a href="https://sigilipelli.github.io/llm-dev-mastery-path/">🧠 LLM Dev</a>
  <a href="https://sigilipelli.github.io/rag-mastery-path/">📚 RAG</a>
  <a href="https://sigilipelli.github.io/edge-ai-mastery-path/">📱 Edge AI</a>
  <a href="https://sigilipelli.github.io/claude-training-mastery-path/">🔶 Claude Training</a>
  <a href="https://sigilipelli.github.io/ai-tools-mastery-path/">🧰 AI Tools</a>
  <a href="https://sigilipelli.github.io/ml-math-mastery-path/">➗ ML Math Foundations</a>
  <a href="https://sigilipelli.github.io/ai-agents-mastery-path/">🕹️ AI Agents</a>
  <a href="https://sigilipelli.github.io/mlops-mastery-path/">🔁 MLOps</a>
  <a href="https://sigilipelli.github.io/computer-vision-mastery-path/">👁️ Computer Vision</a>
</div>
</details>
<details class="mastery-card">
<summary>Embedded Systems <span class="mastery-count">5</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/embedded-mastery-path/">🔌 Embedded</a>
  <a href="https://sigilipelli.github.io/embedded-linux-mastery-path/">🐧 Embedded Linux</a>
  <a href="https://sigilipelli.github.io/embedded-python-mastery-path/">🐍 Embedded Python</a>
  <a href="https://sigilipelli.github.io/freertos-mastery-path/">⏱️ FreeRTOS</a>
  <a href="https://sigilipelli.github.io/s32k-mastery-path/">🔧 S32K</a>
</div>
</details>
<details class="mastery-card">
<summary>Leadership & Management <span class="mastery-count">5</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/product-manager-mastery-path/">📋 Product Manager</a>
  <a href="https://sigilipelli.github.io/product-lead-mastery-path/">🧭 Product Lead</a>
  <a href="https://sigilipelli.github.io/project-manager-mastery-path/">📅 Project Manager</a>
  <a href="https://sigilipelli.github.io/ai-manager-mastery-path/">🤖 AI Manager</a>
  <a href="https://sigilipelli.github.io/servant-leadership-mastery-path/">🤝 Servant Leadership</a>
</div>
</details>
<details class="mastery-card">
<summary>Professional Skills <span class="mastery-count">2</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/english-fluency-mastery-path/">🗣️ English Fluency & IELTS</a>
  <a href="https://sigilipelli.github.io/workday-mastery-path/">🧑‍💼 Workday</a>
</div>
</details>
<details class="mastery-card">
<summary>Careers & Interviews <span class="mastery-count">2</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/dsa-interview-mastery-path/">🧩 DSA & Coding Interviews</a>
  <a href="https://sigilipelli.github.io/system-design-mastery-path/">🏗️ System Design</a>
</div>
</details>
<details class="mastery-card">
<summary>Process & APIs <span class="mastery-count">3</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/agile-mastery-path/">🔄 Agile/Scrum/Kanban</a>
  <a href="https://sigilipelli.github.io/rest-api-mastery-path/">🔗 REST API</a>
  <a href="https://sigilipelli.github.io/playwright-mastery-path/">🎭 Playwright</a>
</div>
</details>
<details class="mastery-card">
<summary>Infrastructure & Ops <span class="mastery-count">12</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/server-ops-mastery-path/">🖥️ Server Ops</a>
  <a href="https://sigilipelli.github.io/linux-mastery-path/">🐧 Linux</a>
  <a href="https://sigilipelli.github.io/nodemcu-mastery-path/">📶 NodeMCU/IoT</a>
  <a href="https://sigilipelli.github.io/terraform-mastery-path/">🌍 Terraform</a>
  <a href="https://sigilipelli.github.io/kubernetes-mastery-path/">☸️ Kubernetes</a>
  <a href="https://sigilipelli.github.io/docker-mastery-path/">🐳 Docker</a>
  <a href="https://sigilipelli.github.io/github-mastery-path/">🐙 GitHub & Git</a>
  <a href="https://sigilipelli.github.io/cicd-mastery-path/">🔁 CI/CD</a>
  <a href="https://sigilipelli.github.io/ansible-mastery-path/">🤖 Ansible</a>
  <a href="https://sigilipelli.github.io/kafka-mastery-path/">📨 Kafka</a>
  <a href="https://sigilipelli.github.io/microservices-mastery-path/">🧩 Microservices</a>
  <a href="https://sigilipelli.github.io/observability-mastery-path/">📈 Observability</a>
</div>
</details>
</div>

