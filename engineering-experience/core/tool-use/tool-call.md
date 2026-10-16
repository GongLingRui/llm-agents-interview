every modern agent harness has 2 categories of tools.

Prebuilt tools which ship with the harness（read,write,edit,bash,grep.ls,find)

External tools defined by user during design time runtime


```typescript
export interface ToolDefinition<TParams, TDetails> {
    name: string;                // used in the model's tool calls
    label: string;               // human-readable, for the UI
    description: string;        // the description shown to the LLM
    parameters: TParams;         // a TypeBox schema
    execute(
        toolCallId: string,
        params: Static<TParams>,
        signal: AbortSignal | undefined,
        onUpdate: AgentToolUpdateCallback<TDetails> | undefined,
        ctx: ExtensionContext,
    ): Promise<AgentToolResult<TDetails>>;
    // optional: prepareArguments, renderCall, renderResult, executionMode ...
}
```
