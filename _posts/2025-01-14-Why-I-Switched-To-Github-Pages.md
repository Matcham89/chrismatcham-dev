---
classes: wide
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /assets/images/header.jpg
---

<br />

# Why I Switched My Blog from Hostinger to GitHub Pages

Blogging should be simple. For me, it’s about sharing my learning journey and reiterating what I’ve learned—whether it’s Kubernetes, Google Cloud, or any of the other technologies I work with. Yet, my blogging experience with Hostinger became anything but simple, which led me to migrate my blog to GitHub Pages.

Here’s my story, frustrations, and why GitHub Pages turned out to be the perfect platform for my needs.

---

#### The Frustrations with Hostinger’s Website Builder

When I started blogging on Hostinger, I used their website builder UI. At first, it seemed promising: a user-friendly drag-and-drop interface, a quick way to publish posts, and decent customization options. But things started falling apart when I tried to post content with code snippets—an essential part of my blogs.

#### Markdown and Code Formatting Woes

My blogs often include YAML, Python, and other code formats. On Hostinger, the only way to include code snippets was through an embedded code block feature. This might sound fine on paper, but it introduced a cascade of issues:

1. **Formatting Challenges:**
   - Indentation wasn’t respected consistently, which is a dealbreaker for languages like YAML where spacing is critical.
   - The embedded code blocks had limited customization options, making it difficult to present different code types in a readable format.

2. **Copy-Paste Problems:**
   - There was no straightforward way to allow viewers to copy the code directly from the site. The whole point of including code snippets is to let readers try things out themselves!

3. **Time-Consuming Debugging:**
   - Instead of focusing on writing content, I found myself debugging HTML embed code complications. I had to constantly switch between documentation and trial-and-error fixes. This process was frustrating and inefficient.

I have no doubt that someone with more expertise in web development could’ve overcome these hurdles, but as a platform engineer, I wanted to spend my time sharing knowledge—not troubleshooting a blogging platform.

While Hostinger’s website builder may work for some, it simply didn’t align with my needs.

---

#### The Switch to GitHub Pages

Fed up with the complications, I decided to migrate my blog to GitHub Pages. Here’s how I did it and why it’s been a game-changer.

#### Choosing the Right Framework

I chose the [Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/) theme for my GitHub Pages blog. It’s a Jekyll theme that’s highly customizable, open-source, and perfect for writing technical blogs. With minimal updates and a few personal tweaks, I had a clean and professional-looking site.

#### What I Love About GitHub Pages

1. **Markdown-First Approach:**
   - I can write all my posts in Markdown—a format I’m already comfortable with.
   - Code snippets are a breeze to include. Simply wrap them in triple backticks (```) with the appropriate language tag, and GitHub Pages takes care of the syntax highlighting.

   Example:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: example-pod
   spec:
     containers:
       - name: nginx
         image: nginx:latest
   ```

2. **Ease of Deployment:**
   - Deploying a new post is as simple as committing a Markdown file to my GitHub repository. No clunky UI, no complicated processes—just a smooth git push.

3. **Cost:**
   - Hosting on GitHub Pages is completely free. Compared to Hostinger’s ongoing costs, this is a huge win.

4. **Consistency and Simplicity:**
   - My code snippets now render consistently across all posts. Readers can also easily copy the code blocks directly from the site.

5. **Version Control:**
   - Every post is version-controlled by default. I can track changes, revert mistakes, and collaborate if needed.

---

#### Weighing the Pros and Cons

#### Hostinger Website Builder

**Pros:**
- User-friendly for basic sites.
- Decent customization for non-technical users.

**Cons:**
- Limited and inconsistent support for code snippets.
- No built-in solution for copyable code blocks.
- Time-consuming to debug embed issues.
- Ongoing hosting costs.

#### GitHub Pages with Minimal Mistakes

**Pros:**
- Markdown-based, ideal for technical content.
- Built-in syntax highlighting for multiple programming languages.
- Free hosting.
- Simple and efficient deployment process.
- Full control and version history via Git.

**Cons:**
- Requires familiarity with Git and Markdown (though the learning curve is small).
- Limited to static content (not an issue for most blogs).

---

#### Final Thoughts

Switching from Hostinger to GitHub Pages has been one of the best decisions I’ve made for my blog. It aligns perfectly with my goals: a simple, smooth, and consistent platform where I can focus on writing and sharing knowledge—not wrestling with formatting issues.

If you’re a developer or technical writer frustrated with traditional website builders, I highly recommend giving GitHub Pages a try. It’s not just a platform; it’s a productivity booster.