---
classes: wide
header:
  overlay_color: '#000'
  overlay_filter: '0.5'
  overlay_image: /assets/images/header.jpg
  teaser: /assets/images/gcp-art-logo.png
  og_image: /assets/images/gcp-art-logo.png
---

# I Tried Building My Own MCP Tool From Scratch!

In this blog post, I'll share my journey of building a custom MCP (Managed Control Plane) tool from scratch, despite not being an expert programmer. Using the K agents toolset and some clever integrations, I developed a tool that scrapes my YouTube videos and creates blog posts from their transcripts, streamlining content creation. If you've ever wanted to explore MCPs or automate video-to-blog workflows, this overview provides useful insights and practical steps.

## What Is an MCP Tool and Why Build One?

MCP tools facilitate building managed control planes in a standardized way, helping developers avoid reinventing the wheel for common infrastructure and API challenges. The Kubernetes native K agents solution stood out to me because it offers a quick start, rich community support, and tooling such as the MCP inspector CLI tool, making it easy to interact locally or in Kubernetes environments.

I wanted to experiment with building an MCP tool not to master programming but to understand the ecosystem and realize something helpful for my content workflow.

## My Concept: Video-to-Blog Scraper

Creating more videos leaves me less time to write blogs. So, I designed a simple tool that:

- Takes a YouTube video URL or ID,
- Pulls metadata like title and description,
- Fetches the transcript,
- Cleans and proofreads the raw text into polished blog content.

The goal is to have a semi-automated, reviewable flow where the AI agent drafts the blog post from video content, then I approve the results before publishing.

## Building the Tool Using KMCP and Python

To build this tool:

- I used the KMCP CLI and layered my Python code inside the generated MCP tool directory.
- Majority of the Python code, including API integrations and cleaning logic, was assisted by ChatGPT.
- The tool calls the YouTube API to get video info and transcripts, using an API key from Google Cloud.
- It supports outputting a clean transcript and metadata for any input video ID or URL.

The tool runs locally  94 no Kubernetes needed at this stage  94 and works with the MCP inspector CLI to test the interaction smoothly.

## How It Works in Practice

When running the tool:

- The MCP inspector discovers two tools: the built-in echo and my YouTube scraper.
- I input a YouTube video ID, and the tool retrieves the title, description, and cleaned transcript.
- The transcript is free from filler words like "um" and "ah," making it suitable for blog conversion.

Eventually, my plan is to chain this with other MCP tools to:

- Automatically create structured Markdown blog posts,
- Push GitHub pull requests for review,
- Receive alerts for pending content approval.

This setup automates much of the repetitive content creation, leaving me free to focus on review and publishing.

## Why Use MCP Tools Over Simple Scripts?

At first, I wondered why not just use a standalone Python script. The advantage of MCP tools is:

- Standardization: The tool can plug into any client or platform that supports MCPs and Studio protocol.
- Extensibility: Easily add authentication, agent-to-agent communication, or deploy in Kubernetes.
- Community and ecosystem: Benefit from existing frameworks and tooling without reinventing infrastructure.

This makes MCP tools accessible to less seasoned programmers, opening doors to building powerful solutions and valuable portfolio projects that highlight modern cloud-native tech skills.

## Next Steps and Learning

As I continue, I plan to:

- Add authentication and security features,
- Explore agent gateways for communication between tools,
- Set up scheduled jobs or commands for periodic scraping,
- Integrate with GitHub	2s MCP tool for pull request automation.

This journey has been educational, showing how approachable MCP development is with the right frameworks and some AI assistance.

---

## Takeaways

- MCP tools provide a standardized framework for building cloud-native, extensible, and interoperable tools.
- Using K agents and KMCP CLI, you can quickly scaffold and run MCP tools locally or in Kubernetes.
- Automating video transcript scraping into blog posts saves time and creates a reusable workflow.
- AI tools like ChatGPT can boost productivity by generating code snippets and cleaning logic.
- This approach is beginner-friendly and great for showcasing modern dev skills involving Kubernetes, MCPs, and AI integration.

---

<iframe width="480" height="270" src="https://www.youtube.com/embed/LNNbFsjgP7c" title="I Tried Building My Own MCP Tool From Scratch!" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
