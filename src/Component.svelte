<script lang="ts">
  import { getContext } from "svelte";

  type Row = Record<string, unknown>;

  type LineEvent = (payload: {
    lineData: Row;
    lineNumber: number;
  }) => unknown;

  export let dataProvider: { rows?: Row[] } | null = null;
  export let nextLineNumber: string = "";
  export let event: LineEvent | undefined = undefined;

  const { styleable, Provider } = getContext("sdk");
  const component = getContext("component");

  let currentLineNumber: number | null = null;
  let currentLineData: Row | null = null;

  let dataContext: {
    currentLineNumber: number | null;
    currentLineData: Row | null;
  } = {
    currentLineNumber: null,
    currentLineData: null
  };

  $: nextLineChanged(nextLineNumber, dataProvider?.rows ?? []);

  function nextLineChanged(value: string, rows: Row[]) {
    const text = String(value ?? "").trim();
    const index = Number(text);

    console.log("[ForEach] traitement", {
      value,
      index,
      rowCount: rows.length,
      eventType: typeof event
    });

    if (
      text === "" ||
      !Number.isInteger(index) ||
      index < 0 ||
      index >= rows.length
    ) {
      currentLineNumber = null;
      currentLineData = null;
      dataContext = { currentLineNumber, currentLineData };

      console.log("[ForEach] aucune ligne valide");
      return;
    }

    const row = rows[index];

    currentLineNumber = index;
    currentLineData = row;
    dataContext = { currentLineNumber, currentLineData };

    if (typeof event !== "function") {
      console.error("[ForEach] event n'est pas une fonction", event);
      return;
    }

    const payload = {
      lineData: row,
      lineNumber: index
    };

    console.log("[ForEach] appel de event", payload);

    try {
      Promise.resolve(event(payload)).catch(error => {
        console.error("[ForEach] échec asynchrone de event", error);
      });
    } catch (error) {
      console.error("[ForEach] échec de event", error);
    }
  }
</script>

<div use:styleable={$component.styles}>
  <Provider data={dataContext}>
    {#if currentLineNumber !== null}
      Currently processing line {currentLineNumber}.
    {:else}
      Not currently processing
    {/if}

    <slot />
  </Provider>
</div>
