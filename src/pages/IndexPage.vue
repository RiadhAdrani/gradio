<script lang="ts">
import { computed } from "@vue/reactivity";
import { defineComponent, reactive } from "vue";

import logo from "../assets/transparent.png";
import "../css/main.css";

export default defineComponent({
  name: "IndexPage",
  components: {},
  setup() {
    const bg = reactive({
      type: "linear-gradient",
      direction: 90,
      structure: [
        ["#ff0000", 0],
        ["#00ffff", 50],
        ["#0000ff", 100],
      ],
    });

    const scrollIntoColor = (index: number) => {
      const items = document.querySelector(".color-items") as HTMLElement;

      const item = items.children.item(index) as HTMLElement;

      item.scrollIntoView();
      (item.querySelector(".color-value-input") as HTMLElement).click();
    };

    const getStructure = computed(() => bg.structure);

    const getDirection = computed(() => bg.direction);

    const getType = computed(() => bg.type);

    const changeDirection = (deg: number) => {
      bg.direction = deg;
    };

    const backgroundGradient = computed(
      () =>
        `${bg.type}(${bg.direction}${
          bg.type === "linear-gradient" ? "deg" : ""
        },${[...bg.structure]
          .sort((a, b) => {
            return (a[1] as number) - (b[1] as number);
          })
          .map((item) => {
            item as Array<string>;
            return `${item[0]} ${item[1]}%`;
          })
          .join(",")})`
    );

    const backgroundLinearGradient = computed(
      () =>
        `${bg.type}(90deg,${[...bg.structure]
          .sort((a, b) => {
            return (a[1] as number) - (b[1] as number);
          })
          .map((item) => {
            item as Array<string>;
            return `${item[0]} ${item[1]}%`;
          })
          .join(",")})`
    );

    const changeColorOptions = (
      index: number,
      color: string,
      position: number
    ) => {
      const newStructure = [
        ...bg.structure.slice(0, index),
        [color, Math.min(Math.max(position, 0), 100)],
        ...bg.structure.slice(index + 1),
      ];

      bg.structure = newStructure;
    };

    const addColor = (value: string, position: number) => {
      bg.structure = [...bg.structure, [value, Math.round(position)]];
    };

    const removeColor = (index: number) => {
      if (bg.structure.length <= 2) return;

      bg.structure = [
        ...bg.structure.slice(0, index),
        ...bg.structure.slice(index + 1),
      ];
    };

    const allowDelete = computed(() => bg.structure.length > 2);

    const calculateColorBetween = (
      color1: string,
      color2: string,
      percent: number
    ) => {
      const g1 = color1
        .replace("#", "")
        .match(/.{1,2}/g)
        ?.map((hex) => parseInt(hex, 16));
      const g2 = color2
        .replace("#", "")
        .match(/.{1,2}/g)
        ?.map((hex) => parseInt(hex, 16));

      const r = g1?.map((n, i) => {
        const hex = Math.round((n + (n * percent) / 100 + g2![i]) / 3).toString(
          16
        );

        return hex.length == 2 ? hex : `0${hex}`;
      });

      return "#" + r?.join("");
    };

    const addColorInSlider = (e: MouseEvent) => {
      const slider = document.querySelector(".slider");

      const { left, right } = slider!.getBoundingClientRect();

      const position = e.clientX;

      let percent = ((position - left) / (right - left)) * 100;

      let high: any;
      let low: any;

      [...bg.structure]
        .sort((a, b) => {
          return (a[1] as number) - (b[1] as number);
        })
        .forEach((colorData) => {
          if (colorData[1] < percent) {
            if (low === undefined) {
              low = colorData;
            } else {
              if (colorData[1] > low[1]) {
                low = colorData;
              }
            }
          }

          if (colorData[1] > percent) {
            if (high === undefined) {
              high = colorData;
            } else if (colorData[1] < high[1]) {
              high = colorData;
            }
          }
        });

      low = low || high;
      high = high || low;

      let color = "#000000";

      if (low[0] == high[0]) {
        color = low[0];
      } else {
        color = calculateColorBetween(
          low[0],
          high[0],
          ((percent - low[1]) / high[1]) * 100
        );
      }

      addColor(color, percent);
    };

    const onSlide = (e: DragEvent, index: number) => {
      var img = new Image();
      img.src = logo;
      e.dataTransfer!.setDragImage(img, 0, 0);

      const slider = document.querySelector(".slider");

      const { left, right } = slider!.getBoundingClientRect();

      const position = e.clientX;

      let percent = ((position - left) / (right - left)) * 100;

      if (0 <= percent && percent <= 100) {
        changeColorOptions(
          index,
          bg.structure[index][0] as string,
          Math.round(percent)
        );
      }
    };

    const setRadialType = () => {
      (bg.direction as any) = "circle";
      bg.type = "radial-gradient";
    };

    const setLinearType = () => {
      (bg.direction as any) = 90;
      bg.type = "linear-gradient";
    };

    return {
      getStructure,
      getDirection,
      getType,
      backgroundGradient,
      backgroundLinearGradient,
      allowDelete,
      changeColorOptions,
      changeDirection,
      addColor,
      addColorInSlider,
      removeColor,
      onSlide,
      setRadialType,
      setLinearType,
      scrollIntoColor,
    };
  },
});
</script>

<template>
  <h4 class="header">Gradio</h4>
  <div class="gradient-preview" :style="{ background: backgroundGradient }" />
  <div class="container gradient-options">
    <p class="gradient-code">
      <span style="font-weight: 900; font-style: italic">background:</span>
      {{ backgroundGradient }}
    </p>
    <div class="gradient-type">
      <q-btn label="Linear" size="10px" no-caps @click="setLinearType" />
      <q-btn label="Radial" size="10px" no-caps @click="setRadialType" />
    </div>
    <div
      class="slider"
      :style="{ background: backgroundLinearGradient }"
      @click="addColorInSlider"
    >
      <div
        class="slider-item"
        v-for="(color, index) in getStructure"
        :style="{ left: color[1] + '%' }"
        draggable="true"
        @drag="(e) => onSlide(e, index)"
        @click.stop="() => scrollIntoColor(index)"
        @contextmenu.prevent="() => removeColor(index)"
      >
        <div :style="{backgroundColor: color[0] as string}"></div>
      </div>
    </div>
    <div class="gradient-direction">
      <q-knob
        v-if="getType === 'linear-gradient'"
        show-value
        v-model="getDirection"
        color="primary"
        :max="360"
        @drag-value="(e : number) => {changeDirection(e)}"
      ></q-knob>
    </div>
    <div class="color-items">
      <div class="color-item" v-for="(color, index) in getStructure">
        <input
          class="color-value-input"
          type="color"
          :value="color[0]"
          @input="(e : Event) => changeColorOptions(index, (e.currentTarget as HTMLInputElement).value, color[1] as number )"
        />
        <p class="color-value">{{ color[0] }}</p>
        <div class="color-position">
          <input
            :value="color[1]"
            type="number"
            max="100"
            min="0"
            @input="(e : Event) => changeColorOptions(index, color[0] as string, parseInt((e.currentTarget as HTMLInputElement).value) )"
          /><span>%</span>
        </div>
        <q-btn
          v-if="allowDelete"
          @click="() => removeColor(index)"
          size="10px"
          icon="delete"
          no-caps
        ></q-btn>
      </div>
    </div>
    <div class="add-color">
      <q-btn size="10px" label="add" @click="() => addColor('#000000', 100)" />
    </div>
  </div>
</template>
