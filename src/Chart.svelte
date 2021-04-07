<script>
  import { onMount } from "svelte";
  
  import * as chrt from "chrt";
  import {yAxis, xAxis, xAxisRange} from 'chrt-axis';
  
  export let data;
  export let id;
  export let label;
  export let charts;
  export let position;

  const W = 750;
  const H = 175;
  const MONTHS = ['Jan.', 'Feb.', 'Mar.', 'Apr.', 'May', 'Jun.', 'Jul.', 'Aug.', 'Sep.', 'Oct.', 'Nov.', 'Dec.'];

  let el;

  console.log('CHART, data', id, label, data);

  const chart = chrt.Chrt();

  onMount(() => {
    console.log("onMount", Math.min(...data[id].map(d => d.value)));

    chart
      .node(el)
      .svg()
      .size(W, H)
      .margins({
        bottom: 20,
        left: 0,
        right: 0,
      })
      .padding({
        left: 30,
        right: 30,
        top: 30,
      })
      .x(null, null, { scale: "time" })
      .y([Math.min(0, ...data[id].map(d => d.value)), null])
      .add(
        xAxis()
          .zero(0)
          .orient(position == 0 ? 'top' : 'bottom')
          .interval("month")
          .setTickPosition("outside")
          .setLabelPosition('outside')
          .format(d => (`${MONTHS[d.getMonth()]}`))
      )
      .add(
        chrt
          .chrtColumns()
          .fill('#aaa')
          .color('#444')
          .strokeWidth(1)
          .width(1)
          .data(data[id], (d) => ({
            x: new Date(d.startDay),
            y: d.value,
          }))
          .add(chrt.chrtLabel(label))
      );

  });
</script>

<div class="chart" id="{id}" bind:this={el}>
</div>

<style>
  .chart {
    display: block;
    height: 100%;
    margin: 0;
    padding: 0;
    width: 100%;
  }
</style>
