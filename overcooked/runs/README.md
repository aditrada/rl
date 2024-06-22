# Runs summary

Note: Covergence criterion is:
```python
if ((average_episode_reward_over_last_100[-1] > 140 and np.all(np.array(total_reward_per_episode[-5:]) >= 140)) or
    np.all(np.array(total_reward_per_episode[-20:]) >= 140)):
    print(f"\nConverged: Achieved an average reward >140 on episode {episode_idx}")
    break
```
That is:
- The average over last 100 episodes is > 140 AND all of last 5 episodes had reward >= 140 OR
- All of last 20 episodes have reward >= 140

This was done to mitigate catastrophic forgetting.

## **Cramped Room**
| run_name | gamma | epsilon_decay | batch_sz | optimizer | NN | tau | buffer_size | alpha | converges on |
|--------|--------|---------|------|--------|----------------|-----------|----------|--------|-------|
| run1   |**0.99**| 0.995   | 64   | Adam   | (128, 128, 6)  | 0.001     | 50,000   | 5e-4   | 1000  |
| run2   |**0.90**| 0.995   | 64   | Adam   | (128, 128, 6)  | 0.001     | 50,000   | 5e-4   | 700   |
| run3   |**0.80**| 0.995   | 64   | Adam   | (128, 128, 6)  | 0.001     | 50,000   | 5e-4   | 900   |
| run4   |**0.85**| 0.995   | 64   | Adam   | (128, 128, 6)  | 0.001     | 50,000   | 5e-4   | 800   |
|**run5**|**0.95**| 0.995   | 64   | Adam   | (128, 128, 6)  | 0.001     | 50,000   | 5e-4   | 700   |
|--------|--------|---------|---------------|----------------|-----------|----------|--------|-------|
| run6   | 0.95   | 0.995   | 64   | Adam   | (128, 128, 6)  | 0.001     | 50,000   |**5e-3**| N/A   |
| run7   | 0.95   | 0.995   | 64   | Adam   | (128, 128, 6)  | 0.001     | 50,000   |**1e-4**| 1000  |
|--------|--------|---------|---------------|----------------|-----------|----------|--------|-------|
| run8   | 0.95   |**0.995**| 64   | Adam   | (128, 128, 6)  | 0.001     | 50,000   | 5e-4   | 1200  |
| run9   | 0.95   |**0.990**| 64   | Adam   | (128, 128, 6)  | 0.001     | 50,000   | 5e-4   | 800   |
|--------|--------|---------|---------------|----------------|-----------|----------|--------|-------|
| run10  | 0.95   | 0.990   | 64   | Adam   | (128, 128, 6)  | 0.001     |**10,000**| 5e-4   | 705   |
|--------|--------|---------|---------------|----------------|-----------|----------|--------|-------|
|**run21**|  0.95 | 0.995   | 64   | Adam   | (128, 256, 6)  | 0.001     | 50,000   | 5e-4   |577,14m|



### Comments
* **run1**
    - It looks like both agents settled into distinct roles: agent 1 was the one removing soup and serving
    and agent 2 was cooking soup; thus, agent 1 got higher shaped reward.
    - Overall, the optimla strategy was found and max possible reward was achieved, thus constant reward
    on the after training phase.

* **run2**
    - Making gamma smaller helped with faster convergence (300 episodes faster). This makes sense, as unlike,
    Lunar Lander there is no "final" end goal we want t achieve, rather the goal is to make as many soups as possible,
    so we have to think more about the short-term goals.

* **run3**
    - This converged slower than run2, suggesting the agents may be paying more attention to the short-term "shaped" rewards than the "longer" term reward of making the soup.
    - However, the number of soups made increased linearly over the 900 episodes, unlike in run2, where there was an
    increase, then decrease, and then rapid increase. To learn more about this trend, I started run4 with gamma of 0.85

* **run4**
    - This converged faster than run3 (100 episodes sooner)
    - A thing to note about both run2 and run3, is that eventhough they converged slower than run2 (which converged in 700) episodes, both had more episodes over 140 sooner (run2: from ep 420; run3: from ep 200), they only did not converge due to our convergence "criterion" of averge over last 100 being greater than 140.

* **run5**
    - This gave the best performance during after training, achieving a constant reward of 240, unlike run2 which alternated between reward of 220 and 240.

Possible exlanation of the rise than plateau than rise in reward for training: multi-agent nature: https://g.co/gemini/share/0bf072849f3a.

* **run8**
    - epsilon decay was 0.999, so agents spent more time exploring, therefore it took longer to converge, but there was no rise, plateau, rise pattern; it was more of a steady growth.

* **run8**
    - epsilon decay was 0.990, and agent converged in 800 episodes, and same rise, plateau, rise pattern seen as with epsilon decay of 0.995. Maybe, epsilon decay does not matter as much when reducing from 0.995 to 0.990. The agents explored less with 0.990 compared to 0.995, and took slightly longer to converge.
    - Also the replay buffer is filled with experiences that were random tranistions, with a slower decay schedule,
    the replay buffer will be filled with experiences with more "learned" tranistions. Maybe???

* **run10**
    - Idea: maybe the rise, plateau, and ise again for cramped_room, is that it takes time for each agent to learn that the environment is not fixed (due to another player that is also learning) and account for that. And maybe it takes time because the replay buffer is large, so they use "old" experiences to train, so try reduce the buffer size and see if that improves convergence or gets rid of that pattern.
    - The same rise, plateau, rise pattern seen with buffer of 10,000 so maybe this might not be the case.


## **Assymetric Advantages**


| run_name | gamma | epsilon_decay | batch_sz | optimizer | NN | tau | buffer_size | alpha | converges on |
|--------|--------|---------|------|--------|----------------|-----------|--------|--------|-------|
| run11  | 0.95   | 0.995   | 64   | Adam   | (128, 128, 6)  | 0.001     | 50,000 | 5e-4   | 711   |
|**run22**|  0.95 | 0.995   | 64   | Adam   | (128, 256, 6)  | 0.001     | 50,000 | 5e-4   |602,14m|


### Comments
* **run10**
    - Maximum reward of 400 was acheived. Though in after training, the reward flactuated between 260 and 400, this is becuase one agent was fuly trained "optimally" to be on the right, and ohter on the left; when this was switched, the reawrd decreased to 260. This shows that "cooperation" was not fully learned, the agent was overfitted to one side of the kitchen.
    - Looking at the GIF for the episode that achieved a reward of 400; its very clear that the optimal strategy was found. The agent on the left focused solely on delivering soups (as the serving counter was the closest to him); the agent on the right focused solely on putting onions to cook (as the onion counter was closest to him). So, in this case, they did fully learn to cooperate; but again, the same reward could not be achieved if thier positions in the kitchen were switched.


## **Coordination Ring**

| run_name | gamma | epsilon_decay | batch_sz | optimizer | NN | tau | buffer_size | alpha | converges on |
|--------|--------|---------|------|--------|-----------------|-----------|--------|--------|-------|
| run12  | 0.95   | 0.995   | 64   | Adam   | (128, 128, 6)   | 0.001     | 50,000 | 5e-4   | 1101   |
| run12.1| 0.95   | 0.995   | 64   | Adam   | (128, 128, 6)   | 0.001     | 50,000 | 5e-4   | 1516   |
| run12.2| 0.95   | 0.995   | 64   | Adam   | (128, 128, 6)   | 0.001     | 50,000 | 5e-4   | 1815   |
| run13  | 0.95   |**0.999**| 64   | Adam   | (128, 128, 6)   | 0.001     | 50,000 | 5e-4   | 1934   |
| run14  | 0.95   | 0.995   | 64   | Adam   |**(128, 256, 6)**| 0.001     | 50,000 | 5e-4   | 1105   |
| run14.1| 0.95   | 0.995   | 64   | Adam   |**(128, 256, 6)**| 0.001     | 50,000 | 5e-4   | 1316   |
| run15  | 0.95   | 0.995   | 64   | Adam   |**(256, 256, 6)**| 0.001     | 50,000 | 5e-4   | 1210   |
| run16  | 0.95   | 0.995   | 64   | Adam   |**(512, 512, 6)**| 0.001     | 50,000 | 5e-4   | 1117   |
|**run23**|  0.95 | 0.995   | 64   | Adam   | (128, 256, 6)   | 0.001     | 50,000 | 5e-4   |1100,32m|


