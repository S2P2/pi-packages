# Pi Packages

Shared language for Pi packages in this workspace.

## Language

**Subagent**:
An agent to which a parent agent delegates work, with its own conversation context.
_Avoid_: Skill (skills supply instructions rather than a separate agent)

**Delegation**:
Assigning work to a subagent while retaining responsibility for how its result contributes to the parent’s goal.
_Avoid_: Workflow (a workflow determines which work to assign and how to coordinate it)
