---
classes: wide
header:
  overlay_color: '#000'
  overlay_filter: 0.5
  overlay_image: https://img.youtube.com/vi/LNNbFsjgP7c/maxresdefault.jpg
  teaser: https://img.youtube.com/vi/LNNbFsjgP7c/maxresdefault.jpg
  og_image: https://img.youtube.com/vi/LNNbFsjgP7c/maxresdefault.jpg
  
  date: 2025-08-16T00:00:00+00:00
---

# I Tried Building My Own MCP Tool From Scratch

In the fast-paced world of tech engineering, staying current with the latest advancements is crucial. In this blog, I share my experience building my own MCP (Managed Control Plane) tool from scratch, despite not being a seasoned programmer. Using the Kubernetes-native KMCP solution, I managed to get a working tool that scrapes YouTube videos and generates blog posts automatically.

## Choosing KMCP and Getting Started
I chose K agents' KMCP tool, a Kubernetes-native agentic solution that standardizes MCP development, making it easier to create and manage tools without worrying about hitting standards manually. After downloading the tool and running the CLI inspector, I was surprised at how straightforward and quick the setup was. The tool provides an interactive inspector to test MCP tools locally before deploying.

## Building My YouTube Scraper MCP Tool
To solve my problem of having more videos but less time to blog, I built a tool that scrapes my YouTube videos and creates clean, proofed blogs from them. Since my videos include many filler words and conversational points, I wanted the tool to not only scrape transcripts but also polish them for readability. I didn't write most of the Python code myself; instead, I used ChatGPT to help generate it.

The MCP tool fetches video information and transcripts using the YouTube API, which required setting up a Google Cloud project and getting an API key. The tool runs locally without Kubernetes, demonstrating flexibility and ease of debugging.

## Exploring the MCP Tool in Action
Running the KMCP inspector alongside my tool showed that it successfully discovered and loaded my custom YouTube tool along with the built-in echo tool. I tested it by pasting a video ID from my YouTube history, retrieving the video title, description, and a cleaned transcript. Though not perfect yet, the setup proved effective for my use case.

## Future Plans and Reflections
Next, I plan to enhance the tool with authentication and integrate it with agent-to-agent communications for more complex workflows. Ultimately, the goal is for the tool to automatically generate blog pull requests on GitHub Pages, alerting me when a blog post is ready for review.

Reflecting on why use MCP tools versus standalone scripts, I realized MCP tools provide a standardized, reusable framework that anyone can run with minimal configuration. This approach opens doors for more people to build powerful tools and promotes adoption in the Kubernetes ecosystem.

## Takeaways
- KMCP offers an approachable way to build MCP tools quickly with minimal setup.
- Leveraging AI assistance like ChatGPT can accelerate development, especially if you're not a seasoned coder.
- MCP tools standardize and simplify integration with Kubernetes-native workflows and tools.
- Running MCP tools locally before deploying helps with debugging and iteration.
- This approach is great for automating repetitive tasks and can be extended for more complex workflows with authentication and agent communication.

<iframe width="480" height="270" src="https://www.youtube.com/embed/LNNbFsjgP7c" title="I Tried Building My Own MCP Tool From Scratch" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
