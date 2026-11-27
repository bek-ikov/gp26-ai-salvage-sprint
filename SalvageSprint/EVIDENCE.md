# Week 1 evidence

Keep this short. Bullet points, a small table and one diagram or code trace are enough. This is your checkpoint record, not an extra report. Replace the prompts as you work.

Name: Asilbek Khalikov
Exact Unity version: 6000.3.26f1
Checkpoint commit / repository: https://github.com/bek-ikov/gp26-ai-salvage-sprint
Current stage and anything unfinished: Beginning of Stage 4

## Initial prediction

Three observations the rival needs, including one it might not have: Visibility, reachability, and distance.

Seeing a pickup across a gap does not prove reachability because: The AI might not have an accessible path to that pickup.

One question about NPC behaviour: How does an NPC decide what the next course of action should be when it's intended goal gets abruptly hindered?
## One complete trace

Observe → Infer → Choose task → Movement → Evaluate navigation feedback → Update memory after failure → Repeat

Scenario: The AI detects two available pickups. Pickup 3 is closer but unreachable due to an obstacle, while pickup 7 has valid route. (Hidden pickup test)

1. Observation
PickupSensor.Capture() gathers information about nearby pickups.
| Property | Value |
|---|---|---|
| Carrying | false |
| Pickup 3 | Distance: 16, Available, Visible |
| Pickup 7 | Distance: 49, Available, Visible |

2. Decision (Infer)
DecisionPolicy.Infer() sleects the closest available candidate, producing:
- NeedsDelivery: false
- HasUsableTarget: true
- SelectedTargetId: 3
- RetainedTarget: false

DecisionPolicy.Choose() then selects:
- Task: Collect
- TargetId: 3
- Reason: Nearest candidate; reselected each tick

3. Movement
RivalBrain.Update() requests movement towards pickup 3 through Navigator.Request(). The NavMeshAgent fails to find a complete route. Navigator.Poll() uses NavigationFeedback.Classify() to identify the failed route and returns TravelStatus.Failed.

4. Evaluation after feedback
RivalBrain.Update() records the failed target in failedUntil, causing it to be marked as blocked for six seconds. The next observation is captured with this information. If DecisionPolicy.Infer() excludes blocked candidates, pickup 7 becomes the target.

Explore → Collect when an eligible pickup is selected.
Collect → Deliver after successful collection gives cargo.
Deliver → Collect or Explore after delivery, depending on available knowledge.
Collect → another Collect target or Explore when the current target becomes unusable

Use one actual situation. Include the observation data, derived facts, selected task and reason, movement request/result, and how feedback affects a later decision. Identify the responsible code locations. A tiny state diagram can accompany this trace.

## Scenario record

| Scenario | Predicted result | Observed result and short evidence / remaining issue |
|---|---|---|
| Normal loop and restart | The AI looks for the nearest scrap piece and goes to pick it up. After the scrap piece is obtained, it delivers it to the depot. | The AI does as predicted, and wins after picking up and delivering 5 pieces. |
| Carrying cargo | The AI delivers the carried cargo and returns to the usual observe → collect → deliver loop. | The AI does as predicted. |
| Hidden / remembered pickup | The AI disregards the hidden scrap piece and searches for another one. | The AI detects the hidden scrap piece repeatedly, despite not having a direct line-of-sight route to it. This leads to the AI being stuck in place. Picking up the hidden scrap piece as a player frees the AI from the detect → fail loop.  |
| Target taken | The AI looks for another target and routes to it. | The AI briefly enters the Explore state, before finding a new target and routing to it. |
| Unreachable target | The AI disregards the unreachable scrap piece and searches for another one. | Much like the issue in the hidden cargo test, the AI is stuck in place due to the detect → fail loop. Due to the fact that the player can't reach the scrap piece either, the AI is stuck that way. |
| No usable pickup | The AI explores until it finds a pickup | The AI explores until it completes one whole loop. Since the AI deems that there are no pickups on the map, the game ends in a draw once it comes back to the depot. |

Supplied test result, with any added tests identified: 14 pass, 3 fail
Live walkthrough checked by / recording link if used:

## One correction and one independent check

The original weakness and why it happened: The AI would get hung up if the nearest pickup was blocked. This is due to the fact that Infer() did not consider if the target was visible, remembered or blocked.
What I changed and why it addresses the cause: I added a visibility and memory check in Infer(), meaning that the AI will only consider remembered or directly visible targets.
My counterexample or experiment: F4 Hidden pickup scene
Prediction, observed result, and one limitation:
- Prediction: The AI will not go for the nearer, but obstructed, pickup.
- Observed result: The AI ignores the blocked pickup and travels to the unobstructed one. Upon delivery, the AI enters explore mode until it gets line of sight of the obstructed pickup. The AI then travels to the pickup and gets stuck, since the depot is now obstructed.
- Limitation: The AI can get stuck if it enters an area where vision of the depot is blocked.

## Collaboration and help

People I discussed or playtested with: Max, Avdel, Alex
Code, sources or substantive tools/AI assistance I used, and how I verified them: Used the teacher repo to get an idea of what the correct implementation looks like.
If unfinished: what works, exact reproduction steps, and the help I need:
