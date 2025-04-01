---
classes: wide
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /assets/images/header.jpg
  teaser: /assets/images/github-pages.png
  og_image: /assets/images/github-pages.png
---

<br />

## Intro
Kubernetes is a powerful tool, but it can feel overwhelming at first. If you're preparing for the CKAD (Certified Kubernetes Application Developer) exam, you might be wondering where to start or how to study effectively. I’ve been there! In this post, I’ll share my experience, what worked for me, and how you can best prepare—even if you're new to Kubernetes.


I passed the CKAD on my first attempt, and I want to help you do the same. I’ll cover essential tips, study strategies, and key focus areas to ensure you’re well-prepared and confident on exam day.


## Exam Environment
Before we dive into study techniques, let’s go over what to expect during the exam.


- **Remote Proctored:** The exam is taken online with a proctor watching you through your webcam.
- **Strict Rules:** Your desk must be clear—no notes, no extra monitors (unless allowed), and no external devices.
- **External Webcam:** Highly recommended, as it helps with proctor verification.
- **Allowed:**
  - Access to Kubernetes documentation (`kubernetes.io`)
  - A glass of water
  - Breaks
  - An external monitor (this helps a lot!)


Knowing these details in advance will help you avoid unnecessary stress on exam day.




## Quick Note: Grading
The CKAD exam is graded by **automation scripts**, not a human. That means:


- No partial credit for “almost correct” in the wrong place.
- Typos or incorrect YAML indentation can cost you points.
- Precision and accuracy are key!


This is why practicing commands and YAML formatting is so important.




## Tips for Success


### 1. Learn to Love kubectl Aliases
`k=kubectl` Alias is available for the exam, but it's configured—no need to set it up yourself.


### 2. Master kubectl Autocompletion
Autocompletion is pre-enabled in the exam environment. Use **Tab** to autocomplete commands and arguments, which reduces typos and speeds up your workflow.


### 3. Context Switching is Critical
Every question may use a different Kubernetes context. If you forget to switch contexts, you’ll make changes in the wrong cluster and lose points.


- Each question provides a **context-switching command**—**clicking it copies it automatically**.
- Always treat each question as a separate environment.


### 4. Use the Copy Option
If the **copy button** is available, use it. This prevents spelling mistakes, which can cause lost points.


### 5. Manage Your Time Wisely
The exam consists of **16-22** questions in **2 hours**, meaning you have about **5-7 minutes per question**.


- Some questions are quick, others take longer.
- If you get stuck, **don’t panic!** Flag the question, make a note, and move on.
- You don’t need a perfect score—just enough to pass.


### 6. Read the Question Twice
Exam pressure can cause you to skim questions and misunderstand what’s being asked.


- Read **carefully** before typing.
- Make sure you understand **exactly** what the question is asking for before writing any commands.


### 7. YAML & VIM Are Your Best Friends
You **must** be comfortable with YAML and VIM. Here are some essential VIM commands:
```
vim filename   # Open or create a file
i              # Insert mode (edit text)
esc            # Exit insert mode
:wq            # Save and quit
:w             # Save
:q             # Quit
:q!            # Force quit (without saving)
u              # Undo (outside insert mode)
```
Practicing VIM before the exam will save you a lot of time.


### 8. Essential Terminal Commands
Besides `kubectl`, you must be comfortable with basic Linux commands:
```
cp             # copy
mv             # move
echo           # repeat
ls             # list
cd             # change directory
ssh            # secure shell
echo >         # repeat to
```
If you usually rely on an IDE like VSCode, spend some time working **exclusively in the terminal**.


### 9. Get Comfortable Navigating the Documentation
The fact that you **can** use Kubernetes documentation in the exam is a huge advantage!


- **Practice finding answers quickly.**
- Know where key sections like Deployments, ConfigMaps, and Secrets are located.


### 10. `kubectl explain` is a Lifesaver
If you're unsure about a YAML structure, use:
```
kubectl explain pod.spec.containers
```


or


```
kubectl explain deployment
```
It provides quick insights into resource definitions for lots of different resources.


### 11. `kubectl -h` Helps With Command Syntax
If you forget how to structure a command, use:
```
kubectl create secret -h
```
This is faster than searching the documentation. Great for secrets, cronjobs, deployments and configmaps.


### 12. `-o yaml` and `--dry-run=client` Are Your Best Friends
These flags help generate YAML templates quickly:
```
kubectl create deployment nginx --image=nginx -o yaml --dry-run=client
```
Save the output, modify it, and apply it using:
```
kubectl create deployment nginx --image=nginx -o yaml --dry-run=client > deployment.yaml
kubectl apply -f deployment.yaml
```




## Key Focus Areas
For the CKAD exam I focused on these topics:


### 1. PersistentVolume & PersistentVolumeClaim
- Creating **PersistentVolumes (PV)** and **PersistentVolumeClaims (PVC)**.
- Correctly mounting volumes to pods.


### 2. CronJobs
- Understanding **schedule syntax** and required YAML structure.


### 3. ServiceAccounts
- Creating and troubleshooting **ServiceAccounts** and **RoleBindings**.
- Using `kubectl get -o yaml` and `kubectl describe` for debugging.


### 4. Rollouts
- `kubectl rollout` manages deployments beyond just restarting them.
- Learn how to view rollout history and perform rollbacks.


### 5. Canary Deployments
- Learn how **labels** help control traffic.
- Remeber you can use `kubectl scale` to adjust replica counts.
- Also, whilst it may be obvious to some, understand ratios. For example, if you    have 10 replicas of a deployment and you want to perform a canary deployment of 20% canary and 80% stable. That's 2 replicas of the new canary deployment and 8 replicas of the stable deployment.
  Worth noting that the replica count can be managed via the `kubectl scale` command.


### 6. Resource Limits & Requests
- Understanding how to **set and update** resource limits.


### 7. LimitRange
- Learn how Kubernetes **enforces resource limits** on pods and containers.


### 8. Secrets
- Creating and mounting **Secrets** as environment variables and volumes.


### 9. Logs
- Deployment logs may contain **both old and new pod logs**.
- Use `kubectl logs` efficiently.
- Remember when you look at deployment logs, this can include an old pod and new pod. But looking at a pod log is fine grained to that single pod.


### 10. Labels
- Labels are key for **NetworkPolicies** and **Canary Deployments**.
- Using `kubectl label` and `kubectl edit` effectively.


### 11. Docker / Podman
- Be able to save a Docker image:
```
docker save --output busybox.tar busybox
```
- Learn **both** Docker and Podman.


### 12. `kubectl top pods`
- Identify high CPU/memory usage.
- Use `--sort-by` to quickly find problem areas.




## Resources


Here's a list of the resources that I used and believe helped me pass the CKAD exam:


- [OG Kubernetes Docs](https://kubernetes.io/) - The go-to source for all Kubernetes documentation and explanations.  
- [Killer Shell](https://killer.sh) - Truly harder than the real exam. If you can pass this, you can pass anything.  
- [A Cloud Guru](https://www.pluralsight.com/cloud-guru) - Great labs and a brilliant course, IMO.  
- [Killer Coda](https://killercoda.com/) - From the creator of Killer Shell, Killer Coda is another great tool.  
- **Practice! Practice! Practice!**  




## Final Thoughts
The CKAD exam is tough, but you **can** pass with preparation.
Focus on:
- Practicing YAML & VIM
- Using the terminal efficiently
- Mastering `kubectl`
- Navigating the documentation quickly


This exam isn't about memorization—it’s about being **comfortable working in Kubernetes**.
Stay calm, manage your time, and trust your preparation.


🚀 **Good luck!**