### Comments
* **run12**
    - The exact same DQN from cramped_room and assymetric_advantages converged in 1101 episodes.
    - But after visaualizing the episodes, we can see that the agents have not fully learned to utilize the environment, as only one oven is used, the other oven is completely ignored!
    - From ruin 12.2, both stoves were used, but there was no coordination.

* **run13**
    - Trying to use 0.999 as epsilon-decay so that the agents would spend more time exploring did not help either. It resulted in the same score. But at least both ovens were being used.
    - Two hypotheses: (1) the network is constrained, so maybe increase the size of the NN, (2) DQN doesn't scale well to these layouts.

* **run14**
    - Even with a bigger second layer, they still only use 1 stove.

* **run15**
    - Doubling the size of the DQN from run 12 did not yield any improvement, so maybe this is a cooperation/coordination problem, and DQN is simply limited with it.

* **run16**
    - Extremely slow learning. It took 1h30min for the first 1000 episodes.
    - Quadrupling the original network made no difference to the convergence performance, so it is definetly a cooperation/coordination problem.


## **Circuit**

| run_name | gamma | epsilon_decay | batch_sz | optimizer | NN | tau | buffer_size | alpha | converges on |
|--------|--------|---------|------|--------|----------------------|-----------|--------|--------|-------|
| run17  | 0.95   | 0.997   | 64   | Adam   | (128, 256, 6)        | 0.001     | 50,000 | 5e-4   | 1600 (R=120/140)   |
| run18  | 0.95   | 0.998   | 64   | Adam   | (128, 256, 128, 6)   | 0.001     | 50,000 | 5e-4   | N/A   |
| run19  | 0.95   | 0.997   | 64   | Adam   | (128, 256, 6)        | 0.001     | 50,000 | 5e-4   | N/A   |
|**run24**|  0.95 | 0.998   | 64   | Adam   | (128, 256, 6)        | 0.001     | 50,000 | 5e-4   | N/A (reached 140)|
|**run25**|  0.95 | 0.995   | 64   | Adam   | (128, 256, 6)        | 0.001     | 50,000 | 5e-4   | N/A (only 40s)  |



### Comments
* **run17**
    - The agent reached a state of flactuating between 140 and 120 around episode 1500, and that did not change for the next 1500 episodes. It seems the configuration for run17 can't get more than 140 reward, it changes to 120 reward
    when the agent positions are switched.
    - When we visualize the episode, we have the same problem as coordination ring, the agents use only one stove, and there is no propoer cooperation between them.

* **run18**
    - Idea for this run: maybe to get the agents to use both stoves, they need to explore more, for that I increased epsilon decay rate to 0.998 from 0.997 and also increased the netweork size to 4 layers. Both these approaches were tried on coordination ring and they did not work, so I don't have much hope they will work here, but I really want the reward to be greater than 140. If this does not work, I'll try adding the other reward shaping options.
    - Using a bigger neural network did not help at all. Even after 2500 episodes, only a maximum reward of 100 was achieved.

* **run19**
    - Also, with `"POT_DISTANCE_REW": 2,`. But the agent got stuck in 60/80 reward from episode 1500. The agents are getting stuck in a local optima with this layout, I got really lucky with run 17. Final reward was 100 after 3000 episodes.


## **Forced Coordination**

| run_name | gamma | epsilon_decay | batch_sz | optimizer | NN | tau | buffer_size | alpha | converges on |
|--------|--------|---------|------|--------|----------------------|-----------|--------|--------|-------|
| run20  | 0.95   | 0.997   | 64   | Adam   | (128, 256, 6)        | 0.001     | 50,000 | 5e-4   | N/A   |
|**run26**|  0.95 | 0.995   | 64   | Adam   | (128, 256, 6)        | 0.001     | 50,000 | 5e-4   | N/A   |


### Comments
* **run20**
    - The run did not even make one soup over 3000 episodes of training!


