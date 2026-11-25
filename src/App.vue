<template>
  <div
    class="flex h-screen w-screen overflow-hidden font-sans text-gray-700 bg-gray-100 dark:bg-gray-900 dark:text-gray-300">
    <aside
      class="flex flex-col w-96 bg-white shadow-2xl z-1000 overflow-y-auto border-r border-gray-200 dark:bg-gray-800 dark:border-gray-700">
      <div
        class="p-5 border-b border-gray-100 dark:border-gray-700 flex justify-between items-center">
        <h2
          class="text-xl font-bold text-gray-800 dark:text-white flex items-center gap-2">
          <svg
            width="40"
            height="40"
            viewBox="0 0 100 100"
            fill="none"
            xmlns="http://www.w3.org/2000/svg">
            <circle
              cx="50"
              cy="50"
              r="35"
              stroke="#3B82F6"
              stroke-width="2"
              stroke-dasharray="4 4" />
            <circle cx="50" cy="50" r="22" fill="#DBEAFE" />
            <path
              d="M50 35V65M50 65L40 55M50 65L60 55"
              stroke="#1D4ED8"
              stroke-width="5"
              stroke-linecap="round"
              stroke-linejoin="round" />
            <rect x="75" y="25" width="10" height="10" rx="2" fill="#1D4ED8">
              <animateTransform
                attributeName="transform"
                type="rotate"
                from="0 50 50"
                to="360 50 50"
                dur="10s"
                repeatCount="indefinite" />
            </rect>
          </svg>
          卫星底图下载器
          <span
            class="text-xs bg-blue-100 text-blue-700 px-2 py-0.5 rounded-full dark:bg-blue-900 dark:text-blue-300">
            Pro
          </span>
        </h2>
        <label class="switch">
          <input type="checkbox" :checked="isDark" @change="toggleDark" />
          <span class="slider"></span>
        </label>
      </div>

      <div class="flex-1 p-5 flex flex-col gap-6">
        <div
          class="text-xs h-8 font-bold text-center py-2 px-3 rounded-md border transition-colors duration-300"
          :class="[
            currentMode === 'polygon'
              ? 'bg-green-50 text-green-700 border-green-200 dark:bg-green-900/30 dark:text-green-400 dark:border-green-800'
              : currentMode === 'rect'
                ? 'bg-gray-100 text-gray-600 border-gray-200 dark:bg-gray-700 dark:text-gray-300 dark:border-gray-600'
                : 'bg-yellow-50 text-yellow-600 border-yellow-100 dark:bg-yellow-900/30 dark:text-yellow-400 dark:border-yellow-800',
          ]">
          {{ modeText }}
        </div>

        <div
          class="bg-white rounded-xl border border-gray-100 shadow-sm p-4 hover:shadow-md transition-shadow dark:bg-gray-800 dark:border-gray-700">
          <div
            class="text-sm font-bold text-gray-800 mb-3 flex items-center gap-2 dark:text-gray-200">
            <span class="text-lg">📍</span>
            按区域轮廓下载
          </div>
          <Select
            :modelValue="searchQuery"
            :options="allRegions"
            @update:modelValue="selectRegion"
            placeholder="选择或输入区域关键字" />
        </div>

        <div
          class="bg-white rounded-xl border border-gray-100 shadow-sm p-4 dark:bg-gray-800 dark:border-gray-700">
          <div
            class="text-sm font-bold text-gray-800 mb-2 flex items-center gap-2 dark:text-gray-200">
            <span class="text-lg">📐</span>
            按矩形框选下载
          </div>
          <p class="text-xs text-gray-500 leading-relaxed dark:text-gray-400">
            点击地图左侧工具栏的
            <strong
              class="text-gray-900 bg-gray-200 px-1 rounded dark:text-gray-100 dark:bg-gray-700">
              ⬛ 矩形
            </strong>
            图标画框。
            <br />
            <span class="text-blue-500 dark:text-blue-400">
              提示：画完后可以拖动角点微调范围。
            </span>
          </p>
        </div>

        <div
          class="bg-white rounded-xl border border-gray-100 shadow-sm p-4 dark:bg-gray-800 dark:border-gray-700">
          <div
            class="text-sm font-bold text-gray-800 mb-3 flex items-center gap-2 dark:text-gray-200">
            <span class="text-lg">⚙️</span>
            设置
          </div>

          <div class="mb-2 flex justify-between items-center">
            <label class="text-xs font-medium text-gray-600 dark:text-gray-400">
              缩放级别 (Zoom)
            </label>
            <span
              class="text-xs font-bold text-blue-600 bg-blue-50 px-2 py-0.5 rounded dark:bg-blue-900/50 dark:text-blue-400">
              {{ zoomLevel }}
            </span>
          </div>

          <input
            type="range"
            min="6"
            max="18"
            v-model.number="zoomLevel"
            class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-600 dark:bg-gray-700" />

          <div
            class="flex justify-between mt-2 text-[10px] text-gray-400 dark:text-gray-500">
            <span>国家级 (6)</span>
            <span>省市级 (10)</span>
            <span>街道级 (18)</span>
          </div>

          <div
            v-if="tileCount > 1000"
            class="mt-3 text-xs bg-amber-50 text-amber-700 p-2 rounded border border-amber-100 flex items-start gap-1 dark:bg-amber-900/30 dark:text-amber-400 dark:border-amber-800">
            ⚠️
            <span>
              瓦片量
              <strong>{{ tileCount }}</strong>
              张，可能导致内存不足。
            </span>
          </div>

          <div class="mt-4">
            <div class="mb-2 flex justify-between items-center">
              <label
                class="text-xs font-medium text-gray-600 dark:text-gray-400">
                底图
              </label>
            </div>
            <div class="grid grid-cols-3 gap-2">
              <button
                v-for="option in baseMapOptions"
                :key="option.id"
                class="flex-1 py-1.5 px-1 text-xs rounded-lg border transition-colors cursor-pointer truncate"
                :class="
                  option.id === currentBaseMap
                    ? 'bg-blue-600 text-white border-blue-600 dark:bg-blue-600 dark:border-blue-600'
                    : 'bg-white text-gray-600 border-gray-200 hover:bg-gray-50 dark:bg-gray-700 dark:text-gray-300 dark:border-gray-600 dark:hover:bg-gray-600'
                "
                @click="setBaseMap(option.id)">
                {{ option.name }}
              </button>
            </div>
            <label v-if="currentBaseMap === 'tianditu'" class="block mt-3 text-sm">
              天地图密钥（tk）
              <input v-model.lazy.trim="tiandituKey" type="password" placeholder="输入后按回车或点击空白处" @keyup.enter="($event.target as HTMLInputElement).blur()" class="w-full border p-1 dark:bg-gray-800" />
            </label>
            <label v-if="currentBaseMap === 'tianditu'" class="block mt-2 text-sm">
              <input type="checkbox" v-model="rememberKey" /> 在此浏览器记住密钥
            </label>
            <p v-if="keyStorageError && currentBaseMap === 'tianditu'" class="text-xs text-amber-600">{{ keyStorageError }}</p>
            <label class="block mt-3 text-sm">文件格式
              <select v-model="fileFormat" :disabled="isDownloading" class="border dark:bg-gray-800">
                <option value="png">PNG</option><option value="tif">GeoTIFF（含坐标）</option>
              </select>
            </label>
            <label v-if="fileFormat === 'tif'" class="block mt-2 text-sm">
              <input type="checkbox" v-model="correctGcj" :disabled="isDownloading" />
              中国大陆 GCJ-02 图源近似纠偏
            </label>
            <label v-if="activeLabels" class="block mt-3 text-sm">导出内容
              <select v-model="mode" :disabled="isDownloading" class="border dark:bg-gray-800">
                <option value="both">影像＋道路地名</option><option value="labels">仅透明注记</option><option value="base">仅卫星影像</option>
              </select>
            </label>
          </div>
        </div>
      </div>

      <div class="p-5 mt-auto">
        <button
          class="w-full py-3 px-4 cursor-pointer bg-blue-600 hover:bg-blue-700 text-white text-sm font-bold rounded-xl shadow-lg shadow-blue-200 disabled:bg-gray-300 disabled:text-gray-500 disabled:shadow-none disabled:cursor-not-allowed transition-all transform active:scale-[0.98] dark:shadow-blue-900/30 dark:disabled:bg-gray-700 dark:disabled:text-gray-500"
          :disabled="!currentMode || isDownloading || (currentBaseMap === 'tianditu' && !tiandituKey)"
          @click="startDownload">
          {{ isDownloading ? `处理中 ${downloadProgress}%` : "开始下载" }}
        </button>

        <div
          v-if="isDownloading"
          class="mt-3 h-2 w-full bg-gray-200 rounded-full overflow-hidden dark:bg-gray-700">
          <div
            class="h-full bg-green-500 transition-all duration-300 ease-out"
            :style="{ width: downloadProgress + '%' }"></div>
        </div>

        <div class="text-center mt-3 text-xs text-gray-400 font-medium min-h-4">
          {{ statusText }}
        </div>
      </div>
    </aside>

    <div
      id="map"
      ref="mapContainer"
      class="flex-1 h-full z-0 bg-gray-900"></div>
  </div>
</template>

<script setup lang="ts">
  import { onMounted, ref, shallowRef, computed, watch } from "vue";
  import Select from "./components/Select.vue";
  import L from "leaflet";
  import "leaflet/dist/leaflet.css";
  import "@geoman-io/leaflet-geoman-free";
  import "@geoman-io/leaflet-geoman-free/dist/leaflet-geoman.css";

  type DownloadMode = "rect" | "polygon" | null;
  type BaseMapId =
    | "esri_light_gray"
    | "esri_dark_gray"
    | "esri_hill_shade"
    | "esri_imagery_firefly"
    | "esri_imagery"
    | "gaode_vector"
    | "tianditu"
    | "google_satellite";

  interface TileCoord {
    x: number;
    y: number;
  }

  interface Point {
    x: number;
    y: number;
  }

  interface GeoJSONResult {
    type: "Polygon" | "MultiPolygon";
    coordinates: any[];
  }

  interface RegionItem {
    adcode: number;
    name: string;
    center: [number, number];
    centroid: [number, number];
    level: string;
    bbox: [number, number, number, number];
    childrenNum?: number;
    parent?: {
      adcode: number | null;
    };
    subFeatureIndex?: number;
    acroutes?: number[];
    pinyin?: string[];
  }

  interface BaseMapOption {
    id: BaseMapId;
    name: string;
    url: string;
    labelUrl?: string;
    labelOptions?: L.TileLayerOptions;
  }

  const mapContainer = ref<HTMLElement | null>(null);
  const map = shallowRef<L.Map | null>(null);
  const cityLayer = shallowRef<L.Layer | null>(null);
  const rectLayer = shallowRef<L.Layer | null>(null);
  const baseMapLayer = shallowRef<L.TileLayer | null>(null);
  const labelLayer = shallowRef<L.TileLayer | null>(null);

  const searchQuery = ref<number>();
  const allRegions = ref<RegionItem[]>([]);
  const isRegionLoading = ref(false);
  const regionLoadError = ref("");
  const isSearching = ref(false);
  const searchError = ref("");
  const zoomLevel = ref(13);
  const currentMode = ref<DownloadMode>(null);
  const cityGeoJSON = ref<GeoJSONResult | null>(null);
  const tileCount = ref(0);
  const isDownloading = ref(false);
  const downloadProgress = ref(0);
  const statusText = ref("请搜索城市或绘制矩形...");
  const currentBaseMap = ref<BaseMapId>("gaode_vector");
  const isDark = ref(false);

  const updateThemeState = (dark: boolean) => {
    isDark.value = dark;
    if (dark) {
      document.documentElement.classList.add("dark");
    } else {
      document.documentElement.classList.remove("dark");
    }
  };

  const toggleDark = () => {
    const newState = !isDark.value;
    updateThemeState(newState);
    localStorage.setItem("theme", newState ? "dark" : "light");
  };

  const keyStorageName = "sat-hunter:tianditu-browser-key";
  let savedKey = "";
  try { savedKey = localStorage.getItem(keyStorageName) || ""; } catch { /* Storage can be disabled. */ }
  const tiandituKey = ref(savedKey);
  const rememberKey = ref(Boolean(savedKey));
  const keyStorageError = ref("");
  watch([rememberKey, tiandituKey], ([remember, key]) => {
    try {
      if (remember && key) localStorage.setItem(keyStorageName, key);
      else localStorage.removeItem(keyStorageName);
      keyStorageError.value = "";
    } catch {
      keyStorageError.value = "浏览器未允许更新保存的密钥，可在站点设置中管理数据。";
    }
  });
  const googleTiles = "https://mt0.google.com/vt?hl=zh-CN&x={x}&y={y}&z={z}&lyrs=";
  function tiandituTiles(layer: "img" | "cia") {
    return "https://t0.tianditu.gov.cn/" + layer + "_w/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}&LAYER=" + layer + "&tk=" + encodeURIComponent(tiandituKey.value);
  }
  const baseMapOptions: BaseMapOption[] = [
    {
      id: "gaode_vector",
      name: "高德 卫星影像",
      url: "https://webst01.is.autonavi.com/appmaptile?style=6&x={x}&y={y}&z={z}",
    },
    {
      id: "esri_light_gray",
      name: "浅色灰底图",
      url: "https://server.arcgisonline.com/ArcGIS/rest/services/Canvas/World_Light_Gray_Base/MapServer/tile/{z}/{y}/{x}",
    },
    {
      id: "esri_dark_gray",
      name: "暗色灰底图",
      url: "https://server.arcgisonline.com/ArcGIS/rest/services/Canvas/World_Dark_Gray_Base/MapServer/tile/{z}/{y}/{x}",
    },
    {
      id: "esri_imagery",
      name: "ESRI 卫星影像",
      url: "https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}",
    },
    {
      id: "esri_hill_shade",
      name: "地形图",
      url: "https://server.arcgisonline.com/arcgis/rest/services/Elevation/World_Hillshade/MapServer/tile/{z}/{y}/{x}",
    },
    {
      id: "esri_imagery_firefly",
      name: "萤火虫影像混合图",
      url: "https://fly.maptiles.arcgis.com/arcgis/rest/services/World_Imagery_Firefly/MapServer/tile/{z}/{y}/{x}",
    },
    {
      id: "google_satellite",
      name: "Google 卫星影像",
      url: googleTiles + "s",
    },
    {
      id: "tianditu",
      name: "天地图 卫星影像",
      get url() { return tiandituTiles("img"); },
    },
  ];

  const selectedData = computed(() =>
    allRegions.value.find((op) => op.adcode === searchQuery.value),
  );

  const modeText = computed(() => {
    if (currentMode.value === "rect") return "当前模式: 矩形框选";
    if (currentMode.value === "polygon")
      return `当前模式: 区域轮廓 (${selectedData.value?.name})`;
    return "当前模式: 等待操作";
  });

  watch(zoomLevel, () => updateEstimate());

  const setBaseMap = (id: BaseMapId, force = false) => {
    if (!map.value) return;
    if (!force && currentBaseMap.value === id && baseMapLayer.value) return;
    currentBaseMap.value = id;
    if (baseMapLayer.value) {
      map.value.removeLayer(baseMapLayer.value);
      baseMapLayer.value = null;
    }
    if (labelLayer.value) {
      map.value.removeLayer(labelLayer.value);
      labelLayer.value = null;
    }
    const config = baseMapOptions.find((item) => item.id === id);
    if (!config || (id === "tianditu" && !tiandituKey.value)) return;
    const layer = L.tileLayer(config.url, {
      maxZoom: 19,
      crossOrigin: "anonymous",
    });
    layer.addTo(map.value);
    baseMapLayer.value = layer;
    refreshLabels();
  };

  const fileFormat = ref("png");
  const correctGcj = ref(true);
  watch(currentBaseMap, id => { correctGcj.value = id === "gaode_vector"; });
  const mode = ref("both");
  const labelsUrl = "https://webst01.is.autonavi.com/appmaptile?style=8&lang=zh_cn&size=1&scale=1&x={x}&y={y}&z={z}";
  const activeLabels = computed(() => {
    switch (currentBaseMap.value) {
      case "gaode_vector": return labelsUrl;
      case "google_satellite": return googleTiles + "h";
      case "tianditu": return tiandituTiles("cia");
      default: return "";
    }
  });
  watch(tiandituKey, () => {
    if (currentBaseMap.value === "tianditu") setBaseMap("tianditu", true);
  });
  function refreshLabels() {
    if (!map.value) return;
    if (labelLayer.value) map.value.removeLayer(labelLayer.value);
    labelLayer.value = null;
    if (baseMapLayer.value && activeLabels.value && mode.value !== "base") {
      labelLayer.value = L.tileLayer(activeLabels.value, {maxZoom: 19, zIndex: 2}).addTo(map.value);
    }
  }
  watch(mode, refreshLabels);
  function loadTile(template: string, x: number, y: number, z: number): Promise<HTMLImageElement> {
    return new Promise((resolve, reject) => {
      const img = new Image();
      img.crossOrigin = "anonymous";
      img.onload = () => resolve(img);
      img.onerror = () => reject(new Error("瓦片读取失败"));
      img.src = template.replace("{x}", String(x)).replace("{y}", String(y)).replace("{z}", String(z));
    });
  }
  const loadAddressData = async () => {
    isRegionLoading.value = true;
    regionLoadError.value = "";
    try {
      const res = await fetch(
        "https://geo.datav.aliyun.com/areas_v3/bound/all.json",
      );
      if (!res.ok) {
        throw new Error(`Failed to load address data: ${res.status}`);
      }
      const ct = res.headers.get("content-type") || "";
      if (!ct.includes("application/json")) {
        throw new Error("Invalid content type for address data");
      }
      const data = await res.json();
      if (Array.isArray(data)) {
        allRegions.value = data as RegionItem[];
      } else {
        allRegions.value = [];
        regionLoadError.value = "区域数据格式不正确";
      }
    } catch (e) {
      regionLoadError.value = "区域数据加载失败";
      console.error(e);
    } finally {
      isRegionLoading.value = false;
    }
  };

  onMounted(() => {
    const theme = localStorage.getItem("theme");
    const mediaQuery = window.matchMedia("(prefers-color-scheme: dark)");

    if (theme === "dark") {
      updateThemeState(true);
    } else if (theme === "light") {
      updateThemeState(false);
    } else {
      updateThemeState(mediaQuery.matches);
    }

    mediaQuery.addEventListener("change", (e) => {
      updateThemeState(e.matches);
      localStorage.removeItem("theme");
    });

    if (!mapContainer.value) return;

    // 1. 创建 Map
    const mapInstance = L.map(mapContainer.value).setView(
      [39.9042, 116.4074],
      10,
    );
    map.value = mapInstance;

    // 2. 底图
    setBaseMap(currentBaseMap.value);

    mapInstance.pm.addControls({
      position: "topleft",
      drawCircleMarker: false,
      drawMarker: false,
      drawCircle: false,
      drawPolyline: false,
      drawPolygon: false, // 如果需要手绘多边形可设为 true
      drawText: false,
      drawRectangle: true, // 核心功能

      editMode: true, // 允许编辑顶点
      dragMode: false, // 禁止整体拖拽(防止误操作)
      cutPolygon: false,
      removalMode: true,
      rotateMode: false,
    });

    // 设置绘制样式
    mapInstance.pm.setPathOptions({
      color: "#3b82f6", // Tailwind Blue-500
      fillColor: "#3b82f6",
      fillOpacity: 0.2,
      weight: 2,
    });

    // 设置全局语言
    mapInstance.pm.setLang("zh");

    // 监听创建 (画完框)
    mapInstance.on("pm:create", (e: any) => {
      // 1. 清除旧的
      clearAllLayers();

      const layer = e.layer;
      rectLayer.value = layer;
      setMode("rect");

      // 2. 监听该层的编辑事件 (拖拽角点)
      layer.on("pm:edit", () => {
        updateEstimate();
      });
    });

    // 监听删除 (点击删除按钮)
    mapInstance.on("pm:remove", (e: any) => {
      if (e.layer === rectLayer.value) {
        rectLayer.value = null;
        setMode(null);
      }
    });

    loadAddressData();
  });

  const clearAllLayers = (resetMode = true) => {
    // 移除城市轮廓
    if (cityLayer.value && map.value) {
      map.value.removeLayer(cityLayer.value);
      cityLayer.value = null;
    }

    // 移除矩形框 (Geoman 已经把它加到地图上了，所以需要手动 remove)
    if (rectLayer.value && map.value) {
      map.value.removeLayer(rectLayer.value);
      rectLayer.value = null;
    }

    cityGeoJSON.value = null;
    if (resetMode) {
      setMode(null);
    }
  };

  const setMode = (mode: DownloadMode) => {
    currentMode.value = mode;
    updateEstimate();
  };

  const selectRegion = (adcode: number) => {
    searchQuery.value = adcode;
    searchError.value = "";
    const level = { country: 6, province: 10, city: 10, district: 14 }?.[
      selectedData.value?.level ?? ""
    ];
    if (level) {
      zoomLevel.value = level;
    }

    searchLocation(adcode);
  };

  const searchLocation = async (adcode: number) => {
    // 搜索时清除之前的框
    clearAllLayers(false);
    setMode("polygon");

    statusText.value = "正在获取轮廓...";

    try {
      const endpoint = `https://geo.datav.aliyun.com/areas_v3/bound/${adcode}.json`;

      const res = await fetch(endpoint);
      if (!res.ok) {
        throw new Error(`Failed to fetch boundary: ${res.status}`);
      }
      const ct = res.headers.get("content-type") || "";
      if (!ct.includes("application/json")) {
        throw new Error("Invalid content type for boundary data");
      }
      const json = await res.json();

      let geometry: GeoJSONResult | null = null;
      if (
        json &&
        json.type === "FeatureCollection" &&
        Array.isArray(json.features)
      ) {
        const polys: any[] = [];
        json.features.forEach((f: any) => {
          if (f && f.geometry) {
            if (f.geometry.type === "Polygon") {
              polys.push(f.geometry.coordinates);
            } else if (f.geometry.type === "MultiPolygon") {
              polys.push(...f.geometry.coordinates);
            }
          }
        });
        if (polys.length > 0) {
          geometry = { type: "MultiPolygon", coordinates: polys };
        }
      } else if (
        json &&
        (json.type === "Polygon" || json.type === "MultiPolygon")
      ) {
        geometry = { type: json.type, coordinates: json.coordinates };
      }

      if (!geometry) {
        searchError.value = "该地点没有详细的轮廓数据";
        setMode(null);
        return;
      }

      cityGeoJSON.value = geometry;
      if (map.value && cityGeoJSON.value) {
        const layer = L.geoJSON(cityGeoJSON.value as any, {
          style: { color: "#ef4444", weight: 2, fillOpacity: 0.1 },
        }).addTo(map.value);
        cityLayer.value = layer;
        map.value.fitBounds(layer.getBounds());
        // setMode("polygon"); // 已在开始时设置
        updateEstimate();
        statusText.value = `已选中: ${selectedData.value?.name}`;
      }
    } catch (e) {
      searchError.value = "网络请求失败";
      console.error(e);
      setMode(null);
    } finally {
      isSearching.value = false;
    }
  };

  const updateEstimate = () => {
    let bounds: L.LatLngBounds | null = null;

    if (currentMode.value === "rect" && rectLayer.value) {
      // @ts-ignore
      bounds = rectLayer.value.getBounds();
    } else if (currentMode.value === "polygon" && cityLayer.value) {
      // @ts-ignore
      bounds = cityLayer.value.getBounds();
    }

    if (!bounds) {
      tileCount.value = 0;
      statusText.value = "请操作左侧面板...";
      return;
    }

    const z = zoomLevel.value;
    const nw = bounds.getNorthWest();
    const se = bounds.getSouthEast();
    const tMin = latLonToTile(nw.lat, nw.lng, z);
    const tMax = latLonToTile(se.lat, se.lng, z);

    tileCount.value =
      (Math.abs(tMax.x - tMin.x) + 1) * (Math.abs(tMax.y - tMin.y) + 1);
    statusText.value = `预计瓦片: ${tileCount.value} 张`;
  };

  // Approximate mainland WGS84 -> GCJ-02 transform used only for explicitly selected tiles.
  function gcjOffset(lon: number, lat: number) {
    const x = lon - 105, y = lat - 35, p = Math.PI;
    const wave = (v: number, n: number) => Math.sin(v * n * p);
    const common = (20 * wave(x, 6) + 20 * wave(x, 2)) * 2 / 3;
    const dLat = -100 + 2*x + 3*y + 0.2*y*y + 0.1*x*y + 0.2*Math.sqrt(Math.abs(x)) + common
      + (20*wave(y,1) + 40*wave(y,1/3) + 160*wave(y,1/12) + 320*wave(y,1/30))*2/3;
    const dLon = 300 + x + 2*y + 0.1*x*x + 0.1*x*y + 0.1*Math.sqrt(Math.abs(x)) + common
      + (20*wave(x,1) + 40*wave(x,1/3) + 150*wave(x,1/12) + 300*wave(x,1/30))*2/3;
    const rad = lat*p/180, e = 0.006693421622965943, axis = 6378245;
    const m = 1 - e * Math.sin(rad)**2;
    return {lon: dLon*180*Math.sqrt(m)/(axis*Math.cos(rad)*p),
      lat: dLat*180*m**1.5/(axis*(1-e)*p)};
  }

  function geoTiff(ctx: CanvasRenderingContext2D, left: number, top: number, z: number, correct: boolean) {
    const w = ctx.canvas.width, h = ctx.canvas.height, world = 256 * 2**z;
    const half = Math.PI * 6378137, res = 2 * half / world;
    // Offset in global XYZ pixels at a WGS84 pixel position.
    function shift(px: number, py: number) {
      const lon = px/world*360-180;
      const lat = Math.atan(Math.sinh(Math.PI*(1-2*py/world)))*180/Math.PI;
      const d = gcjOffset(lon,lat);
      const q = project(lat+d.lat,lon+d.lon,z);
      return {x:q.x-px,y:q.y-py};
    }
    let ox = left, oy = top, width = w, height = h;
    if (correct) {
      let xmin = Infinity, ymin = Infinity, xmax = -Infinity, ymax = -Infinity;
      function bound(x: number, y: number) {
        let px = left+x, py = top+y;
        for (let i=0;i<4;i++) {
          const d = shift(px,py);
          px = left+x-d.x; py = top+y-d.y;
        }
        xmin=Math.min(xmin,px); xmax=Math.max(xmax,px);
        ymin=Math.min(ymin,py); ymax=Math.max(ymax,py);
      }
      for(let x=0;x<w;x+=16){bound(x,0);bound(x,h);}
      for(let y=0;y<h;y+=16){bound(0,y);bound(w,y);}
      bound(w,h); ox=Math.floor(xmin)-1; oy=Math.floor(ymin)-1;
      width=Math.ceil(xmax)-ox+1; height=Math.ceil(ymax)-oy+1;
    }
    // Little-endian baseline RGBA TIFF, PixelIsArea, EPSG:3857.
    const tags = 14, extras = 8+2+tags*12+4, bits = extras, scale=bits+8;
    const tie=scale+24, keys=tie+48, raster=keys+32;
    const header=new ArrayBuffer(raster), v=new DataView(header);
    const u16=(p:number,n:number)=>v.setUint16(p,n,true);
    const u32=(p:number,n:number)=>v.setUint32(p,n,true);
    const dbl=(p:number,n:number)=>v.setFloat64(p,n,true);
    u16(0,0x4949);u16(2,42);u32(4,8);u16(8,tags);
    let pos=10;
    function tag(id:number,type:number,count:number,value:number){
      u16(pos,id);u16(pos+2,type);u32(pos+4,count);
      if(type===3 && count===1) u16(pos+8,value); else u32(pos+8,value);
      pos+=12;
    }
    tag(256,4,1,width);tag(257,4,1,height);tag(258,3,4,bits);
    tag(259,3,1,1);tag(262,3,1,2);tag(273,4,1,raster);tag(277,3,1,4);
    tag(278,4,1,height);tag(279,4,1,width*height*4);tag(284,3,1,1);
    tag(338,3,1,2);
    tag(33550,12,3,scale);tag(33922,12,6,tie);tag(34735,3,16,keys);
    [8,8,8,8].forEach((n,i)=>u16(bits+2*i,n));
    dbl(scale,res);dbl(scale+8,res);dbl(tie+24,ox*res-half);dbl(tie+32,half-oy*res);
    [1,1,0,3,1024,0,1,1,1025,0,1,1,3072,0,1,3857].forEach((n,i)=>u16(keys+i*2,n));
    const parts: BlobPart[]=[header];
    for(let y=0;y<height;y+=64){
      const rows=Math.min(64,height-y);
      if(!correct){parts.push(new Uint8Array(ctx.getImageData(0,y,w,rows).data).buffer);continue;}
      const block=new Uint8Array(width*rows*4);
      for(let r=0;r<rows;r++){
        const py=oy+y+r+0.5;
        const nodes=[];
        for(let x=0;x<width+16;x+=16) nodes.push(shift(ox+x+0.5,py));
        const ys=nodes.map(d=>Math.floor(py+d.y-top));
        const lo=Math.max(0,Math.min(...ys)-1), hi=Math.min(h-1,Math.max(...ys)+1);
        if(hi<lo) continue;
        const source=ctx.getImageData(0,lo,w,hi-lo+1).data;
        for(let x=0;x<width;x++){
          const a=nodes[x>>4]!, b=nodes[(x>>4)+1]!, f=(x%16)/16;
          const sx=Math.floor(ox+x+0.5+a.x+(b.x-a.x)*f-left);
          const sy=Math.floor(py+a.y+(b.y-a.y)*f-top);
          if(sx<0||sx>=w||sy<lo||sy>hi) continue;
          const idx=((sy-lo)*w+sx)*4;
          block.set(source.subarray(idx,idx+4),(r*width+x)*4);
        }
      }
      parts.push(block.buffer);
    }
    return new Blob(parts,{type:"image/tiff"});
  }

  const startDownload = async () => {
    let bounds: L.LatLngBounds | undefined;
    if (currentMode.value === "rect" && rectLayer.value) {
      // @ts-ignore
      bounds = rectLayer.value.getBounds();
    } else if (currentMode.value === "polygon" && cityLayer.value) {
      // @ts-ignore
      bounds = cityLayer.value.getBounds();
    }

    if (!bounds) return;
    if (tileCount.value > 1500) {
      alert("瓦片数量过多，请缩小范围或降低 Zoom 级别");
      return;
    }

    const format = fileFormat.value;
    const center = bounds.getCenter();
    const rectify = correctGcj.value && center.lng >= 72 && center.lng <= 138 && center.lat >= 0.8 && center.lat <= 56;
    const name = currentMode.value === "polygon" ? searchQuery.value : "terra_snap_map";
    isDownloading.value = true;
    downloadProgress.value = 0;
    statusText.value = "准备下载...";

    const z = zoomLevel.value;
    const nw = bounds.getNorthWest();
    const se = bounds.getSouthEast();

    const tMin = latLonToTile(nw.lat, nw.lng, z);
    const tMax = latLonToTile(se.lat, se.lng, z);

    const minX = Math.min(tMin.x, tMax.x);
    const maxX = Math.max(tMin.x, tMax.x);
    const minY = Math.min(tMin.y, tMax.y);
    const maxY = Math.max(tMin.y, tMax.y);

    const tilesX = maxX - minX + 1;
    const tilesY = maxY - minY + 1;
    const totalTiles = tilesX * tilesY;

    const canvas = document.createElement("canvas");
    canvas.width = tilesX * 256;
    canvas.height = tilesY * 256;
    const ctx = canvas.getContext("2d");

    if (!ctx) {
      isDownloading.value = false;
      return;
    }

    let loaded = 0;
    const clipGeometry = currentMode.value === 'polygon' ? cityGeoJSON.value : null;
    const activeBaseMap =
      baseMapOptions.find((item) => item.id === currentBaseMap.value) ||
      baseMapOptions[0];
    if (!activeBaseMap) {
      isDownloading.value = false;
      return;
    }
    const overlay = activeLabels.value;
    const output = overlay ? mode.value : "base";
    const sources = output === "both" ? [activeBaseMap.url, overlay]
      : [output === "labels" ? overlay : activeBaseMap.url];
    let next = 0;
    let failed = false;
    async function worker() {
      while (next < totalTiles && !failed) {
        const index = next++;
        const col = index % tilesX;
        const row = Math.floor(index / tilesX);
        try {
          for (const url of sources) {
            const tile = await loadTile(url, minX + col, minY + row, z);
            ctx!.drawImage(tile, col * 256, row * 256);
          }
          downloadProgress.value = Math.floor(++loaded / totalTiles * 100);
        } catch {
          failed = true;
        }
      }
    }
    await Promise.all(Array.from({length: Math.min(8, totalTiles)}, worker));
    if (failed) {
      isDownloading.value = false;
      statusText.value = "瓦片读取失败，请检查图源连接与跨域支持后重试。";
      return;
    }

    // 城市轮廓裁剪
    if (clipGeometry) {
      statusText.value = "应用轮廓裁剪...";
      ctx.globalCompositeOperation = "destination-in";
      ctx.beginPath();

      const originWorldPixelX = minX * 256;
      const originWorldPixelY = minY * 256;

      const getCanvasPixel = (lat: number, lng: number): Point => {
        const p = project(lat, lng, z);
        return {
          x: p.x - originWorldPixelX,
          y: p.y - originWorldPixelY,
        };
      };

      drawGeoJSONPath(ctx, clipGeometry, getCanvasPixel);
      ctx.fill();
      ctx.globalCompositeOperation = "source-over";
    }

    statusText.value = "导出图片...";

    const pNW = project(nw.lat, nw.lng, z);
    const pSE = project(se.lat, se.lng, z);
    const originX = minX * 256;
    const originY = minY * 256;

    let cropX = pNW.x - originX;
    let cropY = pNW.y - originY;
    let cropW = pSE.x - pNW.x;
    let cropH = pSE.y - pNW.y;

    cropX = Math.max(0, cropX);
    cropY = Math.max(0, cropY);
    if (cropX + cropW > canvas.width) cropW = canvas.width - cropX;
    if (cropY + cropH > canvas.height) cropH = canvas.height - cropY;

    const finalCanvas = document.createElement("canvas");
    finalCanvas.width = cropW;
    finalCanvas.height = cropH;
    const finalCtx = finalCanvas.getContext("2d");

    if (finalCtx) {
      finalCtx.drawImage(
        canvas,
        cropX,
        cropY,
        cropW,
        cropH,
        0,
        0,
        cropW,
        cropH,
      );

      canvas.width = canvas.height = 0;
      try {
        const blob = format === "tif"
          ? geoTiff(finalCtx, originX+cropX, originY+cropY, z, rectify)
          : await new Promise<Blob | null>(resolve => finalCanvas.toBlob(resolve, "image/png"));
        if (!blob) throw new Error("图片编码失败");
        const url = URL.createObjectURL(blob);
        const link = document.createElement("a");
        link.href = url;
        link.download = `${name}_${activeBaseMap.id}_z${z}_${output}.${format}`;
        link.click();
        setTimeout(() => URL.revokeObjectURL(url), 60000);
        statusText.value = "下载完成";
      } catch {
        statusText.value = "导出失败，请缩小范围后重试";
      } finally {
        isDownloading.value = false;
      }
    } else {
      isDownloading.value = false;
      statusText.value = "无法创建输出画布";
    }
  };

  const latLonToTile = (lat: number, lon: number, zoom: number): TileCoord => {
    const x = Math.floor(((lon + 180) / 360) * Math.pow(2, zoom));
    const y = Math.floor(
      ((1 -
        Math.log(
          Math.tan((lat * Math.PI) / 180) + 1 / Math.cos((lat * Math.PI) / 180),
        ) /
          Math.PI) /
        2) *
        Math.pow(2, zoom),
    );
    return { x, y };
  };

  const project = (lat: number, lng: number, zoom: number): Point => {
    let siny = Math.sin((lat * Math.PI) / 180);
    siny = Math.min(Math.max(siny, -0.9999), 0.9999);
    return {
      x: (256 * Math.pow(2, zoom) * (lng + 180)) / 360,
      y:
        256 *
        Math.pow(2, zoom) *
        (0.5 - Math.log((1 + siny) / (1 - siny)) / (4 * Math.PI)),
    };
  };

  const drawGeoJSONPath = (
    ctx: CanvasRenderingContext2D,
    geometry: GeoJSONResult,
    converter: (lat: number, lng: number) => Point,
  ) => {
    if (geometry.type === "Polygon") {
      drawPolygonCoordinates(ctx, geometry.coordinates, converter);
    } else if (geometry.type === "MultiPolygon") {
      geometry.coordinates.forEach((polyCoords: any[]) => {
        drawPolygonCoordinates(ctx, polyCoords, converter);
      });
    }
  };

  const drawPolygonCoordinates = (
    ctx: CanvasRenderingContext2D,
    coordinates: any[],
    converter: (lat: number, lng: number) => Point,
  ) => {
    coordinates.forEach((ring: any[]) => {
      ring.forEach((coord, index) => {
        const p = converter(coord[1], coord[0]);
        if (index === 0) ctx.moveTo(p.x, p.y);
        else ctx.lineTo(p.x, p.y);
      });
      ctx.closePath();
    });
  };
