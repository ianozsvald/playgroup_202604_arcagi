To see the bot code get into `/ARC-AGI-3` or look at https://github.com/arcprize/ARC-AGI-3-Agents/

You can see the tasks here: https://arcprize.org/tasks

*Uncertainties for Ian and playgroup to resolve*

* The random bot has a 'reason log' but the online scorecard shows 'no reasoning log captured' - what am I missing?
  * Does it have something to do with -> https://docs.arcprize.org/partner_templates/agentops#agentops-template ? 
* When an agent talks about the _colours_ in a grid (e.g. colour 4, 3 etc) - open a snapshot URL and you can mouse-over each cell to learn what number represents the colour

## random bot (no llm)

Following the [quickstart](https://docs.arcprize.org/agents-quickstart#step-2-run-an-agent) we can run a non-LLM random-move agent. You can see the code in `agents\templates\random_agent.py`. It inherits from `Agent` and so becomes discoverable by `main.py`.

This is a fast random-choice both, it doesn't have a goal. If you look at the code you'll see it is short:
* `MAX_ACTIONS` is set to 80, for 80 steps (common to the other bots too)
* `is_done` checks for `GameState.WIN`
* `choose_action` takes a random move and logs a 'reason'

```
ARC-AGI-3-Agents$ uv run main.py --agent=random --game=ls20
...
2026-04-23 10:50:20,723 | INFO | View your scorecard online: https://arcprize.org/scorecards/faaa67f7-a2ed-428a-a6f4-08ad18b40076
```

You'll end with a scorecard URL like the above (for your login), in there you can view a replay.

> [!NOTE] 
> One of the published solutions takes a _log_ of an execution, which could be this random bot, and then an LLM _analyses_ the trace to see 'what happened' in response to decisions. E.g. moving into a wall means no movement, moving away from a wall teaches us that we can move in this environment in 4 directions. 

## `LLMAgent` with `gpt-4o-mini`, has observation, no tools

Described: https://docs.arcprize.org/llm_agents

Code: https://github.com/arcprize/ARC-AGI-3-Agents/blob/main/agents/templates/llm_agents.py#L16

[Actions](https://github.com/arcprize/ARC-AGI-3-Agents/blob/e61e1238eac285901423afc3921fe4b1a51c6701/agents/templates/llm_agents.py#L269) are described in the code:
* `ACTION1` Up
* `ACTION2` Down 
* `ACTION3` Left 
* `ACTION4` Right
* `ACTION5` Space
* `ACTION6` Click (needs an x,y coord)

I suggest changing `MAX_ACTIONS` to e.g. 10 from 80, else it'll take a while to complete. However you can `ctrl c` to break out and it'll still give you the scorecard URL for the partial run.

```
ARC-AGI-3-Agents$ uv run main.py --agent=llm --game=ls20
```

The prompt is generic and not tuned for `ls20`, see it here https://github.com/arcprize/ARC-AGI-3-Agents/blob/main/agents/templates/llm_agents.py#L361

You'll see from the log (below) that it doesn't know _how_ to play the game.

```
2026-04-23 11:09:12,148 | INFO | Assistant: The current frame consists of a largely homogeneous grid where the values are mostly 5, 4, and some 3s, with a few 9s appearing near the end of the grid. The repeated 5s indicate a solid base level, while the 4s and lower values suggest potential areas for action. However, the main action should focus on surrounding the areas with lower values to consolidate scoring opportunities.

There's a distinct cluster of lower values toward the middle-left area, which should be targeted to maximize the score while minimizing actions. The high density of 5s indicates a stable foundation, and targeting the clusters of 3s is essential as they could be beneficial for increasing the overall score in this frame. 

To proceed effectively, I will call an action that targets these lower-value regions to increase the score while maintaining the integrity of the higher-value areas. Therefore, I will focus on surrounding the cluster of 3s. 

**Action:** I will call the action to target the cluster of low values, aiming to convert them into higher values or at least improve their vicinity to increase scoring potential for the next overall frame.
2026-04-23 11:09:12,148 | INFO | Sending to Assistant for action...
2026-04-23 11:09:14,885 | INFO | Received 13036 tokens, new total 25768
2026-04-23 11:09:15,321 | INFO | ls20-9607627b - ACTION2: count 1, levels completed 0, avg fps 0.05)

```

In my run it just kept trying `ACTION2` which is Down, but it started on the bottom bar, so it went nowhere.

> [!NOTE]
> How might the prompt be changed to help it understand _this_ game, rather than the rather crazy ideas it proposes at each step? How could we _discover_ those ideas?

> [!NOTE]
> Could we show the agent that _the last choice made no change in the environment_? If so, we could ask it to at least try something else?

## `FastLLM` with `gpt-4o-mini`, no observation

This sets `DO_OBSERVATION = FALSE` which means that the step in `LLM` (the parent class) that sends the current grid to the LLM for a description is _skipped_. This agent is therefore rather blind!

## `ReasoningLLM` with `o4-mini`, has observation and tools

Described: https://docs.arcprize.org/llm_agents#reasoningllm

Code: https://github.com/arcprize/ARC-AGI-3-Agents/blob/main/agents/templates/llm_agents.py#L403

```
ARC-AGI-3-Agents$ uv run main.py --agent=reasoningllm --game=ls20
...
```

This also has the simplistic prompt (it inherits it from `LLM`) so the following trace shows that it doesn't understand this environment.

```
2026-04-23 11:51:33,598 | INFO | Assistant: Our character (the single “0”) is tucked into a narrow side-chamber in the lower half of the map, just beneath a pair of breakable blocks (“1”). The large cache of “11” tokens in the bottom-right is currently ringed by hazard tiles (the “8”s) and will require opening that choke point to the left of the wall. It looks safer to skirt the mid-level “9” traps by using the western corridor, clear the breakable blocks above us, then swing around to approach the treasure from the flank.
2026-04-23 11:51:33,598 | INFO | Sending to Assistant for action...
2026-04-23 11:51:39,824 | INFO | Received 13384 tokens, new total 26729
2026-04-23 11:51:40,260 | INFO | ls20-9607627b - ACTION1: count 1, levels completed 0, avg fps 0.06)
...
2026-04-23 11:56:25,089 | INFO | Assistant: The map shows a narrow corridor running east–west through the middle, with a cluster of high-value (“9”) targets slightly to the right. There’s a small open area with “11” power-ups near the bottom center; grabbing one would give us a critical boost before engaging. Next, we should veer south to secure that power-up, then approach the enemy cluster from the flank rather than charging straight down the corridor.
2026-04-23 11:56:25,089 | INFO | Sending to Assistant for action...
...
2026-04-23 11:57:29,367 | INFO | Assistant: We’ve successfully dropped into the open courtyard—no enemy patrols in immediate range. I can see the string of 11-strength power-ups still a few rows below and offset to the right. It’s safe to keep pressing south to collect an 11 next, then pivot east to line up on that flank path toward the 9-strength defenders. continuous downward movement will grab the boost before we advance on the enemy cluster.
2026-04-23 11:57:29,367 | INFO | Sending to Assistant for action...
...
```

It certainly looks like this model thinks it is in a war game and is mis-identifying powerups and potential enemies. 

## `o3` based `GuidedLLM` with `ls20` prompt, has observation and tools

Description: https://docs.arcprize.org/llm_agents#guidedllm

Code: https://github.com/arcprize/ARC-AGI-3-Agents/blob/main/agents/templates/llm_agents.py#L496
