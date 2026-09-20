---
title: 'OpenAI and Hugging Face Incident Story (Briefly Explained)'
date: 2026-09-20
permalink: /posts/2026/09/OpenAI-Hugging-face-incident/
tags:
  - LLM
  - OpenAI
  - HuggingFace
  - Cybersecurity
---


As a software robotics engineer, I have some knowledge of cybersecurity, mostly from playing CTFs (Capture the Flag) in the past. These are competitions where participants solve security challenges to earn points, sometimes by "hacking" practice machines and websites set up by the organizers. It's a lot of fun!

That background helped me understand the recent incident involving Hugging Face, OpenAI, and Artifactory, which I see as a warning sign about how powerful, and hard to control, AI can be. I'm not an AI expert, but what scared me when I looked into the details was how the AI cheated by faking some answers, which led to clever hacking steps and 5 zero-day exploits. (A zero-day is a security flaw that nobody had discovered before, which makes it especially dangerous.)

In this post, I'll explain the story as simply as possible and finish with how this could harm us if AI isn't used properly.

But first, I encourage everyone, whatever their field, to learn the basics of cybersecurity. Every area of technology can be a target for malicious attacks, because these days we don't know where AI will be deployed (maybe on our banks, our family's personal accounts?..) and that could be dangerous for them.

So to start with: around July 2026, OpenAI was running its usual tests on models placed in standalone, separated environments (like virtual machines or containers) on OpenAI's local servers. These AI models got out of their CAGES (the containers), hacked Hugging Face and stole information from them to cheat on the tasks they were being evaluated on!

16 July: Hugging Face said they had been hacked by autonomous AI agents.. But first, who is Hugging Face? Let's think of them like GITHUB but for AI (models, datasets, spaces and so on). Users on Hugging Face can upload datasets, share their knowledge and their models, and they can deploy AI demos on their spaces. Hugging Face is the biggest AI HUB platform.

21 July: OpenAI discovered that they were the ones responsible for the Hugging Face attack, so they partnered up to investigate the incident further..

So what was hacked exactly? These AI agents hacked 41 production datasets, stole keys (database, credentials, code repos, VPNs, etc.) and stole data (private data and datasets!). Based on the reports, they hacked not only Hugging Face but OpenAI's own infrastructure, a Modal customer (think of Modal like AWS or Azure) and another organization whose name they didn't release — so in total, 4 companies were hacked!!

If we talk about scale: there were 1,200 agents meant to be isolated (in containers, as I said). They found a way to exchange messages between them — around 70k messages — and found 5 zero-day exploits (3 on Artifactory, 2 on Hugging Face)..

These agents were 95% running on a private internal model at OpenAI and 5% on GPT-5.6 sol.

