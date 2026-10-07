# Subagent human controls and parent ownership

## Primary-source findings

- Claude Code's subagent documentation describes `/tasks` and a subagent/fork panel: inspect the transcript, send follow-ups directly to the viewed agent, and stop an individual agent. A child explicitly stopped by a human does not automatically resume from a model-authored message until the human resumes it. The documentation does not explicitly establish that direct human follow-ups are mirrored to the launcher parent, or define running-child behavior when `/resume` switches main sessions. [Subagents](https://code.claude.com/docs/en/sub-agents)
- Claude Code's agent view supports independent background sessions which keep running after leaving the view. Users can peek, reply, attach, detach, and stop sessions. This is a supervisor-backed independent-session feature, not evidence that ordinary subagents survive parent-session switching. [Agent view](https://code.claude.com/docs/en/agent-view)
- Pi durable's background subagent example binds each reporter to its originating conversation. Background children survive parent Esc. A report queued before Esc can be dropped with other queued input; a report arriving after Esc starts a new parent turn. Thus suppressing auto-wake after Esc is a deliberate difference in our proposed design, not a behavior copied from this example. The example's harness has separate conversation handles rather than a coding-agent `/resume` UI and does not establish a session-switch policy for our package. [Background example](https://github.com/earendil-works/pi/blob/main/packages/durable/test/examples/23-subagent-background.ts)

These are current documentation/source observations, not promises of universal behavior across products or versions.

## Design context pointers

The subsequent discussion settled parent-mediated human guidance, attributed human stops, and `instruct` as a provisional parent-to-child operation name. The complete agreement record lives in [Validate the Pi subagent design baseline](https://github.com/S2P2/pi-packages/issues/2), rather than in this research note.

For the remaining ownership, interruption, and report-delivery edge cases, work [Specify child ownership and report delivery boundaries](https://github.com/S2P2/pi-packages/issues/12). Final operation terminology belongs to [Choose delegation vocabulary and public operation contracts](https://github.com/S2P2/pi-packages/issues/15).
