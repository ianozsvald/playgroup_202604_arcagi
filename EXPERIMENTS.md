To see the bot code get into `/ARC-AGI-3` or look at https://github.com/arcprize/ARC-AGI-3-Agents/

You can see the tasks here: https://arcprize.org/tasks

## random bot

Following the [https://docs.arcprize.org/agents-quickstart#step-2-run-an-agent](quickstart) we can run a non-LLM random-move agent. You can see the code in `agents\templates\random_agent.py`. It inherits from `Agent` and so becomes discoverable by `main.py`.

```
ARC-AGI-3-Agents$ uv run main.py --agent=random --game=ls20
...
2026-04-23 10:50:20,723 | INFO | View your scorecard online: https://arcprize.org/scorecards/faaa67f7-a2ed-428a-a6f4-08ad18b40076
```

You'll end with a scorecard URL like the above (for your login), in there you can view a replay.

> [!NOTE] 
> One of the published solutions takes a _log_ of an execution, which could be this random bot, and then an LLM _analyses_ the trace to see 'what happened' in response to decisions. E.g. moving into a wall means no movement, moving away from a wall teaches us that we can move in this environment in 4 directions. 

## `o3` based `GuidedLLM` with `ls20` prompt
