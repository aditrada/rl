# Runs summary

## **Cramped Room**

| Run | gamma | lambda | batch_size | n_epochs | actor_lr | critic_lr | epsilon | target_kl_div | entropy_coeff | max_grad_norm | max_iters | NN | Converges on |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| run1  | 0.99 | 0.97 | 64 | 15 | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.01 | 0.5 | 400 | (128, 128, 128, 128, 128, 6) | 1010 |
| run2  | 0.95 | 0.97 | 64 | 15 | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.01 | 0.5 | 400 | (256, 256, 256, 128, 128, 6) | 990  |
|**run22**| 0.95| 0.97| 512| 6  | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.2  | 0.5 | 300, 10| (256, 256, 128, 128, 6) | 640,18m  |


## **Assymetric Advantages**
| Run | gamma | lambda | batch_size | n_epochs | actor_lr | critic_lr | epsilon | target_kl_div | entropy_coeff | max_grad_norm | max_iters | NN | Converges on |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| run3  | 0.95 | 0.97 | 64 | 15 | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.01 | 0.5 | 400 | (256, 256, 256, 128, 128, 6) | 700?  |
|**run23**| 0.95| 0.97| 512| 6  | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.2  | 0.5 | 300, 10| (256, 256, 128, 128, 6) | 670,23m  |


## **Coordination Ring**

| Run | gamma | lambda | batch_size | n_epochs | actor_lr | critic_lr | epsilon | target_kl_div | entropy_coeff | max_grad_norm | max_iters, traj | NN | Converges on |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| run4  | 0.95 | 0.97 | 64 | 15 | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.01 | 0.5 | 400, 5 | (256, 256, 256, 128, 128, 6) | N/A  |
| run5  | 0.99 | 0.98 | 64 | 15 | 4e-4 | 4e-4 | 0.05| 0.2  | 0.2  | 0.5 | 400, 5 | (256, 256, 256, 128, 128, 6) | N/A  |
| run6  | 0.95 | 0.97 | 64 | 10 | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.2  | 0.5 | 400, 5 | (256, 256, 256, 128, 128, 6) | N/A  |
| run7  | 0.95 | 0.97 | 128| 8  | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.2  | 0.5 | 500, 10| (256, 256, 256, 128, 128, 6) | 2690, 1h4m  |
| run8  | 0.95 | 0.97 | 128| 6  | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.2  | 0.5 | 300, 10| (256, 256, 128, 128, 6) | 1920  |
| run9  | 0.95 | 0.97 | 256| 6  | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.2  | 0.5 | 300, 10| (256, 256, 128, 6) | 2010  |
| run12 | 0.95 | 0.97 | 512| 6  | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.2  | 0.5 | 300, 10| (256, 256, 128, 128, 6) | 1810  |
| run13 | 0.95 | 0.97 | 512| 6  | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.35 | 0.5 | 300, 10| (256, 256, 128, 128, 6) | N/A |
|**run24**| 0.95| 0.97| 512| 6  | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.2  | 0.5 | 300, 10| (256, 256, 128, 128, 6) | 1810,57m |


### Comments
* **run5**: these were copied from overcooked ai repo (some of them), and it clearly did not work. The increase in entropy coefficient is to incrase explration (with anealing rate), and this makes sense as run4 did not converge due to not enough exploration. But run5 did not converge, amybe due to too much clipping, epsilon is 0.05, so policy can't change much.

* **run6**: Reducing n_epochs to 10 seems to have improved perfomrance: likely due to avoidance of overfitting. So, maybe larger rollout and just playing for longer may help convergence; that is increase max_iter. Maybe also try aealing cliip rate/epsilon, in next run.

* **run7**: Here I, changed the rollout size to 10, thus each trajectory will be 4000 in length, and the mini-batch size in 128 to compensate for the doubling the size of the training data. And this run had the best performance of all runs in coordination ring. However, I did not account for the increase in trajectory length with max_iters. So even after 1200 episodes, the iter was just 120, and thus learning rate and epsilon anealing did not really work, and this was visible in the agents when thier performance stagnated after reaching episode 900, with rewards at 40/60 and not improving; probably because the high learning rate and epsilon prevented convergence.
    - So maybe try reduce max_iter to 250 (so max of 2500 episodes) next time.
    - Also try epsilon anealing?
    - But, at episode 1509, shaped reward for agent is 70, so the agents are not exploring, but entropy coefficient is high (as anealing rate is low), so maybe the network is overfitting. So maybe reduce the number of epochs to 6?

* **run8**:
    - From run7: best performace so far but not converging fast enough, stagnating at 40/60 since episode 800.
    - So, new things to try:
        - Number of epochs: 8 -> 6 (to prevent overfitting, and thus stagnation)
        - Reduce max_iter: 400 -> 250 (so anealing can take place in time for learning rat and entropy coeff (exploration))
        - epsilon anealing will try next round.

* **run8**:
    -  Tried a smaller network (by removing a 256-unit layer) and used fewer epocs (8 to 6), and performance improved.
    - Thus, for next run, try remove even another layer and fix enropy coefficent decay.

