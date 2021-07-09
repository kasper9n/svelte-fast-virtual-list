<script>
  import { onMount } from 'svelte'

  export let getItem
  export let itemCount = 0
  export let itemHeight = 0
  let startIndex = 0
  let endIndex = 0
  let height = 0
  let scrollTop = 0
  let visibleIndexes = []
  let visibleCount = 0

  let viewport
  function handleScroll(e) {
    const target = e.target
    scrollTop = target.scrollTop
  }

  let mounted = false
  onMount(() => {
    scrollTop = viewport.scrollTop
    mounted = true
  })

  function getStartIndex(scrollTop, itemHeight) {
    let topPixel = scrollTop - itemHeight * 5
    if (topPixel < 0) topPixel = 0
    return Math.floor(topPixel / itemHeight)
  }
  function getEndIndex(scrollTop, height, itemHeight, itemCount) {
    const newHeight = itemCount * itemHeight
    let bottomPixel = scrollTop + height + itemHeight * 5
    if (bottomPixel > newHeight) bottomPixel = newHeight
    return Math.ceil(bottomPixel / itemHeight)
  }

  $: if (mounted) updateView(height, itemHeight, itemCount)
  function updateView(height, itemHeight, itemCount) {
    startIndex = getStartIndex(scrollTop, itemHeight)
    endIndex = getEndIndex(scrollTop, height, itemHeight)

    let newIndexes = []
    for (let i = 0; i < endIndex - startIndex; i++) {
      newIndexes.push(startIndex + i)
    }
    visibleIndexes = newIndexes
    visibleCount = visibleIndexes.length
  }

  $: if (mounted) updateItems(scrollTop)
  function updateItems(scrollTop) {
    let newStartIndex = getStartIndex(scrollTop, itemHeight)
    let newEndIndex = startIndex + visibleCount

    if (endIndex < newStartIndex || startIndex < newEndIndex) {
      updateView(height, itemHeight, itemCount)
      return
    }
    startIndex = newStartIndex
    endIndex = newEndIndex

    for (let i = 0; i < visibleIndexes.length; i++) {
      if (visibleIndexes[i] < startIndex) {
        visibleIndexes[i] += visibleCount
      } else if (visibleIndexes[i] > endIndex) {
        visibleIndexes[i] -= visibleCount
      }
    }
    // visibleIndexes = visibleIndexes
  }
</script>

<div
  class="viewport"
  bind:this={viewport}
  bind:clientHeight={height}
  on:scroll={handleScroll}
  on:dragleave
  on:keydown
  tabindex="0">
  <div class="content" style="height: {itemCount * itemHeight}px;">
    {#each Array(visibleCount) as _, i}
      <slot
        item={getItem(visibleIndexes[i])}
        index={visibleIndexes[i]}
        pos={visibleIndexes[i] * itemHeight} />
    {/each}
  </div>
</div>

<style>
  .viewport {
    height: 100%;
    overflow-y: scroll;
    outline: none;
    background-color: inherit;
    border: 1px solid black;
  }
</style>
