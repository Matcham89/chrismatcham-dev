---
classes: wide
header:
  overlay_color: "#000"
  overlay_filter: 0.5
  overlay_image: https://img.youtube.com/vi/LNNbFsjgP7c/maxresdefault.jpg
  teaser: https://img.youtube.com/vi/LNNbFsjgP7c/maxresdefault.jpg
  og_image: https://img.youtube.com/vi/LNNbFsjgP7c/maxresdefault.jpg
  date: 2023-08-13T00:00:00Z
---

# I Tried Building My Own MCP Tool From Scratch

In the rapidly evolving world of tech engineering, staying current is crucial. In this post, I share my journey of building my own MCP (Microservice Control Plane) tool from scratch using K agents' toolkit, despite not being a seasoned programmer in languages like Python or Go. This experience not only taught me about MCPs but also showed how accessible creating such tools can be with the right resources.

## Getting Started with MCP Tools and K agents
I decided to use K agents' Kubernetes-native agentic solution for my project. Their MCP tool simplifies building MCPs by adhering to standards and letting you focus on functionality. After installing the MCP inspector tool via NPM, I was able to interact with my MCP tool locally before full deployment. The CLI tool was straightforward, enabling quick spin-up of custom MCPs with ease.

## My MCP Tool Idea: Video to Blog Automation
The concept I chose was to automate blog writing by scraping my YouTube videos' content. I wanted a tool that could extract transcripts, clean up filler words and conversational detours, and convert the content into a readable blog format. This idea came from my struggle to balance video creation and blog writing workloads.

## Building the Tool Using MCP Infrastructure and Python
Using the KMCP CLI, I generated the entire project directory, including core Python tools, tests, Docker files, and more. Most of the code, especially the Python scripts, was generated with the help of ChatGPT, which saved a lot of effort. I integrated my custom tool called "YouTube" into the KMCP CLI, which pulls video metadata and transcripts via the YouTube API using an API key I created in Google Cloud. The tool retrieves and cleans relevant video data, ready for further processing.

## Running and Testing My Tool Locally
I ran the tool locally without Kubernetes, using the MCP inspector and CLI. The tool supports various transport types like Studio and SSE. Upon running, the tool auto-discovered my custom YouTube tool alongside the built-in echo tool. I tested it by pasting a YouTube video ID, and it fetched the video title, description, and transcript successfully.

## Reflections and Next Steps
This project was a valuable educational experience in MCP development. The goal is to eventually connect this tool with other MCP tools, automate blog post creation on GitHub Pages via pull requests, and add authentication and agent-to-agent communication. The MCP ecosystem standardizes management for microservices, making tools portable and accessible for people at various skill levels.

## Takeaways
- MCP tools like K agents enable developers to quickly build functional microservice tools without deep coding knowledge.
- Automation of repetitive tasks, such as blog writing from videos, can be streamlined with custom MCP tools.
- Leveraging cloud APIs and LLMs can enhance tool capabilities and workflows.
- Local testing is feasible without full Kubernetes deployment, simplifying development.
- MCP tools promote standardized interfaces usable across different platforms and clients, improving tool portability and collaboration.

<iframe width="480" height="270" src="https://www.youtube.com/embed/LNNbFsjgP7c" title="I Tried Building My Own MCP Tool From Scratch" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
