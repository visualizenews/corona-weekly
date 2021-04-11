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

  const chart = chrt.Chrt();

  const labelFormat = (d) => `Week ${new Date(data[id][d].startDay).getDate()}/${new Date(data[id][d].startDay).getMonth() + 1}`;

  onMount(() => {
    const max = Math.max(... data[id].map(d => d.value));
    const maxObj = data[id].find(d => d.value === max);
    
    console.log('CHART', id, data[id], max, maxObj);

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
      .x(null, null)
      .y([Math.min(0, ...data[id].map(d => d.value)), null]);
    if (position == 0) {
      chart.add(
        xAxis()
          .zero(0)
          .orient('top')
          .setTickPosition("outside")
          .setLabelPosition('outside')
          .class('axis-label')
          .format((d) => labelFormat(d))
          .color('#444')
          .width(2)
      ).add(
        xAxis()
          .zero(0)
          .orient('bottom')
          .hideTicks()
          .hideLabels()
          .color('#444')
          .width(2)
      );
    }
    if (position < charts - 1 && position > 0) {
      chart.add(
        xAxis()
          .zero(0)
          .orient('bottom')
          .hideTicks()
          .hideLabels()
          .color('#444')
          .width(2)
      )
    } 
    if (position == charts - 1) {
      chart.add(
        xAxis()
          .zero(0)
          .orient('bottom')
          .setTickPosition("outside")
          .setLabelPosition('outside')
          .class('axis-label')
          .format((d) => labelFormat(d))
          .color('#444')
          .width(2)
      );
    }

    chart.add(
      chrt
        .chrtColumns()
        .fill(d => ((d.value === max) ? 'url(#stripes2)' : 'url(#stripes)'))
        .color(d => ((d.value === max) ? '#a00' : '#444'))
        .strokeWidth(2)
        .width(.6)
        .data(data[id], (d, i) => ({
          x: i, //new Date(d.startDay),
          y: d.value,
          startDay: new Date(d.startDay),
          endDay: new Date(d.endDay),
        }))
    );

    chart.add(
      yAxis()
        .hideAxis()
        .hideLabels()
        .hideTicks()
        .add(yAxisRange()
          .dashed()
          .stroke('#a00')
          .strokeWidth(2)
          .from(max)
        )
    );
  });
</script>

<div class="chart" id="{id}" bind:this={el}>
  <div class="axis-label"><g><text /></g></div>
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
