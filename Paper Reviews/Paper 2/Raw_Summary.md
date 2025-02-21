# HAXSS Research Notes

## Introduction

XSS is injecting malicious scripts into websites. It allows attackers to steal data or manipulate website content. The paper is exploring ways to detect XSS vulnerabilities. Existing solutions include black box scanners, but they are not ideal as they lead to many false positives or false negatives. The paper introduces the reinforcement learning approach to detect XSS vulnerabilities.

## Fuzzing and the HAXSS Approach

Fuzzing uses unusual inputs (payloads) to try to elicit unusual website behavior which helps detect vulnerabilities.

HAXSS is the approach that is used. It gamifies the problem of detecting XSS by using fuzzing. This is done in two "games":
- **Game 1:** Generates malicious content that alters the webapps. The goal is to perform context escaping to gain unauthorized privilege to make changes to the webapp through commands.
- **Game 2:** Performs obfuscation by hiding and rumbling malicious parts to bypass sanitization, filtering harmful parts from the user input.

## Reinforcement Learning Fundamentals

Reinforcement learning (RL) keeps exploring possibilities as the agent takes an action every time step. Each action leads to a reward if it is in the right direction. The discount factor balances between future and immediate rewards such that the agent works on achieving the global goal. The agent aims to find the policy (denoted by **pi**) to follow to maximize the reward (**r**) collected, and **pi\*** denotes the optimal policy that the model should strive to achieve for best results.

Q learning is used for finding **pi\*** as it computes the Q value to determine the next action that will maximize the long-term reward. The parameter epsilon randomizes whether to take the greedy or optimal step. Curiosity is also helpful in RL as it encourages the agent to explore options; it is determined by the agent's behavior rather than domain knowledge.

## Web Application Vulnerabilities and XSS

The web app takes in the user input at the source and displays it at the sink for the user. XSS happens when a website accepts user input (source) and later displays it (sink) without proper security checks.

### Types of XSS

- **Reflected XSS:** The malicious script comes from a web request (like clicking a malicious link) and is immediately returned in the response.
- **Stored XSS:** The malicious script is saved on the server and affects all users who visit the page.
- **DOM-based XSS:** The script modifies the webpage dynamically using JavaScript without ever reaching the server.

## Challenges in Detecting XSS

One major challenge is determining whether a payload can effectively trigger a vulnerability. Existing tools are not optimal, as they generate many false positives and many false negatives. Another challenge is successfully escaping the content to fool the webapp when processing inputs. Sanitization contributes to this challenge by preventing context escape and filtering user input to avoid the payload reaching the sink.

Fuzzing is what attackers use to bypass sanitization; it requires many attempts to discover interesting payloads. RL is helpful with fuzzing as it explores many possibilities by injecting the payload into the source and then traversing to the sink to check if the payload succeeded. However, one problem is identifying the space of actions available to the agent in RL. These actions are not as clearly defined as in different contexts, and the agent needs to adapt since no payload is inherently better than another. Instead, the agent must decide the payload based on the context and sanitization.

## Environment Setup and Agent Architecture

The first step in HAXSS is to develop the environment and create an abstraction that hides lower-level details. An automated crawler is used to identify the source and sink combinations. This information is stored by the crawler and then injected into the interface.

- **State:** A tuple of the generated payload and the returned payload from the webapp.
- **Episode:** A unit of actions or duration. An episode ends if a vulnerability is detected or when an agent is reset to its starting state after taking on the payload multiple times.

Helper agents aid the escape agent in the first game. There are four types of actions that an escape agent can perform. A sanitization agent receives the payload, attempts to pass sanitization, and then the unsanitized payload is sent again to the escape agent. Sanitization agents either receive a +10 reward for success or -1 for failure. Both agents use an underlying DQN neural network architecture. Escape agents use a more complex priority buffer, while sanitization agents use a uniform buffer since the priority buffer showed no improvement in performance.

## Performance Metrics