* **run9**:
    -  Tried an even smaller network, by removing a 128-unit layer, and also reduced epochs from 8 to 6. The performance only got worse by 90 episodes. I don't know because of which of the two reasons though or just a random seed.

* **run12**:
    - Seems like using a bigger batch_size helped compared to run 8, or this may just be the random seed.
    - Observation: if you look at the during train graphs for run12, and run11; in both the shaped reward increases, linearly, but common_reward increases super-linearly only in run12 and not in run11. This indicates that the number of soup pickups, etc are improving but placement of the soups is not improving. This, then means that in run11 (circuit), there was no enough exploration to find the right placement and thus intrinsic reward might do really well.

* **run13**:
    - In this run, we were testing the effect of high entropy coefficient (0.35) to encourage exploration. And this
    run did not do well at all. Too much time exploring and not enugh time exploting. The fact that there is no
    experience replay also means that the agent may not be necessarily learning as much from previous exploration.
    - Hmm, what happens if we lower entropy_coeff to 0.1

## **Forced Coord**

| Run | gamma | lambda | batch_size | n_epochs | actor_lr | critic_lr | epsilon | target_kl_div | entropy_coeff | max_grad_norm | max_iters, traj | NN | Converges on |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|**run25**| 0.95| 0.97| 512| 6  | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.2  | 0.5 | 150, 10| (256, 256, 128, 128, 6) | N/A |


## **Circuit**

| Run | gamma | lambda | batch_size | n_epochs | actor_lr | critic_lr | epsilon | target_kl_div | entropy_coeff | max_grad_norm | max_iters, traj | NN | Converges on |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| run10 | 0.95 | 0.97 | 256| 6  | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.2  | 0.5 | 450, 10| (256, 256, 128, 6) | N/A  |
| run11 | 0.99 | 0.97 | 512| 6  | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.35 | 0.5 | 450, 10| (256, 256, 128, 128 6) | N/A  |
|**run26**| 0.95| 0.97| 512| 6  | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.2  | 0.5 | 150, 10| (256, 256, 128, 128, 6) | N/A |

### Comments
* **run10**:
    - This was the previously trained agents from run9, and they performed horribly. Curriculum learning may not have helped because the high loss during the first iterations of training (due to 0 reward and high entropy) may have caused all the previously learned representations to be lost with massive weight changes.
    - Perhaps it did not work becuase the network was too small? remember this is hust a (256, 256, 128, 6) NN, so next run increase this to (256, 256, 128, 128, 6).
    - Also increase entropy coefficient to 0.35 to encourage even more exploration, as it looks like the agents have not eplored enough from run10.
    - Also, according to MAPPO paper, larger mini-batches helped, so increase the batch size to 512
    - Observation: There was clearly reward hacking, they started optimizing for the shaped rewards instead of the actual reward of making the soups, so increase gamma to 0.99. This kinda makes sense, as the path from ingredients to stove is many steps away, so they have to consider long-term reward more.

* **run11**:
    - Performance was still very very bad. I don't know which of the changes made caused the performance to worsen: the batch size, the NN size, gamma, entropy coeff, or using a non-trained agents.
        - I don't think it is gamma, or NN.
        - Probably, the performance at the start (inital 1500) was bad becuase of high entropy coefficient.
        - Maybe batch size increase caused it to be worse: have to test this next.
        - I'll test these hparams on coordination ring so that I can get a more clear signal. I'll do it one by one: first change batch size (to 512, run12), then entropy_coeff (to 0.35, run13)

TODO:
    - make sure entropy coefiicient decay is correct
    - Also try epsilon anealing?

#### To for PPO
Based on: https://bair.berkeley.edu/blog/2021/07/14/mappo/
1. Training Data Usage:
    - We recommend using 15 training epochs for easy tasks, and 10 or 5 epochs for more difficult tasks. We hypothesize that the number of training epochs can control the challenge of non-stationarity in MARL.
    - We additionally avoid splitting a batch of data into mini-batches, as this results in the best performance.

2. PPO Clipping:
    - We generally observe that smaller ϵ values correspond to more stable training, whereas larger ϵ values result in more volatility in MAPPO’s performance.




# Shared Critic PPO Runs Summary

## **Cramped Room**

| Run | gamma | lambda | batch_size | n_epochs | actor_lr | critic_lr | epsilon | target_kl_div | entropy_coeff | max_grad_norm | max_iters, traj | NN | Converges on |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|**run27**| 0.95| 0.97| 512| 6  | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.2  | 0.5 | 300, 10| (256, 256, 128, 128, 6) | 810 |

## **Assymetric Advantages**

| Run | gamma | lambda | batch_size | n_epochs | actor_lr | critic_lr | epsilon | target_kl_div | entropy_coeff | max_grad_norm | max_iters, traj | NN | Converges on |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|**run28**| 0.95| 0.97| 512| 6  | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.2  | 0.5 | 300, 10| (256, 256, 128, 128, 6) | 1210 |


## **Coordination Ring**

