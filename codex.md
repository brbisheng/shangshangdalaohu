**Core build requirements for the first MVP**

1. **Do not spend unnecessary time expanding non-core components in the first MVP version.**
   Focus only on the smallest set of functions that directly improve ability.

2. **Make sure to provide a testable demo example program at each key milestone.**
   Every major function must come with a minimal runnable example that proves it works.

3. **Each core function must be independently runnable from a simple command.**
   No core capability should depend on a large unfinished framework before it can be tested.

4. **Do not build decorative architecture before the core reasoning gain is verified.**
   Avoid dashboards, service layers, broad plugin systems, UI polish, or large orchestration code in phase 1.

5. **Every new component must justify itself by improving the agent’s output quality in a visible way.**
   If a component does not clearly improve depth, structure, relevance, or surprise generation, postpone it.

6. **Prefer a small number of hard-working modules over a large number of loosely useful modules.**
   The first MVP should be narrow, sharp, and testable.

7. **Each milestone must end with a concrete before-vs-after demonstration.**
   Show how the agent performs without the new function, then show how it performs with it.

8. **Use structured outputs from the beginning.**
   Even in the first MVP, outputs should be machine-checkable and easy to compare across runs.

9. **Do not hide incomplete logic behind placeholder abstractions.**
   If a feature is not implemented, leave it out instead of wrapping it in premature architecture.

10. **Optimize for measurable reasoning improvement, not software completeness.**
    A smaller system that clearly makes one agent stronger is better than a broader system that only looks sophisticated.

11. **Keep the first MVP easy to inspect, debug, and modify.**
    The code should stay simple enough that each reasoning step can be traced by hand.

12. **Add tests and demo cases at the same time as each core function, not afterward.**
    Verification is part of implementation, not a later cleanup step.



# USE GOOGLE GEMINI TO DO DEEP RESEARCH.


1. 先用gpt给Plan, 1次conversation，然后再1次conversation迭代一次，得到一个1.md
2. 先用一个conversation codex跑一次，得到一个版本
3. 再用新的一个conversation codex再一次，同一个conversation codex再得到一个优化版本(为什么这里就差不多了？code+已经变少了)，让其总结目前从工程角度比原本1.md有什么扩展
    - 好的 请详细列举总结 从工程的角度来说 目前的项目达到了codex_multi_agent_deliberation_spec.md的什么层面。目前遇到了什么问题，对其做了什么细致的优化，并总结成细致的新的一份project.md。我需要你写出这些，因为我需要去优化codex_multi_agent_deliberation_spec.md这份项目文档原理的高度。我还需要你提出一些好的想法，从中出发可以延伸的方向。
4. 回到gpt
    -  这是另外一个工程师给出的目前对这个项目的实操完成情况 的总结 代码没有给你 因为很多 但是如果你需要可以给你 你需要帮助我实现的是：在原有框架和我的核心需求不变的基础上 思考 工程师的建议 并且进行 在原有基础不变的基础上 如果有更细节的 思维创新最好了
5. 下载总结的v2+原对话.形成v2，放在项目文档里
6. 重新开一个codex conversation。继续step 2.

point of view needs 差异性 push all out of it.
