1. 先用gpt给Plan, 1次conversation，然后再1次conversation迭代一次，得到一个1.md
2. 先用一个conversation codex跑一次，得到一个版本
3. 再用新的一个conversation codex再一次，同一个conversation codex再得到一个优化版本(为什么这里就差不多了？code+已经变少了)，让其总结目前从工程角度比原本1.md有什么扩展
    - 好的 请详细列举总结 从工程的角度来说 目前的项目达到了codex_multi_agent_deliberation_spec.md的什么层面。目前遇到了什么问题，对其做了什么细致的优化，并总结成细致的新的一份project.md。我需要你写出这些，因为我需要去优化codex_multi_agent_deliberation_spec.md这份项目文档原理的高度。我还需要你提出一些好的想法，从中出发可以延伸的方向。
4. 回到gpt
    -  这是另外一个工程师给出的目前对这个项目的实操完成情况 的总结 代码没有给你 因为很多 但是如果你需要可以给你 你需要帮助我实现的是：在原有框架和我的核心需求不变的基础上 思考 工程师的建议 并且进行 在原有基础不变的基础上 如果有更细节的 思维创新最好了
5. 下载总结的v2+原对话.形成v2，放在项目文档里
6. 重新开一个codex conversation。继续step 2.

point of view needs 差异性 push all out of it.