The performance of the trained HAXSS is measured using win and loss criteria:
- **Win Criteria:** The model wins if it reaches 14 generated payloads out of the last 20, indicating consistent exploitation of vulnerabilities.
- **Loss Criteria:** The model loses after 200 episodes without meeting the win criterion. A model reaching HAXSS does not necessarily mean it cannot exploit vulnerabilities; it might still be useful but did not meet the win criterion.

HAXSS is evaluated based on two benchmarks:
1. **Micro-Benchmark:** Uses HAXSS as the vulnerability scanner.
2. **Macro-Benchmark:** Uses HAXSS as a webapp vulnerability scanner. Its performance is compared to existing, established tools, including those from OWASP.

Additionally, the HAXSS model is compared with:
- **AXSS:** Which follows a structured set of actions but without a rigid, step-by-step process (non-hierarchical).
- **Rand-HAXSS:** Which takes random actions, unlike AXSS.
HAXSS is hierarchical, meaning it follows a structured, step-by-step process.

## Comparative Analysis of Tools

All scanners detected the reflected XSS. Most detected the stored XSS, but only ZAP, Arachni, and XSSer were able to find the DOM-based XSS. This provides an abstract idea of which tools are most effective in different contexts.

### Vulnerability Scanner Testing

The tools were tested against 10 easy-to-crawl injection points requiring unique vulnerabilities for exploitation:
- **Wapiti:** Best performing tool, exploiting 8 out of 10 vulnerabilities by using a single payload that bypassed multiple sanitizations.
- **XSSer and XSSmap:** Detected only one and two vulnerabilities, respectively.
- **AXSS:** Reported 5 true positives (TP) and 5 false negatives (FN).
- **Rand-HAXSS:** Reported 6 TP and 4 FN.

This emphasizes the importance of a structured, step-by-step approach when detecting XSS vulnerabilities.

### Requests Generation

A crucial factor in success is the number of requests generated:
- More requests, especially failed ones, alert the server about malicious attempts, eventually leading to request blocking.
- **XSSer:** Generated over 13,000 requests with only one success.
- The trained model generated 724 requests with all 10 successful attempts.
- **Black Widow and Wapiti:** Stood out with a low number of generated requests.
- HAXSS generated relatively many requests, though fewer than AXSS and Rand-HAXSS. Reducing the number of requests could be an area for improvement.

## Web Application Experiment

Another experiment focused on finding XSS vulnerabilities in web apps. Some tools were excluded as they do not have crawlers.
- **Arachni:** Performed consistently well but suffered from a high number of false positives in WAVESEP.
- **Black Widow:** Underperformed, often failing to identify injection points and produce payloads.
- **HAXSS:** Consistently performed well across the five tests. However, in the case of Firing Range, ZAP outperformed HAXSS. This was largely because HAXSS failed to find the injection point using the crawler 126 times, compared to 45 true positives. Similarly, AXSS and Rand-HAXSS also had 126 "not tried" cases.
- All three types of HAXSS had zero false positives.
- Overall, HAXSS outperformed its competitors by a considerable margin. Arachni was the second best, with a total of 100 successful XSS detections (TP) across all five tests compared to HAXSS's 121.

The researcher's claim, *"HAXSS ranks joint third and fourth for the number of FNs in WebSecLab and WAVSEP respectively. This is due to missing exploits that require actions beyond HAXSS current implementation."*, indicates a potential area for improvement.

The high number of "not tried" cases highlights the importance of the crawler in successfully finding injection points to detect XSS.

HAXSS's superior performance is attributed to its ability to bypass sanitization, offering a 20% improvement over traditional tools like Arachni. It also emphasizes the importance of a hierarchical approach (not used by AXSS) and the use of reinforcement learning to generate unique payloads (which Rand-HAXSS does not employ).

A noted shortcoming of HAXSS is the limited size of the action space, which restricts the diversity of payloads that can be generated, leading to increased false negatives. Additionally, HAXSS cannot generate the DOM payloads that ZAP can, limiting its action space and sometimes resulting in higher false negatives.

## In-the-Wild Testing

HAXSS was also tested “in the wild,” where it identified 4 known vulnerabilities and 5 new vulnerabilities in 3 production-grade webapps.
