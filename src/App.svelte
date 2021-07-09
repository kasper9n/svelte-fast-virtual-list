<script>
  import VirtualList from './VirtualList.svelte'
  let itemCount = 100
  function getItem(index) {
    console.log('getItem')
    let x = index
    for (let i = 0; i < 10_000_000; i++) {
      x++
    }
    return { num: index, x }
  }
</script>

<input type="text" bind:value={itemCount} />
<div class="stuff" style="height: 80vh">
  <VirtualList {getItem} {itemCount} itemHeight={24} let:item={track} let:pos>
    <div class="item" style="top: {pos}px;">num {track.num}, {track.x}, {pos}</div>
  </VirtualList>
</div>
<style>
  .item {
    position: absolute;
    font-family: sans-serif;
  }
</style>
