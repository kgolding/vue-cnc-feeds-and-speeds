<template>
  <div>
    <form @submit.prevent="handleSubmit()" class="d-print-none">
      <div class="row">
        <div class="col-sm-4 form-group">
          <label class="sr-only">Material</label>
          <select v-model="materialId" class="form-control" @change="clearMessage()">
            <option
              v-for="material in cncdata.materials"
              :key="material.id"
              :value="material.id"
            >{{ material.name }}</option>
          </select>
        </div>

        <div class="col-sm-2 form-group">
          <label class="sr-only">Flutes</label>
          <select v-model="flutes" class="form-control" @change="clearMessage()">
            <option v-for="i in 6" :key="i" :value="i">{{ i }} {{ i===1 ? "flute" : "flutes" }}</option>
          </select>
        </div>

        <div class="col-sm-3 form-group">
          <label class="sr-only">Speed (RPM)</label>
          <select v-model="speed" class="form-control" @change="clearMessage()">
            <option v-for="s in getSpeeds" :key="s" :value="s">{{ s }} rpm</option>
          </select>
        </div>

        <div class="col-sm-3 form-group">
          <label class="sr-only">Diameter</label>
          <select v-model="diameterId" class="form-control" @change="clearMessage()">
            <option
              v-for="diameter in cncdata.diameters"
              :key="diameter.id"
              :value="diameter.id"
            >{{ diameter.name }}</option>
          </select>
        </div>
      </div>

      <div class="row">
        <div class="col-sm-10">
          <div class="alert alert-info" v-if="message">{{ message }}</div>
        </div>
        <div class="col-sm-2 text-right">
          <button class="btn btn-primary">Calculate</button>
        </div>
      </div>
    </form>

    <div v-if="results.length > 0" class="mt-3">
      <h2>Calculation results</h2>

      <div class="table-responsive-sm">
        <table class="table table-striped">
          <thead>
            <tr>
              <th>Material</th>
              <th>Speed</th>
              <th>Flutes</th>
              <th>Diameter</th>
              <th>Lowest feed</th>
              <th>Highest feed</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="result in results" :key="result.key">
              <td>{{ result.material }}</td>
              <td>{{ result.speed }}</td>
              <td>{{ result.flutes }}</td>
              <td>{{ result.diameter }}</td>
              <td>{{ ipmConvert(result.lowIpm) }}</td>
              <td>{{ ipmConvert(result.highIpm) }}</td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="d-print-none form-row">
        <div class="col-auto">
          <select class="form-control" v-model="mode">
            <option value="mpm">Metres per minute</option>
            <option value="ipm">Inches per minute</option>
          </select>
        </div>
        <div class="col-auto">
          <button
            type="button"
            class="btn btn-warning"
            @click="clearResults()"
          >Clear results</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import data from "./data";

export default {
  data() {
    return {
      mode: "mpm",
      cncdata: data,
      speed: 16000,
      flutes: 1,
      materialId: 2,
      diameterId: 2,
      message: null,
      results: []
    };
  },
  computed: {
    getSpeeds() {
      const speeds = [];
      for (var s = 10000; s <= 20000; s += 500) {
        speeds.push(s);
      }
      return speeds;
    }
  },
  watch: {
    mode(mode) {
      localStorage.setItem("cnc-mode", mode);
    }
  },
  methods: {
    ipmConvert(ipm) {
      switch (this.mode) {
        case "mpm":
          return (ipm * 0.0254).toFixed(3) + " m/min";
      }
      return ipm + " ipm";
    },
    clearResults() {
      this.results = [];
      localStorage.setItem("cnc-results", "[]");
    },
    clearMessage() {
      this.message = null;
    },
    handleSubmit() {
      // Unique result key
      const key = [
        this.speed,
        this.flutes,
        this.materialId,
        this.diameterId
      ].join(".");

      const result = this.cncdata.results.find(
        r =>
          r.speed === this.speed &&
          r.flutes === this.flutes &&
          r.material === this.materialId &&
          r.diameter === this.diameterId
      );

      if (!result) {
        this.message = "No recommendation available or not recommended";
        return;
      }

      // Clone the result
      let result2 = Object.assign({}, result);

      const m = this.cncdata.materials.find(m => m.id === result.material);
      result2.material = m.name || "?";

      const d = this.cncdata.diameters.find(d => d.id === result.diameter);
      result2.diameter = d.name || "?";

      // Remove existing result if exists
      this.results = this.results.filter(r => r.key !== key);
      result2.key = key;

      this.results.unshift(result2);
      localStorage.setItem("cnc-results", JSON.stringify(this.results));
    }
  },
  created() {
    const results = localStorage.getItem("cnc-results");
    if (results) {
      this.results = JSON.parse(results);
    }
    const mode = localStorage.getItem("cnc-mode");
    if (mode) {
      this.mode = mode;
    }
  }
};
</script>
