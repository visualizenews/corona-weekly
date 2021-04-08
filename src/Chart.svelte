<script>
  import { onMount } from "svelte";
  
  import * as chrt from "chrt";
  import {yAxis, xAxis, xAxisRange, yAxisRange} from 'chrt-axis';
  
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
    console.log("onMount", position, charts);
    const max = data[id].reduce((a,d) => ((d.value > a) ? d.value : a), Number.MIN_SAFE_INTEGER);
    const line = data[id].map((d) => ({
      startDay: d.startDay,
      value: max,
    }));

    chart
      .node(el)
      .svg()
      .size(W, H)
      .margins({
        bottom: 30,
        left: 0,
        right: 0,
      })
      .padding({
        left: 30,
        right: 30,
        top: 30,
      })
      .x(null, null, { scale: "time" })
      .y([Math.min(0, ...data[id].map(d => d.value)), null]);
    if (position == 0) {
      chart.add(
        xAxis()
          .zero(0)
          .orient('top')
          .interval("month")
          .setTickPosition("outside")
          .setLabelPosition('outside')
          .format(d => (MONTHS[d.getMonth()]))
      ).add(
        xAxis()
          .zero(0)
          .orient('bottom')
          .interval("month")
          .hideTicks()
          .format(d => (''))
      );
    }
    if (position < charts - 1 && position > 0) {
      chart.add(
        xAxis()
          .zero(0)
          .orient('bottom')
          .interval("month")
          .hideTicks()
          .format(d => (''))
      )
    } 
    if (position == charts - 1) {
      chart.add(
        xAxis()
          .zero(0)
          .orient('bottom')
          .interval("month")
          .setTickPosition("outside")
          .setLabelPosition('outside')
          .format(d => (MONTHS[d.getMonth()]))
      );
    }

    chart.add(
      chrt
        .chrtColumns()
        .fill(d => ((d.value === max) ? 'red' : 'silver'))
        .color('transparent')
        .strokeWidth(0)
        .width(1)
        .data(data[id], (d) => ({
          x: new Date(d.startDay),
          y: d.value,
        }))
    );

    // chart.add(
    //   chrt
    //     .chrtLine()
    //     .color('#444')
    //     .data(line, (d) => ({
    //       x: new Date(d.startDay),
    //       y: d.value,
    //     }))
    //     .add(chrt.chrtLabel(label))
    // );

    chart.add(
      yAxis()
        .hideAxis()
        .hideLabels()
        .hideTicks()
        .add(yAxisRange()
          .dashed()
          .from(max)
        )
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
