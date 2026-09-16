- Feature Name: explore_genui_mcp
- Start Date: 2026-09-15

# Summary

Uyuni is introducing AI-driven interactions through the Model Context Protocol (MCP), allowing an AI assistant to interact with Uyuni tools and data through a conversational interface.

This RFC proposes exploring Generative UI (GenUI) as a complementary way to present the results of these interactions.

The main use case is an AI assistant combining the outputs of multiple MCP tools. These tools may belong to the Uyuni MCP Server or to different MCP servers. Since the resulting combinations may not be known in advance, there may not be a predefined UI to represent them.

One approach we want to explore is allowing the AI to compose a UI using a finite catalog of predefined components. This RFC refers to this approach as a **Controlled Component Catalog**.

This RFC does not propose adding GenUI to Uyuni. Its purpose is to document the problem, explore possible approaches, and identify the questions that need to be answered before deciding whether GenUI would be useful for Uyuni.

# Motivation

For known workflows and known data structures, Uyuni can provide predefined user interfaces.

AI assistants introduce another type of interaction. A user request can cause the assistant to call several tools and combine their results.

These combinations may occur within the Uyuni MCP Server. For example, an assistant could combine information about systems, available patches, and pending actions.

They may also involve different MCP servers. For example, an assistant could combine information from Uyuni with monitoring data or information from an issue tracker.

The final result in these cases depends on the user's request and on the tools selected by the AI. It may therefore represent a combination of data that was not known when the frontend was designed.

A textual answer, Markdown, or a generic table may be sufficient in many cases. The question this RFC proposes to explore is whether some of these results would benefit from a UI composed dynamically for the data being returned.

The goal is not to replace existing Uyuni screens with generated interfaces. The scope of this exploration is the presentation of results produced from combinations of MCP tools for which a specific UI has not been defined in advance.

# Detailed design

## Initial exploration

An undergraduate thesis explored several approaches to GenUI using the Uyuni MCP Server. The approaches included generating HTML with an LLM and using CopilotKit.

The work showed that it is possible to generate a graphical representation from the results of MCP tools. It also raised questions about how such interfaces could be constrained, tested, and integrated with the Uyuni frontend.

The thesis was an initial exploration. It did not evaluate production aspects such as latency, token usage, failure rates, or user task completion.

These aspects would need to be evaluated before making an architectural decision.

## Controlled Component Catalog

One approach to explore is a finite catalog of predefined UI components.

Instead of asking an LLM to generate arbitrary HTML or frontend code, the LLM would produce structured output describing which components should be displayed and the data provided to them.

For example:

```text
User request
    |
    v
AI assistant
    |
    v
MCP tools
    |
    v
Combined result
    |
    v
Component description
    |
    v
Predefined UI components
    |
    v
Rendered result
```

The component description could use a structured format such as JSON and would need to conform to a defined schema.

The frontend would implement the available components. The generated description could only refer to components and properties supported by the catalog.

An initial experiment could use a small number of components, for example:

* Data tables
* Status summaries
* Time-series charts

The purpose of the experiment would be to determine whether this approach provides useful representations for results that combine multiple MCP tools.

It would also allow us to evaluate how much control and determinism can be achieved by limiting the set of available components.

## Next steps

Before considering an implementation in Uyuni, the following exploratory work could be done:

* Identify concrete MCP use cases where a generated UI may provide value over Markdown or a predefined UI.
* Build small prototypes.
* Compare a component catalog with simpler representations such as Markdown.
* Evaluate existing GenUI frameworks and MCP UI mechanisms.
* Measure latency, token usage, and invalid component descriptions.
* Investigate how dynamically composed interfaces could be tested.

A larger prototype could later be considered if the initial experiments justify further work.

# Drawbacks

* **Additional complexity:** A component catalog requires a schema, validation, component resolution, and integration with the existing frontend.

* **Non-determinism:** If an LLM selects the components, the same or similar data may result in different representations.

* **Testing:** Testing individual components is deterministic, but testing which components the LLM selects and how they are combined requires a different approach.

* **Invalid output:** The generated component description may contain unknown components, invalid properties, or combinations that cannot be rendered.

* **Cost and latency:** Generating a component description requires an additional LLM output. The impact on latency and token usage needs to be measured.

* **Unclear user value:** It is not yet known whether dynamically composed interfaces provide enough value over Markdown, tables, or other generic representations to justify the additional complexity.

# Alternatives

## Markdown

The assistant can return Markdown using tables, lists, code blocks, and other supported elements.

This requires no GenUI-specific architecture and may be sufficient for many use cases.

The exploration should identify cases where a dynamically composed UI provides a useful improvement over this representation.

## Traditional predefined UI

For known workflows, a predefined UI can be implemented as part of the Uyuni frontend.

This remains an option whenever the workflow and the data to be presented are known in advance.

It does not, however, provide a specific UI for arbitrary combinations of MCP tool results.

## Generated HTML

The LLM can generate HTML directly from the tool results.

This was one of the approaches explored in the undergraduate thesis and provides a useful reference point for comparison.

Allowing generated HTML would require addressing how the generated content is validated, rendered, tested, and integrated with the existing frontend.

## GenUI frameworks

Existing GenUI frameworks such as CopilotKit can be evaluated instead of implementing the complete mechanism in Uyuni.

CopilotKit was one of the approaches evaluated in the initial undergraduate project.

Further exploration would be needed to understand how such frameworks would integrate with the Uyuni frontend and whether they fit the use cases described in this RFC.

## MCP UI mechanisms

UI mechanisms in the MCP ecosystem, including MCP Apps, should also be evaluated.

The goal is to understand whether existing MCP mechanisms already address some or all of the problem described by this RFC before implementing an Uyuni-specific solution.

# Unresolved questions

The following questions should be answered through the exploration:

* **Use cases:** What concrete combinations of MCP tool outputs benefit from a generated graphical representation instead of Markdown or a generic table?

* **Component selection:** Should the LLM select the components directly, or should some part of the selection be deterministic?

* **Schema:** What information is needed to describe a UI composed from predefined components? How small can this schema remain while still covering useful cases?

* **Layout:** Is selecting components enough, or does the generated description also need to describe their layout?

* **Interactivity:** Should the initial scope only display information, or should generated interfaces also support actions?

* **Testing:** How can we test both the validity of the generated component description and whether the selected representation is appropriate for the data?

* **Fallback:** What should happen when the generated component description cannot be rendered? One option to evaluate is falling back to Markdown or structured data.

* **Integration:** Can the catalog reuse existing Uyuni frontend components, and what changes would be required to expose them through such a mechanism?

* **Performance:** What latency and token cost does generating the component description add?

* **Existing solutions:** Can an existing GenUI framework or MCP UI mechanism solve the problem without introducing an Uyuni-specific mechanism?
