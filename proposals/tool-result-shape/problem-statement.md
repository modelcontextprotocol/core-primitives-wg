# Tool Result Shape Problem Statement

**Contributors:** @kurtisvg  
**Status:** 💬 Seeking Feedback  
**Last Updated:** Oct 22, 2026

## Background

In the conventional conversational loop, tool results return to the model to interpret before it chooses the next action. **Programmatic Tool Calling (PTC)**, often called "code mode," lets the model write code that calls tools, processes their return values, and composes subsequent calls.

The execution environment consumes intermediate results first. The model-authored code can pass data between tools, save results to a file for later use, or emit selected fields or a computed summary into model context, subject to host controls. This lets the model choose what it needs to see for the current task without first reading every intermediate result.

For example, with `query` and `send_email` exposed as MCP tools, assume the wrappers return application values and surface tool failures:

```python
# Query users, then send each one a welcome email
users = query(sql="SELECT name, email FROM users WHERE welcomed = false")
for user in users:
    send_email(to=user["email"], subject="Welcome!", body="Hi " + user["name"])
print("Sent " + str(len(users)) + " welcome emails")
```

Here, the model chooses to print only the count. It could instead generate code that prints selected rows, or saves the full result to a CSV file and returns only the file path. Saving the records does not require placing their contents in model context.

## Problem

Today, MCP specifications, extensions, and host documentation provide conflicting guidance on how tool results should be handled and which audiences they target. Four aspects of the current result contract make this difficult for callers:

1. **`structuredContent` versus `content` mixes questions of format and audience.** MCP Apps guidance describes `structuredContent` as UI data outside model context, while OpenAI's Apps SDK documentation describes it as shared with the model and component. Core MCP also recommends mirroring structured data into a text block in `content`. A caller cannot infer the intended audience from the choice of field alone. [MCP Apps guidance](https://github.com/modelcontextprotocol/ext-apps/blob/298e884ec3f02daba085acdb02042d73bd00b355/specification/2026-01-26/apps.mdx), [OpenAI reference](https://developers.openai.com/plugins/reference#tool-results), [Core guidance](https://modelcontextprotocol.io/specification/2026-07-28/server/tools#structured-content)

2. **`outputSchema` is optional and covers only one representation.** The specification defines it as the schema for `structuredContent`, not the entire result or the payloads inside `content`. When declared, structured results must conform, but tools can omit both the schema and structured data. Even with a schema, callers still need to know which consumer that validated value is intended for. [Output schema](https://modelcontextprotocol.io/specification/2026-07-28/server/tools#output-schema)

3. **The result contains an array of content blocks, rather than a single application return value.** In the current **2026-07-28** contract, required `content` can mix text, images, audio, resource links, and embedded resources. These blocks have explicit protocol types, including text, but a text block's string may contain prose or serialized data without a declared application schema. Knowing that a block is text does not tell code how to extract records, fields, or other values from it. Optional `structuredContent` sits alongside this array. [Current schema](https://modelcontextprotocol.io/specification/2026-07-28/schema)

4. **Audience annotations add hints without establishing consistent routing.** Individual content blocks can carry `annotations.audience` for `user`, `assistant`, or both. These optional hints do not apply to the sibling `structuredContent` value, identify a programmatic consumer, or guarantee exclusion from model context. Servers cannot rely on them to establish a portable result-delivery contract across client harnesses. [Audience annotations](https://modelcontextprotocol.io/specification/2026-07-28/schema#annotations)

For code mode, there are two issues: **predictable application return values** and **explicit control over what enters model context**. The SQL example depends on `query` returning records with `name` and `email` strings. Its wrapper must know how to extract those records and handle failures, and the query's result contract and database schema must establish their shape.

Server audience hints are also less decisive when generated code chooses how to use a result. The same records may be passed to another tool, saved to a file, or selectively shown to the model. The result contract needs to support that task-specific choice within host policies.

## Requirements

- Make result selection and routing explicit, independently of format, while allowing model-authored code to choose what enters model context within host policies.
- Let clients discover programmatic return types before generating code and validate the application values returned to it.
- Define how programmatic callers consume existing tool and protocol error signals, including any typed error data.
- Define expectations for result stability and changes to declared schemas.
- Encourage typed tool results through SDK defaults, documentation, and validation, with a migration path for existing servers and clients.

## Open questions

- Should schemas be required for all tools, only for programmatic results, or remain optional with stronger SDK support?
- How should programmatic adapters expose return values, errors, multiple possible result shapes, and schema changes?
- How should clients handle tools without usable output schemas?
- Which audience decisions belong to server hints, host policy, and model-authored code?
- Can typing and audience changes ship independently?

## Related artifacts

- [Tool result specification](https://modelcontextprotocol.io/specification/draft/server/tools#tool-result): current fields, schema requirements, and text mirroring guidance.
- [SEP-3279: Negotiated tool result variants](https://github.com/krubenok/modelcontextprotocol/blob/e63642f043f2d621d1d11c19f07b1d581518aad0/seps/3279-negotiated-tool-result-variants.md): the reviewed draft of audience and format selection.
- [SEP-2200: Clarify tool result content visibility](https://github.com/modelcontextprotocol/modelcontextprotocol/pull/2200): earlier proposal to clarify model visibility using the existing fields.
- [PR #2847: Update structuredContent guidance for TextContent blocks](https://github.com/modelcontextprotocol/modelcontextprotocol/pull/2847): the Apps guidance conflict and a proposal to relax JSON mirroring.
- [SEP-2106: JSON Schema 2020-12 support](https://github.com/modelcontextprotocol/modelcontextprotocol/pull/2106): support for arrays, primitives, and composed schemas in tool outputs.
- [PR #371: Add outputSchema and structuredContent](https://github.com/modelcontextprotocol/modelcontextprotocol/pull/371) and [PR #559](https://github.com/modelcontextprotocol/modelcontextprotocol/pull/559): original design and subsequent compatibility changes.
- [Client behavior survey](https://gist.github.com/olaservo/3bb819673c444e4fe282c3af44a1ae01): an August 2026 survey of structured result handling and reliance on text mirrors.
