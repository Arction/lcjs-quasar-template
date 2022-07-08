<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import {
  lightningChart,
  LUT,
  ColorRGBA,
  PalettedFill,
  emptyLine,
  ColorShadingStyles,
  LegendBoxBuilders,
  UIElementBuilders,
  UIOrigins,
  UIDraggingModes,
  Themes,
} from '@arction/lcjs';
import { createWaterDropDataGenerator } from '@arction/xydata';
const el = ref();

const HEATMAP_COLUMNS = 1024;
const HEATMAP_ROWS = 1024;

let dashboard = undefined;
let heatmapSeries2D = undefined;
let surfaceSeries3D = undefined;

onMounted(() => {
  // Initialize empty Dashboard and charts.
  dashboard = lightningChart()
    .Dashboard({
      numberOfColumns: 2,
      numberOfRows: 1,
      theme: Themes.lightNew,
      container: el.value,
    })
    .setColumnWidth(0, 1.0)
    .setColumnWidth(1, 1.8);
  const chart2D = dashboard
    .createChartXY({
      columnIndex: 0,
      rowIndex: 0,
    })
    .setTitle('Generating test data ...');

  const chart3D = dashboard
    .createChart3D({
      columnIndex: 1,
      rowIndex: 0,
    })
    .setTitle('Generating test data ...');

  // Generate example data.

  chart2D.setTitle(`2D Heatmap Grid ${HEATMAP_COLUMNS}x${HEATMAP_ROWS}`);
  chart3D.setTitle(
    `3D Surface Grid ${HEATMAP_COLUMNS}x${HEATMAP_ROWS} color by Y`
  );

  // Define value -> color lookup table.
  const lut = new LUT({
    interpolate: true,
    steps: [
      { value: 0, color: ColorRGBA(0, 0, 0) },
      { value: 40, color: ColorRGBA(255, 215, 0) },
      { value: 60, color: ColorRGBA(255, 0, 0) },
      { value: 80, color: ColorRGBA(0, 0, 255) },
    ],
  });

  // Create series for both 2D and 3D heatmap data visualization and push data in.
  heatmapSeries2D = chart2D
    .addHeatmapGridSeries({
      columns: HEATMAP_COLUMNS,
      rows: HEATMAP_ROWS,
    })
    .setFillStyle(new PalettedFill({ lut }))
    .setWireframeStyle(emptyLine);

  surfaceSeries3D = chart3D
    .addSurfaceGridSeries({
      columns: HEATMAP_COLUMNS,
      rows: HEATMAP_ROWS,
    })
    .setFillStyle(new PalettedFill({ lut, lookUpProperty: 'y' }))
    .setWireframeStyle(emptyLine);

  // Add selector to see difference between Simple and Phong 3D color shading style in Surface grid series.
  const selectorColorShadingStyle = chart3D
    .addUIElement(UIElementBuilders.CheckBox)
    .setPosition({ x: 100, y: 100 })
    .setOrigin(UIOrigins.RightTop)
    .setMargin({ top: 40, right: 8 })
    .setDraggingMode(UIDraggingModes.notDraggable);
  selectorColorShadingStyle.onSwitch((_, state) => {
    surfaceSeries3D.setColorShadingStyle(
      state ? new ColorShadingStyles.Phong() : new ColorShadingStyles.Simple()
    );
    selectorColorShadingStyle.setText(
      `Color shading style: ${state ? 'Phong' : 'Simple'}`
    );
  });
  selectorColorShadingStyle.setOn(false);

  // Add legend with color look up table to chart.
  const legend = chart3D
    .addLegendBox(LegendBoxBuilders.HorizontalLegendBox)
    .add(chart2D)
    .add(chart3D);
  setData();
});

onUnmounted(() => {
  dashboard.dispose();
});

function setData() {
  createWaterDropDataGenerator()
    .setColumns(HEATMAP_COLUMNS)
    .setRows(HEATMAP_ROWS)
    .setVolatility(Math.random() * 100)
    .generate()
    .then((data) => {
      heatmapSeries2D.invalidateIntensityValues(data);
      surfaceSeries3D.invalidateHeightMap(data);
    });
}
</script>

<template>
  <div style="width: 1000px; height: 500px" ref="el"></div>
  <button @click="setData">Set Data</button>
</template>
