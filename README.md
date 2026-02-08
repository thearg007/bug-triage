# 🐛 Bug Triage Agent

**AI-powered issue prioritization with high-efficiency context compression.**

Bug Triage Agent is a Streamlit-based tool designed to help developers and maintainers manage GitHub issues at scale. By combining the power of **Google Gemini AI** for intelligent classification and **ScaleDown** for cost-effective context compression, it automatically prioritizes bugs, assigns severity scores, and estimates resolution times.

---

## Key Features

- **AI-Powered Prioritization**: Uses Gemini 1.5 Flash to categorize issues into P0 (Critical) to P3 (Low).
- **ScaleDown Compression**: Automatically compresses issue context before sending it to the AI, reducing token usage and slashing API costs.
- **Severity Scoring**: Dynamic scoring (0-100) based on AI classification and critical keyword detection.
- **Resolution Estimates**: Automated time estimates for fixing issues based on their complexity.
- **Compression Metrics**: Real-time tracking of token savings and estimated cost reduction.
- **GitHub Integration**: Simply enter a repository URL (e.g., `owner/repo`) to fetch live open issues.
- **Performance Caching**: Built-in caching for GitHub API requests to ensure a snappy user experience.

---

## Tech Stack

- **Frontend/UI**: [Streamlit](https://streamlit.io/)
- **AI Engine**: [Google Gemini API](https://ai.google.dev/)
- **Compression**: [ScaleDown API](https://scaledown.xyz/)
- **Data Fetching**: [GitHub REST API](https://docs.github.com/en/rest)
- **Environment**: Python, `dotenv`, `requests`

---


## Security & Privacy

- **Safe Defaults**: The `.env` file is included in `.gitignore` to prevent accidental leaks of your API keys.
- **Minimal Scopes**: Uses only public GitHub API access by default.

## Contributing

Feel free to open an issue or submit a pull request if you have ideas for new features or improvements!

---

**Developed with ❤️ by [arg007](https://github.com/thearg007)**
