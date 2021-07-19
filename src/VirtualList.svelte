<script>
  import { onMount } from 'svelte'

  export let getItem
  export let itemCount = 0
  export let itemHeight = 0
  let startIndex = -1
  let endIndex = -1
  let height = 0
  let scrollTop = 0
  let visibleItems = []

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

  function keydown(e) {
    if (e.key === ' ') return e.preventDefault()

    // console.log(viewport.scrollTop, viewport.scrollHeight)
    let prevent = true
    if (e.key === 'Home') viewport.scrollTop = 0
    else if (e.key === 'End') viewport.scrollTop = viewport.scrollHeight
    else if (e.key === 'PageUp') viewport.scrollTop -= viewport.clientHeight
    else if (e.key === 'PageDown') viewport.scrollTop += viewport.clientHeight
    else prevent = false
    if (prevent) e.preventDefault()
    // console.log(viewport.scrollTop, viewport.scrollHeight)
  }

  const buffer = 0
  function getStartIndex(scrollTop, itemHeight) {
    let topPixel = scrollTop
    let index = Math.floor(topPixel / itemHeight) - buffer
    return Math.max(0, index)
  }
  function getEndIndex(scrollTop, height, itemHeight, itemCount) {
    let bottomPixel = scrollTop + height
    let index = Math.ceil(bottomPixel / itemHeight) + buffer
    console.log(Math.min(itemCount, index))
    return Math.min(itemCount - 1, index)
  }

  $: if (mounted) updateView(scrollTop, height, itemHeight, itemCount)
  function updateView(scrollTop, height, itemHeight, itemCount) {
    const newStartIndex = getStartIndex(scrollTop, itemHeight)
    const newEndIndex = getEndIndex(scrollTop, height, itemHeight, itemCount)

    let newVisibleItems = []
    for (let i = newStartIndex; i <= newEndIndex; i++) {
      if (i >= startIndex && i <= endIndex) {
        newVisibleItems.push(visibleItems[i - startIndex])
      } else {
        newVisibleItems.push(getItem(i))
      }
    }
    visibleItems = newVisibleItems
    startIndex = newStartIndex
    endIndex = newEndIndex
  }

  export async function refresh() {
    if (mounted) {
      startIndex = -1
      endIndex = -1
      visibleItems = []
      // we need to wait a tick so properties can finish updating
      await tick()
      updateView(scrollTop, height, itemHeight, itemCount)
    }
  }
</script>

<div
  class="viewport"
  bind:this={viewport}
  bind:clientHeight={height}
  on:scroll={handleScroll}
  on:dragleave
  on:keydown
  on:keydown={keydown}
  tabindex="0">
  <div
    class="content"
    style="height: {itemCount * itemHeight}px; padding-top: {startIndex * itemHeight}px;">
    {#each visibleItems as item, i}
      <slot {item} index={startIndex + i} />
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
  .content {
    box-sizing: border-box;
  }
</style>
