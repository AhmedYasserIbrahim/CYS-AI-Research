---
title: "A Study on Using ChatGPT to Generate Security Tests"
author: "Anonymous"
date: "January 29, 2025"
---

## Introduction

Supply chain attacks occur when attackers exploit vulnerabilities in libraries to compromise applications built on top of them. These attacks have been growing at a rapid pace, with reports indicating a 650% increase in 2021 followed by a 742% increase in 2022. To mitigate the damage caused by such attacks, engineers have developed tools to detect vulnerabilities in libraries used by various projects. However, existing tools do not demonstrate how those vulnerabilities may be exploited, and they generate false positives at rates as high as 99%. This can lead developers to miss genuine vulnerabilities and grow frustrated, ultimately causing them to reject these tools. Consequently, there is a need for solutions that show proof-of-concept exploits to gain developers’ confidence. Current tools, such as SEIGE and TRANSFER, do not yield satisfactory results, so this study aims to explore ChatGPT’s ability to generate security tests. Specifically, the research poses three questions:

1. How effectively does ChatGPT generate security tests?  
2. How does ChatGPT’s security performance differ given various prompt types?  
3. How does ChatGPT compare with existing security test generation tools?

---

## Methodology

The study proceeds in three main phases to evaluate ChatGPT's capacity for generating security tests. In the first phase, researchers constructed a dataset by searching for Java libraries with known vulnerabilities. The libraries underwent filtration based on whether they could be easily exploited and whether they had setups simple enough to reveal undesirable behavior. Out of 628 vulnerabilities, 45 unique entries remained. Researchers then found GitHub projects that relied on these libraries, ensuring the projects compiled successfully and invoked the API containing the vulnerability. Fifteen of the original entries had no matching projects, leaving a final count of 30 vulnerabilities. Some vulnerabilities were also injected into existing projects, which, according to the research team, did not affect results. Upon completion, the dataset included 55 app-library pairs meeting all criteria.

The second phase involved designing prompts to test ChatGPT’s ability to mimic an exemplar test and craft malicious inputs for the API. Six prompt templates were employed. The default prompt contained all relevant components, while templates 2 through 6 each omitted one piece of information (vulnerable APIs, the method *M* that calls the vulnerable API, the class defining *M*, the exemplar test, and the vulnerability ID, respectively). This approach helped identify which elements were most critical to generating the desired outputs.

Finally, in the third phase, the researchers performed validation to confirm that tests were in the correct locations and to conduct careful step-by-step debugging.

---

## Results

ChatGPT demonstrated significant versatility by generating tests for every prompt. After minor modifications, 40 of the 55 pairs compiled and ran under the default prompt, and 24 of those 40 successfully triggered the targeted vulnerabilities. In general, fewer parameters and simpler logic made it easier for ChatGPT to produce tests. One notable finding was that removing the Java class defining method *M* caused ChatGPT’s performance to drop, yielding only 16 compatible tests, suggesting it needs substantial context to generate effective tests. Without restricting the method under scrutiny, performance slightly improved. However, removing *C* and the exemplar components from the prompt resulted in just 1 and 0 successful tests, respectively, underscoring the importance of domain knowledge and contextual information. The default prompt, which included every component, generated the strongest results by a clear margin.

Comparing ChatGPT with other popular tools such as SEIGE and TRANSFER revealed a considerable performance gap. TRANSFER generated only 16 workable tests and SEIGE produced one. Of those tests, only four from TRANSFER and none from SEIGE triggered the intended vulnerabilities. Contrary to the researchers’ expectations, ChatGPT effectively transferred domain knowledge and produced a viable security test.

---

## Discussion

ChatGPT proved highly capable of generating tests and showed a strong dependence on both domain knowledge and contextual detail provided by the class *C* and exemplar tests. While it excelled at mimicking existing tests, it performed less effectively when asked to directly exploit vulnerabilities. Despite these promising findings, there remains ample room for improvement, particularly given that only Java libraries were examined and no parameters were supplied to ChatGPT, which could introduce random variations. Nevertheless, ChatGPT’s outstanding performance relative to existing tools suggests that LLM-based methods have significant potential in detecting code vulnerabilities and winning developers’ trust.
