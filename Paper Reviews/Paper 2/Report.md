# HAXSS Report Summary

## Introduction

XSS is a way attackers inject bad scripts into websites to steal data or change content. This report looks at how to find XSS vulnerabilities. Traditional black box scanners often give too many false alerts, so this paper introduces a reinforcement learning approach called HAXSS.

## Approach: Fuzzing and Reinforcement Learning

- **Fuzzing:** Uses unusual input (payloads) to trigger unexpected website behavior.
- **HAXSS:** Turns finding XSS into two “games”:
  - **Game 1:** Creates harmful content to escape context and gain unauthorized access.
  - **Game 2:** Hides malicious parts to bypass filters.
- **Reinforcement Learning (RL):** The agent takes actions, gets rewards, and learns to pick the best actions (policy) using Q learning. Curiosity helps the agent explore different options.

## XSS Vulnerabilities

Websites become vulnerable when they accept user input (source) and show it (sink) without proper checks. There are three main types:
- **Reflected XSS:** Script is sent via a web request and immediately reflected.
- **Stored XSS:** Script is saved on the server and affects many users.
- **DOM-based XSS:** Script changes the webpage using JavaScript.

## Challenges

- **Detection:** It is hard to know if a payload can really trigger a vulnerability.
- **Sanitization:** Filters often block harmful inputs, making it difficult for attackers to bypass security.
- **Action Space:** In RL, defining the best actions for generating payloads is not straightforward.

## Environment and Agent Setup

- An automated crawler finds where to inject payloads (source and sink).
- The RL agent works in episodes (a series of actions) and resets when needed.
- Helper agents support the main escape agent by testing and bypassing sanitization.

## Performance Metrics

The system is measured by:
- **Win:** When 14 out of the last 20 payloads succeed.
- **Loss:** After 200 episodes without enough successes.
- HAXSS is compared using two benchmarks:
  - **Micro-Benchmark:** As a vulnerability scanner.
  - **Macro-Benchmark:** As a full web app scanner.

## Comparative Analysis

- **Tool Comparison:** Tools like ZAP, Arachni, and XSSer were compared.
- **Findings:** 
  - All tools found reflected XSS.
  - Only a few tools detected DOM-based XSS.
  - Wapiti performed well with a single payload, while others like XSSer sent thousands of requests for minimal success.
- **Observations:** A structured, step-by-step (hierarchical) approach like HAXSS performs better than random or less structured methods.

## In-the-Wild Testing

HAXSS was also tested on real web apps. It successfully identified 4 known vulnerabilities and found 5 new ones in production-grade applications.

## Conclusion

HAXSS shows promise by using reinforcement learning and a hierarchical approach to improve XSS detection. While it outperforms some existing tools, areas like expanding the action space and reducing unnecessary requests are still challenges to address.
