<script>
  import { onMount } from "svelte";
  
  import * as chrt from "chrt";
  import {yAxis, xAxis, xAxisRange} from 'chrt-axis';
  
  export let data;
  export let id;
  export let label;

  const W = 750;
  const H = 175;

  let el;

  console.log('CHART, data', id, label, data);

  const chart = chrt.Chrt();

  onMount(() => {
    console.log("onMount");

    chart
      .node(el)
      .svg()
      .size(W, H)
      .margins({
        left: 50,
        right: 50,
      })
      .padding({
        left: 30,
        right: 30,
      })
      //.y([0,50])
      .x([data[id][0], data[id][data[id].length - 1]], null, { scale: "time" })
      .add(
        yAxis(3)
          .orient('left')
          .setTickPosition('outside')
          .setLabelPosition('outside')
          .format(d => d)
      )
      .add(
        xAxis(10)
          .orient('bottom')
          .interval("month")
          .setTickPosition("outside")
          .setLabelPosition('outside')
          .format(d => (`${d.getDate()}/${d.getMonth()+1}`))
      )
      .add(
        chrt
          .chrtBars()
          .fill('red')
          .color('#444')
          .strokeWidth(1)
          .width(1)
          .data(data[id], (d) => ({
            x: d.startDay,
            y: d.value,
          }))
          .color("#f00")
          .add(chrt.chrtLabel(label))
      );

  });
</script>

<div class="chart" id="{id}" bind:this={el}>
</div>

<style>
  .chart {
    background: #1fc;
    display: block;
    height: 100%;
    margin: 0;
    padding: 0;
    width: 100%;
  }
</style>
