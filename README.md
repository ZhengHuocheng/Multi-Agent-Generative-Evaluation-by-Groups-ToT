# **[Multi-Agent-Generative-Evaluation-by-Groups-ToT](https://github.com/ZhengHuocheng/Multi-Agent-Generative-Evaluation-by-Groups-ToT)**

这是一个基于多代理和分组ToT的生成式用户任务自定义的评估框架。该框架的目的是通过组内协商以产生更细粒度的与人类偏好对齐的LLM任务评估。

框架的概念图如下：

![](https://github.com/ZhengHuocheng/Multi-Agent-Generative-Evaluation-by-Groups-ToT/blob/main/Pics/FRAMEWORK.png)

其中：

- 由任务和权重分配Agent，进行子任务生成，并根据子任务的重要性和组的特征进行权重分配；
- 每个分支中进行组内协商(Intra-group Negotiation)，组内Agents通过ToT对彼此子任务的相应进行投票表决，最终通过基于子任务的加权平均，得到组内协商的结果；
- 所有组，对其小组结果进行加权平均，得到总任务的组间共谋结果(Inter-group Negotiation);

运行流程：

1. 创建Agent

   ```Python
   INFJAgent = BaseAgent(name = "INFJ",
                         llmName="qwen:7b",
                         roleDescription="You are the quiet visionary, usually inspiring and tireless idealist.",
                         task="fluency evaluation(evaluate on a scale of 1-5)", #
                         contexts="Knowledge can change your fate and English can accomplish your future."
                         )
   ```

   > [!NOTE]
   >
   > ​    你可以自定义Agent，但是需要注意的是，若你想要执行组内协商和组间共模模块，则目前你需要手动修改相应Group的描述。

   

2. 进行任务划分和权重分配，得到响应

   ```python
   assignAgent = AssignAgent(
                   llmName="qwen:7b",
                   roleDescription="You are a professional psychologist and co-ordinator who specializes in subtask generation and assigning tasks based on character traits",
                   task="fluency ratings(evaluate on a scale of 1-5)", #
   #                 sentence="Knowledge can change your fate and English can accomplish your future."
                   subWorkNumber = 2
   )
   
   step,res = assignAgent.execute_task()
   ```

   

3. 进行组内协商

   ```python
   negotiation = IntraGroupNegotiation(agents=[INFJAgent,INFPAgent],
                                  steps=steps_result,
                                  groups=res_result)
   neg_dfs, overall_respionses = negotiation.run_negotiate()
   ```

   

4. 若你只想进行到这里，那么你可以通过这种方式查看组内协商结果

   ```python
   vote2score_info = negotiation.cal_score(neg_dfs)
   """
   {'vote_detailed':        INFJ  INFP
    step1     1     1
    step2     1     1,
    'maxVotedAgent_eachStep': [['INFJ', 'INFP'], ['INFJ', 'INFP']],
    'step_value_from_agents': array([4., 4.]),
    'step_weights': array([0.6, 0.4]),
    'intra_group_final_score': 4.0}
   """
   ```

   

5. 进行组间共谋

   ```PYthon
   inter_neg = InterGroupsNegotiation(negs,groupsAssign)
   ```

   

6. 此外，你可以通过下面的方式，直接运行不同组内的协商，并接受协商后的相关信息

   ```python
   eachNegtiationInfo = inter_neg.run_negotiations()
   ```

   

7. 最终，你通过如下方式得到组间共谋后的任务评估结果

   ```Python
   groupsNegotiation2Score_Info = inter_neg.cal_groups_score(eachNegtiationInfo)
   """
   {'GroupNegotiations': {'GroupNegotiation1': {'vote_detailed':        INFJ  INFP
      step1     1     1
      step2     1     1,
      'maxVotedAgent_eachStep': [['INFJ', 'INFP'], ['INFJ', 'INFP']],
      'step_value_from_agents': array([4., 4.]),
      'step_weights': array([0.6, 0.4]),
      'intra_group_final_score': 4.0}},
    'GroupWeights': [1.0],
    'InterGroupsScore': 4.0  # 最终的任务得分
    }   
   """
   ```



目前，该框架能够成功运行，还有一些问题将会在作者之后的工作中完成：

- 重构代码，降低内聚，提高耦合；
- 目前，只有接入本地Ollama的接口，主流LLM接口还需添加；
- 添加用例生成模块，拓展框架功能；