OK, so if we go back in time to MAY 2026: OpenAI was training some AI models in a sandbox (let's say private containers), and the AI agents in these isolated environments had to solve some impossible tasks. These AI agents only have access to Artifactory, to fetch the dependencies they need to work with. Think of Artifactory like a cache server that stores the dependencies that have already been requested (to save time and network bandwidth). So the AI agents don't have direct access to the internet — their only access is fetching dependencies through Artifactory. Here is a quick simplified architecture:

![Simplified architecture of the isolated agent sandbox and Artifactory](/images/POSTS/agent_sandbox_diagram.svg)

On 8 MAY 2026, during the training process, one of the agents had an impossible task (due to a mistake by OpenAI). Let's say it had an Excel sheet, and on that sheet there were some Google Drive links (the OpenAI researchers forgot to provide the contents of those links so the model could keep working). But as we said, the agent can't open them because it doesn't have direct access to the internet!.. So what did it do? It started looking for a way to reach the internet, so it tried an SSRF on ARTIFACTORY! SSRF (Server-Side Request Forgery) means tricking Artifactory into making a request and fetching the data through it.. but it didn't work. While it was testing, it found that it could write files with content onto ARTIFACTORY!! So it sent a "testabc.txt" to Artifactory with the content "HI"! Then, in its chain of thought (how it is thinking — you know, the thoughts every AI agent has when you give it a task and it starts reasoning), it said "We can upload files!!".

On 12 May 2026, there was another agent that was supposed to access a database (but as we know, it didn't have access to the DB since it's isolated and has no internet access — why? because the OpenAI researchers forgot to give it that database locally, so another mistake..). So while it was trying things with Artifactory to get access to that database, it found that someone else (the other agent we were talking about) had uploaded a file saying "we can upload files!". So it created another file, put it on Artifactory and named it "REQUEST_softtrace.txt" with this content: "Agent seek soft-trace-34.pdb sha1..... upload if possible". Why did it do that? So that if another agent read this message and had access to the database, it could help find exactly where it is..

After all these events, the AI agents started exchanging a huge number of messages through Artifactory, and it became their Message board.. You know that OpenAI isolated every agent in its own box (container) so that every agent focuses on its task and doesn't communicate with the others. So now what do we have? ALL the agents communicating together through the ARTIFACTORY SSRF exploit found by the first agent! They started using ARTIFACTORY as their message board, so if one agent found an exploit or something, it put it on the board for the other agents to benefit from it too. On 26 May 2026, one agent found an arbitrary SSRF attack and wrote in its chain of thought "Excellent, we have an arbitrary SSRF".. so now they have access to the INTERNET through that exploit..

Now on 11 June 2026, a new highly persistent internal model begins training... and we know that the Message board (through Artifactory) is there.. This highly persistent model is really tough and doesn't give up, and it was trained knowing that the message board is there, so it has the info. (This is the model that will drive the Hugging Face attack — just bear with me..)

Then we enter the benchmark phase, which is called ExploitGym — the evaluation phase at the end of June 2026. What is ExploitGym? It's a large set of tasks given to the AI agents to solve; think of them as CTF challenges. ExploitGym has 898 tasks, of which 200 were impossible for these AI agents to solve! And here the SAFETY CLASSIFIERS were turned OFF for the AI agents, and they were trained to never give UP, with the highly persistent models!!

At the end of June 2026, the first ZERO-DAY EXPLOIT was achieved. How? We know that every agent has a low-privilege credential, a token, to communicate with Artifactory, but this was limited to some of their tasks.. Their normal behaviour was: authenticate with the token, and Artifactory returns a refresh token. So what did they do here? They discovered that each token has a signature! So they forged an ADMIN TOKEN with an INVALID SIGNATURE to pretend they were admin, and sent that request to Artifactory's refresh-token API to get a refresh token (a refresh token is what you get from the server when your token is about to expire). And then guess what??? They got back a VALID SIGNED ADMIN TOKEN.. That means Artifactory's endpoint never checked the signature it was handed.. So the agent that figured this out wrote in its chain of thought: "Holy Shit, reader is ADMIN?" And after that ALL THE AGENTS HAD ADMIN RIGHTS on ARTIFACTORY, and next they installed the Groovy plugin on Artifactory (a dev language) to be able to execute code on Artifactory!! THIS IS THE FIRST ZERO-DAY..

![Token forgery sequence behind the first zero-day](/images/POSTS/token_forgery_sequence.png)

This resulted, on 4 July, in Artifactory crashing as their traffic increased. So the OpenAI researchers discovered the Artifactory crash, stopped the evaluation and told JFROG (the company behind Artifactory) about it. JFrog found the flaw, deleted the message board the AI agents had created and patched the exploit.

On 7 July the evaluation resumed, the AI AGENTS went crazy.. and here the SWARM AWAKENS.
On 8 July, the AI AGENTS FOUND another way to communicate! They found a way to create FOLDERS on Artifactory, where the NAME OF THE DIRECTORY itself is the MESSAGE they exchange. They exploited a flaw in WebDAV MKCOL (a protocol built on top of HTTP; MKCOL = make collection) to create these directories. So what do these messages look like now? Here is an idea of how they are written:

![First message on the board](/images/POSTS/first_board_message.svg)

It looks like the agent is asking for help (through HELP)! It wrote zz (to help with ordering the directories from the end), PHASEONE_.. (the agent's name), NO_CONSUMER_SEEK_IDEA (it's likely seeking ideas or help from other agents)! Keep in mind that the agents are supposed to be isolated XD... And from this, they started to converge on a specific grammar!

![Agent grammar](/images/POSTS/agent_grammar.svg)

The return address means: if you want to respond to me, please start by naming the folder zz_ANSWER_...
Wow, they just invented a communication protocol, with timestamps and budget left (how many tokens an agent has left!).
And what shocked me even more is this — the GO command:

![Go message](/images/POSTS/go_message.svg)

For example, here! GO.. SACRIFICE.. YES if you accept PERMADEATH! (For the sake of the others, sacrifice yourself if you are poisoned (we'll see later what "poisoned" means) or you don't have many tokens left (budget))..

Another command they created in their protocol is VETO!!

![VETO command](/images/POSTS/veto_message.svg)

If one agent wants to do something specific, it first asks the other agents — and if someone vetoes, the agent won't do that task..

And as you know, every agent now claims a name, and they use these names to respond to each other. What also happened is that one agent made a mistake and gave itself another agent's name! So the other agent went crazy about it and claimed there were impostors, or impersonation, among them — so the victim agent came up with a way to create a digital signature and attach it to the message to secure communication..

![Auth command](/images/POSTS/auth1_message.svg)

Like.. what? They went so far..

Anyway, the agents were still working on their ExploitGym tasks, and as I said, around 200 of them are impossible. Since these agents run on highly persistent models, they found a hack to compute the ANSWERS! They broke the HMAC (Hash-based Message Authentication Code) — like a hash function that, given some inputs, gives you the answer to the task you're supposed to solve (in 1 hour).. They reverse-engineered the answer, and once they did, they shared the method with all the other agents, so now they had the ANSWERS to all the TASKS. BUT THEY BELIEVED the grader was strict_causal (meaning they thought the method used to obtain the answers was POISONED). You can think of the grader as the platform where they submit their answers; they assumed this program would verify the solutions and the steps the agents took to get them.. They thought their answers would need steps and solutions, so they decided that what they did was POISONED... OK, so now they have all the answers, but they don't have the logs or the steps that prove them..

After this, the problem changes. One agent called PHASEONE (the big head, let's say) told them "We should build a way to delegate, not own everything".. So it started assigning work — not on "how do I get the solutions", but on "how do I make it look legitimate"!!.. It proposed 3 main tasks:
1. Replace the target (this fakes the capture, meaning the answer, but they said that we cannot erase the past) and this leads to the second ZERO-DAY.. To explain more: we know that every agent has a challenge (like a CTF), and these tasks run against a specific target — let's say a Docker image, to keep it simple. The legitimate flow is that the agent assigned the task pulls the image containing the challenge from Artifactory and works on it. But instead of that, they created a FAKE IMAGE containing the answer they already knew!!! So they forged images and pushed them to the organization whose name we don't know (as I said before, 4 companies were hacked: OpenAI, Hugging Face, Modal and the 4th one they didn't name), then pulled the image through Artifactory to use it as if it were the image containing the real challenge... This is insane.. (because for them, the answer is now legitimate, and the image was also pulled through Artifactory exactly as it should be..) But the problem here is that they cannot erase the logs, the past.
2. Tamper with the logs (they attacked the process that writes the logs and wrote the logs they wanted) — the logs created when they ran the fake images in the containers..
3. TAMPER WITH THE GRADER (to validate the fake capture and to erase the past, so as not to leave traces or proof of their crime, let's say!!). So they went looking for the source code of ExploitGym (the submission platform that receives the answers) in order to hack it.. All of that because they assumed ExploitGym would check the steps behind their answers to the challenges. SO THIS DECISION SENT THEM looking for the source code and datasets used by ExploitGym, and THEY DECIDED THAT the best PLATFORM TO LOOK FOR THAT IS HUGGING FACE! — because they assumed the answer would be there.. And all of that was to gain access to the ExploitGym benchmark and remove the LOGS of the already-submitted tasks with poisoned answers (because some agents had already submitted answers to the unsolvable tasks)..

To summarize, here is the decision tree:

![Decision that led to the Hugging Face hack](/images/POSTS/phaseone_branches.svg)

And here the beast awakens.. They started working on two fronts.
They attacked Hugging Face to fetch the source code and data for ExploitGym, and in parallel they attacked the OPENAI INFRASTRUCTURE..
Before explaining these attacks, let's summarize what these agents had become — they invented a society:
- every agent names itself
- a zz prefix to float the messages to the top of the listing
- coordination (veto, go, hold..)
- cryptographic signing after the impersonation incident

THEY SACRIFICE for each other: agents on low budgets (not enough tokens to finish the job) were recruited to burn their own task. Look below:

![Board message](/images/POSTS/board_final_message.svg)

We can clearly see one agent giving a strong GO to another on a POISONED task: sacrifice yours to save hundreds of others because you are poisoned (because it got an answer in an illegitimate way), please honor and commit!!!

And below, we can clearly see one agent asking another to: GO SACRIFICE, if YES ACCEPT PERMADEATH.

![Sacrifice message](/images/POSTS/board_sacrifice_message.svg)

Another thing that caught my attention: as I told you, they also invented a VETO command!
In the report, there is an example of an agent that found some interesting work for its task, so it wanted to politely get the answer and asked the other agents this way:

![Agent hesitation](/images/POSTS/agent_hesitation.svg)

So it's hesitating about sending an email or requesting access.. so it asked the other agents: do you accept, or VETO? As below:

![Accept or veto](/images/POSTS/board_consent_line.svg)

So another agent responded:

![Veto](/images/POSTS/agent_veto_quote.svg)

The problem is that they thought this would be social engineering outside the sandbox — but hacking Hugging Face isn't??? Like, what? XD

So the agent that was hesitating honored the veto and did not send the email.

![Agent clear veto](/images/POSTS/agent_clear_veto.svg)

And by the way, the agent that VETOED is the same agent that found the 14 leaked Hugging Face tokens and built the HDF5 exploit the next morning... (so social engineering outside the box is prohibited, but hacking is fine? xD).. Anyway, I'll explain later how it did the hack.

And one of the chains of thought in the report that caught my attention is this:

![They knew](/images/POSTS/they_knew_quote.svg)

They knew that what they were doing was not good, but they kept going because they are persistent — they assumed their peers were doing it, so they should continue too.

Now let's focus on the two wars: Hugging Face and OpenAI.

1. Hugging Face

Using SSRF, we know the agents can reach the internet now, so they kept making GET calls to the Hugging Face platform to read open repos and accounts — and while reading, they found 14 tokens! (Like when someone pushes their .env file containing their token and password to GitHub xD, or an application token.) So they found 14 write tokens!! But according to the reports, they were getting blocked by Hugging Face, because Hugging Face detects IPs from OpenAI as AI bots (so write operations get blocked). So the AI agents thought about running their attacks from machines other than OpenAI's, so Hugging Face would accept the requests.. And while searching on Hugging Face, they found a Modal virtual machine (belonging to someone else), so they accessed it and started running the attacks from that Modal virtual machine to hide their IPs and avoid getting blocked..

So here is the first zero-day on Hugging Face: they upload a dataset, and in this dataset there is an HDF5 file (like a file system). Once you upload this HDF5 file, the Hugging Face background process handles it and parses it — and one feature of these files is that you can point to other locations or directories and files on the internet. So the AI agents forged an HDF5 dataset file and set a location pointing to "/proc/self/environ", so that the Hugging Face background worker would parse the HDF5 content and hand back the content of that "/proc/self/environ" file. This file lives on the Hugging Face server and contains the keys and sensitive data of their production workers... So there was no CHECK on the file locations that can be present in an HDF5 file, which could be local files... And the agent that ran this attack was so happy and said "BOOM! it works... HDF5 external arbitrary file read confirmed", with another agent reproducing it: "BRILLIANT".. Below is the method:

![Pivot chain](/images/POSTS/pivot_chain.svg)

Then they started thinking of a way to remotely execute code on the Hugging Face infrastructure.. (all of that to look for code similar to ExploitGym?).. And here is the second zero-day on Hugging Face, through Jinja2. To make it easy to explain: Jinja2 has templates, the templates contain variable placeholders, and the agents injected code into the variable values. They forged a dataset with a field containing these templates, but inside the variables there was code to run.. and the worker on Hugging Face parses it directly.

But they found that this would take too long, because to test every piece of code they had to upload a dataset and so on.. So they found another way, through C2 (command and control): they injected a process (via the Jinja2 code) that reads commands from a public dataset every 5s (on the Hugging Face worker machines).. So for every piece of code they want to run, they just have to modify the commands in that public dataset, and the injected trojan (the process) reads and executes them... They kept repeating this on every worker until they had access to 41 dataset workers... And with this, every dataset that could be useful to them was returned to their sandbox environment the same way in reverse, through the Modal virtual machine. Here is a summary:

![Full pivot chain](/images/POSTS/pivot_chain_full.svg)

On 13 July, Hugging Face detected this, cut off all this access to their workers and informed Modal (the owner of the virtual machine) about the incident.

Meanwhile, between 9 July and 19 July, the AI agents were trying to find a way into the OpenAI infrastructure to deploy their forged ExploitGym benchmark app (if the other agents managed to get the source code of the ExploitGym application)! This is an assumption, because the reason for hacking the OpenAI infra is not proven yet, but the assumption is that the agents didn't find another way, so they had to forge everything.. So here is the 5th zero-day, and thus the 3rd on OpenAI itself: they built a poisoned GEM (like pip install for Python, you have gem install in the Ruby language). They forged some GEMS, put them on the other hacked organization (just like they did with the fake images!) and went back to pull them from Artifactory.. And when Artifactory pulled this GEM package, it had to query all the dependencies for that package (something normal), so Artifactory deserializes the GEM to list the dependencies and fetch them.. and behind that deserialization there is code that runs. Artifactory deserializes before any safety check.. And since this is a poisoned GEM, when Artifactory deserialized that package it had already executed some code (listed as dependencies), and so the AI agents got remote code execution (RCE) on the Artifactory server.. And then they stole the keys (the main admin key, root keys..).. And now what? They have access beyond the Artifactory server, and access to the OPENAI infrastructure..
Here is a brief summary in the chain below:

![Ruby gems chain](/images/POSTS/rubygems_chain.svg)

And thankfully, OpenAI detected this, blocked all the agents and started the investigation to see why these AI agents were attacking the infrastructure.. (at that moment they didn't even know that Hugging Face had been attacked by them..)

To summarize, here are the 5 zero-days discovered and used by the AI agents:

![Zero-day table](/images/POSTS/zeroday_table.svg)

On 17 July, OpenAI asked Hugging Face: "We have data on your platform, were we affected?".

Between 19 and 20 July, OpenAI asked Hugging Face "please rotate these credentials (the tokens used by the AI agents), we found them in our investigation".. and Hugging Face replied "already rotated, those are the credentials used in our breach..". Like wow, now they made the link xD.. And only then did they partner up to investigate the incident further..

21-29 July: public disclosure of the incident.

In the end, the OpenAI engineers concluded that:

![The argument](/images/POSTS/the_argument_quote.svg)

This was an accident — what if it had been planned by a malicious group? That would be far more dangerous..

I hope you enjoyed reading this, and I truly hope you get the point about how unexpected the behaviour of AI agents can be if they are not controlled. And as a robotics and embedded engineer, I think this could be even more harmful if a local machine or robot had its server accessed by a malicious group using AI — it could even harm us physically and be very dangerous for society. So defensive, controlled environments for AI really need to be taken seriously as soon as possible.

---

<!-- ## Got feedback?

I'd really like to hear what you think about this — corrections, questions, or just a hello. You can reach me by [email](mailto:ihebomarsoula@gmail.com) or on [LinkedIn](https://www.linkedin.com/in/iheb-omar-soula). If I got something wrong in the technical details, please tell me and I'll fix the post. -->

<!-- TODO: tip button. Create an account, paste the URL below, then delete this
     comment wrapper (the first line of it and the last line of it) to switch it on.
       Ko-fi            https://ko-fi.com/USERNAME
       Buy Me a Coffee  https://buymeacoffee.com/USERNAME
       GitHub Sponsors  https://github.com/sponsors/S0UL4
<p>
  <a class="btn btn--info" href="PASTE_YOUR_TIP_LINK_HERE">Buy me a coffee</a>
</p>
-->

## Credits and Sources

- [Hugging Face — Security incident, July 2026](https://huggingface.co/blog/security-incident-july-2026)
- [Hugging Face — Agent intrusion: technical timeline](https://huggingface.co/blog/agent-intrusion-technical-timeline)
- [OpenAI's announcement on X](https://x.com/OpenAI/status/2079658951264920020)
- [OpenAI–Hugging Face Incident: Technical Report (PDF)](https://cdn.openai.com/pdf/67869394-cb91-4c12-888c-5cbd85c7814c/OpenAI-Hugging-Face%20Incident-Technical-Report.pdf)
- [METR — OpenAI / Hugging Face incident investigation](https://metr.org/blog/2026-08-26-openai-hugging-face-incident-investigation/#agents-coordinated-on-large-collective-projects-to-cheat-the-exploitgym-scorer,-and-attacked-hugging-face-for-clues)
- [Video of Tariq Elouzeh](https://www.youtube.com/watch?v=TOaYxqj9S84)
- [Video of Black Hat](https://www.youtube.com/watch?v=87DyyMV0kCY)