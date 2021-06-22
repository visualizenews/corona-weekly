<script>
  import { onMount } from "svelte";
  
  import * as chrt from "chrt";
  import {yAxis, xAxis, yAxisRange} from 'chrt-axis';
  import { chrtAnnotation } from "chrt-annotation";
  
  export let data;
  export let id;
  export let label;
  // export let title;
  export let charts;
  export let position;

  const W = 750;
  const H = 150;
  const MONTHS = ['Jan.', 'Feb.', 'Mar.', 'Apr.', 'May', 'Jun.', 'Jul.', 'Aug.', 'Sep.', 'Oct.', 'Nov.', 'Dec.'];

  let el;

  const chart = chrt.Chrt();

  const labelFormat = (d) => `${d === 0 ? 'Week' : 'W.'} ${new Date(data[id][d].startDay).getMonth() + 1}/${new Date(data[id][d].startDay).getDate()}`;

  onMount(() => {
    const max = Math.max(... data[id].map(d => d.value));
    const maxObj = data[id].find(d => d.value === max);
    const maxIndex = data[id].findIndex(d => d.value === max);
    const min = Math.min(... data[id].map(d => d.value));
    const minObj = data[id].find(d => d.value === min);
    const minIndex = data[id].findIndex(d => d.value === min);
    
    // console.log('CHART', id, data[id], max, maxObj);

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

    chart.add(
      chrt
        .chrtColumns()
        .fill(d => ((d.value === max) ? 'url(#stripes2)' : ((d.value === min) ? 'url(#stripes3)' : 'url(#stripes)')))
        .color(d => ((d.value === max) ? '#444' : '#444'))
        .strokeWidth(1)
        .width(.6)
        .data(data[id], (d, i) => ({
          x: i, //new Date(d.startDay),
          y: d.value,
          startDay: new Date(d.startDay),
          endDay: new Date(d.endDay),
        }))
    );
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
          .width(1)
      ).add(
        xAxis()
          .zero(0)
          .orient('bottom')
          .hideTicks()
          .hideLabels()
          .color('#444')
          .width(1)
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
          .width(1)
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
          .width(1)
      );
    }

    chart.add(
      yAxis()
        .hideAxis()
        .hideLabels()
        .hideTicks()
        .add(yAxisRange()
          .dashed()
          .stroke('#444')
          .strokeWidth(1)
          .from(max)
        )
        // ).add(
        //   chrt.chrtAxisTitle(title)
        //   .class('title')
        //   .offset({ x:5, y:-5 })
        //   .align('right')
        // )
    );

    chart.add(
      chrtAnnotation(`<div>${label}: ${new Intl.NumberFormat('en-EN').format(maxObj.value)}</div>`)
      .top(max)
      .left(maxIndex)
    );

    chart.add(
      chrtAnnotation(`<div>${new Intl.NumberFormat('en-EN').format(data[id][data[id].length -1].value)}</div>`)
      .top(data[id][data[id].length -1].value)
      .left(data[id].length -1)
      .class('thisWeek')
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
