<script>
  import { onMount } from "svelte";
  
  import * as chrt from "chrt";
  import {yAxis, xAxis, yAxisRange, xAxisRange} from 'chrt-axis';
  import { chrtAnnotation } from "chrt-annotation";
  
  export let data2020;
  export let data2021;
  export let id;
  export let label;
  export let charts;
  export let position;

  const W = 950;
  const H = position === 0 ? 250 : (position === 4 ? 180 : 150);
  const labelFormat = (d) => ((d > 0 && d <= 50) ? `Week ${d}` : null);

  
  let el;
  const chart = chrt.Chrt();

  onMount(() => {
    const max = Math.max(... data2020[id].map(d => d.value), ... data2021[id].map(d => d.value));
    const maxObj = data2020[id].find(d => d.value === max) || data2021[id].find(d => d.value === max);
    const maxIndex = data2020[id].findIndex(d => d.value === max) || data2021[id].findIndex(d => d.value === max);
    const min = Math.min(... data2020[id].map(d => d.value), ... data2021[id].map(d => d.value));
    const minObj = data2020[id].find(d => d.value === min) || data2021[id].find(d => d.value === min);
    const minIndex = data2020[id].findIndex(d => d.value === min) || data2021[id].findIndex(d => d.value === min);
    
    chart
      .node(el)
      .svg()
      .size(W, H)
      .margins({
        bottom: (position === 4) ? 30 : 0,
        left: 0,
        right: 0,
        top: (position === 0) ? 30 : 0,
      })
      .padding({
        left: 10,
        right: 10,
        top: (position === 0) ? 60 : 40,
      })
      .x([1, 53])
      .y([Math.min(0, ...data2020[id].map(d => d.value), ...data2021[id].map(d => d.value)), null]);

    chart.add(
      chrt
        .chrtLine()
        .area()
        .fill('url(#dots)')
        .color('#ababab')
        .width(1)
        .data(data2020[id], (d, i) => ({
          x: d.week,
          y: d.value,
        }))
    );


    chart.add(
      chrt
        .chrtLine()
        .fill('transparent')
        .color('#444')
        .width(2)
        .data(data2021[id], (d, i) => ({
          x: d.week,
          y: d.value,
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
    );

    chart.add(
      xAxis()
        .hideAxis()
        .hideLabels()
        .hideTicks()
        .add(xAxisRange()
          .dashed()
          .stroke('#ababab')
          .strokeWidth(1)
          .from(21)
        )
        .add(xAxisRange()
          .dashed()
          .stroke('#ababab')
          .strokeWidth(1)
          .from(9)
        )
        .add(xAxisRange()
          .dashed()
          .stroke('#ababab')
          .strokeWidth(1)
          .from(45)
        )
        .add(xAxisRange()
          .dashed()
          .stroke('#ababab')
          .strokeWidth(1)
          .from(53)
        )
    );

    chart.add(
      chrtAnnotation(`<div>${label}:<br />${new Intl.NumberFormat('en-EN').format(maxObj.value)} Week of ${new Date(maxObj.datetime).getMonth() + 1}/${new Date(maxObj.datetime).getDate()}/${new Date(maxObj.datetime).getFullYear()}</div>`)
      .top(max)
      .left(maxObj.week)
      .class('marker')
      .class(position === 1 ? 'reverse' : 'normal')
    );

    if (position === 0) {
      chart.add(
        chrtAnnotation(`<div>Week of 5/18/2020:<br />End of Lockdown</div>`)
        .top(max / 2)
        .left(21)
        .class('milestone')
      );
      chart.add(
        chrtAnnotation(`<div>Week of 2/24/2020:<br />First Official<br />Covid Case in Italy</div>`)
        .top(max)
        .left(9)
        .class('milestone')
      );
      chart.add(
        chrtAnnotation(`<div>Week of 11/2/2020:<br />Lockdowns are now<br />Region-Based</div>`)
        .top(max / 2)
        .left(45)
        .class('milestone-2')
      );
      chart.add(
        chrtAnnotation(`<div>Week of 12/25/2020:<br />Vaccination Campaign begins</div>`)
        .top(max / 2)
        .left(53)
        .class('milestone-2')
      );
    }
    
    chart.add(
      chrtAnnotation(`<div>2021</div>`)
      .top(data2021[id][0].value)
      .left(data2021[id][0].week)
      .class('title')
      .class('title2021')
    );
    
    chart.add(
      chrtAnnotation(`<div>2020</div>`)
      .top(data2020[id][0].value)
      .left(data2020[id][0].week)
      .class('title')
      .class('title2020')
    );

    // chart.add(
    //   chrtAnnotation(`<div>${new Intl.NumberFormat('en-EN').format(data[id][data[id].length -1].value)}</div>`)
    //   .top(data[id][data[id].length -1].value)
    //   .left(data[id].length -1)
    //   .class('thisWeek')
    // );
  });
</script>

<div class="chart chart-{id}" id="{id}" bind:this={el}>
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
