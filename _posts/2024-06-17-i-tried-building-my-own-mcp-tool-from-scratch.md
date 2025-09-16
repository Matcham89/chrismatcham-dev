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

In the fast-paced tech world, staying updated is vital for engineers. This post shares an exciting journey of building a custom MCP (Managed Control Plane) tool from scratch, even without deep coding expertise. Discover how using Kubernetes-native tools and some clever automation, you can create practical solutions that simplify complex workflows.

## What is an MCP Tool?

MCP tools help standardize managing Kubernetes-based workflows and APIs. The author chose the Kubernetes-native K agents tool, known as KMCP, which provides an agent-based solution and recently released an MCP toolkit. This toolkit helps anyone build MCPs without worrying about compliance or standards.

With KMCP, you simply define what your tool should do, and the system builds the MCP around it. The author found the setup straightforward, with CLI tools and an inspector to interact with the MCP before full deployment.

## Building a YouTube Video to Blog Tool

The author wanted a tool that could scrape YouTube videos' transcripts and turn them into clean blog posts — a perfect solution to save time when making many videos but having less time to write blogs.

Key points about the tool:

- It uses Python but the author mostly leveraged ChatGPT to generate the code.
- Pulls YouTube API data, including video info and transcripts, with an API key from Google Cloud.
- The MCP tool is incorporated within KMCP’s CLI setup.
- Runs locally, no Kubernetes cluster required for now.
- Supports various transport protocols like studio SSE and HTTP.
- Can be inspected and managed via the KMCP inspector.

## How the Tool Works

1. The author defines the MCP "YouTube" tool with functions to get video info, including transcript, title, description.
2. Runs locally using the KMCP CLI and inspector.
3. Paste in a YouTube video ID or URL, the tool scrapes the video’s transcript and metadata.
4. The transcript is cleaned and made readable by removing filler words.
5. The author envisions it generating markdown blog posts ready for GitHub Pages with a simple pull request for approval.

This approach highlights that MCP tools are more than just scripting API calls — it's about packaging powerful, reusable automation that can plug into Kubernetes-native tools and developer workflows.

## Why Use an MCP Tool?

The author reflects on why the MCP approach is valuable compared to just writing a standalone Python script:

- MCP tools are standardized and reusable across different clients and platforms.
- They integrate easily into Kubernetes environments.
- They support authentication, agent gateways, and agent-to-agent communication.
- They provide discoverability through tooling like the KMCP inspector.
- MCPs open doors for developers less familiar with complex programming to contribute powerful solutions.
- Building an MCP tool is a great portfolio piece with modern buzzwords like Kubernetes, LLMs, and agents in 2025.

## Next Steps and Advice

The author plans to:

- Add authentication and agent gateway features.
- Automate blog creation and pull request submissions.
- Offer walkthrough tutorials for those interested in building similar MCP tools.
- Encourage the community to join the KMCP project for learning and support.

For anyone curious to get started, the KMCP ecosystem and this YouTube scraping example show how quickly you can build useful MCP tools in mere minutes, even with minimal programming background.

## Takeaways

- MCP tools simplify and standardize building Kubernetes-centric automation.
- KMCP provides a user-friendly toolkit and inspector for fast development.
- Automating YouTube transcripts to blogs is an excellent practical use case.
- Combining MCPs with other automation tools can streamline developer workflows.
- Building MCP tools opens opportunities for career growth and learning modern infrastructure concepts.

<iframe width="480" height="270" src="https://www.youtube.com/embed/LNNbFsjgP7c" title="I Tried Building My Own MCP Tool From Scratch!" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
