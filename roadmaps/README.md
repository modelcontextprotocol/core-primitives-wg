# Core Primitives Working Group Roadmap

The initial work areas below follow the **Improved Primitives** priority area in
the [MCP roadmap](https://modelcontextprotocol.io/development/roadmap#4-improved-primitives).
They provide a starting point for WG discussion; detailed proposals, owners, and
release targets will be recorded as the work develops.

| Work area | Goal | Topic label |
| --- | --- | --- |
| Tool result shape | Make `tools/call` results consistent across structured and unstructured output. | `roadmap:tool-result-shape` |
| Progressive discovery | Let clients learn a server's tools and resources as needed, with defined interactions with caching. | `roadmap:progressive-discovery` |
| Primitive annotations | Evaluate audience and priority annotations, clarify useful semantics, and consider deprecation where they do not serve implementers. | `roadmap:primitive-annotations` |

Discovery work needs coordination with the
[Progressive Disclosure WG](https://modelcontextprotocol.io/community/working-groups/progressive-disclosure)
and the [Transports WG](https://github.com/modelcontextprotocol/transports-wg),
particularly around discovery and caching behavior.

Keep problem statements and proposals under [`proposals/`](../proposals/). As
release plans are agreed, add dated roadmap documents here and update the link in
the root [README](../README.md).

Topic labels identify related work and do not imply release commitments or
priority. They are managed in [`.github/labels.yml`](../.github/labels.yml),
previewed on pull requests, and synchronized on `main`. Existing labels omitted
from that file are preserved.
