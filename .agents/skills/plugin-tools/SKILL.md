---
name: plugin-tools
description: Add formulas, actions, indexed sync tables, or skills to a Superhuman plugin. Use when editing these capabilities or improving tool discovery and data retrieval in an existing plugin.
---

# Add plugin tools and data

Use the installed SDK types and the [SDK reference](https://docs.superhuman.com/packs/build/latest/reference/sdk/) to check available fields. The human-facing term is plugin. The exported definition remains `pack`.

## Tools

In Docs, users call formulas in the formula editor and run actions through buttons or automations. In Go, the agent selects the same formulas as tools from their names and descriptions. Use this shared authoring workflow for both surfaces.

Use `pack.addFormula()` for a read operation. Set `isAction: true` for an operation that changes external data. Give tools concrete names and descriptions that explain when to use them. Describe parameter formats and allowed values where the model can read them.

```ts
pack.addFormula({
  name: "ExchangeRate",
  description: "Get the exchange rate between two ISO 4217 currency codes.",
  parameters: [
    sdk.makeParameter({
      type: sdk.ParameterType.String,
      name: "base",
      description: "Base currency code, such as USD.",
    }),
    sdk.makeParameter({
      type: sdk.ParameterType.String,
      name: "target",
      description: "Target currency code, such as EUR.",
    }),
  ],
  resultType: sdk.ValueType.Number,
  execute: async function ([base, target], context) {
    if (base === target) {
      return 1;
    }
    let response = await context.fetcher.fetch({
      method: "GET",
      url: "https://api.frankfurter.dev/v1/latest" +
        `?base=${encodeURIComponent(base)}` +
        `&symbols=${encodeURIComponent(target)}`,
    });
    return response.body.rates[target];
  },
});
```

This example requires `pack.addNetworkDomain("frankfurter.dev")` and an `sdk` import. Surface useful errors with `sdk.UserVisibleError`. Check remote response shapes before using their values.

Go cannot use parameter autocomplete as an enumeration or supply varargs. Describe short value lists in the parameter description. For variable arguments, expose a formula with a supported parameter shape. Use `isExperimental` only after checking its availability and behavior in the installed SDK.

## Indexed data

Use `pack.addSyncTable()` with a schema that defines stable IDs, a display property, and a link property. Go retrieves indexed content, so configure `index.properties` for long text. Use `index.contextProperties` for short fields needed to interpret each chunk and `index.filterableProperties` for useful filters. Check current limits in the SDK reference.

In Docs, users add a sync table to import rows into a document; indexing is needed for Go retrieval. Test the table in each surface the plugin targets.

Use continuations to page remote results. A local sync execution proves the returned rows, but indexing and retrieval need a test in Go. For updates from Go, expose an action rather than relying on two-way sync.

## Plugin skills

Plugin skills apply to Go. Docs users use the formulas, actions, and sync tables directly.

Use `pack.addSkill()` for reusable instructions that help Go choose or combine the plugin's tools. This is a skill supplied by the user's Superhuman plugin, distinct from the coding-agent skills in this bundle. Check the [Go authoring guide](https://docs.superhuman.com/packs/build/latest/agents/) and installed SDK types for the supported fields. Verify routing in Go after upload.

Before completing a tool or data change, follow the [local checks](../plugin-test-upload/SKILL.md#local-checks) and require successful validation and build, or report the specific blocker. Upload only when the user requests it.
