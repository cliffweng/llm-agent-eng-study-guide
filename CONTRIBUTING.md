# Contributing

Thanks for helping improve the guide. A few ground rules:

- **Scope**: one topic per file under `topics/`. Keep each topic readable in ~10 minutes.
- **Template**: follow the structure already used in existing topic files (Why it matters, Core concepts, Mental model, Interview questions, Watch, Further reading).
- **Links must be real**: only link to YouTube videos and articles you have personally verified exist (open the URL, confirm the title/content). Never guess a video ID or URL. Prefer Karpathy, 3Blue1Brown, Hugging Face, LangChain, Anthropic, OpenAI, and DeepLearning.AI, but any reputable, verified source is fine.
- **No invented product direction**: this guide covers LLM/agent engineering fundamentals for software engineers. If you want to propose a new topic or reorganize the curriculum, open an issue first.
- **Local preview**:
  ```bash
  bundle install
  bundle exec jekyll serve
  ```
  Then open `http://localhost:4000`.
- **Pull requests**: keep them focused (one topic or fix per PR) and describe what changed and why.