</script>

<style scoped>
  .switch {
    font-size: 12px;
    position: relative;
    display: inline-block;
    width: 3.5em;
    height: 2em;
  }

  .switch input {
    opacity: 0;
    width: 0;
    height: 0;
  }

  .slider {
    position: absolute;
    cursor: pointer;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: #e5e7eb;
    transition: 0.4s;
    border-radius: 30px;
    overflow: hidden;
  }

  .slider:before {
    position: absolute;
    content: "";
    height: 1.4em;
    width: 1.4em;
    border-radius: 20px;
    left: 0.3em;
    bottom: 0.3em;
    transition: 0.4s;
    transition-timing-function: cubic-bezier(0.81, -0.04, 0.38, 1.5);
    background-color: #fff;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }

  .switch input:checked + .slider {
    background-color: #3d3d3d;
  }

  .switch input:checked + .slider:before {
    transform: translateX(1.5em);
    background-color: transparent;
    box-shadow: inset 8px -4px 0px 0px #fff;
  }

  .slider::after {
    content: "☀️";
    position: absolute;
    right: 0.5em;
    top: 50%;
    transform: translateY(-50%);
    font-size: 0.8em;
    opacity: 1;
    transition: opacity 0.3s;
  }

  .switch input:checked + .slider::after {
    opacity: 0;
  }

  :deep(.leaflet-bottom) {
    display: none;
  }
</style>