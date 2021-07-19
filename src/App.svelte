<script>
  import VirtualList from './VirtualList.svelte'
  let itemCount = 100
  let rowLoadingMs = 0
  function sleep(delay) {
    var start = new Date().getTime()
    let counter = 0
    while (new Date().getTime() < start + delay) {
      counter++
    }
    return counter
  }
  function getItem(index) {
    return { num: index, x: sleep(rowLoadingMs) }
  }
  let refresh
</script>

<label>
  Item count:
  <input type="number" bind:value={itemCount} />
</label>
<button on:click={refresh}>Refresh</button>
<label>
  Row loading time (ms):
  <input type="number" bind:value={rowLoadingMs} />
</label>

<div class="stuff" style="height: 80vh">
  <VirtualList {getItem} {itemCount} itemHeight={24} bind:refresh let:item={track} let:index>
    <div class="item" class:odd={index % 2 === 0}>num {track.num}, {track.x}</div>
  </VirtualList>
</div>

<style>
  .item {
    font-family: sans-serif;
    height: 24px;
  }
  .odd {
    background-color: #eee;
  }
</style>
