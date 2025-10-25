Svelte Virtual Table
=======================

A virtual table component for Svelte. It only renders the data that is visible on screen. This means you can scroll
through millions of rows without any perf issue.

This is possibly the only virtual list that can handle the scrollable height larger than 16,777,200 pixels, which is
the Google Chrome's limit for a div's height and padding. Other virtual lists would break down due to this limit. 16M 
in pixels isn't as high as we think. 1,000,000 rows whose row height is 20px is already 20,000,000px in height. 
Read more about how we circumvent this limit here: [Improving a virtual list and overcoming Chrome's limitation](some url)

Here are the improvements:

1. Support as many rows as your browser's memory allows.
2. Support variable height for each row.
3. Support optional sticky top rows and sticky left columns.
4. Support the detection of reaching the bottom. This is useful for supporting a load-more mechanism.
5. Support wider rows with a horizontal scrollbar.
6. Support saving the scroll positions on both axes in the case where you want to restore the previous scroll positions.
7. Support row index.

The caveats:
* You must provide the exact height of *every* row and the exact width of *every* column. This can be easily done using `canvas` and `measureText`.
* The row height can be changed but needs a full refresh.
* The row height and column width should be whole numbers. In the case of >500,000 rows, we've encountered a rendering issue if the numbers aren't whole.

The test page generates 20,000,000 rows with a single column. The scrolling is smooth on Mac M4.

This package is inspired by https://github.com/sveltejs/svelte-virtual-list (@sveltejs/svelte-virtual-list). It was
initially built for [Backdoor](https://github.com/tanin47/backdoor), A Postgres Data Querying and Editing Tool that you can embed into your JVM app.

Installation
-------------

`npm install @tanin/svelte-virtual-table`

Usage
------

You can see several examples in the test folder, `./test`. The gallery is in `./test/App.svelte`.

```sveltehtml
<script lang="ts">
import VirtualTable, {type Item} from "../src/VirtualTable.svelte";

let scrollLeft: number = 0;
let scrollTop: number = 0;

const columns: any[] = [
  {name: 'Username', width: 100},
  {name: 'Address', width: 350}
]

const items: Item[] = Array.from({length: 80}, (_, i) => ({
  rowHeight: getRowHeight(i),
  values: [
    `User ${i + 1}`,
    `Address ${i + 1}, Street ${i + 1}, City ${i + 1}`,
  ]
}));

let startIndex: number = 0;
let endIndex: number = 0;

function getRowHeight(rowIndex: number) {
  // Simulate variable heights
  if ((rowIndex % 10) === 0) {
    return 28 * 2;
  } else {
    return 28;
  }
}
</script>

<div class="container">
  <div>Indices: {startIndex} - {endIndex}, total: {items.length}</div>
  <VirtualTable
    let:item
    let:index
    bind:startIndex
    bind:endIndex
    items={items}
    initialScrollLeft={scrollLeft}
    initialScrollTop={scrollTop}
    onBottomReached={() => {
      console.log("onBottomReached")
    }}
    onScrolled={(scrollLeft_, scrollTop_) => {
      scrollLeft = scrollLeft_;
      scrollTop = scrollTop_;
    }}
  >
    <div slot="header" class="header">
      {#each columns as column, index (index)}
        <div style="min-width: {column.width}px;width: {column.width}px;max-width: {column.width}px;">{column.name}</div>
      {/each}
    </div>
    <div class="row" style="height: {item.rowHeight}px;">
      {#each columns as column, colIndex (colIndex)}
        <div style="min-width: {column.width}px;width: {column.width}px;max-width: {column.width}px;">
          {item.values[colIndex]}
        </div>
      {/each}
    </div>
  </VirtualTable>
</div>

<style>
.container {
  height: 400px;
  width: 600px;
  display: flex;
  flex-direction: column;
  align-items: stretch;
  justify-content: stretch;
  border: 2px solid #333;
  position: relative;
  margin: 10px;
}

.header {
  border-left: 1px #ccc solid;
  font-size: 16px;
  display: flex;
  font-weight: bold;
  white-space: pre;
}

.header > div {
  display: block;
  box-sizing: border-box;
  padding: 4px;
  background: #eee;
  border-top: 1px #ccc solid;
  border-right: 1px #ccc solid;
  border-bottom: 1px #ccc solid;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: pre;
}

.row {
  font-size: 16px;
  display: flex;
  border-left: 1px #ccc solid;
}

.row > div {
  box-sizing: border-box;
  padding: 4px;
  border-right: 1px #ccc solid;
  border-bottom: 1px #ccc solid;
  white-space: pre;
  overflow: hidden;
  text-overflow: ellipsis;
}
</style>
```

Development
------------

1. Run `npm install`
2. Run `npm run rollup` and visit `./test/index-rollup.html`
4. Run `npm run webpack` and visit `./test/index-webpack.html` to test webpack.

Test
-----

1. Scrolling up to the top and seeing the first row
2. Scrolling down to the bottom and see the last row.
3. Drag the scrollbar thumb to the bottommost. We should not see it flicker.