| Run | gamma | lambda | batch_size | n_epochs | actor_lr | critic_lr | epsilon | target_kl_div | entropy_coeff | max_grad_norm | max_iters, traj | NN | Converges on |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| run14 | 0.95 | 0.97 | 512| 6  | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.2  | 0.5 | 300, 10| (256, 256, 128, 128, 6) | N/A  |
| run15 | 0.95 | 0.97 | 512| 6  | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.2  | 0.5 | 300, 10| (256, 256, 128, 128, 6) | 2510  |
| run16 | 0.95 | 0.97 | 512| 6  | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.2  | 0.5 | 300, 10| (256, 256, 128, 128, 6) | N/A  |
| run17 | 0.95 | 0.97 | 512| 6  | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.2  | 0.5 | 300, 10| (256, 256, 128, 128, 6) | 1690  |
|**run29**| 0.95| 0.97| 512| 6  | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.2  | 0.5 | 150, 10| (256, 256, 128, 128, 6) |  |


### Comments
* **run14**:
    - The learning seems to be much slower than independent PPO agents. Agent 0 learned pretty much nothing, his shaped_reward was stuck at 20ish and agent1 shaped_reward was at 60ish.
    - Hypothesis:
        - shared critic may need more layers to have more representational power.
        - I am still using individual rewards for calculation of GAEs, so maybe this is sending mixed signals?
        - Will make both these changes for the next run.

* **run15**:
    - Used a bigger NN for the critic: (256, 256, 256, 128, 128, 6)
    - Thinking a little more about using "combined reward" (common + both agents shaped reward) for calculating GAEs, this is clearly wrong. GAEs are used to tell the actor policy if the poilicy is improving, and this should only be affected by common reward and the agent's shaped reward; not the other agent's shaped reward. Why should an agent be rewarded if the other agent picks up onions?
        - And by using the same target for the actor training too, we are simply just training the same agent twice. There is no difference in training data seen by both the agents!
        - Another problem, since we are adding up all the rewards, the extra reward achieved by an extra soup delivery is now a smaller share.
    - But surprisingly, the performance improved over run14. Again this could be because of the bigger NN or the common GAE calculation or both.
    - The performance worse than without shared critic version, and not improved, so that is still concerning.
    - Next run, try without shared GAE calculation version.
    - One more observation is that one agent completely dominates the other.

* **run16**:
    - Ran the same configuration as run15, with two changes:
        1. The critic NN size is same, but the number of epochs for critic training is 16 (double that of run15).
        2. Used each agent's reward to calculate the GAEs.
    - I ran this run twice, and the agents never converged, even after 3000 episodes.
    - So, using each agent's individual reward when training a shared-critic does not work!

* **run17**:
    - Ran the exact same configuration as run16, but now used the combined_rewards to calculate the GAEs, and the run converged in 1690 episodes.

## **Circuit**

| Run | gamma | lambda | batch_size | n_epochs | actor_lr | critic_lr | epsilon | target_kl_div | entropy_coeff | max_grad_norm | max_iters, traj | NN | Converges on |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| run18 | 0.95 | 0.97 | 512| 6  | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.2  | 0.5 | 300, 10| (256, 256, 128, 128, 6) | N/A  |
|**run31**| 0.95| 0.97| 512| 6  | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.2  | 0.5 | 150, 10| (256, 256, 128, 128, 6) |  |

### Comments
* **run18**:
    - Tried the same configuration as run17 but now on circuit layout, and it never converged (after 3000 ep). The shaped rewards increased but no soups were ever made.

## **Forced Coordinataion**

| Run | gamma | lambda | batch_size | n_epochs | actor_lr | critic_lr | epsilon | target_kl_div | entropy_coeff | max_grad_norm | max_iters, traj | NN | Converges on |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|**run30**| 0.95| 0.97| 512| 6  | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.2  | 0.5 | 150, 10| (256, 256, 128, 128, 6) |  |


# Shared Ploicy and Critic PPO Runs Summary

## **Coordination Ring**
| Run | gamma | lambda | batch_size | n_epochs | actor_lr | critic_lr | epsilon | target_kl_div | entropy_coeff | max_grad_norm | max_iters, traj | NN | Converges on |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| run19 | 0.95 | 0.97 | 512| 20 | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.2  | 0.5 | 300, 10| (512, 256, 256, 128, 128, 128 6) | 1680  |
| run20 | 0.95 | 0.97 | 512| 10 | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.2  | 0.5 | 300, 10| (256, 256, 128, 128, 128, 6) | 1630  |
| run21 | 0.95 | 0.97 | 512| 6  | 4e-4 | 4e-4 | 0.2 | 0.02 | 0.2  | 0.5 | 300, 10| (256, 256, 128, 128 6) | 1910  |


### Comments
* **run19**:
    - critic: (256, 256, 256, 128, 128, 6)
    - This run was rather slowed (but it is promising). It is slow of course due to the network size (7 layers) and 20 epochs. I'll try reduce both next run to see the perfromance.

* **run20**:
    - Reducing capacity did not affect convergence performance, let's reduce network size further.

* **run21**:
    - Reducing capacity further compared to run20 made performance worse, so run20 had optimal network size and epochs.
